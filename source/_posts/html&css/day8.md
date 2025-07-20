---
title: CSS 笔记 - 第8天
date: 2025.06.12
updated: 2025.06.12
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
# CSS 基础知识笔记 - 第8天

## 定位

### 相对定位
特点：
-   元素相对于其**正常位置**进行定位。
-   设置相对定位的元素仍然占据其在文档流中的原始空间，不会影响其他元素的布局。
-   通过 `top`, `right`, `bottom`, `left` 属性进行偏移。
-   即使元素偏移，其原始占位空间依然保留。
-   常用于为绝对定位的子元素提供参照物。

```css
.relative-box {
  position: relative;
  left: 20px;
  top: 10px;
  background-color: lightblue;
  padding: 10px;
}
```
#### 利用相对定位实现盒子在父盒子居中
严格来说，单独使用相对定位（`position: relative;`）本身并不能直接实现一个元素在父元素中精确的水平和垂直居中，因为相对定位的主要作用是相对于元素自身在正常文档流中的位置进行偏移，它不改变元素在文档流中的占位。

然而，我们可以结合相对定位和其他属性（如 `left`, `top` 和 `transform`）来实现居中效果，或者将相对定位用于父元素，以便子元素可以使用绝对定位进行更灵活的居中。

**方法一：结合 `left: 50%; top: 50%;` 和 `transform: translate(-50%, -50%);` (推荐给子绝父相中的子元素)**

这种方法通常用于**绝对定位**的子元素（`position: absolute;`）在其已定位的父元素（通常是 `position: relative;`）中居中。虽然标题是“利用相对定位”，但这种居中技巧的核心是绝对定位的子元素，父元素的相对定位是为了给子元素提供定位上下文。

如果硬要说“利用相对定位实现盒子在父盒子居中”，可以理解为父盒子设置了 `position: relative;`，然后子盒子通过某种方式在其中居中。

```html
<div class="parent-container">
  <div class="child-box-center-abs">
    我是一个在父容器中居中的盒子 (使用绝对定位)
  </div>
</div>
```

```css
.parent-container {
  position: relative; /* 父元素相对定位，为子元素提供定位上下文 */
  width: 300px;
  height: 200px;
  background-color: lightgray;
  border: 1px solid black;
}

.child-box-center-abs {
  position: absolute; /* 子元素绝对定位 */
  width: 100px;
  height: 50px;
  background-color: lightblue;
  border: 1px solid blue;

  /* 关键步骤 */
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
```

**解释：**
1.  父元素 `.parent-container` 设置 `position: relative;`。
2.  子元素 `.child-box-center-abs` 设置 `position: absolute;`。
3.  `left: 50%;` 和 `top: 50%;` 将子元素的左上角移动到父元素的中心点。
4.  `transform: translate(-50%, -50%);` 再将子元素向左和向上平移自身宽度和高度的50%，从而使其自身的中心点与父元素的中心点对齐，实现真正的居中。

**如果坚持只用相对定位来尝试“居中”一个元素（不常用且效果有限）：**

如果你想让一个元素相对于它自己在文档流中的原始位置进行偏移，使其看起来“居中”于某个概念上的区域，可以通过计算偏移量来实现。但这通常不是精确的父容器居中。

例如，如果一个相对定位的元素，你知道它距离父容器左边多少，父容器宽度多少，自身宽度多少，你可以计算 `left` 的百分比值。

```html
<div class="outer-area">
  <div class="relatively-centered-box">
    我尝试通过相对定位偏移来居中
  </div>
</div>
```

```css
.outer-area {
  width: 400px;
  height: 300px;
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  /* 为了演示，我们假设 outer-area 内部有一个隐形的居中线 */
}

.relatively-centered-box {
  position: relative;
  width: 150px;
  height: 80px;
  background-color: lightcoral;
  border: 1px solid red;

  /* 水平居中尝试：假设父容器宽度400px，子容器宽度150px
     需要向右移动 (400 - 150) / 2 = 125px
     如果父容器没有明确的定位上下文，这里的百分比是相对于自身计算的，
     所以直接用 left: 50% 然后 transform 的方式更适合绝对定位。
     对于相对定位，通常需要知道精确的偏移值或父级尺寸。
  */
  left: 125px; /* 精确像素值 */
  /* 或者 left: calc(50% - 75px); 如果想基于父级50%再减去自身宽度一半，但这需要父级是定位上下文且子元素是绝对定位才更直观 */

  /* 垂直居中尝试：类似地计算 top 值 */
  top: 110px; /* (300 - 80) / 2 = 110px */
}
```
这种方式的缺点是：
-   需要提前知道父元素和子元素的尺寸才能精确计算偏移量。
-   如果父元素或子元素的尺寸发生变化，偏移量需要重新计算。
-   它并没有真正地将元素从文档流中取出并相对于父元素定位，只是在原始位置基础上进行视觉上的移动。

**总结：**
虽然标题提及“利用相对定位实现盒子在父盒子居中”，但最常见和最有效的居中方法（特别是同时水平和垂直居中）是**父元素相对定位，子元素绝对定位，并结合 `left: 50%; top: 50%; transform: translate(-50%, -50%);`**。单独使用相对定位进行精确的父容器内居中是比较困难且不灵活的。

如果只是想让一个元素在页面某个区域水平居中，并且该元素是块级元素，更简单的方法是设置 `margin-left: auto; margin-right: auto;` (前提是元素有固定宽度)。

### 绝对定位
特点：
-   元素相对于其**最近的已定位祖先元素**（即 `position` 属性为 `relative`, `absolute`, `fixed` 或 `sticky` 的祖先元素）进行定位。
-   如果不存在已定位的祖先元素，则相对于初始包含块（通常是 `<html>` 元素）进行定位。
-   元素会从正常的文档流中完全脱离，不再占据空间，后续元素会填充其原来的位置。
-   通过 `top`, `right`, `bottom`, `left` 属性进行定位。
-   可以使用 `z-index` 属性控制堆叠顺序。

```css
.parent {
  position: relative; /* 作为绝对定位子元素的参照物 */
  width: 200px;
  height: 200px;
  background-color: lightgray;
}

.absolute-box {
  position: absolute;
  top: 30px;
  right: 10px;
  width: 50px;
  height: 50px;
  background-color: lightcoral;
}
```

### 固定定位
特点：
-   元素相对于**浏览器视口**（viewport）进行定位。
-   元素会从正常的文档流中完全脱离，不占据空间。
-   即使页面滚动，固定定位的元素也会保持在视口的同一位置。
-   通过 `top`, `right`, `bottom`, `left` 属性进行定位。
-   常用于创建固定的导航栏、回到顶部按钮等。
-   可以使用 `z-index` 属性控制堆叠顺序。

```css
.fixed-box {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: lightgreen;
  padding: 10px;
  border-radius: 5px;
}
```

### 组合使用
相对定位、绝对定位和固定定位可以组合使用，以实现更复杂的布局效果。常见的组合是：
-   **父相子绝**：父元素设置相对定位（`position: relative;`），子元素设置绝对定位（`position: absolute;`）。这样，子元素的定位就会以父元素为参照。这是非常常用的一种布局技巧。

```html
<div style="position: relative; width: 300px; height: 200px; border: 1px solid black;">
  父元素 (relative)
  <div style="position: absolute; top: 10px; left: 10px; background-color: yellow;">
    子元素 (absolute)
  </div>
</div>
```


### 堆叠顺序 z-index
当多个定位元素在视觉上发生重叠时，可以使用 `z-index` 属性来控制它们的堆叠顺序。
特点：
-   `z-index` 属性只对**定位元素**（即 `position` 属性值为 `relative`, `absolute`, `fixed`, 或 `sticky` 的元素）有效。
-   `z-index` 的值可以是一个整数，正数、负数或零。
-   值越大的元素会显示在值越小的元素的上方。
-   如果没有设置 `z-index` 或者 `z-index` 值相同，则遵循HTML中后来者居上的原则（即在HTML代码中后出现的元素会覆盖先出现的元素）。
-   父元素的 `z-index` 会影响其子元素的堆叠上下文。

```css
.box1 {
  position: absolute;
  top: 10px;
  left: 10px;
  width: 100px;
  height: 100px;
  background-color: red;
  z-index: 1; /* 在 box2 下方 */
}

.box2 {
  position: absolute;
  top: 50px;
  left: 50px;
  width: 100px;
  height: 100px;
  background-color: blue;
  z-index: 2; /* 在 box1 上方 */
}
```

## css精灵
CSS 精灵（CSS Sprites）是一种网页图片应用处理方式。它允许你将一个页面涉及到的所有零星图片都包含到一张大图中去，然后利用CSS的 `background-image`，`background-repeat`，和 `background-position` 属性进行背景定位，这样就可以将大图的某个部分准确地显示在需要的位置。

**优点：**
1.  **减少HTTP请求数**：将多个小图片合并成一张大图片，浏览器只需要请求一次，从而减少了服务器的压力和网络延迟，加快了页面加载速度。
2.  **提前加载资源**：由于所有图片都在一张图中，可以一次性加载所有需要的图像资源。
3.  **维护方便**：图片资源集中管理，修改和替换相对容易。

**缺点：**
1.  **图片合并工作量**：需要手动或使用工具将小图片合并成大图，并精确计算每个小图片在大图中的位置。
2.  **维护成本**：当需要修改某个小图标时，可能需要重新生成整个精灵图，并更新CSS中对应的 `background-position`。
3.  **图片大小**：如果精灵图过大，首次加载时间可能会较长。
4.  **可访问性**：对于背景图片，屏幕阅读器可能无法很好地解读，需要额外的ARIA属性来增强可访问性。
5.  **缩放问题**：在高DPI屏幕上，精灵图可能会出现模糊，需要准备多倍图或使用SVG等矢量图形。

**使用方法：**
1.  **创建精灵图**：将多个小图标或图片排列在一张大图上。
2.  **设置CSS**：
    *   为需要显示图标的元素设置 `background-image` 为该精灵图。
    *   设置 `background-repeat: no-repeat;` 防止图片重复。
    *   通过 `background-position` 属性，将精灵图中对应的小图标定位到元素的背景中。通常需要设置元素的 `width` 和 `height` 与小图标的尺寸一致。

**示例：**
假设我们有一张名为 `sprite.png` 的精灵图，其中包含两个图标，第一个图标在 (0, 0) 位置，尺寸为 20x20px；第二个图标在 (-20px, 0) 位置（即向左偏移20px），尺寸也为 20x20px。

```html
<div class="icon icon-home"></div>
<div class="icon icon-settings"></div>
```

```css
.icon {
  width: 20px;
  height: 20px;
  background-image: url('sprite.png');
  background-repeat: no-repeat;
}

.icon-home {
  background-position: 0 0;
}

.icon-settings {
  background-position: -20px 0; /* 或者 0 -20px 如果图标是垂直排列的 */
}
```
现代前端开发中，虽然CSS精灵仍然是一种优化手段，但随着HTTP/2的普及（多路复用减少了请求数量的负面影响）以及SVG图标和字体图标的流行，CSS精灵的使用场景相对减少了一些，但其核心思想（减少请求）依然有价值。

## 垂直对齐方式vertical-align
`vertical-align` 属性用于设置行内元素（inline）、行内块元素（inline-block）或表格单元格（table-cell）的垂直对齐方式。

**重要特性：**
-   它**不影响**块级元素的垂直对齐。
-   它影响元素自身在其所在的行框盒子（line box）中的对齐，或者表格单元格内容的对齐。
-   对于行内元素和行内块元素，它指定了元素如何相对于其父元素的基线（baseline）或其他行内元素对齐。

**常用属性值：**

*   **相对于父元素基线：**
    *   `baseline` (默认值): 元素的基线与父元素的基线对齐。
    *   `sub`: 元素的基线降低到父元素的下标基线位置。
    *   `super`: 元素的基线升高到父元素的上标基线位置。
    *   `text-top`: 元素的顶部与父元素内容的顶部对齐（忽略父元素的 `padding` 和 `border`）。
    *   `text-bottom`: 元素的底部与父元素内容的底部对齐（忽略父元素的 `padding` 和 `border`）。
    *   `<length>`: 元素的基线相对于父元素的基线升高或降低指定的长度值。正值表示升高，负值表示降低。
    *   `<percentage>`: 元素的基线相对于父元素的 `line-height` 升高或降低指定的百分比。正值表示升高，负值表示降低。

*   **相对于行框盒子：**
    *   `middle`: 元素的中部与父元素基线加上父元素 x-height 的一半的位置对齐。对于图片等替换元素，通常意味着图片的垂直中点与行框的基线加上x-height的一半对齐。
    *   `top`: 元素的顶部与它所在的行框盒子的顶部对齐。
    *   `bottom`: 元素的底部与它所在的行框盒子的底部对齐。

**示例：**

```html
<div style="border: 1px solid black; line-height: 50px;">
  <span style="font-size: 20px;">Text</span>
  <img src="image.png" alt="example" style="width: 30px; height: 30px; vertical-align: middle;">
  <span style="background-color: lightblue; vertical-align: top;">Top Aligned</span>
  <span style="background-color: lightgreen; vertical-align: baseline;">Baseline (default)</span>
  <span style="background-color: lightcoral; vertical-align: bottom;">Bottom Aligned</span>
</div>
```

```css
/* 在上面的HTML示例中，不同的span和img元素会根据其vertical-align值在其行框内垂直对齐。*/
.parent-div {
  line-height: 100px; /* 定义一个行高，以便观察对齐效果 */
  border: 1px solid #ccc;
}

.parent-div img {
  width: 50px;
  height: 50px;
}

.align-baseline {
  vertical-align: baseline;
}

.align-middle {
  vertical-align: middle;
}

.align-top {
  vertical-align: top;
}

.align-bottom {
  vertical-align: bottom;
}

.align-text-top {
  vertical-align: text-top;
}

.align-text-bottom {
  vertical-align: text-bottom;
}
```

**注意：** `vertical-align` 的行为有时可能比较复杂，特别是在处理不同字体大小、行高以及混合内容（如文本和图片）时。理解基线（baseline）和行框盒子（line box）的概念对于掌握 `vertical-align`至关重要。

## 过渡 transition
CSS 过渡（`transition`）允许你在元素从一种样式逐渐改变为另一种样式时，控制动画的速度和过程。这使得状态变化更加平滑和吸引人，而不是瞬间完成。

**过渡的组成部分：**

1.  **`transition-property`**: 指定应用过渡效果的 CSS 属性的名称。
    *   可以指定一个或多个属性名称，用逗号分隔 (e.g., `width, color`)。
    *   `all`: 所有可动画的属性都将应用过渡效果（默认值）。
    *   并非所有CSS属性都可以过渡，通常只有那些具有明确中间值的属性（如长度、颜色、数字等）才可以。

2.  **`transition-duration`**: 指定过渡效果完成所需的时间，单位为秒 (s) 或毫秒 (ms)。
    *   例如：`0.5s` (0.5秒), `300ms` (300毫秒)。
    *   默认值为 `0s`，表示没有过渡效果，变化会立即发生。

3.  **`transition-timing-function`**: 指定过渡效果的时间曲线，即动画在不同时间点的速度。
    *   `ease` (默认值): 慢速开始，然后加速，然后慢速结束。
    *   `linear`: 匀速。
    *   `ease-in`: 慢速开始。
    *   `ease-out`: 慢速结束。
    *   `ease-in-out`: 慢速开始和结束（比 `ease` 更明显）。
    *   `cubic-bezier(n,n,n,n)`: 自定义贝塞尔曲线，可以创建更复杂的缓动效果。

4.  **`transition-delay`**: 指定过渡效果开始前的延迟时间，单位为秒 (s) 或毫秒 (ms)。
    *   例如：`1s` (延迟1秒后开始过渡)。
    *   默认值为 `0s`，表示立即开始过渡。

**简写属性 `transition`：**
可以将以上四个属性合并为一个简写属性 `transition`。值的顺序通常是：
`transition: [property] [duration] [timing-function] [delay];`

如果省略某个值，浏览器会使用其默认值。

**示例：**

```html
<div class="box-transition">Hover over me</div>
```

```css
.box-transition {
  width: 100px;
  height: 100px;
  background-color: lightblue;
  /* 定义过渡效果：作用于 width 和 background-color 属性，
     持续时间0.5秒，使用 ease-in-out 缓动函数，无延迟 */
  transition-property: width, background-color; /* 或者直接用 all */
  transition-duration: 0.5s;
  transition-timing-function: ease-in-out;
  /* transition-delay: 0s; */ /* 默认值，可以不写 */

  /* 或者使用简写形式 */
  /* transition: width 0.5s ease-in-out, background-color 0.5s ease-in-out; */
  /* 如果所有属性的过渡参数都一样，可以更简洁 */
  /* transition: all 0.5s ease-in-out; */
}

.box-transition:hover {
  width: 200px;
  background-color: lightcoral;
}
```

当鼠标悬停在 `div.box-transition` 元素上时，它的宽度会从 `100px` 平滑过渡到 `200px`，背景颜色会从 `lightblue` 平滑过渡到 `lightcoral`，整个过程持续0.5秒，并采用 `ease-in-out` 的速度曲线。

**多属性过渡：**
可以为不同的属性设置不同的过渡参数，用逗号分隔：
```css
.element {
  transition: width 0.5s ease-in, height 1s linear 0.2s, opacity 0.3s;
}
```
在这个例子中：
-   `width` 属性的过渡持续0.5秒，使用 `ease-in` 函数，无延迟。
-   `height` 属性的过渡持续1秒，使用 `linear` 函数，延迟0.2秒开始。
-   `opacity` 属性的过渡持续0.3秒，使用默认的 `ease` 函数和无延迟。

过渡是创建交互式和动态用户界面的重要工具，能够显著提升用户体验。

## 透明度 opacity
`opacity` 属性用于设置元素的透明度级别。它是一个介于 `0`（完全透明）和 `1`（完全不透明）之间的数值。

**基本用法：**

```css
.transparent-box {
  opacity: 0.5; /* 50% 透明度 */
}
```

**注意事项：**
-   `opacity` 属性会影响元素及其所有子元素的透明度。
-   透明度为 `0` 时，元素不可见；透明度为 `1` 时，元素完全可见。
-   介于 `0` 和 `1` 之间的值会导致元素部分透明，背景会透过元素显示出来。

**示例：**

```html
<div class="box" style="width: 100px; height: 100px; background-color: red; opacity: 0.7;">
  我是一个透明的盒子
</div>
```

```css
.box {
  width: 100px;
  height: 100px;
  background-color: red;
  opacity: 0.7; /* 70% 不透明度，30% 透明度 */
}
```

在这个示例中，红色盒子的透明度被设置为 `0.7`，这意味着它是70%不透明的，30%透明的。你可以看到盒子后面的内容，但盒子本身是可见的。

**与背景色结合使用：**
当 `opacity` 与背景色结合使用时，可以创建各种视觉效果。例如，设置一个半透明的黑色背景可以使文本更加突出。

```css
.semi-transparent-bg {
  background-color: rgba(0, 0, 0, 0.5); /* 黑色，50% 透明度 */
  color: white;
  padding: 10px;
}
```

```html
<div class="semi-transparent-bg">
  这段文字在半透明黑色背景上
</div>
```

在这个例子中，使用了 `rgba` 颜色值，其中 `a` 通道控制透明度。这个 div 的背景是半透明的黑色，文本是白色的。

**过渡透明度：**
你还可以使用 CSS 过渡平滑地改变元素的透明度。

```css
.fade-box {
  width: 100px;
  height: 100px;
  background-color: blue;
  opacity: 1;
  transition: opacity 0.5s ease;
}

.fade-box:hover {
  opacity: 0.3; /* 鼠标悬停时变为 30% 不透明 */
}
```

当鼠标悬停在 `.fade-box` 元素上时，它的透明度会在0.5秒内从 `1` 过渡到 `0.3`，创建一个淡出效果。

**注意：** 在使用 `opacity` 时要注意可访问性，确保文本和背景之间有足够的对比度，以便于阅读和理解内容。

## 光标类型 cursor
`cursor` 属性用于指定当用户将鼠标指针悬停在元素上方时，光标的显示方式。通过改变光标类型，可以为用户提供有关当前可用操作的视觉提示。

**常见的光标类型：**
-   `default`: 默认光标，通常是一个箭头。
-   `pointer`: 指示链接或可点击元素的手形光标。
-   `text`: 文本光标，表示可以编辑文本的区域。
-   `move`: 表示元素可被移动的光标。
-   `not-allowed`: 表示不允许的操作的光标，通常伴随一个斜杠。
-   `crosshair`: 十字准星光标。
-   `wait`: 表示正在等待的光标，通常是一个沙漏或旋转的圆圈。

**基本用法：**

```css
.clickable {
  cursor: pointer; /* 鼠标悬停时显示手形光标 */
}

.text-editable {
  cursor: text; /* 鼠标悬停时显示文本光标 */
}

.movable {
  cursor: move; /* 鼠标悬停时显示移动光标 */
}

.not-allowed {
  cursor: not-allowed; /* 鼠标悬停时显示不允许的光标 */
}
```

**示例：**

```html
<button class="clickable">点击我</button>
<div class="text-editable">这里可以编辑文本</div>
<div class="movable">我可以被移动</div>
<div class="not-allowed">不允许的操作</div>
```

```css
button {
  cursor: pointer; /* 按钮上总是显示手形光标 */
}

.text-editable {
  cursor: text; /* 文本可编辑区域显示文本光标 */
}

.movable {
  cursor: move; /* 可移动元素显示移动光标 */
}

.not-allowed {
  cursor: not-allowed; /* 不允许操作的区域显示不允许光标 */
}
```

