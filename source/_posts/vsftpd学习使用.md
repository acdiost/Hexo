---
title: vsftpd学习使用
categories:
  - CentOS
tags:
  - vstfpd
  - ftp
date: 2020-02-28 10:09:59
mp3: https://link.hhtjim.com/163/535648576.mp3
cover: https://www.bing.com//th?id=OHR.OtterCreekVT_ZH-CN0564511657_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---

# vsftpd 3.0.2使用实操

## 概述：

完整详细配置在测试之后。认真看完实操中内容方有收获！

安装使用：

系统：CentOS7, vsftpd版本：3.0.2

**先把防火墙和SELinux关闭**

```bash
yum install vsftpd ftp -y
systemctl start vsftpd
systemctl enable vsftpd
```

修改配置后都要重启服务。

## 工作模式：

主动模式：默认

​	客户端随机端口告诉服务器端

被动模式：Passive

​	服务器端随机端口告诉客户端

```bash
# 启用并修改被动模式数据传输端口
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=35000
```



​	**需要防火墙放行vsftpd的端口**

## 登录方式：

### 匿名账户登录：

​	账户：ftp、anonymous

​	默认登录目录：/var/ftp/

#### 匿名控制权限anon：

```bash
anonymous_enable=YES # 启用匿名访问
anon_umask=022 # 匿名用户上传文件权限掩码
anon_root=/var/ftp # 匿名用户登录后的根目录
anon_upload_enable=YES # 允许匿名用户上传文件，不关闭selinux则报：553 Could not create file.
anon_mkdir_write_enable=YES # 允许匿名用户创建目录
anon_other_write_enable=YES # 允许匿名用户删除、覆盖、重命名的权限
anon_max_rate=0 # 限制传输速率，0为不限制 单位:byte
```

#### 测试匿名用户：

##### 匿名用户登录：

默认安装好启用就可以使用账户名为anonymous和ftp，无密码。默认可以浏览和下载

##### 匿名用户上传文件：

##### 测试实操：

```bash
# 配置文件启用
vim /etc/vsftpd/vsftpd.conf
# 29行 anon_upload_enable=YES
cd /var/ftp
mkdir upload
# 授权写入
chmod o+w upload
# 客户端操作
    [dawn@192 ~]$ ftp 192.168.1.21
    Connected to 192.168.1.21 (192.168.1.21).
    220 (vsFTPd 3.0.2)
    Name (192.168.1.21:dawn): ftp
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,146,249).
    150 Here comes the directory listing.
    drwxr-xr-x    2 0        0              22 Feb 27 13:08 pub
    -rw-r--r--    1 0        0               0 Feb 27 11:47 test
    drwxr-xrwx    2 0        0              49 Feb 27 13:00 upload
    226 Directory send OK.
    ftp> cd upload
    250 Directory successfully changed.
    ftp> put ftp
    local: ftp remote: ftp
    227 Entering Passive Mode (192,168,1,21,162,132).
    150 Ok to send data.
    226 Transfer complete.
# 此时无法下载上传的文件
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,109,30).
    150 Here comes the directory listing.
    -rw-------    1 14       50              0 Feb 27 13:13 ftp
    226 Directory send OK.
    ftp> get ftp
    local: ftp remote: ftp
    227 Entering Passive Mode (192,168,1,21,147,169).
    550 Failed to open file.
# 配置文件新增umask=022并重启
    ftp> put ftp
    local: ftp remote: ftp
    227 Entering Passive Mode (192,168,1,21,47,132).
    150 Ok to send data.
    226 Transfer complete.
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,228,115).
    150 Here comes the directory listing.
    -rw-r--r--    1 14       50              0 Feb 27 13:32 ftp
    226 Directory send OK.
    ftp> get ftp
    local: ftp remote: ftp
    227 Entering Passive Mode (192,168,1,21,96,4).
    150 Opening BINARY mode data connection for ftp (0 bytes).
    226 Transfer complete.
# 允许创建目录、删除、修改、覆盖文件
    # 默认已有
    anon_mkdir_write_enable=YES
    # 新增：
    anon_other_write_enable=YES
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,75,172).
    150 Here comes the directory listing.
    -rw-r--r--    1 14       50              0 Feb 27 13:32 ftp
    226 Directory send OK.
    ftp> rename ftp test
    350 Ready for RNTO.
    250 Rename successful.
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,167,16).
    150 Here comes the directory listing.
    -rw-r--r--    1 14       50              0 Feb 27 13:32 test
    226 Directory send OK.
    ftp> delete test
    250 Delete operation successful.
```

#### 进入目录时提示信息：

在目录下新建`.message`文件，在其中写入要提示的信息即可。配置文件默认开启

39行 `dirmessage_enable=YES`

##### 完整配置：

```bash
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
anon_upload_enable=YES
anon_umask=022
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```



### 本地登录：

​	账户：/etc/passwd  /etc/shadow

​	默认登录目录：用户家目录

#### 本地用户权限local：

```bash
local_enable=YES # 是否启用本地用户
local_umask=022 # 本地用户上传文件后文件权限掩码
local_root=/var/ftp # 本地用户登录后的ftp根目录
local_max_rate=0 # 本地用户限制传输速率
chroot_local_user=YES # 是否将用户禁在主目录中
ftpd_banner=# 登录后显示的欢迎语
# 类似白名单和黑名单
userlist_enable=YES & userlist_deny=YES # 禁止/etc/vsftpd/user_list中出现的用户登录
userlist_enable=YES & userlist_deny=NO # 仅允许/etc/vsftpd/user_list中出现的用户登录
# 配置文件ftpusers： 文件中出现的用户都不能登录，权限比user_list更高。不需要重启服务，立刻生效。
```

#### 测试本地用户：

##### 配置文件：

```bash
# 将用户限制在家目录中
chroot_local_user=YES
# 允许chroot_list中的用户可以随意切换目录
chroot_list_enable=YES
# 名单位置，需手动创建
chroot_list_file=/etc/vsftpd/chroot_list
```



##### 创建用户登录：

**`useradd -s /sbin/nologin` 创建的用户无法登陆时:**

 ftp 会根据/etc/shells 这个文件判断一个用户是否为有效用户，会阻止那些shell不在/etc/shells里的用户登陆。默认是没有`/sbin/nologin`的，需要新增

```bash
echo '/sbin/nologin' >> /etc/shells
    [root@localhost vsftpd]# cat /etc/shells
    /bin/sh
    /bin/bash
    /usr/bin/sh
    /usr/bin/bash
    /sbin/nologin

```

##### 测试实操：

```bash
# 创建两个用户：
    [root@localhost upload]# useradd -s /sbin/nologin test
    [root@localhost upload]# passwd test
    Changing password for user test.
    New password:123
    BAD PASSWORD: The password is shorter than 8 characters
    Retype new password:123
    passwd: all authentication tokens updated successfully.
    [root@localhost upload]# useradd -s /sbin/nologin testban
    [root@localhost upload]# passwd testban
    Changing password for user testban.
    New password:123
    BAD PASSWORD: The password is shorter than 8 characters
    Retype new password:123
    passwd: all authentication tokens updated successfully.
# 测试登录test
    [dawn@192 ~]$ ftp 192.168.1.21
    Connected to 192.168.1.21 (192.168.1.21).
    220 Welcome
    Name (192.168.1.21:dawn): test
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> ls -a
    227 Entering Passive Mode (192,168,1,21,126,36).
    150 Here comes the directory listing.
    drwx------    2 1001     1001           62 Feb 27 13:58 .
    drwx------    2 1001     1001           62 Feb 27 13:58 ..
    -rw-r--r--    1 1001     1001           18 Aug 08  2019 .bash_logout
    -rw-r--r--    1 1001     1001          193 Aug 08  2019 .bash_profile
    -rw-r--r--    1 1001     1001          231 Aug 08  2019 .bashrc
    226 Directory send OK.
# 禁止testban登录
    [root@localhost vsftpd]# cat user_list
    # vsftpd userlist
    # If userlist_deny=NO, only allow users in this file
    # If userlist_deny=YES (default), never allow users in this file, and
    # do not even prompt for a password.
    # Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
    # for users that are denied.
    root
    bin
    daemon
    adm
    lp
    sync
    shutdown
    halt
    mail
    news
    uucp
    operator
    games
    nobody
    testban
# 测试登录
    [dawn@192 ~]$ ftp 192.168.1.21
    Connected to 192.168.1.21 (192.168.1.21).
    220 Welcome
    Name (192.168.1.21:dawn): test
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> quit
    221 Goodbye.
    [dawn@192 ~]$ ftp 192.168.1.21
    Connected to 192.168.1.21 (192.168.1.21).
    220 Welcome
    Name (192.168.1.21:dawn): testban
    530 Permission denied.
    Login failed.
# 测试改变家目录
    [root@localhost vsftpd]# cat chroot_list
    test

    # test 可以改变
    [dawn@192 ~]$ ftp 192.168.1.21
    Connected to 192.168.1.21 (192.168.1.21).
    220 Welcome
    Name (192.168.1.21:dawn): test
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> cd /
    250 Directory successfully changed.
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,121,195).
    150 Here comes the directory listing.
    lrwxrwxrwx    1 0        0               7 Oct 17 12:19 bin -> usr/bin
    dr-xr-xr-x    5 0        0            4096 Oct 17 12:34 boot
    drwxr-xr-x   20 0        0            3200 Feb 27 12:57 dev
    drwxr-xr-x   78 0        0            8192 Feb 27 14:38 etc
    drwxr-xr-x    5 0        0              49 Feb 27 14:02 home
    lrwxrwxrwx    1 0        0               7 Oct 17 12:19 lib -> usr/lib
    lrwxrwxrwx    1 0        0               9 Oct 17 12:19 lib64 -> usr/lib64
    drwxr-xr-x    2 0        0               6 Apr 11  2018 media
    drwxr-xr-x    2 0        0               6 Apr 11  2018 mnt
    drwxr-xr-x    3 0        0              20 Dec 24 15:12 opt
    dr-xr-xr-x  137 0        0               0 Feb 27 12:57 proc
    dr-xr-x---   14 0        0            4096 Feb 27 15:04 root
    drwxr-xr-x   26 0        0             780 Feb 27 12:57 run
    lrwxrwxrwx    1 0        0               8 Oct 17 12:19 sbin -> usr/sbin
    drwxr-xr-x    2 0        0               6 Apr 11  2018 srv
    dr-xr-xr-x   13 0        0               0 Feb 27 12:57 sys
    drwxrwxrwt    8 0        0             172 Feb 27 12:57 tmp
    drwxr-xr-x   13 0        0             155 Oct 17 12:19 usr
    drwxr-xr-x   20 0        0             278 Feb 27 11:33 var
    226 Directory send OK.
    # 取消禁止登录后，testban 不可以改变
    [dawn@192 ~]$ ftp 192.168.1.21
    Connected to 192.168.1.21 (192.168.1.21).
    220 Welcome
    Name (192.168.1.21:dawn): testban
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> cd /
    250 Directory successfully changed.
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,124,173).
    150 Here comes the directory listing.
    226 Directory send OK.
```



##### 完整配置：

创建chroot_list文件，写入需要改变家目录的用户：`touch /etc/vsftpd/chroot_list`

```bash
anonymous_enable=NO
local_enable=YES
write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
tcp_wrappers=YES

local_umask=022
ftpd_banner=Welcome
chroot_local_user=YES
# 500 OOPS: vsftpd: refusing to run with writable root inside chroot()。
#从2.3.5之后，vsftpd增强了安全检查，如果用户被限定在了其主目录下，则该用户的主目录不能再具有写权限了！如果检查发现还有写权限，就会报该错误。
# 可以用命令chmod a-w /home/user去除用户主目录的写权限，或者配置文件新增allow_writeable_chroot=YES
allow_writeable_chroot=YES
# 启用本地用户可切出家目录。可以下载根目录下有r权限的文件
chroot_list_enable=YES
# chroot_list中的用户可以切出家目录
chroot_list_file=/etc/vsftpd/chroot_list
# 启用user_list中用户不可登录
userlist_enable=YES
# 黑名单模式user_list中用户不可登录，为NO则只有user_list中用户可登录
userlist_deny=YES
```



### 虚拟账户：

​	手动创建，找一个系统用户作为虚拟账户的映射用户，借助系统用户家目录为默认登录目录。每个权限可单独定制

##### 创建虚拟用户数据库文件

奇数行为用户，偶数行为密码

`vi /etc/vsftpd/vsftpd.user`

```
[root@localhost vsftpd]# cat vsftpd.user
upload
123
createdir
123
superuser
123
```

   

##### 加密配置文件

`db_load -T -t hash -f vsftpd.user vsftpd.db`

修改数据库文件权限

`chmod 600 vsftpd.db`

##### 创建系统用户作为虚拟账户映射用户,无需设密码

1. 创建的用户无法登陆时见上文原因
2. `useradd -d /ftp/ -s /sbin/nologin ftpuser`
3. 修改ftpuser的家目录的读权限`chmod o+r /ftp/`

##### 创建支持虚拟用户的pam认证文件

1. `cp /etc/pam.d/vsftpd /etc/pam.d/vsftpd.pam`

2. ```bash
   # 清空内容添加如下内容：
   auth required pam_userdb.so db=/etc/vsftpd/vsftpd
   # 数据库vsftpd名称为自定义名称
   account required pam_userdb.so db=/etc/vsftpd/vsftpd
   ```

##### 编辑配置文件/etc/vsftpd/vsftpd.conf，修改并新增如下配置

```bash
# 认证文件/etc/pam.d/下
pam_service_name=vsftpd.pam
# 启用虚拟账户登录
guest_enable=YES
# 系统映射账户名称
guest_username=ftpuser
# 虚拟账户配置文件目录路径，其中每个文件名称和用户名对应
user_config_dir=/etc/vsftpd/users
```

修改自定义的匿名配置文件，因为虚拟用户用的也是该配置

##### 编辑单个虚拟用户的权限

```bash
# 允许虚拟用户上传
echo 'anon_upload_enable=YES' > /etc/vsftpd/users/upload
# 允许虚拟用户创建目录
echo 'anon_mkdir_write_enable=YES' > /etc/vsftpd/users/createdir
# 允许虚拟用户上传和其他权限
echo 'anon_upload_enable=YES' > /etc/vsftpd/users/superuser
echo 'anon_other_write_enable=YES' >> /etc/vsftpd/users/superuser
# 指定虚拟用户访问目录
local_root=/ftp/user/
```

##### 测试实操：

```bash
# 上传文件用户upload
    Name (192.168.1.21:dawn): upload
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> ls
    227 Entering Passive Mode (192,168,1,21,133,188).
    150 Here comes the directory listing.
    226 Directory send OK.
    ftp> put ftp
    local: ftp remote: ftp
    227 Entering Passive Mode (192,168,1,21,159,66).
    150 Ok to send data.
    226 Transfer complete.
    # 不可以创建目录
    ftp> mkdir qwe
    550 Permission denied.

# 创建目录createdir，不可传文件
    [dawn@192 ~]$ ftp 192.168.1.21
    Connected to 192.168.1.21 (192.168.1.21).
    220 Welcome
    Name (192.168.1.21:dawn): createdir
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> mkdir abc
    257 "/abc" created
    ftp> put ftp
    local: ftp remote: ftp
    227 Entering Passive Mode (192,168,1,21,162,112).
    550 Permission denied.
 # 上传和修改等权限
    ftp> put test
    local: test remote: test
    227 Entering Passive Mode (192,168,1,21,84,149).
    150 Ok to send data.

    ftp> rename ftp vsftpd
    350 Ready for RNTO.
    250 Rename successful.

    ftp> delete vsftpd
    250 Delete operation successful.

```



##### 完整配置：

```bash
anonymous_enable=NO
local_enable=YES
write_enable=YES
anon_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd.pam
guest_enable=YES
guest_username=ftpuser
user_config_dir=/etc/vsftpd/users

userlist_enable=YES
tcp_wrappers=YES

ftpd_banner=Welcome
allow_writeable_chroot=YES
```

#### 虚拟配置加本地用户配置：

```bash
anonymous_enable=NO
local_enable=YES
write_enable=YES
anon_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd.pam
guest_enable=YES
guest_username=ftpuser
user_config_dir=/etc/vsftpd/users

userlist_enable=YES
tcp_wrappers=YES

local_umask=022
ftpd_banner=Welcome
chroot_local_user=YES
allow_writeable_chroot=YES
chroot_list_enable=YES
userlist_deny=YES
chroot_list_file=/etc/vsftpd/chroot_list
```



## 使用openssl加密传输

1. 生成密钥和证书

```bash
cd /etc/ssl/certs/
# 生成密钥
openssl genrsa -out vsftpd.key 2048
# 生成证书
openssl req -new -key vsftpd.key -out vsftpd.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:sc
Locality Name (eg, city) [Default City]:cd
Organization Name (eg, company) [Default Company Ltd]:acdiost
Organizational Unit Name (eg, section) []:acdiost
Common Name (eg, your name or your server's hostname) []:acdiost.com
Email Address []:your@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:不设置，调用证书需解密。
An optional company name []:不设置
# 生成证书
openssl x509 -req -days 365 -sha256 -in vsftpd.csr -signkey vsftpd.key -out vsftpd.crt
# 证书文件权限修改
chmod -R 500 vsftpd.*

```

​      

2. 修改配置文件

```bash
# 启用ssl
ssl_enable=YES
# 开启tls、ssl支持
ssl_tlsv1=YES
ssl_sslv2=YES
ssl_sslv3=YES
# 允许匿名和虚拟用户使用ssl
allow_anon_ssl=YES
# 强制匿名和虚拟用户登录传输使用ssl
force_anon_logins_ssl=YES
force_anon_data_ssl=YES
# 强制本地用户登录传输使用ssl
force_local_logins_ssl=YES
force_local_data_ssl=YES
# rsa证书位置
rsa_cert_file=/etc/ssl/certs/vsftpd.crt
# rsa密钥位置
rsa_private_key_file=/etc/ssl/certs/vsftpd.key
```

3. 重启服务

### 配置加密后的问题：

- windows上显示ftp文件夹错误

- Linux上提示：530 Non-anonymous sessions must use encryption.

Linux命令行版本和Windows资源管理器不支持加密访问，需要使用支持加密的客户端进行访问如：**FileZilla**



如需命令行版本或Windows资源管理器访问`ssl_enable=NO`即可