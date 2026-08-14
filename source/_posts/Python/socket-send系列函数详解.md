---
title: Python socket 模块 send 系列函数详解
date: 2026-08-14
updated: 2026-08-14
tags: [Python, socket, 网络编程]
categories: Python
keywords: Python, socket, send, sendall, sendto, sendmsg, sendmsg_afalg, sendfile, 网络编程
description: Python socket 模块 send 系列发送函数（send、sendall、sendto、sendmsg、sendfile）的作用、返回值与区别详解
toc: true
toc_number: true
copyright: true
cover: https://picsum.photos/id/301/800/450
---

# Python socket 模块 send 系列函数详解

`socket` 模块提供 5 个常用发送函数，可沿三个维度区分：

1. **目标地址来源** —— `send` / `sendall` 依赖已连接的套接字；`sendto` / `sendmsg` 由 `address` 参数指定（无需连接）
2. **是否保证发完** —— `sendall` 阻塞直到全部发送；`send` 可能只发一部分，需自行处理剩余数据
3. **是否发送辅助数据（控制消息）** —— 仅 `sendmsg` 支持

| 函数 | 需已连接 | 保证全部发送 | 辅助数据 | 返回值 |
|---|---|---|---|---|
| `send` | ✅ | ❌ | ❌ | 发送的字节数 |
| `sendall` | ✅ | ✅ | ❌ | `None` |
| `sendto` | ❌（指定 address） | ❌ | ❌ | 发送的字节数 |
| `sendmsg` | ❌（可指定 address） | ❌ | ✅ | 发送的字节数 |
| `sendfile` | ✅ | —（发到 EOF） | ❌ | 发送的总字节数 |

所有发送函数的 `flags` 参数均默认为 `0`，含义同 `recv`。

## 1. send — 基础发送

```python
n = conn.send(b'hello')   # 套接字需已连接，返回发送的字节数
```

- 套接字**必须连接到**远程套接字
- 返回发送的字节数，但**不保证全部发送**
- 只传输部分数据时，应用需继续调用 `send` 发送剩余部分

## 2. sendall — 发送全部数据

```python
conn.sendall(b'hello')   # 持续发送直到全部发完或出错，成功返回 None
```

- 与 `send` 不同，会持续发送直到**所有数据都发送**或发生错误
- 成功返回 `None`；出错抛异常，且无法确定已发送多少数据
- 3.5 起，套接字超时指发送所有数据的**总时长上限**，而非单次发送

## 3. sendto — 发往指定地址

```python
n = sock.sendto(b'hello', ('1.2.3.4', 8080))   # 无需连接，目标由 address 指定
n = sock.sendto(b'hello', 0, ('1.2.3.4', 8080))  # 显式指定 flags
```

- 套接字**不应连接**远程套接字，目标由 `address` 参数指定
- 返回发送的字节数
- **UDP 无连接场景的标准发送方式**

## 4. sendmsg — 从多个缓冲区发送 + 辅助数据

```python
n = sock.sendmsg([b'head', b'body'], ancdata, flags, address)
```

- 从一系列缓冲区**收集**（gather）非辅助数据并连成一条消息发送
- `buffers`：类字节对象的可迭代对象
- `ancdata`：`(cmsg_level, cmsg_type, cmsg_data)` 可迭代对象；某些系统每次仅支持一条控制消息
- `address` 非 `None` 时为消息指定目标地址
- 返回发送的非辅助数据字节数

```python
import socket, array

def send_fds(sock, msg, fds):
    return sock.sendmsg([msg], [(socket.SOL_SOCKET, socket.SCM_RIGHTS, array.array("i", fds))])
```

- 与 `recvmsg` 配套，通过 AF_UNIX 的 `SCM_RIGHTS` 传递文件描述符

### sendmsg_afalg — AF_ALG 专用

`sendmsg` 的专用版本，用于 Linux 内核加密套接字（AF_ALG），设置模式、IV、AEAD 关联数据长度和标志。可用性：Linux >= 2.6.38。

## 5. sendfile — 高性能发送文件

```python
n = conn.sendfile(f, offset=0, count=None)   # 发送文件直到 EOF，返回总字节数
```

- 通过 `os.sendfile` 高性能发送文件，返回发送的总字节数
- `file` 必须是**二进制模式打开的常规文件对象**；`os.sendfile` 不可用或非常规文件时回退到 `send`
- `offset` 指定起始读取位置；`count` 指定传输总字节数，缺省则发到 EOF
- 文件位置在返回（或出错）时更新，出错后可用 `file.tell()` 得知已发送字节数
- 套接字必须是 `SOCK_STREAM`，**不支持非阻塞套接字**

## 6. 选择指南

| 场景 | 推荐函数 |
|---|---|
| TCP 发送数据（不关心是否发完） | `send` |
| TCP 发送数据（必须全部发完） | `sendall` |
| UDP 发送数据 | `sendto` |
| 发送控制消息 / 传递文件描述符 | `sendmsg` |
| 高性能发送文件 | `sendfile` |

## 小结

- 三个维度记忆：**目标地址**（`to`）、**保证发完**（`all`）、**辅助数据**（`msg`）
- 名字后缀即功能：`to` 指定地址、`all` 保证发完、`msg` 支持控制消息
- TCP 用 `send` / `sendall`，UDP 用 `sendto`，跨进程传 fd 用 `sendmsg`，发文件用 `sendfile`

## 延伸阅读

- [recv 系列函数详解]({% post_link socket-recv系列函数详解 %}) — 与发送函数对应的接收函数
