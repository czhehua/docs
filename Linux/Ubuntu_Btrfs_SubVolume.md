# 主题：Ubuntu 使用 Btrfs 安装并配置子卷

**Revision:** 2.0  
**最后更新:** 2026/11/10

- [主题：Ubuntu 使用 Btrfs 安装并配置子卷](#主题ubuntu-使用-btrfs-安装并配置子卷)
	- [1. 需求](#1-需求)
	- [2. 安装与配置步骤（新）](#2-安装与配置步骤新)
	- [3. 安装和配置步骤（旧）](#3-安装和配置步骤旧)
		- [3.1. 安装时选择文件系统为 Btrfs](#31-安装时选择文件系统为-btrfs)
		- [3.2. 创建并转换 Btrfs 子卷](#32-创建并转换-btrfs-子卷)
			- [3.2.1. 关闭 swap](#321-关闭-swap)
			- [3.2.3. 创建 Btrfs 子卷](#323-创建-btrfs-子卷)
			- [3.2.4. 编辑  fstab](#324-编辑--fstab)
			- [3.2.4. 编辑并更新 grub 配置](#324-编辑并更新-grub-配置)
			- [3.2.5. 确认 Btrfs 子卷已正确挂载](#325-确认-btrfs-子卷已正确挂载)
			- [3.2.6. 清理迁移前的文件系统](#326-清理迁移前的文件系统)
			- [3.2.7. 重新创建 swap 文件](#327-重新创建-swap-文件)
		- [3.3. 对 Btrfs 卷中已有文件进行压缩](#33-对-btrfs-卷中已有文件进行压缩)
		- [3.4. 安装 Snapper 用于管理快照](#34-安装-snapper-用于管理快照)
			- [3.4.1. 安装 Snapper](#341-安装-snapper)
			- [3.4.3. 查看 snapshot 列表](#343-查看-snapshot-列表)
			- [3.4.4. 手动创建备份](#344-手动创建备份)
			- [3.4.4. 手动删除备份](#344-手动删除备份)
			- [3.4.5. 禁用开机备份](#345-禁用开机备份)


## 1. 需求 

Btrfs 是一个相对较新的文件系统，通常发音为 b-tree FS 或 butter FS，它提供了许多先进的功能，如CoW（写时文件复制）、快照、压缩、校验和等。

## 2. 安装与配置步骤（新） 

参考 Github [ubuntu-btrfs-install](https://github.com/sirwobbythefirst/ubuntu-btrfs-install) 项目，安装后运行 `ubuntu-btrfs-install.sh` 脚本，可实现自动配置。

以 Ubuntu 26.04 LTS 在 VMware 虚拟机中安装为例，主要步骤如下：

1. 关机状态下，虚拟机设置的 Options - Advanced 将虚拟机设置为 UEFI 启动。
2. 进入到 Ubuntu 26.04 安装器，选择手动分区。
   - `/sda1` 创建一个 2GB 的 ext4 分区，挂载点为 `/boot`
   - `/sda2` 安装器会自自动创建一个 1GB 的 FAT32 分区 ，挂载点为  `/boot/efi`
   - `/sda3` 剩余所有磁盘空间，创建 Btrfs 分区，挂载点为 `/`
3. 点击安装，但安装完后，先不要重启，运行如下命令：
   - `sudo bash ./ubuntu-btrfs-install.sh sda3 sda1 sda2`
4. 脚本运行完成后，重启进入系统。
5. 运行 `sudo btrfs subvolume list /` 可查看各子卷。
6. 安装 snapper 以及 btrfs-assistant：
   - 安装工具：`sudo apt update && sudo apt install -y snapper btrfs-assistant`
   - 为 Snapper 创建 Root 配置：`sudo snapper -c root create-config /`
   - 打开 btrfs-assistant 程序，在 Snapper Settings 设置自动备份的份数，关闭 `Snapper boot` 后点击 `Apply systemd changes`

## 3. 安装和配置步骤（旧）  

注意：这里是旧版内容，仅供存档参考。请按新版本设置。

本章节主要参照如下网页，将介绍使用 Btrfs 安装 Ubuntu 系统时，需要注意的一些细节。
- [正确地将Ubuntu安装到Btrfs中](https://edgeneko.aiursoft.cn/2024/08/17/BtrfsWithUbuntuServer/)

### 3.1. 安装时选择文件系统为 Btrfs

安装过程与普通安装流程类似，只需注意如下要点：
- 根分区（/）的文件系统选择为 Btrfs 并格式化。
- 此时无需为家目录（/home）创建新的分区，后续将使用 Btrfs 子卷进行管理。

安装完后成，正常登录系统，并进行接下来的配置。

### 3.2. 创建并转换 Btrfs 子卷

Ubuntu 24.04 安装器不再使用子卷格式安装，而是将所有文件安装在根子卷中。
- 不使用分卷管理将：无法单独对系统文件（或用户数据）进行快照等。
- 在下面的步骤中，会介绍如何创建以下子卷并将其挂载到对应目录。

使用 `sudo Btrfs subvolume list /` 命令，查看 Btrfs 子卷的情况（未配置前为空）：

在下面的步骤中，将创建以下子卷，并将其挂载到对应目录：

| 子卷名称   | 挂载目录    | 用途         |
| :--------- | :---------- | :----------- |
| @          | /           | 系统根目录   |
| @home      | /home       | 用户数据     |
| @cache     | /var/cache  | apt等缓存    |
| @log       | /var/log    | 日志         |
| @swap      | /swapfile   | swap文件     |
| @snapshots | /.snapshots | 根目录的快照 |

对于 /tmp 路径，后续将直接使用 tmpfs 进行挂载，根据[此文章](https://ubuntu.com/blog/data-driven-analysis-tmp-on-tmpfs)，使用 tmpfs 可以提高性能并减少对磁盘的写入。

**注意**：
- 接下来的操作会对系统所在的 Btrfs 卷进行修改，请确认输入了正确的命令并谨慎操作。操作失误可能导致系统故障甚至数据丢失。重要数据建议先备份。
- 操作要求您具有一定的 Linux 与 Btrfs 基础知识，如编辑 /etc/fstab、挂载或取消挂载卷、进行 grub-install 等操作。建议在开始前先完整阅读以下步骤，确保您理解每一步的含义。

#### 3.2.1. 关闭 swap

如果 Btrfs 卷中存放了 swap 文件（例如：安装 Ubuntu Server 时会自动创建），需要先关闭并删除文件以便对根目录进行快照：

使用 `swapon --show` 查看当前的swap文件，如有则使用 `swapoff -a` 关闭并删除。

```bash
sudo swapoff -a
sudo rm /swap.img  # Change the path to your swap file
```

然后编辑 /etc/fstab 文件，删除或注释掉 swap 文件对应的条目（一般位于文件末尾）。

```
sudo nano /etc/fstab
# /swap.img     none    swap    sw      0       0
```

#### 3.2.3. 创建 Btrfs 子卷

为 Btrfs 文件系统的根子卷创建一个可写快照：

```bash
sudo Btrfs subvolume snapshot / /@
```

创建所需的其它子卷，并将文件移动到对应的子卷中：

```bash
sudo Btrfs subvolume create /@home
sudo mv /@/home/* /@home

sudo Btrfs subvolume create /@cache
sudo mv /@/var/cache/* /@cache

sudo Btrfs subvolume create /@log
sudo mv /@/var/log/* /@log

sudo Btrfs subvolume create /@swap
sudo mkdir /@/swapfile  # Create a directory for the mount point

sudo Btrfs subvolume create /@snapshots
sudo mkdir /@/.snapshots  # Create a directory for the mount point
```

清空tmp目录，稍后将使用 tmpfs 来挂载：

```bash
sudo rm -rf /@/tmp/*
```

#### 3.2.4. 编辑  fstab

编辑 /@/etc/fstab，设置各个子卷的挂载点信息。

```bash
sudo vi /@/etc/fstab
# /dev/disk/by-uuid/<UUID> / Btrfs defaults 0 1
```

找到类似如下项目，其中的 UUID 为 Btrfs 所在分区的标识符。
```txt
/dev/disk/by-uuid/<UUID> / Btrfs defaults 0 1
```

修改其内容，并为其他子卷设置挂载项如下：
```txt
/dev/disk/by-uuid/<UUID> / Btrfs subvol=@,compress=zstd:3,noatime,space_cache=v2,commit=120,autodefrag 0 1
/dev/disk/by-uuid/<UUID> /home Btrfs subvol=@home,noatime,space_cache=v2,autodefrag 0 1
/dev/disk/by-uuid/<UUID> /.snapshots Btrfs subvol=@snapshots,noatime,space_cache=v2 0 1
/dev/disk/by-uuid/<UUID> /var/cache Btrfs subvol=@cache,noatime,space_cache=v2 0 1
/dev/disk/by-uuid/<UUID> /var/log Btrfs subvol=@log,noatime,space_cache=v2 0 1
/dev/disk/by-uuid/<UUID> /swapfile Btrfs subvol=@swap,defaults 0 1

tmpfs /tmp tmpfs defaults,noatime,mode=1777 0 0
```

上面已给出各子卷的示例挂载配置，并在最后一行已添加了 /tmp 的挂载点。一些参数的含义：

| 项目 | 含义 |
| :-- | :--- |
| /dev/disk/by-uuid/<UUID> | 通过 UUID 指定要挂载的设备，防止设备名变更无法挂载 |
| / 等 | 挂载目标路径 |
| Btrfs | 指定的文件系统类型 |
| subvol=@​​ 等 | 指定挂载 Btrfs 子卷名 |
| compress=zstd:3​​ | 启用透明压缩。目前只能设置根子卷，其他子卷继承该设置  |
| noatime | 禁止更新文件的访问时间，减少磁盘写入 |
| space_cache=v2 | 使用 v2 版空间缓存，提升空闲空间管理的效率 |
| commit=120​​ | 设置每 120 秒同步一次事务到磁盘，减少频繁提交 |
| autodefrag | 自动对文件进行碎片整理，适用于频繁修改的文件 |
| 0 | 表示不需要 dump 备份工具备份此文件系统 |
| 1 | 启动时用 fsck 检查此文件系统 (Btrfs 通常非必要) |

#### 3.2.4. 编辑并更新 grub 配置

编辑 grub 默认配置，以便在后续启动时进行操作：

```bash
sudo nano /etc/default/grub
```

修改如下两项，以便开机时能显示 grub 菜单：

```txt
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=10
```

修改 GRUB_CMDLINE_LINUX_DEFAULT 的值，桌面版和服务器版略有不同：

```txt
# For Ubuntu Desktop
GRUB_CMDLINE_LINUX_DEFAULT="rootflags=subvol=@ quiet splash"
# For Ubuntu Server
GRUB_CMDLINE_LINUX_DEFAULT="rootflags=subvol=@"
```

更新 grub 配置并重启系统：

```bash
sudo update-grub

sudo reboot
```

当显示 GRUB 菜单时，按下 e 键，编辑 grub 启动项。  

找到 linux 和 initrd 开头的两行，在路径开头加上 /@ 前缀。

修改前：
```txt
linux   /boot/ ...
initrd  /boot/...
```
修改后：
```txt
linux   /@/boot/ ...
initrd  /@/boot/...
```

检查 linux 行中的参数中有无 "rootflags=subvol=@"，正常情况下应该已添加。

按 F10 键开始引导系统。

#### 3.2.5. 确认 Btrfs 子卷已正确挂载

进入系统后，确认目前的根目录由@子卷挂载：

```bash
mount | grep ' / '
```

输出类似如下内容：

```bash
/dev/sda1 on / type Btrfs (rw,noatime,compress=zstd:3,space_cache=v2,autodefrag,commit=120,subvolid=256,subvol=/@)
```

如果输出中没有 subvol=@，则说明根目录没有正确挂载，需要检查之前的步骤是否有错误。

查看各子卷，输入如下命令：

```bash
sudo Btrfs subvolume list /
```

返回的信息类似如下内容：

```txt
ID 256 gen 321 top level 5 path @
ID 257 gen 311 top level 5 path @home
ID 258 gen 320 top level 5 path @cache
ID 259 gen 321 top level 5 path @log
ID 260 gen 174 top level 5 path @swap
ID 261 gen 304 top level 5 path @snapshots
```

重新安装 grub 已永久应用新的挂载方式：

```bash
# For BIOS Boot:
sudo grub-install /dev/sdX  # Change the boot device name 
# For UEFI Boot:
sudo grub-install --efi-directory=/boot/efi
```

更新 grub 配置并重启系统：

```bash
sudo update-grub

sudo reboot
```

#### 3.2.6. 清理迁移前的文件系统

首先挂载 Btrfs 根子卷，列出其文件：

```bash
sudo mkdir -p /mnt/sysdrv
sudo mount /dev/sda1  /mnt/sysdrv  # Change the root device, e.g. /dev/nvme0n1p1

cd /mnt/sysdrv
ls -la
```

返回的信息中包含了 @ 开头的子卷，同时也包含了迁移前的系统：

```txt
'@/'
'@cache/'
'@home/'
...
bin/
boot/
usr/
...
```

删除迁移前的系统目录，但保留刚创建的所有子卷，在bash中运行以下命令：
- 请确保已正确挂载了根子卷，并确认要删除的文件是正确的。在 echo 的结果中确保没有删除不应删除的文件。

```bash
shopt -s extglob  # Enable extended globbing

sudo echo !(@*)  # Make sure you are deleting the correct files

sudo rm -rf !(@*)  # Perform the deletion

shopt -u extglob  # Disable extended globbing
```

到这里为止，Ubuntu 系统已经成功迁移到子卷格式下。

#### 3.2.7. 重新创建 swap 文件

如有必要，可重新创建 swap 文件：

```bash
cd /swapfile
sudo touch swap.img
sudo chattr +C swap.img  # Disable CoW on the swap file
sudo fallocate -l 2G swap.img  # Change the size to your needs
sudo chmod 600 swap.img
sudo mkswap swap.img
```

编辑 /etc/fstab 以添加 swap 文件：

```bash
sudo nano /etc/fstab
```

在文件最后添加如下行：
```
/swapfile/swap.img     none    swap    sw      0       0
```

使用swapon -a命令启用swap文件：

```bash
sudo swapon -a

sudo swapon --show
```

返回信息如下：

```txt
NAME               TYPE SIZE USED PRIO
/swapfile/swap.img file   2G   0B   -2
```

### 3.3. 对 Btrfs 卷中已有文件进行压缩

由于 Btrfs 是写时复制，对于已有的文件默认不会压缩，可使用如下命令，对根目录和家目录进行压缩：

```bash
sudo btrfs filesystem defragment -r -czstd /

sudo btrfs filesystem defragment -r -czstd /home
```

使用 compsize 工具可查看文件的压缩率。

```bash
sudo apt install btrfs-compsize

sudo compsize -x /

sudo compsize -x /home
```

在 Btrfs 文件系统中，使用 `df -h` 命令显示的空间可能不准确，可以用如下命令查看实际空间占用：

```bash
sudo btrfs filesystem usage /

sudo btrfs filesystem du -s /

sudo btrfs filesystem du -s /home
```

**注意**：
- 如果是在 Vmware 虚拟机中运行，动态分配的 vmdk 文件的大小，取决于客户机内 Btrfs 已分配的空间，无法自动收缩。即使执行 `sudo vmware-toolbox-cmd disk shrink /` 也无法减少 vmdk 文件的大小。

### 3.4. 安装 Snapper 用于管理快照

​Snapper​​ 是专为 ​​Btrfs​​ 和 ​​LVM​​ 文件系统设计的开源快照管理工具，由 openSUSE 团队开发，支持快照创建、对比、回滚及自动化管理。

#### 3.4.1. 安装 Snapper

使用 apt 安装 snapper：

```bash
sudo apt install snapper
```

初次使用前需要配置 snapper，由于已经创建的 @snapshot 子卷的挂载点与其冲突，可按如下方案解决。

卸载并删除 .snapshot 挂载点：
​
```bash
sudo umount /.snapshots
sudo rmdir /.snapshots
```

为 Snapper 创建配置：

```bash
sudo snapper -c root create-config /
```

删除 Snapper 自动创建的新子卷​：

```bash
sudo Btrfs subvolume delete /.snapshots
```

重新创建挂载点目录并挂载 @sanpshot 子卷：

```bash
sudo mkdir /.snapshots
sudo mount -a  # Use settings in /etc/fstab to mount
```

#### 3.4.3. 查看 snapshot 列表

运行 snapper list 查看当前 snapshot：

```bash
sudo sanpper list
```

其返回结果类似如下：
```txt
 # | Type   | Pre # | Date | User | Cleanup | Description | Userdata
---+--------+-------+------+------+---------+-------------+---------
0  | single |       |      | root |         | current     |
```

其中 #0 为当前状态，不可删除。

#### 3.4.4. 手动创建备份

```bash
sudo snapper -c root create -d test1
```

#### 3.4.4. 手动删除备份

删除单个备份：
```bash
sudo snapper delete --sync 1
```

删除多个备份：
```bash
sudo snapper delete --sync 1 2
```

删除多个连号的备份：
```bash
sudo snapper delete --sync 1-3
```

#### 3.4.5. 禁用开机备份

Snapper 每次开机会创建一个备份，可通过如下命令关闭：

```bash
sudo systemctl disable snapper-boot.timer
```

更多 Snapper 的用法，可访问 [Arch Wiki](https://wiki.archlinux.org/title/Snapper)，或查阅 `man snapper` 手册。

（完）