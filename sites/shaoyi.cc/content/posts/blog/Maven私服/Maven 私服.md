---
title: "Maven 私服"
date: 2022-07-04T19:18:23+08:00
lastmod: 2022-07-04T19:18:23+08:00
author: ["shaoyi"]
keywords: 
- Maven
categories: 
- Maven
tags: 
- Maven
description: "项目中关于Maven 私服配置"
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
    image: "/img/nexus.png" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

---







## 项目打包到Maven 私服 

### 配置Maven

> 配置私服仓库的用户和密码

- 在maven安装目录 /conf/setting.xml 中的servers下添加

```xml
<servers>
     <server>
       <id>nexus-snapshots</id>
       <username>admin</username>
       <password>123456</password>
     </server>
     <server>
       <id>nexus-releases</id>
       <username>admin</username>
       <password>123456</password>
     </server>
</servers>
```



### 在需要打包的maven模块中，配置pom文件内容

在maven工程的pom.xml配置部署的仓库，**注意pom.xml和setting.xml中的id属性要一致**

```xml
<!-- 发布maven私服 -->
<distributionManagement>
      <repository>
          <id>nexus-snapshots</id>
          <name>shishiyi-framework-SNAPSHOTS</name>
          <url>http://127.0.0.1:30010/repository/3rd-part/</url>  // 私服仓库对应的url
      </repository>
      <snapshotRepository>
          <id>nexus-repository</id>
          <name>shishiyi-framework-REPOSITORY</name>
          <url>http://127.0.0.1:30010/repository/3rd-part/</url>
      </snapshotRepository>
</distributionManagement>
```



### 使用deploy命令上传

> mvn source:jar deploy -Dmaven.test.skip=true





## 项目获取私服仓库中的包

在项目根部的pom文件中添加下列内容

```xml
<!-- 远程nexus仓库 -->
    <repositories>
        <repository>
            <id>nexus-snapshots</id>
            <url>http://127.0.0.1:30010/repository/3rd-part/</url>
        </repository>
        <repository>
            <id>nexus-repository</id>
            <url>http://127.0.0.1:30010/repository/3rd-part/</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>nexus-snapshots</id>
            <url>http://127.0.0.1:30010/repository/3rd-part/</url>
        </pluginRepository>
        <pluginRepository>
            <id>nexus-repository</id>
            <url>http://127.0.0.1:30010/repository/3rd-part/</url>
        </pluginRepository>
    </pluginRepositories>
```

