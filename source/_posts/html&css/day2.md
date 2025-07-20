---
title: HTML 基础知识笔记 - 第2天
date: 2025.06.05
updated:
tags:
categories:
keywords:
description: HTML 基础知识笔记 - 第2天
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

# HTML 基础知识笔记 - 第2天

## 列表

### 无序列表
无序列表使用 `<ul>` 和 `<li>` 标签创建：
```html
<ul>
    <li>第一项</li>
    <li>第二项</li>
    <li>第三项</li>
</ul>
```
注意事项：
- `<ul>` 标签定义无序列表
- `<li>` 标签定义列表项
- 默认使用实心圆点作为列表项标记

### 有序列表
有序列表使用 `<ol>` 和 `<li>` 标签创建：
```html
<ol>
    <li>第一项</li>
    <li>第二项</li>
    <li>第三项</li>
</ol>
```
注意事项：
- 同无序列表
- 默认使用数字作为列表项标记
- 可通过 CSS 修改列表样式

### 定义列表
定义列表使用 `<dl>`、`<dt>` 和 `<dd>` 标签创建：
```html
<dl>
    <dt>列表标题</dt>
    <dd>列表描述/详情</dd>
</dl>
```
注意事项：
- `<dl>` 定义定义列表
- `<dt>` 定义列表中的项目（术语）
- `<dd>` 定义列表中项目的描述

## 表格

基本表格结构：
```html
<table border="1"> 
    <tr> <!-- 行标签 -->
        <th>title1</th> <!-- 表头 -->
        <th>title2</th>
    </tr> 
    <tr>
        <td>elem1</td> <!-- 表内数据 -->
        <td>elem2</td>
    </tr>
</table>
```

表格可以使用以下标签组织结构：
```html
<thead> <!-- 表头部分 -->
    <tr>...</tr>
</thead>
<tbody> <!-- 表格主体 -->
    <tr>...</tr>
</tbody>
<tfoot> <!-- 表格底部 -->
    <tr>...</tr>
</tfoot>
```

### 合并单元格

#### 跨行合并
使用 `rowspan` 属性进行跨行合并：
```html
<td rowspan="2">跨两行的单元格</td>
```

#### 跨列合并
使用 `colspan` 属性进行跨列合并：
```html
<td colspan="2">跨两列的单元格</td>
```
注意：删除对应行/列的内容以使合并生效

## 表单

### input 输入框
```html
<input type="text" placeholder="请输入内容">
```

属性说明：
- `type="text"` - 创建文本输入框
- `type="password"` - 创建密码输入框
- `placeholder` - 输入框提示文字
- `type="radio"` - 创建单选按钮
- `name` - 表单元素名称，同名的单选按钮为一组
- `checked` - 默认选中
- `type="file"` - 创建文件上传
- `multiple` - 允许多文件上传
- `type="checkbox"` - 创建复选框

### select 下拉菜单

```html
<select>
    <option>选项1</option>
    <option>选项2</option>
    <option>选项3</option>
    <option selected>选项4</option>
</select>
```

属性说明：
- `selected` - 默认选中的选项

### textarea 文本区域

```html
<textarea name="message" id="message" cols="30" rows="10">提示文字</textarea>
```

属性说明：
- `name` - 表单元素名称
- `id` - 元素唯一标识
- `cols` - 列数（宽度），建议用 CSS 设置
- `rows` - 行数（高度），建议用 CSS 设置

### label 标签

用于关联文本和表单控件，提高可访问性和用户体验。

#### 写法一
```html
<label for="username">用户名：</label>
<input type="text" id="username">
```
label 中只有内容，不包括表单控件。设置 label 的 `for` 属性值和表单控件的 `id` 一致。

#### 写法二
```html
<label>
    用户名：<input type="text">
</label>
```
label 直接包裹文字和控件，不需要 `for` 属性。

### button 按钮
```html
<button type="submit">提交</button>
```

类型说明：
- `type="submit"` - 提交表单
- `type="reset"` - 重置表单
- `type="button"` - 普通按钮，需要通过 JavaScript 添加功能

### form 表单
```html
<form action="process.php" method="post">
    <!-- 表单元素 -->
</form>
```

默认换行

属性说明：

- `action` - 表单提交的目标 URL
- `method` - 提交方法，常用 `get` 或 `post`

## div 和 span

用于页面布局和内容分组：

```html
<div>块级元素，默认换行</div>
<span>内联元素，默认不换行</span>
```

用途说明：
- `<div>` - 块级元素，通常用于页面布局
- `<span>` - 内联元素，通常用于文本样式化

## 字符实体（转义）

HTML 中的特殊字符需要使用字符实体来表示：

| 显示结果 | 描述 | 实体名称 | 实体编号 |
|---------|------|---------|---------|
| &nbsp;  | 不断行的空格 | `&nbsp;` | `&#160;` |
| &lt;    | 小于号 | `&lt;` | `&#60;` |
| &gt;    | 大于号 | `&gt;` | `&#62;` |
| &amp;   | 和号 | `&amp;` | `&#38;` |
| &quot;  | 引号 | `&quot;` | `&#34;` |
| &copy;  | 版权符号 | `&copy;` | `&#169;` |
| &reg;   | 注册商标 | `&reg;` | `&#174;` |
