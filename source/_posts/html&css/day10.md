---
title: CSS 笔记 - 第10天
date: 2025.06.14
updated: 2025.06.14
tags: 
categories: 
keywords:
description: css
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
# CSS 基础知识笔记 - 第10天

## 平面转换-平移
通过 CSS transform 属性和 translate() 函数实现元素的平面平移。
- `transform: translate(x, y);`：元素在水平方向平移 x，垂直方向平移 y。
- `transform: translateX(x);`：元素在水平方向平移 x。
- `transform: translateY(y);`：元素在垂直方向平移 y。
  单位可以是 px, %, em 等。

**样例：**
```html
<div class="box translate-example"></div>
<style>
.box {
  width: 100px;
  height: 100px;
  background-color: lightblue;
  margin: 20px;
}
.translate-example {
  transform: translate(50px, 20px); /* 向右平移50px，向下平移20px */
}
</style>
```

## 平面转换-旋转
通过 CSS transform 属性和 rotate() 函数实现元素的平面旋转。
- `transform: rotate(angle);`：元素顺时针旋转指定的角度。角度单位可以是 deg (度), grad (梯度), rad (弧度), turn (圈)。
  默认旋转中心是元素的中心点 (50% 50%)。可以通过 `transform-origin` 属性修改旋转中心。
- 旋转会改变坐标轴指向

**样例：**
```html
<div class="box rotate-example"></div>
<style>
.rotate-example {
  transform: rotate(45deg); /* 顺时针旋转45度 */
  transform-origin: top left; /* 设置旋转中心为左上角 */
}
</style>
```

## 平面转换-组合
可以将多个转换函数组合在一起使用，用空格分隔。
- `transform: translate(50px, 100px) rotate(45deg);`：元素先平移，然后旋转。
  注意：转换的顺序会影响最终结果。

**样例：**
```html
<div class="box combined-example"></div>
<style>
.combined-example {
  transform: translateX(100px) rotate(30deg) scale(1.2); /* 先向右平移100px，然后旋转30度，最后放大1.2倍 */
}
</style>
```

## 平面转换-缩放
通过 CSS transform 属性和 scale() 函数实现元素的平面缩放。
- `transform: scale(x, y);`：元素在水平方向缩放 x 倍，垂直方向缩放 y 倍。如果只提供一个值，则水平和垂直方向等比例缩放。
- `transform: scaleX(x);`：元素在水平方向缩放 x 倍。
- `transform: scaleY(y);`：元素在垂直方向缩放 y 倍。
  缩放因子大于 1 表示放大，小于 1 表示缩小。默认缩放中心是元素的中心点 (50% 50%)。可以通过 `transform-origin` 属性修改缩放中心。

**样例：**
```html
<div class="box scale-example"></div>
<style>
.scale-example {
  transform: scale(1.5, 0.8); /* 水平方向放大1.5倍，垂直方向缩小到0.8倍 */
}
</style>
```

## 颜色渐变

### 线性渐变
通过 `linear-gradient()` 函数创建线性渐变背景。
- `background-image: linear-gradient(direction, color-stop1, color-stop2, ...);`
- `direction`：渐变的方向。可以是角度 (e.g., `90deg`)，也可以是关键词 (e.g., `to right`, `to bottom right`)。默认为 `to bottom`。
- `color-stop`：颜色断点，可以指定颜色和位置 (e.g., `red`, `blue 50%`)。

**样例：**
```html
<div class="gradient-box linear-gradient-example"></div>
<style>
.gradient-box {
  width: 200px;
  height: 100px;
  margin: 20px;
}
.linear-gradient-example {
  background-image: linear-gradient(to right, red, yellow, green); /* 从左到右，红到黄到绿的渐变 */
}
</style>
```
```html
<div class="gradient-box linear-gradient-example-angle"></div>
<style>
.linear-gradient-example-angle {
  background-image: linear-gradient(45deg, rgba(255,0,0,0), rgba(0,0,255,1)); /* 45度角，从透明红到不透明蓝 */
}
</style>
```

### 径向渐变
通过 `radial-gradient()` 函数创建径向渐变背景。
- `background-image: radial-gradient(shape size at position, start-color, ..., last-color);`
- `shape`：渐变的形状，可以是 `circle` (圆形) 或 `ellipse` (椭圆形，默认)。
- `size`：渐变的大小。可以是关键词 (e.g., `farthest-corner` (默认), `closest-side`, `closest-corner`, `farthest-side`) 或具体的长度/百分比。
- `position`：渐变的中心位置，类似于 `background-position`。默认为 `center`。
- `start-color, ..., last-color`：颜色断点。

**样例：**
```html
<div class="gradient-box radial-gradient-example"></div>
<style>
.radial-gradient-example {
  background-image: radial-gradient(circle, white, skyblue, dodgerblue); /* 圆形渐变，从中心白色到天蓝再到道奇蓝 */
}
</style>
```
```html
<div class="gradient-box radial-gradient-example-shape-size-pos"></div>
<style>
.radial-gradient-example-shape-size-pos {
  background-image: radial-gradient(ellipse farthest-side at 20% 80%, #ff0000 0%, #0000ff 50%, #00ff00 100%); /* 椭圆，最远边，中心在左下角，红到蓝到绿 */
}
</style>
```
