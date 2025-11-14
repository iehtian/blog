---
title: Wakapi Docker 本地部署
date: 2025-11-13
updated: 2025-11-13
tags: [Wakapi, WakaTime, Docker, 开发效率]
categories: DevOps
comments: true
toc: true
toc_number: true
copyright: true
copyright_author: iehtian
description: 使用 Docker Compose 在本地快速部署 Wakapi 并进行持久化与安全加固的完整指南
cover:
---

# Wakapi Docker 本地部署指南

## 1. 前言

Wakapi 是一个自托管的 WakaTime 兼容后端，用于统计你的编码时长、语言分布与编辑器使用情况。适合想掌握个人开发效率又不愿将数据放云端的用户。

## 2. Wakapi 是什么

- 作用: 接收各类编辑器/IDE 的 WakaTime 插件或 Wakapi 客户端发送的心跳 (heartbeats)。
- 展示: 提供网页仪表盘，统计语言、项目、编辑器使用情况。
- 兼容性: 大多数使用 WakaTime API 的客户端可直接指向自托管地址。

## 3. 前置条件

- 已安装 Docker 与 docker-compose (Compose V2 亦可)。
- 端口 3000 未被占用。
- 基础 Linux 命令行操作能力。
- 机器资源: 低占用（几百 MB 内存即可）。

## 4. 推荐目录结构

```
wakapi/
├─ docker-compose.yml
├─ data/          # Wakapi 持久化数据
└─ mysql_data/    # MySQL 数据
```

## 5. docker-compose.yml 解析

下方示例在原版本基础上补充注释与建议。生产环境请替换密码与盐值并使用 .env 管理敏感信息。  
(原文件内容已增强)

```yaml
version: "3.8"

services:
  db:
    image: mysql:8
    container_name: wakapi_db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpass # 修改为强密码
      MYSQL_DATABASE: wakapi
      MYSQL_USER: wakapi
      MYSQL_PASSWORD: wakapi_pass # 修改为强密码
    volumes:
      - ./mysql_data:/var/lib/mysql # 持久化 MySQL 数据
    command: ["--default-authentication-plugin=mysql_native_password"] # 提升兼容性，可选

  wakapi:
    image: ghcr.io/muety/wakapi:latest
    container_name: wakapi
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "3000:3000" # 外部访问端口
    environment:
      WAKAPI_DB_TYPE: "mysql"
      WAKAPI_DB_NAME: "wakapi"
      WAKAPI_DB_USER: "wakapi"
      WAKAPI_DB_PASSWORD: "wakapi_pass"
      WAKAPI_DB_HOST: "db"
      WAKAPI_DB_PORT: "3306"
      WAKAPI_ENV: "prod"
      WAKAPI_PASSWORD_SALT: "replace-with-random-salt" # 使用: openssl rand -hex 32
      WAKAPI_PUBLIC_URL: "http://localhost:3000" # 若反向代理改为 https://your.domain
      # 可选: WAKAPI_MAIL_* / WAKAPI_LOG_LEVEL 等变量
    volumes:
      - ./data:/data # Wakapi 自身数据 (心跳等)
```

## 6. 使用 .env（示例）

不创建真实文件，仅示例:

```
MYSQL_ROOT_PASSWORD=强随机
MYSQL_PASSWORD=强随机
WAKAPI_DB_PASSWORD=强随机
WAKAPI_PASSWORD_SALT=$(openssl rand -hex 32)
```

然后 docker-compose.yml 中以 ${VAR} 引用。

## 7. 部署步骤

1. 创建目录: mkdir wakapi && cd wakapi
2. 写入 docker-compose.yml（按上方示例）。
3. 生成盐值: openssl rand -hex 32 替换 WAKAPI_PASSWORD_SALT。
4. 启动: docker compose up -d
5. 查看日志: docker compose logs -f wakapi
6. 访问: http://localhost:3000
7. 注册首个账户（将作为你个人使用的主账号）。

## 8. 客户端配置

在 IDE 安装 WakaTime 插件:

- 设置 API Endpoint: http://localhost:3000
- 使用 Wakapi 页面生成的 API Key（与 WakaTime 类似）。
  若插件不支持修改 Endpoint，可使用专用 Wakapi 轻量客户端。

## 9. 更新与升级

1. 拉取镜像: docker compose pull
2. 重建: docker compose up -d
3. 清理旧镜像: docker image prune -f

## 10. 备份策略

- 定期 tar 打包 mysql_data 与 data 目录。
- 或使用 mysqldump:
  mysqldump -h 127.0.0.1 -u wakapi -p wakapi > wakapi.sql
- 建议通过 cron 周期化。

## 11. 安全与加固

- 使用强随机密码与 32+ 字节盐值。
- 通过反向代理 (Nginx + HTTPS) 暴露外网。
- 若仅本地使用，可不映射 0.0.0.0，改用 network / SSH 隧道。
- 限制 MySQL 仅内部访问（当前已通过服务间网络实现）。
- 定期更新镜像，关注上游发布。

## 12. 常见问题

Q: wakapi 启动后页面空白?  
A: 检查浏览器请求与容器日志，确认数据库连接是否成功。

Q: MySQL 报认证错误?  
A: 确认用户名、密码与 MYSQL\_\* 变量一致，并是否用 mysql_native_password 插件。

Q: 更换域名后统计失败?  
A: 更新 WAKAPI_PUBLIC_URL 并重启；客户端需重新指向新地址。

## 13. 迁移

- 停止服务: docker compose down
- 复制 data 与 mysql_data 到新服务器。
- 确保版本一致，重新启动。

## 14. 下一步

- 配置反向代理 + HTTPS。
- 开启邮件通知（需要 SMTP 环境变量）。
- 对接 Grafana（高级需求，可使用导出数据构建图表）。

## 15. 快速参考命令

- 查看运行状态: docker compose ps
- 实时日志: docker compose logs -f
- 进入容器: docker exec -it wakapi sh
- 统计磁盘占用: du -sh data mysql_data
