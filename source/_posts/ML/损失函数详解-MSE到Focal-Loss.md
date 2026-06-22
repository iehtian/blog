---
title: 损失函数——MSE 与 MAE（回归模型基础）
date: 2026-06-19
updated: 2026-06-23 01:20:21
tags: [机器学习, 损失函数, 数学, 回归]
categories: 机器学习
keywords: 损失函数,loss function,MSE,MAE,均方误差,平均绝对误差,L2 Loss,L1 Loss,回归
description: 深入讲解回归模型中两种最基础的损失函数——MSE（均方误差）和 MAE（平均绝对误差）的定义、梯度推导、几何直觉及适用场景对比。
top_img:
comments: true
cover: https://picsum.photos/id/210/800/450
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
copyright_author_href:
copyright_url:
copyright_info:
mathjax: false
katex: true
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
---

# 损失函数——MSE 与 MAE（回归模型基础）

## 1. 前言

训练模型时，我们需要一个"考官"来评判预测结果的好坏——预测值和真实值差多少？差得越多，分数越低。这个"考官"就是**损失函数**（Loss Function）。

回归问题中最基础、最常用的两个损失函数是：

- **MSE**（Mean Squared Error，均方误差）—— 对误差求平方
- **MAE**（Mean Absolute Error，平均绝对误差）—— 对误差取绝对值

它们简单，但背后藏着重要的设计思想。选 MSE 还是 MAE，会直接影响模型学到的东西。本文从公式、梯度、直觉和实战四个角度，把这两个损失函数讲透。

## 2. 基本概念

### 2.1 三个术语的区别

| 术语 | 英文 | 含义 |
| --- | --- | --- |
| 损失函数 | Loss Function | 衡量**单个样本**的预测误差 $\ell(y_i, \hat{y}_i)$ |
| 成本函数 | Cost Function | 整个数据集上损失的平均值 $\frac{1}{N}\sum \ell(y_i, \hat{y}_i)$ |
| 目标函数 | Objective Function | 成本函数 + 正则化项，即最终要最小化的东西 |

日常讨论中三者经常混用，本文也沿用这个习惯——说"损失函数"通常指成本函数。

### 2.2 符号约定

- $y_i$ —— 第 $i$ 个样本的真实值
- $\hat{y}_i$ —— 模型对第 $i$ 个样本的预测值
- $e_i = \hat{y}_i - y_i$ —— 预测误差（残差）
- $N$ —— 样本数量

### 2.3 数学符号说明

文档中用到了一些不常见于初等数学的符号，在此集中说明：

#### 基础运算

| 符号 | 名称 | 含义 | 示例 |
| --- | --- | --- | --- |
| $\sum_{i=1}^{N}$ | 求和号（Sigma） | 从 $i=1$ 加到 $i=N$ | $\sum_{i=1}^{3} x_i = x_1 + x_2 + x_3$ |
| $\prod_{i=1}^{N}$ | 求积号（Pi） | 从 $i=1$ 乘到 $i=N$ | $\prod_{i=1}^{3} x_i = x_1 \times x_2 \times x_3$ |
| $|\cdot|$ | 绝对值 | 去掉符号，非负数 | $|{-3}| = 3$ |
| $\|\cdot\|$ | 范数 | 向量"长度"的推广 | 见下方范数说明 |

#### 导数相关

| 符号 | 名称 | 含义 | 示例 |
| --- | --- | --- | --- |
| $\frac{d}{dx}$ | 普通导数 | 单变量函数的变化率 | $\frac{d}{dx}x^2 = 2x$ |
| $\frac{\partial}{\partial x}$ | 偏导数 | 多变量函数对**某一个**变量求导，其他变量视为常数 | $f(x,y)=x^2y$，则 $\frac{\partial f}{\partial x}=2xy$ |
| $\nabla f$ | 梯度（Nabla） | 所有偏导数组成的**向量** | $\nabla f = [\frac{\partial f}{\partial x_1}, \dots, \frac{\partial f}{\partial x_n}]^\mathsf{T}$ |

> **导数和偏导数的区别**：导数用于 $f(x)$ 这种只有一个变量的函数；偏导数用于 $f(x_1, x_2, \dots)$ 这种有多个变量的函数。偏导数回答的问题是"只动这一个变量，其他都不动，函数值怎么变？"

#### 线性代数

| 符号 | 名称 | 含义 |
| --- | --- | --- |
| $\mathbf{w}$ | 粗体小写 | 列向量（一列数字） |
| $X$ | 大写字母 | 矩阵（多行多列的数字表） |
| $X^\mathsf{T}$ | 转置（Transpose） | 矩阵的行列互换——$m \times n$ 变成 $n \times m$ |
| $\|\mathbf{v}\|_2$ | L2 范数 | 欧氏距离，$\|\mathbf{v}\|_2 = \sqrt{v_1^2 + v_2^2 + \dots}$ |
| $\|\mathbf{v}\|_1$ | L1 范数 | 曼哈顿距离，$\|\mathbf{v}\|_1 = \|v_1\| + \|v_2\| + \dots$ |

#### 概率与统计

| 符号 | 名称 | 含义 |
| --- | --- | --- |
| $\mathbb{E}[Y]$ | 期望（Expectation） | 随机变量 $Y$ 的加权平均/理论均值 |
| $\mathcal{N}(\mu, \sigma^2)$ | 正态分布（高斯分布） | 均值为 $\mu$、方差为 $\sigma^2$ 的钟形分布 |
| $\varepsilon \sim \mathcal{N}(0, \sigma^2)$ | "服从"分布 | 读作"$\varepsilon$ 服从均值为 0、方差为 $\sigma^2$ 的正态分布" |
| $\exp(x)$ | 指数函数 | $e^x$ 的另一种写法，$e \approx 2.71828$；当 $x$ 很复杂时写成 $\exp(x)$ 更清晰 |
| $\text{Median}(y)$ | 中位数 | 将数据从小到大排列后位于中间位置的值 |

#### 优化相关

| 符号 | 名称 | 含义 |
| --- | --- | --- |
| $\argmin_x f(x)$ | 最小值的参数 | **使** $f(x)$ 取最小值的那个 $x$，而非最小值本身 |
| $\propto$ | 正比于 | $A \propto B$ 表示 $A$ 与 $B$ 成正比（即 $A = k \cdot B$，$k$ 为常数） |
| $\mathcal{L}$ | 花体 L | 损失函数（Loss）的常用记号，读作"script L" |
| $\hat{y}$ | hat 记号 | 表示预测值/估计值，$\hat{y}$ 读作"y hat" |
| $\mathbf{w}^*$ | 星号上标 | 表示最优解，$\mathbf{w}^*$ 读作"w star" |

> **$\argmin$ 举例**：设 $f(x) = (x-3)^2 + 2$，则 $\min_x f(x) = 2$（最小值是 2），而 $\argmin_x f(x) = 3$（取到最小值时 $x$ 是 3）。一个是"最低点的高度"，一个是"最低点的位置"。

## 3. MSE —— 均方误差

### 3.1 定义与公式

MSE 将每个样本的**误差平方**后求平均：

$$
\text{MSE} = \frac{1}{N} \sum_{i=1}^{N} (\hat{y}_i - y_i)^2 = \frac{1}{N} \sum_{i=1}^{N} e_i^2
$$

也被称为 **L2 Loss**，因为它在对误差向量求 L2 范数（再平方取平均）。

### 3.2 梯度推导

MSE 对预测值 $\hat{y}_i$ 的偏导：

$$
\frac{\partial \text{MSE}}{\partial \hat{y}_i} = \frac{2}{N}(\hat{y}_i - y_i) = \frac{2}{N} e_i
$$

**梯度与误差成正比**——误差越大，梯度越大。这意味着：

- 大误差样本产生强梯度信号，模型会"优先"修正它们
- 小误差样本梯度微弱，模型对它们"不太在乎"

### 3.3 为什么用平方？——几何直觉

平方操作带来了两个关键效应：

**放大效应**：误差 > 1 时，平方后更大

| 误差 $e$ | $e^2$ |
| --- | --- |
| 0.5 | 0.25（缩得更小） |
| 1 | 1 |
| 2 | 4（放大 2 倍） |
| 10 | 100（放大 10 倍） |

**缩小效应**：误差 < 1 时，平方后更小——模型对小误差"宽容"。

这种非线性惩罚让 MSE 对**离群点极端敏感**。一个误差为 10 的样本，其对损失的贡献相当于 100 个误差为 1 的样本。

### 3.4 理论性质：MSE 的最优解是条件均值

对任意一个点 $x$，如果我们用一个常数 $c$ 去预测它的 $y$ 值，MSE 最小的 $c$ 是 $y$ 的条件均值：

$$
\argmin_c \ \mathbb{E}\big[(y - c)^2 \mid x\big] = \mathbb{E}[y \mid x]
$$

**这意味着用 MSE 训练的回归模型，其输出可以解释为"在给定 $x$ 的条件下，$y$ 的期望值"**。这个性质在统计推断中很有价值。

### 3.5 PyTorch 实现

```python
import torch
import torch.nn as nn

mse_loss = nn.MSELoss()

y_true = torch.tensor([1.0, 2.0, 3.0])
y_pred = torch.tensor([1.5, 2.5, 2.5])

loss = mse_loss(y_pred, y_true)
print(f"MSE: {loss:.4f}")
# 手算验证: ((1.5-1)² + (2.5-2)² + (2.5-3)²) / 3
#         = (0.25 + 0.25 + 0.25) / 3 = 0.25
```

### 3.6 适用场景

- 数据**干净**，异常值很少或已被清洗
- 希望**严厉惩罚大误差**（比如金融预测中，大偏差的代价确实很大）
- 需要预测值的**理论可解释性**（条件均值）

## 4. MAE —— 平均绝对误差

### 4.1 定义与公式

MAE 将每个样本的**误差绝对值**求平均：

$$
\text{MAE} = \frac{1}{N} \sum_{i=1}^{N} |\hat{y}_i - y_i| = \frac{1}{N} \sum_{i=1}^{N} |e_i|
$$

也被称为 **L1 Loss**，对应误差向量的 L1 范数。

### 4.2 梯度推导

绝对值函数的导数要分段讨论：

$$
\frac{\partial \text{MAE}}{\partial \hat{y}_i} =
\begin{cases}
-\dfrac{1}{N}, & \hat{y}_i < y_i \quad (\text{预测偏小，需要增大 } \hat{y}_i) \\[10pt]
+\dfrac{1}{N}, & \hat{y}_i > y_i \quad (\text{预测偏大，需要减小 } \hat{y}_i)
\end{cases}
$$

$\hat{y}_i = y_i$ 处不可导，实际中通常直接赋 0。

**梯度的模恒定为 $1/N$，与误差大小无关**。这点和 MSE 截然不同：

| | MSE | MAE |
| --- | --- | --- |
| 误差 = 0.1 时的梯度 | $\propto 0.1$（很小） | $\propto 1$（不变） |
| 误差 = 10 时的梯度 | $\propto 10$（很大） | $\propto 1$（不变） |

### 4.3 恒定梯度带来的影响

**优点**：离群点不会主导梯度。一个误差为 100 的异常样本，对梯度的贡献和误差为 1 的正常样本**完全一样**（梯度值相同，只是符号可能不同）。模型不会被少数极端值"带偏"。

**缺点**：训练末期，当大部分样本的误差已经很小（比如 0.01），梯度仍然是 $\pm 1/N$，不会自动减小。这可能导致模型在最优解附近**来回震荡**，难以精确收敛。相比之下，MSE 在误差趋于 0 时梯度也趋于 0，天然地"越接近越稳"。

### 4.4 理论性质：MAE 的最优解是条件中位数

用常数 $c$ 预测 $y$ 时，MAE 最小的 $c$ 是 $y$ 的条件中位数：

$$
\argmin_c \ \mathbb{E}\big[|y - c| \mid x\big] = \text{Median}(y \mid x)
$$

**用 MAE 训练的回归模型，输出的是条件中位数，而非条件均值**。中位数不受极端值影响，这从理论上解释了 MAE 的鲁棒性。

### 4.5 PyTorch 实现

```python
mae_loss = nn.L1Loss()

y_true = torch.tensor([1.0, 2.0, 3.0])
y_pred = torch.tensor([1.5, 2.5, 2.5])

loss = mae_loss(y_pred, y_true)
print(f"MAE: {loss:.4f}")
# 手算验证: (|1.5-1| + |2.5-2| + |2.5-3|) / 3
#         = (0.5 + 0.5 + 0.5) / 3 = 0.5
```

### 4.6 适用场景

- 数据中有**较多异常值**，且不能或不想清洗
- 希望所有样本被**平等对待**（不论误差大小）
- 追求**鲁棒性**胜过精确收敛

## 5. 如何最小化损失函数

定义了损失函数之后，下一步就是找到让损失最小化的模型参数。以线性回归 $\hat{y} = X\mathbf{w} + b$ 为例（将 $b$ 吸收进 $\mathbf{w}$，$X$ 右侧补一列全 1），目标是最小化 MSE。

求解方法分为两派：**最小二乘法**给出了 MSE 的解析解（正规方程），源自高斯 200 多年前的工作；**梯度下降**则是一种通用的迭代逼近方案，适用于任何可导的损失函数。

### 5.1 最小二乘法与正规方程

**最小二乘法**（Least Squares）定义了一个优化问题——最小化残差平方和。对线性模型而言，这个优化问题的最优解可以通过**正规方程**（Normal Equation）直接求出。两者的关系是：最小二乘法提出目标函数，正规方程是该目标函数梯度为零的一阶必要条件，解之即得闭式解。

#### 历史背景

1801 年，天文学家用望远镜发现了谷神星，但只有短短几周的观测数据，之后它就消失在太阳光中。如何从有限、含噪声的数据推算轨道，重新找到它？

24 岁的高斯（Carl Friedrich Gauss）接下了这个挑战。他的假设很简洁：**使误差平方和最小的轨道参数，是最可信的**。他算出了轨道——天文学家按他算的位置去观测，果然在那里。高斯后来在 1809 年出版的《天体运动论》中系统阐述了这一方法，并给出了概率论基础。勒让德（Legendre）在 1805 年也独立发表了同一方法。

#### 数学推导——从"最小二乘"到"正规方程"

整个过程分两步走，一步对应一个概念。

**第一步：定义最小二乘目标**——这是"最小二乘法"本身。

我们要找到参数 $\mathbf{w}$，使残差平方和最小：

$$
\hat{\mathbf{w}} = \argmin_{\mathbf{w}} \ \underbrace{\|X\mathbf{w} - \mathbf{y}\|_2^2}_{\text{残差平方和}} = \argmin_{\mathbf{w}} \ \sum_{i=1}^{N} (X_i\mathbf{w} - y_i)^2
$$

这个 $\argmin$ 表达式就是**最小二乘法的全部内容**——它的目标就是让红色括号里的东西尽可能小。注意，这里最小化的是残差平方和，和 MSE（残差平方的均值）只差一个常数 $1/N$，最优解完全相同。为推导方便，令 $S(\mathbf{w}) = (X\mathbf{w} - \mathbf{y})^\mathsf{T}(X\mathbf{w} - \mathbf{y})$，展开：

$$
S(\mathbf{w}) = \mathbf{w}^\mathsf{T} X^\mathsf{T} X \mathbf{w} - 2\mathbf{y}^\mathsf{T} X \mathbf{w} + \mathbf{y}^\mathsf{T} \mathbf{y}
$$

**第二步：求导得到正规方程**——这是"怎么解"。

对 $\mathbf{w}$ 求导（用到矩阵求导公式 $\frac{\partial}{\partial \mathbf{w}} \mathbf{w}^\mathsf{T} A \mathbf{w} = 2A\mathbf{w}$，其中 $A$ 为对称矩阵）：

$$
\frac{\partial S}{\partial \mathbf{w}} = 2X^\mathsf{T} X \mathbf{w} - 2X^\mathsf{T} \mathbf{y}
$$

令梯度为零：

$$
2X^\mathsf{T} X \mathbf{w} - 2X^\mathsf{T} \mathbf{y} = 0
$$

**为什么梯度为零就能保证是最小值？** 因为 $S(\mathbf{w})$ 是 $\mathbf{w}$ 的二次型，其 Hessian 矩阵（二阶导）为：

$$
\frac{\partial^2 S}{\partial \mathbf{w}^2} = 2X^\mathsf{T} X
$$

$X^\mathsf{T} X$ 是半正定矩阵（对任意向量 $\mathbf{v}$，$\mathbf{v}^\mathsf{T} X^\mathsf{T} X \mathbf{v} = \|X\mathbf{v}\|^2 \ge 0$），因此 Hessian 处处半正定，$S(\mathbf{w})$ 是**凸函数**。对于凸函数，梯度为零的点就是全局最小值——不存在"找到的是极大值还是极小值"的歧义。当 $X^\mathsf{T} X$ 满秩（即 $X$ 的列线性无关）时，Hessian 正定，最小值唯一。

移项得到**正规方程**（Normal Equations）：

$$
\boxed{X^\mathsf{T} X \mathbf{w} = X^\mathsf{T} \mathbf{y}}
$$

**这就是两者的关系**：把最小二乘法的目标（$S(\mathbf{w})$）对参数求导并令为零，结果就是正规方程。当 $X^\mathsf{T} X$ 可逆时，解出来就是：

$$
\boxed{\mathbf{w}^* = (X^\mathsf{T} X)^{-1} X^\mathsf{T} \mathbf{y}}
$$

这就是 MSE 下线性回归的**闭式解**（closed-form solution）——不需要迭代，一步算出来。

> **总结**：最小二乘法 = 优化问题（$\argmin \|X\mathbf{w} - \mathbf{y}\|^2$）→ 求导令为零 → 正规方程（$X^\mathsf{T} X \mathbf{w} = X^\mathsf{T} \mathbf{y}$）→ 解方程得闭式解。四个箭头串起了一条完整的逻辑链。

#### 路径二：几何推导——无需微积分，用投影直接写出正规方程

如果你不想碰微积分，还有一条更直观的路——**几何**。而且它完全绕过了"最小化误差平方和"这个目标，直接从空间关系得到完全相同的方程。

核心洞察：我们希望 $\hat{\mathbf{y}} = X\mathbf{w}$ 尽可能接近 $\mathbf{y}$。在几何上，"最接近"意味着 $\hat{\mathbf{y}}$ 是 $\mathbf{y}$ 在 $X$ 列空间上的**正交投影**。

- 数据矩阵 $X$ 的每一列是一个 $N$ 维向量，它们张成**列空间**（column space）
- 标签向量 $\mathbf{y}$ 通常**不在**列空间内（否则存在完美拟合）
- 要找的 $\hat{\mathbf{y}} = X\hat{\mathbf{w}}$ 就是 $\mathbf{y}$ 在列空间上的投影点——它是列空间中离 $\mathbf{y}$ 最近的点

正交投影的关键性质：**残差向量垂直于列空间中的所有向量**。列空间由 $X$ 的各列张成，所以残差与 $X$ 的每一列都正交：

$$
X^\mathsf{T}(\mathbf{y} - X\hat{\mathbf{w}}) = 0
$$

展开：

$$
X^\mathsf{T} X \hat{\mathbf{w}} = X^\mathsf{T} \mathbf{y}
$$

**这就是正规方程。** 没有求导，没有令梯度为零——只用了"垂直"这一个几何条件。

投影点可写为 $\hat{\mathbf{y}} = X(X^\mathsf{T} X)^{-1} X^\mathsf{T} \mathbf{y} = P\mathbf{y}$，其中 $P = X(X^\mathsf{T} X)^{-1} X^\mathsf{T}$ 是投影矩阵（帽子矩阵）。当 $X^\mathsf{T} X$ 可逆时，$\hat{\mathbf{w}} = (X^\mathsf{T} X)^{-1} X^\mathsf{T} \mathbf{y}$。

这条几何直觉还有一个额外好处：它直观解释了过拟合——特征越多，列空间维度越高，$\mathbf{y}$ 自然被投影得"更近"，训练误差必然下降，但泛化未必更好。

#### 路径三：概率推导——假设高斯噪声，极大似然自然导出平方误差

第三条路从**数据是怎么生成的**出发。不直接定义损失函数，而是假设一个噪声模型，然后问"什么样的参数最可能产生我们观测到的数据"。

假设观测值等于真实值加独立高斯噪声：

$$
y_i = X_i\mathbf{w} + \varepsilon_i, \quad \varepsilon_i \sim \mathcal{N}(0, \sigma^2)
$$

在给定参数 $\mathbf{w}$ 下，观测到全部数据 $\mathbf{y}$ 的**似然**（likelihood）为：

$$
p(\mathbf{y} \mid X, \mathbf{w}) = \prod_{i=1}^{N} \frac{1}{\sqrt{2\pi\sigma^2}} \exp\!\left(-\frac{(y_i - X_i\mathbf{w})^2}{2\sigma^2}\right)
$$

取负对数似然（negative log-likelihood），把乘积变成求和：

$$
-\log p(\mathbf{y} \mid X, \mathbf{w}) = \frac{1}{2\sigma^2} \sum_{i=1}^{N} (y_i - X_i\mathbf{w})^2 + \text{常数}
$$

**极大似然估计**（MLE）要求最小化负对数似然——也就是最小化 $\sum (y_i - X_i\mathbf{w})^2$。对 $\mathbf{w}$ 求导令为零，同样得到：

$$
X^\mathsf{T} X \mathbf{w} = X^\mathsf{T} \mathbf{y}
$$

这条路径的意义在于：它告诉你**"为什么用平方误差"不是一个随意选择**——如果你的噪声近似服从高斯分布，平方误差就是统计上最优的。反过来说，MAE 不能用正规方程求解，是因为 MAE 对应的噪声模型是拉普拉斯分布 $\varepsilon \sim \text{Laplace}(0, b)$，而非高斯分布。

#### 三条路径，同一个终点

| 路径 | 出发点 | 核心操作 | 结果 |
| --- | --- | --- | --- |
| 代数（最小二乘） | 最小化残差平方和 | 求导令其为零 | $X^\mathsf{T} X \mathbf{w} = X^\mathsf{T} \mathbf{y}$ |
| 几何（正交投影） | 残差 ⟂ 列空间 | $X^\mathsf{T}(\mathbf{y} - X\mathbf{w}) = 0$ | $X^\mathsf{T} X \mathbf{w} = X^\mathsf{T} \mathbf{y}$ |
| 概率（极大似然） | 高斯噪声假 | 最大化似然 | $X^\mathsf{T} X \mathbf{w} = X^\mathsf{T} \mathbf{y}$ |

三条完全不同的思路，殊途同归。**正规方程是线性回归 MSE 最优解的充分必要条件**——不管用哪条路推，最终你都必须解这个方程。最小二乘法只是其中最著名的一条。

#### NumPy 实现

```python
import numpy as np

# 生成数据: y = 3x₁ + 2x₂ + 1 + 噪声
np.random.seed(42)
N = 100
X = np.random.randn(N, 2)
true_w = np.array([3.0, 2.0])
true_b = 1.0
y = X @ true_w + true_b + np.random.randn(N) * 0.5

# 补一列 1（将偏置 b 吸收进 w）
X_aug = np.column_stack([X, np.ones(N)])

# 正规方程一步求解
w = np.linalg.inv(X_aug.T @ X_aug) @ X_aug.T @ y
print(f"解得: w₁={w[0]:.4f}, w₂={w[1]:.4f}, b={w[2]:.4f}")
# 典型输出: w₁=2.9873, w₂=2.0126, b=0.9941
```

#### 正规方程的特点与局限

| 优点 | 缺点 |
| --- | --- |
| 一步得到精确解，无需迭代 | 矩阵求逆复杂度 $O(d^3)$，$d$ 大时不可行 |
| 无需调学习率等超参数 | $X^\mathsf{T} X$ 必须可逆（特征不能线性相关） |
| 数学上优雅，适合理论分析 | 只适用于 MSE（MAE 等没有闭式解） |

$d$（特征维度）在几千以内时正规方程完全可用；当 $d$ 达到上万级别时，$O(d^3)$ 的求逆就不可接受了，此时需要梯度下降。

### 5.2 梯度下降——逐步逼近的迭代解

#### 核心思想

不追求一步到位，而是从任意初始点出发，沿**负梯度方向**一步步移动，逐渐逼近最优解。

类比：你蒙着眼睛站在山上，想走到山谷底部。每一步，你感受脚下的坡度（梯度），然后朝最陡的下坡方向迈一步（参数更新）。重复，直到地面变平（梯度 ≈ 0）。

#### 负梯度方向的数学原理

梯度是一个**向量**，每个分量是函数对对应变量的偏导数：

$$
\nabla f(\mathbf{x}) = \left[ \frac{\partial f}{\partial x_1}, \frac{\partial f}{\partial x_2}, \dots, \frac{\partial f}{\partial x_n} \right]^\mathsf{T}
$$

这个向量有两个关键性质：

| 性质 | 说明 |
|------|------|
| **方向** | 梯度指向函数值**上升最快**的方向 |
| **大小（模长）** | 等于该方向上的变化率（斜率） |

为什么梯度是"最陡上升"方向？由**方向导数**决定——函数 $f$ 在点 $\mathbf{x}$ 处沿单位向量 $\mathbf{u}$ 的方向导数为：

$$
\nabla_{\mathbf{u}} f = \nabla f \cdot \mathbf{u} = \|\nabla f\| \cdot \|\mathbf{u}\| \cdot \cos\theta
$$

其中 $\theta$ 是 $\nabla f$ 与 $\mathbf{u}$ 的夹角：

- 当 $\theta = 0^\circ$（$\mathbf{u}$ 与 $\nabla f$ **同向**）：$\cos\theta = 1$，方向导数取最大值 $+\|\nabla f\|$ → **上升最快**
- 当 $\theta = 180^\circ$（$\mathbf{u}$ 与 $\nabla f$ **反向**）：$\cos\theta = -1$，方向导数取最小值 $-\|\nabla f\|$ → **下降最快**

更严格地，将 $f(\mathbf{x} + \Delta\mathbf{x})$ 在 $\mathbf{x}$ 处一阶泰勒展开：

$$
f(\mathbf{x} + \Delta\mathbf{x}) \approx f(\mathbf{x}) + \nabla f(\mathbf{x})^\mathsf{T} \Delta\mathbf{x}
$$

在固定步长 $\|\Delta\mathbf{x}\| = \alpha$ 的约束下，要使 $f(\mathbf{x} + \Delta\mathbf{x})$ 尽可能小，就要让内积 $\nabla f^\mathsf{T} \Delta\mathbf{x}$ 最小。根据柯西-施瓦茨不等式，当 $\Delta\mathbf{x}$ 与 $\nabla f$ **反向**时内积最小：

$$
\Delta\mathbf{x}^* = -\alpha \cdot \frac{\nabla f}{\|\nabla f\|}
$$

这就是**梯度下降更新公式**的来源：

$$
\mathbf{w}_{t+1} = \mathbf{w}_t - \alpha \cdot \nabla \mathcal{L}(\mathbf{w}_t)
$$

拆解这个公式，回答两个关键问题：

**① 损失函数 $\mathcal{L}$ 在这里扮演什么角色？**

损失函数 $\mathcal{L}(\mathbf{w})$ 是一个"裁判"——输入一组参数 $\mathbf{w}$，输出一个数字，代表这组参数有多差。没有它，$\mathbf{w}$ 只是一堆数字，你无法判断哪组参数更好。

而 $\nabla\mathcal{L}(\mathbf{w}_t)$ 需要拆成三部分来读：

| 部分 | 读法 | 含义 |
| --- | --- | --- |
| $\nabla$ | "nabla" 或"梯度" | 对函数求所有偏导数，输出一个**向量**（而非常数） |
| $\mathcal{L}$ | 损失函数 | 被求梯度的对象——即"对谁求导" |
| $(\mathbf{w}_t)$ | "在 $\mathbf{w}_t$ 处" | 梯度算完之后，代入当前这一步的参数值 $\mathbf{w}_t$ 计算具体数值 |

合起来：$\nabla\mathcal{L}(\mathbf{w}_t)$ 就是**损失函数在 $\mathbf{w}_t$ 这点的梯度向量**——它告诉你：如果把 $\mathbf{w}$ 的每个分量各自稍微调大一点，损失会变大还是变小？所有分量的偏导数合成一个向量，给出了损失函数在当前点的完整"地形信息"。



**② 为什么要用减法？**

因为 $\nabla\mathcal{L}$ 指向损失**增大**的方向，而我们想最小化损失，所以必须朝**相反**方向走——用减法。

以一维情况为例（假设只有一个参数 $w$）：

```
梯度 ∇L > 0：当前位置在谷底右侧（上坡）→ w 需要减小 → w - α·正数 → w 变小 ✓
梯度 ∇L < 0：当前位置在谷底左侧（下坡）→ w 需要增大 → w - α·负数 → w 变大 ✓
```

减法自动把正梯度变成"往左走"、负梯度变成"往右走"，始终朝谷底方向校正。如果用了加法，就会背着谷底越走越远——变成**梯度上升**，用于最大化目标。

回到 MSE 损失 $\mathcal{L}(\mathbf{w}) = \frac{1}{N}\|X\mathbf{w} - \mathbf{y}\|^2$：损失曲面是参数空间 $\mathbf{w}$ 上的一个"碗"，梯度 $\nabla \mathcal{L} = \frac{2}{N}X^\mathsf{T}(X\mathbf{w} - \mathbf{y})$ 指向损失**增大**最快的方向。我们要最小化损失，所以沿 $-\nabla \mathcal{L}$ 走——每一步都选择让损失下降最快的方向，逐步逼近碗底。

#### 算法流程

**输入**：特征矩阵 $X$，标签 $\mathbf{y}$，学习率 $\alpha$，迭代次数 $T$

1. 初始化 $\mathbf{w}$（通常随机或全零）
2. 重复 $T$ 次：
   - 计算梯度：$\mathbf{g} = \nabla \mathcal{L}(\mathbf{w})$
   - 更新参数：$\mathbf{w} \leftarrow \mathbf{w} - \alpha \cdot \mathbf{g}$
3. 返回 $\mathbf{w}$

对于 MSE，梯度为 $\nabla \mathcal{L} = \frac{2}{N} X^\mathsf{T} (X\mathbf{w} - \mathbf{y})$。

#### 实现

```python
# 梯度下降求解线性回归（MSE）
def gradient_descent(X, y, alpha=0.01, epochs=1000):
    N, d = X.shape
    w = np.zeros(d)          # 初始化全零
    loss_history = []

    for t in range(epochs):
        # 前向计算预测值
        y_pred = X @ w
        # 计算梯度（MSE）
        grad = (2 / N) * X.T @ (y_pred - y)
        # 更新参数
        w -= alpha * grad
        # 记录损失
        loss = np.mean((y_pred - y) ** 2)
        loss_history.append(loss)

    return w, loss_history

w_gd, losses = gradient_descent(X_aug, y, alpha=0.01, epochs=1000)
print(f"梯度下降解得: w₁={w_gd[0]:.4f}, w₂={w_gd[1]:.4f}, b={w_gd[2]:.4f}")
```

#### 学习率的影响

学习率 $\alpha$ 是梯度下降中最关键的参数：

```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 3, figsize=(14, 4))
alphas = [0.001, 0.01, 0.5]

for ax, alpha in zip(axes, alphas):
    _, losses = gradient_descent(X_aug, y, alpha=alpha, epochs=200)
    ax.plot(losses)
    ax.set_title(f'α = {alpha}')
    ax.set_xlabel('Epoch')
    ax.set_ylabel('MSE Loss')
    ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

| $\alpha$ 取值 | 现象 |
| --- | --- |
| 太小（如 0.001） | 收敛极慢，需要很多轮迭代 |
| 适中（如 0.01） | 平稳下降，高效收敛 |
| 太大（如 0.5） | 来回震荡甚至**发散**（损失越来越大） |

**调参建议**：从 0.01 开始试，如果损失稳定下降就适当增大，如果震荡就减小。常用候选值：0.1, 0.01, 0.001, 0.0001。

### 5.3 正规方程 vs 梯度下降

| 对比维度 | 正规方程 | 梯度下降 |
| --- | --- | --- |
| 求解方式 | 一步到位（闭式解） | 逐步迭代 |
| 时间复杂度 | $O(d^3)$（矩阵求逆） | 每轮 $O(Nd)$（矩阵乘法） |
| 适用特征维度 | $d$ 在数千以内 | $d$ 可以非常大 |
| 是否需要调参 | 不需要 | 需要调学习率 $\alpha$ |
| 适用损失函数 | **仅 MSE**（二次型才有闭式解） | 任何可导的损失函数（MSE、MAE 等） |
| 能否在线学习 | 不能（需全量数据一次计算） | 能（每来一个样本就更新一步） |
| 收敛保证 | 精确到机器精度 | 接近最优解（浮点精度有限） |

### 5.4 为什么 MAE 不能用正规方程？

正规方程的核心一步是把梯度设为零然后解方程。MSE 的梯度是 $\mathbf{w}$ 的线性函数，所以有解。MAE 的梯度是分段常数（$\pm 1/N$），无法写成一个关于 $\mathbf{w}$ 的线性方程组——没有闭式解。

因此 MAE 只能用梯度下降（或其变体）求解。这也是为什么 MSE 在理论和历史上占据主导地位的原因之一——在计算资源匮乏的年代，"有公式直接算"是非常大的优势。

### 5.5 用梯度下降求解 MAE

虽然 MAE 的梯度在零点不可导，但实践中直接使用次梯度（subgradient）即可：

```python
def mae_gradient_descent(X, y, alpha=0.01, epochs=1000):
    N, d = X.shape
    w = np.zeros(d)
    loss_history = []

    for t in range(epochs):
        y_pred = X @ w
        errors = y_pred - y
        # MAE 次梯度: sign(errors)，误差恰好为 0 时取 0
        subgrad = (1 / N) * X.T @ np.sign(errors)
        w -= alpha * subgrad
        loss = np.mean(np.abs(errors))
        loss_history.append(loss)

    return w, loss_history
```

注意：训练末期可能需要调低学习率，因为 MAE 的梯度不会自动衰减——这在第 4.3 节已经讨论过。

### 5.6 小批量梯度下降

全量梯度下降每轮要遍历全部 $N$ 个样本计算梯度，$N$ 很大时效率低。实践中常用折中方案——**小批量梯度下降**（Mini-batch Gradient Descent）：每轮随机抽取一小批样本（如 32 或 64 个）近似计算梯度。

```python
def mini_batch_gd(X, y, alpha=0.01, epochs=100, batch_size=32):
    N, d = X.shape
    w = np.zeros(d)
    loss_history = []

    for t in range(epochs):
        # 随机打乱数据
        idx = np.random.permutation(N)
        X_shuffled, y_shuffled = X[idx], y[idx]

        # 按批次遍历
        for start in range(0, N, batch_size):
            end = min(start + batch_size, N)
            X_batch = X_shuffled[start:end]
            y_batch = y_shuffled[start:end]

            y_pred = X_batch @ w
            grad = (2 / len(X_batch)) * X_batch.T @ (y_pred - y_batch)
            w -= alpha * grad

        # 每轮结束后在全量数据上评估一次损失
        loss = np.mean((X @ w - y) ** 2)
        loss_history.append(loss)

    return w, loss_history
```

三种梯度下降方式的关系：

| 方式 | 每步样本数 | 梯度精度 | 单步代价 |
| --- | --- | --- | --- |
| 批量梯度下降（BGD） | $N$（全量） | 精确 | 高 |
| 随机梯度下降（SGD） | 1 | 非常粗糙 | 极低 |
| 小批量梯度下降（MBGD） | $m$（如 32） | 中等 | 中等 |

实际项目中 MBGD 是最常用的选择，兼顾了计算效率和梯度稳定性。

## 6. 如何选择：一个简单的判断流程

```
数据中是否有异常值？
├── 没有（或已清洗） → MSE
│    理由：梯度自适应、收敛平滑、理论可解释性好（条件均值）
│
└── 有，且不想/不能清洗
      └── 异常值多吗？
            ├── 很多 → MAE
            │    理由：异常值不会被平方放大，鲁棒性强
            │    注意：训练末期降低学习率以稳定收敛
            │
            └── 少量 → 可以考虑 Huber Loss（MSE 和 MAE 的折中）
                      本文暂不展开，核心思想是：
                      小误差用 MSE，大误差用 MAE，平滑过渡
```

**经验法则**：不确定的时候，**先用 MSE**。它是回归任务的默认选择，大多数情况下表现良好。如果发现模型被少量极端值"带偏"（验证集上表现很差但训练集很好？检查一下是否是被异常值干扰），再切换到 MAE。

## 7. 总结

MSE 和 MAE 看起来只是"平方"和"绝对值"的区别，但这一区别贯穿了优化的所有方面：

| 层面 | MSE 的核心特质 | MAE 的核心特质 |
| --- | --- | --- |
| 几何 | 二次惩罚，大误差被放大 | 线性惩罚，所有误差平权 |
| 梯度 | 自适应——误差大则快，小则慢 | 恒定——永远走一样大的步子 |
| 统计 | 学到条件均值 | 学到条件中位数 |
| 工程 | 收敛平滑，怕异常值 | 鲁棒抗噪，末期需降学习率 |

**一句话总结**：MSE 追求"精确"（但易受干扰），MAE 追求"稳健"（但不够精细）。理解这两者的取舍，是理解更复杂损失函数（Huber、Log-Cosh、分位数损失等）的基础。

**延伸阅读**：

- [归一化与标准化——公式与对比](/2026/06/17/归一化与标准化-公式与对比/) — 损失函数计算前的关键预处理步骤
- [欧氏距离-曼哈顿距离-切比雪夫距离](/2026/06/17/欧氏距离-曼哈顿距离-切比雪夫距离/) — MSE 和 MAE 背后的距离度量：L2 与 L1
