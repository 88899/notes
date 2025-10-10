---
title: 快速定位和解决Ubuntu 服务器磁盘占用问题，完整排查与清理指南
lastmod: 2025-10-10
date: 2025-10-10
description: 在服务器运维中，“磁盘空间满了” 是高频问题。以下将以一个真实场景为例，从发现磁盘占满开始，一步步教你定位占用源、分析原因，并安全清理，全程附具体操作指令和效果验证，新手也能跟着做。
author: AiEDU
categories:
  - DevOps
tags:
  - 技术
series: 免费VPS
cover: https://cdn.jsdelivr.net/gh/dgdghub/dg-pic@main/blog/20251010095940614.png
---

在服务器运维中，“磁盘空间满了” 是高频问题。以下将以一个真实场景为例，从发现磁盘占满开始，一步步教你定位占用源、分析原因，并安全清理，全程附具体操作指令和效果验证，新手也能跟着做。

## 一、发现问题：磁盘空间告警


又收到了服务器监控告警：“根分区 `/` 使用率达 98%”，登录服务器后，首先需要确认磁盘占用的整体情况。

## 二、第一步：确认哪个分区被占满

磁盘满了不一定是根分区，可能是 `/var`、`/home` 等其他分区，第一步必须明确目标。

### 操作指令

bash

```bash
df -h
```

### 示例


```plaintext
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   48G  1.2G  98% /          # 根分区满了
/dev/sdb1       100G   30G   70G  30% /data
tmpfs           3.9G     0  3.9G   0% /dev/shm
```

### 分析

从 `Use%` 列可见，根分区 `/` 使用率 98%，是需要排查的目标。

## 三、第二步：定位根分区下的大目录

知道根分区满了后，需要找出 `/` 目录下哪些子目录占用空间最大，缩小排查范围。

### 操作指令

```bash
# 查看根目录下一级目录的大小（忽略权限错误）
sudo du -sh /* 2>/dev/null
```

### 示例


```plaintext
8.0K    /bin
4.0K    /boot
2.7G    /data
4.0K    /dev
1.2G    /etc
5.3G    /home
0       /init
8.0M    /lib
4.0K    /lib64
16K     /lost+found
4.0K    /media
4.0K    /mnt
12G     /opt
du: cannot access '/proc/1234/task/1234/fd/4': No such file or directory  # 可忽略
0       /proc
4.0K    /root
1.2M    /run
8.0K    /sbin
4.0K    /srv
0       /sys
4.0K    /tmp
28G     /var       # 重点！/var 目录占用 28G
4.5G    /usr
```

### 分析

`/var` 目录占用 28G，是根分区下最大的目录，下一步重点排查 `/var`。

## 四、第三步：深入大目录，逐层排查

进入 `/var` 目录，继续用 `du` 命令找子目录中的 “空间大户”。

### 操作指令


```bash
# 进入/var目录
cd /var
# 查看/var下一级目录的大小
sudo du -sh * 2>/dev/null
```

### 输出示例


```plaintext
4.0K    backups
2.1G    cache
4.0K    crash
4.0K    lib
25G     lib/docker  # 重点！/var/lib/docker 占用 25G
4.0K    local
0       lock
8.0K    log
4.0K    mail
4.0K    opt
0       run
4.0K    spool
4.0K    tmp
```

### 分析

`/var/lib/docker` 占用 25G，而 Docker 的数据通常存在这里，继续排查 Docker 相关目录。

## 五、第四步：定位 Docker 占用的具体位置

Docker 的核心数据目录是 `overlay2`（默认存储驱动），查看其大小：

### 操作指令


```bash
# 查看/var/lib/docker下的子目录大小
cd /var/lib/docker
sudo du -sh * 2>/dev/null
```

### 输出示例


```plaintext
4.0K    builder
4.0K    buildkit
4.0K    containers
4.0K    image
4.0K    network
24G     overlay2  # 重点！overlay2 占用 24G
4.0K    plugins
4.0K    runtimes
4.0K    swarm
4.0K    tmp
4.0K    trust
4.0K    volumes
```

### 分析

`overlay2` 目录占用 24G，这是 Docker 存储镜像和容器数据的核心目录（原因：冗余镜像、停止的容器、运行时数据等）。

## 六、第五步：分析占用原因并安全清理

### 原因分析

`overlay2` 占用大的常见原因：

- 积累了大量未使用的 Docker 镜像（如旧版本、构建失败的镜像）；
- 停止的容器未删除（其读写层数据仍保留）；
- 构建镜像时的缓存未清理；
- 运行中的容器产生大量日志或临时文件。

### 实操清理步骤

#### 步骤 1：查看 Docker 可回收资源

先通过 `docker system df` 确认哪些资源可以清理：


```bash
docker system df
```

#### 输出示例


```plaintext
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          15        3         18G       15G (83%)  # 可回收15G
Containers      8         2         2.5G      2.0G (80%) # 可回收2.0G
Local Volumes   5         1         1.2G      1.0G (83%) # 可回收1.0G
Build Cache     20        0         3.5G      3.5G (100%)# 可回收3.5G
```

#### 步骤 2：一键清理所有未使用资源（推荐）

用 Docker 自带的清理命令，安全删除未被使用的资源：


```bash
# 清理未被任何容器使用的镜像、容器、网络、缓存（-a 包含未引用镜像，-f 强制清理）
docker system prune -a -f
```

#### 可选：清理未使用的卷（谨慎！卷可能含关键数据）

如果确认卷中无重要数据，可加上 `--volumes` 参数：


```bash
docker system prune -a -f --volumes
```

#### 步骤 3：验证清理效果

清理后再次查看 `overlay2` 大小：


```bash
sudo du -sh /var/lib/docker/overlay2
```

#### 输出对比

- 清理前：`24G /var/lib/docker/overlay2`
- 清理后：`5.2G /var/lib/docker/overlay2`（释放约 18.8G，符合预估的可回收空间）

## 七、其他常见占用场景及处理（附案例）

除了 Docker，磁盘满还可能是其他原因，以下是两个高频场景：

### 场景 1：日志文件过大（如 `/var/log`）

#### 排查


```bash
# 查看/var/log下的大文件（大于100M）
sudo find /var/log -type f -size +100M 2>/dev/null
```

#### 输出示例


```plaintext
/var/log/syslog  # 系统日志，大小 2.3G
/var/log/auth.log # 认证日志，大小 1.8G
```

#### 原因

服务频繁报错（如应用程序崩溃）导致日志疯狂写入，或日志未配置轮转。

#### 处理


```bash
# 安全清空日志（保留文件，避免进程写入失败）
sudo truncate -s 0 /var/log/syslog
sudo truncate -s 0 /var/log/auth.log

# 长期解决：配置logrotate（示例：编辑syslog的轮转配置）
sudo nano /etc/logrotate.d/rsyslog
# 确保配置了 size 限制（如 size 100M）和保留份数（如 rotate 7）
```

### 场景 2：已删除但被进程占用的文件

#### 问题表现

明明删除了大文件，`df -h` 显示空间仍满（因为文件被进程占用，未释放）。

#### 排查


```bash
# 查找已删除但被进程占用的文件
sudo lsof +L1
```

#### 输出示例


```plaintext
COMMAND  PID   USER   FD   TYPE DEVICE SIZE/OFF NLINK NAME
nginx   1234   root    1w  REG    8:1  20971520     0 /var/log/nginx/access.log (deleted)
```

#### 处理

重启占用文件的进程，释放空间：


```bash
sudo systemctl restart nginx
```

## 八、总结：完整排查流程

1. **确认满的分区**：`df -h` → 找到 `Use%` 接近 100% 的分区（如 `/`）。
2. **定位大目录**：`sudo du -sh /* 2>/dev/null` → 找出分区下的大目录（如 `/var`）。
3. **逐层深入**：进入大目录，重复 `sudo du -sh *` → 定位到具体目录（如 `/var/lib/docker/overlay2`）。
4. **分析原因**：根据目录类型判断占用源（Docker 资源、日志、临时文件等）。
5. **安全清理**：
    - Docker 资源：`docker system prune -a -f`；
    - 日志文件：`truncate -s 0 文件名`；
    - 已删除的占用文件：重启对应进程。
6. **验证效果**：再次用 `df -h` 或 `du -sh` 确认空间释放。

通过以上步骤，既能精准定位磁盘占用源，又能安全清理，避免误删系统文件导致服务异常。记得定期清理（如加入定时任务），防止磁盘再次占满。