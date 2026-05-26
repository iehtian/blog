---
title: Pydantic 学习笔记
date: 2026-05-26
updated: 2026-05-26
tags: [Python, Pydantic, 数据验证]
categories: Python
keywords: Pydantic, BaseModel, Field, field_validator, 数据验证, Python
description: Pydantic 学习笔记，涵盖 BaseModel、Field、验证器、嵌套模型、序列化等核心用法
toc: true
toc_number: true
copyright: true
---

# Pydantic 学习笔记

## 1. BaseModel 基本用法

```python
from pydantic import BaseModel, EmailStr, Field, field_validator
from datetime import datetime
from typing import Optional

class User(BaseModel):
    id: int
    name: str
    email: EmailStr
    age: int = Field(ge=0, le=150, description="用户年龄")
    signup_time: datetime = Field(default_factory=datetime.now)
    nickname: Optional[str] = None  # 可选字段

user = User(id=1, name="Alice", email="alice@example.com", age="25")
# age="25" → 自动类型转换为 int 25
```

- 继承 `BaseModel` 即可获得自动验证和类型转换
- 验证失败抛出 `ValidationError`
- 类型注解 `name: str` 不是类属性也不是实例属性，只是注解；Pydantic 元类读取注解后在 `__init__` 中动态创建实例属性

## 2. Field 字段配置

`Field` 提供约束验证和默认值：

```python
class Product(BaseModel):
    name: str
    price: float = Field(gt=0)                    # 必须大于 0，必填
    discount: float = Field(ge=0, le=1, default=0.0)  # 0~1 之间，默认 0.0
```

### 常见约束参数

| 参数 | 含义 |
|------|------|
| `gt` | 大于 |
| `ge` | 大于等于 |
| `lt` | 小于 |
| `le` | 小于等于 |
| `default` | 默认值 |
| `description` | 字段描述（用于生成 JSON Schema） |

## 3. @field_validator 自定义验证器

```python
class Product(BaseModel):
    name: str
    price: float = Field(gt=0)

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("name cannot be empty")
        return v.strip()  # 可以修改值（数据清洗）
```

- `@field_validator("name")` 声明验证哪个字段
- `@classmethod` 必须加 — 验证器在实例创建之前运行，实例还不存在，没有 `self`
- `cls` 可用于继承场景，自动指向子类
- `v` 是用户传入的原始值
- 可以做验证（抛异常）+ 数据清洗（返回修改后的值）
- `Field` 只能做简单数值约束，`@field_validator` 可以写任意复杂逻辑

### 为什么用 @classmethod 而不是实例方法？

Pydantic 创建实例的流程：

```text
1. 接收原始输入值
2. 逐个字段验证 ← 验证器在这里执行，实例还不存在，没有 self
3. 所有字段验证通过
4. 创建实例 ← 到这里才有 self
```

## 4. 嵌套模型与列表

```python
class Address(BaseModel):
    city: str
    street: str

class Employee(BaseModel):
    name: str
    addresses: list[Address]
```

Pydantic 运行时会自动将 dict 转换为嵌套模型对象：

```python
emp = Employee(
    name="Bob",
    addresses=[{"city": "Beijing", "street": "Chaoyang Rd"}],  # dict 自动转 Address
)
```

但静态类型检查器（Pyright/Pylance）不识别这个转换，会报类型错误。消除方法：

```python
# 方法 1：直接传 Address 对象
addresses=[Address(city="Beijing", street="Chaoyang Rd")]

# 方法 2：用 model_validate
Employee.model_validate({"name": "Bob", "addresses": [...]})
```

## 5. 类属性 vs 实例属性

### 普通类

```python
class Product:
    tax_rate = 0.1          # 类属性：所有实例共享

    def __init__(self, name):
        self.name = name    # 实例属性：每个实例独立
```

- 类属性通过类名修改（`Product.tax_rate = 0.2`）会影响所有实例
- 通过实例修改（`p.tax_rate = 0.3`）其实是新增实例属性，遮蔽了类属性，不影响其他实例

### Pydantic 模型

```python
from typing import ClassVar

class Product(BaseModel):
    tax_rate: ClassVar[float] = 0.1   # 类属性
    name: str                          # 实例属性（字段）
```

| | `tax_rate: float = 0.1` | `tax_rate: ClassVar[float] = 0.1` |
|---|---|---|
| 类型 | 实例属性（字段） | 类属性 |
| 每个实例独立存储 | ✅ | ❌ 共享一份 |
| 参与验证 | ✅ | ❌ |
| 出现在 `model_dump()` | ✅ | ❌ |
| 可在 `__init__` 传入 | ✅ | ❌ |

## 6. Python 方法类型

| | 第一个参数 | 访问实例属性 | 访问类属性 |
|---|---|---|---|
| 实例方法 | `self` | ✅ | ✅ |
| 类方法 `@classmethod` | `cls` | ❌ | ✅ |
| 静态方法 `@staticmethod` | 无 | ❌ | ❌（只能通过类名） |

## 7. 字符串 strip() 方法

去除首尾空白字符（空格、制表符、换行符），**不影响中间空格**：

```python
"  Phone  ".strip()    # "Phone"
"\t Hello \n".strip()  # "Hello"
"  ".strip()           # ""
"Hel lo".strip()       # "Hel lo"  中间空格保留
```

变体：`lstrip()` 去左边，`rstrip()` 去右边。去所有空格用 `replace(" ", "")`。

## 8. 序列化与 Schema

### model_dump() — 导出数据

```python
user = User(id=1, name="Alice", email="a@b.com", age=25, nickname=None)

user.model_dump()                     # {"id": 1, "name": "Alice", ...}
user.model_dump(include={"id", "name"})  # 只导出指定字段
user.model_dump(exclude_none=True)    # 排除 None 字段
```

### model_json_schema() — 导出结构描述

```python
User.model_json_schema()
# {"title": "User", "type": "object", "properties": {"id": {"type": "integer"}, ...}}
```

| | `model_dump()` | `model_json_schema()` |
|---|---|---|
| 输出 | 具体数据（dict） | 结构描述（字段、类型、约束） |
| 调用方式 | 实例调用 | 类调用 |
| 用途 | 序列化、传 API | 文档、表单校验、OpenAPI |
| 和实例有关？ | ✅ 不同实例不同 | ❌ 同一个类始终一样 |
