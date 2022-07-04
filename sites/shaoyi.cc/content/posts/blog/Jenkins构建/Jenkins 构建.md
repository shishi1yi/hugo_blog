---
title: "Jenkins部署"
date: 2022-07-04T00:18:23+08:00
lastmod: 2022-07-04T00:18:23+08:00
author: ["shaoyi"]
keywords: 
- Jenkins
categories: 
- Jenkins
tags: 
- Jenkins
description: "Jenkins部署前后端"
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
    image: "/img/jenkins.jpg" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
---



## Jenkins构建

### 前端构建

#### 创建一个【自由风格】的任务

![image-20220627160945298](/img/image-20220627160945298.png)





---



#### 点击【源码管理】

- 填写（选择）项目仓库的地址和用户

- 指定要构建的版本，即项目仓库的远程版本号

![image-20220627161022407](/img/image-20220627161022407.png)





---



#### 选择nodejs版本

> 如果没有选项，说明当前jenkins 没有安装

![image-20220627161413782](/img/image-20220627161413782.png)



---



#### 填写构建的命令

> 这个命令取决于具体工程，和本地第一次执行命令基本一致

> 例如：node -v && npm install -g yarn@1.22.19 && yarn -v && yarn install --pure-lockfile && yarn run build && cd dist && tar -zcvf dist.tar.gz *

> 上述例子意思大致为，获取项目依赖，并打包构建，然后把打好的包压缩

![image-20220627161545170](/img/image-20220627161545170.png)



---



#### 传输打包的文件到指定服务器

【SSH Server】：选择要传输文件的服务器

**Transfers**

- 【Source files】：写你要传输的文件路径
- 【Remove prefix】：要去掉的前缀，不写远程服务器的目录结构将和Source files写的一致
- 【Remote directory】：写你要部署在远程服务器的那个目录地址下，不写就是SSH Servers配置里默认远程目录
- 【Exec command】：传输完了要执行的命令，图中例子是 进入目录,解压缩,解压缩完成后删除压缩包三个命令

![image-20220627162051972](/img/image-20220627162051972.png)



---





### 后端构建

#### 创建maven任务

![image-20220627163119693](/img/image-20220627163119693.png)



---



#### 任务的全局设置

- 指定jdk版本

![image-20220627163242081](/img/image-20220627163242081.png)



---



#### 点击【源码管理】

- 填写（选择）项目仓库的地址和用户

- 指定要构建的版本，即项目仓库的远程版本号

![image-20220627163337340](/img/image-20220627163337340.png)

![image-20220627163408893](/img/image-20220627163408893.png)



---



#### 构建时，指定pom文件

- 跳过测试构建（非必须）

![image-20220627163434235](/img/image-20220627163434235.png)



---



#### 设置只用构建成功的包

![image-20220627163636502](/img/image-20220627163636502.png)



---



#### 传输打包的文件到指定服务器

【SSH Server】：选择要传输文件的服务器

**Transfers**

- 【Source files】：写你要传输的文件路径
- 【Remove prefix】：要去掉的前缀，不写远程服务器的目录结构将和Source files写的一致
- 【Remote directory】：写你要部署在远程服务器的那个目录地址下，不写就是SSH Servers配置里默认远程目录
- 【Exec command】：传输完了要执行的命令



![image-20220627163809466](/img/image-20220627163809466.png)



### Jenkins自动化构建（钩子）

#### 设置触发自动化构建的条件

> 下图框中部分为触发自动构建的条件，本例子触发条件的为【提交事件】

![image-20220701093710780](/img/image-20220701093710780.png)

#### 设置分支过滤

> 本例没有做限制

![image-20220701094211748](/img/image-20220701094211748.png)

#### GitLab设置

- 复制URL

  ![image-20220701094404469](/img/image-20220701094404469.png)

- 点击Generate按钮，token就会自动生成Secret token

![image-20220701094640443](/img/image-20220701094640443.png)

- URL和Secret token，设置在对应的仓库中

![image-20220701094939991](/img/image-20220701094939991.png)



#### 测试



![image-20220701095040964](/img/image-20220701095040964.png)

- GitLab页面提示下图内容，则成功触发Jenkins构建任务

![image-20220701095130638](/img/image-20220701095130638.png)

- Jenkins构建历史已有记录

![image-20220701095311138](/img/image-20220701095311138.png)



## 项目配置私服

- 仓库setting.xml文件配置私服用户密码

![image-20220627164022986](/img/image-20220627164022986.png)



---



> 由于jenkins是docker部署的，会牵扯到一个访问地址

- 如果是本地获取私服包，则需要在项目的根部pom.xml 配置私服地址

`注：id要和setting.xml文件中一致`



![image-20220627164430704](/img/image-20220627164430704.png)



如果是Jenkins构建，则需要把项目的根部pom.xml 改成下图中的ip

![image-20220627164347748](/img/image-20220627164347748.png)