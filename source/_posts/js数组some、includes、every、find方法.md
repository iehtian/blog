---
title: js数组some、includes、every、find方法
tags: [js, 数组, ES6]
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
date: 2026-02-02 22:40:07
updated: 2026-02-02 22:51:58
categories: [前端基础]
keywords: [some, every, find, includes, 数组查找, 判断]
description: 精炼讲解 js 数组 some/every/find/includes：语法、常见用法、对比与易踩坑点，附示例便于快速上手。
top_img:
cover:
copyright_author_href:
copyright_url:
copyright_info:
---

**概览**
some、every、find、includes 都是数组的“判断/查找”方法：
- `some()`：只要有一个满足条件就返回 `true`
- `every()`：所有元素都满足条件才返回 `true`
- `find()`：返回第一个满足条件的元素（找不到则 `undefined`）
- `includes()`：判断是否包含指定值（返回 `true/false`）

**语法**
```js
array.some((element, index, array) => boolean)
array.every((element, index, array) => boolean)
array.find((element, index, array) => boolean)
array.includes(searchElement, fromIndex?)
```

**some() 常见模式**
- 判断是否存在符合条件的元素
```js
const nums = [1, 3, 5, 8]
const hasEven = nums.some(n => n % 2 === 0)  // true
```

- 对象数组中是否包含满足条件的项
```js
const users = [{id:1, role:'user'}, {id:2, role:'admin'}]
const hasAdmin = users.some(u => u.role === 'admin') // true
```

**every() 常见模式**
- 判断是否全部满足条件
```js
const scores = [80, 95, 88]
const allPass = scores.every(s => s >= 60)   // true
```

- 验证表单输入是否都非空
```js
const fields = ['张三', '13800000000', '上海']
const allFilled = fields.every(Boolean)      // true
```

**find() 常见模式**
- 查找第一个匹配的元素（对象）
```js
const list = [{id:1, name:'A'}, {id:2, name:'B'}]
const item = list.find(x => x.id === 2)      // {id:2, name:'B'}
```

- 使用索引（场景：找第一个超过阈值的元素）
```js
const arr = [5, 9, 12]
const firstBig = arr.find((n, i) => n > 10 && i >= 1) // 12
```

**includes() 常见模式**
- 判断是否包含某值
```js
const tags = ['js', '数组', 'ES6']
const hasJs = tags.includes('js')             // true
```

- 指定起始位置
```js
const arr = [1, 2, 3, 2]
arr.includes(2)            // true
arr.includes(2, 2)         // true（从索引2开始，包含2）
arr.includes(2, 3)         // false（从索引3开始）
```

**返回值与差异对比**
```js
const arr = [1, 2, 3, 4]

arr.some(n => n > 3)        // true
arr.every(n => n > 3)       // false
arr.find(n => n > 3)        // 4
arr.includes(3)             // true
```

**容易踩坑（重点）**
1) `some/every/find` 回调必须 `return` 布尔值
```js
const nums = [1, 2, 3]

// ❌ 忘记 return：回调返回 undefined，结果永远为 false / undefined
nums.some(n => { n > 2 })   // false
nums.find(n => { n > 2 })   // undefined

// ✅ 正确：显式或隐式 return
nums.some(n => n > 2)       // true
nums.find(n => n > 2)       // 3
```

2) `find()` 只返回“第一个匹配项”，不是所有匹配项
```js
const arr = [1, 2, 3, 4]
arr.find(n => n % 2 === 0)  // 2
// 若需要所有匹配项，用 filter()
```

3) `includes()` 与 `indexOf()` 的差异
- `includes()` 能正确处理 `NaN`，`indexOf()` 不能
```js
const arr = [1, NaN, 3]
arr.includes(NaN)   // true
arr.indexOf(NaN)    // -1
```

4) `includes()` 判断对象是“引用相等”，不是内容相等
```js
const list = [{id:1}]
list.includes({id:1})        // false

const same = list[0]
list.includes(same)          // true
```
如果需要深入判断内容（如对象字段是否匹配），使用 `some()` + 自定义条件：
```js
const list2 = [{id:1, name:'A'}, {id:2, name:'B'}]
const hasId2 = list2.some(item => item.id === 2)  // true
```

5) `every()` 在空数组上返回 `true`
```js
[].every(x => x > 0)         // true（空集合“全部满足”）
```
如果不希望这样，需先判断长度：`arr.length > 0 && arr.every(...)`

6) `some()` 在空数组上返回 `false`
```js
[].some(x => x > 0)          // false
```

7) `find()` 找不到返回 `undefined`，别直接解构或调用属性
```js
const users = [{id:1, name:'A'}]
const u = users.find(x => x.id === 2) // undefined
// ❌ u.name 会报错
// ✅ 使用可选链或默认值
const name = u?.name ?? '未知'
```

**与 map/filter 的区别（快速对照）**
```js
const arr = [1, 2, 3]
arr.map(n => n * 2)           // [2, 4, 6]   转换
arr.filter(n => n > 1)        // [2, 3]      筛选
arr.some(n => n > 1)          // true        存在性判断
arr.every(n => n > 1)         // false       全部判断
arr.find(n => n > 1)          // 2           查找单个
arr.includes(2)               // true        是否包含
```

**实用小技巧**
- 用 `every(Boolean)` 快速判断全为真值
- 用 `some()` 替代手写循环的“是否存在”逻辑
- `find()` + `??` 设置默认值：
```js
const user = list.find(u => u.id === id) ?? { id, name: '未命名' }
```
