---
title: "IC 服务器 NIS 用户集中管理配置"
date: 2026-06-14T14:20:00+08:00
draft: false
author: "SiliconBlog"
tags: ["NIS", "Linux", "服务器", "IC服务器运维"]
categories: ["IC服务器运维"]
summary: "在完成 NFS 共享 Home 后配置 NIS，将用户账号集中维护在主机，并让从机可直接登录主机创建的用户。"
ShowToc: true
TocOpen: true
---

本文记录 IC 服务器集群中 NIS 的配置流程。目标是在主机集中维护用户账号，从机通过 NIS 识别并登录这些账号。

示例环境：

- NIS 主机：`sh05`，IP 为 `192.168.113.50`
- NIS 从机：`sh06`，IP 为 `192.168.113.51`
- NIS 域名：`workarea`
- 固定端口：`ypserv` 使用 `1011`，`yppasswdd` 使用 `1012`

## 前置条件

一般建议先完成 NFS 配置，再配置 NIS。配置前确认：

- 主机和从机已完成基础系统初始化。
- 从机 `/home` 已挂载主机共享的 `/home`。
- 可通过以下命令在从机确认挂载状态：

```bash
df -h
```

如果从机 `/home` 没有指向主机共享目录，即使 NIS 用户识别成功，登录时也可能因为找不到家目录而异常。

## 1. 主机安装 NIS 组件

在主机安装所需软件包：

```bash
yum -y install ypserv ypbind yp-tools
```

## 2. 主机设置 NIS 域名和 ypserv 端口

编辑 `/etc/sysconfig/network`：

```bash
gedit /etc/sysconfig/network
```

添加：

```ini
NISDOMAIN=workarea
YPSERV_ARGS="-p 1011"
```

这里将 NIS 域名设置为 `workarea`。保存后继续配置密码修改服务端口。

## 3. 主机设置 yppasswdd 端口

编辑 `/etc/sysconfig/yppasswdd`：

```bash
gedit /etc/sysconfig/yppasswdd
```

添加：

```ini
YPPASSWDD_ARGS="--port 1012"
```

保存并退出。

## 4. 主机配置访问控制

编辑 `/etc/ypserv.conf`：

```bash
gedit /etc/ypserv.conf
```

添加：

```text
127.0.0.1/255.255.255.0       : *       : *       : none
192.168.113.0/255.255.255.0   : *       : *       : none
*                              : *       : *       : deny
```

该配置允许本机和 `192.168.113.0/24` 网段访问 NIS 服务，其他来源拒绝。

## 5. 主机配置 hosts

编辑 `/etc/hosts`：

```bash
gedit /etc/hosts
```

添加主从节点映射：

```text
192.168.113.50 sh05
192.168.113.51 sh06
```

保存后继续启动服务。

## 6. 主机启动 NIS 服务

启动并设置开机自启：

```bash
systemctl enable --now ypserv.service
systemctl enable --now yppasswdd.service
```

初始化 NIS 数据库：

```bash
/usr/lib64/yp/ypinit -m
```

执行后会提示添加除主机外的服务器。如果暂时不确定，可以直接按 `Ctrl+D` 保存当前配置，再按 `y` 确认退出。

## 7. 从机指定 NIS 域名

在从机编辑 `/etc/rc.local`：

```bash
gedit /etc/rc.local
```

添加：

```bash
/bin/nisdomainname workarea
```

保存并退出。

## 8. 从机启用 NIS 认证

在从机运行：

```bash
authconfig-tui
```

在图形界面中：

1. 第一个界面选择 `Use NIS`，然后点击 `Next`。
2. 第二个界面中，NIS 域名填写 `workarea`。
3. NIS Server 填写主机 hostname 或 IP，推荐填写 `192.168.113.50`。

完成后，从机端 NIS 认证配置结束。

## 9. 新增用户后的数据库更新

每当主机新增或修改用户后，都需要更新 NIS 数据库：

```bash
cd /var/yp
make
```

看到更新完成提示后，从机才能获取最新账号信息。

## 10. 验证

在主机上创建测试账号：

```bash
useradd user02
echo 123456 | passwd --stdin user02
cd /var/yp
make
```

登录从机后切换到该用户：

```bash
su user02
```

如果可以正常切换账号，说明 NIS 配置成功。也可以重启从机后，在服务器登录界面直接使用主机创建的账号登录：

```bash
reboot
```

如果登录界面输入 NIS 用户后闪退，大概率是没有正确找到 NIS 域，优先检查从机 NIS 域名、NIS Server、主机服务状态和 `/home` 挂载。
