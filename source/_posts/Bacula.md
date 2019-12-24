---
title: Bacula
categories:
  - 工作
tags:
  - bacula
date: 2019-12-24 11:49:44
mp3: https://link.hhtjim.com/163/481160.mp3
cover: https://www.bing.com/th?id=OHR.ReindeerNorway_ZH-CN5913190372_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---

# Bacula

---

[bacula下载链接](https://nchc.dl.sourceforge.net/project/bacula/bacula/9.4.4/bacula-9.4.4.tar.gz)

[所有版本下载](https://sourceforge.net/projects/bacula/files/)

[官方文档](https://www.bacula.org/9.4.x-manuals/en/main/Index.html)

## 服务端安装bacula

```bash
# https://www.bacula.org/9.4.x-manuals/en/main/Installing_Bacula.html#6568
# debian 
# sudo apt-get install gcc g++ qt5-qmake qt5-default

yum install -y gcc gcc-c++ mysql-devel
tar -zxf bacula-9.4.4.tar.gz
cd bacula-9.4.4/

# 官方建议：
./configure \
  --enable-smartalloc \
  --sbindir=/opt/bacula/bin \
  --sysconfdir=/opt/bacula/etc \
  --with-pid-dir=/opt/bacula/working \
  --with-subsys-dir=/opt/bacula/working \
  --with-working-dir=/opt/bacula/working
  
  
./configure \
# 有助于检测内存泄漏
--enable-smartalloc \
# 启用mysql 
--with-mysql \
# 启用构建小巧轻便的Readline替换例程
--enable-conio \
# 启用Bacula图形化管理工具BAT
--enable-bat  \
# 启用托盘监控工具
--enable-tray-monitor

make && make install
```

---

## 客户端安装bacula

```bash
yum install -y gcc gcc-c++
tar -zxf bacula-9.4.4.tar.gz
cd bacula-9.4.4/

./configure  --enable-client-only --enable-smartalloc
make && make install
```

---

## 创建备份还原目录

```bash
mkdir -p /bacula/backup /bacula/restore
sudo chown -R dawn:dawn /bacula/
sudo chmod -R 700 /bacula/

```

---

## 数据库初始化

```bash
su root
cd /etc/bacula/
分配权限
./grant_mysql_privileges -u root -p
创建数据库
./create_mysql_database -u root -p
创建表
./make_mysql_tables -u root -p
```

---

## 配置文件配置

> Bacula的备份和恢复工作由下面几个部分完成
> （1）Director Daemon主要是控制端，控制存储，要备份的服务器，运行依赖于数据库，推荐使用MySQL
> （2）Storage Daemon主要是文件存储，所有的备份都会存储在这里，恢复的时候从这里取文件
> （3）File Daemon就是在客户端运行的进程，备份的时候传出文件，恢复的时候接收文件
> （4）Console是连接Director进行控制的
> （5）Monitor是监控备份过程的，完成或失败会有log和发送email，失败的日志很详细

```bash
su root
cd /etc/bacula/
vi bacula-dir.conf
```

---

## 测试