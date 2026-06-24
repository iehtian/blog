---
title: LaTeX \caption 与 \captionof 详解
date: 2026-06-24
updated: 2026-06-24 22:44:32
tags: [LaTeX, caption, captionof, 浮动体, 排版]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 快速对比 LaTeX 中 \caption 和 \captionof 的用法、区别与适用场景，包括浮动体限制、caption 包使用及常见注意事项。
cover: https://picsum.photos/id/25/800/450
mathjax: false
katex: false
---

# LaTeX `\caption` 与 `\captionof` 详解

## 1. 前言

在 LaTeX 中给图表加标题，最常用的就是 `\caption`。但当你在**非浮动体**环境（如 `minipage`、`center`）里也想加标题时，`\caption` 会直接报错——这时就需要 `\captionof` 出场了。本文快速对比两者的用法和适用场景。

## 2. `\caption`：浮动体专属

`\caption` 只能用在浮动体环境内部，最典型的是 `figure` 和 `table`：

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.6\textwidth]{example.png}
    \caption{这是一张示例图片}
    \label{fig:example}
\end{figure}
```

浮动体（float）的特点是 LaTeX 会**自动调整位置**以达到最佳排版效果。`\caption` 依赖浮动体提供的计数器与编号机制，离开浮动体就无法工作。

注意：`\caption` 放在 `\label` **之前**才能正确生成交叉引用编号。

## 3. `\captionof`：在非浮动体中使用标题

`\captionof` 由 `caption` 宏包提供，语法是 `\captionof{类型}{标题}`。它能让你在任意位置生成标题，无需依赖浮动体：

```latex
\usepackage{caption}   % 导言区加载

% 用法示例
\begin{center}
    \includegraphics[width=0.5\textwidth]{photo.jpg}
    \captionof{figure}{不使用 figure 浮动的图片}
    \label{fig:photo}
\end{center}
```

### 典型适用场景

- **`minipage` 内并列图表**：多个子图放在同一个 `figure` 中，每个 `minipage` 内用 `\captionof` 独立编号。

```latex
\begin{figure}[htbp]
    \centering
    \begin{minipage}{0.45\textwidth}
        \centering
        \includegraphics[width=\textwidth]{a.png}
        \captionof{figure}{子图 A}
    \end{minipage}
    \hfill
    \begin{minipage}{0.45\textwidth}
        \centering
        \includegraphics[width=\textwidth]{b.png}
        \captionof{figure}{子图 B}
    \end{minipage}
\end{figure}
```

- **不希望内容浮动**：比如某段文字中嵌入图表，你想让它**固定**在当前位置，用 `\captionof` 配合 `center` 就能实现，而 `figure` 环境会自动漂移。
- **`table` 以外的表格**：如用 `tabular` 直接写表格但不套 `table` 环境时。

提示：如果需要子图自动编号（a、b、c），推荐搭配 `subcaption` 包使用 `\captionof{subfigure}{...}`。

## 4. 对比总结

| 特性 | `\caption` | `\captionof` |
|------|-----------|-------------|
| 是否需要浮动体 | 必须（`figure`/`table`） | 不需要 |
| 是否需要 `caption` 宏包 | 不需要 | 需要 |
| 内容是否自动浮动 | 是 | 否（固定在当前位置） |
| 适用环境 | `figure`、`table` | `minipage`、`center`、任意位置 |
| 语法 | `\caption{标题}` | `\captionof{类型}{标题}` |

## 5. 常见注意事项

- **编译报错 "Not in outer par mode"**：说明你在不允许使用 `\caption` 的地方用了它，换成 `\captionof` 并加载 `caption` 宏包即可解决。
- **`\captionof` 不会创建浮动体**，所以 `[htbp]` 这样的位置参数对它没有意义——它永远停留在你放置的位置。
- **编号计数器是共享的**：`\captionof{figure}{...}` 会递增 `figure` 计数器，和 `\caption` 保持统一编号序列。
- 如果你已经在用 `subcaption` 包，它内部已加载 `caption`，无需重复 `\usepackage{caption}`。

## 6. 小结

一句话总结：**有浮动体用 `\caption`，没浮动体用 `\captionof`。** 后者让你摆脱浮动体束缚，在任意位置给图表加编号标题，代价是放弃自动浮动排版。
