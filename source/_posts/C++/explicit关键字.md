---
title: C++ explicit 关键字
tags: [C++]
comments: true
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
date: 2026-05-26 23:50:00
categories: [C++]
keywords: C++, explicit, 隐式转换, 构造函数, 转换运算符, 拷贝初始化
description: 介绍 C++ explicit 关键字的作用、用法及最佳实践
cover: https://picsum.photos/id/104/800/450
---

C++ 中的 `explicit` 关键字用于禁止构造函数和转换运算符的隐式转换。本文介绍它解决的问题、具体用法以及最佳实践。

## 1. 隐式转换的问题

当一个类拥有单参数构造函数时，编译器可以自动将参数类型隐式转换为该类类型。这在某些场景下会带来意想不到的行为：

```cpp
class String {
public:
    String(const char* str) { /* ... */ }
};

void print(const String& s) { /* ... */ }

print("hello");  // OK：const char* 隐式转换为 String
```

看起来很方便，但考虑这个例子：

```cpp
class Buffer {
public:
    Buffer(int size) : capacity(size) {}
    int capacity;
};

void process(Buffer b) { /* ... */ }

process(100);     // 意图不明：是想传入大小为 100 的 Buffer，还是误传了整数？
Buffer b = 50;    // 同样令人困惑
```

`100` 被隐式转换成了一个 `Buffer` 对象，这种转换很可能不是程序员的本意，且难以排查。

## 2. explicit 构造函数

在构造函数前加 `explicit`，即可禁止隐式转换：

```cpp
class Buffer {
public:
    explicit Buffer(int size) : capacity(size) {}
    int capacity;
};

Buffer b1(100);   // OK：直接初始化
Buffer b2 = 100;  // 错误：explicit 禁止拷贝初始化中的隐式转换

process(100);     // 错误：不能隐式转换
process(Buffer(100));  // OK：显式构造
```

对比加与不加 `explicit` 的行为：

```cpp
class A {
public:
    A(int x) : val(x) {}
    int val;
};

class B {
public:
    explicit B(int x) : val(x) {}
    int val;
};

A a = 10;   // OK：隐式转换
B b = 10;   // 错误：explicit 禁止隐式转换
```

## 3. explicit 与拷贝初始化

`explicit` 构造函数不能用于拷贝初始化（`=` 语法），但可以直接初始化（`()` 或 `{}` 语法）：

```cpp
class Foo {
public:
    explicit Foo(int x) : val(x) {}
    int val;
};

Foo f1(42);    // OK：直接初始化
Foo f2{42};    // OK：列表初始化（直接初始化的一种）
Foo f3 = 42;   // 错误：拷贝初始化，explicit 构造函数不允许
Foo f4 = Foo(42);  // OK：右侧是显式构造的临时对象，拷贝的是同类对象
```

注意：`Foo f4 = Foo(42)` 之所以合法，是因为右侧已经显式构造了一个 `Foo` 临时对象，这里发生的是同类对象之间的拷贝初始化，而非从 `int` 的隐式转换。

函数传参和返回值同样受影响：

```cpp
void func(Foo f) {}

func(42);         // 错误：隐式转换
func(Foo(42));    // OK：显式构造
func(static_cast<Foo>(42));  // OK：static_cast 也算显式转换
```

## 4. explicit 转换运算符（C++11）

C++11 起，`explicit` 也可用于转换运算符。最常见的用途是 `explicit operator bool()`：

```cpp
class SmartPtr {
public:
    operator bool() const { return ptr != nullptr; }
    void* ptr;
};

SmartPtr p;
int n = p;  // 危险：bool 可继续隐式转换为 int
```

加上 `explicit` 后：

```cpp
class SmartPtr {
public:
    explicit operator bool() const { return ptr != nullptr; }
    void* ptr;
};

SmartPtr p;
if (p) { /* ... */ }     // OK：if 条件上下文允许显式 bool 转换
while (p) { /* ... */ }  // OK：同上
bool b = static_cast<bool>(p);  // OK：显式转换
int n = p;               // 错误：不能隐式转换为 bool 再转 int
```

C++ 标准规定，`explicit operator bool()` 在以下上下文中可以当作隐式转换使用（称为"上下文转换为 bool"）：

- `if`、`while`、`for` 的条件部分
- `!`、`&&`、`||` 的操作数
- 三元运算符 `?:` 的条件部分

这就是标准库智能指针（`std::unique_ptr`、`std::shared_ptr`）的实现方式。

## 5. C++20 的改进

C++20 允许 `explicit` 接受一个布尔常量表达式参数，实现条件性 `explicit`：

```cpp
template<typename T>
class Wrapper {
public:
    // 当 T 可隐式转换时，构造函数不加 explicit；否则加 explicit
    explicit(!std::is_convertible_v<T, int>)
    Wrapper(T val) : value(static_cast<int>(val)) {}

    int value;
};
```

这在模板编程中非常实用，可以根据类型特征决定是否允许隐式转换，而不需要写两个构造函数。

## 6. 最佳实践

- **单参数构造函数几乎总是应该加 `explicit`**，除非你明确需要隐式转换（如 `String(const char*)` 这种语义上自然等价的转换）
- **转换运算符优先使用 `explicit operator bool()`**，避免 bool 继续隐式转换为整数
- **多参数构造函数也可以加 `explicit`**，C++11 起生效（禁止从花括号列表的隐式转换）
- **不要为了方便而省略 `explicit`**，隐式转换带来的 bug 往往难以定位

```cpp
class Point {
public:
    // 多参数构造函数加 explicit，禁止 Point p = {1, 2} 风格的隐式转换
    explicit Point(int x, int y) : x_(x), y_(y) {}
    int x_, y_;
};

Point p1(1, 2);      // OK
Point p2 = {1, 2};   // 错误：explicit 禁止
Point p3{1, 2};      // 错误（C++11 起，explicit 也禁止列表初始化的隐式转换）
```
