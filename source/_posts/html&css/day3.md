---
title: HTML 基础知识笔记 - 第3天
date: 2025.06.06
updated:
tags:
categories:
keywords:
description: HTML 基础知识笔记 - 第3天
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

# CSS 基础知识笔记 - 第3天

## CSS 选择器

CSS 提供了多种选择器用于定位 HTML 元素：

### 基础选择器

1. **标签选择器**
    ```css
    p {
        color: red;
    }
    ```

2. **类选择器**
    ```css
    .box {
        width: 100px;
        height: 100px;
    }
    ```

3. **ID 选择器**
    ```css
    #header {
        height: 80px;
    }
    ```

4. **通配符选择器**
    ```css
    * {
        margin: 0;
        padding: 0;
    }
    ```



## 字体样式

### 字体大小
```css
p {
    font-size: 16px;    /* 像素 */
    font-size: 1.2em;   /* 相对于父元素 */
    font-size: 1.2rem;  /* 相对于根元素 */
}
```

### 字体粗细
```css
p {
    font-weight: normal;  /* 正常粗细，等同于 400 */
    font-weight: bold;    /* 加粗，等同于 700 */
    font-weight: 400;     /* 100-900，值越大越粗 */
}
```

### 字体倾斜
```css
p {
    font-style: normal;  /* 正常 */
    font-style: italic;  /* 斜体 */
}
```

### 行高
```css
p {
    line-height: 1.5;     /* 无单位，表示字体大小的倍数 */
    line-height: 24px;    /* 固定像素值 */
    line-height: 150%;    /* 字体大小的百分比 */
}
```

小技巧：利用行高设置垂直居中效果（单行文本）
```css
.box {
    height: 100px;
    line-height: 100px;  /* 行高 == 盒子高 */
}
```

### 字体族
```css
body {
    font-family: "Microsoft YaHei", Arial, sans-serif;
}
```
常用字体族：
- 衬线字体（serif）：如 Times New Roman
- 无衬线字体（sans-serif）：如 Arial、Helvetica
- 等宽字体（monospace）：如 Courier New
- 草书字体（cursive）
- 幻想字体（fantasy）

### font 复合属性
```css
p {
    /* font: font-style font-weight font-size/line-height font-family */
    font: italic bold 16px/1.5 "Microsoft YaHei", sans-serif;
}
```
注意：
- font-size 和 font-family 是必须的
- 其他属性如果不指定则使用默认值

## 文本样式

### 文本缩进
```css
p {
    text-indent: 2em;  /* 缩进两个字符 */
}
```

### 文本对齐
```css
p {
    text-align: left;    /* 左对齐（默认） */
    text-align: center;  /* 居中对齐 */
    text-align: right;   /* 右对齐 */
    text-align: justify; /* 两端对齐 */
}
```

### 修饰线
```css
p {
    text-decoration: none;         /* 无装饰线（常用于去除链接下划线） */
    text-decoration: underline;    /* 下划线 */
    text-decoration: overline;     /* 上划线 */
    text-decoration: line-through; /* 删除线 */
}
```

### 文本转换
```css
p {
    text-transform: none;       /* 默认 */
    text-transform: uppercase;  /* 全部大写 */
    text-transform: lowercase;  /* 全部小写 */
    text-transform: capitalize; /* 首字母大写 */
}
```

### 字母间距和单词间距
```css
p {
    letter-spacing: 2px;  /* 字母间距 */
    word-spacing: 5px;    /* 单词间距 */
}
```

### 文本颜色
```css
p {
    color: red;           /* 关键字 */
    color: #ff0000;       /* 十六进制 */
    color: rgb(255,0,0);  /* RGB */
    color: rgba(255,0,0,0.5); /* RGBA，带透明度 */
}
```

## 背景样式

### 背景颜色
```css
div {
    background-color: #f5f5f5;
}
```

### 背景图片
```css
div {
    background-image: url('image.jpg');
    
    /* 背景重复 */
    background-repeat: repeat;     /* 默认，水平和垂直方向都重复 */
    background-repeat: no-repeat;  /* 不重复 */
    background-repeat: repeat-x;   /* 仅水平方向重复 */
    background-repeat: repeat-y;   /* 仅垂直方向重复 */
    
    /* 背景位置 */
    background-position: center center; /* 水平和垂直方向都居中 */
    background-position: 50% 50%;       /* 同上，使用百分比 */
    background-position: 100px 200px;   /* 使用像素值 */
    
    /* 背景尺寸 */
    background-size: cover;   /* 覆盖整个容器，可能裁剪图片 */
    background-size: contain; /* 缩放图片以适应容器，保持比例 */
    background-size: 100% 100%; /* 拉伸图片以填充容器 */
    
    /* 背景固定 */
    background-attachment: scroll; /* 默认，随页面滚动 */
    background-attachment: fixed;  /* 固定不动，相对于视口 */
}
```

### background 复合属性
```css
div {
    /* background: color image repeat attachment position/size */
    background: #f5f5f5 url('image.jpg') no-repeat fixed center/cover;
}
```

## CSS 显示模式

### 块级元素
- 独占一行，可设置宽高
- 例如：div, p, h1-h6, ul, li, dl, dt, dd, form, header, nav, footer

### 行内元素
- 一行可显示多个，宽高由内容决定
- 例如：span, a, b, strong, i, em, del, ins

### 行内块元素
- 一行可显示多个，可以设置宽高
- 例如：img, input, button, textarea, select

### 修改显示模式
```css
span {
    display: block;        /* 转为块级 */
}

div {
    display: inline;       /* 转为行内 */
}

div {
    display: inline-block; /* 转为行内块 */
}

div {
    display: none;         /* 隐藏元素（不占位置） */
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