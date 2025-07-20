---
title: socat
date: 2025.06.06
updated: 2025.06.06
tags: [linux]
categories: 
keywords:
description:
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

# Socat

## 简介

Socat (Socket CAT) 是一个多功能的网络工具，可以建立两个双向数据流并在它们之间传输数据。它可以创建多种类型的连接，包括文件、管道、设备、socket、程序等，是一个强大的网络"瑞士军刀"。

## 安装方法

### Linux
```bash
# Debian/Ubuntu
sudo apt-get install socat

# CentOS/RHEL
sudo yum install socat

# Fedora
sudo dnf install socat

# Arch Linux
sudo pacman -S socat
```

### macOS
```bash
# 使用Homebrew
brew install socat
```

### Windows
在Windows上可以通过以下方式使用socat:
- 在WSL (Windows Subsystem for Linux)中安装
- 使用Cygwin或MSYS2并安装socat包

## 基本语法

```
socat [选项] <地址1> <地址2>
```

socat会创建两个流，并在它们之间转发数据。

## 常用示例

### 1. 简单TCP端口转发

将本地8080端口转发到远程服务器的80端口:

```bash
socat TCP-LISTEN:8080,fork TCP:远程服务器:80
```

- `TCP-LISTEN:8080,fork`: 在本地8080端口监听TCP连接，fork选项使每个连接都创建一个新进程
- `TCP:远程服务器:80`: 连接到远程服务器的80端口

### 2. 创建一个简单的回显服务器

```bash
socat TCP-LISTEN:8000,fork EXEC:'/bin/cat'
```

这会创建一个回显服务器，将客户端发送的数据原样返回。

### 3. 文件传输

将文件内容发送到TCP连接:

```bash
socat -u FILE:/path/to/file TCP:目标主机:端口
```

接收文件:

```bash
socat -u TCP-LISTEN:端口,fork OPEN:/path/to/destination,creat
```

### 4. 创建虚拟串口

创建两个连接的虚拟串口（PTY）:

```bash
socat -d -d PTY,raw,echo=0,link=/tmp/ttyS0 PTY,raw,echo=0,link=/tmp/ttyS1
```

### 5. 代理服务器

HTTP代理:

```bash
socat TCP-LISTEN:8080,fork,reuseaddr PROXY:代理服务器:目标网站:80
```

### 6. 创建SSL/TLS连接

作为SSL/TLS客户端:

```bash
socat - OPENSSL:服务器:端口,verify=0
```

作为SSL/TLS服务器:

```bash
socat OPENSSL-LISTEN:端口,cert=server.pem,verify=0,fork -
```

### 7. 调试工具

显示所有传输的数据（十六进制格式）:

```bash
socat -x TCP-LISTEN:端口,fork TCP:目标主机:目标端口
```

### 8. UDP转发

```bash
socat UDP-LISTEN:本地端口,fork UDP:目标主机:目标端口
```

### 9. 连接Unix域套接字

```bash
socat - UNIX-CONNECT:/path/to/socket
```

### 10. 创建命名管道(FIFO)

```bash
socat -u TCP-LISTEN:端口,fork PIPE:/path/to/pipe
```

## 高级选项

### 主要地址类型

- `TCP`, `TCP4`, `TCP6`: TCP连接
- `UDP`, `UDP4`, `UDP6`: UDP连接
- `UNIX`, `UNIX-DGRAM`: Unix域套接字
- `FILE`: 文件
- `STDIO`: 标准输入/输出
- `EXEC`: 执行程序
- `SYSTEM`: 执行shell命令
- `PTY`: 伪终端
- `PIPE`: 管道
- `OPENSSL`: SSL/TLS连接

### 常用选项

- `fork`: 为每个连接创建一个新进程
- `reuseaddr`: 允许重用本地地址
- `bind`: 绑定到特定地址
- `range`: 限制允许连接的IP范围
- `linger`: 控制连接关闭行为
- `tcpwrap`: 使用TCP包装器
- `retry`: 连接失败后重试
- `waitlock`: 等待锁

## 调试技巧

- `-d`, `-dd`, `-ddd`: 增加调试级别
- `-v`: 显示详细信息
- `-x`: 以十六进制显示传输的数据

## 总结

socat是一个功能强大的网络工具，可以用于端口转发、代理服务器、串口测试、SSL/TLS连接、文件传输等多种场景。通过组合不同的地址类型和选项，socat几乎可以满足任何网络连接需求。

## 参考资料

- [Socat官方手册](http://www.dest-unreach.org/socat/doc/socat.html)
- [Socat示例](http://www.dest-unreach.org/socat/doc/socat-ttyoptions.txt)