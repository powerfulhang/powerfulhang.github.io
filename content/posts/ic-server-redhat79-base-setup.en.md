---
title: "IC Server Red Hat 7.9 Base System Setup"
date: 2026-06-14T14:00:00+08:00
draft: false
author: "SiliconBlog"
tags: ["RedHat", "Linux", "Server", "IC Server Operations"]
categories: ["IC Server Operations"]
summary: "Starting from a clean Red Hat 7.9 installation, complete hostname, static network, firewall, SELinux, VMware shared folders, and local ISO yum repository configuration."
ShowToc: true
TocOpen: true
---

This article documents the base initialization process for an IC server after a clean Red Hat 7.9 installation. Once these steps are complete, the server is ready for further NFS, NIS, LSF, and IC software environment configuration.

The example environment used in this article is as follows — replace with your actual server specifications:

- Hostname: `sh05`
- Server IP: `192.168.113.50`
- Network interface: `ens33`
- Red Hat ISO path: `/mnt/hgfs/msa_v1/rhel-server-7.9-x86_64-dvd.iso`

## 1. Set the Hostname

Set the server hostname first:

```bash
hostnamectl set-hostname sh05
```

`sh05` is the node name that will be used throughout subsequent NFS, NIS, and LSF configurations. It is recommended to establish a naming convention before starting any service configuration.

Verify the change:

```bash
hostname
```

## 2. Configure Static Network

This example uses VMware host-only mode. Other network modes can be configured using a similar approach.

Switch to root and edit the network interface configuration:

```bash
su
gedit /etc/sysconfig/network-scripts/ifcfg-ens33
```

Write or modify the file with the following content:

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

`IPADDR` is the current server IP. Restart the network service after modification:

```bash
systemctl restart network.service
```

Confirm the interface address:

```bash
ifconfig
```

> Note: The VM IP subnet must match the host adapter and VMware Virtual Network Editor settings. The `192.168.113.0/24` subnet here is only an example.

## 3. Disable Firewall and SELinux

Check the firewall status:

```bash
systemctl status firewalld.service
```

Disable and stop the firewall:

```bash
systemctl disable firewalld.service
systemctl stop firewalld.service
```

Confirm the status again:

```bash
systemctl status firewalld.service
```

Edit the SELinux configuration:

```bash
gedit /etc/selinux/config
```

Change the setting to:

```ini
SELINUX=disabled
```

Save and reboot:

```bash
reboot
```

After reboot, check SELinux status:

```bash
sestatus
```

When `SELinux status` shows `disabled`, SELinux has been successfully turned off.

## 4. Configure VMware Shared Folders

With the VM powered off, go to VMware's "Edit Virtual Machine Settings", then select "Options -> Shared Folders -> Always Enabled", and add the host directories you want to share. Start the system after configuration.

After boot, create the mount point:

```bash
su
cd /mnt
mkdir hgfs
```

Edit the auto-mount configuration:

```bash
gedit /etc/fstab
```

Append the following line:

```fstab
.host:/   /mnt/hgfs   fuse.vmhgfs-fuse   defaults,allow_other,nonempty   0 0
```

Mount immediately and verify:

```bash
mount -a
df -h
```

## 5. Mount ISO and Configure Local yum Repository

Assuming the system ISO is stored under the shared directory `/mnt/hgfs/msa_v1/`, create the repo file:

```bash
gedit /etc/yum.repos.d/local.repo
```

Write the following content:

```ini
[local]
name=local.repo
baseurl=file:///mnt/iso
enabled=1
gpgcheck=0
```

Create the ISO mount point and mount the image:

```bash
mkdir -p /mnt/iso
mount -t iso9660 -o loop /mnt/hgfs/msa_v1/rhel-server-7.9-x86_64-dvd.iso /mnt/iso
```

Check the mount status:

```bash
df -h
```

List available yum repositories:

```bash
yum repolist
```

When the `local` repository appears in the output, the local ISO repository configuration is complete. You can also try searching for or installing a package to verify that the yum source is working.

## 6. Pre-flight Checklist

Before proceeding to NFS, NIS, and LSF configuration, confirm the following:

- Hostname is set according to your plan.
- All master and slave IPs are in the expected subnet.
- Firewall and SELinux are disabled, or relevant services are allowed per your security policy.
- VMware shared folders can access installation packages and ISOs.
- `yum repolist` shows the local repository.

At this point, the base server configuration is complete.
