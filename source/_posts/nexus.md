---
title: nexus
categories:
  - nexus
tags:
  - nexus
date: 2020-09-17 13:45:18
mp3: https://link.hhtjim.com/163/5179544.mp3
cover: https://www.bing.com/th?id=OHR.MistyVineyard_ZH-CN7642034150_1920x1080.jpg&rf=LaDigue_1920x1080.jpg
---

## 使用Nexus部署私有仓库

支持如下及更多

- maven
- apt
- bower
- docker
- go
- npm
- pypi
- yum
- ruby

> hosted：本地仓库，通常我们会部署自己的构件到这一类型的仓库。比如公司的第二方库。 
> proxy：代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。 
> group：仓库组，用来合并多个hosted/proxy仓库。
> virtual: 虚拟仓库类型（基本不用）

---

### 安装

```bash
mkdir /nexus-data
docker run -d -p 8081:8081 --restart always --name nexus -v /nexus-data:/nexus-data sonatype/nexus3
docker exec -t nexus cat /nexus-data/admin.password
```

### 建立包存储目录

{% asset_img nexus2.png nexus %}
{% asset_img nexus3.png nexus %}

### 配置maven仓库

1. 存储默认使用default
2. 创建maven2proxy
[http://maven.aliyun.com/nexus/content/groups/public](http://maven.aliyun.com/nexus/content/groups/public)

![nexus](nexus9.png)

3. 在maven-public添加maven-aliyun
![nexus](nexus10.png)

4. 配置 maven-release、maven-snapshots，允许上传Jar包
![nexus](nexus11.png)

5. maven settings文件

> yu_ming-ip换成自己的域名或IP，修改server的密码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <pluginGroups>
    <!-- 
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <proxies>
    <!--
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <servers>
  <server>
  <!--这是server的id（注意不是用户登陆的id），该id与distributionManagement中repository元素的id相匹配。 -->
    <id>releases</id>
    <username>admin</username>
    <password>password</password>
  </server>

  <server>
    <id>snapshots</id>
    <username>admin</username>
    <password>password</password>
  </server>
    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

<mirrors>
  <mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <name>admin</name>
    <url>http://yu_ming-ip:8081/repository/maven-public</url>
  </mirror>
</mirrors>

  <profiles>
  <profile>    
    <!--profile的id--> 
    <id>nexus</id>    
    <repositories>    
      <repository>   
        <id>central</id>    
        <url>http://yu_ming-ip:8081/repository/maven-public</url>    
        <releases>    
          <enabled>true</enabled>    
        </releases>    
        <snapshots>    
          <enabled>true</enabled>    
        </snapshots>    
      </repository>   
      <repository>   
        <id>maven-releases</id>    
        <url>http://yu_ming-ip:8081/repository/maven-releases</url>    
        <releases>    
          <enabled>true</enabled>    
        </releases>    
        <snapshots>    
          <enabled>true</enabled>    
        </snapshots>    
      </repository>
      <repository>   
        <id>maven-snapshots</id>    
        <url>http://yu_ming-ip:8081/repository/maven-snapshots</url>    
        <releases>    
          <enabled>true</enabled>    
        </releases>    
        <snapshots>    
          <enabled>true</enabled>    
        </snapshots>    
      </repository>      
    </repositories>   
    <pluginRepositories>   
     <!-- 插件仓库，maven 的运行依赖插件，也需要从私服下载插件 --> 
      <pluginRepository>   
       <!-- 插件仓库的 id 不允许重复，如果重复后边配置会覆盖前边 --> 
        <id>public</id>   
        <name>Public Repositories</name>   
        <url>http://yu_ming-ip:8081/repository/maven-public</url>   
      </pluginRepository>   
    </pluginRepositories>   
  </profile> 
    <!--
    <profile>
      <id>jdk-1.4</id>

      <activation>
        <jdk>1.4</jdk>
      </activation>

      <repositories>
        <repository>
          <id>jdk14</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://www.myhost.com/maven/jdk14</url>
          <layout>default</layout>
          <snapshotPolicy>always</snapshotPolicy>
        </repository>
      </repositories>
    </profile>
    -->

    <!--
     | <plugin>
     |   <groupId>org.myco.myplugins</groupId>
     |   <artifactId>myplugin</artifactId>
     |
     |   <configuration>
     |     <tomcatLocation>${tomcatPath}</tomcatLocation>
     |   </configuration>
     | </plugin>
     | ...
    <profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>
 
  <activeProfiles>
    <!--要激活的profile id--> 
    <activeProfile>nexus</activeProfile> 
  </activeProfiles>
  <!-- 
  <activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
  -->
</settings>
```

6. 发布项目到nexus的pom.xml配置

在settings.xml中配置权限，其中id要与pom文件中的id一致

```xml
<distributionManagement>
    <repository>
        <id>releases</id>
        <name>Releases</name>
        <url>http://{yu_ming-ip}/repository/maven-releases</url>
    </repository>
    <snapshotRepository>
        <id>snapshot</id>
        <name>Snapshot</name>
        <url>http://{yu_ming-ip}/repository/maven-snapshots</url>
    </snapshotRepository>
</distributionManagement>
```
使用命令发布项目
```bash
mvn clean deploy
```

7. 在pom.xml指定nexus
```xml
<repositories>
    <repository>
        <id>nexus</id>
        <name>nexus</name>
        <url>http://yu_ming-ip:8081/nexus/content/groups/public/</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

8. 在项目的pom.xml文件中指定插件仓库

```xml
<pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>nexus</name>
        <url>http://yu_ming-ip:8081/nexus/content/groups/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>
```

9. 测试
![nexus](nexus12.png)

10. 离线包上传
将jar包和pom文件一起传入即可

### 配置apt仓库

1. 创建ubuntu blob stores
2. 创建仓库选择apt(hosted)
3. 键入名称ubuntu
4. 分支Distribution 输入：focal,bionic,xenial,disco,eoan
![nexus](nexus14.png)
5. signing key： 通过`gpg --export-secret-key --armor` 导出，并输入密码
在任意机器生成命令：`apt install rng-tools && gpg --gen-key` ,一直回车再按提示输入相关信息
![nexus](nexus15.png)

https://mirror.tuna.tsinghua.edu.cn/ubuntu/
![nexus](nexus13.png)

6. Ubuntu的软件源配置文件是 `/etc/apt/sources.list`。将系统自带的该文件做个备份，将该文件替换为下面内容:

```bash
# bionic为1804，按操作系统不同更改
deb http://yu_ming-ip:8081/repository/ubuntuproxy/ bionic main restricted universe multiverse
deb http://yu_ming-ip:8081/repository/ubuntuproxy/ bionic-updates main restricted universe multiverse
deb http://yu_ming-ip:8081/repository/ubuntuproxy/ bionic-backports main restricted universe multiverse
deb http://yu_ming-ip:8081/repository/ubuntuproxy/ bionic-security main restricted universe multiverse
```

7. 测试
![nexus](nexus18.png)
![nexus](nexus19.png)

### 配置docker仓库


### 配置go仓库

1. 创建go blob stores
2. 创建仓库选择go(proxy)
3. 键入名称
4. 输入远程存储地址 [https://mirrors.aliyun.com/goproxy/](https://mirrors.aliyun.com/goproxy/) *勾选Use certificates stored*
5. 选择之前创建的blob stores
6. 创建仓库选择go(group)
7. 键入名称
8. 选择创建的go blob stores
9. 在member选项选择刚刚创建好的proxy

![nexus](nexus16.png)

10. 修改环境变量

```bash
# 1.13以上
go env -w GO111MODULE=on
go env -w GOPROXY=http://yu_ming-ip:8081/repository/gogroup/
# 以下
# windows
set GO111MODULE=on

export GO111MODULE=on
export GOPROXY=http://yu_ming-ip:8081/repository/gogroup/
```

11. 测试
`go get -u github.com/golang/sys`

### 配置npm仓库

1. 创建blob stores
2. 创建仓库选择npm(hosted)
3. 键入名称
4. 选择刚刚创建的blob stores
5. 创建仓库选择npm(proxy)
6. 键入名称
7. 输入远程存储地址 [https://registry.npm.taobao.org](https://registry.npm.taobao.org) *勾选Use certificates stored*
8. 选择之前创建的blob stores
9. 创建仓库选择npm(group)
10. 键入名称
11. 选择刚刚创建的blob stores
12. 在member选项选择刚刚创建好的hosted和proxy

```bash
npm config get registry
npm config set registry http://yu_ming-ip:8081/repository/npmgroup/
```

![nexus](nexus4.png)

### 配置pypi仓库

1. 创建pypi blob stores
2. 创建pypihosted
![nexus](nexus5.png)

3. 创建pypiproxy
[http://mirrors.aliyun.com/pypi](http://mirrors.aliyun.com/pypi)

![nexus](nexus6.png)

4. 创建pypigroup
![nexus](nexus7.png)

5. 测试
![nexus](nexus8.png)

```bash
pip install requests -i http://yu_ming-ip:8081/repository/pypigroup/simple --trusted-host yu_ming-ip
```

### 配置yum仓库

1. 创建yum存储空间
![nexus](https://www.zhengxk.com/upload/2020/09/image-46ae11e3763f4e77a18dfc2a263f7fd6.png)

2. 填写对应信息
![nexus](https://www.zhengxk.com/upload/2020/09/image-4af2dcf6037741b5bdc8bade570290e7.png)

3. 创建yumhosted
{% asset_img nexus1.png nexus %}

![nexus](https://www.zhengxk.com/upload/2020/09/image-9583bf64b80843f6bd8ff430242632dd.png)

![nexus](https://www.zhengxk.com/upload/2020/09/image-8fead1242ceb4257a68e1a2cb79aa41b.png)

4. 创建yumproxy
[https://mirrors.aliyun.com/centos](https://mirrors.aliyun.com/centos)

![nexus](https://www.zhengxk.com/upload/2020/09/image-654625e199744670a9c1a12b59092205.png)

![nexus](https://www.zhengxk.com/upload/2020/09/image-c306b66c7f1a4225829d832f47f90600.png)

5. 创建yumgroup
![nexus](https://www.zhengxk.com/upload/2020/09/image-92a92a11d7a54994a5534a65afba372b.png)

![nexus](https://www.zhengxk.com/upload/2020/09/image-e33cf4a35f604480adf013064e39fd37.png)

6. 在 `etc/yum.repos.d/` 备份 `*.repo` 文件并新建 `nexus.repo` 填如下内容

```repo
[nexus]
name=NexusYum
baseurl=http://yu_ming-ip:8081/repository/yumgroup/$releasever/os/$basearch/
enabled=1
gpgcheck=0
```

```bash
yum clean all
yum makecache
```
### 内网全量同步



