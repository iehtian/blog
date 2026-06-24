---
title: C++类的隐式类型转换
tags: []
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
date: 2026-01-15 20:29:31
updated: 2026-01-28 20:38:57
categories:
keywords:
description:
top_img:
cover: https://picsum.photos/id/103/800/450
copyright_author_href:
copyright_url:
copyright_info:
---
```
class MyClass {
public:
    operator int() const { return 42; }
    operator double() const { return 3.14; }
    operator bool() const { return true; }
};

MyClass obj;

// 情况1：赋值给特定类型的变量（看左值类型）
int i = obj;        // 需要 int，调用 operator int()
double d = obj;     // 需要 double，调用 operator double()
bool b = obj;       // 需要 bool，调用 operator bool()

// 情况2：函数参数（看形参类型）
void func(int x) { }
void func2(double x) { }

func(obj);          // 需要 int，调用 operator int()
func2(obj);         // 需要 double，调用 operator double()

// 情况3：运算符上下文（看运算符需要什么类型）
if (obj) { }        // if 需要 bool，调用 operator bool()
int result = obj + 10;  // + 运算，调用 operator int()
```
具体来说，将该对象转换为什么样的类型由编译器决定。
在类中使用诸如 `operator ty() const` 的形式可以将对象进行隐式类型转换