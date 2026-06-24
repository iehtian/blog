---
title: LaTeX parskip 宏包详解
date: 2026-06-24
updated: 2026-06-25 00:17:12
tags: [LaTeX, parskip, 段落间距, 排版, 段落缩进]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 全面介绍 LaTeX parskip 宏包的用法，包括段落间距与缩进的控制、宏包选项详解、与 KOMA-Script 的配合以及常见排版陷阱。
keywords: LaTeX, parskip, 段落间距, 排版, 缩进, KOMA-Script
cover: https://picsum.photos/id/28/800/450
mathjax: false
katex: false
---

# LaTeX `parskip` 宏包详解

`parskip` 宏包一键将 LaTeX 默认的"首行缩进"切换为"段间距分隔"模式，无需手动调整 `\parskip` 和 `\parindent`。适用于技术文档、信件、简历等场景。

## 1. 加载宏包

```latex
\usepackage{parskip}
```

加载后段落排版从"首行缩进"切换为"段间距分隔"，同时取消首行缩进。

## 2. 宏包选项详解

### 2.1 `skip` — 设置段间距大小

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

### 2.2 `indent` — 保留首行缩进

默认取消首行缩进。如需**同时保留段间距和首行缩进**：

```latex
\usepackage[indent]{parskip}
```

注意："段间距"和"首行缩进"通常二选一，同时使用会导致段落分隔过度。

### 2.3 `toc` — 控制目录中的段间距

```latex
\usepackage[toc]{parskip}    % 目录中也应用段间距（默认）
\usepackage[notoc]{parskip}  % 目录中不应用段间距
```

推荐 `notoc` 保持目录紧凑：`\usepackage[skip=small,notoc]{parskip}`

### 2.4 `parfill` — 段落末行填充

```latex
\usepackage[parfill]{parskip}    % 段落末行用空白填充（默认）
\usepackage[noparfill]{parskip}  % 段落末行不填充
```

- `parfill` 填充 `\parfillskip`，保证段落间距固定（非弹性）

### 2.5 选项组合示例

```latex
% 典型配置：小段间距，目录不受影响
\usepackage[skip=small,notoc]{parskip}

% 完全自定义
\usepackage[skip=8pt,indent,notoc,noparfill]{parskip}
```

## 3. 手动设置 `\parskip` 与 `\parindent`

也可手动设置两个 TeX 长度变量：

```latex
\setlength{\parskip}{0.5\baselineskip plus 2pt}  % 段间距
\setlength{\parindent}{0pt}                        % 取消首行缩进
```

### 3.1 为什么推荐使用 `parskip` 宏包

手动设置只改了 `\parskip` 和 `\parindent`。`parskip` 宏包还会调整 `\parfillskip`、列表间距、章节标题前后间距等，确保全局统一。手动设置后 `\tableofcontents` 条目间距可能过大，`parskip` 的 `notoc` 选项可直接处理。

### 3.2 `\parskip` 的弹性长度

`\parskip` 是一个**弹性长度**（rubber length），包含 `plus` 和 `minus` 分量：

```latex
\setlength{\parskip}{6pt plus 2pt minus 1pt}
```

- `6pt` — 自然长度
- `plus 2pt` — 最大可拉伸 2pt
- `minus 1pt` — 最大可收缩 1pt

弹性长度让 TeX 在分页时有一定调整空间，避免页面底部出现不美观的空白。`parskip` 宏包会自动设置合理的弹性值。

## 4. 与 KOMA-Script 的配合

KOMA-Script 文档类（`scrartcl`、`scrreprt`、`scrbook`）内置了 `parskip` 选项，无需加载宏包：

```latex
\documentclass[parskip=half]{scrartcl}   % 段间距为半个行距
\documentclass[parskip=full]{scrartcl}   % 段间距为整个行距
\documentclass[parskip=half-]{scrartcl}  % 段间距为半个行距（末尾无空格）
```

### 4.1 KOMA-Script parskip 选项速查

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

`+`/`-` 后缀对应 `parskip` 宏包的 `parfill`/`noparfill` 选项。

## 5. 段落间距在不同环境中的表现

### 5.1 列表环境

`parskip` 自动调整 `itemize`、`enumerate`、`description` 等列表环境的间距，与段落间距保持一致：

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

### 5.2 章节标题

`parskip` 调整章节标题（`\section`、`\subsection` 等）前后间距，防止标题与正文之间的空白比例失调。

### 5.3 脚注

脚注中的段落通常较短且密集，`parskip` 对其影响较小。如果你对脚注排版有特殊要求，可以使用 `footmisc` 宏包进一步定制。

## 6. 常见问题与注意事项

### 6.1 `\paragraph{}` 标题后的间距

`\paragraph{}` 是 LaTeX 中最低级别的章节标题命令。使用 `parskip` 后，`\paragraph{标题}` 后面的文本可能与标题出现在同一行而非另起一段。解决方法：在 `\paragraph{标题}` 后加空白行，或使用 `\leavevmode`：

```latex
\paragraph{小标题}

段落内容从这里开始。
```

### 6.2 与 `enumitem` 宏包的兼容性

`parskip` 与 `enumitem`（自定义列表环境）兼容。加载顺序：先 `parskip`，后 `enumitem`，后者可在前者基础上进一步定制列表间距：

```latex
\usepackage{parskip}
\usepackage{enumitem}
```

`parskip` 不影响列表时，使用 `enumitem` 的 `nosep` 选项：

```latex
\begin{itemize}[nosep]
    \item 紧凑列表
    \item 不受 parskip 影响
\end{itemize}
```

### 6.3 书信类文档

书信类文档（`letter` 类或 `scrlttr2`）段间距模式**几乎是必选的**——书信排版中极少使用首行缩进。

```latex
\documentclass{letter}
\usepackage[skip=small]{parskip}
```

### 6.4 双栏排版

双栏排版（`twocolumn`）行宽较窄，段间距不宜过大，使用 `skip=small`：

```latex
\documentclass[twocolumn]{article}
\usepackage[skip=small]{parskip}
```

## 7. 小结

`parskip` 核心功能：**把段落排版从"首行缩进"切换为"段间距分隔"**，同时处理目录、列表、弹性长度等细节。

- 加载 `\usepackage{parskip}` 即可切换段落排版风格
- 使用 `skip=small/medium/large` 控制段间距大小
- 使用 `notoc` 保持目录紧凑
- KOMA-Script 用户直接用文档类的 `parskip=half/full` 选项
- 不要混用 `parskip` 和手动 `\setlength`，选一种方式即可
