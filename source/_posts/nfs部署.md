---
title: nfs部署
categories:
  - CentOS
tags:
  - NFS
date: 2020-08-25 22:01:19
mp3: https://link.hhtjim.com/163/22228465.mp3
cover: https://www.bing.com//th?id=OHR.Qixi2020_ZH-CN0736974777_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---

> NFS就是Network File System的缩写，它最大的功能就是可以通过网络，让不同的机器、不同的操作系统可以共享彼此的文件。

## 安装

```bash
sudo dnf install -y nfs-utils
systemctl enable rpcbind
systemctl enable nfs
```

## 配置文件 `/etc/exports`

目录 IP地址(权限)
```conf
mkdir /nfs
chown -R nfsnobody.nfsnobody /nfs

echo '/nfs 192.168.1.1(rw,sync)' >> /etc/exports

systemctl start rpcbind
systemctl start nfs
```

### 参数

|参数|作用|
|:-:|:-:|
|ro|只读|
|rw|读写|
|root_squash|当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户|
|no_root_squash|当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员|
|all_squash|无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户|
|sync|同时将数据写入到内存与硬盘中，保证不丢失数据|
|async|优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据|
|anonuid=xxx|将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）|
|anongid=xxx|将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）|
|secure|限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）|
|insecure|允许客户端从大于1024的tcp/ip端口连接服务器|

## 客户端

> 当使用 `df`,`ls` 命令无法加载磁盘信息或者列出目录，多半是nfs服务未启动或者连接失败导致； `dmesg` 可查看错误。

```bash
sudo yum install -y nfs-utils

showmount -e 192.168.1.1
mount -t 192.168.1.1:/nfs /nfs
```

## 卸载

```bash
umount /nfs
```

## 自动挂载

编辑 `/etc/fstab` 添加如下内容

```conf
192.168.1.1:/nfs /nfs nfs defaults 0 0
```
