---
title: Ubuntu 时间同步客户端配置
date: 2026-09-14
updated: 2026-09-14
tags: [Ubuntu, NTP, chrony, systemd-timesyncd, Linux]
categories: linux
keywords: Ubuntu, NTP, chrony, systemd-timesyncd, timedatectl, 时间同步, 时钟校准
description: Ubuntu 作为客户端的时间同步配置：默认 systemd-timesyncd 与功能更强的 chrony 两套方案，涵盖 NTP 服务器配置、常用查询命令与排错要点。
top_img:
comments: true
cover: https://picsum.photos/id/142/800/450
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

Ubuntu 默认用 `systemd-timesyncd` 做轻量 SNTP 客户端，适合普通主机；需要更精确校准、漂移补偿或对外提供时间服务时，改用 `chrony`。二者只保留其一即可。

## 1. 查看同步状态

```bash
timedatectl                      # 顶部显示 System clock synchronized: yes/no
timedatectl timesync-status      # systemd-timesyncd 的同步详情
systemctl status systemd-timesyncd
```

- `timedatectl` 中 `System clock synchronized: yes` 表示时钟已同步，`NTP service: active` 表示已启用 NTP 同步
- `timesync-status` 展示当前服务器、偏移（Offset）、延迟（Delay）、抖动（Jitter）
- 未启用时用 `timedatectl set-ntp true` 开启（默认即启用）

## 2. systemd-timesyncd 配置

配置文件 `/etc/systemd/timesyncd.conf`，编辑 `[Time]` 段后重启服务。

```ini
[Time]
NTP=ntp.aliyun.com ntp.tencent.com
FallbackNTP=ntp.ubuntu.com pool.ntp.org
RootDistanceMaxSec=5
```

```bash
sudo systemctl restart systemd-timesyncd
timedatectl timesync-status
```

| 选项 | 说明 |
| --- | --- |
| `NTP=` | 主 NTP 服务器，空格分隔多个，留空则用系统默认 |
| `FallbackNTP=` | `NTP=` 为空或不可达时兜底 |
| `RootDistanceMaxSec=` | 可接受的最大根距离（秒），默认 5 |
| `PollIntervalMinSec=` / `PollIntervalMaxSec=` | 轮询间隔上下限 |

- 修改后必须 `restart`，`reload` 不会重新读取 NTP 列表
- 国内服务器用 `ntp.aliyun.com`、`ntp.tencent.com` 延迟更低；通用用 `ntp.ubuntu.com`、`pool.ntp.org`
- 校验配置：`systemd-analyze cat-config systemd/timesyncd.conf` 查看合并后的有效配置；出错信息看 `journalctl -u systemd-timesyncd -b`

### 2.1 drop-in 覆盖（推荐）

直接改主文件一旦配错不易回滚。用 drop-in 目录 `/etc/systemd/timesyncd.conf.d/` 放一个 `.conf` 文件覆盖同名键，主文件保持默认，出错删除该文件即可恢复。

```bash
sudo mkdir -p /etc/systemd/timesyncd.conf.d
sudo tee /etc/systemd/timesyncd.conf.d/ntp.conf <<'EOF'
[Time]
NTP=ntp.aliyun.com ntp.tencent.com
EOF
sudo systemctl restart systemd-timesyncd
```

- drop-in 与主配置合并，同名键后者覆盖，无需改动 `/etc/systemd/timesyncd.conf`
- 恢复默认：删除该 drop-in 文件后 `restart` 即回到系统默认

## 3. chrony 配置（功能更强）

需要漂移补偿、作为局域网时间源，或对精度有要求时用 chrony。

```bash
sudo apt install chrony        # 安装后通常会自动停用 systemd-timesyncd
sudo systemctl enable --now chrony
```

主配置 `/etc/chrony/chrony.conf`：

```text
# 时间源：pool 自动选择就近节点，server 指定固定源
pool ntp.aliyun.com iburst
server ntp.tencent.com iburst

driftfile /var/lib/chrony/chrony.drift   # 记录本机时钟漂移速率
makestep 1.0 3                            # 前 3 次更新内，偏差>1s 直接步进
maxdistance 3.0                           # 可接受的最大根距离（秒），等价 timesyncd 的 RootDistanceMaxSec
maxslewrate 1000                          # 渐进调整（slew）速率上限（ppm），限制时钟漂移速度
rtcsync                                   # 将系统时间回写硬件时钟 RTC
```

| 指令 | 说明 |
| --- | --- |
| `pool` / `server` | 时间源；`pool` 自动就近选择，`server` 固定源，加 `iburst` 加速首次同步 |
| `driftfile` | 漂移速率文件路径，重启后据此快速收敛 |
| `makestep` | `makestep <阈值> <次数>`：前 N 次更新内偏差超阈值直接步进 |
| `maxdistance` | 可接受的最大根距离（秒），默认 3，超出的源被拒绝 |
| `maxslewrate` | slew 速率上限（ppm），默认 83333，设小可减缓调整过快 |
| `rtcsync` | 同步时把系统时间回写硬件时钟 RTC |

```bash
sudo systemctl restart chrony
chronyc sources -v     # 查看时间源及选中状态（^* 表示正在使用）
chronyc tracking       # 当前同步状态：偏移、误差、层级
chronyc makestep       # 立即强制步进一次
```

- `iburst` 启动后立即连续发起请求，加快首次同步
- `sources` 中 `^*` 为当前选中源，`^+` 为候选源，`^?` 为不可达
- `makestep` 使时钟跳变，日志、监控可能因此出现时间断层；追求平滑用默认的渐进调整（slew），并可调低 `maxslewrate`
- 校验配置：`chronyd -p` 打印解析后的配置并退出，语法错误会直接报错

### 3.1 include 引入 drop-in（可选）

chrony 没有 systemd 式的自动 drop-in 目录，但可用 `include` 指令加载额外配置文件，同样做到改动隔离、删除即恢复。Ubuntu 默认配置不含该行，需手动在 `/etc/chrony/chrony.conf` 顶部加：

```text
include /etc/chrony/conf.d/*.conf
```

```bash
sudo mkdir -p /etc/chrony/conf.d
sudo tee /etc/chrony/conf.d/ntp.conf <<'EOF'
server ntp.aliyun.com iburst
EOF
sudo systemctl restart chrony
```

- `include` 支持通配符，可一次加载多个文件
- chrony 4.0+ 的 `sourcedir` 指令等价于 `include 目录/*.sources`，专用于时间源文件
- 恢复默认：删除对应文件后 `restart`

## 4. 选型与注意

| 场景 | 推荐 |
| --- | --- |
| 普通桌面/服务器，仅需保持大致准确 | systemd-timesyncd（默认，零配置） |
| 需要高精度、漂移补偿 | chrony |
| 作为局域网 NTP 服务器对外供时 | chrony |

注意：systemd-timesyncd 与 chrony（或 ntpd）不能同时运行，装 chrony 后确认 timesyncd 已停用。

```bash
systemctl is-active systemd-timesyncd chrony   # 应只有 chrony 为 active
```

## 5. 小结

- 默认 `systemd-timesyncd` 足够，改 `/etc/systemd/timesyncd.conf` 的 `NTP=` 后重启即可
- 高精度或对外供时用 `chrony`，通过 `chronyc sources` / `tracking` 验证
- 始终只保留一个时间同步客户端，避免相互干扰
