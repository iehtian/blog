---
title: LaTeX parskip 宏包详解
date: 2026-06-24
updated: 2026-06-24 23:20:55
tags: [LaTeX, parskip, 段落间距, 排版, 段落缩进]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 全面介绍 LaTeX parskip 宏包的用法，包括段落间距与缩进的控制、宏包选项详解、与 KOMA-Script 的配合以及常见排版陷阱。
cover: https://picsum.photos/id/28/800/450
mathjax: false
katex: false
---

# LaTeX `parskip` 宏包详解

## 1. 前言

LaTeX 默认的段落排版风格是**首行缩进**——每段开头缩进 `\parindent`（默认 15pt 左右），段落之间没有额外间距。这种风格源于传统英文书籍排版，适合密集阅读的印刷文本。

但在很多场景下，你可能更希望段落之间**用空白分隔而非首行缩进**，例如：

- 技术文档、用户手册
- 信件、简历
- 幻灯片（beamer）
- 网页风格的内容排版

`parskip` 宏包就是专门用来处理段落间距与缩进关系的工具——加载它就能一键切换为"段间距分隔"模式，无需手动调整 `\parskip` 和 `\parindent`。

## 2. 加载宏包

在导言区通过 `\usepackage` 加载：

```latex
\usepackage{parskip}
```

仅此一行，LaTeX 的段落排版风格就会从"首行缩进"切换为"段间距分隔"——段落之间自动插入一段垂直空白，同时取消首行缩进。

## 3. 宏包选项详解

`parskip` 提供了多个选项来控制段落间距的行为细节。

### 3.1 `skip` — 设置段间距大小

控制段落之间的垂直间距量：

```latex
\usepackage[skip=small]{parskip}    % 小段间距
\usepackage[skip=medium]{parskip}   % 中等段间距（默认）
\usepackage[skip=large]{parskip}    % 大段间距
```

| 选项值 | 效果 | 适用场景 |
|--------|------|----------|
| `small` | 段间距约为 0.25\baselineskip + 2pt | 紧凑排版，技术文档 |
| `medium` | 段间距约为 0.5\baselineskip + 2pt（默认） | 大多数场景 |
| `large` | 段间距约为 1.0\baselineskip + 2pt | 需要明显分隔的文档 |

也可以使用绝对长度：

```latex
\usepackage[skip=10pt]{parskip}
```

### 3.2 `indent` — 保留首行缩进

默认情况下 `parskip` 会将首行缩进设为零。如果你希望**同时保留段间距和首行缩进**，使用 `indent` 选项：

```latex
\usepackage[indent]{parskip}
```

注意：这种风格比较少见，通常"段间距"和"首行缩进"二选一即可。同时使用可能让段落之间的视觉分隔变得过度。

### 3.3 `toc` — 控制目录中的段间距

`parskip` 会影响目录（Table of Contents）中条目的间距。如果你不希望目录受影响：

```latex
\usepackage[toc]{parskip}    % 目录中也应用段间距（默认）
\usepackage[notoc]{parskip}  % 目录中不应用段间距
```

通常建议使用 `notoc` 来保持目录的紧凑排版：

```latex
\usepackage[skip=small,notoc]{parskip}
```

### 3.4 `parfill` — 段落末行填充

控制段落末行的填充行为：

```latex
\usepackage[parfill]{parskip}    % 段落末行用空白填充（默认）
\usepackage[noparfill]{parskip}  % 段落末行不填充
```

启用 `parfill` 后，段落末行后面会填充 `\parfillskip`，确保段落间距是**固定的**而非弹性的。这有助于保持一致的排版效果。

### 3.5 选项组合示例

```latex
% 典型配置：小段间距，目录不受影响
\usepackage[skip=small,notoc]{parskip}

% 完全自定义
\usepackage[skip=8pt,indent,notoc,noparfill]{parskip}
```

## 4. 手动设置 `\parskip` 与 `\parindent`

如果不使用 `parskip` 宏包，也可以手动设置两个 TeX 长度变量来实现类似效果：

```latex
\setlength{\parskip}{0.5\baselineskip plus 2pt}  % 段间距
\setlength{\parindent}{0pt}                        % 取消首行缩进
```

### 4.1 为什么推荐使用 `parskip` 宏包？

手动设置简单直接，但有以下问题：

- **只改了长度变量**：`parskip` 宏包除了设置 `\parskip` 和 `\parindent` 外，还会调整 `\parfillskip`、列表环境的间距、章节标题前后的间距等，确保全局风格统一。
- **容易遗漏细节**：例如手动设 `\parskip` 后，`\tableofcontents` 中的条目间距可能变得过大，而 `parskip` 的 `notoc` 选项可以轻松处理。
- **可移植性更好**：`parskip` 是一个标准宏包，代码意图明确，换了文档模板后效果一致。

### 4.2 `\parskip` 的弹性长度

`\parskip` 是一个**弹性长度**（rubber length），包含 `plus` 和 `minus` 分量：

```latex
\setlength{\parskip}{6pt plus 2pt minus 1pt}
```

- `6pt` — 自然长度
- `plus 2pt` — 最大可拉伸 2pt
- `minus 1pt` — 最大可收缩 1pt

弹性长度让 TeX 在分页时有一定调整空间，避免页面底部出现不美观的空白。`parskip` 宏包会自动设置合理的弹性值。

## 5. 与 KOMA-Script 的配合

KOMA-Script 文档类（`scrartcl`、`scrreprt`、`scrbook`）内置了 `parskip` 选项，无需加载宏包：

```latex
\documentclass[parskip=half]{scrartcl}   % 段间距为半个行距
\documentclass[parskip=full]{scrartcl}   % 段间距为整个行距
\documentclass[parskip=half-]{scrartcl}  % 段间距为半个行距（末尾无空格）
```

KOMA-Script 的 `parskip` 选项与 `parskip` 宏包功能类似，但由文档类原生支持，集成度更高。如果你的文档类本身就是 KOMA-Script，直接用类的选项即可，无需再加载 `parskip` 宏包。

### 5.1 KOMA-Script parskip 选项速查

| 选项 | 说明 |
|------|------|
| `parskip=no` | 不使用段间距（默认，即首行缩进模式） |
| `parskip=half` | 段间距为 0.5\baselineskip |
| `parskip=full` | 段间距为 1.0\baselineskip |
| `parskip=half+` | 同 half，段落末行拉伸填充 |
| `parskip=half-` | 同 half，段落末行不拉伸 |
| `parskip=full+` | 同 full，段落末行拉伸填充 |
| `parskip=full-` | 同 full，段落末行不拉伸 |
| `parskip=full*` | 同 full，且目录中也应用段间距 |

其中 `half` 和 `full` 后面的 `+`/`-` 后缀控制的是段落末行填充行为，与 `parskip` 宏包的 `parfill`/`noparfill` 选项对应。

## 6. 段落间距在不同环境中的表现

### 6.1 列表环境

`parskip` 会自动调整 `itemize`、`enumerate`、`description` 等列表环境的间距，使其与段落间距保持一致：

```latex
\usepackage[skip=medium]{parskip}

\begin{document}
这是一段正文。

\begin{itemize}
    \item 列表项之间的间距也会相应调整
    \item 与段间距保持视觉协调
\end{itemize}

这是另一段正文。
\end{document}
```

### 6.2 章节标题

`parskip` 会调整章节标题（`\section`、`\subsection` 等）前后的间距，防止标题与正文之间的空白比例失调。

### 6.3 脚注

脚注中的段落通常较短且密集，`parskip` 对其影响较小。如果你对脚注排版有特殊要求，可以使用 `footmisc` 宏包进一步定制。

## 7. 常见问题与注意事项

### 7.1 `\paragraph{}` 标题后的间距

`\paragraph{}` 是 LaTeX 中最低级别的章节标题命令。使用 `parskip` 后，`\paragraph{标题}` 后面的文本可能会与标题出现在同一行而非另起一段。

解决方法是在 `\paragraph{标题}` 后面加一个空白行，或者显式使用 `\leavevmode`：

```latex
\paragraph{小标题}

段落内容从这里开始。
```

### 7.2 与 `enumitem` 宏包的兼容性

`parskip` 与 `enumitem`（自定义列表环境）完全兼容，但加载顺序有讲究：

```latex
\usepackage{parskip}   % 先加载 parskip
\usepackage{enumitem}   % 后加载 enumitem
```

这样 `enumitem` 可以在 `parskip` 的基础上进一步定制列表间距。

如果希望 `parskip` 不影响列表，使用 `enumitem` 的 `nosep` 选项：

```latex
\begin{itemize}[nosep]
    \item 紧凑列表
    \item 不受 parskip 影响
\end{itemize}
```

### 7.3 书信类文档

如果你在写书信（`letter` 类或 `scrlttr2`），段间距模式**几乎是必选的**。书信排版中极少使用首行缩进。

```latex
\documentclass{letter}
\usepackage[skip=small]{parskip}
```

### 7.4 双栏排版

在双栏排版（`twocolumn`）中，段间距不宜过大，建议使用 `skip=small`：

```latex
\documentclass[twocolumn]{article}
\usepackage[skip=small]{parskip}
```

双栏本身行宽较窄，过大的段间距会让页面显得稀疏。

### 7.5 与其他间距宏包的关系

| 宏包 | 功能 | 与 parskip 的关系 |
|------|------|-------------------|
| `setspace` | 控制行间距（`\baselineskip`） | 相互独立，可同时使用 |
| `tocloft` | 自定义目录格式 | `notoc` 选项可避免冲突 |
| `enumitem` | 自定义列表间距 | 兼容，注意加载顺序 |
| `geometry` | 页面边距设置 | 相互独立 |

## 8. 排版风格选择：段间距 vs 首行缩进

| 风格 | 实现方式 | 典型用途 |
|------|----------|----------|
| 首行缩进 | LaTeX 默认 | 学术论文、小说、传统书籍 |
| 段间距分隔 | `parskip` 宏包 | 技术文档、信件、简历、网页风格 |
| 两者兼用 | `\usepackage[indent]{parskip}` | 极少使用，通常不推荐 |
| 两者都不用 | `\setlength{\parindent}{0pt}` | 极少使用，段落界限模糊 |

选择建议：

- **学术论文**：通常保留首行缩进（导师或期刊有明确要求时优先遵循）
- **技术博客/文档**：推荐使用段间距分隔，可读性更好
- **简历/CV**：段间距分隔 + `skip=small`
- **Beamer 幻灯片**：Beamer 默认已取消首行缩进，通常不需要额外处理

## 9. 完整示例

```latex
\documentclass[a4paper]{article}
\usepackage[UTF8]{ctex}           % 中文支持（如果需要）
\usepackage[skip=small,notoc]{parskip}
\usepackage{geometry}
\geometry{margin=2.5cm}

\begin{document}

\tableofcontents
\newpage

\section{引言}

这是第一段。注意段落之间用空白分隔，首行不再缩进。

这是第二段。段落之间的垂直间距由 \texttt{parskip} 宏包自动管理。

\section{方法}

\begin{itemize}
    \item 列表项之间的间距也经过调整
    \item 与正文段间距保持协调
\end{itemize}

以下是另一个段落，注意列表前后的间距也是统一的。

如果你需要临时恢复首行缩进，可以在局部作用域中手动设置：

{
\setlength{\parindent}{2em}
\setlength{\parskip}{0pt}

这段恢复了传统的首行缩进风格（局部生效）。
}

回到默认的段间距风格。

\end{document}
```

## 10. 小结

`parskip` 是一个小而精的宏包，核心功能就一句话：**把段落排版从"首行缩进"切换为"段间距分隔"**。但它同时处理了目录、列表、弹性长度等细节，让段落间距风格的切换真正做到"开箱即用"。

关键要点：

- 加载 `\usepackage{parskip}` 即可一键切换段落排版风格。
- 使用 `skip=small/medium/large` 控制段间距大小。
- 使用 `notoc` 保持目录紧凑。
- KOMA-Script 用户直接用文档类的 `parskip=half/full` 选项。
- 不要混用 `parskip` 和手动 `\setlength`，选一种方式即可。
