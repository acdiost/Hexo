---
title: mysql远程链接配置
categories:
  - mysql
tags:
  - mysql
date: 2019-12-23 11:00:57
mp3: https://link.hhtjim.com/163/26092806.mp3
cover: https://www.bing.com/th?id=OHR.AiringGrievances_ZH-CN5830208720_1920x1080.jpg&rf=LaDigue_1920x1080.jpg

---

### 描述

docker需要链接数据库，发现设置127.0.0.1无法链接

### 安装
```
sudo apt-get install mysql-server
# python相关依赖
sudo apt-get install libmysqlclient-dev python3-dev
sudo mysql_secure_installation
```

### 修改配置文件

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
将`bind-address           = 127.0.0.1`注释掉
`skip-external-locking`后面新增一行字符集编码`character-set-server=utf8`

重启mysql服务
`sudo /etc/init.d/mysql restart`

**检查防火墙**

修改用户信息
```
create user 'dawn'@'%' identified by '密码';
grant all privileges on *.* to 'dawn'@'%' identified by '密码'with grant option;
# mysql8.0 授权更新为不需要认证密码
grant all privileges on *.* to 'dawn'@'%' with grant option;
flush privileges;
```