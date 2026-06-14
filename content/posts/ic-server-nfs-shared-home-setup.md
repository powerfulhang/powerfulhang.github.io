---
title: "IC 服务器 NFS 共享 Home 目录配置"
date: 2026-06-14T14:10:00+08:00
draft: false
author: "SiliconBlog"
tags: ["NFS", "Linux", "服务器", "IC服务器运维"]
categories: ["IC服务器运维"]
summary: "在 IC 服务器主从节点之间配置 NFS，将主机 /home 共享给从机，并给出 fstab 与 autofs 两种挂载方式。"
ShowToc: true
TocOpen: true
---

本文记录在 IC 服务器集群中配置 NFS 的完整流程。目标是将主机 `sh05` 的 `/home` 目录共享给从机，使从机可以使用同一套用户家目录。

示例环境：

- 主机：`sh05`，IP 为 `192.168.113.50`
- 从机：`sh06`，IP 为 `192.168.113.51`
- 共享目录：主机 `/home`
- 客户端挂载点：从机 `/home`

## 前置条件

配置前建议先完成以下检查：

- 主机和从机已完成 Red Hat 7.9 基础初始化。
- 主机和从机可以互相 `ping` 通。
- 本地 yum 源可用。
- 已对虚拟机做快照或备份，便于配置失败时回滚。

## 1. 主机安装 NFS 组件

在主机上安装 NFS 和 RPC 相关软件包：

```bash
yum install -y nfs-utils rpcbind
```

启动并设置 RPC 服务开机自启：

```bash
systemctl enable --now rpcbind.service
```

## 2. 主机配置共享目录

编辑 `/etc/exports`：

```bash
gedit /etc/exports
```

以共享 `sh05:/home` 给 `192.168.113.0/24` 网段为例，添加：

```exports
/home 192.168.113.0/24(rw,no_root_squash,sync)
```

保存后启动 NFS 服务并重新导出共享目录：

```bash
systemctl enable --now nfs
exportfs -arv
```

`exportfs -arv` 很重要，用于让 `/etc/exports` 中的修改立即生效。

## 3. 从机确认共享目录

在从机上查看主机导出的共享目录：

```bash
showmount -e 192.168.113.50
```

如果能看到 `/home`，说明主机端共享已可被从机发现。

## 4. 从机挂载方式一：fstab 手动挂载

这种方式简单直接，但每次重启后需要确认挂载状态。

编辑从机 `/etc/fstab`：

```bash
gedit /etc/fstab
```

添加：

```fstab
192.168.113.50:/home /home nfs defaults 0 0
```

保存后立即挂载：

```bash
mount -a
df -h
```

确认输出中出现 `192.168.113.50:/home` 挂载到 `/home`。

## 5. 从机挂载方式二：autofs 自动挂载

推荐使用 autofs。它会在访问目录时自动挂载，重启后不需要手动执行 `mount -a`。

安装 autofs：

```bash
yum install -y autofs
```

配置 autofs 主映射：

```bash
echo "/- /etc/auto.nfs" >> /etc/auto.master
```

配置 NFS 目录映射：

```bash
echo "/home -fstype=nfs,soft,timeo=30,retry=5 192.168.113.50:/home" > /etc/auto.nfs
```

> 建议这里直接使用主机 IP，避免 hostname 解析异常导致挂载失败。

重启 autofs：

```bash
systemctl restart autofs.service
```

访问并检查挂载：

```bash
df -h
```

如果 `df -h` 中出现主机 `/home` 的挂载记录，则从机配置完成。

## 6. 验证

可以在主机 `/home` 中创建一个测试目录或测试文件，然后在从机 `/home` 中确认是否可见：

```bash
mkdir /home/nfs_test
ls /home
```

验证完成后删除测试目录：

```bash
rmdir /home/nfs_test
```

## 常见问题

- `showmount -e` 看不到共享目录：检查主机 `nfs`、`rpcbind` 是否启动，并重新执行 `exportfs -arv`。
- 从机挂载超时：检查主从网络、IP 网段、防火墙和 SELinux。
- 使用 hostname 挂载失败：优先改用 IP，或先修复 `/etc/hosts`、DNS、NIS 等名称解析配置。

完成 NFS 后，从机即可通过主机共享的 `/home` 使用统一家目录，后续可继续配置 NIS 用户集中管理。
