---
title: HTML 基础知识笔记 - 第4天
date: 2025.06.06
updated:
tags:
categories:
keywords:
description: HTML 基础知识笔记 - 第4天
top_img:
comments:
cover:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
abcjs:
noticeOutdate:
---

# CSS 基础知识笔记 - 第4天

## CSS 选择器
### 复合选择器

1. **后代选择器**
    ```css
    .nav li {
        color: blue;
    }
    ```

2. **子元素选择器**
    ```css
    .nav > li {
        font-size: 16px;
    }
    ```

3. **并集选择器**
    ```css
    h1, h2, h3 {
        font-weight: 700;
    }
    ```

4. **伪类选择器**
    ```css
    a{
        color: green;
    }
    a:hover {
        color: red;
    }
    ```
    
    常用伪类选择器：
    ```css
    /* 未访问的链接 */
    a:link {
        color: blue;
    }
    
    /* 已访问的链接 */
    a:visited {
        color: purple;
    }
    
    /* 鼠标悬停 */
    a:hover {
        color: red;
    }
    
    /* 激活状态（鼠标按下） */
    a:active {
        color: orange;
    }
    
    /* 获取焦点 */
    input:focus {
        border-color: blue;
    }
    
    /* 第一个子元素 */
    li:first-child {
        font-weight: bold;
    }
    
    /* 最后一个子元素 */
    li:last-child {
        border-bottom: none;
    }
    
    /* 奇数项 */
    li:nth-child(odd) {
        background-color: #f5f5f5;
    }
    
    /* 偶数项 */
    li:even-child(even) {
        background-color: #e5e5e5;
    }
    
    /* 特定位置的子元素 */
    li:nth-child(3) {
        color: red;
    }
    ```
    如果要具体设置超链接格式，按照lvha的顺序

5. **属性选择器**
    ```css
    /* 具有特定属性的元素 */
    input[type] {
        border: 1px solid #ccc;
    }
    
    /* 属性值相等 */
    input[type="text"] {
        background-color: #f5f5f5;
    }
    
    /* 属性值包含特定词 */
    a[href*="example"] {
        color: green;
    }
    
    /* 属性值以特定字符开头 */
    a[href^="https"] {
        color: blue;
    }
    
    /* 属性值以特定字符结尾 */
    a[href$=".pdf"] {
        color: red;
    }
    ```

## CSS 特性

### 层叠性
- 相同选择器设置相同的样式，后面的会覆盖前面的

### 继承性
- 子元素会继承父元素的某些样式，如文字颜色、字体大小等
- 一般与文字相关的属性会被继承

### 优先级
- !important > 行内样式 > ID选择器 > 类选择器 > 标签选择器 > 通配符 > 继承
- 权重计算：(行内样式, ID选择器个数, 类选择器个数, 标签选择器个数)
- 例如：#nav .list li 的权重为 (0, 1, 1, 1)，大于 .nav p 的权重 (0, 0, 1, 1)

## Emmet 写法

Emmet 是一种能提高前端开发效率的工具，它使用缩写语法快速生成 HTML 和 CSS 代码。

### HTML Emmet 缩写

1. **子元素**：使用 `>` 符号
    ```
    div>ul>li
    ```
    生成：
    ```html
    <div>
        <ul>
            <li></li>
        </ul>
    </div>
    ```

2. **兄弟元素**：使用 `+` 符号
    ```
    div+p+span
    ```
    生成：
    ```html
    <div></div>
    <p></p>
    <span></span>
    ```

3. **上级元素**：使用 `^` 符号
    ```
    div>p>span^a
    ```
    生成：
    ```html
    <div>
        <p><span></span></p>
        <a></a>
    </div>
    ```

4. **重复元素**：使用 `*` 符号
    ```
    ul>li*5
    ```
    生成：
    ```html
    <ul>
        <li></li>
        <li></li>
        <li></li>
        <li></li>
        <li></li>
    </ul>
    ```

5. **编号**：使用 `$` 符号
    ```
    ul>li.item$*3
    ```
    生成：
    ```html
    <ul>
        <li class="item1"></li>
        <li class="item2"></li>
        <li class="item3"></li>
    </ul>
    ```

6. **属性**：使用 `[attr=value]` 语法
    ```
    a[href='https://example.com' target='_blank']
    ```
    生成：
    ```html
    <a href="https://example.com" target="_blank"></a>
    ```

7. **类与ID**：使用 `.` 和 `#` 符号
    ```
    div#header.container.flex
    ```
    生成：
    ```html
    <div id="header" class="container flex"></div>
    ```

8. **内容**：使用 `{}` 符号
    ```
    p{这是一段文字}
    ```
    生成：
    ```html
    <p>这是一段文字</p>
    ```

### CSS Emmet 缩写

1. **属性缩写**：
    ```
    m10 -> margin: 10px;
    p20 -> padding: 20px;
    w100 -> width: 100px;
    h200 -> height: 200px;
    ```

2. **属性值缩写**：
    ```
    m10-20 -> margin: 10px 20px;
    p10-20-30 -> padding: 10px 20px 30px;
    ```

3. **多个属性**：
    ```
    m10+p20 -> margin: 10px; padding: 20px;
    ```

4. **单位**：
    ```
    w100p -> width: 100%;
    h10e -> height: 10em;
    ```

## CSS 背景属性

### 背景颜色
```css
div {
    background-color: #f5f5f5;
    background-color: rgb(245, 245, 245);
    background-color: rgba(245, 245, 245, 0.5); /* 带透明度 */
}
```

### 背景图片
```css
div {
    background-image: url('image.jpg');
}
```

### 背景重复
```css
div {
    background-repeat: repeat;     /* 默认，水平和垂直方向都重复 */
    background-repeat: no-repeat;  /* 不重复 */
    background-repeat: repeat-x;   /* 仅水平方向重复 */
    background-repeat: repeat-y;   /* 仅垂直方向重复 */
}
```

### 背景附着
```css
div {
    background-attachment: scroll; /* 默认，随页面滚动 */
    background-attachment: fixed;  /* 固定不动，相对于视口 */
    background-attachment: local;  /* 相对于元素内容固定 */
}
```

### 背景位置
```css
div {
    /* 关键字 */
    background-position: top;
    background-position: right;
    background-position: bottom;
    background-position: left;
    background-position: center;
    
    /* 组合关键字 */
    background-position: top right;
    background-position: center center;
    
    /* 百分比 */
    background-position: 50% 50%;     /* 水平和垂直居中 */
    background-position: 0% 0%;       /* 左上角 */
    background-position: 100% 100%;   /* 右下角 */
    
    /* 像素值 */
    background-position: 20px 30px;   /* 距左边20px，距上边30px */
}
```

### 背景尺寸
```css
div {
    background-size: auto;     /* 默认值，保持图片原始大小 */
    background-size: cover;    /* 覆盖整个容器，可能裁剪图片 */
    background-size: contain;  /* 缩放图片以适应容器，保持比例 */
    background-size: 100% 100%; /* 拉伸图片以完全填充容器 */
    background-size: 200px 150px; /* 设置具体尺寸 */
}
```

### 背景原点
```css
div {
    background-origin: padding-box; /* 默认值，从padding区域开始显示 */
    background-origin: border-box;  /* 从border区域开始显示 */
    background-origin: content-box; /* 从内容区域开始显示 */
}
```

### 背景裁剪
```css
div {
    background-clip: border-box;  /* 默认值，延伸至边框 */
    background-clip: padding-box; /* 延伸至内边距 */
    background-clip: content-box; /* 仅在内容区域显示 */
    background-clip: text;        /* 仅在文字区域显示（需配合 -webkit-text-fill-color: transparent; 使用） */
}
```

### 多重背景
```css
div {
    background: 
        url('top.png') top center no-repeat,
        url('middle.png') center center no-repeat,
        url('bottom.png') bottom center no-repeat,
        #f5f5f5;
}
```

### 背景复合属性
```css
div {
    /* background: color image repeat attachment position/size */
    background: #f5f5f5 url('image.jpg') no-repeat fixed center/cover;
}
```

## CSS 显示模式

### 块级元素
```css
div, p, h1-h6, ul, ol, dl, form, header, nav, footer, section, article {
    /* 块级元素的特点 */
    display: block; /* 默认值 */
    width: 100%; /* 默认占满父容器宽度 */
    height: auto; /* 默认由内容决定高度 */
}
```

特点：
- 独占一行，从新行开始，自动换行
- 可以设置宽度和高度
- 宽度默认是父容器的100%
- 可以设置外边距和内边距

### 行内元素
```css
span, a, strong, em, b, i, label, small, cite, code {
    /* 行内元素的特点 */
    display: inline; /* 默认值 */
}
```

特点：
- 一行可以显示多个
- 不能直接设置宽高
- 宽度和高度由内容决定
- 只能设置水平方向的外边距和内边距

### 行内块元素
```css
img, input, button, textarea, select {
    /* 行内块元素的特点 */
    display: inline-block;
}
```

特点：
- 一行可以显示多个（像行内元素）
- 可以设置宽高（像块级元素）
- 可以设置外边距和内边距

### 修改显示模式
```css
/* 将行内元素设置为块级 */
span.block {
    display: block;
}

/* 将块级元素设置为行内 */
div.inline {
    display: inline;
}

/* 将元素设置为行内块 */
p.inline-block {
    display: inline-block;
}

/* 隐藏元素（不占空间） */
.hidden {
    display: none;
}

/* 隐藏元素（占空间） */
.invisible {
    visibility: hidden;
}

/* 弹性布局 */
.flex-container {
    display: flex;
}

/* 网格布局 */
.grid-container {
    display: grid;
}
```