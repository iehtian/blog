---
title: 在 Ubuntu 上手动打包 deb
date: 2026-07-11
updated: 2026-07-11
tags: [Ubuntu, deb, dpkg, packaging, Debian]
categories: linux
keywords: Ubuntu, deb 包, dpkg-deb, 打包, control 文件, DEBIAN, 维护者脚本, postinst, lintian, apt
description: 从 deb 包的目录结构出发，用 dpkg-deb 手动打包一个 Ubuntu 软件包，涵盖 control 字段、文件布局、维护者脚本、构建校验与安装卸载全流程。
top_img:
comments: true
cover: https://picsum.photos/id/201/800/450
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

deb 包本质是一个 ar 归档，内含元数据（控制信息）与实际文件树。`dpkg-deb` 可直接把一个约定结构的目录打包成 `.deb`，无需 debhelper 等工具链，适合打包自有脚本、二进制或配置。

## 1. 目录结构

打包目录需要模拟安装后在系统中的真实路径，外加一个 `DEBIAN/` 存放元数据。

```text
myapp_1.0.0_amd64/
├── DEBIAN/
│   ├── control          # 必需：包元信息
│   ├── postinst         # 可选：安装后脚本
│   └── prerm            # 可选：卸载前脚本
├── usr/
│   ├── bin/
│   │   └── myapp        # 安装到 /usr/bin/myapp
│   └── share/
│       └── doc/myapp/
│           └── README
└── etc/
    └── myapp/
        └── config.yaml  # 安装到 /etc/myapp/config.yaml
```

- `DEBIAN/` 内文件不会被安装，仅供 dpkg 使用；其余目录原样映射到系统根目录
- 目录名约定为 `包名_版本_架构`，仅为可读性，真正的元信息以 `control` 为准
- 可执行文件、维护者脚本需有执行权限（`chmod 755`）

## 2. control 文件

`DEBIAN/control` 是唯一必需的元数据文件，定义包名、版本、依赖等。

```text
Package: myapp
Version: 1.0.0
Section: utils
Priority: optional
Architecture: amd64
Depends: python3 (>= 3.8), libc6
Maintainer: iehtian <you@example.com>
Description: 一个示例命令行工具
 支持多行描述，续行以单个空格开头。
 空行用 " ." 表示。
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `Package` | 是 | 包名，小写、数字、`-`，不含空格 |
| `Version` | 是 | 版本号，格式如 `1.0.0` 或 `1.0.0-1` |
| `Architecture` | 是 | `amd64`、`arm64`，与体系无关用 `all` |
| `Maintainer` | 是 | 维护者，格式 `Name <email>` |
| `Description` | 是 | 首行摘要 + 缩进续行详述 |
| `Depends` | 否 | 运行依赖，安装时强制满足 |
| `Section` | 否 | 分类，如 `utils`、`net`、`admin` |
| `Priority` | 否 | 常用 `optional` |

- `Description` 续行必须以一个空格开头，否则解析报错
- 纯脚本类（如 Python、Shell）用 `Architecture: all`
- `Depends` 会由 apt 自动解析安装；`dpkg -i` 则需手动补齐

## 3. 维护者脚本

维护者脚本在安装/卸载的特定阶段由 dpkg 调用，用于建目录、注册服务、清理残留等。放在 `DEBIAN/` 下，需 `chmod 755`。

| 脚本 | 触发时机 |
| --- | --- |
| `preinst` | 解包前 |
| `postinst` | 解包后（配置阶段），最常用 |
| `prerm` | 移除文件前 |
| `postrm` | 移除文件后 |

```bash
#!/bin/bash
# DEBIAN/postinst —— 安装完成后启用 systemd 服务
set -e

case "$1" in
    configure)
        systemctl daemon-reload
        systemctl enable myapp.service
        ;;
esac

exit 0
```

- 脚本首行必须是 shebang，且以 `set -e` 保证出错即中断
- `$1` 是动作参数：`postinst` 为 `configure`，`prerm` 为 `remove`/`upgrade`
- 卸载时删除文件应交给 dpkg，脚本只处理它管不到的内容（服务、缓存、软链）

## 4. 构建与校验

用 `dpkg-deb --build` 打包，`--root-owner-group` 强制包内文件归属 `root:root`（避免带入本地用户 UID）。

```bash
# 打包，生成 myapp_1.0.0_amd64.deb
dpkg-deb --build --root-owner-group myapp_1.0.0_amd64

# 查看包信息与文件清单
dpkg-deb --info myapp_1.0.0_amd64.deb      # control 内容
dpkg-deb --contents myapp_1.0.0_amd64.deb  # 文件列表

# 用 lintian 检查规范性（可选但推荐）
lintian myapp_1.0.0_amd64.deb
```

- 若省略输出名，`.deb` 与源目录同名；也可显式指定：`dpkg-deb --build 源目录 输出.deb`
- `lintian` 报告 `E:` 为错误、`W:` 为警告，正式分发前应尽量清零
- 权限问题多因未加 `--root-owner-group`，或源目录文件权限不当

## 5. 安装与卸载

```bash
sudo dpkg -i myapp_1.0.0_amd64.deb   # 安装（不自动装依赖）
sudo apt-get install -f              # 补齐缺失依赖

sudo apt install ./myapp_1.0.0_amd64.deb  # 推荐：apt 会自动解析依赖

dpkg -l myapp        # 查看安装状态
dpkg -L myapp        # 列出已安装的文件

sudo apt remove myapp     # 卸载，保留配置文件
sudo apt purge myapp      # 卸载并删除配置文件
```

- `dpkg -i` 不解析依赖，缺依赖会中断，需再跑 `apt-get install -f`
- 本地包用 `apt install ./file.deb`（路径前缀 `./` 必需）可一步装好依赖
- `remove` 保留 `/etc` 下配置，`purge` 连同删除

## 6. 用脚本自动化

手动摆目录、改版本号、设权限重复且易错，可用一个 Shell 脚本把整套流程固化下来：改版本只需动一个变量，运行即出包。

```bash
#!/bin/bash
# build-deb.sh —— 一键构建 myapp 的 deb 包
set -e

PKG=myapp
VERSION=1.0.0
ARCH=amd64
BUILD_DIR="${PKG}_${VERSION}_${ARCH}"

# 1. 清理并按安装后路径建目录树
rm -rf "$BUILD_DIR"
mkdir -p "$BUILD_DIR/DEBIAN"
mkdir -p "$BUILD_DIR/usr/bin"
mkdir -p "$BUILD_DIR/etc/$PKG"

# 2. 拷贝实际文件（源文件放在仓库的 src/、conf/ 下）
install -m 755 src/myapp        "$BUILD_DIR/usr/bin/myapp"
install -m 644 conf/config.yaml "$BUILD_DIR/etc/$PKG/config.yaml"

# 3. 生成 control（变量插值，版本号只在顶部改一次）
cat > "$BUILD_DIR/DEBIAN/control" <<EOF
Package: $PKG
Version: $VERSION
Section: utils
Priority: optional
Architecture: $ARCH
Depends: python3 (>= 3.8)
Maintainer: iehtian <you@example.com>
Description: 一个示例命令行工具
 由 build-deb.sh 自动打包生成。
EOF

# 4. 维护者脚本（用 here-doc 内联，避免额外文件）
cat > "$BUILD_DIR/DEBIAN/postinst" <<'EOF'
#!/bin/bash
set -e
[ "$1" = configure ] && echo "myapp installed."
exit 0
EOF
chmod 755 "$BUILD_DIR/DEBIAN/postinst"

# 5. 打包并校验
dpkg-deb --build --root-owner-group "$BUILD_DIR"
lintian "${BUILD_DIR}.deb" || true

echo "已生成 ${BUILD_DIR}.deb"
```

- 版本、架构、依赖集中在脚本顶部，改一处即可，避免目录名与 `control` 不同步
- `install -m` 同时完成拷贝与权限设置，比 `cp` + `chmod` 更紧凑
- `control` 用普通 here-doc（变量会插值）；`postinst` 用 `<<'EOF'`（引号防止 `$1` 被提前展开）
- `lintian ... || true` 让校验失败不中断脚本；正式发布时应去掉 `|| true` 强制通过
- 接入 CI 时，`VERSION` 可改为从 git tag 读取：`VERSION=$(git describe --tags --abbrev=0)`

## 7. 小结

手动打包 deb 的最小闭环：

1. 按安装后路径布局文件树，附带 `DEBIAN/control`
2. 需要时补维护者脚本（`postinst` 等）并授予执行权限
3. `dpkg-deb --build --root-owner-group` 打包，`lintian` 校验
4. `apt install ./xxx.deb` 安装以自动处理依赖
5. 重复打包用 Shell 脚本固化流程，版本号集中管理

需要长期维护、上传 PPA 或遵循 Debian 政策时，再转向 `debhelper`、`fpm` 等工具链，详见《用工具链打包 deb：debhelper、fpm 与 checkinstall》；语言生态原生方案与 CI 驱动分发见《deb 包现代化打包方案：nfpm、cargo-deb、CPack 与 CI 驱动》。
