# Diskquota

## Diskquota 是什么

Diskquota 是一种用于限制文件系统磁盘使用的技术。 PouchContainer 使用 Diskquota 来限制容器文件系统的磁盘空间使用。众所周知，基于块设备的方法可以通过设置块设备的大小来限制磁盘空间，但基于文件系统的方法则难以实现。Diskquota 就是用于来限制文件系统的磁盘使用的。目前 PouchContainer 支持基于 graphdriver overlayfs 的 Diskquota。

目前底层文件系统中只有 EXT4 和 XFS 支持 Diskquota，另外，有三种方式来实现：**user quota**，**group quota** 和 **project quota**。

限制磁盘使用有两个维度：

* 使用配额（块配额）：用于限制容器文件系统目录使用量（并非 inode 数量）。
* 文件配额（inode配额）：用于限制文件或 inode 的分配。

PouchContainer 目前仅支持块配额，暂时没有支持inode配额的计划。

## PouchContainer 中的 Diskquota

PouchContainer 中的 Diskquota 依赖于运行在容器中的内核版本，下表所示为在不同文件系统中支持 Diskquota 的内核版本。

|| user/group quota | project quota|
|:---:| :----:| :---:|
|EXT4| >= 2.6|>= 4.5|
|XFS|>= 2.6|>= 3.10|

尽管不同的文件系统在相应内核版本下均支持了 Diskquota，但用户依然需要安装 [quota-tools-4.04](https://nchc.dl.sourceforge.net/project/linuxquota/quota-tools/4.04/quota-4.04.tar.gz)。该配额工具目前并未被打包进 PouchContainer 的 rpm 包中，之后我们会对此作出支持。

## 概述

在 PouchContainer 中容器有两种方法访问底层文件系统。一种是容器的 rootfs，另一种是宿主机映射到容器内部的 volume。这两个维度的 Diskquota 均有支持。

### Rootfs Diskquota

用户可以使用选项 `--disk-quota` 来限制一个已经创建的容器的 rootfs 磁盘使用量，例如 `--disk-quota 10g`。设置成功后，可以通过 `df -h` 命令看到 rootfs 的大小为 10GB，表示 Diskquota 已经生效。

```bash
$ pouch run -ti --disk-quota 10g registry.hub.docker.com/library/busybox:latest df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  10.0G     24.0K     10.0G   0% /
tmpfs                    64.0M         0     64.0M   0% /dev
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                    64.0M         0     64.0M   0% /run
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     1.9G         0      1.9G   0% /sys/firmware
tmpfs                     1.9G         0      1.9G   0% /proc/scsi
```

### Volume Diskquota

用户也可以在创建容器时设置 Volume 的磁盘配额。 直接使用 `--option` 或 `-o` 选项来标识磁盘限制，例如 `-o size=10g`。

在创建 Diskquota 限制的 Volume 后，用户可以将该 Volume 绑定到某运行中的容器上。在下面的例子中，运行命令 `pouch run -ti -v volume-quota-test:/mnt registry.hub.docker.com/library/busybox:latest df -h`，在运行中的容器中，`/mnt` 目录的大小被限制为 10GB。

```bash
$ pouch volume create -n volume-quota-test -d local -o mount=/data/volume -o size=10g
Name:         volume-quota-test
Scope:
Status:       map[mount:/data/volume sifter:Default size:10g]
CreatedAt:    2018-3-24 13:35:08
Driver:       local
Labels:       map[]
Mountpoint:   /data/volume/volume-quota-test

$ pouch run -ti -v volume-quota-test:/mnt registry.hub.docker.com/library/busybox:latest df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  20.9G    212.9M     19.6G   1% /
tmpfs                    64.0M         0     64.0M   0% /dev
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                    64.0M         0     64.0M   0% /run
/dev/sdb2                10.0G      4.0K     10.0G   0% /mnt
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     1.9G         0      1.9G   0% /sys/firmware
tmpfs                     1.9G         0      1.9G   0% /proc/scsi
```