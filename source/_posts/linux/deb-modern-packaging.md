---
title: deb 包现代化打包方案：nfpm、cargo-deb、CPack 与 CI 驱动
date: 2026-07-12
updated: 2026-07-12
tags: [Ubuntu, deb, nfpm, cargo-deb, CPack, GoReleaser, sbuild, CI/CD, packaging]
categories: linux
keywords: Ubuntu, deb 打包, nfpm, cargo-deb, CPack, GoReleaser, sbuild, CI 打包, 自建 apt 仓库
description: 传统 debhelper/fpm 之外的现代打包方案——nfpm 零运行时、cargo-deb 与 GoReleaser 语言原生、CPack 免重复声明、sbuild 清洁构建与 CI 驱动分发。
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

传统路线（手动 `dpkg-deb`、`debhelper`/`fpm`/`checkinstall`）已在前两篇覆盖。本文聚焦近年涌现的现代方案：无运行时依赖的 `nfpm`、语言生态原生的 `cargo-deb` 与 `GoReleaser`、免重复声明的 `CPack`，以及 CI 驱动的清洁构建与自动分发。

## 1. 方案全景对比

| 方案 | 定位 | 依赖 | 适合场景 |
| --- | --- | --- | --- |
| `nfpm` | 通用打包器（Go） | 单二进制 | 替代 fpm，CI 友好 |
| `cargo-deb` | Rust 原生打包 | cargo | Rust CLI/服务 |
| `CPack` | CMake 打包扩展 | CMake | C/C++ 项目 |
| `GoReleaser` | Go 全平台分发 | 单二进制 | Go 多格式、多架构分发 |
| `sbuild` | 离线清洁构建 | schroot | Debian 规范构建 |
| CI + 仓库 | 持续集成流水线 | CI 环境 | 团队自动化分发 |

## 2. nfpm — 替代 fpm，零运行时

`fpm` 依赖 Ruby，CI 中需额外安装。`nfpm` 是 Go 重写版：单二进制、YAML 配置、零运行时依赖。

```bash
# 安装（单二进制，无运行时）
wget https://github.com/goreleaser/nfpm/releases/latest/download/nfpm_linux_amd64.tar.gz
tar xf nfpm_linux_amd64.tar.gz && sudo mv nfpm /usr/local/bin/
```

```yaml
# nfpm.yaml —— 声明式配置，随 VCS 走
name: myapp
version: 1.0.0
arch: amd64
maintainer: iehtian <you@example.com>
description: 一个示例命令行工具
depends:
  - python3 >= 3.8
contents:
  - src: ./src/myapp
    dst: /usr/bin/myapp
  - src: ./conf/config.yaml
    dst: /etc/myapp/config.yaml
    file_info:
      mode: 0644
  - src: ./deploy/myapp.service
    dst: /usr/lib/systemd/system/myapp.service
scripts:
  postinstall: ./scripts/postinst.sh
```

```bash
nfpm package -f nfpm.yaml -t .   # 生成 myapp_1.0.0_amd64.deb
```

- 配置文件一次编写、版本号可从环境变量插值：`-v "${VERSION}"`
- 切换 `-t rpm` / `-t apk` 即可产出其他格式，内容声明不变
- 官方提供 GitHub Action（`goreleaser/goreleaser-action`），CI 中一行配置集成

```yaml
# .github/workflows/release.yml 片段
- uses: goreleaser/goreleaser-action@v6
  with:
    args: nfpm package -f nfpm.yaml
```

## 3. cargo-deb — Rust 项目原生

从 `Cargo.toml` 自动推断包名、版本、依赖（动态链接 `.so`），一条命令出包。

```bash
cargo install cargo-deb
cargo deb
```

```toml
# Cargo.toml 中追加包元信息（可选，覆盖默认值）
[package.metadata.deb]
maintainer = "iehtian <you@example.com>"
depends = "libssl3"
section = "utils"
priority = "optional"
```

```bash
cargo deb --variant=minimal  # 静态链接变体，无运行时依赖
```

- 自动生成 `control`、`md5sums`、`changelog`，零额外配置
- 通过 `[package.metadata.deb]` 可声明 systemd unit、man page、conffiles
- 适合 Rust CLI 工具分发（如 `ripgrep`、`bat`、`fd` 均提供 deb）

## 4. CPack — CMake 生态内置

C/C++ 项目用 CMake 构建时，`install()` 规则已定义了文件布局。CPack 复用这些规则直接生成 deb，无需重复声明文件路径。

```cmake
# CMakeLists.txt
install(TARGETS myapp RUNTIME DESTINATION bin)
install(FILES conf/config.yaml DESTINATION etc/myapp)

set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "iehtian <you@example.com>")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libfoo1 (>= 1.0)")
set(CPACK_PACKAGE_VERSION "1.0.0")
include(CPack)
```

```bash
cmake -B build && cmake --build build
cd build && cpack -G DEB         # 复用 install() 规则生成 .deb
```

- `install()` 规则写一次，`make install` 与 deb 打包共用，不重复声明
- 支持组件拆分：`-D CPACK_COMPONENTS_ALL="libs"` 将库与可执行文件打成独立包
- 切换 `-G RPM` 即可产出 rpm，无需另写 spec

## 5. GoReleaser — Go 全平台分发

GoReleaser 内部复用 `nfpm`，在交叉编译完成后自动打包 deb/rpm。一条命令产出所有目标架构的包。

```yaml
# .goreleaser.yaml
builds:
  - binary: myapp
    goos: [linux]
    goarch: [amd64, arm64]

nfpms:
  - package_name: myapp
    maintainer: iehtian <you@example.com>
    description: 一个示例 Go 工具
    formats: [deb, rpm]
    contents:
      - src: ./deploy/myapp.service
        dst: /usr/lib/systemd/system/myapp.service
    scripts:
      postinstall: ./deploy/postinst.sh
```

```bash
goreleaser release --clean
```

- `nfpms` 配置语法与 `nfpm` 一致，无需额外学习
- 交叉编译 + 打包一气呵成，一次产出 `amd64` + `arm64` 的 deb
- 同时支持 Docker 镜像、Homebrew formula、snap，Go 项目全格式分发的标准答案

## 6. sbuild — 离线清洁构建

`debhelper` 在开发机上直接构建可能引入环境残留。`sbuild` 在临时 chroot 中构建，保证依赖完全由 `debian/control` 声明，杜绝"我机器上能编"。

```bash
sudo apt install sbuild schroot
sudo sbuild-createchroot --include=eatmydata noble /srv/chroot/noble \
    http://archive.ubuntu.com/ubuntu

# 在源码目录构建
sbuild --dist=noble
```

- 每次构建从干净 chroot 启动，不依赖宿主机已装软件
- 搭配 `pbuilder` / `cowbuilder` 用 COW 加速重复构建
- 可用 Docker/Podman 替代 chroot，CI 中更易编排

```dockerfile
# 容器化替代：Dockerfile
FROM ubuntu:noble
RUN apt update && apt install -y build-essential debhelper
COPY . /src && cd /src && dpkg-buildpackage -us -uc -b
```

## 7. CI 驱动 + 自建仓库

现代分发流程：Git tag push → CI 构建 → 签名 → 推送到自建 apt 仓库。以 `aptly` 管理仓库、GPG 签名为例：

```bash
# 自建仓库侧（GitHub Actions runner 或独立服务器）
aptly repo create -distribution=noble -component=main myapp
aptly repo add myapp myapp_1.0.0_amd64.deb
aptly publish repo -gpg-key="your-key-id" myapp
```

- `aptly` 支持仓库快照、合并、回滚，比手动 `reprepro` 更灵活
- CI 中版本号从 git tag 读取，`gpg --batch --passphrase` 非交互签名
- 用户只需 `echo "deb [signed-by=/usr/share/keyrings/myapp.gpg] https://apt.example.com/noble ./" > /etc/apt/sources.list.d/myapp.list` 一行即可接入

## 8. 如何选择

```text
Rust 项目                           → cargo-deb
Go 项目，需多格式分发               → GoReleaser
C/C++ 项目，CMake 构建              → CPack
已有成品，要跨格式、CI 友好         → nfpm
需 Debian 级别清洁构建              → sbuild / pbuilder
团队协作，多版本自动分发             → CI + 自建仓库
传统规范，上传 PPA/官方仓库          → debhelper（见第二篇）
单文件脚本，不想装任何工具           → 手动 dpkg-deb（见第一篇）
```

- 语言生态有原生方案优先用原生：自动推断元信息，零配置起步
- 跨语言、跨格式分发选 `nfpm`，配置文件即文档，可复用
- CI 驱动是趋势——无论选哪个工具，最终都应接入 CI 自动打包

## 9. 三篇联动

| 篇目 | 内容 | 适合 |
| --- | --- | --- |
| 第一篇：手动打包 deb | `dpkg-deb` + `control` + 维护者脚本 | 理解底层、简单脚本 |
| 第二篇：传统工具链 | `debhelper` / `fpm` / `checkinstall` | 规范构建、PPA 上传 |
| 第三篇：现代化方案（本文） | `nfpm` / `cargo-deb` / `CPack` / CI | 语言原生、CI 自动化 |

从手动 → 工具链 → 现代自动化，三层递进。理解手动方式才能理解工具替你做掉了什么；传统工具链提供了规范化的中间态；现代方案在工具链基础上进一步减少配置、融入语言生态、走向 CI 驱动。
