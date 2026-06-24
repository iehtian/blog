---
title: LaTeX graphicx 宏包详解
date: 2026-06-24
updated: 2026-06-24 23:20:55
tags: [LaTeX, graphicx, 图片, 插图, 排版]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 全面介绍 LaTeX graphicx 宏包的用法，包括 \includegraphics 的各类参数、图形路径设置、旋转缩放裁剪等实用技巧。
cover: https://picsum.photos/id/26/800/450
mathjax: false
katex: false
---

# LaTeX `graphicx` 宏包详解

## 1. 前言

在 LaTeX 中插入图片几乎是最常见的需求之一，而 `graphicx` 宏包正是实现这一功能的标准工具。它提供了 `\includegraphics` 命令，支持丰富的参数来控制图片的缩放、旋转、裁剪等效果。本文系统梳理 `graphicx` 的常用功能与进阶用法，帮助你灵活地排版插图。

## 2. 加载宏包

在导言区通过 `\usepackage` 加载即可：

```latex
\usepackage{graphicx}   % 支持 graphicx 的所有功能
```

注意：LaTeX 还有一个旧版宏包叫 `graphics`（不带 x），语法更受限。`graphicx` 是其后继版本，支持 `key=value` 形式的可选参数，写法更直观，**推荐始终使用 `graphicx`**。

```latex
% graphics（旧版，不推荐）
\includegraphics[width=0.5\textwidth]{example.png}

% graphicx（推荐，同样写法兼容旧语法，且支持更多 key=value 选项）
\includegraphics[width=0.5\textwidth, angle=90]{example.png}
```

## 3. 核心命令：`\includegraphics`

### 3.1 基本语法

```latex
\includegraphics[选项]{文件名}
```

最简写法：

```latex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
    \includegraphics{photo.jpg}
\end{document}
```

LaTeX 会自动按图片原始尺寸插入。如果图片超出页面宽度，就需要通过选项来控制尺寸。

### 3.2 缩放与尺寸控制

| 参数 | 说明 | 示例 |
|------|------|------|
| `width` | 设定宽度 | `width=0.8\textwidth` |
| `height` | 设定高度 | `height=5cm` |
| `scale` | 按比例缩放 | `scale=0.5`（缩小为原来的 50%） |
| `totalheight` | 设定总高度（含旋转后的高度） | `totalheight=3cm` |
| `keepaspectratio` | 保持纵横比（配合 width 和 height 同时使用时） | `keepaspectratio=true` |

只设 `width` 或只设 `height` 时，另一个维度会自动按比例缩放：

```latex
% 宽度设为版心宽度的一半，高度自动计算
\includegraphics[width=0.5\textwidth]{example.png}

% 高度固定为 4cm，宽度自动计算
\includegraphics[height=4cm]{example.png}
```

同时设置 `width` 和 `height` 会改变图片的纵横比，可能导致图片拉伸变形。如需限定最大尺寸又不想变形，加上 `keepaspectratio`：

```latex
% 图片不会超过 0.8 倍版心宽和 5cm 高，且保持比例不变
\includegraphics[width=0.8\textwidth, height=5cm, keepaspectratio]{example.png}
```

提示：此选项不保证同时满足宽度和高度，而是以"不超过"为原则——最终尺寸会在保持比例的前提下适配其中较小的边界。

### 3.3 旋转

使用 `angle` 参数按角度旋转图片（正值表示逆时针）：

```latex
% 逆时针旋转 90 度
\includegraphics[angle=90]{example.png}

% 宽度设为 5cm 的同时旋转 45 度
\includegraphics[width=5cm, angle=45]{example.png}
```

`origin` 参数指定旋转中心，可取值如 `c`（默认，中心）、`tl`（左上角）、`br`（右下角）等：

```latex
\includegraphics[angle=90, origin=tl]{example.png}
```

### 3.4 裁剪（trim 与 clip）

`trim` 可以从图片四边裁掉指定的长度，语法为 `trim = 左 下 右 上`：

```latex
% 从左、下、右、上分别裁掉 1cm、2cm、1cm、0.5cm
\includegraphics[trim=1cm 2cm 1cm 0.5cm, clip]{example.png}
```

注意：**必须有 `clip` 才会真正裁掉指定区域**，否则 `trim` 只是相当于增加了负边距，裁剪掉的内容仍然可见。

```latex
% 只裁左边 3cm
\includegraphics[trim=3cm 0 0 0, clip]{example.png}
```

### 3.5 页面选择（多页 PDF）

`page` 参数用于指定插入多页 PDF 的第几页：

```latex
% 插入 PDF 文件的第 2 页
\includegraphics[page=2, width=\textwidth]{document.pdf}
```

结合 `trim` 和 `clip`，可以从 PDF 矢量图中选取需要的局部区域，非常实用。

### 3.6 `bb`（BoundingBox，较少使用）

```latex
\includegraphics[bb=0 0 100 200]{example.eps}
```

用于显式指定 EPS 文件的 BoundingBox。大多数情况下 LaTeX 会自动读取，但在某些非标准 EPS 文件中可能需要手动设置。

## 4. 图形路径配置

### 4.1 `\graphicspath`

如果图片都放在某个子目录里，可以用 `\graphicspath` 统一指定搜索路径，避免每次写完整路径：

```latex
\usepackage{graphicx}
\graphicspath{{figures/}{photos/}}    % 多路径用花括号分隔
```

注意：路径末尾的 `/` 必须保留，且每个路径用 `{}` 括起来。

### 4.2 `\DeclareGraphicsExtensions`

指定默认的文件扩展名，书写 `\includegraphics` 时就不用带扩展名了：

```latex
\DeclareGraphicsExtensions{.pdf,.png,.jpg}
```

LaTeX 会按声明的顺序依次搜索匹配的文件。这在你手上有同一个图的 PDF 和 PNG 两个版本、想让引擎优先使用 PDF 时特别有用。

```latex
\graphicspath{{images/}}
\DeclareGraphicsExtensions{.pdf,.png}

% 实际搜索 images/example.pdf，找不到再找 images/example.png
\includegraphics[width=5cm]{example}
```

## 5. 支持的图片格式

不同编译引擎支持的格式有所差异：

| 编译引擎 | 支持格式 |
|----------|----------|
| **pdfLaTeX** | PDF、PNG、JPEG |
| **XeLaTeX** | PDF、PNG、JPEG、EPS、BMP |
| **LuaLaTeX** | PDF、PNG、JPEG、EPS、BMP |
| **LaTeX（DVI 模式）** | EPS |

日常使用建议：**优先使用 PDF（矢量）或 PNG（位图）**，兼容性最好。如果用 XeLaTeX 或 LuaLaTeX 引擎编译，也可以直接插入 JPEG 照片。

## 6. `\includegraphics` 完整参数速查表

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

## 7. 草稿模式

`draft` 选项会加快编译速度：图片不会被真正插入，只显示一个带文件名的框，方便在写作阶段快速迭代：

```latex
% 全局草稿模式：加载宏包时设置
\usepackage[draft]{graphicx}

% 局部覆盖：单张图片强制正常显示
\includegraphics[draft=false]{example.png}
```

反过来，也可以在普通模式下对某张大图暂时使用 `draft` 以加速编译调试：

```latex
\includegraphics[draft]{huge-image.png}
```

## 8. 常用场景示例

### 8.1 等比缩放到版心宽度

```latex
\includegraphics[width=\textwidth]{landscape.jpg}
```

### 8.2 双图并列

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

### 8.3 从 PDF 中截取子图

```latex
\includegraphics[page=3, trim=2cm 5cm 2cm 3cm, clip, width=0.8\textwidth]{paper.pdf}
```

### 8.4 为论文排版准备的高质量插图

```latex
% 统一使用 PDF 矢量图，缩放后字体大小与正文匹配
\includegraphics[scale=0.8]{diagram.pdf}
```

## 9. 常见注意事项

- **编译报错 "Unknown graphics extension"**：说明图片格式不被当前引擎支持。把 EPS 转成 PDF 或改用 XeLaTeX/LuaLaTeX 引擎。
- **图片找不到**：确认 `\graphicspath` 路径末尾的 `/` 没有遗漏，或直接使用绝对路径调试。
- **图片模糊**：检查是否插入了低分辨率位图。对于示意图和图表，优先使用 PDF/SVG 矢量格式，缩放不损失清晰度。
- **`draft` 模式下所有图都不显示**：检查导言区 `\usepackage[draft]{graphicx}` 是否误设了全局 draft，可以通过局部 `draft=false` 覆盖。
- **裁剪不生效**：使用 `trim` 后务必加上 `clip`，否则裁剪区域仍然可见。
- **scale 与 width/height 混用**：虽然语法上合法，但 `scale` 会影响后续 `width`/`height` 的计算基准，容易造成混乱——建议一次只用一个方案。

## 10. 小结

`graphicx` 是 LaTeX 插图的基础设施，核心命令就一个 `\includegraphics`，但参数组合丰富，几乎覆盖了所有插图场景：

- **尺寸控制**：`width`、`height`、`scale`、`keepaspectratio`
- **旋转裁剪**：`angle`、`origin`、`trim`、`clip`
- **多页支持**：`page`
- **路径管理**：`\graphicspath`、`\DeclareGraphicsExtensions`
- **草稿加速**：`draft`

掌握这些用法，就能应对绝大多数 LaTeX 插图需求。如需更复杂的图文混排效果，可进一步配合 `wrapfig`（文字绕排）、`float`（浮动控制）等宏包使用。
