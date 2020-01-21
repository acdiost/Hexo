---
title: docker安装
categories:
  - docker
tags:
  - docker
date: 2020-01-18 19:54:36
mp3: https://link.hhtjim.com/163/27925517.mp3
cover: https://www.bing.com/th?id=OHR.HighlandsSquirrel_ZH-CN1369975915_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---


1. 安装docker，国内安装较慢。完成后将当前用户加入到docker组，便不需使用sudo命令了
`curl -k -ssl https://get.docker.com | sudo sh`  
`sudo usermod -aG docker ${USER}`

2. 新建修改daemon将源换成国内的
`vi /etc/docker/daemon.json`
```
{
  "registry-mirrors": ["https://你的id.mirror.aliyuncs.com"],
  "registry-mirrors": ["https://registry.docker-cn.com"],
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

3. 将2375端口开启，让idea、goland和pycharm可以访问使用。方式不止这一种

`vi /lib/systemd/system/docker.service`
```
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock
```

4. 重启docker
```bash
systemctl daemon-reload
systemctl restart docker
```
