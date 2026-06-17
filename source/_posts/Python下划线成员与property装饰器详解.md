---
title: Python 下划线成员与 property 装饰器详解
date: 2026-06-17
updated: 2026-06-17
tags: [Python, 面向对象, 装饰器]
categories: Python
keywords: Python,下划线,property,getter,setter,deleter,名称改写,name mangling,私有成员
description: 深入解析 Python 中各类下划线命名约定的含义与使用场景，以及 property 装饰器如何实现优雅的 getter/setter 控制。
top_img:
comments: true
cover: https://picsum.photos/id/180/800/450
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
copyright_author_href:
copyright_url:
copyright_info:
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
---

## 前言

Python 中没有 `public`、`private`、`protected` 这样的访问控制关键字。取而代之的，是一套基于下划线的**约定**和**名称改写机制**。这套规则看似简单，但细节很多——单下划线、双下划线、前后双下划线，各自有不同的含义。而 `property` 装饰器则提供了另一种优雅的属性访问控制方式，让 Python 类的接口设计更加灵活。本文将这两块内容合并讲解，厘清所有容易混淆的地方。

## 下划线命名规则总览

Python 中共有五种与下划线相关的命名模式：

| 模式 | 示例 | 含义 |
|------|------|------|
| 单前导下划线 | `_name` | 约定：内部使用，不应外部访问 |
| 单结尾下划线 | `name_` | 避免与 Python 关键字冲突 |
| 双前导下划线 | `__name` | 名称改写（name mangling） |
| 双前导+双结尾下划线 | `__name__` | 魔法方法/属性，Python 保留 |
| 裸下划线 | `_` | 临时或不重要的变量 |

下面逐一展开。

## 单前导下划线：`_name`

这是 Python 中最常见的"私有"约定。在模块、类或函数内部，以 `_` 开头的名称表示"这是内部实现细节，外部不应直接访问"。

### 在模块中

```python
# mymodule.py
def public_api():
    """对外开放的接口"""
    return _internal_helper()

def _internal_helper():
    """内部辅助函数，不对外暴露"""
    return "done"
```

使用 `from mymodule import *` 时，以下划线开头的名称**不会被导入**（除非在 `__all__` 列表中显式声明）。

```python
from mymodule import *

public_api()        # ✅ 可用
_internal_helper()  # ❌ NameError: name '_internal_helper' is not defined
```

### 在类中

```python
class Person:
    def __init__(self, name, age):
        self._name = name   # 约定：内部属性，不建议外部直接读写
        self._age = age     # 但 Python 本身不做任何强制

p = Person("张三", 25)
print(p._name)   # 仍然可以访问，但 IDE 会给出警告
```

再次强调：**这仅仅是约定**，Python 解释器不会阻止你访问。它表达的是开发者"请勿直接使用"的意图。

## 单结尾下划线：`name_`

当变量名与 Python 关键字冲突时，在末尾加一个下划线来规避：

```python
class Person:
    def __init__(self, name, class_):   # class 是关键字，用 class_ 代替
        self.name = name
        self.class_ = class_

def process(type_, list_):             # type 和 list 是内置函数名
    print(type(list_))
```

常见场景：`class_`、`type_`、`list_`、`dict_`、`id_`、`filter_`。

## 双前导下划线：`__name` 与名称改写

这是 Python 中**唯一有实际语法效果**的下划线模式。当一个类属性以 `__` 开头（不以 `__` 结尾）时，解释器会自动触发**名称改写（Name Mangling）**——将该属性名改写为 `_类名__属性名` 的形式。

### 基本示例

```python
class Dog:
    def __init__(self, name):
        self.__name = name      # 实际存储为 _Dog__name

    def __bark(self):           # 实际名称为 _Dog__bark
        print(f"{self.__name} 在叫")

d = Dog("旺财")
# print(d.__name)              # AttributeError: 'Dog' object has no attribute '__name'
print(d._Dog__name)            # 旺财  ← 还是能访问，只是改了名字
d._Dog__bark()                 # 旺财 在叫
```

### 为什么要这么设计

名称改写的目的是**防止子类意外覆盖父类的"私有"属性**，而不是做安全控制。

```python
class Animal:
    def __init__(self):
        self.__id = 0          # 实际是 _Animal__id

class Cat(Animal):
    def __init__(self):
        super().__init__()
        self.__id = 100        # 实际是 _Cat__id，与父类的 _Animal__id 完全独立

c = Cat()
print(c._Animal__id)  # 0
print(c._Cat__id)     # 100
```

两个 `__id` 互不干扰——这正是名称改写的价值所在。如果没有改写机制，子类的 `__id` 会直接覆盖父类的 `__id`。

### 改写规则细节

- 只对类内部定义的名称生效，**事后赋值不会触发改写**
- 只改写双下划线开头、**不以双下划线结尾**的名称（`__init__` 等魔法方法不受影响）
- 改写后名称就是 `_类名__属性名`，规则简单且可预测

```python
class Test:
    def __init__(self):
        self.__var = 1   # 定义时改写

t = Test()
t.__other = 2            # 事后赋值不改写！
print(t.__other)         # 2 —— 可以直接访问
```

### 常见误区

很多教程说 `__name` 是"私有变量，外部无法访问"——这在 Python 中并不准确。改写只是换了名字，目的从来不是安全或权限控制。

```python
class Secret:
    def __init__(self):
        self.__password = "123456"

s = Secret()
s._Secret__password = "hacked"  # 仍然可以访问和修改
```

如果 Guido van Rossum 想要真正的私有变量，他会设计一个访问控制机制，而不是一个改名规则。

## 双前导+双结尾下划线：`__name__`

这类名称是 Python 语言保留的**魔法方法/属性**（也称 dunder methods），由 Python 解释器调用。例如 `__init__`、`__str__`、`__repr__`、`__len__`、`__call__` 等。

**绝对不要自定义以 `__` 开头和结尾的名称**，以免与 Python 的语言机制或未来版本冲突。

```python
class Person:
    def __init__(self, name):     # Python 保留的魔法方法
        self.name = name

    def __str__(self):            # Python 保留的魔法方法
        return f"Person({self.name})"

    # ❌ 不要这样做
    # def __my_custom_dunder__(self):
    #     pass
```

## 裸下划线：`_`

一个单独的 `_` 通常用作"不关心这个值"的占位符：

```python
# 循环中不关心索引
for _ in range(5):
    print("Hello")

# 解包时不关心某些值
name, _, age = ("张三", "男", 28)   # 不关心中间值

# 忽略异常对象
try:
    1 / 0
except ZeroDivisionError as _:
    print("除零错误")
```

在 Python REPL 交互环境中，`_` 还保存着上一次表达式的结果值。

## property 装饰器

理解了前导下划线的约定后，自然会产生一个需求：**如果我约定 `_name` 是内部属性，那外部如何安全地读写它？** 这就是 `property` 要解决的问题。

`property` 允许你将属性的访问（读、写、删除）包装为方法调用，从而实现：
- 属性值的校验
- 计算属性（衍生值）
- 向后兼容（将原本直接暴露的属性升级为带逻辑的方法，外部调用代码无需改动）

### 基本用法

```python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        """getter：获取姓名"""
        return self._name

    @name.setter
    def name(self, value):
        """setter：设置姓名时可加校验"""
        if not isinstance(value, str):
            raise TypeError("姓名必须是字符串")
        if not value.strip():
            raise ValueError("姓名不能为空")
        self._name = value

    @name.deleter
    def name(self):
        """deleter：删除姓名"""
        print(f"正在删除 {self._name}")
        del self._name

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if value < 0 or value > 150:
            raise ValueError(f"年龄 {value} 不合法")
        self._age = value
```

使用方式与普通属性完全一致：

```python
p = Person("张三", 25)

# 读 —— 像访问普通属性一样，实际调用 getter 方法
print(p.name)   # 张三

# 写 —— 像赋值普通属性一样，实际调用 setter 方法
p.name = "李四"  # ✅
# p.name = ""    # ❌ ValueError: 姓名不能为空

# 删 —— 使用 del 关键字，实际调用 deleter 方法
del p.name
```

### 为什么不用 Java 风格的 get/set 方法

```python
# ❌ Python 中不推荐
class Person:
    def get_name(self):
        return self._name

    def set_name(self, value):
        self._name = value

p = Person()
print(p.get_name())      # 啰嗦
p.set_name("张三")        # 不自然
```

`property` 让它变成了这样：

```python
# ✅ Python 写法
p.name           # 干净
p.name = "张三"   # 自然
```

对于一个已有类，如果你最初直接暴露了 `self.name`，后来需要加校验逻辑，可以直接改为 property，**外部调用代码一行都不用改**——这正是 property 设计的核心优势。

### 只读属性

只提供 getter，不提供 setter 即可：

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def area(self):
        """面积是计算属性，不应被赋值"""
        return 3.14159 * self._radius ** 2

c = Circle(5)
print(c.area)    # 78.53975
# c.area = 100   # ❌ AttributeError: can't set attribute
```

### 旧式 property 写法

`@property` 装饰器是 Python 2.6+ 引入的语法糖。在此之前，`property` 是这样用的：

```python
class Person:
    def __init__(self, name):
        self._name = name

    def _get_name(self):
        return self._name

    def _set_name(self, value):
        self._name = value

    def _del_name(self):
        del self._name

    name = property(_get_name, _set_name, _del_name, "文档字符串")
```

语法糖版本可读性好太多，旧式写法仅作了解即可。

## property 实用场景

### 场景一：懒加载/缓存

```python
class DataLoader:
    def __init__(self, filepath):
        self._filepath = filepath
        self._data = None

    @property
    def data(self):
        if self._data is None:
            print("首次读取文件...")
            # 模拟读取大文件
            self._data = {"key": "从文件加载的内容"}
        return self._data

dl = DataLoader("data.json")
print(dl.data)  # 首次读取文件...  {"key": "从文件加载的内容"}
print(dl.data)  # 不打印日志，直接返回缓存
```

### 场景二：反向兼容的升级

```python
# v1.0：直接暴露属性
class User:
    def __init__(self, email):
        self.email = email

# v2.0：需要加校验，改用 property，旧代码无需修改
class User:
    def __init__(self, email):
        self._email = None
        self.email = email    # 通过 setter 赋值，自动校验

    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, value):
        if "@" not in value:
            raise ValueError("邮箱格式不正确")
        self._email = value
```

外部调用 `user.email = "xxx"` 的代码无需任何改动，升级完全透明。

### 场景三：日志追踪

```python
class Config:
    def __init__(self):
        self._debug = False

    @property
    def debug(self):
        return self._debug

    @debug.setter
    def debug(self, value):
        print(f"[Config] debug 从 {self._debug} 变为 {value}")
        self._debug = value
```

## 下划线与 property 的结合

回到最初的问题：在 Python 中如何设计类的接口？

| 做法 | 示例 | 说明 |
|------|------|------|
| 内部存储使用 `_name` | `self._name = name` | 约定：这是内部属性 |
| property 提供受控访问 | `@property` + `@name.setter` | 外部通过 property 读写 |
| 真正不希望子类覆盖的用 `__name` | `self.__id = id` | 触发名称改写，防止子类意外覆盖 |

一个完整的例子：

```python
class BankAccount:
    def __init__(self, owner, balance):
        self._owner = owner           # 保护：子类和外部可访问，但不鼓励直接读写
        self.__balance = balance      # 私有：通过名称改写防止子类覆盖

    @property
    def owner(self):
        return self._owner

    @property
    def balance(self):
        """余额只读"""
        return self.__balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("存款金额必须大于 0")
        self.__balance += amount
        return self.__balance

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("取款金额必须大于 0")
        if amount > self.__balance:
            raise ValueError("余额不足")
        self.__balance -= amount
        return self.__balance
```

```python
acc = BankAccount("张三", 1000)
print(acc.owner)      # 张三           —— 通过 property 读取
print(acc.balance)    # 1000           —— 只读 property
# acc.balance = 5000  # ❌ AttributeError —— 没有 setter

acc.deposit(500)      # ✅ 通过方法修改
acc.withdraw(200)     # ✅ 通过方法修改
print(acc.balance)    # 1300
```

## 总结

- `_name`：约定俗成的"内部使用"，from module import * 不导入，但 Python 不做强制
- `name_`：和后置关键字冲突时用的替代名
- `__name`：名称改写为 `_类名__name`，用于防止子类意外覆盖
- `__name__`：Python 保留的魔法方法，绝不能自定义
- `_`：临时占位符，表示"我不关心这个值"
- `property`：Python 风格的 getter/setter，保持 `obj.attr` 的自然语法，同时允许在访问背后执行逻辑
- 典型模式：内部用 `_name` 存储数据，外部通过 `@property` 提供受控访问
