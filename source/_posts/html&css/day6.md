---
title: CSS 笔记 - 第6天
date: 2025.06.11
updated: 2025.06.11
tags: 
categories: 
keywords:
description: Flex布局（Flexible Box）
top_img: 
comments: true
cover:
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

# CSS 笔记 - 第6天

## flex主轴与侧轴对齐方式

在Flex布局中，容器内的项目沿着两个轴进行排列：主轴（Main Axis）和侧轴（Cross Axis）。默认情况下，主轴是水平方向，侧轴是垂直方向。

### 主轴对齐方式 (justify-content)

用于设置项目在主轴上的对齐方式，属性值包括：

- `flex-start`（默认值）：项目靠主轴起点对齐
- `flex-end`：项目靠主轴终点对齐
- `center`：项目居中对齐
- `space-between`：两端对齐，项目之间的间隔相等
- `space-around`：每个项目两侧的间隔相等，项目之间的间隔是项目与边缘间隔的两倍
- `space-evenly`：项目间距离完全平均分配

```css
.container {
  display: flex;
  justify-content: center; /* 在主轴上居中对齐 */
}
```

### 侧轴对齐方式 (align-items)

用于设置项目在侧轴上的对齐方式，属性值包括：

- `stretch`（默认值）：如果项目未设置高度或为auto，将占满整个容器的高度
- `flex-start`：项目靠侧轴起点对齐
- `flex-end`：项目靠侧轴终点对齐
- `center`：项目居中对齐
- `baseline`：项目第一行文字的基线对齐

```css
.container {
  display: flex;
  align-items: center; /* 在侧轴上居中对齐 */
}
```

### 单个项目的侧轴对齐方式 (align-self)

允许单个项目有不同于其他项目的对齐方式，可覆盖`align-items`属性：

```css
.item {
  align-self: flex-end; /* 该项目在侧轴上靠终点对齐 */
}
```

## 修改主轴方向

通过`flex-direction`属性可以更改主轴的方向，从而改变项目的排列方向。

### flex-direction属性值

- `row`（默认值）：主轴为水平方向，起点在左端
- `row-reverse`：主轴为水平方向，起点在右端
- `column`：主轴为垂直方向，起点在上端
- `column-reverse`：主轴为垂直方向，起点在下端

```css
.container {
  display: flex;
  flex-direction: column; /* 主轴变为垂直方向 */
}
```

修改主轴方向后，主轴和侧轴的概念会相应变化，但各种对齐属性的作用依然是针对主轴和侧轴，而不是水平和垂直方向。

## 弹性伸缩比

Flex布局允许子项目按照一定的比例分配容器的剩余空间，这通过`flex`属性来实现。

### flex属性

`flex`属性是`flex-grow`、`flex-shrink`和`flex-basis`的简写，默认值为`0 1 auto`。

#### flex-grow (弹性增长比例)

定义项目的放大比例，默认为`0`，即如果存在剩余空间，也不放大。

```css
.item {
  flex-grow: 1; /* 平均分配剩余空间 */
}
```

如果所有项目的`flex-grow`都为1，则它们将等分剩余空间。如果有一个项目的`flex-grow`为2，其他项目都为1，则前者占据的剩余空间将是其他项的两倍。

#### flex-shrink (弹性收缩比例)

定义项目的缩小比例，默认为`1`，即如果空间不足，该项目会缩小。

```css
.item {
  flex-shrink: 0; /* 空间不足时不缩小 */
}
```

#### flex-basis (基础尺寸)

定义在分配多余空间之前，项目占据的主轴空间。默认值为`auto`，即项目的本来大小。

```css
.item {
  flex-basis: 200px; /* 项目基础尺寸为200px */
}
```

#### 常用的flex简写值

- `flex: 1;` 等同于 `flex: 1 1 0%;` 表示项目可以弹性增长和收缩，基础尺寸为0
- `flex: auto;` 等同于 `flex: 1 1 auto;` 表示项目可以弹性增长和收缩，基础尺寸为自身宽度/高度
- `flex: none;` 等同于 `flex: 0 0 auto;` 表示项目既不能增长也不能收缩，保持原有尺寸

## 弹性换行与行对齐方式

默认情况下，所有Flex项目都排在一条线（主轴）上。`flex-wrap`属性可以使项目在必要时换行。

### flex-wrap属性

- `nowrap`（默认）：不换行，可能导致项目宽度被压缩
- `wrap`：换行，第一行在上方
- `wrap-reverse`：换行，第一行在下方

```css
.container {
  display: flex;
  flex-wrap: wrap; /* 允许项目换行 */
}
```

### 多行对齐方式 (align-content)

当存在多行时，通过`align-content`属性可以定义多行在侧轴上如何对齐，类似于主轴上的`justify-content`：

- `stretch`（默认）：平均分布，拉伸填满容器
- `flex-start`：与侧轴的起点对齐
- `flex-end`：与侧轴的终点对齐
- `center`：与侧轴的中点对齐
- `space-between`：与侧轴两端对齐，中间平均分布
- `space-around`：每行两侧的间隔相等，行之间的间隔是行与边缘间隔的两倍

```css
.container {
  display: flex;
  flex-wrap: wrap;
  align-content: space-between; /* 多行在侧轴上两端对齐 */
}
```

### flex-flow简写属性

`flex-flow`是`flex-direction`和`flex-wrap`的简写形式：

```css
.container {
  display: flex;
  flex-flow: row wrap; /* 水平方向并且允许换行 */
}
```

## 实际应用案例

### 居中对齐

```css
.center-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 300px;
}
```

### 导航栏布局

```css
.navbar {
  display: flex;
  justify-content: space-between;
}

.nav-left {
  display: flex;
}

.nav-right {
  display: flex;
}
```

### 响应式布局

```css
.responsive-container {
  display: flex;
  flex-wrap: wrap;
}

.responsive-item {
  flex: 1 1 300px; /* 基础宽度300px，但可以伸缩 */
  margin: 10px;
}
```

## 总结

Flex布局是一种强大的CSS布局方式，通过主轴、侧轴的概念和一系列属性，可以轻松实现各种布局效果：

1. 主轴对齐：`justify-content`
2. 侧轴对齐：`align-items`和`align-self`
3. 主轴方向：`flex-direction`
4. 弹性伸缩：`flex`、`flex-grow`、`flex-shrink`和`flex-basis`
5. 换行控制：`flex-wrap`
6. 多行对齐：`align-content`

掌握Flex布局是现代前端开发的基本要求，能够帮助我们更轻松地实现复杂的页面布局。