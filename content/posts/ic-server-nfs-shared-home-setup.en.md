---
title: "IC Server NFS Shared Home Directory Setup"
date: 2026-06-14T14:10:00+08:00
draft: false
author: "SiliconBlog"
tags: ["NFS", "Linux", "Server", "IC Server Operations"]
categories: ["IC Server Operations"]
summary: "Configure NFS between IC server master and slave nodes, sharing the master's /home directory to slaves, with both fstab and autofs mount methods."
ShowToc: true
TocOpen: true
---

This article documents the complete NFS configuration process for an IC server cluster. The goal is to share the master host `sh05`'s `/home` directory to slave nodes, allowing them to use the same set of user home directories.

Example environment:

- Master: `sh05`, IP `192.168.113.50`
- Slave: `sh06`, IP `192.168.113.51`
- Shared directory: master `/home`
- Client mount point: slave `/home`

## Prerequisites

Before configuration, complete the following checks:

- Both master and slave have completed Red Hat 7.9 base initialization.
- Master and slave can `ping` each other.
- Local yum repository is available.
- VM snapshots or backups have been taken for easy rollback if configuration fails.

## 1. Install NFS Components on Master

Install NFS and RPC packages on the master:

```bash
yum install -y nfs-utils rpcbind
```

Start and enable the RPC service:

```bash
systemctl enable --now rpcbind.service
```

## 2. Configure Shared Directory on Master

Edit `/etc/exports`:

```bash
gedit /etc/exports
```

To share `sh05:/home` with the `192.168.113.0/24` subnet, add:

```exports
/home 192.168.113.0/24(rw,no_root_squash,sync)
```

Save, then start the NFS service and re-export the shared directories:

```bash
systemctl enable --now nfs
exportfs -arv
```

`exportfs -arv` is important — it makes changes in `/etc/exports` take effect immediately.

## 3. Verify Shared Directory from Slave

On the slave, check the master's exported shared directories:

```bash
showmount -e 192.168.113.50
```

If you can see `/home`, the master-side share is discoverable by the slave.

## 4. Slave Mount Method 1: fstab Manual Mount

This method is straightforward but requires confirming mount status after each reboot.

Edit the slave's `/etc/fstab`:

```bash
gedit /etc/fstab
```

Add:

```fstab
192.168.113.50:/home /home nfs defaults 0 0
```

Save and mount immediately:

```bash
mount -a
df -h
```

Confirm that `192.168.113.50:/home` appears mounted at `/home` in the output.

## 5. Slave Mount Method 2: autofs Automatic Mount

Using autofs is recommended. It automatically mounts the directory on access and does not require manual `mount -a` after reboot.

Install autofs:

```bash
yum install -y autofs
```

Configure the autofs master map:

```bash
echo "/- /etc/auto.nfs" >> /etc/auto.master
```

Configure the NFS directory map:

```bash
echo "/home -fstype=nfs,soft,timeo=30,retry=5 192.168.113.50:/home" > /etc/auto.nfs
```

> It is recommended to use the master's IP directly here to avoid mount failures caused by hostname resolution issues.

Restart autofs:

```bash
systemctl restart autofs.service
```

Check the mount:

```bash
df -h
```

If the master's `/home` mount record appears in `df -h`, the slave configuration is complete.

## 6. Verification

Create a test directory in the master's `/home`, then check if it is visible on the slave:

```bash
mkdir /home/nfs_test
ls /home
```

Remove the test directory after verification:

```bash
rmdir /home/nfs_test
```

## Common Issues

- `showmount -e` does not show shared directories: Check whether `nfs` and `rpcbind` are running on the master, and re-run `exportfs -arv`.
- Slave mount times out: Check master-slave network connectivity, IP subnet, firewall, and SELinux.
- Mount fails using hostname: Use IP instead, or fix `/etc/hosts`, DNS, or NIS name resolution configuration first.

After completing NFS, slaves can use the unified home directory shared by the master. You can then proceed to configure NIS for centralized user management.
