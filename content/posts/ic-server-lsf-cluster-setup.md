---
title: "IC 服务器 IBM LSF 作业调度集群配置"
date: 2026-06-14T14:30:00+08:00
draft: false
author: "SiliconBlog"
tags: ["LSF", "Linux", "服务器", "IC服务器运维"]
categories: ["IC服务器运维"]
summary: "从安装包解压、主机安装、LSF 环境加载，到从机加入集群和启动 LIM/RES/SBD 服务的完整配置流程。"
ShowToc: true
TocOpen: true
---

本文记录在 IC 服务器集群中配置 IBM LSF 的流程。与 NFS、NIS 类似，LSF 也需要区分主机和从机；不同的是，从机配置时仍需要回到主机修改集群配置，并通过 NFS 共享 `/softwares` 目录让从机使用同一套 LSF 安装环境。

示例环境：

- LSF 主机：`sh05`
- LSF 从机：`sh06`
- LSF 管理用户：`lsfadmin`
- LSF 集群名：`workarea_cluster`
- 安装根目录：`/softwares/IBM_lsf`
- LSF 安装目标：`/softwares/IBM_lsf/app/lsf`

## 前置条件

本文以以下安装包为例：

- `lsf10.1_linux2.6-glibc2.3-x86_64.tar.Z`
- `lsf10.1_x86_lnx310.tar`

LSF 不强制依赖前面的 NFS、NIS 配置，但实际 IC 环境通常建议按“基础系统 -> NFS -> NIS -> LSF”的顺序完成，便于统一用户、共享软件目录和排查节点问题。

## 1. 主机准备安装目录

在主机新建目录：

```bash
mkdir -p /softwares/IBM_lsf/{app,install}
```

假设 `lsf10.1_x86_lnx310.tar` 当前位于 `/mnt/hgfs/msa_v1`，移动到安装目录：

```bash
mv /mnt/hgfs/msa_v1/lsf10.1_x86_lnx310.tar /softwares/IBM_lsf/install/
cd /softwares/IBM_lsf/install
tar xvf lsf10.1_x86_lnx310.tar
```

解压后会得到同名目录，其中包含：

```text
lsf10.1_lnx310-lib217-x86_64.tar.Z
lsf10.1_lsfinstall_linux_x86_64.tar
lsf_std_entitlement.dat
```

继续解压安装程序：

```bash
cd /softwares/IBM_lsf/install/lsf10.1_x86_lnx310
tar xvf lsf10.1_lsfinstall_linux_x86_64.tar
```

解压后会得到 `lsf10.1_lsfinstall` 目录。

## 2. 主机配置安装参数

安装 `ed`，如果已安装可跳过：

```bash
yum install -y ed
```

添加 LSF 管理用户：

```bash
useradd lsfadmin
```

如确实需要指定 UID，可使用：

```bash
useradd lsfadmin -u 1500
```

编辑安装配置文件：

```bash
gedit /softwares/IBM_lsf/install/lsf10.1_x86_lnx310/lsf10.1_lsfinstall/install.config
```

指定以下内容：

```ini
LSF_TOP="/softwares/IBM_lsf/app/lsf"
LSF_ADMINS="lsfadmin"
LSF_CLUSTER_NAME="workarea_cluster"
LSF_MASTER_LIST="sh05"
LSF_ENTITLEMENT_FILE="/softwares/IBM_lsf/install/lsf10.1_x86_lnx310/lsf_std_entitlement.dat"
LSF_TARDIR="/softwares/IBM_lsf/install/lsf10.1_x86_lnx310"
```

## 3. 主机执行安装

进入安装程序目录：

```bash
cd /softwares/IBM_lsf/install/lsf10.1_x86_lnx310/lsf10.1_lsfinstall
```

执行安装：

```bash
./lsfinstall -f install.config
```

安装过程中会有两次确认提示，按原流程输入 `1` 并等待安装完成。

安装完成后加载 LSF 环境：

```bash
cd /softwares/IBM_lsf/app/lsf/
source profile.lsf
```

如果环境中未配置 RSH，需要在 `lsf.conf` 末尾添加 `LSF_RSH=ssh`：

```bash
gedit lsf.conf
```

添加：

```ini
LSF_RSH=ssh
```

设置 LSF 服务开机自启并启动：

```bash
./hostsetup --top="/softwares/IBM_lsf/app/lsf/" --boot="y"
lsfstartup
```

## 4. 主机验证基础服务

常用检查命令：

```bash
systemctl status lsfd.service
bhosts
lsf_daemons status
```

其中：

- `systemctl status lsfd.service` 用于查看服务是否存在并运行。
- `bhosts` 用于查看集群节点状态。
- `lsf_daemons status` 用于查看当前节点 LSF 守护进程状态。

## 5. 主机加入从机节点

配置从机时，仍需要先在主机修改集群配置文件：

```bash
cd /softwares/IBM_lsf/app/lsf/conf
gedit lsf.cluster.workarea_cluster
```

仿照文件中 `sh05` 的配置行，在文件末尾添加从机：

```text
sh06      !         !          1           ()
```

如果允许，也可以直接复制 `sh05` 的配置行并修改节点名，保证最后括号内容为空。

重新加载并重启主机批处理守护进程：

```bash
lsadmin reconfig
badmin mbdrestart
```

## 6. 主机共享 LSF 安装目录

由于从机未单独安装 LSF，需要主机将 `/softwares` 共享给从机。

编辑主机 `/etc/exports`：

```bash
gedit /etc/exports
```

添加：

```exports
/softwares *(rw,sync,no_root_squash)
```

保存后重新导出：

```bash
exportfs -arv
```

至此，主机侧的从机准备工作完成。

## 7. 从机挂载 /softwares

在从机编辑 autofs 配置：

```bash
gedit /etc/auto.nfs
```

添加来自主机的 `/softwares` 共享目录：

```text
/softwares -fstype=nfs,rw,sync,soft 192.168.113.50:/softwares
```

保存并重启 autofs：

```bash
systemctl restart autofs.service
df -h
```

确认 `/softwares` 已从主机挂载成功。

## 8. 从机启动 LSF 服务

从机使用主机共享的 LSF 环境后，加载环境并启动服务：

```bash
cd /softwares/IBM_lsf/app/lsf
source profile.lsf
lsadmin limstartup
lsadmin resstartup
badmin hstartup
```

## 9. 集群验证

回到主机或任意已加载 LSF 环境的节点，检查集群状态：

```bash
bhosts
lsf_daemons status
```

如果 `sh06` 出现在 `bhosts` 输出中，并且状态正常，说明从机已加入 LSF 集群。

## 排查建议

- `bhosts` 看不到从机：检查 `lsf.cluster.workarea_cluster` 是否添加 `sh06`，并确认执行过 `lsadmin reconfig` 与 `badmin mbdrestart`。
- 从机无法执行 LSF 命令：确认 `/softwares` 是否挂载成功，并重新 `source profile.lsf`。
- 守护进程启动失败：优先检查 hostname、主从互通、NFS 挂载、`LSF_RSH=ssh`、以及主从用户环境是否一致。
