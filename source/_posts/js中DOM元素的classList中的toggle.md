---
title: js 中 DOM 元素的 classList.toggle
tags: [js, DOM]
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
date: 2026-01-28 20:49:40
updated: 2026-01-28 20:56:50
categories: [前端基础]
keywords: [classList, toggle, class]
description: 讲解 classList.toggle 的用法、参数与常见场景（展开/折叠、开关态）。
top_img:
cover: https://picsum.photos/id/108/800/450
copyright_author_href:
copyright_url:
copyright_info:
---

**概览**
`classList.toggle()` 用于在元素上添加或移除某个 class，非常适合“展开/折叠”“开关态”这类交互场景。

## 基本用法

```js
const panel = document.querySelector('.panel')
panel.classList.toggle('is-open')
```

如果 `is-open` 不存在就添加，存在则移除。

## 强制设置（第二个参数）

`toggle` 支持一个可选的布尔值参数，用于强制添加或移除：

```js
const panel = document.querySelector('.panel')

// 强制添加
panel.classList.toggle('is-open', true)

// 强制移除
panel.classList.toggle('is-open', false)
```

## 常见场景：按钮开关

```html
<button id="btn">切换</button>
<div id="panel" class="panel">内容</div>
```

```js
const btn = document.querySelector('#btn')
const panel = document.querySelector('#panel')

btn.addEventListener('click', () => {
	panel.classList.toggle('is-open')
})
```

```css
.panel { display: none; }
.panel.is-open { display: block; }
```

## 注意事项

- `classList.toggle()` 返回布尔值：`true` 表示添加成功，`false` 表示移除成功。
- `classList` 只处理类名，不会影响其它属性。
- 同一交互建议统一使用 class 控制样式，避免直接改 style。