---
title: Mermaid 图表语法与使用
date: 2026-08-14
updated: 2026-08-14
tags: [Mermaid, Markdown, 图表, 流程图, 时序图]
categories: 工具
keywords: Mermaid, 流程图, flowchart, 时序图, sequenceDiagram, 类图, classDiagram, 状态图, stateDiagram, ER图, erDiagram, 甘特图, gantt, 饼图, pie, Hexo, Butterfly
description: Mermaid 图表语法速查 — 在 Hexo Butterfly 中启用及流程图、时序图、类图、状态图、ER 图、甘特图、饼图的常用写法
cover: https://picsum.photos/id/310/800/450
comments: true
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

# Mermaid 图表语法与使用

Mermaid 是一种「文本转图表」的 DSL，用类 Markdown 的纯文本描述流程图、时序图、类图等，由前端 JS 渲染为 SVG。无需绘图工具，图表可随代码一起纳入版本管理。

## 1. 在 Butterfly 中启用

在 `_config.butterfly.yml` 中开启 Mermaid：

```yaml
mermaid:
  enable: true       # 全局启用（按需懒加载，无图表的页面不加载脚本）
  code_write: true   # 用 ```mermaid 代码块书写，false 则用 <div class="mermaid">
  theme:
    light: default   # 可选 default / forest / dark / neutral
    dark: dark
```

- `enable: true` 后，仅当页面存在 Mermaid 图时才加载渲染脚本，其余页面无额外开销
- `code_write: true` 用 ```` ```mermaid ```` 代码块书写；`false` 时改用 HTML 块 `<div class="mermaid">...</div>`
- `theme` 按亮/暗模式自动切换主题

## 2. 流程图 flowchart

```mermaid
flowchart TD
    A[开始] --> B{条件成立?}
    B -- 是 --> C[执行操作]
    B -- 否 --> D[结束]
    C --> D
```

- 方向关键字：`TD`（自上而下）、`LR`（左至右）、`BT`、`RL`
- 节点形状由括号决定：`[矩形]`、`{菱形}`、`(圆角)`、`((圆形))`、`[(数据库)]`
- 边：`-->` 实线箭头，`---` 实线，`-- 文本 -->` 带标签

## 3. 时序图 sequenceDiagram

```mermaid
sequenceDiagram
    participant U as 用户
    participant S as 服务器
    U->>S: 发起请求
    S-->>U: 返回响应
```

- `participant` 用 `as` 定义别名
- 实线箭头 `->>`，虚线箭头 `-->>`；`->` / `-->` 为无箭头版本

## 4. 类图 classDiagram

```mermaid
classDiagram
    class Animal {
        +String name
        +void eat()
    }
    class Dog {
        +void bark()
    }
    Animal <|-- Dog : 继承
```

- 成员可见性：`+` 公有、`-` 私有、`#` 保护
- 关系：`<|--` 继承、`*--` 组合、`o--` 聚合、`-->` 依赖

## 5. 状态图 stateDiagram

```mermaid
stateDiagram-v2
    [*] --> 待机
    待机 --> 运行 : 启动
    运行 --> 待机 : 停止
    运行 --> [*] : 退出
```

- `[*]` 表示起始/终止状态
- `状态A --> 状态B : 触发事件` 描述状态转移

## 6. ER 图 erDiagram

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ LINE-ITEM : contains
    CUSTOMER {
        int id
        string name
    }
```

- 关系基数：`||` 恰一、`o{` 零或多、`|{` 一或多、`o|` 零或一

## 7. 甘特图 gantt / 饼图 pie

```mermaid
gantt
    title 项目计划
    dateFormat YYYY-MM-DD
    section 开发
    需求分析 :a1, 2026-08-01, 5d
    编码     :a2, after a1, 10d
```

```mermaid
pie title 浏览器份额
    "Chrome" : 65
    "Safari" : 20
    "Edge" : 15
```

- 甘特图：`dateFormat` 声明日期格式，`section` 分组，任务用 `任务名 :id, 起止, 时长`
- 饼图：`pie title 标题`，数据项 `"名称" : 数值`，数值为占比

## 小结

- Mermaid 用纯文本描述图表，主流图类型均可表达
- Butterfly 下开启 `mermaid.enable` 并按需懒加载，`code_write` 决定书写方式
- 按图表类型记关键词：`flowchart`、`sequenceDiagram`、`classDiagram`、`stateDiagram-v2`、`erDiagram`、`gantt`、`pie`
