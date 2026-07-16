---
title: Rust 迭代器入门：Iterator trait、适配器与惰性求值
date: 2026-07-02
updated: 2026-07-02
tags: [Rust, Iterator, 迭代器, 适配器]
categories: rust
keywords: Rust Iterator, 迭代器, IntoIterator, 惰性求值, iterator adaptor, consuming adaptor, collect, map, filter
description: 从零理解 Rust 迭代器：Iterator trait 的 next 机制、IntoIterator 与 for 循环脱糖、iterator adaptors 与 consuming adaptors 的分类与链式调用。
top_img:
comments: true
cover: https://picsum.photos/id/1050/800/450
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

## 1. Iterator trait：一切从 next 开始

`Iterator` trait 的定义（标准库简化）：

```rust
pub trait Iterator {
    type Item;                          // 迭代产生的元素类型

    fn next(&mut self) -> Option<Self::Item>;  // 核心：必须实现的方法

    // 其他 70+ 个方法都有默认实现，基于 next 构建
}
```

**整个迭代器体系只靠一个方法驱动：`next()`。** 每次调用推进迭代器一步，返回 `Some(元素)`，耗尽后返回 `None`。

手动推进迭代器：

```rust
let v = vec![1, 2, 3];
let mut iter = v.iter();    // 创建迭代器（此时未做任何事）

assert_eq!(iter.next(), Some(&1));  // 推进第一步
assert_eq!(iter.next(), Some(&2));  // 推进第二步
assert_eq!(iter.next(), Some(&3));  // 推进第三步
assert_eq!(iter.next(), None);      // 耗尽
```

- `iter.next()` 需要 `&mut self`——迭代器自身有状态，记录"走到了哪里"
- 返回值是 `Option<Self::Item>`，这就是为什么迭代器天然处理"可能有、也可能没有下一个"的场景
- 你自己通常不直接写 `next()`，由 `for` 循环或 consuming adaptor 代劳

## 2. 创建迭代器：IntoIterator 与三种 iter 方法

`for` 循环不直接依赖 `Iterator`，而是依赖 `IntoIterator` trait：

```rust
pub trait IntoIterator {
    type Item;
    type IntoIter: Iterator<Item = Self::Item>;

    fn into_iter(self) -> Self::IntoIter;
}
```

`for item in collection` 的脱糖过程：

```rust
// 你写的：
for item in collection {
    println!("{}", item);
}

// 编译器展开为：
let mut iter = collection.into_iter();
loop {
    match iter.next() {
        Some(item) => println!("{}", item),
        None => break,
    }
}
```

### 三种迭代方式

`Vec<T>` 是 Rust 中最常见的可迭代类型，它提供了三种创建迭代器的方法：

| 方法 | 签名 | 返回类型 | 所有权 | 用途 |
|---|---|---|---|---|
| `iter()` | `&self` | `Iter<'_, T>` → 元素为 `&T` | 借用，不转移 | 只读遍历 |
| `iter_mut()` | `&mut self` | `IterMut<'_, T>` → 元素为 `&mut T` | 可变借用 | 遍历并修改 |
| `into_iter()` | `self` | `IntoIter<T>` → 元素为 `T` | 消耗容器 | 转移所有权 |

```rust
let v = vec![1, 2, 3];

// iter()：只读借用
for x in v.iter() {
    println!("{}", x);       // x 是 &i32 类型
}
// v 仍可用

// iter_mut()：可变借用
let mut v = vec![1, 2, 3];
for x in v.iter_mut() {
    *x += 1;                 // x 是 &mut i32，解引用后修改
}
// v 变为 [2, 3, 4]，仍可用

// into_iter()：消耗所有权
for x in v.into_iter() {
    println!("{}", x);       // x 是 i32（转移出来了）
}
// v 已不可用——所有权被消耗
```

常见容器的三种方式行为一致：`Vec`、`HashMap`、`HashSet`、`String` 等均遵循此模式。

- `for item in &v` 等价于 `v.iter()`，`for item in &mut v` 等价于 `v.iter_mut()`
- `into_iter()` 消耗容器后不能再使用——这是 Rust 所有权系统的保证

## 3. Iterator Adaptors：惰性转换器

**Iterator adaptor 接收迭代器、返回新迭代器，不立即执行。** 它们只是一个"转换计划"，直到某个 consuming adaptor 驱动时才真正跑起来。

核心语义：

```rust
let v = vec![1, 2, 3, 4, 5];
let mapped = v.iter().map(|x| x * 2);   // 什么都没算，只构建了包装
// mapped 是一个 Map<Iter<...>> 类型，此时 x * 2 尚未执行
```

必须有消费动作才会驱动计算：

```rust
let result: Vec<_> = mapped.collect();  // collect 驱动 map 执行
// result = [2, 4, 6, 8, 10]
```

### 常用 Iterator Adaptors

**map** — 逐元素变换：

```rust
let v: Vec<i32> = (0..5).map(|x| x * 10).collect();
// [0, 10, 20, 30, 40]
```

**filter** — 按条件筛选：

```rust
let evens: Vec<_> = (0..10).filter(|x| *x % 2 == 0).collect();
// [0, 2, 4, 6, 8]
// filter 闭包返回 bool，返回 true 的元素保留
```

**take / skip** — 取前 n 个 / 跳过前 n 个：

```rust
let v: Vec<_> = (0..10).take(3).collect();  // [0, 1, 2]
let v: Vec<_> = (0..10).skip(5).collect();  // [5, 6, 7, 8, 9]
```

**enumerate** — 为每个元素附加索引：

```rust
for (i, val) in ['a', 'b', 'c'].iter().enumerate() {
    println!("{}: {}", i, val);
}
// 0: a
// 1: b
// 2: c
```

**chain** — 拼接两个迭代器：

```rust
let v: Vec<_> = (1..3).chain(10..13).collect();
// [1, 2, 10, 11, 12]
```

**zip** — 将两个迭代器按位配对：

```rust
let v: Vec<_> = (0..3).zip(['a', 'b', 'c']).collect();
// [(0, 'a'), (1, 'b'), (2, 'c')]
// 任一迭代器耗尽即停止——这里到索引 2 为止
```

- Iterator adaptor 可以无限链式串联，每个返回新的迭代器类型
- 闭包参数命名保持简洁（`x`、`v`、`i`），链式调用中不引入无意义变量名

## 4. Consuming Adaptors：消费适配器

**Consuming adaptor 是迭代器管道的终点**——它调用 `next()` 循环推进，最终消耗迭代器，返回一个具体值或集合。一旦调用，之前的惰性链条全部执行。

### 4.1 收集与聚合

**collect** — 将迭代器收集到集合：

```rust
let squares: Vec<i32> = (1..5).map(|x| x * x).collect();
// [1, 4, 9, 16]

// 也可收集到 HashMap
use std::collections::HashMap;
let map: HashMap<_, _> = [(1, "a"), (2, "b")].into_iter().collect();
// {1: "a", 2: "b"}
```

`collect` 的目标类型靠类型标注推断——`Vec<i32>`、`HashMap<_, _>`、`String` 等，只要目标类型实现了 `FromIterator`。

**sum / product** — 求和 / 求积：

```rust
let total: i32 = (1..=100).sum();      // 5050
let prod: i32 = (1..=5).product();     // 120
```

**count** — 计数：

```rust
let n = (0..100).filter(|x| x % 2 == 0).count();
// 50（0 到 99 中的偶数个数）
```

### 4.2 查找与判定

**find** — 查找第一个满足条件的元素：

```rust
let first = (0..100).find(|x| x % 7 == 0 && x > 50);
// Some(56)
// 返回 Option，找不到返回 None
```

**any / all** — 存在性 / 全称判定：

```rust
let has_even = [1, 3, 5, 8].iter().any(|x| x % 2 == 0);  // true
let all_even  = [2, 4, 6, 8].iter().all(|x| x % 2 == 0);  // true
let all_even  = [2, 4, 5, 8].iter().all(|x| x % 2 == 0);  // false
```

- `any` / `all` 有短路语义：`any` 遇到第一个 true 立即停止，`all` 遇到第一个 false 立即停止

### 4.3 折叠与遍历

**fold** — 带初始值的累积计算：

```rust
fn fold<B, F>(self, init: B, f: F) -> B
where F: FnMut(B, Self::Item) -> B;
```

```rust
// 累加（等价于 sum）
let total = (1..=5).fold(0, |acc, x| acc + x);  // 15

// 拼接字符串
let s = ['h', 'e', 'l', 'l', 'o'].iter()
    .fold(String::new(), |mut acc, c| {
        acc.push(*c);
        acc
    });
// "hello"
```

`fold` 的执行顺序（以 `[a, b, c].fold(init, f)` 为例）：

```rust
// 等价于：
// let acc = init;
// let acc = f(acc, a);
// let acc = f(acc, b);
// let acc = f(acc, c);
// acc
```

**for_each** — 对每个元素执行副作用操作：

```rust
(0..5).for_each(|x| println!("{}", x));

// 等价于：
// for x in 0..5 { println!("{}", x); }
```

`for_each` 与 `for` 循环功能等价，但在链式调用中更自然：

```rust
(0..10)
    .filter(|x| x % 2 == 0)
    .map(|x| x * x)
    .for_each(|x| println!("{}", x));
// 0 4 16 36 64
```

## 5. 链式调用：组合 Iterator Adaptors 与 Consuming Adaptors

迭代器的真正威力体现在链式组合——多个 adaptor 串联成一个管道，最后由 consuming adaptor 驱动执行：

```rust
let v: Vec<i32> = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 需求：取前 8 个元素 → 保留偶数 → 平方 → 求和
let total: i32 = v.iter()       // 创建迭代器（不执行）
    .take(8)                    // 取前 8 个（不执行）        [1,2,3,4,5,6,7,8]
    .filter(|x| *x % 2 == 0)   // 保留偶数（不执行）          [2,4,6,8]
    .map(|x| x * x)            // 平方（不执行）              [4,16,36,64]
    .sum();                    // ← 只有这里才真正驱动执行    120

println!("sum of squares of even numbers in first 8: {}", total);
// 120
```

整个链条在 `.sum()` 之前没有任何元素被处理——这是一个完整的惰性管道。

**惰性求值的两大优势：**

- **性能**：不会为中间步骤分配临时 `Vec`，每个元素一路走完整个管道再到下一个元素
- **组合性**：adaptor 可以自由组合、复用，行为由最后一步的 consuming adaptor 决定

对比手动循环写法：

```rust
// 迭代器链式写法：声明式，意图清晰
let total: i32 = v.iter().take(8).filter(|x| *x % 2 == 0).map(|x| x * x).sum();

// 等价的 for 循环写法：命令式，变量和流程交织
let mut total = 0;
for (i, x) in v.iter().enumerate() {
    if i >= 8 { break; }
    if x % 2 == 0 {
        total += x * x;
    }
}
```

两种写法等价，但迭代器链声明式表达"做什么"而非"怎么做"，意图更直观。

## 6. 小结

- **Iterator trait** 的核心是 `next()`，返回 `Option<Item>`。所有其他方法基于 `next` 默认实现
- **创建迭代器**通过 `iter()`（只读借用）、`iter_mut()`（可变借用）、`into_iter()`（消耗所有权），`for` 循环脱糖为 `into_iter()` + `next()` 循环
- **Iterator Adaptors**（`map`、`filter`、`take`、`skip` 等）是惰性转换器——不调用就不执行
- **Consuming Adaptors**（`collect`、`sum`、`find`、`fold`、`for_each` 等）是管道终点——驱动整个链条执行
- 链式组合实现声明式数据处理，中间不产生临时分配
