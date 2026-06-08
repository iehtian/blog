---
title: C++类初始化方法
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
date: 2026-05-26 23:44:15
categories: [C++]
keywords: C++, 类初始化, 列表初始化, 构造函数, 成员初始化列表, new动态初始化
description: 介绍 C++ 中类实例化时的几种初始化方式及其区别
---

C++ 提供了多种初始化类对象的方式，不同方式的语义和行为有微妙差异。本文逐一介绍。

## 默认初始化

当不提供任何初始值时发生：

```cpp
class Foo {
public:
    int x;
    std::string s;
};

Foo f;  // 默认初始化
// f.x 未定义，f.s 为空字符串（string 有默认构造函数）
```

注意：内置类型（`int`、`double`、指针等）在默认初始化时值是未定义的，读取属于未定义行为。

## 值初始化

使用空括号或 `{}` 触发，内置类型会被零初始化：

```cpp
Foo f1();    // 注意：这是函数声明，不是对象！
Foo f2{};    // 值初始化：f2.x == 0，f2.s == ""
Foo f3 = {}; // 同上
```

注意：`Foo f1()` 是最令人困惑的解析（Most Vexing Parse），编译器将其视为函数声明。用 `{}` 可以避免这个问题。

## 直接初始化

在括号中提供参数，直接调用匹配的构造函数：

```cpp
class Foo {
public:
    int x;
    Foo(int val) : x(val) {}
};

Foo f1(42);   // 直接初始化，调用 Foo(int)
Foo f2{42};   // 列表初始化，同样调用 Foo(int)
```

## 拷贝初始化

使用 `=` 触发，先构造临时对象再拷贝（编译器可能优化掉拷贝步骤）：

```cpp
Foo f1 = 42;  // 拷贝初始化
Foo f2 = Foo(42);
Foo f3 = {42};
```

拷贝初始化要求对应的构造函数不是 `explicit` 的，否则编译报错：

```cpp
class Bar {
public:
    explicit Bar(int val) : x(val) {}
    int x;
};

Bar b1(10);   // OK：直接初始化
Bar b2 = 10;  // 错误：explicit 构造函数不能用于拷贝初始化
```

## 列表初始化（C++11）

使用花括号 `{}` 进行初始化，C++11 引入。有两个重要特点：

### 禁止窄化转换

```cpp
class Foo {
public:
    Foo(int val) : x(val) {}
    int x;
};

Foo f1(3.14);  // OK：double 隐式截断为 int
Foo f2{3.14};  // 错误：窄化转换，double → int 丢失信息
```

### 优先匹配 initializer_list 构造函数

```cpp
class Bar {
public:
    Bar(int a, int b) { std::cout << "two ints" << std::endl; }
    Bar(std::initializer_list<int>) { std::cout << "initializer_list" << std::endl; }
};

Bar b1(1, 2);   // 输出 "two ints"
Bar b2{1, 2};   // 输出 "initializer_list"
```

这一点是 `vector` 等容器容易踩坑的地方：

```cpp
std::vector<int> v1(10, 1);   // 10 个元素，每个值为 1
std::vector<int> v2{10, 1};   // 2 个元素：10 和 1
```

## new 动态初始化

使用 `new` 在堆上创建对象时，同样遵循上述初始化规则：

```cpp
Foo* p1 = new Foo;      // 默认初始化，p1->x 未定义
Foo* p2 = new Foo();    // 值初始化，p2->x == 0
Foo* p3 = new Foo(42);  // 直接初始化，调用 Foo(int)
Foo* p4 = new Foo{};    // 值初始化（C++11），同 new Foo()
Foo* p5 = new Foo{42};  // 列表初始化（C++11），同 new Foo(42)
```

`new[]` 创建数组时：

```cpp
Foo* arr1 = new Foo[3];    // 每个元素默认初始化
Foo* arr2 = new Foo[3]();  // 每个元素值初始化
Foo* arr3 = new Foo[3]{};  // 同上（C++11）
```

注意：动态分配的对象必须手动 `delete` 释放，建议优先使用智能指针：

```cpp
auto sp1 = std::make_shared<Foo>();     // 值初始化
auto sp2 = std::make_shared<Foo>(42);   // 直接初始化
auto up = std::make_unique<Foo[]>(3);   // C++20，每个元素值初始化
```

## 聚合初始化

对于聚合类型（无自定义构造函数、无私有/保护成员、无基类、无虚函数），可以直接用 `{}` 初始化成员：

```cpp
struct Point {
    int x;
    int y;
};

Point p1 = {1, 2};       // C++11 之前就支持
Point p2{1, 2};           // C++11
Point p3 = {.x = 1, .y = 2};  // C++20 指定初始化
```

C++11 起也支持对含默认成员初始值的聚合类型初始化：

```cpp
struct Config {
    int width = 800;
    int height = 600;
};

Config c1{};               // width=800, height=600
Config c2{1024};           // width=1024, height=600
Config c3{1024, 768};      // width=1024, height=768
```

## 构造函数中的成员初始化列表

成员初始化列表是构造函数中初始化成员的首选方式，比在构造函数体内赋值更高效：

```cpp
class Person {
    std::string name;
    int age;
public:
    // 成员初始化列表：直接构造
    Person(const std::string& n, int a) : name(n), age(a) {}

    // 构造函数体内赋值：先默认构造再赋值，效率低
    // Person(const std::string& n, int a) {
    //     name = n;
    //     age = a;
    // }
};
```

以下情况必须使用成员初始化列表：

- `const` 成员
- 引用成员
- 没有默认构造函数的类类型成员

```cpp
class Wrapper {
    const int id;
    std::string& ref;
    Person person;
public:
    // 必须用初始化列表
    Wrapper(int i, std::string& r, const std::string& name, int age)
        : id(i), ref(r), person(name, age) {}
};
```

成员的初始化顺序取决于声明顺序，而非初始化列表中的书写顺序。写错顺序可能导致未定义行为：

```cpp
class Bad {
    int b;
    int a;
public:
    Bad(int val) : a(val), b(a) {}  // 危险！b 先于 a 初始化
    // b 初始化时 a 还是未定义值
};
```

## C++11 类内默认初始值

可以在类定义中直接为成员提供默认值：

```cpp
class Settings {
    int timeout = 30;
    bool verbose = false;
    std::string host = "localhost";
public:
    Settings() = default;                    // 使用所有默认值
    Settings(int t) : timeout(t) {}          // 只改 timeout，其余用默认值
};
```

这比在多个构造函数的初始化列表中重复写默认值更简洁，也避免了遗漏。

## 小结

| 方式 | 语法 | 关键点 |
|------|------|--------|
| 默认初始化 | `Foo f;` | 内置类型值未定义 |
| 值初始化 | `Foo f{};` | 内置类型零初始化 |
| 直接初始化 | `Foo f(42);` | 直接调用构造函数 |
| 拷贝初始化 | `Foo f = 42;` | 受 `explicit` 限制 |
| 列表初始化 | `Foo f{42};` | 禁止窄化，优先匹配 `initializer_list` |
| new 动态初始化 | `new Foo(42);` | 堆上分配，遵循相同初始化规则 |
| 聚合初始化 | `Point p{1,2};` | 适用于聚合类型 |
| 成员初始化列表 | `: name(n)` | 构造函数中首选方式 |
