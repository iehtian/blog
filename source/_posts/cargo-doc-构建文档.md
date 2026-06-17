---
title: 使用 cargo doc --open 构建并浏览本地 Rust 依赖文档
date: 2026-06-17
updated: 2026-06-17
tags: [Rust, cargo, doc]
categories: rust
keywords: cargo doc, Rust 文档, 离线文档, 远程开发
description: 介绍如何使用 cargo doc --open 为本地依赖构建文档并在浏览器中查看，以及在远程开发机上通过 HTTP 共享文档的方法。
top_img:
comments: true
cover: https://picsum.photos/id/1015/800/450
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

## 背景

在 Rust 开发中，查阅依赖库的文档是高频需求。默认我们会访问 [docs.rs](https://docs.rs) 来查看在线文档，但这需要网络连接，且文档版本可能与本地 `Cargo.lock` 中锁定的版本不一致。

`cargo doc` 命令可以**根据你本地的依赖版本构建文档**，完美解决版本匹配问题，离线也能查阅。

## 基本用法

在项目根目录下执行：

```bash
cargo doc --open
```

这条命令做了两件事：

1. **构建文档**：为当前项目及其所有依赖生成 HTML 文档，输出到 `target/doc/` 目录。
2. **自动打开浏览器**：构建完成后在默认浏览器中打开文档首页。

提示：如果只想构建文档而不打开浏览器，去掉 `--open` 参数即可：

```bash
cargo doc
```

## 常用参数

### 只构建当前包的文档

默认会构建所有依赖的文档，耗时较长。如果只需要当前包的文档：

```bash
cargo doc --no-deps --open
```

### 构建特定包的文档

```bash
cargo doc -p <包名> --open
```

### 包含私有项

默认只生成公开 API 的文档。如需包含私有项（通常用于内部查阅）：

```bash
cargo doc --document-private-items --open
```

## 远程主机上的使用

在远程开发机或服务器上，`--open` 参数无法直接打开本地浏览器。此时可以手动启动一个 HTTP 服务来浏览文档。

文档构建完成后，输出目录位于 `target/doc/`：

```bash
# 进入文档目录
cd target/doc

# 启动 HTTP 服务（Python 3 自带）
python3 -m http.server 8081
```

然后在本地浏览器访问 `http://<远程主机IP>:8081` 即可浏览。

完整操作流程示例：

```bash
# 1. 进入项目目录
cd /home/iehtian/rust/guessing_game

# 2. 构建文档
cargo doc

# 3. 启动 HTTP 服务供远程访问
cd target/doc
python3 -m http.server 8081
```

本地浏览器访问 `http://<服务器地址>:8081` 即可看到文档首页，搜索和导航完全可用。

注意：如果服务器有防火墙，需要放行对应端口（如 8081）：

```bash
# 如果使用 ufw
sudo ufw allow 8081/tcp
```

## 不需要 HTTP 服务的替代方案

对于 VS Code 用户，如果通过 Remote-SSH 连接远程主机，可以使用 **Live Preview** 等扩展直接在远程主机上打开 HTML 文件预览。

或者直接将 `target/doc/` 目录下载到本地，用本地浏览器打开 `index.html`：

```bash
scp -r user@host:/path/to/project/target/doc /tmp/rust-doc
# 然后在本地打开 /tmp/rust-doc/index.html
```

## 小结

- `cargo doc` 构建的是**与你 `Cargo.lock` 版本完全一致**的文档，不会出现本地 API 与在线文档不匹配的问题。
- 本地开发直接用 `cargo doc --open`，一步到位。
- 远程开发机构建后用 `python3 -m http.server` 临时共享，简单高效。
- 善用 `--no-deps` 加快构建速度，只在你需要查看依赖文档时再全量构建。
