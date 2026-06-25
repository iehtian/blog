---
title: sklearn 中 fit_transform 与 transform 详解
date: 2026-06-17
updated: 2026-06-25 20:34:05
tags: [sklearn, 机器学习, 数据预处理, fit_transform, transform, fit]
categories: 机器学习
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 深入解析 sklearn 中 fit、transform、fit_transform 的区别与联系，理解为何训练集和测试集必须分开处理，避免数据泄露
cover: https://picsum.photos/id/161/800/450
keywords: fit_transform,transform,fit,sklearn,数据归一化,StandardScaler,MinMaxScaler,数据泄露
---

# fit_transform 与 transform 详解

## 1. 核心概念

sklearn 中所有数据预处理类都遵循一套统一的接口设计，由三个方法构成：

| 方法 | 作用 | 输入 | 输出 |
|------|------|------|------|
| `fit` | 从数据中**学习**参数 | 训练数据 X | 自身（含学习到的参数） |
| `transform` | 使用已学习的参数**转换**数据 | 任意数据 X | 转换后的数据 |
| `fit_transform` | 先 fit 再 transform | 训练数据 X | 转换后的数据 |

三者的关系可以简单理解为：

```python
# 以下两行等价
scaler.fit(X_train).transform(X_train)
scaler.fit_transform(X_train)
```

## 2. 以 StandardScaler 为例

`StandardScaler` 的作用是将数据转换为均值为 0、方差为 1 的标准正态分布，公式为：

```
X_scaled = (X - mean) / std
```

其中 `mean` 和 `std` 就是需要从训练数据中**学习**的参数。

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

# 模拟训练数据
X_train = np.array([[1], [2], [3], [4], [5]])
X_test  = np.array([[1.5], [2.5], [6.0]])

scaler = StandardScaler()

# 方式一：分开调用 fit + transform
scaler.fit(X_train)              # 学习 mean=3, std≈1.58
X_train_scaled = scaler.transform(X_train)

# 方式二：一步到位
# X_train_scaled = scaler.fit_transform(X_train)

# 测试集：只能 transform，绝不能 fit！
X_test_scaled = scaler.transform(X_test)

print("训练集 mean:", scaler.mean_)   # [3.]
print("训练集 std:", scaler.scale_)   # [1.58113883]
print("训练集结果:\n", X_train_scaled)
print("测试集结果:\n", X_test_scaled)
```

关键参数被存储在 `scaler.mean_` 和 `scaler.scale_` 中（以下划线结尾的属性表示从数据中学到的参数）。

## 3. 以 MinMaxScaler 为例

`MinMaxScaler` 将数据线性映射到 `[0, 1]` 区间（默认），公式为：

```
X_scaled = (X - X_min) / (X_max - X_min)
```

```python
from sklearn.preprocessing import MinMaxScaler

X_train = np.array([[10], [20], [30], [40], [50]])
X_test  = np.array([[15], [25], [60]])

scaler = MinMaxScaler()

# 在训练集上 fit_transform
X_train_scaled = scaler.fit_transform(X_train)
print("训练集结果:\n", X_train_scaled)
# [[0.  ]
#  [0.25]
#  [0.5 ]
#  [0.75]
#  [1.  ]]

# 测试集只用 transform
X_test_scaled = scaler.transform(X_test)
print("测试集结果:\n", X_test_scaled)
# [[0.125]   → (15-10)/(50-10)
#  [0.375]   → (25-10)/(50-10)
#  [1.25 ]]  → (60-10)/(50-10)  超出 [0,1]！

print("min:", scaler.min_)   # [10.]
print("max:", scaler.data_max_)  # [50.]
```

注意：测试集中的 `60` 超出了训练集的范围，因此缩放后值为 1.25，超过了 `[0, 1]` 区间。这正是 `MinMaxScaler` 的一个特性——它不保证测试集一定落在 `[0, 1]` 内。

## 4. 为什么训练集和测试集必须分开处理

这是初学者最容易犯的错误，也是最需要理解的设计理念。下面分别用 `StandardScaler` 和 `MinMaxScaler` 展示正确与错误的做法——两个 Scaler 遵循完全相同的原则。

### 错误做法：在全部数据上 fit

```python
# ❌ StandardScaler — 错误
X_all = np.vstack([X_train, X_test])
scaler = StandardScaler()
X_all_scaled = scaler.fit_transform(X_all)  # 数据泄露！mean/std 混入了测试集信息

# ❌ MinMaxScaler — 同样错误
X_all = np.vstack([X_train, X_test])
scaler = MinMaxScaler()
X_all_scaled = scaler.fit_transform(X_all)  # 数据泄露！min/max 混入了测试集信息
```

### 正确做法：仅在训练集上 fit

```python
# ✅ StandardScaler — 正确
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # 只用训练集学习 mean 和 std
X_test_scaled  = scaler.transform(X_test)        # 测试集用训练集的 mean/std 转换

# ✅ MinMaxScaler — 同样正确
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)  # 只用训练集学习 min 和 max
X_test_scaled  = scaler.transform(X_test)        # 测试集用训练集的 min/max 转换
```

### 原因

想象一个真实场景：你训练好模型后部署上线，新来的数据你无法预知它们的分布。如果 fit 时混入了测试集（或未来的数据），你就**偷看**了不该看的信息，这被称为**数据泄露（Data Leakage）**。

具体来说：

- **fit** 是在从数据中学习统计信息（均值、方差、最大最小值等），这些信息**只能**来自训练集
- **transform** 是应用已经学好的规则，对任何数据都是同一套规则
- 测试集模拟的是"模型从未见过的未来数据"，因此必须用训练集学到的参数去转换

## 5. 常见 Transformer 的 fit 学了什么

不同的预处理类，`fit` 学到的参数不同：

| Transformer | fit 学到的参数 |
|-------------|---------------|
| `StandardScaler` | `mean_`, `scale_`（标准差） |
| `MinMaxScaler` | `data_min_`, `data_max_` |
| `MaxAbsScaler` | `max_abs_` |
| `RobustScaler` | `center_`（中位数）, `scale_`（IQR） |
| `OneHotEncoder` | `categories_`（每个特征的类别列表） |
| `LabelEncoder` | `classes_`（所有标签） |
| `SimpleImputer` | `statistics_`（均值/中位数/众数） |

## 6. 完整训练流程示例

无论使用 `StandardScaler` 还是 `MinMaxScaler`，训练流程的模式完全一致——差异仅在于选择哪个 Scaler，`fit`/`transform` 的调用方式不变。

### 使用 StandardScaler

```python
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)

model = LogisticRegression()
model.fit(X_train_scaled, y_train)
print(f"StandardScaler 准确率: {model.score(X_test_scaled, y_test):.4f}")
```

### 使用 MinMaxScaler

```python
from sklearn.preprocessing import MinMaxScaler

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)

model = LogisticRegression()
model.fit(X_train_scaled, y_train)
print(f"MinMaxScaler 准确率: {model.score(X_test_scaled, y_test):.4f}")
```

可以看到，除了 Scaler 的类名不同，`fit_transform` / `transform` 的调用模式完全一样。这就是 sklearn 统一接口设计的价值——学会一个，全部通用。

## 7. 常见问题

**Q1: 能不能对训练集多次调用 `fit_transform`？**

不能。第二次调用 `fit_transform` 会重新 fit，之前学到的参数会被覆盖。如果只是想再次转换，用 `transform`。

**Q2: 交叉验证中怎么处理？**

每一折交叉验证中，只用该折的训练部分 fit，验证部分 transform。sklearn 的 `Pipeline` + `cross_val_score` 会自动处理这一点：

```python
from sklearn.pipeline import Pipeline

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', LogisticRegression())
])

# cross_val_score 内部会在每一折自动 fit_transform 训练部分，transform 验证部分
from sklearn.model_selection import cross_val_score
scores = cross_val_score(pipe, X, y, cv=5)
```

**Q3: `fit_transform` 返回的是什么类型？**

返回的是 NumPy 数组（`numpy.ndarray`）。如果想保留 DataFrame 格式，可以手动转换回去。

**Q4: 为什么有些 Transformer 只有 `fit_transform` 没有单独的 `fit`？**

确实有少量类（如 `KBinsDiscretizer` 的某些场景）更倾向连用，但绝大多数标准 Transformer 都支持三个方法分开调用，这是 sklearn 的设计约定。

## 8. 小结

- `fit`：看数据，学参数
- `transform`：用参数，转数据
- `fit_transform`：在训练集上一步到位
- 铁律：**只在训练集上 fit/fit_transform，测试集只 transform**
