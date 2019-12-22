---
title: CentOS更换阿里云源
categories:
  - CentOS
tags:
  - CentOS
mp3: 'https://link.hhtjim.com/163/1407551413.mp3'
cover: 'https://cn.bing.com/th?id=OHR.SeventeenSolstice_EN-US6457012478_1920x1080.jpg'
date: 2019-12-22 11:45:11
---
### 备份原源
```
yum -y install wget
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

### 下载阿里源文件
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 更新
```
yum makecache
```