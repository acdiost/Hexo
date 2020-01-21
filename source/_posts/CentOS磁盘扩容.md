---
title: CentOS磁盘扩容
categories:
  - CentOS
tags:
  - CentOS
date: 2019-12-26 19:47:43
mp3: https://link.hhtjim.com/163/17047700.mp3
cover: https://www.bing.com/th?id=OHR.SloveniaAlps_EN-US3700970842_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---

> 今天，同事让分一个21T的磁盘进行扩容。将2T容量放在根分区，其余容量分给/data/目录下。
>
> 总结：
>
> - 根分区还有20k可用，必须要清理一定空间后才能正确扩容。
> - 将2T容量挂载到根分区后，剩余容量无法使用，目前未找到解决方法，除了重装，远程的机器，未见其身。

### 操作系统信息

> CentOS7

### 查看磁盘使用情况

```bash
# 自用mint
df -Th
            Filesystem     Type      Size  Used Avail Use% Mounted on
            udev           devtmpfs  1.9G     0  1.9G   0% /dev
            tmpfs          tmpfs     385M  1.9M  383M   1% /run
            /dev/sda4      ext4       46G   21G   24G  47% /
            tmpfs          tmpfs     1.9G  108M  1.8G   6% /dev/shm
            tmpfs          tmpfs     5.0M  4.0K  5.0M   1% /run/lock
            tmpfs          tmpfs     1.9G     0  1.9G   0% /sys/fs/cgroup
            /dev/sda2      ext4       44G  2.4G   39G   6% /home
            /dev/sda1      vfat      511M  6.1M  505M   2% /boot/efi
            /dev/sda3      ext4       28G  2.8G   24G  11% /opt
            tmpfs          tmpfs     385M   64K  385M   1% /run/user/1000

```

挂载一块磁盘

### lsblk列出磁盘信息

```bash
# 自用mint
root@dawn:/home/dawn/GitHub/Hexo/source/_posts# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 119.2G  0 disk 
├─sda1   8:1    0   512M  0 part /boot/efi
├─sda2   8:2    0  44.2G  0 part /home
├─sda3   8:3    0    28G  0 part /opt
└─sda4   8:4    0  46.6G  0 part /


```

**以sdb为例**

### fdisk /dev/sdb对磁盘进行分区

```bash
# 自用mint
root@dawn:/home/dawn/GitHub/Hexo/source/_posts# fdisk /dev/sda3

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

The old ext4 signature will be removed by a write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xa3589bc9.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table
   ######## 操作
   n # 增加分区
   p # 选择创建主分区
   1-4 # 选择分区号
   # 回车 默认
   # 回车 默认
   w # 保存并退出
```



### 修改分区号

```bash
fdisk /dev/sdb
t # 修改分区系统id
1-4 # 选择要修改的分区
8e # 指定要修改的id号 8e表示LVM
w # 保存并退出
# 重启一下
```



### 将磁盘格式化为xfs

```bash
mkfs -t xfs /dev/sdb
```



### 扩容分区

```bash
pvcreate /dev/sdb # 将物理硬盘初始化为物理卷
vgextend centos /dev/sdb # 向卷组增加物理卷来增加卷组容量
vgdisplay # 显示LVM卷组元数据信息
lvextend -L+扩容容量(M,G) /dev/mapper/centos-root /dev/sdb # 一般被增加的大小要小于可用大小
vgdisplay
xfs_growfs /dev/mapper/centos-root # xfs文件系统扩容命令，ext3/ext4文件系统用resize2fs命令来扩大或缩小
# 检查根分区是否扩容
df -Th
```

