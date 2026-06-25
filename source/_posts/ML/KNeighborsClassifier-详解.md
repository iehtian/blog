---
title: KNeighborsClassifier——K 近邻分类器详解
date: 2026-06-19
updated: 2026-06-25 20:34:05
tags: [机器学习, 分类算法, sklearn, KNN, KNeighborsClassifier, 监督学习, 距离度量]
categories: 机器学习
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 深入讲解 sklearn 中 KNeighborsClassifier 的原理、核心参数、距离度量选择、k 值影响及实战用法，并给出避坑指南
cover: https://picsum.photos/id/102/800/450
keywords: KNeighborsClassifier,KNN,K近邻,分类算法,sklearn,机器学习,距离度量,交叉验证,k值选择
katex: true
---

# KNeighborsClassifier——K 近邻分类器详解

## 1. 什么是 KNN

**K 近邻（K-Nearest Neighbors, KNN）** 基于"相似样本具有相似标签"的假设进行分类，属于**惰性学习**（Lazy Learning）——没有显式训练阶段，仅在预测时计算。`KNeighborsClassifier` 是 sklearn 中 KNN 分类器的官方实现。

算法流程：

1. **计算距离**：计算待分类样本与训练集中每个样本之间的距离；
2. **找 K 近邻**：选出距离最近的 K 个训练样本；
3. **投票表决**：根据 K 个邻居的类别进行多数投票，票数最多的类别即为预测结果。

KNN 没有显式的"训练"阶段——它只是将训练数据存储起来，等到预测时才进行计算。因此也被称为**基于实例的学习**（Instance-Based Learning）。

## 2. 核心原理

### 2.1 距离计算

对于两个 $n$ 维向量 $x = (x_1, x_2, \dots, x_n)$ 和 $y = (y_1, y_2, \dots, y_n)$，KNN 默认使用**闵可夫斯基距离**，其定义为：

$$
D_p(x, y) = \left( \sum_{i=1}^{n} |x_i - y_i|^p \right)^{1/p}
$$

其中 $p$ 是距离参数：

- $p = 1$：曼哈顿距离
- $p = 2$（默认值）：欧氏距离

KNN 默认使用 $p=2$ 即欧氏距离：

$$
d(x, y) = \sqrt{\sum_{i=1}^{n} (x_i - y_i)^2}
$$

### 2.2 多数表决

选定 K 个最近邻居后，分类结果由投票决定：

$$
\hat{y} = \arg\max_{c} \sum_{i=1}^{K} \mathbb{1}(y_i = c)
$$

其中 $\mathbb{1}(\cdot)$ 是指示函数，$y_i$ 是第 $i$ 个邻居的标签。即统计每个类别在 K 个邻居中出现的次数，取出现次数最多的类别。

### 2.3 加权投票

当设置 `weights='distance'` 时，每个邻居的投票权重与其距离成反比——距离越近的邻居对分类结果的贡献越大：

$$
\hat{y} = \arg\max_{c} \sum_{i=1}^{K} \frac{1}{d(x, x_i)} \cdot \mathbb{1}(y_i = c)
$$

这种做法可以减少远距离邻居对投票结果的干扰，尤其适合样本分布不均的场景。

## 3. 核心参数详解

```python
from sklearn.neighbors import KNeighborsClassifier

KNeighborsClassifier(
    n_neighbors=5,        # K 值：多少个最近邻居参与投票
    weights='uniform',    # 投票权重：'uniform' 或 'distance'
    algorithm='auto',     # 搜索算法：'auto', 'ball_tree', 'kd_tree', 'brute'
    leaf_size=30,         # BallTree / KDTree 的叶节点大小
    p=2,                  # 闵可夫斯基距离的幂参数（p=2 即欧氏距离）
    metric='minkowski',   # 距离度量方式
    metric_params=None,   # 距离函数的额外参数
    n_jobs=None,          # 并行任务数
)
```

### 3.1 `n_neighbors` —— K 值

最重要的超参数，决定了参与投票的邻居数量。

- **K 太小**（如 K=1）：模型对噪声敏感，容易过拟合。单个异常点可能导致错误分类；
- **K 太大**：决策边界过于平滑，可能忽略数据中的局部特征，导致欠拟合；
- **经验法则**：通常取奇数（避免二分类平票），初始值可取 $K \approx \sqrt{N}$（$N$ 为训练样本数），再通过交叉验证调优。

### 3.2 `weights` —— 投票权重

| 值 | 行为 |
| --- | --- |
| `'uniform'`（默认） | 所有 K 个邻居权重相同，纯多数投票 |
| `'distance'` | 权重与距离成反比，近邻影响更大 |
| 自定义 callable | 传入自定义权重函数 |

```python
# uniform: K 个邻居一视同仁
knn_uniform = KNeighborsClassifier(n_neighbors=5, weights='uniform')

# distance: 距离越近权重越大
knn_distance = KNeighborsClassifier(n_neighbors=5, weights='distance')
```

### 3.3 `algorithm` —— 搜索算法

控制如何在高维空间中高效搜索最近邻：

| 值 | 说明 |
| --- | --- |
| `'auto'`（默认） | 根据训练数据自动选择最优算法 |
| `'brute'` | 暴力搜索，逐个计算所有样本的距离——$O(n \cdot d)$ |
| `'kd_tree'` | KD 树——适合低维数据（$d < 20$），$O(d \log n)$ |
| `'ball_tree'` | Ball 树——适合中高维数据，$O(d \log n)$ |

```python
# 数据量小时 brute 可能更快（没有建树开销）
knn = KNeighborsClassifier(algorithm='brute')

# 高维稀疏数据推荐 ball_tree
knn = KNeighborsClassifier(algorithm='ball_tree')
```

### 3.4 `p` 与 `metric` —— 距离度量

默认使用闵可夫斯基距离，通过 `p` 控制具体类型：

```python
# 欧氏距离（默认 p=2）
KNeighborsClassifier(p=2)

# 曼哈顿距离
KNeighborsClassifier(p=1)

# 也可以直接指定 metric
KNeighborsClassifier(metric='euclidean')
KNeighborsClassifier(metric='manhattan')
KNeighborsClassifier(metric='chebyshev')
```

sklearn 也内置了余弦距离等更多度量：

```python
# 余弦距离（常用于文本相似度场景）
knn = KNeighborsClassifier(metric='cosine')
```

### 3.5 `leaf_size` —— 叶节点大小

控制 BallTree / KDTree 中叶节点的最大样本数：

- **较大值**：建树更快，但查询略慢（更多样本需要暴力比对）；
- **较小值**：建树略慢，查询更快；
- 默认值 `30` 在大多数场景下表现良好，一般无需修改。

### 3.6 `n_jobs` —— 并行处理

KNN 的预测阶段需要计算与所有训练样本的距离，这部分可以利用多核并行加速。`n_jobs=-1` 使用全部 CPU 核心。

## 4. 拟合后的属性与方法

`fit()` 完成后，`KNeighborsClassifier` 对象会暴露一系列属性和方法，用于查询模型状态、进行预测和获取近邻信息。理解这些接口的含义和区别是正确使用 KNN 的前提。

### 4.1 模型属性（Attributes）

以下属性在 `fit()` 之后可用，均为**以单下划线结尾的实例属性**（sklearn 约定：fit 后生成的属性以 `_` 结尾）。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `classes_` | ndarray of shape (n_classes,) | 训练数据中的类别标签数组 |
| `effective_metric_` | str 或 callable | 实际使用的距离度量方式。若初始化时传入 `metric='minkowski'`，此处会被解析为 `'minkowski'` 字符串；若传入自定义函数，此处保存该函数引用 |
| `effective_metric_params_` | dict | 实际传递给距离函数的额外参数。当 `metric='minkowski'` 时，此字典包含 `{'p': 2}`（或你指定的 p 值） |
| `n_features_in_` | int | `fit()` 时看到的特征数量 |
| `feature_names_in_` | ndarray of shape (n_features_in_,) | `fit()` 时看到的特征名称（仅当输入为 DataFrame 时有值） |
| `n_samples_fit_` | int | `fit()` 时使用的训练样本数量 |
| `outputs_2d_` | bool | 输出是否为二维。当 `fit()` 时传入的 `y` 是多维数组时为 `True` |

**关键区分**：上述属性都是 `KNeighborsClassifier` 实例的**顶层属性**，直接通过 `knn.classes_`、`knn.effective_metric_` 等方式访问。这与 `GridSearchCV` 的 `cv_results_` 不同——`cv_results_` 本身是一个 dict，它的 key（如 `mean_test_score`）不是 GridSearchCV 的顶层属性，必须通过 `grid.cv_results_['mean_test_score']` 访问。

```python
knn = KNeighborsClassifier(n_neighbors=5, metric='minkowski', p=2)
knn.fit(X_train, y_train)

# 顶层属性——直接访问
print(knn.classes_)                   # [0 1 2]
print(knn.effective_metric_)          # 'minkowski'
print(knn.effective_metric_params_)   # {'p': 2}
print(knn.n_features_in_)             # 4
print(knn.n_samples_fit_)             # 120
print(knn.outputs_2d_)                # False
```

### 4.2 核心方法（Methods）

#### `fit(X, y)` —— 存储训练数据

KNN 的 `fit()` 并不做实际"训练"，只是将训练数据存入内部结构（构建搜索树或直接存储原始数据），因此**几乎瞬间完成**。

```python
knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train, y_train)  # 存储数据，构建搜索树（取决于 algorithm 参数）
```

#### `predict(X)` —— 预测类别标签

返回每个输入样本的预测类别：

```python
y_pred = knn.predict(X_test)
# 返回: array([1, 0, 2, 2, 1, ...])
```

#### `predict_proba(X)` —— 预测各类别概率

返回每个样本在各个类别上的概率分布（K 个邻居中各类别所占比例）：

```python
proba = knn.predict_proba(X_test[:3])
# 返回 shape (3, n_classes) 的概率数组
# 例: [[0. , 0.8, 0.2],
#       [1. , 0. , 0. ],
#       [0. , 0.2, 0.8]]
```

注意：当 `weights='distance'` 时，概率按距离加权计算，而非简单计数。

#### `kneighbors(X, n_neighbors, return_distance)` —— 查询 K 近邻

KNN 最核心的方法之一，返回每个查询样本的 K 个最近邻的**距离**和**索引**。`n_neighbors` 若不指定则使用初始化时的 `n_neighbors` 值。

```python
# 同时返回距离和邻居索引
distances, indices = knn.kneighbors(X_test[:3])
print(distances)  # shape (3, 5): 每个样本的 5 个最近邻距离
print(indices)    # shape (3, 5): 每个样本的 5 个最近邻在训练集中的索引

# 只返回邻居索引（不返回距离）
indices = knn.kneighbors(X_test[:3], return_distance=False)
```

**用途举例**——查看测试样本的最近邻类别：

```python
distances, indices = knn.kneighbors(X_test[:3])
for i, (dist, idx) in enumerate(zip(distances, indices)):
    neighbor_labels = y_train.iloc[idx].values if hasattr(y_train, 'iloc') else y_train[idx]
    print(f"样本 {i}: 最近邻索引 = {idx}, 距离 = {dist}, 邻居标签 = {neighbor_labels}")
```

#### `kneighbors_graph(X, n_neighbors, mode)` —— 构建 K 近邻图

返回样本间的稀疏邻接矩阵（scipy sparse matrix），`mode='connectivity'` 返回 0/1 矩阵，`mode='distance'` 返回距离矩阵：

```python
# 返回稀疏连接矩阵
A = knn.kneighbors_graph(X_test, n_neighbors=3, mode='connectivity')
print(A.toarray())  # 转为密集矩阵查看——shape (n_queries, n_samples_fit)
```

这在谱聚类、流形学习等需要构建样本间关系图的场景中非常有用。

#### `score(X, y)` —— 计算准确率

等价于 [`accuracy_score`]({% post_link accuracy_score-分类准确率详解 %})（即 `accuracy_score(y, predict(X))`），是分类器最常用的快捷评估方法：

```python
print(knn.score(X_test, y_test))  # 返回 float，测试集准确率
```

#### `get_params(deep)` 与 `set_params(**params)` —— 参数读写

继承自 sklearn BaseEstimator 的基础方法，用于读取和修改模型参数：

```python
# 获取当前参数
params = knn.get_params()
print(params)  # {'algorithm': 'auto', 'leaf_size': 30, 'metric': 'minkowski', ...}

# 修改参数（无需重新创建对象）
knn.set_params(n_neighbors=10, weights='distance')
# 注意：修改参数后需要重新调用 fit()
```

### 4.3 方法速查表

| 方法 | 返回类型 | 说明 |
| --- | --- | --- |
| `fit(X, y)` | self（当前对象） | 存储训练数据，构建搜索结构 |
| `predict(X)` | ndarray (n_samples,) | 预测类别标签 |
| `predict_proba(X)` | ndarray (n_samples, n_classes) | 预测各类别概率 |
| `kneighbors(X, ...)` | tuple (distances, indices) | 查询 K 个最近邻的距离与索引 |
| `kneighbors_graph(X, ...)` | sparse matrix | 构建 K 近邻稀疏邻接矩阵 |
| `score(X, y)` | float | 计算预测准确率 |
| `get_params()` | dict | 获取当前所有参数 |
| `set_params(**params)` | self | 修改参数（需重新 fit） |

### 4.4 基础实战——鸢尾花分类

结合上述属性和方法，一个完整的 KNN 分类流程如下：

```python
from sklearn.datasets import load_iris
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report

# 1. 加载数据
iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 2. 创建并拟合
knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train, y_train)

# 3. 查看模型属性
print("类别:", knn.classes_)
print("实际距离度量:", knn.effective_metric_)
print("度量参数:", knn.effective_metric_params_)
print("特征数:", knn.n_features_in_)
print("训练样本数:", knn.n_samples_fit_)

# 4. 预测与评估
print("准确率:", knn.score(X_test, y_test))
y_pred = knn.predict(X_test)
print(classification_report(y_test, y_pred, target_names=iris.target_names))

# 5. 查询近邻
dist, idx = knn.kneighbors(X_test[:1])
print("最近邻索引:", idx)
print("最近邻距离:", dist)
print("最近邻标签:", y_train[idx[0]])
```

配合 `GridSearchCV` 可自动搜索最优 `n_neighbors` 和 `weights`：

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsClassifier()),
])

param_grid = {
    'knn__n_neighbors': range(1, 31),
    'knn__weights': ['uniform', 'distance'],
    'knn__metric': ['euclidean', 'manhattan'],
}

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy', n_jobs=-1)
grid.fit(X_train, y_train)
print("最优参数:", grid.best_params_)
print("最优得分:", grid.best_score_)
```

## 5. K 值的影响深入分析

### 5.1 偏差-方差权衡

| K 值 | 模型复杂度 | 偏差 | 方差 | 表现 |
| --- | --- | --- | --- | --- |
| K=1 | 极高 | 极低 | 极高 | 对噪声极度敏感，容易过拟合 |
| K 适中 | 中等 | 中等 | 中等 | 平衡偏差与方差，泛化能力好 |
| K=N（全部样本） | 极低 | 极高 | 极低 | 忽略所有特征，始终预测多数类 |

### 5.2 经验公式

常用的经验公式（仅供参考，最终应以交叉验证为准）：

- $K \approx \sqrt{N}$，$N$ 为训练样本数；
- $K$ 通常取奇数，避免二分类中平票；
- 对于多分类任务，$K$ 不应是类别数的整数倍（避免系统性平票）。

## 6. KNN 的优缺点

### 优点

- **简单直观**：算法原理易于理解，无需复杂的数学推导；
- **无需训练**：没有训练阶段，新增样本无需重新训练；
- **天然支持多分类**：投票机制天然适用于多类别场景；
- **可解释性强**：可以明确指出"因为最近的哪几个邻居是某个类别"。

### 缺点

- **计算开销大**：每次预测都需要计算与全部训练样本的距离，复杂度 $O(N \cdot d)$；
- **内存占用高**：需要存储全部训练数据；
- **维度灾难**：高维空间中距离变得不再有区分度，KNN 效果急剧下降；
- **对尺度敏感**：特征量纲不一致会严重影响距离计算结果；
- **对不平衡数据敏感**：多数类在投票中占据天然优势；
- **无法处理缺失值**：缺失值会导致无法计算距离。

## 7. 常见问题与避坑指南

### 7.1 忘记标准化数据

KNN 基于距离计算，如果特征尺度差异巨大，结果将严重失真。

```python
# 错误！特征未标准化
knn = KNeighborsClassifier()
knn.fit(X_train, y_train)  # 若某特征范围是 [0, 10000]，另一特征是 [0, 1]，后者几乎无贡献

# 正确：使用 Pipeline 嵌入标准化
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsClassifier(n_neighbors=5)),
])
pipe.fit(X_train, y_train)
```

### 7.2 高维数据下 KNN 失效

当特征维度很高时（如文本 TF-IDF 特征），所有样本之间的距离趋向于相同值，KNN 失去区分能力。对于高维稀疏数据，考虑使用：

- 降维（PCA、t-SNE）后再用 KNN；
- 改用对高维友好的模型（如 SVM、随机森林）。

### 7.3 类别不平衡

当某类样本数远大于其他类时，多数类会在 K 个邻居中占据优势。

**应对策略**：

```python
# 使用 distance 权重，让近邻的影响大于远邻
KNeighborsClassifier(n_neighbors=5, weights='distance')

# 或使用 class_weight 平衡的替代方案，如 sklearn 的 BalancedBaggingClassifier
```

### 7.4 大数据集下的性能问题

KNN 预测时需要扫描全部训练样本，百万级数据集下预测极慢。

**应对策略**：

- 使用 KDTree 或 BallTree 算法（`algorithm='ball_tree'`）；
- 对训练集做采样或原型选择（如浓缩近邻算法）；
- 考虑改用支持向量机或随机森林等有显式决策边界的模型。

### 7.5 K 值选择不当

K=1 时模型会完美拟合训练数据（训练集准确率 100%），但泛化能力差。不要被训练集满分迷惑——务必使用交叉验证或独立验证集来评估模型。

## 8. 小结

- KNN 是基于"近朱者赤，近墨者黑"思想的最直观分类算法，无需训练过程；
- **K 值**是最关键的超参数：太小容易过拟合，太大会欠拟合，应通过交叉验证选择；
- **数据标准化**是 KNN 的硬性前提——不标准化会严重影响距离计算的结果；
- 距离度量可根据场景选择：欧氏距离适合低维连续特征，曼哈顿距离对异常值更鲁棒；
- KNN 不适合高维稀疏数据和大规模数据集，这些场景下应考虑其他算法；
- 结合 `Pipeline` 和 `GridSearchCV` 可以将预处理和参数调优整合到统一的流程中。
