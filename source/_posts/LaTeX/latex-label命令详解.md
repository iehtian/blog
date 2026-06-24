---
title: LaTeX \label 命令详解
date: 2026-06-24
updated: 2026-06-24 23:20:55
tags: [LaTeX, label, ref, 交叉引用, 浮动体, 编号]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 深入讲解 LaTeX \label 命令的工作原理、正确放置位置以及常见错误，包括与 \caption 的顺序关系、计数器机制和 \ref / \pageref 的使用方法。
cover: https://picsum.photos/id/26/800/450
mathjax: false
katex: false
---

# LaTeX `\label` 命令详解

## 1. 前言

`\label` 是 LaTeX 交叉引用系统的核心命令，它和 `\ref`、`\pageref` 配合使用，能让你在文档任意位置引用章节、公式、图表、列表等编号对象。但 `\label` 有一个很容易踩坑的特性：**它记录的永远是"在此之前的最后一个编号对象"**。放错位置，引用编号就会指向错误的对象。本文深入讲解 `\label` 的工作原理和正确用法。

## 2. `\label` 的工作原理

`\label{key}` 被调用时，并不生成任何可见文字，而是做两件事：

1. 记录**当前页码**（供 `\pageref` 使用）
2. 记录**最近一次被递增的计数器及其当前值**（供 `\ref` 使用）

关键就在这里：`\label` 不关心它后面有什么——它只关心它前面最近发生了什么编号动作。换句话说，**`\label` 总是引用在此之前编号过的一个对象**。

```latex
\section{引言}
\label{sec:intro}   % ← 记录的是 section 计数器的当前值
```

上面代码中，`\section{引言}` 先将 `section` 计数器递增为 1，紧接着 `\label{sec:intro}` 记录下"section 计数器 = 1"。之后在文档任何位置用 `\ref{sec:intro}` 就能得到 "1"。

注意：`\label` 本身不会触发任何计数器的递增，它只是"快照"当前计数器的值。

## 3. 正确放置 `\label` 的位置

### 3.1 章节标题

对于 `\section`、`\subsection` 等章节命令，`\label` 放在标题命令之后即可：

```latex
\section{实验方法}
\label{sec:method}
```

或者在标题参数内部：

```latex
\section{实验方法\label{sec:method}}
```

两种写法效果相同，都能正确记录章节编号。但注意不要放在 `\section` 之前，否则会记录到上一节（或 0，如果还没有任何章节）：

```latex
\label{sec:method}   % 错误！记录的是前一个章节的编号
\section{实验方法}
```

### 3.2 公式

在公式环境中，`\label` 放在公式内部即可：

```latex
\begin{equation}
    E = mc^2
    \label{eq:einstein}
\end{equation}
```

对于 `align`、`gather` 等多行公式环境，`\label` 可以放在任意一行（通常放在你想引用的那一行），每条 `\\` 之前的行都可以有自己的 `\label`：

```latex
\begin{align}
    a &= b + c \label{eq:step1} \\
    d &= e + f \label{eq:step2}
\end{align}
```

### 3.3 列表项

在 `enumerate` 列表中使用 `\label`，必须放在 `\item` 之后：

```latex
\begin{enumerate}
    \item 第一步 \label{item:step1}
    \item 第二步 \label{item:step2}
\end{enumerate}
```

## 4. 浮动体中的 `\label`：最重要的规则

浮动体（`figure`、`table`）中使用 `\label` 是最容易出错的地方。核心规则只有一条：

> **`\label` 必须跟在 `\caption` 命令后面，并且两者都要处在浮动体环境内部。**

### 4.1 正确写法

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.6\textwidth]{example.png}
    \caption{示例图片}
    \label{fig:example}   % ← 在 \caption 之后，浮动体内部
\end{figure}
```

或者把 `\label` 放进 `\caption` 的参数里：

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.6\textwidth]{example.png}
    \caption{示例图片\label{fig:example}}   % ← 写在 \caption 里面也可以
\end{figure}
```

### 4.2 为什么必须放在 `\caption` 之后？

来回顾一下 `\label` 的工作原理：它记录的是**最近被递增的计数器**。在 `figure` 环境中：

- `\caption` 会将 `figure` 计数器递增 1，并生成 "图 1: …" 这样的标题
- 紧接其后的 `\label` 记录到 `figure` 计数器的最新值

如果 `\label` 放在 `\caption` **之前**，会发生什么？

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.6\textwidth]{example.png}
    \label{fig:example}   % 错误！此时 figure 计数器还没递增
    \caption{示例图片}
\end{figure}
```

这时 `\label` 记录的可能是上一个 `figure` 的编号（或者更糟，是上一个被编号的任意对象）。用 `\ref` 引用时就会得到错误的编号。

### 4.3 为什么 `\label` 必须在浮动体内部？

如果把 `\label` 放在浮动体**外部**：

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.6\textwidth]{example.png}
    \caption{示例图片}
\end{figure}
\label{fig:example}   % 错误！在浮动体外面
```

由于浮动体会自动漂移（浮动到页面的顶部、底部或单独一页），而 `\label` 留在了源代码中的原始位置。当 LaTeX 最终排版时，`\label` 所处的页码和 `\caption` 所处的页码可能完全不同——`\pageref` 会返回错误的页码，`\ref` 也可能受到浮动体排序的影响产生意外结果。

一句话记住：`\label` 放在它想引用的对象的"下一个时刻"、处于同一环境中。

## 5. `\label` 与 `\captionof`

当使用 `caption` 宏包的 `\captionof` 命令在非浮动体环境中生成标题时，`\label` 的规则不变——仍然跟在 `\captionof` 后面：

```latex
\begin{center}
    \includegraphics[width=0.5\textwidth]{photo.jpg}
    \captionof{figure}{不使用浮动的图片}
    \label{fig:photo}   % ← 正确：在 \captionof 之后
\end{center}
```

`\captionof` 同样会递增对应类型的计数器（这里是 `figure`），因此 `\label` 的位置逻辑和浮动体中的规则完全一样。

## 6. 交叉引用：`\ref` 与 `\pageref`

`\label` 记录的信息需要通过 `\ref` 和 `\pageref` 来读取：

| 命令 | 作用 | 示例输出 |
|------|------|---------|
| `\ref{key}` | 返回计数器编号 | "3.2"、"1"、"I"（罗马数字） |
| `\pageref{key}` | 返回对象所在页码 | "24"、"156" |
| `\nameref{key}` | 返回对象标题文字 | "实验方法" |

`\nameref` 需要使用 `hyperref` 宏包。另外，为了让编号中包含章节号（如 "图 3.2" 而非 "图 2"），你需要在导言区配置计数器的编号格式，例如 `\numberwithin{figure}{section}`。

引用示例：

```latex
如图 \ref{fig:example} 所示，详见第 \pageref{fig:example} 页。
% 编译输出：如图 1 所示，详见第 3 页。
```

## 7. 常见错误与排查

| 错误 | 原因 | 解决方法 |
|------|------|---------|
| 引用编号全为 0 或不正确 | `\label` 放在了 `\caption` 之前 | 将 `\label` 移到 `\caption` 之后 |
| 引用编号指向了错误的章/图 | `\label` 放在了编号命令（`\section`、`\caption`）之前 | 确保 `\label` 紧跟目标对象之后 |
| `\pageref` 返回的页码不对 | `\label` 放在了浮动体外部 | 将 `\label` 移入浮动体内部 |
| 编译出现 "undefined reference" 警告 | 首次编译时 `.aux` 文件尚未生成 | 再编译一次即可，连续编译两次是正常的 LaTeX 工作流 |
| `??` 出现在输出中 | `\label` 的 key 拼写错误或未定义 | 检查 `\label{key}` 和 `\ref{key}` 的 key 是否一致 |

## 8. 小结

`\label` 的正确用法可以浓缩为三条规则：

1. **`\label` 总是记录在此之前最后一个编号对象**——它只向后看，不向前看
2. **在章节、公式、列表等非浮动环境中**，`\label` 紧跟目标命令之后即可
3. **在浮动体（`figure`、`table`）中**，`\label` 必须放在 `\caption` 之后、浮动体环境内部——或者直接写在 `\caption` 参数里

记住这三条，`\label` 的交叉引用就不会再出错了。
