---
title: systemd
date: 2025.06.04
updated: 2025.06.04
tags: 
categories: 
keywords:
description: linux
top_img: 
comments: true
cover:
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

# systemctl 完整参数列表

## 概述

systemctl 是 systemd 系统和服务管理器的命令行工具，用于控制 systemd 服务、挂载点、设备等。

## 命令语法

```bash
systemctl [OPTIONS...] COMMAND [UNIT...]
```

## 全局选项 (Global Options)

### 基本选项

- `--version` - 显示程序版本
- `--help`, `-h` - 显示帮助信息
- `--no-pager` - 不使用分页器
- `--no-legend` - 不显示列标题和提示
- `--no-ask-password` - 不请求用户密码
- `--system` - 连接到系统服务管理器（默认）
- `--user` - 连接到用户服务管理器
- `--global` - 对所有用户启用/禁用单元文件
- `--runtime` - 临时更改，重启后失效
- `--quiet`, `-q` - 静默模式
- `--root=PATH` - 在指定的根目录下操作

### 输出控制

- `--output=MODE`, `-o` - 更改日志输出模式
  - `short` - 默认输出格式
  - `short-iso` - 带ISO 8601时间戳
  - `short-precise` - 精确时间戳
  - `short-monotonic` - 单调时间戳
  - `verbose` - 详细输出
  - `export` - 适合导出的格式
  - `json` - JSON格式输出
  - `json-pretty` - 格式化的JSON
  - `json-sse` - 服务器推送事件JSON
  - `cat` - 仅显示消息内容
- `--lines=N`, `-n` - 显示最后N行日志
- `--follow`, `-f` - 持续输出新日志行
- `--since=TIME` - 显示指定时间之后的日志
- `--until=TIME` - 显示指定时间之前的日志

### 操作选项

- `--force`, `-f` - 强制执行操作
- `--kill-who=WHO` - 选择要终止的进程
  - `main` - 主进程
  - `control` - 控制进程
  - `all` - 所有进程
- `--signal=SIGNAL`, `-s` - 指定发送的信号
- `--fail` - 如果任何操作失败则退出
- `--irreversible` - 将作业标记为不可逆
- `--ignore-dependencies` - 忽略单元依赖关系
- `--show-types` - 显示单元类型
- `--job-mode=MODE` - 指定作业模式
  - `replace` - 替换现有作业（默认）
  - `fail` - 如果冲突则失败
  - `isolate` - 隔离启动
  - `ignore-dependencies` - 忽略依赖
  - `ignore-requirements` - 忽略需求

### 主机相关

- `--host=HOST`, `-H` - 在远程主机上操作
- `--machine=CONTAINER`, `-M` - 在本地容器中操作

## 单元管理命令 (Unit Management)

### 状态查询

- `status [UNIT...]` - 显示单元状态
  - `--lines=N`, `-n` - 显示最后N行日志
  - `--output=MODE`, `-o` - 日志输出格式
  - `--no-pager` - 不使用分页器
  - `--full` - 显示完整的单元名称和日志消息
- `show [UNIT...]` - 显示单元属性
  - `--property=NAME`, `-p` - 只显示指定属性
  - `--all`, `-a` - 显示所有属性（包括空值）
  - `--value` - 只显示属性值，不显示属性名
- `cat [UNIT...]` - 显示单元文件内容
- `list-units [PATTERN...]` - 列出已加载的单元
  - `--type=TYPE`, `-t` - 按类型过滤单元
  - `--state=STATE` - 按状态过滤单元
  - `--all`, `-a` - 显示所有单元（包括非活动的）
  - `--recursive`, `-r` - 递归显示依赖单元
  - `--reverse` - 显示反向依赖
  - `--after` - 显示应在指定单元之后启动的单元
  - `--before` - 显示应在指定单元之前启动的单元
  - `--failed` - 仅显示失败的单元
- `list-unit-files [PATTERN...]` - 列出已安装的单元文件
  - `--type=TYPE`, `-t` - 按类型过滤单元文件
  - `--state=STATE` - 按状态过滤单元文件
  - `--root=PATH` - 在指定根目录下查找
- `list-dependencies [UNIT]` - 显示单元依赖关系
  - `--reverse` - 显示反向依赖
  - `--after` - 显示排序依赖（After=）
  - `--before` - 显示排序依赖（Before=）
  - `--all`, `-a` - 显示所有依赖层级
- `list-sockets [PATTERN...]` - 列出套接字单元
  - `--all`, `-a` - 显示所有套接字
  - `--show-types` - 显示套接字类型
- `list-timers [PATTERN...]` - 列出定时器单元
  - `--all`, `-a` - 显示所有定时器
- `list-jobs [PATTERN...]` - 列出活动作业

### 单元控制

- `start UNIT...` - 启动单元
  - `--no-block` - 不等待操作完成
  - `--job-mode=MODE` - 指定作业模式
- `stop UNIT...` - 停止单元
  - `--no-block` - 不等待操作完成
- `reload UNIT...` - 重新加载单元配置
  - `--no-block` - 不等待操作完成
- `restart UNIT...` - 重启单元
  - `--no-block` - 不等待操作完成
- `try-restart UNIT...` - 如果运行则重启单元
  - `--no-block` - 不等待操作完成
- `reload-or-restart UNIT...` - 重新加载或重启单元
  - `--no-block` - 不等待操作完成
- `try-reload-or-restart UNIT...` - 尝试重新加载或重启单元
  - `--no-block` - 不等待操作完成
- `isolate UNIT` - 启动单元并停止所有其他单元
  - `--no-block` - 不等待操作完成
- `kill UNIT...` - 向单元发送信号
  - `--kill-who=WHO` - 选择要终止的进程（main/control/all）
  - `--signal=SIGNAL`, `-s` - 指定发送的信号
- `clean UNIT...` - 清理单元运行时数据
  - `--what=RESOURCES` - 指定要清理的资源类型
    - `configuration` - 配置
    - `state` - 状态
    - `cache` - 缓存
    - `logs` - 日志
    - `runtime` - 运行时数据
    - `all` - 所有资源

### 单元文件管理

- `enable UNIT...` - 启用单元文件
  - `--now` - 启用后立即启动单元
  - `--runtime` - 仅在本次启动期间启用
  - `--global` - 为所有用户启用（用户单元）
- `disable UNIT...` - 禁用单元文件
  - `--now` - 禁用前先停止单元
  - `--runtime` - 仅在本次启动期间禁用
  - `--global` - 为所有用户禁用（用户单元）
- `reenable UNIT...` - 重新启用单元文件
  - `--now` - 重新启用后立即启动单元
  - `--runtime` - 仅在本次启动期间重新启用
- `preset UNIT...` - 重置单元启用状态
  - `--preset-mode=MODE` - 指定预设模式
    - `full` - 完全预设（默认）
    - `enable-only` - 仅启用
    - `disable-only` - 仅禁用
- `preset-all` - 重置所有单元启用状态
- `is-enabled UNIT...` - 检查单元是否启用
- `mask UNIT...` - 屏蔽单元
  - `--runtime` - 仅在本次启动期间屏蔽
  - `--now` - 屏蔽前先停止单元
- `unmask UNIT...` - 取消屏蔽单元
- `link PATH...` - 链接单元文件到单元文件搜索路径
- `add-wants TARGET UNIT...` - 添加Wants依赖
- `add-requires TARGET UNIT...` - 添加Requires依赖
- `edit UNIT...` - 编辑单元文件
- `get-default` - 获取默认目标
- `set-default TARGET` - 设置默认目标

### 状态检查

- `is-active UNIT...` - 检查单元是否活动
- `is-failed UNIT...` - 检查单元是否失败
- `is-system-running` - 检查系统是否正在运行

## 系统管理命令 (System Management)

### 系统状态

- `list-machines` - 列出本地容器和主机
- `list-environments` - 列出环境变量

### 系统控制

- `daemon-reload` - 重新加载systemd管理器配置
- `daemon-reexec` - 重新执行systemd管理器
- `emergency` - 进入紧急模式
- `rescue` - 进入救援模式
- `halt` - 停机
  - `--force`, `-f` - 强制停机
  - `--no-wall` - 不向所有用户发送关机消息
- `poweroff` - 关机
  - `--force`, `-f` - 强制关机
  - `--no-wall` - 不向所有用户发送关机消息
- `reboot` - 重启
  - `--force`, `-f` - 强制重启
  - `--no-wall` - 不向所有用户发送重启消息
- `kexec` - 通过kexec重启
- `exit [EXIT_CODE]` - 请求用户管理器退出
- `switch-root ROOT [INIT]` - 切换到不同的根文件系统
- `suspend` - 挂起系统
- `hibernate` - 休眠系统
- `hybrid-sleep` - 混合睡眠
- `suspend-then-hibernate` - 挂起后休眠

## 环境管理命令 (Environment Management)

- `show-environment` - 显示环境变量
- `set-environment VARIABLE=VALUE...` - 设置环境变量
- `unset-environment VARIABLE...` - 取消设置环境变量
- `import-environment [VARIABLE...]` - 导入环境变量

## 日志管理相关

### 日志查看选项（与journalctl配合使用）

当使用 `systemctl status` 时，以下选项影响日志显示：

- `--lines=N`, `-n` - 显示最后N行日志
- `--output=MODE`, `-o` - 日志输出格式
- `--no-pager` - 不使用分页器

## 单元类型 (Unit Types)

systemctl 可以操作以下类型的单元：

- `.service` - 服务单元（默认类型）
- `.socket` - 套接字单元
- `.target` - 目标单元
- `.device` - 设备单元
- `.mount` - 挂载单元
- `.automount` - 自动挂载单元
- `.swap` - 交换单元
- `.timer` - 定时器单元
- `.path` - 路径单元
- `.slice` - 切片单元
- `.scope` - 作用域单元

## 退出状态码

- `0` - 成功
- `1` - 一般性错误
- `2` - 无效参数
- `3` - 未实现的功能
- `4` - 用户没有必要的权限
- `5` - 程序未安装

## 常用示例

### 服务管理

```bash
# 启动服务
systemctl start nginx

# 停止服务
systemctl stop nginx

# 重启服务
systemctl restart nginx

# 查看服务状态
systemctl status nginx

# 启用服务（开机自启）
systemctl enable nginx

# 启用服务并立即启动
systemctl enable --now nginx

# 禁用服务
systemctl disable nginx

# 禁用服务并立即停止
systemctl disable --now nginx

# 查看服务是否启用
systemctl is-enabled nginx

# 查看服务是否运行
systemctl is-active nginx

# 重新启用服务并立即启动
systemctl reenable --now nginx

# 屏蔽服务并立即停止
systemctl mask --now nginx

# 取消屏蔽服务
systemctl unmask nginx
```

### 系统信息

```bash
# 列出所有服务
systemctl list-units --type=service

# 列出所有失败的服务
systemctl list-units --failed

# 列出所有启用的服务
systemctl list-unit-files --state=enabled

# 列出所有禁用的服务
systemctl list-unit-files --state=disabled

# 列出所有屏蔽的服务
systemctl list-unit-files --state=masked

# 显示服务详细信息
systemctl show nginx

# 显示服务特定属性
systemctl show nginx --property=ActiveState,LoadState

# 查看系统启动耗时
systemd-analyze

# 查看启动链
systemctl list-dependencies

# 查看反向依赖
systemctl list-dependencies --reverse

# 查看定时器状态
systemctl list-timers

# 查看套接字状态
systemctl list-sockets
```

### 系统控制

```bash
# 重新加载systemd配置
systemctl daemon-reload

# 查看默认启动目标
systemctl get-default

# 设置默认启动目标
systemctl set-default multi-user.target

# 系统重启
systemctl reboot

# 系统关机
systemctl poweroff
```

## 相关文件

- `/etc/systemd/system/` - 系统单元文件目录
- `/usr/lib/systemd/system/` - 包管理器安装的单元文件
- `/run/systemd/system/` - 运行时单元文件
- `~/.config/systemd/user/` - 用户单元文件目录
- `/etc/systemd/system.conf` - systemd主配置文件
- `/etc/systemd/user.conf` - 用户管理器配置文件

## 注意事项

1. 大多数systemctl命令需要root权限（除了查询命令）
2. 使用 `--user` 选项可以管理用户服务
3. 单元名称可以不带 `.service` 后缀（对于服务单元）
4. 使用通配符时要注意shell的展开规则
5. `daemon-reload` 在修改单元文件后必须执行
6. 某些命令在容器环境中可能受限
