---
title: uv 使用命令指南
date: 2025-11-14
updated: 2025-11-14
tags: [uv, Python, 包管理, 虚拟环境, 依赖管理]
categories: Python
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 使用 Astral 的超快 Python 包管理器 uv 的安装与常用命令速查，包括项目初始化、依赖管理、虚拟环境与运行命令
cover:
---

# uv 使用命令指南

## 1. 前言

uv 是 Astral 出品的超快 Python 包与项目管理工具，集合了依赖解析、锁定、虚拟环境与命令运行等能力，适合个人与团队项目快速上手。

## 2. 安装与升级

建议使用官方安装脚本，跨平台且无需依赖 Python 预装。

- macOS/Linux

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# 验证
uv --version
```

- Windows（PowerShell）

```powershell
irm https://astral.sh/uv/install.ps1 | iex
# 验证
uv --version
```

- 升级
  重新执行上述安装脚本或使用系统包管理器升级。

## 3. 快速开始（新项目）

```bash
# 1) 初始化项目（创建 pyproject.toml 与锁文件）
uv init myapp
cd myapp

# 2) 添加运行时依赖
uv add requests

# 3) 添加开发依赖
uv add --dev pytest

# 4) 同步依赖（创建并更新虚拟环境）
uv sync

# 5) 运行代码或工具（无需手动激活虚拟环境）
uv run python -c "import requests; print('ok')"
uv run pytest -q
```

## 4. 管理虚拟环境与 Python

```bash
# 创建/刷新项目虚拟环境（默认 .venv）
uv venv

# 指定 Python 版本（先确保本机安装了对应版本，或用 uv 安装）
uv python install 3.12
uv venv --python 3.12

# 可选：手动激活
# macOS/Linux
source .venv/bin/activate
# Windows (PowerShell)
.venv\Scripts\Activate.ps1
```

提示：使用 uv run 执行命令时，无需激活虚拟环境。

## 5. 依赖管理（新增/移除/升级/锁定）

```bash
# 新增依赖（支持 extras 与版本约束）
uv add "httpx[http2]==0.27.*"

# 新增开发依赖
uv add --dev black

# 移除依赖
uv remove httpx

# 锁定/刷新依赖图（根据 pyproject 生成/更新锁文件）
uv lock

# 升级锁内版本（全量升级）
uv lock --upgrade
# 或仅升级单个依赖（指定后再 lock/sync）
uv add --upgrade requests

# 同步到虚拟环境（确保环境与锁一致）
uv sync
```

## 6. 运行命令与脚本

```bash
# 运行 Python 脚本
uv run python app.py

# 运行项目内工具（使用项目依赖环境）
uv run pytest -q
uv run black .

# 以项目环境运行临时命令
uv run python -c "print('hello uv')"
```

## 7. 与 requirements.txt 互操作

已有 requirements.txt 的项目，可直接使用 uv 的 pip 兼容子命令：

```bash
# 基于 requirements.txt 安装到当前项目的虚拟环境
uv venv
uv pip install -r requirements.txt

# 从当前环境导出（简易方式）
uv pip freeze > requirements.txt
```

建议逐步迁移到 pyproject.toml + uv.lock 的工作流，以获得更快的解析与可重复构建。

## 8. 缓存与清理

```bash
# 清理缓存（释放磁盘）
uv cache prune
```

提示：定期清理可缩减磁盘占用；首次安装包可能会稍慢（重新填充缓存）。

## 9. 常见问题

- 运行 uv run 时提示找不到依赖
  执行 uv sync 确保虚拟环境与锁文件一致，或先 uv add 后再 uv sync。

- VS Code 解释器选择
  指向项目根目录的 .venv（.venv/bin/python 或 .venv\Scripts\python.exe），或直接在任务中使用 uv run。

- Python 版本冲突/不可用
  使用 uv python install 3.x 安装所需版本，然后 uv venv --python 3.x 重新创建环境。

## 10. 快速参考命令

- 初始化：uv init demo && cd demo
- 新增依赖：uv add requests
- 开发依赖：uv add --dev pytest
- 同步环境：uv sync
- 运行命令：uv run python app.py
- 虚拟环境：uv venv
- 安装 Python：uv python install 3.12
- requirements.txt：uv pip install -r requirements.txt
- 导出环境：uv pip freeze > requirements.txt
- 清理缓存：uv cache prune
