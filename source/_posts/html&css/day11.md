---
title: CSS 笔记 - 第11天
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
# CSS 基础知识笔记 - 第11天

## 空间转换-视距
在进行 3D 转换时，`perspective` 属性用于定义观察者与 z=0 平面之间的距离。它能让 3D 转换看起来更真实，具有近大远小的效果。
- `perspective: none | length;`
- `length`：通常以像素 (px) 为单位。值越小，透视效果越明显，变形越大。
- 该属性通常应用在父元素上，影响其所有进行 3D 转换的子元素。也可以通过 `transform: perspective(length);` 应用在单个元素上，但效果略有不同。

**样例：**
```html
<div class="perspective-container">
  <div class="box p-box1">Box 1</div>
  <div class="box p-box2">Box 2</div>
</div>
<style>
.perspective-container {
  width: 200px;
  height: 200px;
  border: 1px solid #ccc;
  margin: 50px;
  perspective: 300px; /* 设置视距 */
}
.box {
  width: 100px;
  height: 100px;
  background-color: lightcoral;
  margin: 50px auto;
  text-align: center;
  line-height: 100px;
  transition: transform 1s;
}
.p-box1:hover {
  transform: rotateY(45deg);
}
.p-box2 {
  background-color: lightseagreen;
}
.p-box2:hover {
  transform: translateZ(50px) rotateY(45deg); /* 向观察者平移并旋转 */
}
</style>
```

## 空间转换-平移
通过 `transform` 属性和 `translate3d(x, y, z)` 或 `translateZ(z)` 函数实现元素在三维空间中的平移。
- `transform: translate3d(tx, ty, tz);`：元素在 X、Y、Z 轴上分别平移 tx, ty, tz。
- `transform: translateX(tx);`：沿 X 轴平移。
- `transform: translateY(ty);`：沿 Y 轴平移。
- `transform: translateZ(tz);`：沿 Z 轴平移。正值表示向观察者靠近，负值表示远离观察者。需要配合 `perspective` 才能看到 Z 轴平移的效果。

**样例：**
```html
<div class="perspective-container">
  <div class="box translate3d-example">TranslateZ</div>
</div>
<style>
/* .perspective-container 和 .box 样式同上一个例子 */
.translate3d-example:hover {
  transform: translateZ(100px); /* 鼠标悬浮时向观察者移动100px */
}
</style>
```

## 空间转换-旋转
通过 `transform` 属性和 `rotate3d(x, y, z, angle)` 或 `rotateX(angle)`, `rotateY(angle)`, `rotateZ(angle)` 函数实现元素在三维空间中的旋转。
- `transform: rotate3d(x, y, z, angle);`：元素围绕一个由 `[x, y, z]` 向量定义的轴旋转指定的角度。
- `transform: rotateX(angle);`：围绕 X 轴旋转。
- `transform: rotateY(angle);`：围绕 Y 轴旋转。
- `transform: rotateZ(angle);`：围绕 Z 轴旋转（效果同 2D 旋转 `rotate(angle)`）。
  同样，`rotateX` 和 `rotateY` 的效果需要 `perspective` 才能明显观察到。

**样例：**
```html
<div class="perspective-container">
  <div class="box rotateX-example">RotateX</div>
  <div class="box rotateY-example">RotateY</div>
</div>
<style>
/* .perspective-container 和 .box 样式同上一个例子 */
.rotateX-example { background-color: lightgoldenrodyellow; }
.rotateX-example:hover {
  transform: rotateX(60deg); /* 鼠标悬浮时围绕X轴旋转60度 */
}
.rotateY-example { background-color: lightpink; }
.rotateY-example:hover {
  transform: rotateY(60deg); /* 鼠标悬浮时围绕Y轴旋转60度 */
}
</style>
```

## 空间转换-立体呈现 transform-style
`transform-style` 属性定义了嵌套元素如何在 3D 空间中呈现。
- `transform-style: flat;` (默认值): 所有子元素在父元素的平面内呈现。
- `transform-style: preserve-3d;`：所有子元素在 3D 空间中独立定位。这个属性非常重要，用于创建复杂的多层 3D 结构。它必须应用在父元素上，并且该父元素本身也需要有 `transform` 属性（即使是 `transform: none;` 也可以激活）。

**样例：创建一个立方体**
```html
<div class="cube-container">
  <div class="cube">
    <div class="face front">Front</div>
    <div class="face back">Back</div>
    <div class="face right">Right</div>
    <div class="face left">Left</div>
    <div class="face top">Top</div>
    <div class="face bottom">Bottom</div>
  </div>
</div>
<style>
.cube-container {
  width: 200px;
  height: 200px;
  margin: 100px;
  perspective: 800px; /* 父容器视距 */
}
.cube {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d; /* 关键：使子元素在3D空间呈现 */
  transform: rotateX(-30deg) rotateY(-45deg);
  transition: transform 2s;
}
.cube:hover {
  transform: rotateX(-30deg) rotateY(-135deg);
}
.face {
  position: absolute;
  width: 200px;
  height: 200px;
  border: 1px solid grey;
  background-color: rgba(173, 216, 230, 0.7); /* lightblue with alpha */
  font-size: 20px;
  text-align: center;
  line-height: 200px;
  color: black;
  opacity: 0.8;
}
.front  { transform: translateZ(100px); }
.back   { transform: rotateY(180deg) translateZ(100px); }
.right  { transform: rotateY(90deg) translateZ(100px); }
.left   { transform: rotateY(-90deg) translateZ(100px); }
.top    { transform: rotateX(90deg) translateZ(100px); }
.bottom { transform: rotateX(-90deg) translateZ(100px); }
</style>
```

## 空间转换-缩放
通过 `transform` 属性和 `scale3d(sx, sy, sz)` 或 `scaleZ(sz)` 函数实现元素在三维空间中的缩放。
- `transform: scale3d(sx, sy, sz);`：元素在 X、Y、Z 轴上分别缩放 sx, sy, sz 倍。
- `transform: scaleX(sx);`：沿 X 轴缩放。
- `transform: scaleY(sy);`：沿 Y 轴缩放。
- `transform: scaleZ(sz);`：沿 Z 轴缩放。`scaleZ` 的效果通常不直接可见，除非元素有厚度或者与其他 3D 转换结合使用。

**样例：**
```html
<div class="perspective-container">
  <div class="box scale3d-example">Scale3D</div>
</div>
<style>
/* .perspective-container 和 .box 样式同上一个例子 */
.scale3d-example { background-color: mediumpurple; }
.scale3d-example:hover {
  transform: scale3d(1.2, 1.2, 1.5) rotateY(30deg); /* 放大并旋转 */
}
</style>
```

## 动画效果

### 动画定义key-frame
`@keyframes` 规则用于创建动画。通过指定动画过程中特定时间点的 CSS 样式，浏览器会在这些关键帧之间创建平滑的过渡。
- 语法：
  ```css
  @keyframes animationname {
    from { /* CSS 样式 */ } /* 0% */
    percentage { /* CSS 样式 */ } /* 例如 50% */
    to { /* CSS 样式 */ }   /* 100% */
  }
  ```
- `animationname` 是动画的名称，稍后会通过 `animation-name` 属性引用。
- `from` 等同于 `0%`，`to` 等同于 `100%`。可以定义多个百分比关键帧。

**样例：一个简单的颜色和位置变化动画**
```css
@keyframes colorAndMove {
  0% {
    background-color: red;
    transform: translateX(0);
  }
  50% {
    background-color: yellow;
    transform: translateX(100px);
  }
  100% {
    background-color: blue;
    transform: translateX(0);
  }
}
```

### 动画使用 animation
定义了 `@keyframes` 后，需要使用 `animation` 相关属性将动画应用到元素上。
主要属性包括：
- `animation-name`: 指定要应用的 `@keyframes` 名称。
- `animation-duration`: 动画完成一次所需的时间 (e.g., `2s`, `500ms`)。
- `animation-timing-function`: 动画的速度曲线 (e.g., `linear`, `ease`, `ease-in`, `ease-out`, `ease-in-out`, `cubic-bezier()`, `steps()`)。
- `animation-delay`: 动画开始前的延迟时间。
- `animation-iteration-count`: 动画播放的次数 (e.g., `3`, `infinite`)。
- `animation-direction`: 动画是否轮流反向播放 (e.g., `normal`, `reverse`, `alternate`, `alternate-reverse`)。
- `animation-fill-mode`: 动画不播放时（开始前、结束后或两者）元素的样式 (e.g., `none`, `forwards`, `backwards`, `both`)。
- `animation-play-state`: 控制动画的播放状态 (e.g., `running`, `paused`)。

**简写属性：**
`animation: name duration timing-function delay iteration-count direction fill-mode play-state;`

**样例：应用上面定义的 `colorAndMove` 动画**
```html
<div class="animated-box"></div>
<style>
.animated-box {
  width: 100px;
  height: 100px;
  background-color: red; /* 初始颜色 */
  animation-name: colorAndMove;
  animation-duration: 4s;
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
  animation-direction: alternate;
}
/* @keyframes colorAndMove 定义同上 */
</style>
```

### 走马灯样例
走马灯效果通常指内容从一侧平滑滚动到另一侧，并循环。
可以使用 `@keyframes` 改变 `transform: translateX()` 的值来实现。

**样例：**
```html
<div class="marquee-container">
  <div class="marquee-content">这是一段滚动的文字... 这是一段滚动的文字... 这是一段滚动的文字...</div>
</div>
<style>
.marquee-container {
  width: 300px; /* 可视区域宽度 */
  overflow: hidden; /* 隐藏超出部分 */
  border: 1px solid black;
  background-color: #f0f0f0;
}
.marquee-content {
  display: inline-block; /* 使宽度由内容决定 */
  white-space: nowrap; /* 防止文字换行 */
  padding-left: 100%; /* 初始时将内容完全移出可视区域右侧 */
  animation: marquee-scroll 15s linear infinite;
}
@keyframes marquee-scroll {
  0% {
    transform: translateX(0); /* 从右侧开始（因为padding-left:100%） */
  }
  100% {
    transform: translateX(-200%); /* 移动到左侧完全消失（内容宽度 + 容器宽度） */
                                  /* 这个值需要根据实际内容宽度调整，或者使用JS计算 */
  }
}
/* 更精确的走马灯，通常需要JS计算内容宽度以确保无缝滚动，
   或者使用重复的内容块。纯CSS实现时，translateX的结束值需要仔细调整。
   例如，如果.marquee-content的实际宽度是600px，容器是300px，
   则需要从 translateX(0) 到 translateX(-600px - 300px) = translateX(-900px)
   或者更简单的方式是复制内容，让内容宽度大于容器宽度，然后移动一个内容宽度。
*/
</style>
```
**改进的走马灯样例 (使用重复内容思路):**
```html
<div class="marquee-container-improved">
  <span class="marquee-text-improved">这是一段滚动的文字...&nbsp;&nbsp;&nbsp;</span>
  <span class="marquee-text-improved">这是一段滚动的文字...&nbsp;&nbsp;&nbsp;</span> <!-- 重复内容 -->
</div>
<style>
.marquee-container-improved {
  width: 300px;
  overflow: hidden;
  border: 1px solid black;
  background-color: #f0f0f0;
  white-space: nowrap; /* 父元素设置 nowrap */
}
.marquee-text-improved {
  display: inline-block;
  animation: marquee-scroll-improved 10s linear infinite;
}
@keyframes marquee-scroll-improved {
  0% {
    transform: translateX(0%);
  }
  100% {
    transform: translateX(-100%); /* 移动一个自身宽度的距离 */
  }
}
</style>
```


### 精灵动画与多组动画

#### 精灵动画
精灵动画 (Sprite Animation) 是通过快速连续显示一张包含多个帧的图片（精灵图）中的不同部分，来创建动画效果。
这通常用于游戏角色或小的动态图标。
CSS 中可以通过改变 `background-position` 并配合 `steps()` 定时函数来实现。

1.  **准备显示区域**：一个固定大小的 `div`，`overflow: hidden;`。
2.  **定义动画**：使用 `@keyframes`，在每个步骤中改变 `background-position-x` (或 `y`) 的值，以显示精灵图的下一帧。
3.  **使用动画**：应用动画到 `div`，并使用 `animation-timing-function: steps(N);`，其中 N 是精灵图一行的帧数。`steps(N)` 会将动画分割成 N 个离散的步骤，而不是平滑过渡。

**样例：**
假设有一张精灵图 `sprite.png`，宽度为 800px，包含 8 帧，每帧 100px 宽。
```html
<div class="sprite-animation"></div>
<style>
.sprite-animation {
  width: 100px;  /* 单帧宽度 */
  height: 150px; /* 单帧高度 */
  background-image: url('path/to/your/sprite.png'); /* 替换为你的精灵图路径 */
  background-repeat: no-repeat;
  animation-name: playSprite;
  animation-duration: 0.8s; /* 8帧，每帧0.1s */
  animation-timing-function: steps(8); /* 精灵图有8帧 */
  animation-iteration-count: infinite;
}

@keyframes playSprite {
  from { background-position-x: 0px; }
  to { background-position-x: -800px; } /* 总宽度，显示完所有帧 */
}
</style>
```
**注意:** 你需要一张实际的精灵图才能看到效果。例如，一个人物走路的8帧雪碧图。

#### 多组动画
一个元素可以同时应用多个动画。在 `animation` 属性中，用逗号分隔多个动画的定义。
`animation: animation1_settings, animation2_settings, ...;`
或者分别设置：
`animation-name: name1, name2;`
`animation-duration: duration1, duration2;`
以此类推。

**样例：一个元素同时进行颜色变化和旋转**
```html
<div class="multi-animated-box">Multi Animation</div>
<style>
@keyframes changeColorMulti {
  0% { background-color: gold; }
  50% { background-color: orangered; }
  100% { background-color: gold; }
}

@keyframes rotateBoxMulti {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.multi-animated-box {
  width: 150px;
  height: 150px;
  margin: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  /* 应用多个动画 */
  animation-name: changeColorMulti, rotateBoxMulti;
  animation-duration: 3s, 5s;
  animation-timing-function: ease-in-out, linear;
  animation-iteration-count: infinite, infinite;
  animation-direction: alternate, normal;
}
</style>
```
在这个例子中，`multi-animated-box` 会同时进行颜色变化（3秒一个周期，来回变化）和旋转（5秒一个周期，持续顺时针旋转）。