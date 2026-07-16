---
title: 用工具链打包 deb：debhelper、fpm 与 checkinstall
date: 2026-07-11
updated: 2026-07-11
tags: [Ubuntu, deb, debhelper, fpm, checkinstall, packaging]
categories: linux
keywords: Ubuntu, deb 打包, debhelper, dh_make, dpkg-buildpackage, fpm, checkinstall, debian/rules, PPA
description: 手动 dpkg-deb 之外的三条工具链路线——debhelper 标准工作流、fpm 跨格式打包、checkinstall 拦截 make install，对比各自定位与最小用法。
top_img:
comments: true
cover: https://picsum.photos/id/216/800/450
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

手动 `dpkg-deb` 需自己摆目录、写 `control`。当项目要从源码构建、长期维护或上传 PPA 时，改用工具链更省心：你只写声明式规则，工具负责编译、摆放、打包。本文对比三条主流路线。

## 1. 三者定位对比

| 工具 | 定位 | 适用场景 |
| --- | --- | --- |
| `debhelper` (`dh_make`/`dpkg-buildpackage`) | Debian 官方标准工作流 | 从源码构建、上传 PPA/官方仓库、需遵循 Debian 政策 |
| `fpm` | 跨格式打包器（deb/rpm/apk…） | 已有成品文件，一条命令出包，多发行版分发 |
| `checkinstall` | 拦截 `make install` 自动成包 | 已能 `make install` 的老项目，快速留痕便于卸载 |

- 追求规范、可复现、可上游 → `debhelper`
- 追求「一条命令、多种格式」→ `fpm`
- 只想让 `make install` 装的东西能被 `dpkg` 管理 → `checkinstall`

## 2. debhelper 标准工作流

`debhelper` 通过 `debian/` 目录下的声明文件描述包，`dpkg-buildpackage` 一键完成构建到打包。

```bash
sudo apt install debhelper dh-make build-essential

# 在源码目录（含 Makefile）内初始化 debian/ 模板
cd myapp-1.0.0
dh_make --native --single --packagename myapp_1.0.0

# 编辑 debian/ 下的声明文件后，构建二进制包
dpkg-buildpackage -us -uc -b
# 生成的 .deb 出现在上级目录
```

初始化后 `debian/` 的关键文件：

| 文件 | 作用 |
| --- | --- |
| `control` | 包元信息与依赖（同手动方式，但由工具校验） |
| `rules` | 构建脚本，本质是 Makefile，通常仅需一行 `dh $@` |
| `changelog` | 版本历史，`Version` 从这里读取，用 `dch` 维护 |
| `install` | 声明「哪个文件装到哪」，替代手动 mkdir/cp |
| `copyright` | 许可证信息，上传官方仓库必需 |

`debian/rules` 最小形态：

```makefile
#!/usr/bin/make -f
%:
	dh $@
```

`debian/install` 用一份清单描述文件布局，不再手动建目录：

```text
src/myapp        usr/bin
conf/config.yaml etc/myapp
```

- `dh $@` 会自动串起 `dh_auto_build`（跑 `make`）、`dh_install`、`dh_installdeb` 等一系列步骤
- 版本号由 `debian/changelog` 顶部条目决定，用 `dch -i` 递增并记录变更
- `-us -uc` 跳过 GPG 签名（本地测试用），上传 PPA 时需去掉并配置签名
- `--native` 表示源码与打包同仓；上游 tar 分离的项目去掉它

## 3. fpm 一条命令出包

`fpm`（Effing Package Management）把「现成文件」直接压成包，无需 `debian/` 目录，且同一命令可切换输出格式。

```bash
sudo apt install ruby ruby-dev build-essential
sudo gem install fpm

# -s dir：源是目录树；-t deb：目标格式 deb
fpm -s dir -t deb \
    -n myapp -v 1.0.0 -a amd64 \
    --depends "python3 >= 3.8" \
    --maintainer "iehtian <you@example.com>" \
    --description "一个示例命令行工具" \
    --after-install postinst.sh \
    src/myapp=/usr/bin/myapp \
    conf/config.yaml=/etc/myapp/config.yaml
```

- `源路径=目标路径` 语法直接声明文件映射，省去手动摆目录树
- 把 `-t deb` 换成 `-t rpm`、`-t apk` 即可产出其他格式，命令其余部分不变
- `--after-install` / `--before-remove` 对应维护者脚本 `postinst` / `prerm`
- 不做源码编译，只负责「打包已有产物」；编译仍需自己先完成
- 依赖 Ruby 环境，比 debhelper 重，但比手写 `control` 灵活

## 4. checkinstall 拦截安装

`checkinstall` 监控 `make install` 实际写入了哪些文件，据此自动生成并安装一个 deb，让原本「散装安装」的软件可被 `dpkg` 卸载。

```bash
sudo apt install checkinstall

# 在 ./configure && make 完成后，用 checkinstall 代替 make install
sudo checkinstall --pkgname=myapp --pkgversion=1.0.0 --backup=no
```

- 交互式询问包信息后，跟踪 `make install` 的写入动作并打成 deb，同时完成安装
- 适合没有打包脚本、只提供 `make install` 的传统源码项目
- 生成的 `.deb` 留在当前目录，可复制到其他机器安装
- 记录可能不完整（如安装脚本用了非常规路径），仅适合快速留痕，不适合正式分发

## 5. 如何选择

```text
从源码构建、要上游/PPA、追求规范 → debhelper
已有成品、想一条命令产出多格式    → fpm
只有 make install、想快速可卸载   → checkinstall
文件极简、不想装任何工具          → 手动 dpkg-deb（见上一篇）
```

- 团队长期维护的项目优先 `debhelper`，规范换来可复现与可上游
- 内部快速分发、跨发行版交付选 `fpm`，学习成本最低
- `checkinstall` 是「过渡方案」，能规范化时应尽早迁移到前两者

手动打包的基础流程见《在 Ubuntu 上手动打包 deb》；现代化方案（`nfpm`、`cargo-deb`、`CPack`、CI 驱动）见《deb 包现代化打包方案：nfpm、cargo-deb、CPack 与 CI 驱动》。
