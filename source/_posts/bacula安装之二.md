---
title: bacula安装之二
categories:
  - 工作
tags:
  - bacula
date: 2019-12-25 18:44:54
mp3: 
cover: https://www.bing.com/th?id=OHR.WarsawXmas_ZH-CN5981724395_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---

# Bacula安装之二

## docker容器作为客户端

```bash
# 安装docker 安装较慢
curl -k -ssl https://get.docker.com | sudo sh
# 新建修改daemon将源换成国内的
# 将2375端口开启，让idea和pycharm可以访问使用
vi /lib/systemd/system/docker.service
#ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock
systemctl daemon-reload
systemctl restart docker
sudo docker pull centos
sudo docker run -dit -p 9101:9101 -p 9102:9102 -p 9103:9103 --name bacula-client centos
# 安装bacula
sudo docker cp bacula-9.4.4.tar.gz bacula-client:/tmp
sudo docker exec -it bacula-client bash
yum install -y wget
cat /etc/os-release
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
yum makecache
yum install -y gcc gcc-c++ make
 cd /tmp/
tar -zxf bacula-9.4.4.tar.gz
cd bacula-9.4.4
./configure --enable-client-only --enable-smartalloc
make && make install

```

---

## ubuntu作为服务端

```bash
su root
tar -zxf bacula-9.4.4.tar.gz
cd bacula-9.4.4
./configure --enable-conio --enable-smartalloc --with-mysql
make && make install
cd /etc/bacula/
./grant_mysql_privileges
# Privileges for user bacula granted on database bacula. 用户名为bacula，密码为空
./create_mysql_database
# Creation of bacula database succeeded.
./make_mysql_tables  -u bacula
# Creation of Bacula MySQL tables succeeded.

```

## 服务端配置文件配置

```bash
su root
cd /etc/bacula/
# 修改前备份
cp bacula-dir.conf bacula-dir.conf.bak
cp bacula-sd.conf bacula-sd.conf.bak
cp bacula-fd.conf bacula-fd.conf.bak
cp bconsole.conf bconsole.conf.bak
vi bacula-dir.conf
```

---

> 组成 Bacula 备份系统有三个主要的部分,包括主控端、存储端和客户端,这三个部分都 有 各 自 的 配 置 文 件 , 相 对 应 的 是 主 控 端 ( bacula-dir.conf )、 存 储 端(bacula-sd.conf)和客户端(bacula-fd.conf)。
>
> 服务端和存储端因测试放在一起

**bacula-dir.conf配置**

**主控端的Director项：**

>   1、Name和Console端Director项的Name一致；
>
>   2、Name要和介质端Director项的Name一致；
>
>   3、Name要和客户端Director项的Name一致；
>
>   4、Password项要和Console端的Password项的密码一致；

**主控端的Storage项：**

>   1、Device要和介质端Device项的Name一致；
>
>   2、MediaType要和介质端Device项的MediaType一致；
>
>   3、Password要和介质端Director项的Pasword一致；
>
> **主控端的Client项：**
>
>   Password要和客户端Director项的Password一致；

**原bacula-dir.conf**

```conf
Director {                            # define myself
  Name = dawn-dir
  DIRport = 9101                # where we listen for UA connections
  QueryFile = "/etc/bacula/query.sql"
  WorkingDirectory = "/opt/bacula/working"
  PidDirectory = "/var/run"
  Maximum Concurrent Jobs = 20
  Password = "zA8O1vlF+wMWxRQO41V2J8XFm3oiAUP1AiQgIoxvchqR"         # Console password
  Messages = Daemon
}
JobDefs {
  Name = "DefaultJob"
  Type = Backup
  Level = Incremental
  Client = dawn-fd
  FileSet = "Full Set"
  Schedule = "WeeklyCycle"
  Storage = File1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/opt/bacula/working/%c.bsr"
}
Job {
  Name = "BackupClient1"
  JobDefs = "DefaultJob"
}
Job {
  Name = "BackupCatalog"
  JobDefs = "DefaultJob"
  Level = Full
  FileSet="Catalog"
  Schedule = "WeeklyCycleAfterBackup"
  # This creates an ASCII copy of the catalog
  # Arguments to make_catalog_backup.pl are:
  #  make_catalog_backup.pl <catalog-name>
  RunBeforeJob = "/etc/bacula/make_catalog_backup.pl MyCatalog"
  # This deletes the copy of the catalog
  RunAfterJob  = "/etc/bacula/delete_catalog_backup"
  Write Bootstrap = "/opt/bacula/working/%n.bsr"
  Priority = 11                   # run after main backup
}
Job {
  Name = "RestoreFiles"
  Type = Restore
  Client=dawn-fd
  Storage = File1
  FileSet="Full Set"
  Pool = File
  Messages = Standard
  Where = /tmp/bacula-restores
}
FileSet {
  Name = "Full Set"
  Include {
    Options {
      signature = MD5
    }
    File = /sbin
  }
  Exclude {
    File = /opt/bacula/working
    File = /tmp
    File = /proc
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
  }
}
Schedule {
  Name = "WeeklyCycle"
  Run = Full 1st sun at 23:05
  Run = Differential 2nd-5th sun at 23:05
  Run = Incremental mon-sat at 23:05
}
Schedule {
  Name = "WeeklyCycleAfterBackup"
  Run = Full sun-sat at 23:10
}
FileSet {
  Name = "Catalog"
  Include {
    Options {
      signature = MD5
    }
    File = "/opt/bacula/working/bacula.sql"
  }
}
Client {
  Name = dawn-fd
  Address = dawn
  FDPort = 9102
  Catalog = MyCatalog
  Password = "FY2ntz2HzTZ5peV/uJqL/Ek4aSUvgTMLGZOka7Ush39R"          # password for FileDaemon
  File Retention = 60 days            # 60 days
  Job Retention = 6 months            # six months
  AutoPrune = yes                     # Prune expired Jobs/Files
}
Autochanger {
  Name = File1
  Address = dawn                # N.B. Use a fully qualified name here
  SDPort = 9103
  Password = "y9Av5fGEgYzJeL3fxgqaKQ4vKtTDue0nDykP4VSqI2H/"
  Device = FileChgr1
  Media Type = File1
  Maximum Concurrent Jobs = 10        # run up to 10 jobs a the same time
  Autochanger = File1                 # point to ourself
}
Autochanger {
  Name = File2
  Address = dawn                # N.B. Use a fully qualified name here
  SDPort = 9103
  Password = "y9Av5fGEgYzJeL3fxgqaKQ4vKtTDue0nDykP4VSqI2H/"
  Device = FileChgr2
  Media Type = File2
  Autochanger = File2                 # point to ourself
  Maximum Concurrent Jobs = 10        # run up to 10 jobs a the same time
}
Catalog {
  Name = MyCatalog
  dbname = "bacula"; dbuser = "bacula"; dbpassword = ""
}
Messages {
  Name = Standard
  mailcommand = "/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
  operatorcommand = "/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
  mail = root@localhost = all, !skipped
  operator = root@localhost = mount
  console = all, !skipped, !saved
  append = "/opt/bacula/log/bacula.log" = all, !skipped
  catalog = all
}
Messages {
  Name = Daemon
  mailcommand = "/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
  mail = root@localhost = all, !skipped
  console = all, !skipped, !saved
  append = "/opt/bacula/log/bacula.log" = all, !skipped
}
Pool {
  Name = Default
  Pool Type = Backup
  Recycle = yes                       # Bacula can automatically recycle Volumes
  AutoPrune = yes                     # Prune expired volumes
  Volume Retention = 365 days         # one year
  Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable
  Maximum Volumes = 100               # Limit number of Volumes in Pool
}
Pool {
  Name = File
  Pool Type = Backup
  Recycle = yes                       # Bacula can automatically recycle Volumes
  AutoPrune = yes                     # Prune expired volumes
  Volume Retention = 365 days         # one year
  Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable
  Maximum Volumes = 100               # Limit number of Volumes in Pool
  Label Format = "Vol-"               # Auto label
}
Pool {
  Name = Scratch
  Pool Type = Backup
}
Console {
  Name = dawn-mon
  Password = "mcdb6oVkXEUq4GzqnZabGge2b8TALFTezxQQzwhrs+hP"
  CommandACL = status, .status
}

```

**释义：**

```
Director {#全局设置                            
  Name = bacula-test-dir                #名称要和Console端、SD端、FD端一致
  DIRport = 9101                        #启动的端口
  QueryFile = "/usr/libexec/bacula/query.sql"
  WorkingDirectory = "/var/spool/bacula/"
  PidDirectory = "/var/run"
  Maximum Concurrent Jobs = 20            #同时执行的最大任务数量
  Password = "u3R1MUpLlKodHFr4PiqGQMGJfaNfhiXb14ul6xcpBA8d"    #密码和Console端一致
  Messages = Daemon
}
 
JobDefs {                            #默认的任务设置
  Name = "DefaultJob"                #任务名称
  Type = Backup                        #任务的类型，有备份、恢复、校验、管理等
  Level = Incremental                    #增量备份，有差异、全备份
  Client = bacula-test-fd                #备份的客户端
  FileSet = "Full Set"                    #选择客户端需要备份额目录及文件
  Schedule = "WeeklyCycle"                #备份周期
  Storage = File1                        #存储的方式
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/spool/bacula/%c.bsr"        #将作业记录写入文件
}
Job {
  Name = "Backup-17.98"                                #定义一个备份任务
  Type = Backup
  Level = Incremental
  Client = bacula-17-98                                #需要备份的客户端
  FileSet = "Windows"                                   #需要备份的目录和文件
  Schedule = "WeeklyCycle"
  Storage = File1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/spool/bacula/windows%c.bsr"
}
Job {
  Name = "BackupClient1"
  JobDefs = "DefaultJob"
}
Job {
  Name = "BackupCatalog"                    #定义一个元数据备份任务
  JobDefs = "DefaultJob"
  Level = Full
  FileSet="Catalog"
  Schedule = "WeeklyCycleAfterBackup"
  RunBeforeJob = "/usr/libexec/bacula/make_catalog_backup.pl MyCatalog"
  RunAfterJob  = "/usr/libexec/bacula/delete_catalog_backup"
  Write Bootstrap = "/var/spool/bacula/%n.bsr"
  Priority = 11                   
}
Job {
  Name = "RestoreFiles"                    #定义一个恢复任务
  Type = Restore
  Client=bacula-test-fd
  FileSet="Full Set"
  Storage = File1
  Pool = File
  Messages = Standard
  Where = /tmp/bacula-restores
}
FileSet {
  Name = "Full Set"                    #定义需要备份的目录及文件
  Include {
    Options {
      signature = MD5                    #采用MD5
      Compression = GZIP                #启用GZIP压缩
        }
    File = /var/lib/mysql                #需要备份的目录
  }
  Exclude {                                #无需备份的目录和文件
    File = /var/spool/bacula/
    File = /tmp
    File = /proc
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
  }
}
FileSet {
  Name = "Windows"                        #定义一个新的需要备份的设置
  Include {
    Options {
      signature = MD5
      compression = GZIP
      IgnoreCase = yes
    }
    File = "D:/msdn"
  }
}
Schedule {
  Name = "WeeklyCycle"                    #定义备份周期
  Run = Full 1st sun at 23:05
  Run = Differential 2nd-5th sun at 23:05
  Run = Incremental mon-sat at 23:05
}
Schedule {
  Name = "WeeklyCycleAfterBackup"        #另一种不同的备份周期
  Run = Full sun-sat at 23:10
}
FileSet {
  Name = "Catalog"                        #定义需要备份的元数据
  Include {
    Options {
      signature = MD5
    }
    File = "/var/spool/bacula/bacula.sql"
  }
}
Client {
  Name = bacula-test-fd                #定义需要备份的客户端
  Address = 192.168.17.100
  FDPort = 9102
  Catalog = MyCatalog
  Password = "nb5dURhiVT3RPsHkFvXetau9LrnL/FhY/ORWkFOTPamT"#密码要和FD端的一致
  File Retention = 30 days            #定义文件记录在 Catalog 数据库中的保留时间,不影响已存档的备份文件
  Job Retention = 3 months            #定义任务记录在 Catalog 数据库中的保留时间,不影响已存档的备份文件
  AutoPrune = yes                     #自动修剪
}
 
Client {
  Name = bacula-17-98
  Address = 192.168.17.98
  FDPort = 9102
  Catalog = MyCatalog
  Password = "nb5dURhiVT3RPsHkFvXetau9LrnL/FhY/ORWkFOTPamT"
  File Retention = 30 days
  Job Retention = 3 months
  AutoPrune = yes
}
Storage {
  Name = File1
  Address = 192.168.17.100                #存储端的地址
  SDPort = 9103
  Password = "qy5ILHHJNXLCUWGudeC1ke1hPioBo+ye2bUBU5ncxADg"
  Device = FileChgr1                #指定存储的设备。引用存储端配置文件的 Device{}的 Name 值
  Media Type = File1                    #与存储端配置文件的 Device{}的 Media Type 值相同
  Maximum Concurrent Jobs = 10        #定义此存储端所允许同时进行的任务最大数量
}
Storage {
  Name = File2
  Address = 192.168.17.100               
  SDPort = 9103
  Password = "qy5ILHHJNXLCUWGudeC1ke1hPioBo+ye2bUBU5ncxADg"
  Device = FileChgr2
  Media Type = File2
  Maximum Concurrent Jobs = 10       
}
Catalog {
  Name = MyCatalog                    #指定 Catalog{}名称
  dbname = "bacula"; dbuser = "bacula"; dbpassword = "bacula"
}
Messages {
  Name = Standard                    #定义日志
  append = "/var/log/bacula.log" = all, !skipped
  append = "/var/log/bacula.err.log" = error,warning,fatal
  catalog = all
}
Messages {
  Name = Daemon
  append = "/var/log/bacula.log" = all, !skipped
  append = "/var/log/bacula.err.log" = error,warning,fatal
}
Pool {
  Name = Default
  Pool Type = Backup
  Recycle = yes                       #是否重复使用 Volumes
  AutoPrune = yes                     #自动清理
  Maximum Volume Jobs = 1
  Volume Retention = 15 days         #卷的保留时间
  Maximum Volume Bytes = 5G          #Volume容量
  Maximum Volumes = 100               #Volumes数量l
  Label Format = "db-${Year}-${Month:p/2/0/r}-${Day:p/2/0/r}-id${JobId}"#卷命名方式
}
Pool {
  Name = File
  Pool Type = Backup
  Recycle = yes                       
  AutoPrune = yes                     
  Maximum Volume Jobs = 1
  Volume Retention = 15 days         
  Maximum Volume Bytes = 50G          
  Maximum Volumes = 100              
  Label Format = "db-${Year}-${Month:p/2/0/r}-${Day:p/2/0/r}-id${JobId}"
}
# Scratch pool definition
Pool {
  Name = Scratch
  Pool Type = Backup
}
Console {
  Name = bacula-test-mon
 Password = "fX2cKUkN2RSiLTkeQd5LCPDDw61PzGkuit2+7DvWxVG1"
  CommandACL = status, .status
}
```

---

**原bacula-sd.conf**

```
Storage {                             # definition of myself
  Name = dawn-sd
  SDPort = 9103                  # Director's port
  WorkingDirectory = "/opt/bacula/working"
  Pid Directory = "/var/run"
  Plugin Directory = "/usr/lib64"
  Maximum Concurrent Jobs = 20
}
Director {
  Name = dawn-dir
  Password = "y9Av5fGEgYzJeL3fxgqaKQ4vKtTDue0nDykP4VSqI2H/"
}
Director {
  Name = dawn-mon
  Password = "mUitzQLPo9zeWguyuRUltIsNJP5E2+vQFOwV915Yi5KM"
  Monitor = yes
}
Autochanger {
  Name = FileChgr1
  Device = FileChgr1-Dev1, FileChgr1-Dev2
  Changer Command = ""
  Changer Device = /dev/null
}
Device {
  Name = FileChgr1-Dev1
  Media Type = File1
  Archive Device = /tmp
  LabelMedia = yes;                   # lets Bacula label unlabeled media
  Random Access = Yes;
  AutomaticMount = yes;               # when device opened, read it
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Device {
  Name = FileChgr1-Dev2
  Media Type = File1
  Archive Device = /tmp
  LabelMedia = yes;                   # lets Bacula label unlabeled media
  Random Access = Yes;
  AutomaticMount = yes;               # when device opened, read it
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Autochanger {
  Name = FileChgr2
  Device = FileChgr2-Dev1, FileChgr2-Dev2
  Changer Command = ""
  Changer Device = /dev/null
}
Device {
  Name = FileChgr2-Dev1
  Media Type = File2
  Archive Device = /tmp
  LabelMedia = yes;                   # lets Bacula label unlabeled media
  Random Access = Yes;
  AutomaticMount = yes;               # when device opened, read it
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Device {
  Name = FileChgr2-Dev2
  Media Type = File2
  Archive Device = /tmp
  LabelMedia = yes;                   # lets Bacula label unlabeled media
  Random Access = Yes;
  AutomaticMount = yes;               # when device opened, read it
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Messages {
  Name = Standard
  director = dawn-dir = all
}

```

**释义：**

```
Storage {                             #定义存储端名称
  Name = bacula-test-sd
  SDPort = 9103                  
  WorkingDirectory = "/var/spool/bacula/"
  Pid Directory = "/var/run"
  Maximum Concurrent Jobs = 20
}
Director {
  Name = bacula-test-dir
  Password = "qy5ILHHJNXLCUWGudeC1ke1hPioBo+ye2bUBU5ncxADg"
}
Director {
  Name = bacula-test-mon
  Password = "7zSF0B9viPBuwkKpPC2PoJw1qxOEuvCxbWDxzXc1cHk9"
  Monitor = yes
}
 Autochanger {#是否自动更换设备
  Name = FileChgr1
  Device = FileChgr1-Dev1, FileChgr1-Dev2#定义需更换的设备
  Changer Command = ""
  Changer Device = /dev/null
}
 
Device {
  Name = FileChgr1-Dev1
  Media Type = File1#类型是文件（可以是DVD，磁带）
  Archive Device = /tmp#存放位置
  LabelMedia = yes;                   #媒体标注
  Random Access = Yes;
  AutomaticMount = yes;               #自动挂载
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Device {
  Name = FileChgr1-Dev2
  Media Type = File1
  Archive Device = /tmp
  LabelMedia = yes;                   
  Random Access = Yes;
  AutomaticMount = yes;               
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Autochanger {
  Name = FileChgr2
  Device = FileChgr2-Dev1, FileChgr2-Dev2
  Changer Command = ""
  Changer Device = /dev/null
}
 
Device {
  Name = FileChgr2-Dev1
  Media Type = File2
  Archive Device = /tmp
  LabelMedia = yes;                   
  Random Access = Yes;
  AutomaticMount = yes;               
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Device {
  Name = FileChgr2-Dev2
  Media Type = File2
  Archive Device = /tmp
  LabelMedia = yes;                   
  Random Access = Yes;
  AutomaticMount = yes;               
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}
Messages {
  Name = Standard
  director = bacula-test-dir = all
}
```

---

## 客户端配置文件配置

**客户端bacula-fd.conf**

```
# List Directors who are permitted to contact this File daemon
Director {
  Name = eec556ab8f52-dir # 要与服务端bacula-dir一致
  Password = ""要与服务端bacula-dir中client密码一致
}
# Restricted Director, used by tray-monitor to get the
Director {
  Name = eec556ab8f52-mon
  Password = ""
  Monitor = yes
}

# "Global" File daemon configuration specifications
FileDaemon {                          # this is me
  Name = eec556ab8f52-fd
  FDport = 9102                  # where we listen for the director
  WorkingDirectory = /opt/bacula/working
  Pid Directory = /var/run
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib64
}

Messages {
  Name = Standard
  director = eec556ab8f52-dir = all, !skipped, !restored
}
```

`bacula-sd -tc /etc/bacula/bacula-sd.conf`验证配置文件

