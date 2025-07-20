---
title: docker + clash
date: 2025.06.09
updated: 2025.06.09
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

```
#!/bin/bash

set -e

echo "🔧 安装 Privoxy..."
sudo apt update
sudo apt install -y privoxy

echo "📝 配置 Privoxy 转发 SOCKS5 到 HTTP..."

# 添加配置
sudo tee /etc/privoxy/config > /dev/null <<EOF
listen-address 127.0.0.1:8118
forward-socks5   /               127.0.0.1:7897 .
EOF

echo "🔁 重启 Privoxy..."
sudo systemctl restart privoxy
sudo systemctl enable privoxy

echo "🐳 配置 Docker 使用 HTTP 代理..."

sudo mkdir -p /etc/systemd/system/docker.service.d

sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf > /dev/null <<EOF
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:8118"
Environment="HTTPS_PROXY=http://127.0.0.1:8118"
EOF

echo "🔄 重新加载 systemd 并重启 Docker 服务..."
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart docker

echo "✅ 完成！你现在可以使用 Docker 拉取镜像，比如："
echo "   docker pull hello-world"

```