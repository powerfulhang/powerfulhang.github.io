---
title: "IC 服务器 Red Hat 7.9 基础环境初始化"
date: 2026-06-14T14:00:00+08:00
draft: false
author: "SiliconBlog"
tags: ["RedHat", "Linux", "服务器", "IC服务器运维"]
categories: ["IC服务器运维"]
summary: "从纯净 Red Hat 7.9 系统开始，完成 hostname、静态网络、防火墙、SELinux、VMware 共享目录和本地 ISO yum 源配置。"
ShowToc: true
TocOpen: true
---

本文记录 IC 服务器在安装纯净 Red Hat 7.9 后的基础初始化流程。完成这些步骤后，服务器即可继续配置 NFS、NIS、LSF 以及后续 IC 软件环境。

文中的示例环境如下，可按实际服务器规划替换：

- 主机名：`sh05`
- 服务器 IP：`192.168.113.50`
- 网卡名：`ens33`
- Red Hat ISO 路径：`/mnt/hgfs/msa_v1/rhel-server-7.9-x86_64-dvd.iso`

## 1. 设置 Hostname

先设置服务器 hostname：

```bash
hostnamectl set-hostname sh05
```

`sh05` 是后续 NFS、NIS、LSF 配置中会继续使用的节点名，建议在开始配置服务前先确定命名规则。

查看是否生效：

```bash
hostname
```

## 2. 配置静态网络

本文示例虚拟机使用 VMware 仅主机模式，其他网络模式可参考同样思路配置。

进入 root 用户并编辑网卡配置：

```bash
su
gedit /etc/sysconfig/network-scripts/ifcfg-ens33
```

写入或修改为以下内容：

```ini
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.113.50
```

其中 `IPADDR` 是当前服务器 IP。修改后重启网络服务：

```bash
systemctl restart network.service
```

确认网卡地址是否生效：

```bash
ifconfig
```

> 注意：虚拟机 IP 网段需要与主机网卡、VMware 虚拟网络编辑器中的网段匹配。这里的 `192.168.113.0/24` 只作为示例。

## 3. 关闭防火墙和 SELinux

先查看防火墙运行状态：

```bash
systemctl status firewalld.service
```

关闭并禁用防火墙：

```bash
systemctl disable firewalld.service
systemctl stop firewalld.service
```

再次确认状态：

```bash
systemctl status firewalld.service
```

编辑 SELinux 配置：

```bash
gedit /etc/selinux/config
```

将配置项修改为：

```ini
SELINUX=disabled
```

保存后重启系统：

```bash
reboot
```

重启后检查 SELinux 状态：

```bash
sestatus
```

当 `SELinux status` 显示为 `disabled` 时，说明 SELinux 已关闭。

## 4. 配置 VMware 共享目录

在虚拟机关机状态下，进入 VMware 的“编辑虚拟机设置”，依次选择“选项 -> 共享文件夹 -> 总是启用”，并按提示添加主机中的共享文件夹目录。设置完成后启动系统。

启动后创建挂载点：

```bash
su
cd /mnt
mkdir hgfs
```

编辑自动挂载配置：

```bash
gedit /etc/fstab
```

在文件末尾添加：

```fstab
.host:/   /mnt/hgfs   fuse.vmhgfs-fuse   defaults,allow_other,nonempty   0 0
```

立即挂载并验证：

```bash
mount -a
df -h
```

## 5. 挂载 ISO 并配置本地 yum 源

假设系统镜像存放在共享目录 `/mnt/hgfs/msa_v1/` 下，先创建 repo 文件：

```bash
gedit /etc/yum.repos.d/local.repo
```

写入以下内容：

```ini
[local]
name=local.repo
baseurl=file:///mnt/iso
enabled=1
gpgcheck=0
```

创建 ISO 挂载点并挂载镜像：

```bash
mkdir -p /mnt/iso
mount -t iso9660 -o loop /mnt/hgfs/msa_v1/rhel-server-7.9-x86_64-dvd.iso /mnt/iso
```

查看挂载情况：

```bash
df -h
```

查看当前可用 yum 源：

```bash
yum repolist
```

当输出中出现前面配置的 `local` 源时，本地镜像源配置完成。也可以尝试搜索或安装一个软件包来验证 yum 源是否可用。

## 6. 后续检查清单

继续配置 NFS、NIS、LSF 前，建议确认：

- hostname 已按规划设置。
- 主机和从机 IP 均处于预期网段。
- 防火墙和 SELinux 已关闭，或已按实际安全策略放通相关服务。
- VMware 共享目录能正常访问安装包和 ISO。
- `yum repolist` 能看到本地源。

至此，服务器基础配置完成。
