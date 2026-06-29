---
title: clangd 安装与配置
date: 2026-06-29 21:40:49
updated: 2026-06-29 21:58:06
tags: [clangd, LSP, C++, VSCode, 工具]
categories: 工具
keywords: clangd, LSP, C++, 安装, VSCode, RPATH
description: clangd 本地安装、系统安装及 VSCode 配置指南，包含 RPATH 验证与目录结构说明
cover: https://picsum.photos/id/105/800/450
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

# clangd 安装与配置

clangd 是 LLVM 官方的 C/C++ LSP 服务器，为编辑器提供代码补全、跳转、诊断等功能。

## 1. 验证二进制独立性

从 LLVM 发布页下载的 clangd 依赖内置头文件，二进制与 `lib/` 目录不可拆分。

```bash
readelf -d /path/to/clangd_22.1.0/bin/clangd | grep -i 'rpath\|runpath'
```

- 若输出包含 `RUNPATH` 或 `RPATH`，表明二进制依赖同目录下的 `../lib` 查找资源
- **不能**将 `clangd` 二进制单独移动到其他位置而丢弃 `lib/` 目录

## 2. 本地安装（推荐）

适用于单用户、无需 root 的场景。

```bash
cp -r ~/clangd_22.1.0/bin/clangd ~/.local/bin/
cp -r ~/clangd_22.1.0/lib ~/.local/
```

安装后目录结构：

```
~/.local/
├── bin/clangd          # 已在 PATH 中
└── lib/clang/22/include/
```

提示：确保 `~/.local/bin` 在 `PATH` 中。大多数发行版默认包含此路径。

## 3. 系统安装

适用于多用户共享的场景，需要 root 权限。

```bash
sudo mv ~/clangd_22.1.0 /opt/clangd_22.1.0
sudo ln -s /opt/clangd_22.1.0/bin/clangd /usr/local/bin/clangd
```

## 4. VSCode 配置

在 VSCode 的 `settings.json` 中指定 clangd 路径：

```json
{
    "clangd.path": "/path/to/your/clangd"
}
```

- 本地安装：`"~/.local/bin/clangd"`
- 系统安装：`"/usr/local/bin/clangd"`（已在 PATH 中则可省略此配置）

## 5. 验证安装

```bash
clangd --version
```

确认版本信息与安装路径一致后，重启 VSCode 即可生效。
