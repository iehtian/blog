---
title: Python socket 模块创建套接字详解
date: 2026-08-14
updated: 2026-08-14
tags: [Python, socket, 网络编程]
categories: Python
keywords: Python, socket, socket.socket, socketpair, create_connection, create_server, fromfd, fromshare, SocketType, 网络编程
description: Python socket 模块创建套接字的函数详解，涵盖 socket、socketpair、create_connection、create_server、fromfd、fromshare 的用法、参数与区别
toc: true
toc_number: true
copyright: true
cover: https://picsum.photos/id/302/800/450
---

# Python socket 模块创建套接字详解

`socket` 模块提供 8 个用于创建套接字的函数/对象，按用途分为三类：

1. **底层构造** —— `socket()`、`socketpair()`
2. **高层便捷函数** —— `create_connection()`、`create_server()`
3. **从已有资源构建** —— `fromfd()`、`fromshare()`，以及 `SocketType`、`has_dualstack_ipv6()`

## 1. socket — 核心构造器

```python
s = socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `family` | `int` | `AF_INET` | 地址族：`AF_INET`、`AF_INET6`、`AF_UNIX`、`AF_CAN`、`AF_PACKET`、`AF_RDS` |
| `type` | `int` | `SOCK_STREAM` | 套接字类型：`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW` 等 `SOCK_` 常量 |
| `proto` | `int` | `0` | 协议号，通常为 0；`AF_CAN` 时为 `CAN_RAW`/`CAN_BCM`/`CAN_ISOTP`/`CAN_J1939` |
| `fileno` | `int` | `None` | 从已有文件描述符构建，自动检测 `family`/`type`/`proto` |

- `fileno` 指定时，三个参数自动检测，可用显式参数覆盖（仅影响 Python 表示，不影响 OS 资源）
- 与 `fromfd()` 不同，`fileno` 返回**同一个套接字**（非副本），可用 `close()` 关闭分离的套接字
- 新创建的套接字不可继承
- 3.7 起 `SOCK_NONBLOCK` / `SOCK_CLOEXEC` 位标志会被清除，`socket.type` 不反映它们（但仍传给底层 `socket()` 调用）

```python
sock = socket.socket(
    socket.AF_INET,
    socket.SOCK_STREAM | socket.SOCK_NONBLOCK)
# sock 仍是非阻塞的，但 sock.type 返回 socket.SOCK_STREAM
```

## 2. socketpair — 一对已连接套接字

```python
s1, s2 = socket.socketpair()   # 参数同 socket()，默认族 AF_UNIX
```

- 创建一对**已连接**的套接字对象
- 默认族 `AF_UNIX`（平台不支持则为 `AF_INET`）
- 常用于父子进程通信或本地测试（如 `recvmsg_into` 的示例）

## 3. create_connection — 高层 TCP 客户端连接

```python
s = socket.create_connection(("example.com", 80), timeout=10)
```

| 参数 | 说明 |
|---|---|
| `address` | `(host, port)` 二元组 |
| `timeout` | 连接前设置套接字超时，缺省用 `getdefaulttimeout()` |
| `source_address` | `(host, port)`，连接前绑定源地址 |
| `all_errors` | `True` 时抛 `ExceptionGroup` 包含所有尝试错误 |

- 比 `connect()` 高层：非数字主机名会解析 `AF_INET` 和 `AF_INET6`，依次尝试直到成功，天然兼容 IPv4/IPv6
- 失败默认抛**最后一个地址**的异常；`all_errors=True` 时抛包含全部错误的 `ExceptionGroup`

## 4. create_server — 便捷创建 TCP 服务器

```python
s = socket.create_server(("", 8080), family=AF_INET6, dualstack_ipv6=True)
```

| 参数 | 说明 |
|---|---|
| `address` | `(host, port)` 二元组 |
| `family` | `AF_INET` 或 `AF_INET6` |
| `backlog` | `listen()` 队列大小，缺省用合理默认 |
| `reuse_port` | 是否设置 `SO_REUSEPORT` |
| `dualstack_ipv6` | 双栈，同时接受 IPv4/IPv6 |

- 返回**绑定到** `address` 的 TCP 套接字
- POSIX 平台会设置 `SO_REUSEADDR`，便于 `TIME_WAIT` 状态立即重用
- `dualstack_ipv6=True` 且平台支持时，IPv4 连接表现为 IPv4 映射 IPv6 地址；不支持则抛 `ValueError`

```python
import socket

addr = ("", 8080)  # 所有接口，端口 8080
if socket.has_dualstack_ipv6():
    s = socket.create_server(addr, family=socket.AF_INET6, dualstack_ipv6=True)
else:
    s = socket.create_server(addr)
```

## 5. fromfd / fromshare — 从已有资源构建

```python
s = socket.fromfd(fd, socket.AF_INET, socket.SOCK_STREAM)
```

- `fromfd(fd, family, type, proto=0)`：复制文件描述符 `fd` 构建套接字，`fd` 来自 `file.fileno()`
- **不校验** `fd` 是否指向套接字；套接字假定为阻塞模式
- 用于获取/设置作为标准输入输出传入的套接字选项（如 Unix inetd 启动的服务器）
- `fromshare(data)`：从 `socket.share()` 获取的数据实例化套接字，仅 Windows

## 6. SocketType / has_dualstack_ipv6

- `SocketType`：表示套接字对象类型的类型对象，等同于 `type(socket.socket())`
- `has_dualstack_ipv6()`：平台支持创建同时处理 IPv4/IPv6 的 TCP 套接字时返回 `True`

## 7. 选择指南

| 场景 | 推荐 |
|---|---|
| 创建普通 TCP/UDP 套接字 | `socket.socket()` |
| 创建一对互通套接字（进程通信/测试） | `socket.socketpair()` |
| 客户端连接（IPv4/IPv6 自动） | `socket.create_connection()` |
| 快速启动 TCP 服务器 | `socket.create_server()` |
| 包装已有 fd（如 inetd 场景） | `socket.fromfd()` |
| 跨进程共享套接字（Windows） | `socket.fromshare()` |

## 小结

- 底层用 `socket()` / `socketpair()`，高层用 `create_connection()` / `create_server()`
- `socket(fileno=...)` 复用同一个 fd，`fromfd()` 复制 fd，二者语义不同
- `create_connection` 自动解析 IPv4/IPv6，`create_server` 可开双栈并支持端口复用
