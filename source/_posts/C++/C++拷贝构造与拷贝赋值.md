---
title: C++拷贝构造与拷贝赋值
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
date: 2026-01-15 20:18:54
updated: 2026-01-28 20:38:57
categories:
keywords:
description:
top_img:
cover: https://picsum.photos/id/101/800/450
copyright_author_href:
copyright_url:
copyright_info:
---

这能禁用拷贝构造和拷贝赋值
```
template <typename T>
class smart_ptr {
…
smart_ptr(const smart_ptr&)
= delete;
smart_ptr& operator=(const smart_ptr&)
= delete;
…
};
```
考虑如下场景
```
smart_ptr<int> p1(new int(42));
smart_ptr<int> p2(p1);        // 调用拷贝构造函数
smart_ptr<int> p3 = p1;       // 也是调用拷贝构造函数（不是赋值！）
```
或者
```
smart_ptr<int> p1(new int(42));
smart_ptr<int> p2(new int(100));
p2 = p1;                      // 调用拷贝赋值运算符
```
那么
```
// 假设允许拷贝
smart_ptr<int> p1(new int(42));  // p1 拥有内存地址 0x1000
smart_ptr<int> p2 = p1;           // p2 也指向 0x1000

// 现在 p1 和 p2 都认为自己拥有这块内存
// 当它们的析构函数被调用时...
// p1 析构：delete 0x1000  ✓ 成功
// p2 析构：delete 0x1000  ✗ 重复释放！程序崩溃
```
这是因为拷贝构造默认是浅拷贝
```
普通成员
class Point {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    // 编译器自动生成的拷贝构造相当于：
    // Point(const Point& other) : x(other.x), y(other.y) {}
};

Point p1(10, 20);
Point p2(p1);  // p2.x = 10, p2.y = 20
```
```
指针成员
class Bad {
    int* ptr;
public:
    Bad(int val) : ptr(new int(val)) {}
    ~Bad() { delete ptr; }
    // 编译器生成的拷贝构造：
    // Bad(const Bad& other) : ptr(other.ptr) {}  // 只拷贝地址！
};

Bad b1(42);
Bad b2(b1);  // b1.ptr 和 b2.ptr 指向同一块内存
// 析构时崩溃：同一内存被释放两次
```
拷贝赋值同理