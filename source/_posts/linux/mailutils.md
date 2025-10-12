---
title: 使用 mailutils 配合 Postfix 通过 QQ SMTP 发送邮件
date: 2025.10.12
updated:
tags: [linux]
categories:
keywords:
description: 本文演示在 Debian/Ubuntu 上通过 Postfix 使用 QQ SMTP 发送邮件的基础配置。
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

## 1. 安装

```sh
sudo apt install mailutils -y
```

## 2. Postfix 配置向导

按向导逐步选择/填写：

1. 选择配置类型：第三个选项  
   ![Postfix 配置界面](PixPin_2025-10-12_17-10-16.png)

2. 使用默认值  
   ![Postfix 配置界面](PixPin_2025-10-12_17-23-44.png)

3. 填写中继主机（relayhost）：`[smtp.qq.com]:587`  
   ![Postfix 配置界面](PixPin_2025-10-12_19-23-35.png)

4. 指定 root 邮件接收人（本地用户名或外部邮箱），此处示例为本地用户名  
   ![Postfix 配置界面](PixPin_2025-10-12_19-24-59.png)

5. 指定应投递到本地的域名。仅发不收，留空即可  
   ![Postfix 配置界面](PixPin_2025-10-12_17-33-08.png)

6. 系统策略：默认即可  
   ![Postfix 配置界面](PixPin_2025-10-12_17-34-56.png)

7. 允许作为中继的网段：仅本地环回（默认）  
   ![Postfix 配置界面](PixPin_2025-10-12_17-35-57.png)

8. 本地邮箱文件大小限制：使用默认的无限制  
   ![Postfix 配置界面](PixPin_2025-10-12_17-38-12.png)

9. 本地地址扩展分隔符：留空或默认  
   ![Postfix 配置界面](PixPin_2025-10-12_17-38-23.png)

10. 网络协议监听：默认 all 即可

## 3. 在 main.cf 中开启 TLS 与 SASL

编辑配置并追加以下内容：

```conf
# 以下为 TLS 与 SASL 认证配置（relayhost 已在向导中设置为 [smtp.qq.com]:587）
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
# 可选：增加调试信息
# debug_peer_level = 2
```

```sh
sudo vim /etc/postfix/main.cf
```

## 4. 配置 QQ 邮箱授权信息

编辑密码文件并生成映射、设置权限：

```sh
sudo vim /etc/postfix/sasl_passwd
```

文件内容示例（请替换为你的邮箱与授权码）：

```conf
[smtp.qq.com]:587 youremail@qq.com:你的授权码
```

生成映射并限权：

```sh
sudo postmap /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd*
```

## 5. 配置发件地址映射（generic）

先查看当前主机名：

```sh
grep "^myhostname" /etc/postfix/main.cf
# 内容示例：myhostname = iehtian-VMware-Virtual-Platform
```

编辑 generic 映射：

```sh
sudo vim /etc/postfix/generic
```

内容示例（将本机账户映射为你的 QQ 邮箱）：

```conf
iehtian@iehtian-VMware-Virtual-Platform youremail@qq.com
@iehtian-VMware-Virtual-Platform youremail@qq.com
```

## 6. 重启服务

```sh
sudo systemctl restart postfix
```

## 7. 发送测试邮件

```sh
echo "测试邮件 $(date)" | mail -s "测试邮件" destmail@qq.com
```
