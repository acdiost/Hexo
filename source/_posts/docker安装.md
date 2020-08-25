---
title: docker、nodejs安装
categories:
  - docker
  - nodejs
tags:
  - docker
  - nodejs
  - npm
date: 2020-01-18 19:54:36
mp3: https://link.hhtjim.com/163/27925517.mp3
cover: https://www.bing.com/th?id=OHR.HighlandsSquirrel_ZH-CN1369975915_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---

## docker

1. 安装docker，任选其一，完成后将当前用户加入到docker组，便不需使用sudo命令了
```bash
# 官方
curl -k -ssl https://get.docker.com | sudo sh
# 阿里云
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
# daocloud
curl -sSL https://get.daocloud.io/docker | sh

# 当前用户加入到docker组
sudo usermod -aG docker ${USER}
```

2. 镜像下载加速，新建修改daemon将源换成国内的
`vi /etc/docker/daemon.json`
```json
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

5. File-->settings-->build,execution,deployment-->docker-->TCP socket填入:`tcp://127.0.0.1:2375`

## nodejs

```bash
curl -sL https://rpm.nodesource.com/setup_lts.x | sudo bash -

yum install -y nodejs


alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"
```