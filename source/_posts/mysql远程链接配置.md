---
title: mysql远程链接配置
categories:
  - mysql
tags:
  - mysql
date: 2019-12-23 11:00:57
mp3:
cover: https://www.bing.com/th?id=OHR.AiringGrievances_ZH-CN5830208720_1920x1080.jpg&rf=LaDigue_1920x1080.jpg

---

### 描述

docker需要链接数据库，发现设置127.0.0.1无法链接

### 修改配置文件

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
将`bind-address           = 127.0.0.1`注释掉

重启mysql服务
`sudo /etc/init.d/mysql restart`

**检查防火墙**

修改用户信息

`grant all privileges on *.* to '用户名'@'%' identified by '密码'with grant option;`

`flush privileges;`