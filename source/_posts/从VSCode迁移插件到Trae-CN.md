---
title: 从 VSCode 迁移插件到 Trae-CN
date: 2026-06-16 10:00:00
updated: 2026-06-16 10:00:00
tags:
  - VSCode
  - Trae
  - IDE
categories:
  - 工具
keywords: VSCode, Trae, 插件迁移, cp, 扩展, extensions
description: 介绍一种通过 cp 命令将 VSCode 已安装插件快速迁移到 Trae-CN 的方法，适用于批量迁移场景。
cover: https://picsum.photos/id/1025/800/450
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
comments: true
aside: true
---

## 前言

[Trae](https://www.trae.ai/) 是字节跳动推出的 AI IDE，国内版（Trae-CN）基于 VSCode 内核构建，兼容大部分 VSCode 插件生态。当从 VSCode 切换到 Trae-CN 时，逐个在插件市场重新搜索安装十分繁琐。本文介绍一种利用 `cp` 命令直接复制插件目录的批量迁移方法。

## 原理

VSCode 和 Trae-CN 的插件本质上是相同结构的目录，只是存放路径不同：

- **VSCode 插件目录**：`~/.vscode/extensions/`
- **Trae-CN 插件目录**：`~/.trae-cn/extensions/`

由于两者底层架构一致，大多数插件可以直接复制使用，无需重新下载。

需要注意的是，VSCode 的插件分为**本地插件**和**远程插件**两类：

- **本地插件**：运行在本地机器上，如主题、格式化工具等
- **远程插件（Remote - SSH）**：当你通过 Remote SSH 连接到远程服务器时，部分插件会安装在远程服务器上，如语言服务、调试器等

远程插件的存放路径与本地不同，下文分别说明。

## 操作步骤

### 1. 确认 VSCode 插件目录

先查看 VSCode 已安装的插件：

```bash
ls ~/.vscode/extensions/
```

输出类似：

```
esbenp.prettier-vscode-10.1.0
ms-python.python-2024.0.1
dbaeumer.vscode-eslint-2.4.4
...
```

### 2. 确认 Trae-CN 插件目录

检查 Trae-CN 的插件目录是否存在，不存在则创建：

```bash
mkdir -p ~/.trae-cn/extensions/
```

### 3. 批量复制插件

将 VSCode 的所有插件一次性复制到 Trae-CN：

```bash
cp -r ~/.vscode/extensions/* ~/.trae-cn/extensions/
```

如果只想迁移部分插件，可以指定插件目录名：

```bash
cp -r ~/.vscode/extensions/esbenp.prettier-vscode-10.1.0 ~/.trae-cn/extensions/
cp -r ~/.vscode/extensions/ms-python.python-2024.0.1 ~/.trae-cn/extensions/
```

### 4. 迁移远程插件（Remote SSH）

如果你使用 VSCode 的 Remote SSH 连接远程服务器，远程插件安装在服务器上的 `~/.vscode-server/extensions/` 目录中，对应 Trae-CN 的远程插件目录为 `~/.trae-cn-server/extensions/`。

在**远程服务器**上执行：

```bash
# 创建目标目录
mkdir -p ~/.trae-cn-server/extensions/

# 批量复制远程插件
cp -r ~/.vscode-server/extensions/* ~/.trae-cn-server/extensions/
```

如果只想迁移个别远程插件，同样可以指定目录名：

```bash
cp -r ~/.vscode-server/extensions/ms-python.python-2024.0.1 ~/.trae-cn-server/extensions/
```

### 5. 重启 Trae-CN

复制完成后，重启 Trae-CN 即可在插件列表中看到已迁移的插件。若迁移了远程插件，重新连接远程服务器后即可生效。

## 注意事项

- **版本兼容性**：极少数插件依赖特定 VSCode 版本的 API，若 Trae-CN 的 VSCode 内核版本与插件要求的版本差异较大，可能出现兼容问题。遇到这种情况，建议在 Trae-CN 插件市场安装该插件的最新版本。
- **二进制依赖**：部分插件（如 C/C++、Python 语言服务）包含平台相关的二进制文件，复制后通常可正常工作，但如果两份 IDE 运行环境差异较大（如架构不同），需要重新安装。
- **不要覆盖已有插件**：如果 Trae-CN 中已有同名插件（版本可能更新），复制时会被覆盖。建议先检查目标目录，或使用备份策略：

```bash
# 备份 Trae-CN 现有插件
cp -r ~/.trae-cn/extensions/ ~/.trae-cn/extensions-backup/

# 使用 -n 参数避免覆盖已有文件
cp -rn ~/.vscode/extensions/* ~/.trae-cn/extensions/
```

- **Windows 用户**：Windows 下插件目录路径为 `%USERPROFILE%\.vscode\extensions\` 和 `%USERPROFILE%\.trae-cn\extensions\`，可使用 `xcopy` 或在 PowerShell 中使用 `Copy-Item` 完成类似操作：

```powershell
Copy-Item -Recurse "$env:USERPROFILE\.vscode\extensions\*" "$env:USERPROFILE\.trae-cn\extensions\"
```

- **Mac 用户**：Mac 下插件目录路径与 Linux 一致，操作方式相同。
- **本地与远程目录不要混淆**：本地插件对应 `~/.vscode/extensions/` → `~/.trae-cn/extensions/`，远程插件对应 `~/.vscode-server/extensions/` → `~/.trae-cn-server/extensions/`。两者路径不同，复制时请确认操作的是正确的目录。

## 小结

利用 `cp` 命令迁移插件是一种简单快捷的方式，核心在于 VSCode 和 Trae-CN 共享相同的插件目录结构。无论是本地插件还是远程服务器上的插件，只需复制对应目录即可完成迁移。对于大多数插件，复制即可用；遇到兼容问题时，再回到插件市场单独更新即可。
