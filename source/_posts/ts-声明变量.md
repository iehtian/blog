---
title: ts 声明变量
tags:
	- TypeScript
	- 变量
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
date: 2025-12-11 23:34:25
updated: 2025-12-12 19:25:57
categories:
	- TypeScript
keywords: ts, typescript, 变量, 声明
description: 在已经熟悉 JavaScript 的前提下，快速回顾 TypeScript 中变量声明的主要差异和写法。
top_img:
cover:
copyright_author_href:
copyright_url:
copyright_info:
---

已熟悉 JS 变量声明，下面集中看 TS 的写法和特性。

## 基础声明

```ts
// 推荐使用 let / const，避免 var
const siteName: string = "my-blog"
let counter: number = 0

// 类型推断：未写类型也能推断出 number
let speed = 30
// speed = "fast" // 会报错

// 可选的 undefined/null，需要打开严格模式或显式联合
let maybeId: number | undefined
```

## 字面量与联合类型

```ts
// 字面量类型把值锁定在有限集合
type Theme = "light" | "dark"
let theme: Theme = "light"

// 联合类型让变量接受多种形态
let id: string | number = 42
id = "user-42"
```

## 数组与元组

```ts
// 两种等价数组写法
const names: string[] = []
const scores: Array<number> = []

// 元组：长度和每个位置的类型都固定
const position: [number, number] = [116.4, 39.9]

// 只读数组与元组
const readonlyTags: readonly string[] = ["ts", "notes"]
const readonlyPoint: readonly [number, number] = [0, 1]
```

## 对象与接口

```ts
// 内联类型
const user: { id: number; name: string; readonly role?: string } = {
	id: 1,
	name: "Alice",
	role: "admin",
}

// 接口/类型别名复用
interface User {
	id: number
	name: string
	role?: "admin" | "guest"
}

const bob: User = { id: 2, name: "Bob" }
```

## 类型断言与非空断言

```ts
// 类型断言：告诉编译器我们更具体的类型
const raw = JSON.parse('{"id":3,"name":"Eve"}') as User

// 非空断言：在确认不为 null/undefined 时使用
declare const el: HTMLElement | null
el!.style.display = "block"
```

## 其他小提示

- 尽量依赖类型推断，只有在边界和公共接口处显式标注。
- 为外部数据、接口入参写清楚类型，减少运行期错误。
- 用 `readonly` 防止被无意修改，用 `?` 标记可选字段。