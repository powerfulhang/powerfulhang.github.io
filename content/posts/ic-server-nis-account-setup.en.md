---
title: "IC Server NIS Centralized Account Management Setup"
date: 2026-06-14T14:20:00+08:00
draft: false
author: "SiliconBlog"
tags: ["NIS", "Linux", "Server", "IC Server Operations"]
categories: ["IC Server Operations"]
summary: "After setting up NFS shared home, configure NIS to centrally manage user accounts on the master, allowing slaves to log in with master-created users."
ShowToc: true
TocOpen: true
---

This article documents the NIS configuration process for an IC server cluster. The goal is to centrally manage user accounts on the master, with slaves recognizing and allowing login for those accounts via NIS.

Example environment:

- NIS master: `sh05`, IP `192.168.113.50`
- NIS slave: `sh06`, IP `192.168.113.51`
- NIS domain: `workarea`
- Fixed ports: `ypserv` uses `1011`, `yppasswdd` uses `1012`

## Prerequisites

It is generally recommended to complete NFS configuration before NIS. Before starting, confirm:

- Both master and slave have completed base system initialization.
- The slave's `/home` is mounted from the master's shared `/home`.
- You can verify the mount status on the slave with:

```bash
df -h
```

If the slave's `/home` does not point to the master's shared directory, even if NIS user recognition succeeds, login may fail because the home directory cannot be found.

## 1. Install NIS Components on Master

Install the required packages on the master:

```bash
yum -y install ypserv ypbind yp-tools
```

## 2. Set NIS Domain Name and ypserv Port on Master

Edit `/etc/sysconfig/network`:

```bash
gedit /etc/sysconfig/network
```

Add:

```ini
NISDOMAIN=workarea
YPSERV_ARGS="-p 1011"
```

This sets the NIS domain name to `workarea`. Save and proceed to configure the password service port.

## 3. Set yppasswdd Port on Master

Edit `/etc/sysconfig/yppasswdd`:

```bash
gedit /etc/sysconfig/yppasswdd
```

Add:

```ini
YPPASSWDD_ARGS="--port 1012"
```

Save and exit.

## 4. Configure Access Control on Master

Edit `/etc/ypserv.conf`:

```bash
gedit /etc/ypserv.conf
```

Add:

```text
127.0.0.1/255.255.255.0       : *       : *       : none
192.168.113.0/255.255.255.0   : *       : *       : none
*                              : *       : *       : deny
```

This allows access from localhost and the `192.168.113.0/24` subnet, and denies all other sources.

## 5. Configure hosts on Master

Edit `/etc/hosts`:

```bash
gedit /etc/hosts
```

Add the master-slave node mappings:

```text
192.168.113.50 sh05
192.168.113.51 sh06
```

Save and proceed to start services.

## 6. Start NIS Services on Master

Start and enable the services:

```bash
systemctl enable --now ypserv.service
systemctl enable --now yppasswdd.service
```

Initialize the NIS database:

```bash
/usr/lib64/yp/ypinit -m
```

When prompted to add servers beyond the master, you can press `Ctrl+D` to save the current configuration, then press `y` to confirm.

## 7. Set NIS Domain Name on Slave

On the slave, edit `/etc/rc.local`:

```bash
gedit /etc/rc.local
```

Add:

```bash
/bin/nisdomainname workarea
```

Save and exit.

## 8. Enable NIS Authentication on Slave

On the slave, run:

```bash
authconfig-tui
```

In the text-based UI:

1. On the first screen, select `Use NIS`, then click `Next`.
2. On the second screen, enter `workarea` as the NIS domain.
3. For the NIS Server, enter the master's hostname or IP — using `192.168.113.50` is recommended.

After completion, the slave-side NIS authentication configuration is done.

## 9. Update Database After Adding Users

Whenever users are added or modified on the master, the NIS database must be updated:

```bash
cd /var/yp
make
```

Only after the update completes will slaves receive the latest account information.

## 10. Verification

Create a test account on the master:

```bash
useradd user02
echo 123456 | passwd --stdin user02
cd /var/yp
make
```

Log into the slave and switch to the test user:

```bash
su user02
```

If you can switch to the account successfully, NIS is configured correctly. You can also reboot the slave and try logging in directly from the login screen using a master-created account:

```bash
reboot
```

If the login screen flashes back after entering a NIS user's credentials, the most likely cause is that the NIS domain was not found. Check the slave's NIS domain name, NIS Server setting, master service status, and `/home` mount first.
