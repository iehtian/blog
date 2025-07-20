---
title: HTML 基础知识笔记 - 第1天
date: 2025.06.05
updated:
tags:
categories:
keywords:
description: HTML 基础知识笔记 - 第1天
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

# HTML 基础知识笔记 - 第1天

## 标题和段落

### 标题
HTML 提供了六个级别的标题标签：
```html
<h1></h1> <!-- h1只能有一个 -->
<h2></h2>
<h3></h3>
<h4></h4>
<h5></h5>
<h6></h6>
```

### 段落
段落使用 `<p>` 标签定义：
```html
<p></p>
```

## 换行与水平线

这两个元素都是单标签（不需要闭合标签）。

### 换行
换行标签用于在不开始新段落的情况下换行：
```html
<br>
```

### 水平线
水平线标签用于创建分隔线：
```html
<hr>
```

## 文本格式化

HTML 提供了多种文本格式化标签：
```html
<strong></strong> <!-- 加粗 -->
<em></em> <!-- 倾斜 -->
<ins></ins> <!-- 下划线 -->
<del></del> <!-- 删除线 -->
```

## 图像

图像使用单标签 `<img>` 插入（注意不是 image）：
```html
<img src="" alt="" title="" width="" height="">
```

属性说明：
- `src`：指定图片的路径（必需的属性）
- `alt`：当图片无法显示时的替代文本（提高可访问性）
- `title`：鼠标悬停时显示的提示文本
- `width`：指定图片宽度（可以是像素值或百分比）
- `height`：指定图片高度（可以是像素值或百分比）

## 超链接

超链接使用 `<a>` 标签创建：
```html
<a href="" target=""></a>
```

属性说明：
- `href`：指定链接的目标 URL（必需的属性）
- `target`：指定链接的打开方式，常见值有：
  - `_self`：在当前窗口打开（默认值）
  - `_blank`：在新窗口或标签页打开
  - `_parent`：在父框架中打开
  - `_top`：在整个窗口中打开

## 音频与视频
 
### 音频
音频使用 `<audio>` 标签插入：
```html
<audio src="" controls autoplay loop></audio>
```

属性说明：
- `src`：指定音频文件的路径
- `controls`：显示音频播放控件（播放、暂停、音量等）
- `autoplay`：自动播放音频（注意：许多浏览器会阻止自动播放）
- `loop`：循环播放音频

### 视频
视频使用 `<video>` 标签插入：
```html
<video src="" controls width="" height="" poster="" autoplay loop></video>
```

属性说明：
- `src`：指定视频文件的路径
- `controls`：显示视频播放控件
- `width`/`height`：设置视频播放器的宽度和高度
- `poster`：指定视频未播放时显示的图像
- `autoplay`：自动播放视频（同样受浏览器限制）
- `loop`：循环播放视频
