---
title: LaTeX array 宏包详解
date: 2026-06-24
updated: 2026-06-25 00:17:12
tags: [LaTeX, array, 表格, 排版, 列格式]
categories: LaTeX
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 全面介绍 LaTeX array 宏包的用法，包括自定义列类型、`>` 和 `<` 修饰符、固定宽度列、表格列格式的高级定制等实用技巧。
keywords: LaTeX, array, 表格, tabular, 列格式, booktabs, 排版
cover: https://picsum.photos/id/27/800/450
mathjax: false
katex: false
---

# LaTeX `array` 宏包详解

`array` 宏包扩展了 `tabular`/`array` 环境的列格式能力：增加固定宽度列类型 `m`/`b`、列前后修饰符 `>{...}`/`<{...}`、自定义列类型 `\newcolumntype`。

## 1. 加载宏包

```latex
\usepackage{array}
```

`array` 宏包同时增强 `tabular`（文本模式）和 `array`（数学模式）两个环境，加载后两个环境都享有全部列格式功能。

## 2. 基础列类型回顾

在介绍 `array` 宏包的扩展功能之前，先回顾标准 LaTeX 已有的列类型：

| 列类型 | 说明 | 示例 |
|--------|------|------|
| `l` | 左对齐，宽度自适应 | `\begin{tabular}{l}` |
| `c` | 居中对齐，宽度自适应 | `\begin{tabular}{c}` |
| `r` | 右对齐，宽度自适应 | `\begin{tabular}{r}` |
| `p{宽度}` | 段落列，指定宽度，顶部对齐 | `\begin{tabular}{p{3cm}}` |

## 3. 新增的固定宽度列类型

### 3.1 `m{宽度}` — 垂直居中对齐

`p{宽度}` 在多行内容时默认顶部对齐。`m{宽度}` 将内容垂直居中：

```latex
\begin{tabular}{|p{2cm}|m{2cm}|}
    \hline
    多行内容很长很长 & 多行内容同样很长很长 \\
    \hline
\end{tabular}
```

- `p` 列内容顶部对齐，`m` 列内容垂直居中

### 3.2 `b{宽度}` — 底部对齐

`b{宽度}` 将内容的垂直参考线放在段落底部：

```latex
\begin{tabular}{|p{2cm}|b{2cm}|}
    \hline
    多行内容的这一列 &
    同样很长很长很长很长很长的另一列 \\
    \hline
\end{tabular}
```

### 3.3 三种固定宽度列的对比

| 列类型 | 对齐方式 | 使用场景 |
|--------|----------|----------|
| `p{宽度}` | 顶部对齐 | 表格中大多数常规段落列 |
| `m{宽度}` | 垂直居中 | 相邻列内容不对称、需要视觉平衡时 |
| `b{宽度}` | 底部对齐 | 需要将多列表格按底部对齐时 |

## 4. `>` 和 `<` 修饰符

这是 `array` 宏包最强大的特性之一：**在每个列的前后插入任意命令**。

### 4.1 语法

```latex
>{命令}    % 在列的开头（每个单元格内容之前）插入命令
<{命令}    % 在列的末尾（每个单元格内容之后）插入命令
```

### 4.2 常用场景

**场景一：将某一列设为粗体**

```latex
\begin{tabular}{>{\bfseries}l c r}
    粗体左对齐 & 居中 & 右对齐 \\
    第二行内容  & 内容 & 内容 \\
\end{tabular}
```

第一列的所有单元格自动变为粗体，无需在每个单元格中手动写 `\textbf{}`。

**场景二：将某一列设为数学模式**

```latex
\begin{tabular}{>{$}c<{$} | >{$}c<{$}}
    \alpha & \beta  \\
    \gamma & \delta \\
\end{tabular}
```

`>{$}c<{$}` 等价于自动为每个单元格包裹 `$...$`，省去反复输入数学符号的麻烦。

**场景三：将某一列设为打字机字体**

```latex
\begin{tabular}{>{\ttfamily}l l}
    代码命令    & 说明 \\
    \textbackslash includegraphics & 插图 \\
    \textbackslash caption         & 设置标题 \\
\end{tabular}
```

第一列自动使用等宽字体，非常适合排版命令说明表格。

**场景四：修改列间距**

```latex
\begin{tabular}{>{\hspace{1cm}}l l}
    数据 & 值 \\
    A    & 1  \\
    B    & 2  \\
\end{tabular}
```

第一列每个单元格内容前增加 1cm 的水平空白。

### 4.3 `<` 修饰符示例

```latex
% 在第一列末尾自动添加句点
\begin{tabular}{l<{.} r}
    苹果  & 5  \\
    香蕉  & 12 \\
    橙子  & 8  \\
\end{tabular}

% 在第三列每个单元格末尾追加 ^\circ（度符号），显示为 90°、45°
\begin{tabular}{>{$}c<{$} >{$}c<{^\circ$}}
    角度 & 值 \\
    \alpha & 90  \\
    \beta  & 45  \\
\end{tabular}
```

## 5. `\newcolumntype` — 自定义列类型

当某个列定义反复使用时，`\newcolumntype` 可以将其封装为一个有意义的名称。

### 5.1 语法

```latex
\newcolumntype{新类型名}[参数个数]{列定义}
```

- 类型名**必须是一个字母**（大小写均可）
- `[参数个数]` 可选，用于需要传参的列类型（如指定宽度）
- `[参数个数]` 的值决定了使用该列类型时需传入几个参数

### 5.2 示例

**不含参数：**

```latex
\newcolumntype{L}{>{\raggedright\arraybackslash}p{3cm}}
\newcolumntype{C}{>{\centering\arraybackslash}p{3cm}}
\newcolumntype{R}{>{\raggedleft\arraybackslash}p{3cm}}
```

这样就可以用 `L`、`C`、`R` 表示三种对齐方式的固定宽段落列：

```latex
\begin{tabular}{|L|C|R|}
    \hline
    左对齐内容 & 居中内容 & 右对齐内容 \\
    \hline
\end{tabular}
```

**含一个参数（宽度可变的段落列）：**

```latex
\newcolumntype{L}[1]{>{\raggedright\arraybackslash}p{#1}}
\newcolumntype{C}[1]{>{\centering\arraybackslash}p{#1}}
\newcolumntype{R}[1]{>{\raggedleft\arraybackslash}p{#1}}
```

使用时传入宽度参数：

```latex
\begin{tabular}{|L{3cm}|C{2cm}|R{4cm}|}
    \hline
    左对齐（3cm列宽） & 居中（2cm列宽） & 右对齐（4cm列宽） \\
    \hline
\end{tabular}
```

**含两个参数（带自定义前后修饰）：**

```latex
\newcolumntype{E}[2]{>{#1}#2<{#1}}
```

使用 `E{\bfseries}{c}` 创建一个自动加粗的居中对齐列，等价于 `>{\bfseries}c<{\bfseries}`，展示了 `\newcolumntype` 的灵活性。

### 5.3 `\arraybackslash` 的作用

在 `>{...}` 中使用 `\centering`、`\raggedright`、`\raggedleft` 等命令时，必须用 `\arraybackslash` 收尾，否则 `\\` 换行命令会被重新定义，导致表格换行失效：

```latex
% ❌ 错误：\\ 被 \centering 重新定义
\newcolumntype{C}[1]{>{\centering}p{#1}}

% ✅ 正确：\arraybackslash 恢复了 \\ 的原始定义
\newcolumntype{C}[1]{>{\centering\arraybackslash}p{#1}}
```

规则：只要 `>{...}` 里出现 `\centering`、`\raggedright` 或 `\raggedleft`，就追加 `\arraybackslash`。

## 6. 控制列间距

### 6.1 `\setlength` 调整表格列间距

默认情况下，表格单元格左右各有 `\tabcolsep`（默认 6pt）的间距：

```latex
\setlength{\tabcolsep}{10pt}   % 增加到 10pt
\setlength{\tabcolsep}{2pt}    % 减少到 2pt（紧凑表格）
```

对于 `array` 环境（数学模式），对应的长度变量是 `\arraycolsep`（默认 5pt）：

```latex
\setlength{\arraycolsep}{3pt}
```

### 6.2 取消列间距

通过 `@{}` 可以取消或替换列间距：

```latex
\begin{tabular}{@{} l @{\hspace{1cm}} r @{}}
    左对齐 & 右对齐 \\
\end{tabular}
```

- `@{}` — 取消该位置的默认间距
- `@{...}` — 用 `...` 替换默认间距

### 6.3 `!{}` 修饰符

`@{}` 会移除原有列间距。`!{}` 只**额外插入**内容，保留原有间距：

```latex
\begin{tabular}{c !{\vrule width 2pt} c}
    A & B \\
    C & D \\
\end{tabular}
```

`!{\vrule width 2pt}` 在两列之间插入一条 2pt 粗的竖线，同时保留原有的列间距。

## 7. 表格线的高级控制

`array` 宏包还增强了 `\hline` 的行为：配合 `\firsthline` 和 `\lasthline` 可以让表格首尾的横线更美观，不过这两个命令需要配合 `\usepackage{array}` 使用。

此外，`\extrarowheight` 长度变量可以在每行上方增加额外高度，避免行内容与横线贴得过紧：

```latex
\setlength{\extrarowheight}{3pt}
```

搭配 `\arrayrulewidth`（线宽）和 `\doublerulesep`（双线间距）可以实现更精细的表格排版：

```latex
\setlength{\arrayrulewidth}{1pt}      % 表格线加粗
\setlength{\doublerulesep}{2pt}        % 双线间距
\setlength{\extrarowheight}{4pt}       % 行高增加
```

注意：`\extrarowheight` 是 `array` 宏包引入的长度变量，不使用该宏包时不存在此变量。

## 8. 完整参数速查表

### 8.1 新增列类型

| 列类型 | 说明 |
|--------|------|
| `m{宽度}` | 固定宽度，垂直居中对齐 |
| `b{宽度}` | 固定宽度，底部对齐 |

### 8.2 修饰符

| 修饰符 | 语法 | 说明 |
|--------|------|------|
| `>{...}` | `>{命令}列类型` | 在列内容之前插入命令 |
| `<{...}` | `列类型<{命令}` | 在列内容之后插入命令 |
| `!{...}` | `!{内容}` | 插入内容而不移除默认列间距 |

### 8.3 新增命令

| 命令 | 说明 |
|------|------|
| `\newcolumntype{名}[n]{定义}` | 定义新的列类型，可带参数 |
| `\arraybackslash` | 恢复 `\\` 的换行功能（配合 `\centering` 等使用） |
| `\extrarowheight` | 长度变量，控制每行的额外高度 |

## 9. 进阶示例

### 9.1 学术论文三线表

```latex
\documentclass{article}
\usepackage{array}
\usepackage{booktabs}

\begin{document}
\begin{tabular}{>{\bfseries}l c c c}
    \toprule
    模型      & 准确率  & 精确率  & 召回率 \\
    \midrule
    ResNet-50 & 0.923  & 0.918  & 0.929  \\
    ViT-B/16  & 0.941  & 0.937  & 0.945  \\
    ConvNeXt  & 0.953  & 0.950  & 0.956  \\
    \bottomrule
\end{tabular}
\end{document}
```

使用 `>{bfseries}` 让第一列（模型名）自动加粗，视觉上更突出。

### 9.2 自定义货币列类型

```latex
\newcolumntype{Y}{>{$}r<{$}}       % 数学模式右对齐
\newcolumntype{Z}{>{\$}r}          % 文本模式，左侧加 $ 符号

\begin{tabular}{l Z}
    商品     & 价格 \\
    机械键盘  & 89.99 \\
    显示器    & 299.00 \\
\end{tabular}
```

`Z` 列自动在数值前添加美元符号，表格代码更加简洁。

### 9.3 带注释的代码说明表

```latex
\newcolumntype{P}{>{\ttfamily\footnotesize}l}
\newcolumntype{Q}{>{\raggedright\arraybackslash}p{6cm}}

\begin{tabular}{P Q}
    \textbackslash usepackage\{array\} & 加载 array 宏包 \\
    \textbackslash newcolumntype       & 定义新的列类型 \\
    >\{\textbackslash bfseries\}c       & 将某列设为粗体居中 \\
\end{tabular}
```

### 9.4 颜色交替行

```latex
\usepackage[table]{xcolor}
\usepackage{array}

\newcolumntype{+}{>{\columncolor{gray!20}}l}  % 灰色背景列

\begin{tabular}{+ l + l}
    姓名  & 成绩 & 姓名  & 成绩 \\
    张三  & 92   & 李四  & 88   \\
    王五  & 76   & 赵六  & 95   \\
\end{tabular}
```

## 10. booktabs 宏包 — 专业表格线

### 10.1 booktabs 概述

标准 LaTeX 的 `\hline` 横线粗细一致，竖线交叉处有缝隙。`booktabs` 提供三种粗细的专业表格线，遵循"三线表"规范：无竖线、横线有粗细层次（顶线/底线粗，中间分隔线细）、线与内容间距适当。

### 10.2 加载宏包

```latex
\usepackage{array}
\usepackage{booktabs}   % 与 array 完全兼容
```

### 10.3 核心命令

`booktabs` 用三条命令替代了传统的 `\hline`：

| 命令 | 对应传统命令 | 默认线宽 | 用途 |
|------|-------------|----------|------|
| `\toprule` | — | 0.08em | 表格顶线，最粗 |
| `\midrule` | `\hline` | 0.05em | 表头与数据之间的分隔线，较细 |
| `\bottomrule` | — | 0.08em | 表格底线，与顶线同粗 |

基本用法：

```latex
\begin{tabular}{l c c}
    \toprule
    名称 & 数值1 & 数值2 \\
    \midrule
    A    & 1     & 2     \\
    B    & 3     & 4     \\
    \bottomrule
\end{tabular}
```

对比传统 `\hline` 的效果：

- `\hline` 表格：所有横线粗细一致，顶部和底部线条与 `\midrule` 无区别，视觉上没有层次感
- `booktabs` 表格：顶线和底线明显更粗，表头和数据之间用细线分隔，读者的视线自然聚焦在表头和表尾之间

### 10.4 `\cmidrule` — 部分列横线

`\cmidrule` 是 `\cline` 的升级版，只在指定的列范围内画横线。它产生的线条更细（默认 0.03em），且两端自动略微缩短，避免相邻的 `\cmidrule` 连接在一起：

```latex
\begin{tabular}{l c c c c}
    \toprule
    \multicolumn{2}{c}{A组} & \multicolumn{2}{c}{B组} \\
    \cmidrule(lr){2-3} \cmidrule(lr){4-5}
    名称 & 数值1 & 数值2 & 数值3 & 数值4 \\
    \midrule
    项目1 & 10 & 20 & 30 & 40 \\
    项目2 & 50 & 60 & 70 & 80 \\
    \bottomrule
\end{tabular}
```

`\cmidrule` 的可选修剪参数：

| 参数 | 说明 |
|------|------|
| `(l)` | 左端缩短（默认缩短量） |
| `(r)` | 右端缩短（默认缩短量） |
| `(lr)` | 左右两端都缩短 |
| `(l{长度})` | 左端缩短指定长度，如 `(l{5pt})` |
| `(r{长度})` | 右端缩短指定长度，如 `(r{3pt})` |

缩短后同一条水平线上的多条 `\cmidrule` 之间保留微小间隙，视觉上区分不同的列分组。

### 10.5 `\addlinespace` — 增加行间距

在表格中插入额外的垂直间距，比手动在 `\\` 后加 `[间距]` 更语义化且更一致：

```latex
\begin{tabular}{l c}
    \toprule
    类别 & 数值 \\
    \midrule
    第一组数据  & 100 \\
    第二组数据  & 200 \\
    \addlinespace
    第三组数据  & 300 \\
    第四组数据  & 400 \\
    \bottomrule
\end{tabular}
```

`\addlinespace` 的默认间距为 0.5em，也可以通过可选参数自定义：

```latex
\addlinespace[3pt]   % 增加 3pt 额外间距
```

### 10.6 调整线的粗细与间距

`booktabs` 提供了五个长度变量，可在导言区或正文中调整：

| 长度变量 | 默认值 | 作用 |
|----------|--------|------|
| `\heavyrulewidth` | 0.08em | `\toprule` 和 `\bottomrule` 的线宽 |
| `\lightrulewidth` | 0.05em | `\midrule` 的线宽 |
| `\cmidrulewidth` | 0.03em | `\cmidrule` 的线宽 |
| `\aboverulesep` | 0.40ex | 横线上方与内容的间距 |
| `\belowrulesep` | 0.65ex | 横线下方与内容的间距 |

调整示例：

```latex
\setlength{\heavyrulewidth}{1.5pt}   % 加粗顶线和底线
\setlength{\lightrulewidth}{0.8pt}   % 加粗中间分隔线
\setlength{\aboverulesep}{0.5ex}     % 增加横线上方的间距
```

### 10.7 `\specialrule` — 完全自定义的横线

当 `\toprule`、`\midrule`、`\bottomrule` 的预设无法满足需求时，`\specialrule` 允许完全自定义线宽和上下间距：

```latex
\specialrule{线宽}{上方间距}{下方间距}
```

示例：

```latex
\begin{tabular}{l c}
    \toprule
    名称 & 值 \\
    \specialrule{1pt}{4pt}{6pt}  % 粗分隔线，上下留足空白
    重要数据 & 100 \\
    \bottomrule
\end{tabular}
```

适用于需要在表格中间插入一条醒目的分隔线来划分不同语义区域的场景。

### 10.8 与 array 配合：最佳实践

`booktabs` 与 `array` 宏包配合是 LaTeX 表格排版的标准组合——`array` 定制列格式，`booktabs` 呈现专业表格线：

```latex
\documentclass{article}
\usepackage{array}
\usepackage{booktabs}

\begin{document}

% array 的 >{} 修饰符 + booktabs 的专业线
\begin{tabular}{>{\bfseries}l c c c}
    \toprule
    模型      & 准确率  & 精确率  & 召回率 \\
    \midrule
    ResNet-50 & 0.923  & 0.918  & 0.929  \\
    ViT-B/16  & 0.941  & 0.937  & 0.945  \\
    ConvNeXt  & 0.953  & 0.950  & 0.956  \\
    \bottomrule
\end{tabular}

\end{document}
```

在这个例子中，`>{\bfseries}l` 让第一列自动加粗（`array` 的功能），`\toprule`/`\midrule`/`\bottomrule` 则提供了专业的三线表结构（`booktabs` 的功能）。

### 10.9 注意事项

- **不要混用竖线和 `booktabs`**：`booktabs` 的设计理念是不使用竖线。在 `{|c|c|}` 中使用 `\toprule` 虽能编译，但竖线与横线粗细反差过大，视觉效果差。
- **不要混用 `\hline` 和 `\toprule`/`\midrule`**：两种风格的横线混用会破坏表格的视觉层次感。要么全程使用 `\hline`，要么全程使用 `booktabs` 命令。
- **用 `\cmidrule` 替代 `\cline`**：加载 `booktabs` 后应优先使用 `\cmidrule`，它的默认间距和修剪参数更符合专业排版规范。
- **间距由宏包管理**：`\aboverulesep` 和 `\belowrulesep` 自动管理线与内容的间距，无需额外手动添加空白。

## 11. 常见问题与注意事项

- **`\centering` 后 `\\` 失效**：在 `>{...}` 中使用 `\centering`、`\raggedright`、`\raggedleft` 后必须追加 `\arraybackslash`，否则表格换行命令 `\\` 会被覆盖。
- **`\newcolumntype` 类型名冲突**：如果提示已定义该类型名，换一个尚未使用的字母即可。建议用大写字母命名自定义列类型，避免与标准列类型 `l`、`c`、`r`、`p` 混淆。
- **`m{宽度}` 对单行内容无效果**：如果列的文本只有一行，`p`、`m`、`b` 的垂直对齐差异看不出来——只有多行内容才能体现。
- **`<{...}` 与 `>{...}` 的顺序**：当相邻列同时使用 `A列<{...}` 和 `>{...}B列` 时，命令的执行顺序是：A列内容 → A列 `<{...}` 命令 → B列 `>{...}` 命令 → B列内容。
- **编译兼容性**：`array` 宏包与绝大多数 LaTeX 宏包兼容（`booktabs`、`longtable`、`tabularx`、`xcolor` 等），这些宏包多数会自动加载 `array`。

## 12. 小结

`array` 宏包虽然"小"，但它是 LaTeX 表格排版中不可或缺的基础组件。核心功能可以概括为三点：

1. **扩展列类型**：`m{宽度}` 和 `b{宽度}` 补充了标准 `p{宽度}` 的垂直对齐选择。
2. **列前后修饰**：`>{...}` 和 `<{...}` 让每一列可以自动应用字体、对齐、数学模式等格式，大幅减少重复代码。
3. **自定义列类型**：`\newcolumntype` 将常用列定义封装为可复用类型，让表格源码更简洁、更易维护。

掌握 `array` 宏包后，再配合 `booktabs`（专业表格线）、`tabularx`（自动计算列宽）、`multirow` / `multicolumn`（单元格合并）等宏包，就能排出专业美观的 LaTeX 表格。
