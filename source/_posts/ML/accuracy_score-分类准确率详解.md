---
title: accuracy_score——分类准确率详解
date: 2026-06-19
updated: 2026-06-19
tags: [机器学习, 分类算法, sklearn, 评估指标, accuracy_score, 混淆矩阵, 模型评估]
categories: 机器学习
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 深入讲解 sklearn 中 accuracy_score 的用法、参数、计算原理，以及在不平衡数据集中的陷阱与替代指标选择
cover: https://picsum.photos/id/190/800/450
keywords: accuracy_score,准确率,分类评估,sklearn,机器学习,混淆矩阵,precision,recall,f1,类别不平衡
katex: true
---

# accuracy_score——分类准确率详解

## 1. 前言

在分类任务中，模型训练完成后需要一个直观的指标来衡量"预测对了多少"。**准确率（Accuracy）**是最朴素、最常用的分类评估指标——预测正确的样本数占总样本数的比例。

`accuracy_score` 是 sklearn 中计算准确率的标准函数，也是 [`KNeighborsClassifier`]({% post_link KNeighborsClassifier-详解 %}) 的 `score()`、`LogisticRegression.score()` 等分类器默认使用的评分方法。本文将深入讲解其用法、参数、计算原理及使用陷阱。

## 2. 什么是准确率

### 2.1 定义

准确率定义为：**正确分类的样本数占总样本数的比例**。

$$
\text{Accuracy} = \frac{\text{正确预测的数量}}{\text{总样本数量}}
$$

用数学公式表达：

$$
\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}
$$

其中：

- **TP（True Positive）**：真实为正类，预测也为正类；
- **TN（True Negative）**：真实为负类，预测也为负类；
- **FP（False Positive）**：真实为负类，预测为正类（误报）；
- **FN（False Negative）**：真实为正类，预测为负类（漏报）。

### 2.2 直观理解

```
真实标签:  [0, 1, 1, 0, 1, 0, 0, 1, 1, 0]
预测标签:  [0, 1, 0, 0, 1, 0, 1, 1, 1, 0]
           ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑
           ✓  ✓  ✗  ✓  ✓  ✓  ✗  ✓  ✓  ✓

准确率 = 8 / 10 = 0.8
```

## 3. 函数签名与参数

```python
from sklearn.metrics import accuracy_score

accuracy_score(
    y_true,          # 真实标签
    y_pred,          # 预测标签
    normalize=True,  # True 返回比例（0~1），False 返回正确数量
    sample_weight=None,  # 样本权重
)
```

### 3.1 `y_true` 与 `y_pred` —— 真实与预测标签

两个参数都是一维数组，可以是 Python list、numpy array 或 pandas Series，长度必须相同。

```python
from sklearn.metrics import accuracy_score

y_true = [0, 1, 1, 0, 1]
y_pred = [0, 1, 0, 0, 1]

print(accuracy_score(y_true, y_pred))  # 0.8（第 3 个样本预测错误）
```

### 3.2 `normalize` —— 控制返回格式

| 值 | 返回 | 含义 |
| --- | --- | --- |
| `True`（默认） | `float`，范围 [0, 1] | 正确预测的比例 |
| `False` | `int` | 正确预测的绝对数量 |

```python
y_true = [1, 0, 1, 1, 0, 1]
y_pred = [1, 1, 1, 1, 0, 0]

print(accuracy_score(y_true, y_pred, normalize=True))   # 0.666...
print(accuracy_score(y_true, y_pred, normalize=False))  # 4（共 4 个样本预测正确）
```

当 `normalize=False` 时，等价于：

```python
import numpy as np
correct_count = np.sum(np.array(y_true) == np.array(y_pred))
print(correct_count)  # 4
```

### 3.3 `sample_weight` —— 样本权重

为每个样本指定不同的权重，用于调整不同样本在准确率计算中的贡献：

```python
y_true = [1, 0, 1, 0]
y_pred = [1, 1, 1, 0]

# 无权重
print(accuracy_score(y_true, y_pred))  # 0.75（第 2 个样本预测错误）

# 为第 2 个样本（真实标签为 0 的负类）赋予更高权重
sample_weight = [0.5, 2.0, 0.5, 0.5]
print(accuracy_score(y_true, y_pred, sample_weight=sample_weight))  # 0.5714
```

加权计算过程：

$$
\text{Weighted Accuracy} = \frac{\sum_{i: \hat{y}_i = y_i} w_i}{\sum_{i} w_i}
$$

分子只有正确预测样本的权重之和，分母是所有样本权重之和。上例中第 2 个样本权重为 2.0 且预测错误，因此对分子贡献为 0，准确率被拉低。

`sample_weight` 的典型使用场景：

- **类别不平衡**：为少数类样本赋予更高权重；
- **样本重要性差异**：某些样本的业务价值更高；
- **时间衰减**：越近的样本权重越大。

## 4. 与混淆矩阵的关系

准确率可以从**混淆矩阵**直接推导。混淆矩阵的每个元素对应一个 TP/TN/FP/FN 计数：

```
预测\真实   正类(P)    负类(N)
正类(P)      TP        FP
负类(N)      FN        TN
```

```python
from sklearn.metrics import confusion_matrix, accuracy_score
import numpy as np

y_true = [0, 1, 1, 0, 1, 0, 0, 1, 1, 0]
y_pred = [0, 1, 0, 0, 1, 0, 1, 1, 1, 0]

cm = confusion_matrix(y_true, y_pred)
print(cm)
# [[4, 1],     TN=4, FP=1
#  [1, 4]]     FN=1, TP=4

# 从混淆矩阵手动计算准确率
accuracy_manual = np.trace(cm) / np.sum(cm)
print(accuracy_manual)                  # 0.8
print(accuracy_score(y_true, y_pred))   # 0.8（一致）
```

`np.trace(cm)` 是混淆矩阵对角线之和（TP + TN），`np.sum(cm)` 是全部样本数。

## 5. 基础实战

```python
from sklearn.datasets import load_iris
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# 1. 加载数据
iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 2. 训练模型
knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train, y_train)

# 3. 预测
y_pred = knn.predict(X_test)

# 4. 三种等价方式计算准确率
print(accuracy_score(y_test, y_pred))   # 直接调用函数
print(knn.score(X_test, y_test))        # 分类器的 score() 方法（内部也是 accuracy_score）
print((y_test == y_pred).mean())        # 手动计算
```

注意：`knn.score(X_test, y_test)` 内部调用的是 `accuracy_score`，三者结果完全一致。

## 6. 交叉验证中的准确率

在交叉验证中，`accuracy` 是最常用的评分指标之一：

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold

knn = KNeighborsClassifier(n_neighbors=5)
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

# scoring='accuracy' 等价于使用 accuracy_score
scores = cross_val_score(knn, X, y, cv=cv, scoring='accuracy')

print(f"各折得分: {scores}")
print(f"平均准确率: {scores.mean():.4f} ± {scores.std():.4f}")
```

sklearn 中 `scoring='accuracy'` 对应的内部 scorer 就是 `accuracy_score`。

## 7. 准确率的陷阱——类别不平衡

准确率最大的缺陷在于：**当类别严重不平衡时，高准确率不代表模型好**。

### 7.1 一个极端的例子

```python
import numpy as np
from sklearn.metrics import accuracy_score

# 模拟欺诈检测：正类（欺诈）占 1%，负类（正常）占 99%
np.random.seed(42)
y_true = np.zeros(1000)
y_true[:10] = 1  # 10 个欺诈样本

# 一个"聪明"的模型：全部预测为负类（正常）
y_pred_all_negative = np.zeros(1000)

print(accuracy_score(y_true, y_pred_all_negative))  # 0.99！
```

模型什么都没学到，却拿到了 99% 的准确率——因为它"猜多数类就对了"。但实际问题中，漏掉欺诈交易的代价远高于误判正常交易。

### 7.2 对比其他指标

```python
from sklearn.metrics import precision_score, recall_score, f1_score

print(f"准确率:   {accuracy_score(y_true, y_pred_all_negative):.4f}")   # 0.9900
print(f"精确率:   {precision_score(y_true, y_pred_all_negative):.4f}")  # 0.0（零除警告）
print(f"召回率:   {recall_score(y_true, y_pred_all_negative):.4f}")     # 0.0
print(f"F1 分数:  {f1_score(y_true, y_pred_all_negative):.4f}")         # 0.0
```

准确率高达 99%，但精确率、召回率、F1 全部为 0——这才是模型真实水平的反映。

## 8. 何时该用、何时不该用准确率

### 适合使用准确率的场景

- 各类别样本数量**大致均衡**；
- 所有类别的**分类代价相同**（漏报和误报的业务损失相等）；
- 需要一个**全局直观指标**向非技术人员展示模型整体表现。

### 不适合使用准确率的场景

| 场景 | 问题 | 替代指标 |
| --- | --- | --- |
| 类别严重不平衡（如欺诈检测、罕见病诊断） | 高准确率可能源于"全猜多数类" | Precision / Recall / F1 |
| 某些类别的误判代价极高（如癌症筛查漏诊） | 准确率无法区分误报和漏报 | Recall（关注漏报）或 Precision（关注误报） |
| 多分类中关注某一特定类别 | 准确率给出全局平均，掩盖了特定类别的表现 | 宏平均 F1 / 加权 F1 |

### 选择合适的评分指标

```python
from sklearn.metrics import get_scorer_names

# 查看 sklearn 中所有可用的分类评分指标
print(get_scorer_names())
```

常用替代指标速查：

| 指标 | scoring 参数 | 关注点 | 适用场景 |
| --- | --- | --- | --- |
| 准确率 | `'accuracy'` | 整体正确率 | 类别均衡 |
| 精确率 | `'precision'` / `'precision_macro'` | 预测为正的样本中有多少真正为正 | 减少误报（如垃圾邮件拦截） |
| 召回率 | `'recall'` / `'recall_macro'` | 真实正类中有多少被找出来 | 减少漏报（如疾病筛查） |
| F1 | `'f1'` / `'f1_macro'` | 精确率与召回率的调和平均 | 需要兼顾两者 |
| ROC AUC | `'roc_auc'` | 排序能力 | 类别不均衡时的排序任务 |

## 9. accuracy_score 源码解析

`accuracy_score` 的实现非常简洁，理解其源码有助于掌握它的本质：

```python
# sklearn.metrics._classification 中的简化逻辑
def _accuracy_score(y_true, y_pred, normalize, sample_weight):
    # 1. 计算每个样本是否正确
    score = (y_true == y_pred)  # bool 数组

    # 2. 如果有样本权重，乘上权重
    if sample_weight is not None:
        score = score * sample_weight

    # 3. 聚合
    if normalize:
        return score.sum() / (sample_weight.sum() if sample_weight is not None else len(y_true))
    else:
        return score.sum()
```

核心就是 `y_true == y_pred` 然后取均值（或加权均值）。

## 10. 手动实现

脱离 sklearn，一行代码即可实现基本的准确率计算：

```python
def my_accuracy_score(y_true, y_pred):
    """手动实现准确率计算"""
    correct = sum(t == p for t, p in zip(y_true, y_pred))
    return correct / len(y_true)

# 或者用 numpy
import numpy as np
def my_accuracy_score_np(y_true, y_pred):
    return np.mean(np.array(y_true) == np.array(y_pred))

# 验证
from sklearn.metrics import accuracy_score
y_true = [0, 1, 1, 0, 1]
y_pred = [0, 1, 0, 0, 1]

print(my_accuracy_score(y_true, y_pred))        # 0.8
print(accuracy_score(y_true, y_pred))           # 0.8
```

## 11. 小结

- `accuracy_score` 是分类任务中最直观的评估指标，计算简单且易于理解；
- 通过 `normalize` 参数可切换比例或计数模式，通过 `sample_weight` 可为不同样本赋予不同权重；
- 准确率与混淆矩阵直接关联——`Accuracy = (TP + TN) / Total`；
- **最大陷阱**：类别不平衡时高准确率具有欺骗性，必须结合 Precision、Recall、F1 等指标综合评估；
- 决策原则：各类别均衡且代价相等时用准确率，否则根据业务关注点选择更适合的指标。
