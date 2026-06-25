---
title: sklearn 中 LinearRegression 详解
date: 2026-06-20
updated: 2026-06-25 20:34:05
tags: [sklearn, 机器学习, 线性回归, Python, LinearRegression, 最小二乘法]
categories: 机器学习
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 深入解析 sklearn 中 LinearRegression 的每一个参数、属性与方法，配合完整示例和常见问题，彻底掌握线性回归在 sklearn 中的使用
cover: https://picsum.photos/id/220/800/450
keywords: LinearRegression,sklearn,线性回归,最小二乘法,OLS,fit,predict,score,coef_,intercept_
katex: true
---

# sklearn 中 LinearRegression 详解

`LinearRegression` 是 sklearn 中对**普通最小二乘法**（Ordinary Least Squares, OLS）的封装，一句话就能完成线性模型训练。

## 1. 核心原理

`LinearRegression` 试图找到一组系数 $w = (w_1, w_2, \dots, w_p)$ 和一个截距 $b$，使得预测值 $\hat{y}$ 和真实值 $y$ 之间的均方误差最小：

$$
\min_{w, b} \frac{1}{N} \sum_{i=1}^{N} (y_i - (w \cdot x_i + b))^2
$$

sklearn 内部使用 `scipy.linalg.lstsq` 求解最小二乘问题，底层基于 LAPACK 驱动，计算的是最小范数解。

## 2. 参数详解

`LinearRegression` 的构造函数一共有四个参数：

| 参数 | 类型 | 默认值 | 可选值 | 说明 |
|------|------|--------|--------|------|
| `fit_intercept` | `bool` | `True` | `True` / `False` | 是否计算截距 $b$。设为 `False` 时强制回归线过原点（即 $b=0$），适用于数据已经中心化或物理意义上必须过零点的场景 |
| `copy_X` | `bool` | `True` | `True` / `False` | 是否在计算前复制输入矩阵 `X`。设为 `False` 可能覆盖原始数据以节省内存，但通常不需要改动 |
| `n_jobs` | `int` | `None` | `None` / 正整数 | 并行计算的线程数。`None` 表示 1。此参数仅在特征数 > 1 且样本量足够大时生效，加速效果因数据规模而异 |
| `positive` | `bool` | `False` | `True` / `False` | 是否强制所有系数 $w_i \ge 0$。当业务上要求特征只能产生正向影响时使用（0.24 版本新增） |

```python
from sklearn.linear_model import LinearRegression

# 默认配置
model = LinearRegression()                                # 有截距，不强制正系数

# 强制过原点
model_no_intercept = LinearRegression(fit_intercept=False)

# 强制所有系数非负
model_positive = LinearRegression(positive=True)

# 多核加速
model_fast = LinearRegression(n_jobs=-1)                  # 使用所有可用核心
```

## 3. 属性详解

模型训练后（调用 `fit` 之后），以下属性被填充（以下划线结尾的属性表示从数据中学到的）：

| 属性 | 类型 | 形状 | 说明 |
|------|------|------|------|
| `coef_` | `ndarray` | `(n_features,)` 或 `(n_targets, n_features)` | 每个特征的权重系数 $w$。单目标回归为一维数组，多目标回归（`y` 为二维）时为二维数组 |
| `intercept_` | `float` 或 `ndarray` | 标量 或 `(n_targets,)` | 截距 $b$（偏置项）。`fit_intercept=False` 时为 `0.0`。多目标回归时为数组 |
| `rank_` | `int` | 标量 | 输入矩阵 `X` 的秩。可用于检测多重共线性——若 `rank_` 远小于特征数，说明存在高度相关特征 |
| `singular_` | `ndarray` | `(min(X, y),)` | 输入矩阵 `X` 的奇异值。可用于进一步诊断数值稳定性 |
| `n_features_in_` | `int` | 标量 | 训练时看到的特征数量，供 `predict` 校验输入维度 |
| `feature_names_in_` | `ndarray` | `(n_features_in_,)` | 训练时看到的特征名称（仅在 `fit` 传入 DataFrame 时填充） |

```python
model.fit(X, y)

print(model.coef_)                   # [1.5, -0.3]  → 两个特征的权重
print(model.intercept_)              # 2.0           → 截距
print(model.rank_)                   # 2             → X 的秩
print(model.n_features_in_)          # 2
```

## 4. 方法详解

### fit

```python
model.fit(X, y, sample_weight=None)
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `X` | `array-like`, shape `(n_samples, n_features)` | 必填 | 训练数据特征矩阵 |
| `y` | `array-like`, shape `(n_samples,)` 或 `(n_samples, n_targets)` | 必填 | 目标值。二维时为多目标回归 |
| `sample_weight` | `array-like`, shape `(n_samples,)` | `None` | 每个样本的权重。不传则所有样本等权 |

返回值：`self`（即模型自身），因此支持链式调用。

背后发生了什么：

1. 检查输入数据维度、类型是否合法
2. 如果 `fit_intercept=True`，在 `X` 左侧拼接一列全 1（对应截距项）
3. 调用 `scipy.linalg.lstsq` 求解 $w = (X^\mathsf{T}X)^{-1}X^\mathsf{T}y$
4. 将解的前 $p$ 个分量存入 `coef_`，第 $p+1$ 个分量（截距）存入 `intercept_`

### predict

```python
y_pred = model.predict(X)
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `X` | `array-like`, shape `(n_samples, n_features)` | 待预测的特征矩阵 |

返回值：`ndarray`，shape `(n_samples,)` 或 `(n_samples, n_targets)`。

```python
y_pred = model.predict(X_test)
# y_pred = X_test @ model.coef_ + model.intercept_   ← 等价手算
```

### score

```python
r2 = model.score(X, y, sample_weight=None)
```

返回 $R^2$（决定系数），衡量模型拟合优度：

$$
R^2 = 1 - \frac{\sum_i (y_i - \hat{y}_i)^2}{\sum_i (y_i - \bar{y})^2}
$$

- $R^2=1$ 表示完美拟合
- $R^2=0$ 表示模型等价于一直预测均值
- $R^2<0$ 表示模型比瞎猜还要差

### get_params / set_params

遵循 sklearn 统一的 Estimator 接口：

```python
params = model.get_params()           # {'fit_intercept': True, 'copy_X': True, ...}
model.set_params(fit_intercept=False) # 修改参数，返回 self
```

这两个方法在 `GridSearchCV` 自动化调参中起关键作用——`GridSearchCV` 通过 `get_params` 获取可调参数的键名列表，通过 `set_params` 为每一组候选参数重新配置模型。

## 5. 完整使用示例

```python
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.datasets import load_diabetes
import numpy as np

# 1. 加载数据
X, y = load_diabetes(return_X_y=True)

# 2. 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 3. 训练模型
model = LinearRegression()
model.fit(X_train, y_train)

# 4. 查看学到的参数
print(f"截距: {model.intercept_:.4f}")
print(f"系数: {model.coef_}")

# 5. 预测
y_pred = model.predict(X_test)

# 6. 评估
print(f"训练集 R²: {model.score(X_train, y_train):.4f}")
print(f"测试集 R²: {model.score(X_test, y_test):.4f}")
```

输出示例：

```
截距: 151.7140
系数: [ 37.9  -241.9   542.5   347.6  -931.6   518.1   163.5   275.0   736.3    48.7]
训练集 R²: 0.5279
测试集 R²: 0.4526
```

一个系数为正，表示该特征对目标值有正向影响（特征值增大，预测值随之增大）；系数为负则反之。绝对值越大，影响越强——但前提是特征尺度可比（见常见问题 Q3）。

## 6. 常见问题

**Q1: 什么时候不用 `fit_intercept`？**

当数据已经**中心化**（每列均值为零），或者业务上明确回归线必须过原点时，可以设 `fit_intercept=False`。后者常见的场景是物理公式——比如 $F=ma$，质量为零时力必须为零。一般情况下保持默认 `True`。

**Q2: `coef_` 的大小能直接判断特征重要性吗？**

不能直接比较。如果特征 X1 的量纲是"米"，X2 的量纲是"毫米"，即使 X2 实际重要性远低于 X1，它的系数绝对值可能大 1000 倍。需要先标准化（`StandardScaler`）再比较，或使用 `sklearn.inspection.permutation_importance`。

**Q3: 线性回归需要做特征缩放吗？**

**训练本身不需要**——最小二乘法在数学上不受特征尺度影响，`fit` 后的 `coef_` 会自动按比例缩放。但以下场景建议做标准化：

- 使用**正则化**（Ridge / Lasso）时——正则化对系数施加惩罚，尺度不一致会导致惩罚不公
- 使用**梯度下降**求解时（如 `SGDRegressor`）——不同尺度会导致优化困难
- **解释系数**时——如 Q2 所述，尺度统一后系数才有可比性

**Q4: 如何检测多重共线性？**

当两个或多个特征高度相关时，$(X^\mathsf{T}X)$ 接近奇异矩阵，解法在数值上不稳定，`coef_` 会变得极其敏感（数据微调系数就大变）。检测方法：

```python
model.fit(X, y)
print(f"特征数: {X.shape[1]}, 秩: {model.rank_}")
# 若 rank_ 显著小于特征数，存在共线性
```

也可以通过 VIF（方差膨胀因子）或相关系数矩阵热力图进一步诊断。解决方法包括删除高相关特征、使用正则化（Ridge）或降维（PCA）。

**Q5: `positive=True` 什么时候用？**

当你从业务逻辑上明确知道某些特征只能正向影响目标值时使用。例如：

- 广告投入 → 销售额（投入越多，销量不应下降）
- 房屋面积 → 房价（面积越大，价格不应更低）
- 学习时间 → 考试分数（多学不应导致分数降低）

开启后，解法从 OLS 变为约束优化问题，底层使用 `scipy.optimize.nnls`。

**Q6: `predict` 和 `score` 分别用哪个数据集？**

`predict` 和 `score` 可以用于任意数据集。评估模型时通常同时关注训练集和测试集的表现：

```python
train_r2 = model.score(X_train, y_train)   # 训练集表现 → 判断是否欠拟合
test_r2  = model.score(X_test, y_test)      # 测试集表现 → 判断泛化能力
```

训练集 $R^2$ 高而测试集 $R^2$ 低 → 可能过拟合；两者都低 → 可能欠拟合；两者都高且接近 → 理想状态。

## 7. 小结

`LinearRegression` 是 sklearn 中最简单的回归模型——四个参数控制行为，`fit` 求解最小二乘，`predict` 输出预测值，`score` 返回 $R^2$。简单背后是可靠的数值计算和统一的接口设计，是学习 sklearn 的最佳起点。
