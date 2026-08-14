---
title: Python socket 模块 recv 系列函数详解
date: 2026-08-14
updated: 2026-08-14
tags: [Python, socket, 网络编程]
categories: Python
keywords: Python, socket, recv, recvfrom, recvmsg, recv_into, recvfrom_into, recvmsg_into, 网络编程
description: Python socket 模块 recv 系列六个接收函数（recv、recvfrom、recvmsg、recv_into、recvfrom_into、recvmsg_into）的作用、返回值与区别详解
toc: true
toc_number: true
copyright: true
cover: https://picsum.photos/id/300/800/450
---

# Python socket 模块 recv 系列函数详解

`socket` 模块提供 6 个接收函数，可沿三个维度区分：

1. **是否返回发送方地址** —— `recvfrom` / `recvfrom_into` / `recvmsg` / `recvmsg_into` 返回，`recv` / `recv_into` 不返回
2. **是否写入调用方提供的缓冲区（零拷贝）** —— `recv_into` / `recvfrom_into` / `recvmsg_into` 写入已有 buffer，其余创建新 `bytes`
3. **是否接收辅助数据（控制消息）** —— 仅 `recvmsg` / `recvmsg_into` 支持

| 函数 | 返回地址 | 写入已有缓冲区 | 辅助数据 | 返回值 |
|---|---|---|---|---|
| `recv` | ❌ | ❌ | ❌ | `bytes` |
| `recvfrom` | ✅ | ❌ | ❌ | `(bytes, address)` |
| `recv_into` | ❌ | ✅ | ❌ | `nbytes` |
| `recvfrom_into` | ✅ | ✅ | ❌ | `(nbytes, address)` |
| `recvmsg` | ✅ | ❌ | ✅ | `(data, ancdata, msg_flags, address)` |
| `recvmsg_into` | ✅ | ✅（分散写入） | ✅ | `(nbytes, ancdata, msg_flags, address)` |

所有函数的 `flags` 参数均默认为 `0`，含义参见 Unix 手册页 `recv(2)`。

## 1. recv — 基础接收

```python
data = conn.recv(1024)   # 最多接收 1024 字节，返回 bytes
```

- 返回**最多** `bufsize` 字节的 `bytes` 对象
- 返回空 `b''` 表示对端已关闭连接（TCP 半关闭）
- 面向连接（TCP）场景最常用，无需关心发送方地址

## 2. recvfrom — 带地址接收

```python
data, addr = sock.recvfrom(1024)
# addr 为发送方地址，格式取决于地址族（IPv4 → ('1.2.3.4', port)）
```

- 与 `recv` 相同，但额外返回发送方地址 `(bytes, address)`
- **UDP 无连接，必须用 `recvfrom` 才能知道数据来自谁**

## 3. recv_into / recvfrom_into — 写入已有缓冲区

```python
buf = bytearray(1024)
n = sock.recv_into(buf)            # 写入 buf，返回字节数
n, addr = sock.recvfrom_into(buf)  # 额外返回地址
```

- 把数据写入调用方提供的 `buffer`，而不是新建 `bytes`
- 返回接收到的字节数 `nbytes`（`recvfrom_into` 额外返回地址）
- `nbytes` 缺省或为 0 时，接收缓冲区可用大小
- 循环接收大量数据时避免反复分配内存，性能更好

## 4. recvmsg — 接收辅助数据（控制消息）

```python
data, ancdata, msg_flags, addr = sock.recvmsg(1024, ancbufsize)
```

返回值是 4 元组：

| 项 | 类型 | 说明 |
|---|---|---|
| `data` | `bytes` | 常规数据 |
| `ancdata` | `list` | `(cmsg_level, cmsg_type, cmsg_data)` 列表，辅助数据 |
| `msg_flags` | `int` | 接收消息条件的各种标志按位或 |
| `address` | — | 发送方地址（未连接时可用，否则值未指定） |

- `ancbufsize` 决定辅助数据接收缓冲区大小，默认 `0` 表示不接收辅助数据
- 典型用途：AF_UNIX 下通过 `SCM_RIGHTS` 在进程间传递文件描述符
- 若 `recvmsg` 抛异常，会先尝试关闭已接收到的文件描述符

```python
import socket, array

def recv_fds(sock, msglen, maxfds):
    fds = array.array("i")
    msg, ancdata, flags, addr = sock.recvmsg(msglen, socket.CMSG_LEN(maxfds * fds.itemsize))
    for cmsg_level, cmsg_type, cmsg_data in ancdata:
        if cmsg_level == socket.SOL_SOCKET and cmsg_type == socket.SCM_RIGHTS:
            fds.frombytes(cmsg_data[:len(cmsg_data) - (len(cmsg_data) % fds.itemsize)])
    return msg, list(fds)
```

- 辅助数据缓冲区大小可用 `CMSG_SPACE()` 或 `CMSG_LEN()` 计算
- 部分接收的辅助数据若超出缓冲区末尾，会发出 `RuntimeWarning` 并返回缓冲区内的部分

## 5. recvmsg_into — 分散写入多个缓冲区

```python
s1, s2 = socket.socketpair()
b1 = bytearray(b'----')
b2 = bytearray(b'0123456789')
b3 = bytearray(b'--------------')
s1.send(b'Mary had a little lamb')
n, ancdata, msg_flags, addr = s2.recvmsg_into([b1, memoryview(b2)[2:9], b3])
# n == 22；数据依次填满 b1 → b2[2:9] → b3（b1=b'Mary', b3=b'little lamb---'）
```

- 行为同 `recvmsg`，但把非辅助数据**分散**写入一系列缓冲区（scatter），而非返回新 `bytes`
- `buffers` 必须是可写缓冲区的可迭代对象（如 `bytearray`、`memoryview`）
- 返回 `nbytes`（写入的总字节数）替代 `data`
- 缓冲区数量受 `SC_IOV_MAX` 限制

## 6. 选择指南

| 场景 | 推荐函数 |
|---|---|
| TCP 连接接收数据 | `recv` |
| UDP 接收数据（需发送方地址） | `recvfrom` |
| 循环接收大流量，避免反复分配内存 | `recv_into` / `recvfrom_into` |
| 传递文件描述符 / 接收控制消息 | `recvmsg` |
| 接收控制消息且需分散写入多个缓冲区 | `recvmsg_into` |

## 小结

- 三个维度记忆：**地址**（`from`）、**缓冲区**（`into`）、**辅助数据**（`msg`）
- 名字后缀即功能：`from` 带地址、`into` 写缓冲区、`msg` 支持控制消息
- TCP 用 `recv`，UDP 用 `recvfrom`，高性能用 `*_into`，跨进程传 fd 用 `recvmsg`

## 延伸阅读

- [send 系列函数详解]({% post_link socket-send系列函数详解 %}) — 与接收函数对应的发送函数
