---
title: GridSearchCV——网格搜索与交叉验证详解
date: 2026-06-19
updated: 2026-06-19
tags: [机器学习, 超参数调优, sklearn, 交叉验证, 网格搜索, GridSearchCV, 模型选择]
categories: 机器学习
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 深入讲解 sklearn 中 GridSearchCV 的原理、核心参数、cv_results_ 结果分析及常见避坑指南
cover: https://picsum.photos/id/180/800/450
keywords: GridSearchCV,网格搜索,交叉验证,超参数调优,sklearn,机器学习,cv_results_,参数网格
katex: true
---

# GridSearchCV——网格搜索与交叉验证详解

## 1. 什么是 GridSearchCV

`GridSearchCV` 是 sklearn 中 `model_selection` 模块提供的超参数调优工具，其名称揭示了两个核心步骤：

- **Grid Search（网格搜索）**：穷举参数组合——将各参数的候选值做笛卡尔积，形成一张"参数网格"，逐一尝试；
- **CV（Cross-Validation，交叉验证）**：对每一组参数组合，使用 K 折交叉验证评估模型性能，取平均得分作为该组合的评价指标。

最终选出交叉验证得分最高的那组参数，并在全部训练数据上重新训练出最终模型。

## 2. 核心原理

### 3.1 网格穷举

假设要调优 SVM 的两个参数：`C` 和 `gamma`：

```python
param_grid = {
    'C':     [0.1, 1, 10],
    'gamma': [0.01, 0.1, 1],
}
```

笛卡尔积产生 $3 \times 3 = 9$ 组参数组合：

| 编号 | C | gamma |
| --- | --- | --- |
| 1 | 0.1 | 0.01 |
| 2 | 0.1 | 0.1 |
| 3 | 0.1 | 1 |
| 4 | 1 | 0.01 |
| ... | ... | ... |
| 9 | 10 | 1 |

GridSearchCV 会逐一训练并评估这 9 组参数。

### 3.2 交叉验证评估

对每一组参数，使用 K 折交叉验证计算得分：

1. 将训练集等分为 $K$ 份；
2. 依次以其中 $K-1$ 份训练、1 份验证，重复 $K$ 次；
3. 取 $K$ 次得分的平均值作为该组参数的最终得分。

```
折数 K=5 示意：

折 1: [验证] [训练] [训练] [训练] [训练]  → score₁
折 2: [训练] [验证] [训练] [训练] [训练]  → score₂
折 3: [训练] [训练] [验证] [训练] [训练]  → score₃
折 4: [训练] [训练] [训练] [验证] [训练]  → score₄
折 5: [训练] [训练] [训练] [训练] [验证]  → score₅

最终得分 = mean(score₁, score₂, score₃, score₄, score₅)
```

### 3.3 总训练次数

若参数组合数为 $N$，交叉验证折数为 $K$，则总训练次数为：

$$
\text{Total fits} = N \times K
$$

上例中 $9 \times 5 = 45$ 次训练。若参数维度增加，训练次数会**指数级增长**——这是网格搜索的主要瓶颈。

## 3. 核心参数详解

```python
from sklearn.model_selection import GridSearchCV

GridSearchCV(
    estimator,        # 基础模型（如 SVC()、RandomForestClassifier()）
    param_grid,       # 参数网格：dict 或 list of dict
    scoring=None,     # 评分指标：'accuracy'、'f1'、'roc_auc' 等
    n_jobs=None,      # 并行任务数，-1 表示使用全部 CPU
    cv=5,             # 交叉验证折数（默认 5）
    verbose=0,        # 日志详细程度，越大越详细
    refit=True,       # 是否用最优参数在全量数据上重新训练
    return_train_score=False,  # 是否返回训练集得分
)
```

### 4.1 `estimator` —— 基础模型

传入一个 sklearn 兼容的 estimator 实例。GridSearchCV 内部会克隆该对象用于各参数组合的训练。

```python
from sklearn.svm import SVC
estimator = SVC(kernel='rbf')  # 固定 kernel，仅搜索 C 和 gamma
```

### 4.2 `param_grid` —— 参数网格

支持两种形式：

**形式一：单个字典**（笛卡尔积搜索）：

```python
param_grid = {
    'C':     [1, 10, 100],
    'gamma': [0.01, 0.001],
    'kernel': ['rbf', 'linear'],
}
# 共 3 × 2 × 2 = 12 组
```

**形式二：字典列表**（分段搜索，不取笛卡尔积）：

```python
param_grid = [
    {'kernel': ['rbf'],  'C': [1, 10], 'gamma': [0.01, 0.001]},   # 4 组
    {'kernel': ['linear'], 'C': [1, 10]},                          # 2 组
]
# 共 6 组，linear 不与 gamma 组合（linear 核无 gamma 参数）
```

第二种形式在搜索不同模型或不同参数子集时非常有用，避免了无意义的组合。

### 4.3 `scoring` —— 评分指标

指定用于评估和选择最优参数的指标。可以是字符串或自定义 scorer。

常用字符串：

| 任务 | 指标 | scoring 参数值 |
| --- | --- | --- |
| 分类 | 准确率 | `'accuracy'` |
| 分类 | F1 分数 | `'f1'`、`'f1_macro'`、`'f1_weighted'` |
| 分类 | AUC | `'roc_auc'` |
| 回归 | 均方误差 | `'neg_mean_squared_error'` |
| 回归 | R² | `'r2'` |

注意：sklearn 约定 scorer 值越大越好，因此 MSE 等误差指标会加 `neg_` 前缀取负数。

支持多个评分指标：

```python
scoring = ['accuracy', 'f1_weighted', 'roc_auc_ovr']
# refit 需指定以哪个指标选最优参数
GridSearchCV(..., scoring=scoring, refit='f1_weighted')
```

### 4.4 `cv` —— 交叉验证策略

`cv` 参数不仅可以是整数，还可以传入各种 splitter：

```python
from sklearn.model_selection import StratifiedKFold, TimeSeriesSplit, GroupKFold

# 分层 K 折（分类任务推荐，保持各类别比例）
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

# 时间序列划分（时间序列数据专用，不打乱顺序）
cv = TimeSeriesSplit(n_splits=5)

# 分组 K 折（相同 group 不会被拆分到不同折）
cv = GroupKFold(n_splits=5)
```

### 4.5 `fit` —— 执行搜索与训练

`fit(X, y, **fit_params)` 是 GridSearchCV 的核心方法，调用它才会真正启动网格搜索。其内部流程为：

1. 遍历 `param_grid` 中的每一组参数组合；
2. 对每组参数，执行 $K$ 折交叉验证，记录每折的得分与训练耗时；
3. 汇总所有结果到 `cv_results_`；
4. 若 `refit=True`，用 `best_params_` 在**全部 `X`** 上再训练一次，得到 `best_estimator_`。

```python
grid = GridSearchCV(SVC(), param_grid, cv=5)
grid.fit(X_train, y_train)  # 返回 grid 自身，触发全部搜索流程
```

`**fit_params` 会透传给底层 estimator 的 `fit()` 方法。典型场景是某些模型需要额外参数：

```python
# XGBoost 的 eval_set、early_stopping_rounds 等
grid.fit(
    X_train, y_train,
    eval_set=[(X_val, y_val)],
    verbose=False,
)
```

`fit()` 返回 `self`（即 GridSearchCV 实例本身），因此支持链式调用，也意味着调用后可以直接访问 `grid.best_params_` 等属性。

### 4.6 `refit` —— 重训练

默认 `refit=True`，在 `fit()` 的网格搜索阶段找到最优参数后，会用**全部训练数据**重新训练一次模型，得到 `best_estimator_`，可直接用于预测。

若使用了多指标评分，可指定 `refit='f1_weighted'` 告知以哪个指标选出最优参数。

## 4. 搜索结果分析——`fit` 后的属性与 `cv_results_` 详解

`GridSearchCV.fit()` 完成后，搜索对象上会挂载一系列以 `_` 结尾的属性，供后续分析和使用。以下逐一说明每个属性的含义、类型及访问方式。

### 4.1 顶层属性 —— 直接挂载在 `grid` 对象上

以下属性在 `fit()` 后直接挂载在 GridSearchCV 实例上，通过 `grid.属性名` 访问：

| 属性 | 类型 | 访问方式 | 说明 |
| --- | --- | --- | --- |
| `best_params_` | `dict` | `grid.best_params_` | 交叉验证得分最高的那组参数 |
| `best_score_` | `float` | `grid.best_score_` | 最优参数对应的交叉验证平均得分 |
| `best_estimator_` | estimator 对象 | `grid.best_estimator_` | 用最优参数在全量训练数据上重新训练好的模型，可直接 `.predict()` |
| `best_index_` | `int` | `grid.best_index_` | 最优参数在 `cv_results_` 各数组中的索引位置 |
| `cv_results_` | `dict of ndarray` | `grid.cv_results_` | **所有参数组合的详细结果，其内部结构见 4.2 节** |
| `scorer_` | scorer 对象 | `grid.scorer_` | 实际使用的评分器 |
| `n_splits_` | `int` | `grid.n_splits_` | 实际执行的交叉验证折数 |
| `refit_time_` | `float` | `grid.refit_time_` | 全量 refit 所消耗的时间（秒） |
| `multimetric_` | `bool` | `grid.multimetric_` | 是否使用了多指标评分 |

**访问示例**：

```python
grid = GridSearchCV(SVC(), param_grid, cv=5).fit(X_train, y_train)

# 顶层属性直接用 . 访问
print(grid.best_params_)       # {'C': 10, 'gamma': 0.01}
print(grid.best_score_)        # 0.9732
print(grid.best_index_)        # 5
y_pred = grid.best_estimator_.predict(X_test)   # 直接预测
```

### 4.2 `cv_results_` 内部结构 —— 它本身是一个 dict

下面按类别逐一说明 dict 中的 key。

#### 4.2.1 参数列 —— key 以 `param_` 为前缀

每个被搜索的参数在 dict 中对应一个 key，命名规则为 `param_` + 参数名。这些 key 的值是长度为 $N$（参数组合总数）的 numpy 数组，记录了该组参数组合的候选值。

```python
# 例：param_grid = {'C': [0.1, 1], 'gamma': [0.01, 0.1]}
# 共 2×2=4 组，grid.cv_results_ 中包含：

grid.cv_results_['param_C']     # array([0.1, 0.1, 1. , 1. ])
grid.cv_results_['param_gamma'] # array([0.01, 0.1, 0.01, 0.1])
```

当参数以双下划线嵌套时（如 Pipeline），key 名原样保留双下划线：

```python
# param_grid = {'svm__C': [0.1, 1]}
grid.cv_results_['param_svm__C']  # key 名保留双下划线
```

#### 4.2.2 得分列 —— `mean_test_score`、`std_test_score`、`rank_test_score` 与每折明细

这些 key 的数量取决于 `scoring` 的配置和 `return_train_score` 的值。

**始终存在的 key**：

| dict 中的 key | 含义 |
| --- | --- |
| `mean_test_score` | 每组参数在 $K$ 折验证集上的平均得分 |
| `std_test_score` | 每组参数在 $K$ 折验证集上得分的标准差 |
| `rank_test_score` | 按 `mean_test_score` 降序排名（1 = 最优） |

**每折明细 key**（`cv=K` 时有 $K$ 个）：

| dict 中的 key | 含义 |
| --- | --- |
| `split0_test_score` | 第 1 折验证集得分 |
| `split1_test_score` | 第 2 折验证集得分 |
| ... | ... |
| `split{K-1}_test_score` | 第 K 折验证集得分 |

**仅当 `return_train_score=True` 时才出现的 key**：

| dict 中的 key | 含义 |
| --- | --- |
| `mean_train_score` | 每组参数在 $K$ 折训练集上的平均得分 |
| `std_train_score` | 训练集得分的标准差 |
| `split0_train_score` ... | 每折训练集得分明细 |

**多指标评分时**（`scoring=['accuracy', 'f1']`），以上 key 会按指标名展开：

| dict 中的 key | 含义 |
| --- | --- |
| `mean_test_accuracy` | 准确率的交叉验证均值 |
| `mean_test_f1` | F1 的交叉验证均值 |
| `rank_test_accuracy` | 按准确率排名 |
| `rank_test_f1` | 按 F1 排名 |

```python
# 示例：访问 dict 中的具体 key
grid = GridSearchCV(SVC(), param_grid, cv=5, scoring='accuracy').fit(X, y)
res = grid.cv_results_

print("参数组合数:", len(res['mean_test_score']))       # dict 的 key 访问
print("第 1 组在第 3 折的得分:", res['split2_test_score'][0])
print("第 5 组平均得分:", res['mean_test_score'][4])
print("第 5 组标准差:", res['std_test_score'][4])
```

#### 4.2.3 时间列

| dict 中的 key | 含义 |
| --- | --- |
| `mean_fit_time` | 每组参数平均训练耗时（秒） |
| `std_fit_time` | 训练耗时标准差 |
| `mean_score_time` | 每组参数平均评分耗时（秒） |
| `std_score_time` | 评分耗时标准差 |

### 4.3 `cv_results_` 的典型分析操作

#### 4.3.1 转为 DataFrame 并按排名查看

```python
import pandas as pd

df = pd.DataFrame(grid.cv_results_)

# 按排名显示所有列
print(df.sort_values('rank_test_score').to_string())
```

#### 4.3.2 按自定义条件筛选

```python
# 筛出 C=10 且平均得分 > 0.95 的组合
mask = (df['param_C'] == 10) & (df['mean_test_score'] > 0.95)
print(df[mask][['param_C', 'param_gamma', 'mean_test_score', 'std_test_score']])
```

#### 4.3.3 从中提取最优参数

```python
res = grid.cv_results_

# 方法 1：通过 best_index_ 定位
best_idx = grid.best_index_
print("最优参数:", {k: v[best_idx] for k, v in res.items() if k.startswith('param_')})
print("最优得分:", res['mean_test_score'][best_idx])

# 方法 2：直接使用顶层属性（推荐）
print(grid.best_params_)
print(grid.best_score_)
```

#### 4.3.4 诊断过拟合：对比训练集与测试集得分

```python
grid = GridSearchCV(
    SVC(), param_grid, cv=5,
    return_train_score=True      # 必须开启
)
grid.fit(X_train, y_train)

df = pd.DataFrame(grid.cv_results_)

# 训练集 vs 验证集差距
df['gap'] = df['mean_train_score'] - df['mean_test_score']
print(df.sort_values('rank_test_score')[
    ['param_C', 'param_gamma', 'mean_train_score', 'mean_test_score', 'gap']
].head(10))
```

若 `gap` 过大（如训练得分 0.99，验证得分仅 0.85），说明该组参数存在**过拟合**风险。

#### 4.3.5 可视化参数空间

```python
import matplotlib.pyplot as plt

df = pd.DataFrame(grid.cv_results_)

pivot = df.pivot_table(
    values='mean_test_score',
    index='param_C',
    columns='param_gamma',
)

plt.figure(figsize=(8, 6))
plt.imshow(pivot, cmap='viridis', aspect='auto')
plt.colorbar(label='Mean Test Score')
plt.xticks(range(len(pivot.columns)), pivot.columns)
plt.yticks(range(len(pivot.index)), pivot.index)
plt.xlabel('gamma')
plt.ylabel('C')
plt.title('GridSearchCV 得分热力图')
plt.show()
```

### 4.4 `best_estimator_` 深入使用

`best_estimator_` 是一个**已经训练好**的模型实例，可以直接调用其所有方法：

```python
best_model = grid.best_estimator_

# 预测
y_pred = best_model.predict(X_test)
y_proba = best_model.predict_proba(X_test)   # 分类器

# 查看模型自身的参数
print(best_model.get_params())

# 对于树模型，查看特征重要性
if hasattr(best_model, 'feature_importances_'):
    print(best_model.feature_importances_)

# 对于线性模型，查看系数
if hasattr(best_model, 'coef_'):
    print(best_model.coef_)
```

### 4.5 多指标评分下的属性行为

当 `scoring` 为列表或字典时，`best_score_` 和 `best_index_` 的行为取决于 `refit`：

```python
grid = GridSearchCV(
    SVC(), param_grid,
    scoring=['accuracy', 'f1_weighted'],
    refit='f1_weighted',  # 以 f1_weighted 为基准选出最优参数
    cv=5,
)
grid.fit(X_train, y_train)

# best_score_ 返回 refit 指定指标的得分
print(grid.best_score_)          # f1_weighted 的最优得分
print(grid.best_params_)         # f1_weighted 最高时对应的参数
print(grid.best_estimator_)      # 用该参数 refit 的模型

# cv_results_ 中同时包含两个指标的全部列
print(grid.cv_results_.keys())   # 包含 mean_test_accuracy, mean_test_f1_weighted 等
```

## 5. 常见问题与避坑指南

### 5.1 数据泄露

**错误做法**：在拆分训练集/测试集之前对全量数据做标准化，再用 GridSearchCV。

```python
# 错误！已导致数据泄露
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)              # 测试集信息泄露进了训练集
X_train, X_test = train_test_split(X_scaled, y)
grid.fit(X_train, y_train)
```

**正确做法**：使用 `Pipeline` 将预处理嵌入搜索流程。

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC()),
])

param_grid = {
    'svm__C':     [0.1, 1, 10],   # 注意双下划线前缀
    'svm__gamma': [0.01, 0.1, 1],
}

grid = GridSearchCV(pipe, param_grid, cv=5)
grid.fit(X_train, y_train)  # 每折交叉验证都会独立 fit scaler
```

使用 Pipeline 后，每个交叉验证折内的标准化只会用该折的训练数据来 `fit`，彻底杜绝数据泄露。

### 5.2 参数组合爆炸

假设 5 个参数，每个 5 个候选值，`cv=5`：

$$
\text{总训练次数} = 5^5 \times 5 = 15{,}625
$$

**应对策略**：

- 先用粗粒度大范围搜索，锁定范围后精细搜索；
- 用 `RandomizedSearchCV` 替代穷举；
- 用 `HalvingGridSearchCV` 加速；
- 基于经验缩小候选值范围。

### 5.3 评分指标选择不当

分类任务中，当类别不均衡时，`accuracy` 会高估模型表现——例如正样本仅占 1%，全预测负类也有 99% 准确率。此时应使用 `f1` 或 `roc_auc`。

```python
# 类别不均衡时
GridSearchCV(
    SVC(),
    param_grid,
    scoring='f1_weighted',  # 而非 'accuracy'
    cv=StratifiedKFold(n_splits=5),  # 保持折内类别比例
)
```

### 5.4 `best_score_` 与测试集得分的落差

`best_score_` 是交叉验证的平均得分，由于在参数搜索中选择了"最好"的那组，该值可能**略偏乐观**。最终应以独立测试集的结果为准。

### 5.5 `n_jobs` 与内存

`n_jobs=-1` 会并行运行，但每个任务会克隆一份 estimator 和数据。如果模型或数据很大，可能导致内存溢出。此时应适当减少并行数。

## 6. 嵌套交叉验证——无偏性能估计

当需要同时调参和估计模型泛化性能时，简单的 train/test split 会导致乐观偏差。**嵌套交叉验证**提供无偏的性能估计：

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold

# 外层交叉验证：评估泛化性能
outer_cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
# 内层交叉验证：调参
inner_cv = StratifiedKFold(n_splits=3, shuffle=True, random_state=42)

clf = GridSearchCV(
    SVC(), param_grid, cv=inner_cv
)

# 外层循环——每次用不同的测试折
scores = cross_val_score(clf, X, y, cv=outer_cv, scoring='accuracy')

print(f"嵌套交叉验证得分: {scores.mean():.4f} ± {scores.std():.4f}")
```

外层 K 折的每一折中，GridSearchCV 都会在内层做一次完整的网格搜索——计算开销很大，但结果更可靠。

## 7. 小结

- `GridSearchCV` 穷举参数组合并配合交叉验证评估，是寻找最优超参数的系统化方法；
- 参数组合数随维度指数增长，建议控制候选值数量或使用 `RandomizedSearchCV` 替代；
- 务必使用 `Pipeline` 将数据预处理嵌入搜索流程，避免交叉验证中的数据泄露；
- 类别不均衡时使用分层交叉验证和合适的评分指标（`f1`、`roc_auc`）；
- `best_score_` 可作为参考，但最终性能应以独立测试集为准；
- 对性能估计要求严格的场景，可使用嵌套交叉验证。
