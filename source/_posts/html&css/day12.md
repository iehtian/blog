---
title: CSS 笔记 - 第12天
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
# CSS 基础知识笔记 - 第12天

## 移动端适配
移动端适配的目标是让网页在不同尺寸的移动设备上都能良好显示，提供优秀的用户体验。主要方法包括响应式设计、弹性布局和字体大小的动态调整等。

### 媒体查询 (Media Queries)
媒体查询是 CSS3 的一个模块，它允许我们根据设备的特性（如视口宽度、高度、方向、分辨率等）来应用不同的 CSS 样式。

**语法：**
```css
@media mediatype and|not|only (mediafeature and|or|not mediafeature) {
  /* CSS 规则 */
}
```
- `mediatype`：媒体类型，如 `screen` (屏幕), `print` (打印), `all` (所有设备，默认)。
- `and|not|only`：逻辑操作符。
- `mediafeature`：媒体特性，如 `width`, `max-width`, `min-width`, `orientation` (landscape | portrait), `resolution` 等。

**常用断点 (Breakpoints)：**
断点是媒体查询中用于区分不同设备尺寸的临界值。常见的断点划分参考（具体值可根据项目需求调整）：
- 小屏幕手机: `max-width: 575.98px`
- 手机: `min-width: 576px` and `max-width: 767.98px`
- 平板: `min-width: 768px` and `max-width: 991.98px`
- 桌面: `min-width: 992px` and `max-width: 1199.98px`
- 大桌面: `min-width: 1200px`

**样例：根据屏幕宽度改变布局和字体大小**
```html
<div class="container">
  <div class="box">Box 1</div>
  <div class="box">Box 2</div>
  <div class="box">Box 3</div>
</div>
<style>
body {
  font-size: 16px;
}
.container {
  display: flex;
  flex-direction: row; /* 默认横向排列 */
  background-color: #f0f0f0;
  padding: 10px;
}
.box {
  background-color: lightblue;
  border: 1px solid blue;
  padding: 20px;
  margin: 10px;
  flex: 1;
  text-align: center;
}

/* 当屏幕宽度小于或等于 768px 时 */
@media screen and (max-width: 768px) {
  body {
    font-size: 14px;
  }
  .container {
    flex-direction: column; /* 变为纵向排列 */
  }
  .box {
    background-color: lightcoral;
  }
}

/* 当屏幕宽度小于或等于 480px 时 */
@media screen and (max-width: 480px) {
  body {
    font-size: 12px;
  }
  .box {
    padding: 10px;
    background-color: lightgreen;
  }
}
</style>
```

### rem方式实现等比例缩放
`rem` (root em) 是一个相对单位，相对于根元素 (`<html>`) 的 `font-size`。
通过动态改变根元素的 `font-size`，所有使用 `rem` 作为单位的元素尺寸都会等比例缩放，从而实现整体页面的适配。

**实现步骤：**
1.  **设置视口 (Viewport)**：在 HTML 的 `<head>` 中添加 viewport meta 标签，确保页面宽度与设备宽度一致，并禁止用户缩放（可选）。
    ```html
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    ```
2.  **动态设置根字体大小**：通常使用 JavaScript 根据屏幕宽度动态计算并设置 `<html>` 元素的 `font-size`。一种常见的做法是将设计稿宽度（例如 750px）等分为若干份（例如 100份），那么 1rem 就等于 `750px / 100 = 7.5px` (在设计稿上)。然后根据当前设备宽度等比例计算实际的 `font-size`。

**JavaScript 动态设置根 `font-size` 样例：**
```html
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>REM Adaptaion</title>
  <script>
    (function(doc, win) {
      var docEl = doc.documentElement,
          resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
          recalc = function() {
            var clientWidth = docEl.clientWidth;
            if (!clientWidth) return;
            // 假设设计稿宽度为 750px，希望 1rem = 100px (在750px宽的设计稿上)
            // 则 根font-size = clientWidth / (750 / 100) = clientWidth / 7.5
            // 或者，如果希望设计稿宽度分为10份，1rem = 75px
            // 则 根font-size = clientWidth / 10
            var rootFontSize = 100 * (clientWidth / 750);
            docEl.style.fontSize = rootFontSize + 'px';
            // 例如，当设备宽度为 375px 时，根 font-size = 100 * (375 / 750) = 50px
            // 此时，设计稿上 100px 的元素，CSS中写 1rem，实际渲染为 50px
          };
      if (!doc.addEventListener) return;
      win.addEventListener(resizeEvt, recalc, false);
      doc.addEventListener('DOMContentLoaded', recalc, false);
    })(document, window);
  </script>
  <style>
    body { margin: 0; font-size: 0.32rem; /* 假设设计稿上body字体是32px */ }
    .header {
      width: 7.5rem; /* 对应设计稿 750px */
      height: 0.8rem; /* 对应设计稿 80px */
      background-color: orange;
      font-size: 0.4rem; /* 对应设计稿 40px */
      text-align: center;
      line-height: 0.8rem;
      color: white;
    }
    .content {
      width: 7rem; /* 对应设计稿 700px */
      padding: 0.25rem; /* 对应设计稿 25px */
      background-color: lightblue;
    }
  </style>
</head>
<body>
  <div class="header">Header (750px wide on design)</div>
  <div class="content">
    This content area is 700px wide on the design draft. Its padding is 25px.
    The base font size for the body is set to 0.32rem.
  </div>
</body>
```
**优点：**
-   可以实现页面元素的等比例缩放，保持设计稿的视觉比例。
-   计算相对简单，维护方便。

**缺点：**
-   需要 JavaScript 介入。
-   对于一些不需要等比例缩放的场景（如边框 `1px`），处理起来可能稍显麻烦（可以使用 `px` 单位，或者通过伪元素缩放）。

### vw/vh方式
`vw` (viewport width) 和 `vh` (viewport height) 是 CSS3 引入的视口单位。
- `1vw` = 视口宽度的 1%
- `1vh` = 视口高度的 1%
- `vmin` = `vw` 和 `vh` 中较小的值
- `vmax` = `vw` 和 `vh` 中较大的值

使用 `vw/vh` 可以直接根据视口大小来设置元素的尺寸，实现流式布局和等比例缩放，无需 JavaScript。

**样例：使用 vw/vh 实现适配**
```html
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>VW/VH Adaptaion</title>
  <style>
    body { margin: 0; font-size: 4vw; /* 字体大小随视口宽度变化 */ }
    .banner {
      width: 100vw; /* 宽度占满视口 */
      height: 20vh; /* 高度为视口高度的20% */
      background-color: seagreen;
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 5vw; /* banner内字体 */
    }
    .item-container {
      display: flex;
      width: 100vw;
    }
    .item {
      width: 50vw; /* 每个item占视口宽度的一半 */
      height: 30vh;
      padding: 2vw;
      box-sizing: border-box;
      border: 0.5vw solid #ccc; /* 边框也使用vw */
    }
    .item1 { background-color: lightpink; }
    .item2 { background-color: lightskyblue; }
  </style>
</head>
<body>
  <div class="banner">Welcome!</div>
  <div class="item-container">
    <div class="item item1">Item 1 (50vw wide)</div>
    <div class="item item2">Item 2 (50vw wide)</div>
  </div>
</body>
```
**优点：**
-   纯 CSS 实现，无需 JavaScript。
-   直接与视口尺寸关联，语义清晰。

**缺点：**
-   对于非常小的尺寸，所有元素都会变得非常小，可能影响可读性。
-   `px` 转换 `vw` 计算可能比较繁琐，但可以使用 SASS/LESS 等预处理器简化。
-   某些情况下（如固定宽高的弹窗），可能不如 `rem` 灵活。

**结合使用：**
在实际项目中，通常会将媒体查询、`rem`、`vw/vh` 以及弹性布局（Flexbox/Grid）等多种技术结合起来，以达到最佳的适配效果。
例如，可以使用媒体查询设置不同的根字体大小基准，然后页面主体内容使用 `rem`，而某些需要严格占满视口或按比例显示的模块使用 `vw/vh`。