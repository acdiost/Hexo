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
#make install-autostart     //自动启动守护进程
```

---

## 客户端安装bacula

```bash
yum install -y gcc gcc-c++
tar -zxf bacula-9.4.4.tar.gz
cd bacula-9.4.4/

./configure  --enable-client-only
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
# 242行编辑数据库信息，新创建用户为bacula，密码为空，数据库名为bacula
```

---

## 测试

**启动测试**

```bash
/etc/bacula/bacula start
# Starting the Bacula Storage daemon
# Starting the Bacula File daemon
# Starting the Bacula Director daemon
bacula status
# bacula-sd (pid 40493) is running...
# bacula-fd (pid 40502) is running...
# bacula-dir (pid 40510) is running...
```

```bash
[root@localhost bacula]# bconsole
Connecting to Director localhost:9101
1000 OK: 103 localhost.localdomain-dir Version: 9.4.4 (28 May 2019)
Enter a period to cancel a command.
*

*status
Status available for:
     1: Director
     2: Storage
     3: Client
     4: Scheduled
     5: Network
     6: All
Select daemon type for status (1-6):6
```

**实操备份**

```bash
# 新建需要备份的测试目录
mkdir /baculatest
echo "test" > /baculatest/test.txt
# 修改bacula-dir.conf将Fileset中File = /sbin 改为 File = /baculatest/ 约125行。然后重启
bacula restart
bconsole
#运行run，选择1
*run
Automatically selected Catalog: MyCatalog
Using Catalog "MyCatalog"
A job name must be specified.
The defined Job resources are:
     1: BackupClient1
     2: BackupCatalog
     3: RestoreFiles
Select Job resource (1-3): 1
Run Backup job
JobName:  BackupClient1
Level:    Incremental
Client:   localhost.localdomain-fd
FileSet:  Full Set
Pool:     File (From Job resource)
Storage:  File1 (From Job resource)
When:     2019-11-06 08:25:45
Priority: 10
OK to run? (yes/mod/no): yes
Job queued. JobId=1
You have messages.
# 完成后将有输出 ok
# 查看状态status可简写s
*s
Status available for:
     1: Director
     2: Storage
     3: Client
     4: Scheduled
     5: Network
     6: All
Select daemon type for status (1-6): 1
localhost.localdomain-dir Version: 9.4.4 (28 May 2019) x86_64-pc-linux-gnu redhat (Core)
Daemon started 06-Nov-19 08:24, conf reloaded 06-Nov-2019 08:24:52
 Jobs: run=2, running=0 mode=0,0
 Heap: heap=270,336 smbytes=106,320 max_bytes=132,779 bufs=317 max_bufs=346
 Res: njobs=3 nclients=1 nstores=2 npools=3 ncats=1 nfsets=2 nscheds=2

Scheduled Jobs:
Level          Type     Pri  Scheduled          Job Name           Volume
===================================================================================
Incremental    Backup    10  06-Nov-19 23:05    BackupClient1      Vol-0001
Full           Backup    11  06-Nov-19 23:10    BackupCatalog      Vol-0001
====

Running Jobs:
Console connected at 06-Nov-19 08:25
No Jobs running.
====

Terminated Jobs:
 JobId  Level      Files    Bytes   Status   Finished        Name
====================================================================
     1  Full           2         5   OK       06-Nov-19 08:25 BackupClient1
     2  Restore        0         0   Error    06-Nov-19 08:37 RestoreFiles

====


# 在控制台将有提示：You have mail in /var/spool/mail/root
# 安装mailx并查看邮件
yum install -y mailx
[root@localhost baculatest]# mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 1 message 1 new
>N  1 Bacula                Wed Nov  6 08:25  65/3414  "Bacula: Backup OK of "
& 1
# 节选，我这里sd报错了
  SD Errors:              0
  FD termination status:  OK
  SD termination status:  OK
  Termination:            Backup OK
& exit
```

**删除创建的文件并恢复**

```bash
rm test.txt

*run
A job name must be specified.
The defined Job resources are:
     1: BackupClient1
     2: BackupCatalog
     3: RestoreFiles
Select Job resource (1-3): 3

```



---



### 总结：需要的配置

*详细参数按自己的需求修改*

**服务端bacula-dir.conf需要新增客户端的配置**

```
# baucla版本9.4.4
Job{                                                          #job的作用是定义一个备份任务，一些参数像差异备份 备份周期 日志
  Name= 客户端名称
  Type = Backup
  Level = Incremental
  Client =  客户端名称
  FileSet = 要备份的文件名称
  Schedule = "WeeklyCycle"
  Storage = File
  Messages = Standard
  Pool = File
  Priority = 10
  Write Bootstrap = "/opt/bacula/var/bacula/working/%c.bsr"
}
 
FileSet {                                                    #在fileset里面定义客户端要备份的文件或者目录
  Name = 要备份的文件名称
  Include {
    Options {
      signature = MD5
    }
    File = "文件路径"
  }
 
Client {                                                    #定义客户端的一些参数，比如ip 端口 等
  Name = 客户端名称
  Address = 客户端IP地址
  FDPort = 9102
  Catalog = MyCatalog
  Password = "客户端密码"          # password for FileDaemon
  File Retention = 30 days            # 30 days
  Job Retention = 6 months            # six months
  AutoPrune = yes                     # 修剪Prune expired Jobs/Files
}
```

**客户端bacula-fd.conf的配置**

```
# baucla版本9.4.4
Director {
  Name = 客户端名称
  Password = "客户端密码"
}
 
#
# Restricted Director, used by tray-monitor to get the
#   status of the file daemon
#
Director {
  Name = 服务端名称
  Password="Monitor密码"
  Monitor = yes
}
 
#
# "Global" File daemon configuration specifications
#
FileDaemon {                          # this is me
  Name = 服务端名称
  FDport = 9102                  # where we listen for the director
  WorkingDirectory = /opt/bacula/working
  Pid Directory = /var/run
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib64
}
 
# Send all messages except skipped files back to Director
Messages {
  Name = Standard
  director = 服务端名称 = all, !skipped, !restored
}
```



---



## 未完，待续...