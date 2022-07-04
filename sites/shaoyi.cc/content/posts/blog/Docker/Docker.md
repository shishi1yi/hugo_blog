---
title: "Docker 容器"
date: 2020-07-04T19:18:23+08:00
lastmod: 2022-07-04T19:18:23+08:00
author: ["shaoyi"]
keywords: 
- Docker
categories: 
- Docker
tags: 
- Docker
description: "Docker的基本使用方式"
weight:
slug: ""
draft: false # 是否为草稿
comments: true
reward: true # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "/img/docker.png" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false


---





# Docker安装步骤

## 安装Docker

> 安装

帮助文档：

```shell
# yum安装gcc相关环境（需要确保 虚拟机可以上外网 ）
yum -y install gcc 
yum -y install gcc-c++

#1、卸载旧的Docker版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
#2、需要的安装包
yum install -y yum-utils

#更新yum软件包索引
yum makecache fast 

#3、设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo    #默认是国外的镜像，速度比较慢
 	
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo    #阿里云的镜像地址安装(推荐使用国内的镜像)  
    
#4、安装Docker  docker-ce 社区版 		ee 企业版  
yum install docker-ce docker-ce-cli containerd.io


#5、启用Docker
systemctl start docker

#6、查看是否安装成功
docker version

```

<img src="https://s1.ax1x.com/2020/08/31/dXP25j.png" alt="查看Docker是否安装成功" style="zoom:150%;" />

```shell
#7、hello-world
docker run hello-world
```

<img src="https://s1.ax1x.com/2020/08/31/dXFU1I.png" style="zoom:150%;" />

```shell
#8、查看一下 下载的这个hello-world镜像
[root@shishiyi /]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        8 months ago        13.3kB

```

了解：如何卸载Docker

```shell
#1、卸载docker的依赖
yum remove docker-ce docker-ce-cli containerd.io

#2、删除资源
rm -rf /var/lib/docker

#	/var/lib/docker 	为docker的默认工作路径
```



## 阿里云镜像加速

1、找到阿里云的【容器镜像服务】——>【镜像中心】——>【镜像加速器】

<img src="https://s1.ax1x.com/2020/08/31/dXnypF.png" style="zoom:150%;" />



2、选择对应的系统配置使用

```shell
sudo mkdir -p /etc/docker	#创建目录并在目录下加入镜像配置

sudo tee /etc/docker/daemon.json <<-'EOF'	
{
  "registry-mirrors": ["https://rksuvr7q.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload	#重启镜像

sudo systemctl restart docker	#重启Docker
```



# Docker常用命令

## 帮助命令

```shell
docker version		#显示docker的版本信息
docker info			#显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help   #帮助命令
```

帮助文档链接：https://docs.docker.com/reference/

## 镜像命令

**docker images** 	**查看所有本地的主机上的镜像**

```shell
[root@shishiyi ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        8 months ago        13.3kB

#解释
REPOSITORY  镜像的仓库源（即名字）
TAG			镜像的标签
IMAGE ID	镜像的id
CREATED 	镜像的创建时间
SIZE		镜像的大小

#可选项
  -a, --all             # 列出所有镜像
  -q, --quiet           # 只显示镜像的id

```

**docker search**	**搜索镜像**

```shell
[root@shishiyi ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   9918                [OK]                
mariadb                           MariaDB is a community-developed fork of MyS…   3629                [OK]                

#可选项 搜索过滤
--filter=STARS=3000		#表示搜索出来的镜像为STARS大于3000的
[root@shishiyi ~]# docker search mysql --filter=STARS=3000
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql               MySQL is a widely used, open-source relation…   9918                [OK]                
mariadb             MariaDB is a community-developed fork of MyS…   3629                [OK]   

[root@shishiyi ~]# docker search mysql --filter=STARS=4000
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql               MySQL is a widely used, open-source relation…   9918                [OK]                

```

**docker pull	下载镜像**

```shell
#下载镜像 docker pull 镜像名称:[tag]
[root@shishiyi ~]# docker pull mysql
Using default tag: latest		#如果不写tag,即版本号，默认就是latest
latest: Pulling from library/mysql
bf5952930446: Pull complete 	#分层下载，docker images的核心  联合文件系统
8254623a9871: Pull complete 
938e3e06dac4: Pull complete 
ea28ebf28884: Pull complete 
f3cef38785c2: Pull complete 
894f9792565a: Pull complete 
1d8a57523420: Pull complete 
6c676912929f: Pull complete 
ff39fdb566b4: Pull complete 
fff872988aba: Pull complete 
4d34e365ae68: Pull complete 
7886ee20621e: Pull complete 
Digest: sha256:c358e72e100ab493a0304bda35e6f239db2ec8c9bb836d8a427ac34307d074ed		#签名
Status: Downloaded newer image for mysql:latest	
docker.io/library/mysql:latest		#真实地址

#等价于它
docker pull mysql
docker pull docker.io/library/mysql:latest

#指定版本下载
[root@shishiyi ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
bf5952930446: Already exists 
8254623a9871: Already exists 
938e3e06dac4: Already exists 
ea28ebf28884: Already exists 
f3cef38785c2: Already exists 
894f9792565a: Already exists 
1d8a57523420: Already exists 
5f09bf1d31c1: Pull complete 
1b6ff254abe7: Pull complete 
74310a0bf42d: Pull complete 
d398726627fd: Pull complete 
Digest: sha256:da58f943b94721d46e87d5de208dc07302a8b13e638cd1d24285d222376d6d84
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

<img src="https://s1.ax1x.com/2020/09/01/djvVf0.png" style="zoom:150%;" />

**docker rmi** **删除镜像**

```shell
[root@shishiyi ~]# docker rmi -f 镜像id								 #删除指定镜像
[root@shishiyi ~]# docker rmi -f 镜像id 镜像id 镜像id 镜像id			 #删除多个镜像id	
[root@shishiyi ~]# docker rmi -f $(docker images -aq)				  #删除全部镜像
```



## 容器命令

**有镜像才能创建容器，Linux下载一个centos测试**

```shell
docker pull centos
```

**新建容器并启动**

```shell
docker run [可选参数] image

# 参数说明
--name="Name"		容器名字 ，用来区分容器
-d					后台方式运行
-it					使用交互方式运行，进入容器查看内容
-p(小写)			   指定容器的端口，例如 -p 8080:8080
	-p ip:主机端口：容器端口
	-p 主机端口：容器端口（常用）
	-p 容器端口
	容器端口
-P(大写)     		   随机指定端口

#测试，启动进入容器
[root@shishiyi ~]# docker run -it centos /bin/bash
[root@17e6da9a3c45 /]# ls										#查看容器内的centos,基础版，很多命令不完善的
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
```

**列出所有运行的容器**

```shell
#docker ps 命令  列出当前正在运行的容器
-a				#列出运行过和正在运行的容器
-n=? 			#列出最近创建的容器
-q				#只显示容器的编号
	
[root@shishiyi ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@shishiyi ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
17e6da9a3c45        centos              "/bin/bash"         3 minutes ago       Exited (127) 46 seconds ago                       relaxed_babbage
f8decff51b25        bf756fb1ae65        "/hello"            18 hours ago        Exited (0) 18 hours ago                           boring_curran

```

**退出容器**

```shell
exit # 退出容器并停止
Ctrl + P + Q #退出容器且不停止
```

**删除容器**

```shell
docker rm 容器id				       #删除指定容器，不能删除正在运行的容器，如果强制删除 rm -f
docker rm -f $(docker ps -aq)	    #删除所有容器
docker ps -a -q|xargs docker rm		#删除所有容器
```

**启动和停止容器**

```shell
docker start 容器id		#启动指定容器
docker restart 容器id		#重启指定容器
docker stop 容器id		#停止当前正在运行指定容器
docker kill 容器id		#强制停止当前指定容器
```

## **其它常用命令**

**查看日志**

```shell
#命令
docker logs -f -t --tail 容器id

#[root@shishiyi /]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              
5c97ab1298f8        centos              "/bin/bash"         6 minutes ago       Up 6 minutes                            

#例如
-tf				#显示日志
--tail number	#要是显示的日志条数
[root@shishiyi ~]# docker logs -tf --tail 10 5c97ab1298f8
```

**查看容器中进程信息**

```shell
# 命令 
docker top 容器id

#例如
[root@shishiyi /]# docker top 5c97ab1298f8
UID                 PID                 PPID                C                   STIME               TTY                 TIME           
root                16042               16018               0                   15:27               pts/0               00:00:00           

```

**查看镜像的元数据**

```shell
#命令
docker inspect 容器id

#测试
[root@shishiyi /]# docker inspect 5c97ab1298f8
[
    {
        "Id": "5c97ab1298f8820d95e8cda27ea9156e5c249ad2d90ade5d7a3d064401bf8ecb",
        "Created": "2020-09-01T07:27:36.571377434Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 16042,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-09-01T07:27:36.940479781Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:0d120b6ccaa8c5e149176798b3501d4dd1885f961922497cd0abef155c869566",
        "ResolvConfPath": "/var/lib/docker/containers/5c97ab1298f8820d95e8cda27ea9156e5c249ad2d90ade5d7a3d064401bf8ecb/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/5c97ab1298f8820d95e8cda27ea9156e5c249ad2d90ade5d7a3d064401bf8ecb/hostname",
        "HostsPath": "/var/lib/docker/containers/5c97ab1298f8820d95e8cda27ea9156e5c249ad2d90ade5d7a3d064401bf8ecb/hosts",
        "LogPath": "/var/lib/docker/containers/5c97ab1298f8820d95e8cda27ea9156e5c249ad2d90ade5d7a3d064401bf8ecb/5c97ab1298f8820d95e8cda27ea9156e5c249ad2d90ade5d7a3d064401bf8ecb-json.log",
        "Name": "/nifty_margulis",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/59787dd31a270bf3606b8f7960f91eb54aaf5926b0c940b32bc7488ac4a91edc-init/diff:/var/lib/docker/overlay2/cfd05b0bbde49a9a0c58c89767084e493d0b018ef3c1bc0220a022015610b97a/diff",
                "MergedDir": "/var/lib/docker/overlay2/59787dd31a270bf3606b8f7960f91eb54aaf5926b0c940b32bc7488ac4a91edc/merged",
                "UpperDir": "/var/lib/docker/overlay2/59787dd31a270bf3606b8f7960f91eb54aaf5926b0c940b32bc7488ac4a91edc/diff",
                "WorkDir": "/var/lib/docker/overlay2/59787dd31a270bf3606b8f7960f91eb54aaf5926b0c940b32bc7488ac4a91edc/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "5c97ab1298f8",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20200809",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "24dac8793fce9c49029e6f472af4e01bc23c25af31fbc8d16ce60075ad9b8b33",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/24dac8793fce",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "51415885283d49663fb7ca11668b55ad10d433dbe170e5a0699f05f77bd5b5df",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "927dcbdbe869574ba0ba3e6e0bf4012f1b67fe1f2d1f4846d8da44c96afe0d2f",
                    "EndpointID": "51415885283d49663fb7ca11668b55ad10d433dbe170e5a0699f05f77bd5b5df",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]

```

**进入当前正在运行的容器**

```shell
#命令
docker exec -it 容器id bashShell

#测试
[root@shishiyi /]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS             
5c97ab1298f8        centos              "/bin/bash"         10 minutes ago      Up 10 minutes                           
[root@shishiyi /]# docker exec -it  5c97ab1298f8  /bin/bash
[root@5c97ab1298f8 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var


#方式二
#命令
docker attach 容器id
#测试
[root@shishiyi /]# docker attach 5c97ab1298f8
正在执行的代码.......



#docker exec    是进入容器后重新开启一个终端，可以在容器里操作
#docker attach  是进入容器正在执行的终端，不会启用新的进程！
```

**从容器内拷贝文件到主机上**

```shell
#命令
docker cp 容器id:容器内路径  目的的主机路径
```



# commit镜像

```shell
docker commit	提交容器成为一个新的副本

#命令和git原理类似
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名称:[TAG]
```

实战测试

```shell
#1、启动一个默认的tomcat容器

#2、发现默认tomcat容器是没有webapps应用的， 镜像的原因，官方的镜像默认webapps下面是没有文件的

#3、自己拷贝基本文件到webapps下

#4、然后将上述操作后的tomcat容器提交为一个镜像！之后便可以使用这个新建的镜像创建容器
```

<img src="https://s1.ax1x.com/2020/09/02/wSdnXQ.png"  />



# 容器数据卷

>方式一：直接使用命令来挂载 -v

```shell
docker run -it -v 主机目录:容器内的目录


#启动起来时候我们可以通过docker inspect 容器id   来查看挂载的目录
```

## 具名挂载和匿名挂载

```shell
#匿名挂载 

-v 容器内路径
#-v后面只写了容器内的路径，没有写主机路径，这种就是匿名挂载



#具名挂载

-v 卷名:容器内路径
#-v后面只写了一个名字，而不是具体的主机目录，然后加上容器内路径，这种就是具名挂载

```

所有的docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volumes/xxxx/_data`

我们通过具名挂载可以方便找到我们的一个卷，大多数情况是使用`具名挂载`

```shell
#如何确定是具名挂载还是匿名挂载，还是指定路径挂载
-v 容器内路径			 #匿名挂载
-v 卷名:容器内路径	   		#具名挂载
-v /宿主机路径:容器内路径   #指定路径挂载
```

拓展

```shell
#	通过 -v 容器内路径：ro  rw 改变读写权限
ro	readonly	#只读
rw	readwrite	#可读可写

#	一旦这个设置了容器权限，容器对我们挂载出来的内容就有了限定！
docker run -d -P --name nginx01 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx01 -v juming-nginx:/etc/nginx:rw nginx

#ro 只要看到ro就说明这个路径只能通过宿主机来操作，容器内部是无法操作！
```



# DockerFile构建镜像

## DockerFile指令

```shell
FROM			#	基础镜像，一切从这里开始构建
MAINEAINER		#	镜像是谁写的，姓名+邮箱，譬如 shishiyi<shaoyi.o@qq.com>
RUN				#	镜像构建的时候需要运行的命令
ADD				#	添加文件
WORKDIR			#	镜像的工作目录
VOLUME			#	挂载的目录
EXPOSE 			#	镜像暴露的端口
CMD				#   指定这个容器启动的时候要运行的命令，在用run命令启动容器追加命令会被替换掉
ENTRYPOINT		#	指定这个容器启动的时候要运行的命令，可以在run命令启动容器追加命令
ONBUILD			#	当构建一个被继承 DockerFile 这个时候就会运行 ONBUILD 的指令。触发命令
COPY			#	类似ADD，将我们文件拷贝到镜像中
ENV				#	构建的时候设置环境变量
LABEL			#	设置镜像的标签
```

## 实战测试DockerFile指令

```shell
# 1、编写一个自己的centos的DockerFile文件

[root@shishiyi mydockerfile]# cat mydockerfile-centos 
FROM centos
MAINTAINER shishiyi<shaoyi.0@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash

```

```shell
# 2、通过自己写的dockerfile文件构建镜像
#命令:  docker build -f dockerfile文件路径 -t 镜像名:[tag] .       
#命令结尾一定要加空格和点

[root@shishiyi mydockerfile]# docker build -f mydockerfile-centos -t mycentos:0.1 .
......(中间省略)
Successfully built 09aaf2b838b6
Successfully tagged mycentos:0.1

```

### 对比

**原生的centos**

<img src="https://s1.ax1x.com/2020/09/08/wMMYa8.png" style="zoom: 200%;" />

**通过自己写的DockerFile文件构建镜像的centos**

<img src="https://s1.ax1x.com/2020/09/08/wMQ9df.png" style="zoom: 200%;" />



```shell
# 可以通过命令列出一个镜像是怎么生成的，即dockerfile的命令

#命令： docker history 镜像id
```

<img src="https://s1.ax1x.com/2020/09/08/wMGMRS.png" style="zoom:150%;" />





> CMD和ENTRYPOINT的区别

```shell
CMD 		#这个是指容器启动时要执行的命令，如果在docker run后加了命令则会覆盖CMD指令
ENTRYPOINT  #这个也是容器启动时要执行的命令，使用docker run 命令启动包含ENTRYPOINT的容器，指定会作为参数传递
			#如果确实有需要，也可以在运行容器时通过docker run --entrypoint 覆盖ENTRYPOINT指令
```

> 实战测试两者区别

测试CMD

```shell
#	1、 编写dockerfile文件
[root@shishiyi mydockerfile]# vim mydockerfile-centos-cmdtest
FROM centos
CMD ["ls","-a"]
```

```shell
#	2、构建镜像
[root@shishiyi mydockerfile]# docker build -f mydockerfile-centos-cmdtest -t centos-cmd-test:0.1 .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM centos
 ---> 0d120b6ccaa8
Step 2/2 : CMD ["ls","-a"]
 ---> Running in d7ee5073447a
Removing intermediate container d7ee5073447a
 ---> adc267054d89
Successfully built adc267054d89
Successfully tagged centos-cmd-test:0.1

```

```shell
#	3、启动容器，发现ls -a命令生效了
[root@shishiyi mydockerfile]# docker run centos-cmd-test:0.1
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

#想追加一个命令 -l  变成ls -al
[root@shishiyi mydockerfile]# docker run centos-cmd-test:0.1 -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown.

#CMD情况下  -l却把CMD["ls", "-a"]指令替换了，导致不是命令报错
```

测试ENTRYPOINT指令

```shell
# 编写dockerfile文件
[root@shishiyi mydockerfile]# vim mydockerfile-centos-entrypointtest
FROM centos
ENTRYPOINT ["ls","-a"]

# 构建镜像
[root@shishiyi mydockerfile]# docker build -f mydockerfile-centos-entrypointtest -t centos-entrypoint-test:0.1 .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 0d120b6ccaa8
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in 936e7ec1e16c
Removing intermediate container 936e7ec1e16c
 ---> 68dbcdfd80a6
Successfully built 68dbcdfd80a6
Successfully tagged centos-entrypoint-test:0.1

# 启动容器
[root@shishiyi mydockerfile]# docker run centos-entrypoint-test:0.1
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

#启动容器追加一个命令 -l ,发现直接拼接在了ENTRYPOINT ["ls","-a"]指令后面 
[root@shishiyi mydockerfile]# docker run centos-entrypoint-test:0.1 -l
total 56
drwxr-xr-x  1 root root 4096 Sep  8 04:03 .
drwxr-xr-x  1 root root 4096 Sep  8 04:03 ..
-rwxr-xr-x  1 root root    0 Sep  8 04:03 .dockerenv
lrwxrwxrwx  1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x  5 root root  340 Sep  8 04:03 dev
drwxr-xr-x  1 root root 4096 Sep  8 04:03 etc
drwxr-xr-x  2 root root 4096 May 11  2019 home
lrwxrwxrwx  1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 11  2019 lib64 -> usr/lib64
drwx------  2 root root 4096 Aug  9 21:40 lost+found
drwxr-xr-x  2 root root 4096 May 11  2019 media
drwxr-xr-x  2 root root 4096 May 11  2019 mnt
drwxr-xr-x  2 root root 4096 May 11  2019 opt
dr-xr-xr-x 94 root root    0 Sep  8 04:03 proc
dr-xr-x---  2 root root 4096 Aug  9 21:40 root
drwxr-xr-x 11 root root 4096 Aug  9 21:40 run
lrwxrwxrwx  1 root root    8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 May 11  2019 srv
dr-xr-xr-x 13 root root    0 Sep  8 04:03 sys
drwxrwxrwt  7 root root 4096 Aug  9 21:40 tmp
drwxr-xr-x 12 root root 4096 Aug  9 21:40 usr
drwxr-xr-x 20 root root 4096 Aug  9 21:40 var

```



## 发布镜像

>到DockerHub

在dockerhub官网注册一个账号

dockerhub官网：https://hub.docker.com/

登录dockerhub账号

```shell
[root@shishiyi mydockerfile]# docker login --help

Usage:	docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
  


[root@shishiyi mydockerfile]# docker login -u shishiyi
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

```

登录完毕后就可以提交自己的镜像了

```shell
# 推送镜像的规范是：   docker push 注册用户名/镜像名:tag
# 推送镜像尽量带上版本号

[root@shishiyi mydockerfile]# docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos-entrypoint-test   0.1                 68dbcdfd80a6        3 hours ago         215MB
centos-cmd-test          0.1                 adc267054d89        3 hours ago         215MB
mycentos                 0.1                 09aaf2b838b6        4 hours ago         287MB
centos                   latest              0d120b6ccaa8        4 weeks ago         215MB
[root@shishiyi mydockerfile]# docker push shishiyi/mycentos:0.1
The push refers to repository [docker.io/shishiyi/mycentos]
An image does not exist locally with the tag: shishiyi/mycentos

#发现直接这样推送是不行的， 看错误An image does not exist locally with the tag: shishiyi/mycentos,
#本地没有tag是shishiyi/mycentos的镜像.

#解决办法，增加一个tag
[root@shishiyi mydockerfile]# docker tag mycentos:0.1  shishiyi/mycentos:0.1
[root@shishiyi mydockerfile]# docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
centos-entrypoint-test   0.1                 68dbcdfd80a6        3 hours ago         215MB
centos-cmd-test          0.1                 adc267054d89        4 hours ago         215MB
shishiyi/mycentos        0.1                 09aaf2b838b6        4 hours ago         287MB
mycentos                 0.1                 09aaf2b838b6        4 hours ago         287MB
centos                   latest              0d120b6ccaa8        4 weeks ago         215MB
```

然后再推送即可，记得带上版本号！

<img src="https://s1.ax1x.com/2020/09/08/wQPziq.png" alt="wQPziq.png" style="zoom:200%;" />



# Docker网络（容器互联)



## --link(已过时)

**容器之间是否可以通过名字ping通**

<img src="https://s1.ax1x.com/2020/09/08/wQQmHx.png" alt="wQQmHx.png" style="zoom: 200%;" />

**启动两个容器后，发现直接通过名字并不能ping通**

```shell
#可以使用 --link命令来解决 
[root@shishiyi home]# docker run -d -P --name tomcat03 --link tomcat02 tomcat
d5e88ace3eb8793c00e3441440347c2cbe52784e8771b6882edb47671631eb5f
[root@shishiyi home]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.092 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=3 ttl=64 time=0.068 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=4 ttl=64 time=0.072 ms

#但是不能反向ping通
[root@shishiyi home]# docker exec -it tomcat02 ping tomcat03
ping: tomcat03: Name or service not known

```

其实这个tomcat03之所以能ping通tomcat02，本质是因为在tomcat03本地配置hosts文件

```shell
#查看tomcat03的hosts文件
[root@shishiyi home]# docker exec -it tomcat03  cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat02 639797bb5b9b
172.17.0.4	d5e88ace3eb8

```

**--link命令就是在hosts文件中添加了一行 172.17.0.3	tomcat02 639797bb5b9b**



## 自定义网络

```shell
[root@shishiyi /]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b975ebb49b23        bridge              bridge              local
fd952cfe82a0        host                host                local
50c346e922df        none                null                local
```

**网络模式**

```shell
bridge：桥接docker (默认的，自己创建也是使用bridge 模式)
host ：和宿主机共用网络
none ：不配置网络
```



docker默认使用bridge 模式即docker0，但是docker0 不支持容器之间 通过名字直接访问，所以我们可以选择另一种方式即自定义网络

```shell
#创建自己的网络
#	--driver bridge  选择网络模式(不写默认为bridge)
#	--subnet 192.168.0.0/16    子网的路段 192.168.0.0/16表示从192.168.0.2到192.168.255.255
#	--gateway 192.168.0.1		网关地址即路由

[root@shishiyi /]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynetwork
8ecbbe2766eca4f95f19cc37f731bccaa305b440834abdce4006fe6ef3e5fecb

[root@shishiyi /]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b975ebb49b23        bridge              bridge              local
fd952cfe82a0        host                host                local
8ecbbe2766ec        mynetwork           bridge              local
50c346e922df        none                null                local

```

自己定义的网络就配置好了！

<img src="https://s1.ax1x.com/2020/09/08/wQyrkT.png" alt="wQyrkT.png" style="zoom: 150%;" />

```shell
#	创建容器时指定自己写的网关
[root@shishiyi /]# docker run -d -P --name tomcat-net-01 --net mynetwork tomcat
b39086a191ca5da74eb88fce3bd27ccb5039ea36569ffc4a766b18075ea83cba
[root@shishiyi /]# docker run -d -P --name tomcat-net-02 --net mynetwork tomcat
2c48396a0e18213eb913701d98d494546fa4ca7ce1ce316f04b4e7ac45d2aae7

#	再次查看自己配置的网关
[root@shishiyi /]# docker network inspect mynetwork
[
    {
        "Name": "mynetwork",
        "Id": "8ecbbe2766eca4f95f19cc37f731bccaa305b440834abdce4006fe6ef3e5fecb",
        "Created": "2020-09-08T16:56:59.25870749+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2c48396a0e18213eb913701d98d494546fa4ca7ce1ce316f04b4e7ac45d2aae7": {
                "Name": "tomcat-net-02",
                "EndpointID": "7dcfa2ec9c4dc735aed6625791989ea78e8598d0a27d4540d5ef4ef9e0bfe84c",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "b39086a191ca5da74eb88fce3bd27ccb5039ea36569ffc4a766b18075ea83cba": {
                "Name": "tomcat-net-01",
                "EndpointID": "b988b92f99d56c415e7550c94179cd095ddd75ccb8bd7fb11454d213be6bbcab",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

#	再次通过ip去ping连接
[root@shishiyi /]# docker exec -it tomcat-net-01 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.073 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.062 ms
64 bytes from 192.168.0.3: icmp_seq=3 ttl=64 time=0.076 ms

#	使用了自己写的网关，我们发现可以直接通过名字ping通
[root@shishiyi /]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynetwork (192.168.0.3): icmp_seq=1 ttl=64 time=0.060 ms
64 bytes from tomcat-net-02.mynetwork (192.168.0.3): icmp_seq=2 ttl=64 time=0.065 ms
^C
--- tomcat-net-02 ping statistics ---

```

结论：我们自定义的网络docker已经帮我们维护好了对应的关系，所以推荐使用自定义网络的方式

好处：

不同的集群使用不同的网络，保证了集群的安全和健康



## 网络连通

> 怎么办到不同的网关下的容器之间相互连通

<img src="https://s1.ax1x.com/2020/09/08/wQRTlF.png" alt="wQRTlF.png" style="zoom:150%;" />

<img src="https://s1.ax1x.com/2020/09/08/wQWmp8.png" alt="wQWmp8.png" style="zoom:150%;" />

```shell
#	测试打通docker0网络下的tomcat01 
#	连通之后就是将tomcat01放到了mynetwork网络下
# 	具体命令为：docker network connect mynetwork tomcat01

#	相当于一个容器有了两个ip地址

#	inspect命令查看自定义网络配置，发现确定添加了其它网络下的容器进来
```

<img src="https://s1.ax1x.com/2020/09/08/wQfVb9.png" alt="wQfVb9.png" style="zoom:150%;" />



