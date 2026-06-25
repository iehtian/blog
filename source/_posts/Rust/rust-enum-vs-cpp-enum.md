---
title: Rust enum 与 C++ enum 对比：从判别值到带数据的标签联合
date: 2026-06-25
updated: 2026-06-25 21:01:55
tags: [Rust, C++, enum]
categories: rust
keywords: Rust enum, C++ enum, tagged union, std::variant, match, 模式匹配, 标签联合
description: 从 C++ enum 的局限出发，逐步引出 Rust enum（标签联合）的设计动机、核心语法和模式匹配，帮助有 C++ 基础的开发者快速理解 Rust 的 enum 机制。
top_img:
comments: true
cover: https://picsum.photos/id/1020/800/450
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

## 1. C++ enum 的三种形态与共同局限

C++ 的 `enum` 经历了两个阶段：

```cpp
// C 风格 enum —— 裸整型，作用域污染
enum Color { Red, Green, Blue };
int x = Red;  // 隐式转为 int，编译器不报错
```

```cpp
// C++11 enum class —— 作用域限定，禁止隐式转换
enum class Color { Red, Green, Blue };
Color c = Color::Red;  // 必须带作用域
// int x = Color::Red;  // 编译错误！不能隐式转 int
```

`enum class` 解决了命名污染和隐式转换问题，但**两种 enum 有同一个根本局限：变体只能是一个判别值（本质是整数），不能携带关联数据。**

实际场景中这种需求很常见——例如表示 API 调用结果：

```cpp
// 需求：要么成功（带数据），要么失败（带错误信息）
// C 风格 enum 和 enum class 都做不到，只能靠 std::variant 补救

#include <variant>
#include <string>

struct Success { int code; std::string body; };
struct Failure { int code; std::string message; };
using ApiResult = std::variant<Success, Failure>;
```

先定义独立的 `struct`，再用 `std::variant` 包装——数据定义散落各处，解构访问还需要 `std::visit` 配合泛型 lambda（见第 3 节）。

## 2. Rust enum：原生标签联合

Rust 的 `enum` 原生支持标签联合（tagged union），无需任何标准库模板辅助。

### 2.1 基础定义

```rust
enum Color {
    Red,
    Green,
    Blue,
}
```

和 `enum class` 一样通过 `Color::Red` 访问变体，无隐式转换。到此为止差别不大，关键的区别在下面。

### 2.2 携带数据

每个变体可以直接携带不同类型的数据——声明方式和定义 `struct` 一样自然：

```rust
enum ApiResult {
    Success { code: u16, body: String },  // 具名字段
    Redirect(String),                     // 元组字段（目标 URL）
    Error(u16, String),                   // 元组字段（错误码 + 消息）
    Timeout,                              // 无数据
}
```

对比 C++ 的做法：

| | C++ `std::variant` | Rust `enum` |
|---|---|---|
| 声明方式 | 先分别定义 `struct Success`、`struct Failure` 等，再 `variant<Success, Failure, ...>` 包装 | 数据定义和标签在同一个 `enum` 中完成 |
| 文件组织 | 相关类型散落在 `variant` 声明之外 | 逻辑内聚，变体定义即文档 |

### 2.3 构造

```rust
let ok = ApiResult::Success { code: 200, body: "OK".into() };  // 具名字段用 {}
let moved = ApiResult::Redirect("https://new-url.com".into()); // 元组字段用 ()
let err = ApiResult::Error(404, "Not Found".into());           // 多参数元组
let lost = ApiResult::Timeout;                                 // 无数据变体
```

构造语法与声明一一对应，不需要额外记忆模板实参顺序。

## 3. match 模式匹配

### 3.1 基本写法

```rust
let result = ApiResult::Success { code: 200, body: "OK".into() };

match result {
    ApiResult::Success { code, body } => {
        println!("HTTP {}: {}", code, body);
    }
    ApiResult::Redirect(url) => {
        println!("Redirect to: {}", url);
    }
    ApiResult::Error(code, msg) => {
        println!("Error {}: {}", code, msg);
    }
    ApiResult::Timeout => {
        println!("Request timed out");
    }
}
```

- 每个分支 = `变体名(解构模式) => 表达式`
- 解构时直接以模式匹配取出字段，不需要类似 C++ 的 `std::get<Success>(result)` 按类型提取

对照 C++ 的等价写法：

```cpp
std::visit(
    [](auto&& arg) {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, Success>) {
            // 访问 arg.code, arg.body
        } else if constexpr (std::is_same_v<T, Failure>) {
            // 访问 arg.code, arg.message
        }
    },
    result
);
```

Rust 的 `match` 把模式匹配拆解直接做在了语言层面，不需要 `if constexpr` 的模板元编程技巧。

### 3.2 穷尽性检查

编译器强制覆盖**所有变体**——漏了任何一个编译直接失败，没有警告不提示的机会：

```rust
match result {
    ApiResult::Success { .. } => println!("OK"),
    ApiResult::Error(code, _) => println!("{}", code),
    // 忘记处理 Redirect 和 Timeout → 编译错误
}
```

`std::variant` + `std::visit` 也能通过编译器检查穷尽性（`-Werror=switch`），但需要先写好完整的泛型 lambda / 重载集。Rust 的内建检查零额外成本——只要你用了 `match`，穷尽性自动保证。

### 3.3 if let

只关心一个变体时用 `if let`，避免写一整个 `_ => {}` 分支：

```rust
if let ApiResult::Success { code, .. } = result {
    println!("Status: {}", code);
}
```

还支持 `else`、`else if let` 链式组合，以及搭配 `||`、`&&` 的 match guards（`if` 后跟额外条件）。

## 4. 为 enum 实现方法

和 Rust 的 `struct` 完全一样，`impl` 块对 `enum` 同样生效：

```rust
impl ApiResult {
    fn is_success(&self) -> bool {
        matches!(self, ApiResult::Success { .. })
    }

    fn status_code(&self) -> Option<u16> {
        match self {
            ApiResult::Success { code, .. } => Some(*code),
            ApiResult::Error(code, _) => Some(*code),
            _ => None,  // Redirect / Timeout 没有明确的 HTTP 状态码
        }
    }
}
```

- `&self` 指向当前枚举实例，语义和 `struct` 方法一致
- `match self` 在方法内部对不同变体分发逻辑
- `matches!` 宏是 `match` 的快捷版：需要布尔结果时一行替代多行

C++ 无法直接给 `std::variant<Success, Failure>` 添加成员函数——variant 是标准库类型，只能通过自由函数或 wrapper 类绕过。Rust `enum` 是语言原生类型，`impl` 不受限制。

## 5. 对比总结

| | C++ enum / enum class | C++ std::variant | Rust enum |
|---|---|---|---|
| 携带数据 | ❌ | ✅（需单独定义 struct） | ✅（原生） |
| 声明复杂度 | 低 | 高（类型 + variant 包装） | 低（一体化） |
| 穷尽性检查 | — | ✅（visit 编译期检查） | ✅（语言级内置 match） |
| 解构语法 | — | `std::get<T>()` / `std::visit` | `match` / `if let` 模式解构 |
| 方法定义 | — | 不能直接给 variant 加方法 | `impl` 块，和 struct 一致 |

Rust `enum` 本质上等于 C++ 的 `enum class` **加上** `std::variant` 的能力，并且把所有机制统一在语言级语法中——没有模板套模板的声明负担，`match` 解构由编译器保证穷尽。

`Option<T>` 是 Rust 标准库中最经典的 enum 应用：`Some(T)` 代表有值，`None` 代表无值，等价于 C++ 的 `std::optional<T>`。下一篇介绍 `Option` 的进阶用法和组合子操作。
