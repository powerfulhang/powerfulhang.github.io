---
title: "IC Server IBM LSF Job Scheduling Cluster Setup"
date: 2026-06-14T14:30:00+08:00
draft: false
author: "SiliconBlog"
tags: ["LSF", "Linux", "Server", "IC Server Operations"]
categories: ["IC Server Operations"]
summary: "Complete configuration process from package extraction, master installation, LSF environment loading, to slave node joining and LIM/RES/SBD service startup."
ShowToc: true
TocOpen: true
---

This article documents the IBM LSF configuration process for an IC server cluster. Similar to NFS and NIS, LSF also requires a distinction between master and slave nodes. However, slave configuration still requires modifying cluster configuration files on the master, and the `/softwares` directory is shared via NFS so that slaves can use the same LSF installation.

Example environment:

- LSF master: `sh05`
- LSF slave: `sh06`
- LSF admin user: `lsfadmin`
- LSF cluster name: `workarea_cluster`
- Installation root: `/softwares/IBM_lsf`
- LSF installation target: `/softwares/IBM_lsf/app/lsf`

## Prerequisites

This article uses the following installation packages as examples:

- `lsf10.1_linux2.6-glibc2.3-x86_64.tar.Z`
- `lsf10.1_x86_lnx310.tar`

LSF does not strictly depend on NFS or NIS, but in practice, IC environments typically follow the order "base system -> NFS -> NIS -> LSF" for unified user management, shared software directories, and easier troubleshooting.

## 1. Prepare Installation Directory on Master

Create directories on the master:

```bash
mkdir -p /softwares/IBM_lsf/{app,install}
```

Assuming `lsf10.1_x86_lnx310.tar` is located at `/mnt/hgfs/msa_v1`, move it to the installation directory:

```bash
mv /mnt/hgfs/msa_v1/lsf10.1_x86_lnx310.tar /softwares/IBM_lsf/install/
cd /softwares/IBM_lsf/install
tar xvf lsf10.1_x86_lnx310.tar
```

After extraction, you will find a directory of the same name containing:

```text
lsf10.1_lnx310-lib217-x86_64.tar.Z
lsf10.1_lsfinstall_linux_x86_64.tar
lsf_std_entitlement.dat
```

Continue extracting the installer:

```bash
cd /softwares/IBM_lsf/install/lsf10.1_x86_lnx310
tar xvf lsf10.1_lsfinstall_linux_x86_64.tar
```

This produces the `lsf10.1_lsfinstall` directory.

## 2. Configure Installation Parameters on Master

Install `ed` if not already installed:

```bash
yum install -y ed
```

Add the LSF admin user:

```bash
useradd lsfadmin
```

If you need to specify a UID:

```bash
useradd lsfadmin -u 1500
```

Edit the installation configuration file:

```bash
gedit /softwares/IBM_lsf/install/lsf10.1_x86_lnx310/lsf10.1_lsfinstall/install.config
```

Set the following parameters:

```ini
LSF_TOP="/softwares/IBM_lsf/app/lsf"
LSF_ADMINS="lsfadmin"
LSF_CLUSTER_NAME="workarea_cluster"
LSF_MASTER_LIST="sh05"
LSF_ENTITLEMENT_FILE="/softwares/IBM_lsf/install/lsf10.1_x86_lnx310/lsf_std_entitlement.dat"
LSF_TARDIR="/softwares/IBM_lsf/install/lsf10.1_x86_lnx310"
```

## 3. Execute Installation on Master

Navigate to the installer directory:

```bash
cd /softwares/IBM_lsf/install/lsf10.1_x86_lnx310/lsf10.1_lsfinstall
```

Run the installation:

```bash
./lsfinstall -f install.config
```

During installation, there will be two confirmation prompts — enter `1` and wait for the installation to complete.

After installation, load the LSF environment:

```bash
cd /softwares/IBM_lsf/app/lsf/
source profile.lsf
```

If RSH is not configured in your environment, add `LSF_RSH=ssh` to the end of `lsf.conf`:

```bash
gedit lsf.conf
```

Add:

```ini
LSF_RSH=ssh
```

Set LSF service to start on boot and start it:

```bash
./hostsetup --top="/softwares/IBM_lsf/app/lsf/" --boot="y"
lsfstartup
```

## 4. Verify Base Services on Master

Common verification commands:

```bash
systemctl status lsfd.service
bhosts
lsf_daemons status
```

Where:

- `systemctl status lsfd.service` checks whether the service exists and is running.
- `bhosts` shows cluster node status.
- `lsf_daemons status` shows the LSF daemon status on the current node.

## 5. Add Slave Node on Master

When configuring the slave, you must first modify the cluster configuration file on the master:

```bash
cd /softwares/IBM_lsf/app/lsf/conf
gedit lsf.cluster.workarea_cluster
```

Following the existing `sh05` configuration line, add the slave at the end:

```text
sh06      !         !          1           ()
```

You can also copy `sh05`'s line and modify the node name, ensuring the parentheses at the end are empty.

Reload and restart the master batch daemon:

```bash
lsadmin reconfig
badmin mbdrestart
```

## 6. Share LSF Installation Directory from Master

Since the slave does not have its own LSF installation, the master must share `/softwares` with the slave.

Edit the master's `/etc/exports`:

```bash
gedit /etc/exports
```

Add:

```exports
/softwares *(rw,sync,no_root_squash)
```

Re-export after saving:

```bash
exportfs -arv
```

The master-side preparation for the slave is now complete.

## 7. Mount /softwares on Slave

On the slave, edit the autofs configuration:

```bash
gedit /etc/auto.nfs
```

Add the `/softwares` share from the master:

```text
/softwares -fstype=nfs,rw,sync,soft 192.168.113.50:/softwares
```

Save and restart autofs:

```bash
systemctl restart autofs.service
df -h
```

Confirm that `/softwares` is successfully mounted from the master.

## 8. Start LSF Services on Slave

After the slave has access to the master's shared LSF environment, load the environment and start services:

```bash
cd /softwares/IBM_lsf/app/lsf
source profile.lsf
lsadmin limstartup
lsadmin resstartup
badmin hstartup
```

## 9. Cluster Verification

On the master or any node with the LSF environment loaded, check the cluster status:

```bash
bhosts
lsf_daemons status
```

If `sh06` appears in the `bhosts` output with a normal status, the slave has successfully joined the LSF cluster.

## Troubleshooting Tips

- `bhosts` does not show the slave: Check whether `sh06` was added to `lsf.cluster.workarea_cluster`, and confirm that `lsadmin reconfig` and `badmin mbdrestart` were executed.
- Slave cannot run LSF commands: Verify that `/softwares` is mounted successfully, and re-run `source profile.lsf`.
- Daemon startup fails: Check hostname resolution, master-slave connectivity, NFS mounts, `LSF_RSH=ssh`, and whether the user environment is consistent across master and slave.
