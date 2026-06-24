---
title: LaTeX graphicx 宏包详解
date: 2026-06-24
updated: 2026-06-25 00:17:12
tags: [LaTeX, graphicx, 图片, 插图, 排版]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 全面介绍 LaTeX graphicx 宏包的用法，包括 \includegraphics 的各类参数、图形路径设置、旋转缩放裁剪等实用技巧。
keywords: LaTeX, graphicx, includegraphics, 插图, 图片, 缩放, 旋转, 裁剪
cover: https://picsum.photos/id/26/800/450
mathjax: false
katex: false
---

# LaTeX `graphicx` 宏包详解

`graphicx` 是 LaTeX 插图的标准宏包，核心命令 `\includegraphics` 通过 `key=value` 参数控制图片的缩放、旋转、裁剪等效果。

## 1. 加载宏包

在导言区加载：

```latex
\usepackage{graphicx}
```

注意：旧版 `graphics` 宏包语法受限，`graphicx` 支持 `key=value` 可选参数，**始终使用 `graphicx`**。

## 2. 核心命令：`\includegraphics`

### 2.1 基本语法

```latex
\includegraphics[选项]{文件名}
\includegraphics{photo.jpg}   % 不指定选项 → 按原始尺寸插入
```

- 超出页面宽度时用 `width`/`height`/`scale` 控制

### 2.2 缩放与尺寸控制

| 参数 | 说明 | 示例 |
|------|------|------|
| `width` | 设定宽度 | `width=0.8\textwidth` |
| `height` | 设定高度 | `height=5cm` |
| `scale` | 按比例缩放 | `scale=0.5`（缩小为原来的 50%） |
| `totalheight` | 设定总高度（含旋转后的高度） | `totalheight=3cm` |
| `keepaspectratio` | 保持纵横比（配合 width 和 height 同时使用时） | `keepaspectratio=true` |

只设 `width` 或只设 `height` 时，另一个维度会自动按比例缩放：

```latex
% 只设 width → 高度自动按比例缩放
\includegraphics[width=0.5\textwidth]{example.png}

% 只设 height → 宽度自动按比例缩放
\includegraphics[height=4cm]{example.png}

% 等比缩放到版心宽度（常用）
\includegraphics[width=\textwidth]{landscape.jpg}

% scale 按比例缩放
\includegraphics[scale=0.8]{diagram.pdf}

% 同时设 width 和 height → 拉伸图片；加 keepaspectratio 以"不超过"原则适配较小边界
\includegraphics[width=0.8\textwidth, height=5cm, keepaspectratio]{example.png}
```

双图并列是 width 控制的典型场景：

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.45\textwidth]{left.png}
    \hfill
    \includegraphics[width=0.45\textwidth]{right.png}
    \caption{两张图并列}
    \label{fig:sidebyside}
\end{figure}
```

### 2.3 旋转

使用 `angle` 参数按角度旋转图片（正值表示逆时针）：

```latex
% angle 逆时针旋转；origin 指定旋转中心（c=中心（默认），tl=左上角，br=右下角等）
\includegraphics[angle=90]{example.png}
\includegraphics[width=5cm, angle=45]{example.png}
\includegraphics[angle=90, origin=tl]{example.png}
```

### 2.4 裁剪（trim 与 clip）

`trim` 可以从图片四边裁掉指定的长度，语法为 `trim = 左 下 右 上`：

```latex
% 从左、下、右、上分别裁掉 1cm、2cm、1cm、0.5cm
\includegraphics[trim=1cm 2cm 1cm 0.5cm, clip]{example.png}
% 只裁左边 3cm
\includegraphics[trim=3cm 0 0 0, clip]{example.png}
```

注意：**必须有 `clip` 才会真正裁掉指定区域**，否则 `trim` 只相当于增加负边距，裁剪掉的内容仍然可见。

### 2.5 页面选择（多页 PDF）

`page` 参数用于指定插入多页 PDF 的第几页：

```latex
% 插入 PDF 文件的第 2 页
\includegraphics[page=2, width=\textwidth]{document.pdf}

% page + trim + clip：从 PDF 矢量图中截取局部区域
\includegraphics[page=3, trim=2cm 5cm 2cm 3cm, clip, width=0.8\textwidth]{paper.pdf}
```

### 2.6 `bb`（BoundingBox，较少使用）

```latex
\includegraphics[bb=0 0 100 200]{example.eps}
```

用于显式指定 EPS 文件的 BoundingBox。大多数情况下 LaTeX 会自动读取，但在某些非标准 EPS 文件中可能需要手动设置。

## 3. 图形路径配置

### 3.1 `\graphicspath`

`\graphicspath` 统一指定搜索路径，图片集中存放时无需每次写完整路径：

```latex
\usepackage{graphicx}
\graphicspath{{figures/}{photos/}}    % 多路径用花括号分隔
```

注意：路径末尾的 `/` 必须保留，且每个路径用 `{}` 括起来。

### 3.2 `\DeclareGraphicsExtensions`

指定默认文件扩展名，`\includegraphics` 无需带扩展名：

```latex
\DeclareGraphicsExtensions{.pdf,.png,.jpg}
```

LaTeX 按声明顺序依次搜索匹配文件。同一个图有 PDF 和 PNG 两个版本时，声明 `.pdf` 在前即可优先使用 PDF。

```latex
\graphicspath{{images/}}
\DeclareGraphicsExtensions{.pdf,.png}

% 实际搜索 images/example.pdf，找不到再找 images/example.png
\includegraphics[width=5cm]{example}
```

## 4. 支持的图片格式

不同编译引擎支持的格式有所差异：

| 编译引擎 | 支持格式 |
|----------|----------|
| **pdfLaTeX** | PDF、PNG、JPEG |
| **XeLaTeX** | PDF、PNG、JPEG、EPS、BMP |
| **LuaLaTeX** | PDF、PNG、JPEG、EPS、BMP |
| **LaTeX（DVI 模式）** | EPS |

日常使用：**优先使用 PDF（矢量）或 PNG（位图）**，兼容性最好。

## 5. `\includegraphics` 完整参数速查表

以下按功能分类汇总 `\includegraphics` 支持的关键参数：

| 分类 | 参数 | 可选值 | 说明 |
|------|------|--------|------|
| 尺寸 | `width` | 长度（如 `0.5\textwidth`、`5cm`） | 图片宽度 |
| 尺寸 | `height` | 长度 | 图片高度 |
| 尺寸 | `scale` | 数值 | 缩放比例 |
| 尺寸 | `totalheight` | 长度 | 旋转后的总高度 |
| 尺寸 | `keepaspectratio` | `true` / `false` | 保持纵横比 |
| 旋转 | `angle` | 角度值 | 旋转角度 |
| 旋转 | `origin` | `c`、`tl`、`tr`、`bl`、`br` 等 | 旋转中心点 |
| 裁剪 | `trim` | `左 下 右 上`（四个长度） | 裁剪区域 |
| 裁剪 | `clip` | `true` / `false` | 是否实际裁剪 |
| 页面 | `page` | 整数 | 多页 PDF 的页码 |
| 其他 | `draft` | `true` / `false` | 草稿模式（只显示文件名不插入图片） |
| 其他 | `type` | 文件扩展名 | 覆盖自动检测的图片类型 |
| 其他 | `ext` | 文件扩展名 | 指定扩展名 |
| 其他 | `read` | 文件扩展名 | 指定读取文件的扩展名 |

## 6. 草稿模式

`draft` 模式下图片不实际插入，只显示文件名框，加速编译：

```latex
\usepackage[draft]{graphicx}            % 全局草稿模式
\includegraphics[draft=false]{a.png}    % 单张强制正常显示
\includegraphics[draft]{big.png}        % 单张开启草稿
```

## 7. 小结

`graphicx` 是 LaTeX 插图的基础设施，核心命令就一个 `\includegraphics`，但参数组合丰富，几乎覆盖了所有插图场景：

- **尺寸控制**：`width`、`height`、`scale`、`keepaspectratio`
- **旋转裁剪**：`angle`、`origin`、`trim`、`clip`
- **多页支持**：`page`
- **路径管理**：`\graphicspath`、`\DeclareGraphicsExtensions`
- **草稿加速**：`draft`

掌握这些用法，就能应对绝大多数 LaTeX 插图需求。如需更复杂的图文混排效果，可进一步配合 `wrapfig`（文字绕排）、`float`（浮动控制）等宏包使用。
