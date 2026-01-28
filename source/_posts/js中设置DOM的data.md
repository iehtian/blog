---
title: js 中设置 DOM 的 data
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
date: 2026-01-28 20:39:28
updated: 2026-01-28 20:49:03
categories: [前端基础]
keywords: [dataset, data-*, DOM]
description: 介绍 dataset 的读写方式、命名规则、查询与注意事项，附常见示例。
top_img:
cover:
copyright_author_href:
copyright_url:
copyright_info:
---

**概览**
`dataset` 是 DOM 元素上的数据集合，用于读写自定义属性 `data-*`。设置会自动反映到 HTML 属性上，读取时则从属性映射回来。

## 基本读写

```js
const el = document.querySelector('#app')

// 写入
el.dataset.timeNow = String(Date.now())

// 读取
const time = el.dataset.timeNow
```

## 命名规则与映射

- `dataset` 使用驼峰命名，HTML 使用连字符：

```js
el.dataset.timeNow = '10:00'
// HTML 中显示为：data-time-now="10:00"
```

- 多词字段都遵循该映射：

```js
el.dataset.userId = '42'
// HTML：data-user-id="42"
```

## 查询带 data 属性的元素

```js
const nodes = document.querySelectorAll('[data-time-now]')
```

## 删除 data 属性

```js
delete el.dataset.timeNow
// HTML 中 data-time-now 将被移除
```

## 注意事项

- `dataset` 的值 **全部是字符串**。
	- 复杂数据需自行序列化/反序列化：

```js
const payload = { id: 1, name: 'Alice' }
el.dataset.user = JSON.stringify(payload)

const user = JSON.parse(el.dataset.user)
```

- `dataset` 只会映射 `data-*` 属性，不会影响其它属性。

## 小示例：基于 data 属性绑定

```html
<button id="btn" data-time-now="10:00">保存</button>
```

```js
const btn = document.querySelector('#btn')
btn.addEventListener('click', () => {
	const time = btn.dataset.timeNow
	console.log('当前时间：', time)
})
```