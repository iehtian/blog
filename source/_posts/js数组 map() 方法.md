---
title: js数组 map() 方法
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
date: 2025-12-12 21:48:00
updated: 2025-12-12 22:20:37
categories: [前端基础]
keywords: [map, 数组遍历, 转换, ES6]
description: 精炼讲解 js 数组 map()：语法、常见模式（基础转换、对象属性提取、数组解构、添加新属性、使用索引）、箭头函数要点、与其他数组方法的对比与技巧。示例简洁、结构清晰、逻辑顺畅。
top_img:
cover:
copyright_author_href:
copyright_url:
copyright_info:
---

**概览**
map() 是数组的转换方法：遍历每个元素并返回“新数组”。它不改变原数组，常用于格式化数据、提取字段、添加计算属性等。

**语法**
```js
const newArray = array.map((element, index, array) => {
  return transformedElement
})
```
- element: 当前元素
- index: 当前索引（可选）
- array: 原数组（可选）

**常见模式**
- 基础转换（数字/字符串）
```js
const nums = [1, 2, 3]
const doubled = nums.map(n => n * 2)           // [2, 4, 6]
const names = ['张三', '李四']
const hi = names.map(n => `你好, ${n}`)         // ['你好, 张三', '你好, 李四']
```

- 提取对象属性（单个/多个）
```js
const users = [{id:1, name:'张三', age:25}, {id:2, name:'李四', age:30}]
const onlyNames = users.map(u => u.name)       // ['张三','李四']
const light = users.map(u => ({ name: u.name, age: u.age }))
```

- 数组解构转对象（可跳过、可重命名）
```js
const rows = [
  ['张三', '2024-01-15', '男'],
  ['李四', '2024-02-20', '女']
]
const all = rows.map(([name, date, gender]) => ({ name, date, gender }))
const part = rows.map(([name, , gender]) => ({ name, gender }))
const renamed = rows.map(([n, , g]) => ({ studentName: n, studentGender: g }))
```

- 复杂解构与类型转换 / 剩余参数
```js
const csv = [['产品A','100','50','北京'], ['产品B','200','30','上海']]
const products = csv.map(([name, price, , location]) => ({ name, price: Number(price), location }))

const data = [['张三',25,'工程师','Python','Go'], ['李四',30,'设计师','Figma']]
const employees = data.map(([name, age, role, ...skills]) => ({ name, age, role, skills }))
```

- 添加/计算新属性（保留原属性）
```js
const items = [{ name:'A', price:100 }, { name:'B', price:200 }]
const withTax = items.map(p => ({ ...p, tax: p.price * 0.1, total: p.price * 1.1 }))
```

- 使用索引（编号/生成 id）
```js
const fruits = ['苹果','香蕉','橙子']
const numbered = fruits.map((f, i) => `${i + 1}. ${f}`)
const withId = fruits.map((f, i) => ({ id: i, name: f }))
```

**箭头函数要点**
```js
// 简写：无花括号，隐式返回
nums.map(n => n * 2)

// 完整：有花括号，需显式 return
nums.map(n => { const r = n * 2; return r })

// 返回对象时需用圆括号包裹字面量
nums.map(n => ({ value: n }))   // ✅
// nums.map(n => { value: n })  // ❌（被解析为代码块，不是返回值）
```

**重要特点**
- 不改变原数组（纯函数）
- 返回新数组（长度与原数组相同）
- 回调需要返回值（否则得到由 `undefined` 组成的数组）
- 可链式调用：`array.map(...).filter(...).reduce(...)`

**对比其他数组方法**
```js
const arr = [1, 2, 3, 4, 5]
arr.map(n => n * 2)                  // 转换：长度不变
arr.filter(n => n > 3)               // 筛选：长度变化
arr.forEach(n => console.log(n))     // 遍历：无返回值
arr.reduce((sum, n) => sum + n, 0)   // 累积：返回单个值
```

**常用技巧**
- 解构跳过元素：`[[1,2,3]].map(([a, , c]) => ({ a, c }))`
- 对象属性简写：`const obj = { name }` 等价 `{ name: name }`
- 展开运算符保留原属性：`arr.map(item => ({ ...item, extra: 1 }))`
- 链式处理：`arr.map(n => n*2).filter(n => n>5).map(n => ({ value: n }))`

**解构与简写的含义与作用**
以 `const mapped = rows.map(([name, , gender]) => ({ name, gender }))` 为例：
- `[name, , gender]`：数组解构，声明变量并从位置提取值（可理解为 `const name = row[0]`）。
- `{ name, gender }`：对象属性简写，等价 `{ name: name, gender: gender }`。

**等价写法对比**
```js
// 1) 解构 + 简写
rows.map(([name, , gender]) => ({ name, gender }))

// 2) 索引访问
rows.map(row => ({ name: row[0], gender: row[2] }))

// 3) 完整对象字面量
rows.map(([name, , gender]) => { return { name: name, gender: gender } })
```

**ES6 对象简写规则**
当属性名与变量名相同，可写成简写：
```js
const name = '张三'; const age = 30
const person1 = { name, age }            // ✅ 简写
const person2 = { name: name, age: age } // ✅ 等价
// 两者结果一致：{ name: '张三', age: 30 }
```