---
title: "IC 服务器 RedHat 7.9 系统初始化配置"
date: 2025-06-14T10:00:00+08:00
draft: false
author: "SiliconBlog"
tags: ["RedHat", "Linux", "服务器", "IC服务器运维"]
categories: ["IC服务器运维"]
summary: "RedHat 7.9 系统安装完成后的初始化配置：hostname、网络、防火墙、SELinux、共享文件夹与本地镜像源。"
ShowToc: true
TocOpen: true
---

本文介绍 IC 服务器在完成 RedHat 7.9 系统安装后的初始化配置步骤。

## 1. 设置 Hostname

```bash
hostnamectl set-hostname sh05
```

将 `sh05` 替换为需要设置的 hostname。hostname 会用于后续的配置服务，请谨慎设置。

设置完成后，可通过以下命令查看是否生效：

```bash
hostname
```

## 2. 设置网络

本文所配置的虚拟机系统的网络环境为仅主机模式，其他网络模式配置类似。

```bash
su
gedit /etc/sysconfig/network-scripts/ifcfg-ens33
```

添加/设置如下内容：

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

`IPADDR` 即为该服务器 IP，设置完成后，重启网络服务：

```bash
systemctl restart network.service
```

此时可通过 `ifconfig` 命令查看网络设置是否生效。

> **注意**：虚拟机 IP 的网段需要在主机网卡和虚拟机网络编辑器中进行配置，不同机器的网络环境可能不同，此处仅作参考。

## 3. 关闭防火墙和 SELinux

查看防火墙的运行状态：

```bash
systemctl status firewalld.service
```

关闭防火墙：

```bash
systemctl disable firewalld.service
systemctl stop firewalld.service
```

确认防火墙已关闭：

```bash
systemctl status firewalld.service
```

关闭 SELinux：

```bash
gedit /etc/selinux/config
```

在该文件中修改设置项 `SELINUX=disabled`，然后重启系统：

```bash
reboot
```

重启后验证：

```bash
sestatus
```

当显示 `SELinux status` 的状态为 `disabled` 时，表示 SELinux 已被关闭。

## 4. 设置共享文件夹并挂载 ISO 为本地镜像源

### 4.1 配置共享文件夹

将虚拟机关机后，点击"编辑虚拟机设置"，依次选择 **选项 → 共享文件夹 → 总是启用**，并在下方根据提示添加主机中用于共享的文件夹目录（可设置别名），然后重启系统。

系统启动后，编辑 `/etc/fstab`，保证系统每次重启后都会自动挂载来自主机的共享文件：

```bash
su
cd /mnt
mkdir hgfs
gedit /etc/fstab
```

在文件末尾添加如下内容，保存并退出：

```
.host:/   /mnt/hgfs   fuse.vmhgfs-fuse   defaults,allow_other,nonempty   0 0
```

立刻使得挂载生效：

```bash
mount -a
```

### 4.2 挂载 ISO 为本地镜像源

假设系统镜像存放在 `/mnt/hgfs/msa_v1/` 下，现将该镜像挂载为本地镜像源。

新建 yum 文件 `local.repo`：

```bash
gedit /etc/yum.repos.d/local.repo
```

在文件中添加如下内容：

```ini
[local]
name=local.repo
baseurl=file:///mnt/iso
enabled=1
gpgcheck=0
```

创建挂载点并挂载镜像：

```bash
mkdir -p /mnt/iso
mount -t iso9660 -o loop /mnt/hgfs/msa_v1/rhel-server-7.9-x86_64-dvd.iso /mnt/iso
```

### 4.3 验证配置

查看当前系统的挂载情况：

```bash
df -h
```

查看当前可用的（本地）镜像源：

```bash
yum repolist
```

当输出前文中配置好的 yum 源时即代表配置成功。此外，也可尝试性地搜索或者安装一些本地源的包以验证配置的正确性。

---

*至此，服务器的基本配置已完成，后续为服务配置过程。*
