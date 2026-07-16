---
title: Rust `?` 运算符：错误传播的快捷方式
date: 2026-07-02
updated: 2026-07-02
tags: [Rust, error-handling, Result, Option, try-operator]
categories: rust
keywords: Rust, ? 运算符, 问号运算符, try 运算符, Result, Option, 错误传播, error propagation, From trait, anyhow
description: 从 match 样板代码出发，理解 ? 运算符的等价展开、错误传播链路、From 自动类型转换，以及 Option 上的使用方式与常见模式。
top_img:
comments: true
cover: https://picsum.photos/id/1055/800/450
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
copyright_author_href:
copyright_url:
copyright_info:
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
---

## 1. ? 是什么

`?` 是 Rust 中**传播错误的语法糖**。它把两件事压缩成一个字符：

- `Ok(v)` → 取出 `v`，继续执行
- `Err(e)` → 从当前函数 `return Err(e.into())`，不继续执行

```rust
fn read_config() -> Result<String, std::io::Error> {
    let content = std::fs::read_to_string("config.toml")?; // 如果 Err，直接向上返回
    Ok(content)
}
```

一行 `?` 替代了手写 match 的五行样板。

## 2. 等价展开：`?` → `match`

编译器对 `?` 的机械展开如下：

```rust
// expression?  展开为 ↓

match expression {
    Ok(v) => v,
    Err(e) => return Err(From::from(e)),
}
```

关键点：

- **`From::from(e)`** —— 调用 `From` trait 做错误类型转换。这意味着 `?` 不仅能传播错误，还能自动把一种错误类型转成另一种。
- **`return`** —— 直接退出当前函数，不是 `break`，不是 `continue`。

### 2.1 自动类型转换：From trait 的作用

```rust
fn parse_and_read(path: &str) -> Result<String, Box<dyn std::error::Error>> {
    let num: i32 = "42".parse()?;          // ParseIntError → Box<dyn Error>
    let content = std::fs::read_to_string(path)?; // io::Error → Box<dyn Error>
    Ok(content)
}
```

两种不同的错误类型（`ParseIntError`、`io::Error`）都能通过 `?` 转成同一个 `Box<dyn Error>`，因为标准库为它们实现了 `From<T> for Box<dyn Error>`。

提示：`Box<dyn Error>` 是快速原型阶段的常用"万能错误类型"，正式项目推荐用 `anyhow`（应用层）或 `thiserror`（库层）替代。

### 2.2 与 unwrap 的区别

| | `?` | `unwrap` |
|---|---|---|
| 行为 | 传播错误，不 panic | `Err` 时 panic |
| 返回类型要求 | 函数必须返回 `Result`/`Option` | 无要求 |
| 适用场景 | 可恢复错误，向上传递 | 逻辑上不可能出错处 |

```rust
// unwrap：出错就崩
let v = fallible_op().unwrap();

// ?：出错交给调用者处理
let v = fallible_op()?;
```

## 3. 错误传播链路

多层函数调用中，`?` 让错误自动向上冒泡，无需每层都 match：

```rust
use std::fs;
use std::io;

fn read_username(path: &str) -> Result<String, io::Error> {
    Ok(fs::read_to_string(path)?)
}

fn load_user(path: &str) -> Result<User, io::Error> {
    let name = read_username(path)?; // read_username 的 Err 穿透这里
    Ok(User { name })
}

fn init_app(path: &str) -> Result<App, io::Error> {
    let user = load_user(path)?;     // read_username → load_user 的 Err 最终到达这里
    Ok(App { user })
}
```

同一份错误会沿着调用栈逐层向上，每层只加一个 `?`，无需 if/else 嵌套。

对比不用 `?` 的等价写法：

```rust
fn load_user(path: &str) -> Result<User, io::Error> {
    let name = match read_username(path) {
        Ok(n) => n,
        Err(e) => return Err(e),
    };
    Ok(User { name })
}
```

每层多 4 行，且错误类型不同时还需要手动 `map_err`。

## 4. 何时能用 ?

### 4.1 函数返回类型约束

`?` 只能在返回类型为 `Result` 或 `Option` 的函数中使用——编译器需要知道早期返回应该返回什么：

```rust
// ✅ 返回 Result
fn a() -> Result<i32, MyError> {
    let x = fallible()?;
    Ok(x)
}

// ✅ 返回 Option
fn b() -> Option<i32> {
    let x = maybe()?;  // None 时 return None
    Some(x)
}

// ❌ 编译错误：函数不返回 Result/Option
fn c() -> i32 {
    let x = fallible()?; // error[E0277]: the `?` operator can only be used in a function
    x                     // that returns `Result` or `Option`
}
```

### 4.2 main 返回 Result

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string("config.toml")?; // main 里也能用 ?
    println!("{}", content);
    Ok(())
}
```

`main` 返回 `Result` 时，`Err` 会被运行时打印并设置非零退出码。

### 4.3 Option 上的 ?

`?` 对 `Option<T>` 的行为对称：

```rust
// opt?  展开为 ↓
match opt {
    Some(v) => v,
    None => return None,
}
```

示例——链式提取多层 Option：

```rust
fn city_of(user: &User) -> Option<&str> {
    Some(user.address.as_ref()?.city.as_str())
    // address 是 None → 直接 return None
    // address 是 Some → 取 city 字段
}
```

## 5. 常用模式

### 5.1 ? 与 map_err 配合

当错误类型不匹配且没有 `From` 实现时，用 `map_err` 手动转换：

```rust
fn parse_config(raw: &str) -> Result<Config, ConfigError> {
    let port: u16 = raw
        .lines()
        .find(|l| l.starts_with("port="))?
        .strip_prefix("port=")?
        .parse()
        .map_err(|e| ConfigError::Parse(format!("invalid port: {e}")))?;
    Ok(Config { port })
}
```

这里 `Option?` 和 `Result?` 混用在同一表达式链中——`find` 和 `strip_prefix` 返回 `Option`，`parse` 返回 `Result`。

### 5.2 闭包中的 ?：类型推断陷阱

闭包中使用 `?`，编译器会推断闭包的返回类型：

```rust
let items = vec!["1", "2", "x", "4"];

// ❌ 编译器报错：闭包返回类型不明确
let nums: Vec<i32> = items
    .iter()
    .map(|s| s.parse::<i32>()?)  // ? 把闭包返回类型推断为 Result<i32, _>
    .collect();                    // 但 Vec<i32> 期望的是 i32，不是 Result

// ✅ 用 filter_map 处理 Option
let nums: Vec<i32> = items
    .iter()
    .filter_map(|s| s.parse::<i32>().ok())
    .collect();
```

`map` 中对每个元素执行闭包，`?` 让闭包返回 `Result<i32, _>`。一旦遇到 `"x"` 解析失败，整个 `map` 链就停止。这种情况要么用 `filter_map` + `.ok()` 跳过失败项，要么在 `map` 外收集 `Result<Vec<_>, _>` 再 `?`。

```rust
// ✅ 收集整个 Result，一处 ? 即可
let nums: Result<Vec<i32>, _> = items.iter().map(|s| s.parse::<i32>()).collect();
let nums = nums?;
```

### 5.3 try 块（不稳定特性）

`try { }` 块允许在非函数出口的局部作用域中使用 `?`，将错误捕获为外层 `Result`：

```rust
#![feature(try_blocks)]

let result = try {
    let a = fallible_a()?;
    let b = fallible_b()?;
    a + b
};
// result: Result<i32, Error>
```

`try` 块内的 `?` 退出的是 `try` 块本身，而非外层函数。

## 6. 小结

`?` 的核心价值是消除错误传播的样板代码：

| 机制 | 作用 |
|---|---|
| `expr?` | `Ok(v) → v`，`Err(e) → return Err(e.into())` |
| `From::from` | 自动转换错误类型，无需手动 `map_err` |
| 类型约束 | 函数必须返回 `Result` 或 `Option` |
| `main` 支持 | `main() → Result<(), E>` 后可在入口函数使用 `?` |

理解 `?` = 理解那一段固定的 match 展开。此后所有用法都是这个展开的组合。
