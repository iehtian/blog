---
title: LaTeX \label 命令详解
date: 2026-06-24
updated: 2026-06-25 00:17:12
tags: [LaTeX, label, ref, 交叉引用, 浮动体, 编号]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 深入讲解 LaTeX \label 命令的工作原理、正确放置位置以及常见错误，包括与 \caption 的顺序关系、计数器机制和 \ref / \pageref 的使用方法。
keywords: LaTeX, label, ref, 交叉引用, 浮动体, 编号, caption
cover: https://picsum.photos/id/56/800/450
mathjax: false
katex: false
---

# LaTeX `\label` 命令详解

`\label{key}` 记录**在此之前最后一个被递增的计数器**及其当前页码，供 `\ref` 和 `\pageref` 使用。放错位置（如放到 `\caption` 之前），引用编号即指向错误对象。

## 1. `\label` 的工作原理

`\label{key}` 不生成可见文字，只做两件事：

1. 记录**当前页码**（供 `\pageref` 使用）
2. 记录**最近一次被递增的计数器及其当前值**（供 `\ref` 使用）

`\label` 只向后看——**它总是引用在此之前编号过的对象**。

```latex
\section{引言}
\label{sec:intro}   % 记录 section 计数器的当前值
```

- `\section{引言}` 递增 section 计数器 → `\label{sec:intro}` 快照该值
- `\label` 本身不触发计数器递增，仅记录快照

## 2. 正确放置 `\label` 的位置

### 2.1 章节标题

对于 `\section`、`\subsection` 等，`\label` 放在标题命令之后即可：

```latex
\section{实验方法}
\label{sec:method}

% 或在参数内部
\section{实验方法\label{sec:method}}
```

注意：不要放在 `\section` 之前——那会记录到上一个编号对象：

```latex
\label{sec:method}   % 错误！记录的是前一个编号
\section{实验方法}
```

### 2.2 公式

在公式环境中，`\label` 放在公式内部即可：

```latex
\begin{equation}
    E = mc^2
    \label{eq:einstein}
\end{equation}
```

对于 `align`、`gather` 等多行公式环境，每条 `\\` 之前的行都可以有自己的 `\label`：

```latex
\begin{align}
    a &= b + c \label{eq:step1} \\
    d &= e + f \label{eq:step2}
\end{align}
```

### 2.3 列表项

在 `enumerate` 列表中使用 `\label`，必须放在 `\item` 之后：

```latex
\begin{enumerate}
    \item 第一步 \label{item:step1}
    \item 第二步 \label{item:step2}
\end{enumerate}
```

## 3. 浮动体中的 `\label`

浮动体（`figure`、`table`）中使用 `\label` 是最容易出错的地方。核心规则只有一条：

> **`\label` 必须跟在 `\caption` 命令后面，并且两者都要处在浮动体环境内部。**

### 3.1 正确写法

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

### 3.2 为什么必须放在 `\caption` 之后

`\label` 记录最近被递增的计数器。`\caption` 递增 `figure`/`table` 计数器——`\label` 只有在其之后才能拿到正确的编号值。

`\label` 放在 `\caption` 之前 → 记录的是上一个对象的编号：

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.6\textwidth]{example.png}
    \label{fig:example}   % 错误！figure 计数器尚未递增
    \caption{示例图片}
\end{figure}
```

### 3.3 为什么 `\label` 必须在浮动体内部

浮动体会自动漂移。`\label` 放在浮动体外部意味着它的最终页码与 `\caption` 不同，`\pageref` 返回错误页码：

```latex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=0.6\textwidth]{example.png}
    \caption{示例图片}
\end{figure}
\label{fig:example}   % 错误！在浮动体外面
```

规则：**`\label` 必须与目标对象处于同一环境，紧跟 `\caption` 之后。**

## 4. `\label` 与 `\captionof`

`\captionof` 同样递增对应计数器，`\label` 规则不变——紧跟 `\captionof` 之后：

```latex
\begin{center}
    \includegraphics[width=0.5\textwidth]{photo.jpg}
    \captionof{figure}{不使用浮动的图片}
    \label{fig:photo}   % ← 正确：在 \captionof 之后
\end{center}
```

## 5. 交叉引用：`\ref` 与 `\pageref`

`\label` 记录的信息需要通过 `\ref` 和 `\pageref` 来读取：

| 命令 | 作用 | 示例输出 |
|------|------|---------|
| `\ref{key}` | 返回计数器编号 | "3.2"、"1"、"I"（罗马数字） |
| `\pageref{key}` | 返回对象所在页码 | "24"、"156" |
| `\nameref{key}` | 返回对象标题文字 | "实验方法" |

`\nameref` 需要 `hyperref` 宏包。编号中包含章节号（如 "图 3.2"）需在导言区配置 `\numberwithin{figure}{section}`。

引用示例：

```latex
如图 \ref{fig:example} 所示，详见第 \pageref{fig:example} 页。
% 编译输出：如图 1 所示，详见第 3 页。
```

## 6. 常见错误与排查

| 错误 | 原因 | 解决方法 |
|------|------|---------|
| 引用编号全为 0 或不正确 | `\label` 放在了 `\caption` 之前 | 将 `\label` 移到 `\caption` 之后 |
| 引用编号指向了错误的章/图 | `\label` 放在了编号命令（`\section`、`\caption`）之前 | 确保 `\label` 紧跟目标对象之后 |
| `\pageref` 返回的页码不对 | `\label` 放在了浮动体外部 | 将 `\label` 移入浮动体内部 |
| 编译出现 "undefined reference" 警告 | 首次编译时 `.aux` 文件尚未生成 | 再编译一次即可，连续编译两次是正常的 LaTeX 工作流 |
| `??` 出现在输出中 | `\label` 的 key 拼写错误或未定义 | 检查 `\label{key}` 和 `\ref{key}` 的 key 是否一致 |

## 7. 小结

`\label` 的正确用法浓缩为三条规则：

1. **`\label` 总是记录在此之前最后一个编号对象**——它只向后看，不向前看
2. **在章节、公式、列表等非浮动环境中**，`\label` 紧跟目标命令之后即可
3. **在浮动体（`figure`、`table`）中**，`\label` 必须放在 `\caption` 之后、浮动体环境内部——或者直接写在 `\caption` 参数里

记住这三条，`\label` 的交叉引用就不会再出错了。
