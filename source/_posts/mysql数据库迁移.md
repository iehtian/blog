---
title: MySQL 数据库迁移（Docker → Docker）
tags:
  - MySQL
  - Docker
  - 数据库迁移
  - 备份与恢复
comments: true
toc: true
toc_number: true
toc_style_simple: false
copyright: true
copyright_author: iehtian
mathjax: false
katex: false
aplayer: false
highlight_shrink: false
aside: true
abcjs: false
noticeOutdate: false
date: 2025-12-01 23:26:46
updated: "2025-12-02 23:09:31"
categories:
  - 数据库
  - 运维
keywords: MySQL, Docker, 数据库迁移, mysqldump, 备份, 恢复
description: 使用 mysqldump 将 MySQL 从旧 Docker 容器迁移到新容器的完整步骤、最佳实践与常见问题排查示例。
top_img:
cover:
copyright_author_href:
copyright_url:
copyright_info:
---

本文记录如何用 MySQL 自带的导入导出工具（mysqldump/mysql）在 Docker 环境下进行“容器到容器”的数据库迁移。示例数据库名为 `dateplan`，容器名为 `dateplan-db`。

docker composer 如下

```yaml
services:
  dateplan-db:
    image: mysql
    container_name: dateplan-db
    environment:
      MYSQL_ROOT_PASSWORD: iehtian
      MYSQL_DATABASE: dateplan
      MYSQL_USER: dateplan_user
      MYSQL_PASSWORD: dateplan_pass
    ports:
      - "3308:3306"
    volumes:
      - dateplan-db-data:/var/lib/mysql
    restart: unless-stopped
volumes: dateplan-db-data:ubuntu
```

把这个 docker 容器连同数据一并迁移至另一个服务器

1.在旧服务器（Current Server）备份数据
首先，在当前目录下运行以下命令，将数据库导出为一个 SQL 文件。

```bash
# 使用 docker exec 执行内部的 mysqldump
# 密码即 docker-compose 中配置的 MYSQL_ROOT_PASSWORD
docker exec dateplan-db mysqldump -u root -piehtian dateplan > dateplan_backup.sql
```

2. 传输文件到新服务器
   你需要传输两个文件：docker-compose.yml 和 dateplan_backup.sql。

```bash
# 假设新服务器 IP 为 1.2.3.4，用户名为 user
scp docker-compose.yml dateplan_backup.sql user@1.2.3.4:~/booking_system/
```
 
3. 在新服务器（New Server）启动并恢复
   登录到新服务器，进入目录：

```bash
# 1. 启动容器
docker-compose up -d
# 2. 等待几十秒，确保 MySQL 完全启动（查看日志确认：docker logs dateplan-db）
# 3. 导入数据
cat dateplan_backup.sql | docker exec -i dateplan-db mysql -u root -piehtian dateplan
```
