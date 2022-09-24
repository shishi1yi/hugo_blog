---
title: "消息中间件之RabbitMQ"
date: 2020-07-04T19:18:23+08:00
lastmod: 2022-08-06T16:24:23+08:00
author: ["十十乙"]
keywords: 
- RabbitMQ
categories: 
- RabbitMQ
tags: 
- RabbitMQ
description: "RabbitMQ使用方式"
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
    image: "" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false




---



# Docker 安装RabbitMQ

>这里注意获取镜像的时候要获取management版本的，不要获取last版本的，management版本的才带有管理界面。

**查询镜像**

```shell
#搜索RabbitMQ镜像
[root@shishiyi /]# docker search rabbitmq:management

```

得到下列结果

![0eAeED.png](https://s1.ax1x.com/2020/09/29/0eAeED.png)

**下载镜像**

```shell
#下载RabbitMQ镜像
[root@shishiyi /]# docker pull rabbitmq:management

```

出现如下结果

![0eAIr6.png](https://s1.ax1x.com/2020/09/29/0eAIr6.png)

**运行镜像**

```shell
#运行RabbitMQ镜像
[root@shishiyi /]# docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management

```

**访问管理界面**

地址为   服务器ip:15672  用户名和密码都是guest

![0eVwn0.png](https://s1.ax1x.com/2020/09/29/0eVwn0.png)

# Java方式实现RabbitMQ

## 1、helloword



**生产者**

```java


import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;


public class Provider {

    public static void main(String[] args) throws IOException, TimeoutException {
        //创建mq的连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //设置连接的服务器IP
        connectionFactory.setHost("服务器ip");
        //设置端口号
        connectionFactory.setPort(5672);
        //设置连接的虚拟主机（类似数据库中的库）
        connectionFactory.setVirtualHost("/ems");
        //设置用户名
        connectionFactory.setUsername("ems");
        //设置密码
        connectionFactory.setPassword("123");
        //获取连接
        Connection connection = connectionFactory.newConnection();
        //获取连接的通道
        Channel channel = connection.createChannel();
        //通道绑定消息队列
        //参数1：队列名称，如果不存在则自动创建
        //参数2：是否持久队列
        //参数3：是否独占队列
        //参数4： 是否消费完毕删除队列
        //参数5：额外附加参数
        channel.queueDeclare("hello",false,false,false,null);
        //发布消息
        //参数1：交换机；参数2：队列名称；参数3：传递消息的额外设置；参数4：消息内容
        channel.basicPublish("","hello",null,"hello rabbitmq".getBytes());
        //关闭资源
        channel.close();
        connection.close();

    }
}


```

**消费者**

```java


import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Consumer {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("服务器ip");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare("hello",false,false,false,null);
        //参数1：队列名称，参数2：开始消息的自动确认机制，参数3：消费时的回调接口
        channel.basicConsume("hello",true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("消费的内容："+new String(body));
            }
        });

    }
}

```

## 提取公共部分，创建工具类

**RabbitMQUtils**

```java


import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class RabbitMQUtils {
    private static ConnectionFactory connectionFactory = null;
    static {
        connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("服务器ip");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123");
    }
	//获取连接
    public static Connection getConnection(){
        try {
            Connection connection = connectionFactory.newConnection();
            return connection;
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        return null;
    }
	//关闭资源
    public static void clone(Channel channel , Connection connection){
        try {
            if (null !=channel)
                channel.close();
            if (null != connection)
                connection.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
    }
}

```

## 2、Work queues

**生产者**

```java

package com.shishiyi.work;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.shishiyi.utils.RabbitMQUtils;

import java.io.IOException;

public class Provider {
    public static void main(String[] args) throws IOException {
        Connection connection = RabbitMQUtils.getConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare("work",false,false,false,null);
        for (int i = 0; i < 10 ; i++) {
            channel.basicPublish("","work",null,"work queue".getBytes());
        }
        RabbitMQUtils.clone(channel,connection);
    }
}
```

**消费者1**

```java

    Connection connection = RabbitMQUtils.getConnection();
    Channel channel = connection.createChannel();
    channel.queueDeclare("work",false,false,false,null);
    channel.basicConsume("work",true,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("work 消费者1："+new String(body));
        }
    });

```

**消费者2**

```java

    Connection connection = RabbitMQUtils.getConnection();
    Channel channel = connection.createChannel();
    channel.basicConsume("work",true,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("work 消费者2："+new String(body));
        }
    });

```

## 3、fanout

**生产者**

```java
 
//将通道声明为交换机
//参数1：交换机名称；参数2：交换机类型  fanout为广播类型
channel.exchangeDeclare("logs","fanout");
//发送消息
channel.basicPublish("logs","",null,"fanout type message".getBytes());
RabbitMQUtils.clone(channel,connection);

```

**消费者一**

```java

//通道声明为交换机
channel.exchangeDeclare("logs","fanout");
//临时的队列
String queue = channel.queueDeclare().getQueue();
//通道绑定临时队列和交换机
channel.queueBind(queue,"logs","");
channel.basicConsume(queue,true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("fanout1====>"+new String(body));
    }
});
```

**消费者二**

```java

//通道声明为交换机
channel.exchangeDeclare("logs","fanout");
//临时队列
String queue = channel.queueDeclare().getQueue();
//通道绑定临时队列和交换机
channel.queueBind(queue,"logs","");
channel.basicConsume(queue,true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("fanout2====>"+new String(body));
    }
});
```

## 4、direct

**生产者**

```java

channel.exchangeDeclare("direct_logs","direct");
String routingKey = "info";
channel.basicPublish("direct_logs",routingKey,null,("direct发送了一条routingKey为"+routingKey+"的消息").getBytes());

```

**消费者一**

```java

    channel.exchangeDeclare("direct_logs","direct");
    String queue = channel.queueDeclare().getQueue();
    channel.queueBind(queue,"direct_logs","error");
    channel.basicConsume(queue,true,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("direct消费者1: "+new String(body));
        }
    });

```

**消费者二**

```java

    channel.exchangeDeclare("direct_logs","direct");
    String queue = channel.queueDeclare().getQueue();
    channel.queueBind(queue,"direct_logs","info");
    channel.queueBind(queue,"direct_logs","error");
    channel.basicConsume(queue,true,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("direct消费者2："+new String(body));
        }
    });
```

## 5、topic

- *符号可以代替一个单词。
- ＃符号可以替代零个或多个单词。

**生产者**

```java

channel.exchangeDeclare("topic_logs","topic");
String routingKey = "user.find.save";
channel.basicPublish("topic_logs",routingKey,null,("发送了一条routingKey为"+routingKey+"的消息").getBytes());

```

**消费者一**

```java

    channel.exchangeDeclare("topic_logs","topic");
    String queue = channel.queueDeclare().getQueue();
    String routingKey = "user.*";
    channel.queueBind(queue,"topic_logs",routingKey);
    channel.basicConsume(queue,true,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("topic消费者1："+new String(body));
        }
    });
```

**消费者二**

```java

    channel.exchangeDeclare("topic_logs","topic");
    String queue = channel.queueDeclare().getQueue();
    String routingKey = "user.#";
    channel.queueBind(queue,"topic_logs",routingKey);
    channel.basicConsume(queue,true,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("topic消费者2："+new String(body));
        }
    });
```





# Springboot实现方式

## 消费者方绑定队列和交换机的方式

>## 环境

```xml
<!--创建Springboot工程，导入Maven依赖-——>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

编写application.yml文件

```yml

spring:
  application:
    name: rabbitmq_springboot
  rabbitmq:
    host: 服务器ip
    port: 5672
    username: ems #用户名 
    password: 123 #密码
    virtual-host: /ems	#虚拟机名称
```

### 1、helloword

**生产者**

```java

@Autowired
private RabbitTemplate rabbitTemplate;

@Test
public void test() {
    rabbitTemplate.convertAndSend("hello","hello world");
}
```

**消费者**

```java

import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queuesToDeclare = @Queue("hello"))
public class Consumer_hello {
    @RabbitHandler
    public void receive(String message){
        System.out.println("消费的内容："+message);
    }
}
```



### 2、Work queues

**生产者**

```java

@Autowired
private RabbitTemplate rabbitTemplate;
@Test
public void work(){
    for (int i = 0; i < 10; i++) {
        rabbitTemplate.convertAndSend("work"," work "+i);
    }
}

```

**消费者**

```java

import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class Consumer_Work {

    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void receive1(String messages){
        System.out.println("work消费者1:"+messages);
    }
    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void receive2(String messages){
        System.out.println("work消费者2:"+messages);
    }
}
```



### 3、fanout

**生产者**

```java

@Autowired
private RabbitTemplate rabbitTemplate;
@Test
public void fanout(){
    rabbitTemplate.convertAndSend("fanout_logs","","fanout发的消息");
}
```



**消费者**

```java

import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class Consumer_Fanout {
    @RabbitListener(bindings = {
            @QueueBinding(
                value = @Queue, 	//创建临时队列
                exchange = @Exchange(value = "logs_fanout",type = "fanout") 	//自定义交换机名称和类型，并绑定交换机
    )})
    public void receive1(String messages){
        System.out.println("fanout消费者1:"+messages);
    }

    @RabbitListener(bindings = {
            @QueueBinding(
                value = @Queue,     //创建临时队列
                exchange = @Exchange(value = "logs_fanout",type = "fanout")     //自定义交换机名称和类型，并绑定交换机
    )})
    public void receive2(String messages){
        System.out.println("fanout消费者2:"+messages);
    }
}
```





### 4、direct

**生产者**

```java
@Autowired
private RabbitTemplate rabbitTemplate;
@Test
public void route(){
    String routingKey = "info";
    rabbitTemplate.convertAndSend("route_logs",routingKey,"info的消息");
}
```

**消费者**

```java

import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class Consumer_Route {
    @RabbitListener(
            bindings = @QueueBinding(
                    value = @Queue,		//创建临时队列
                    exchange = @Exchange(value = "route_logs",type = "direct"),		//自定义交换机名称和类型，并绑定交换机
                    key = {"error"}		//设置路由
            )
    )
    public void receive1(String messages){
        System.out.println("Route消费者1: "+messages);
    }

    @RabbitListener(
            bindings = @QueueBinding(
                    value = @Queue,		//创建临时队列
                    exchange = @Exchange(value = "route_logs",type = "direct"),		//自定义交换机名称和类型，并绑定交换机
                    key = {"info","error"}		//设置路由
            )
    )
    public void receive2(String messages){
        System.out.println("Route消费者2: "+messages);
    }
}
```





### 5、topic

**生产者**

```java

@Autowired
private RabbitTemplate rabbitTemplate;
@Test
public void topic(){
    String routingKey = "user.find";
    rabbitTemplate.convertAndSend("topic_logs",routingKey,"topic发送的消息");
}
```

**消费者**

```java

import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class Consumer_Topic {
    @RabbitListener(
            bindings = {
                    @QueueBinding(
                            value = @Queue,		//创建临时队列
                            exchange = @Exchange(value = "topic_logs",type = "topic"),		//自定义交换机名称和类型，并绑定交换机
                            key = {"user.#"}
                    )
            }
    )
    public void receive1(String messages){
        System.out.println("topic消费者1: "+messages);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,		//创建临时队列
            exchange = @Exchange(value = "topic_logs",type = "topic"),		//自定义交换机名称和类型，并绑定交换机
            key = {"user.*"}
    ))
    public void receive2(String messages){
        System.out.println("topic消费者2: "+messages);
    }
}
```



## 生产者方绑定队列和交换机的方式

### 1、fanout

> ### 生产者方

在pom.xml文件导入Maven依赖

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

配置application.yml文件

```yml

server:
  port: 8001
spring:
  application:
    name: rabbitmq_springboot_fanout_provider
  rabbitmq:
    host: 服务器ip
    port: 5672 #rabbitmq端口号
    username: shishiyi	#用户名
    password: 123	#密码
    virtual-host: /shishiyi	#虚拟机名称
```

编写配置类

```java


import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConfigRabbitmqFanout {
    //创建三个队列分别为fanoutQueue1，fanoutQueue2，fanoutQueue3
    @Bean
    public Queue fanoutQueue1(){
        return QueueBuilder.durable("fanoutQueue1").build();
    }
    @Bean
    public Queue fanoutQueue2(){
        return QueueBuilder.durable("fanoutQueue2").build();
    }
    @Bean
    public Queue fanoutQueue3(){
        return QueueBuilder.durable("fanoutQueue3").build();
    }
    //创建fanout类型的交换机
    @Bean
    public FanoutExchange fanoutExchange(){
        return ExchangeBuilder.fanoutExchange("fanoutExchange").build();
    }

    //把三个队列分别和交换机进行绑定
    @Bean
    public Binding bindingExchange1(){
        return BindingBuilder.bind(fanoutQueue1()).to(fanoutExchange());
    }
    @Bean
    public Binding bindingExchange2(){
        return BindingBuilder.bind(fanoutQueue2()).to(fanoutExchange());
    }
    @Bean
    public Binding bindingExchange3(){
        return BindingBuilder.bind(fanoutQueue3()).to(fanoutExchange());
    }

}

```

编写生产者的接口，发送消息

```java

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FanoutController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendFanout")
    public void sendMessages(){
        rabbitTemplate.convertAndSend("fanoutExchange","","fanout类型发送的消息");
    }
}
```

启动项目，调用接口

![0MNvSe.png](https://s1.ax1x.com/2020/10/01/0MNvSe.png)

检查rabbitMq管理页，消息已经发送成功

![0QWwo8.png](https://s1.ax1x.com/2020/10/02/0QWwo8.png)

> ### 消费者方

在pom.xml文件导入Maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

配置application.yml文件

```yml
server:
  port: 7001
spring:
  application:
    name: rabbitmq_springboot_fanout_consumer
  rabbitmq:
    host: 服务器ip
    port: 5672 #rabbitmq端口号
    username: shishiyi	#用户名
    password: 123	#密码
    virtual-host: /shishiyi	#虚拟机名称
```

编写监听类

```java

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class FanoutReceiver {
    //监听对应的队列名称
    @RabbitListener(queues = "fanoutQueue1")
    public void fanout1(Message message){
        System.out.println("fanoutQueue1 消费者接受的消息："+message);
    }
    @RabbitListener(queues = "fanoutQueue2")
    public void fanout2(Message message){
        System.out.println("fanoutQueue2 消费者接受的消息："+message);
    }
    @RabbitListener(queues = "fanoutQueue3")
    public void fanout3(Message message){
        System.out.println("fanoutQueue3 消费者接受的消息："+message);
    }
}

```

启动项目，查看控制台

![0QWSrq.png](https://s1.ax1x.com/2020/10/02/0QWSrq.png)



### 2、direct

> ### 生产者方

pom.xml文件

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

application.yml文件

```yml

server:
  port: 8002
spring:
  application:
    name: rabbitmq_springboot_direct_provider
  rabbitmq:
    host: 服务ip
    port: 5672
    username: shishiyi
    password: 123
    virtual-host: /shishiyi
```

编写配置类

```java

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConfigRabbitmqDirect {
    @Bean
    public Queue directQueue1(){
        return QueueBuilder.durable("directQueue1").build();
    }
    @Bean
    public Queue directQueue2(){
        return QueueBuilder.durable("directQueue2").build();
    }
    @Bean
    public DirectExchange directExchange(){
        return ExchangeBuilder.directExchange("directExchange").build();
    }
    //绑定对应的队列，以及路由key
    @Bean
    public Binding bindingExchangeQueue1(){
        return BindingBuilder.bind(directQueue1()).to(directExchange()).with("info");
    }
    @Bean
    public Binding bindingExchangeQueue2(){
        return BindingBuilder.bind(directQueue2()).to(directExchange()).with("error");
    }
}

```

编写生产者的接口，发送消息

```java

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DirectController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/sendDirect")
    public void sendMessages(){	//路由key值为info
        rabbitTemplate.convertAndSend("directExchange","info","这是direct类型,路由为info的消息");
    }

}
```

启动项目，调用接口

![0M0T5q.png](https://s1.ax1x.com/2020/10/01/0M0T5q.png)

查看rabbitmq控制台，由于发送消息方法是往路由key值为info队列发送，所以只有路由key为info的directQueue1队列可以接收到，而路由key值为error的directQueue2队列接受不到消息

![0QfPOI.png](https://s1.ax1x.com/2020/10/02/0QfPOI.png)

> ### 消费者方

pom.xml文件

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

application.yml文件

```yml

server:
  port: 7002
spring:
  application:
    name: rabbitmq_springboot_direct_consumer
  rabbitmq:
    host: 服务器ip
    port: 5672 #rabbitmq端口号
    username: shishiyi	#用户名
    password: 123	#密码
    virtual-host: /shishiyi	#虚拟机名称
```

编写监听类

```java

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class DirectReceiver {
    @RabbitListener(queues = "directQueue1")
    public void directQueue1(Message message){
        System.out.println("directQueue1 消费者接收到的消息："+message);
    }
    @RabbitListener(queues = "directQueue2")
    public void directQueue2(Message message){
        System.out.println("directQueue2 消费者接收到的消息："+message);
    }
}

```

启动项目，查看控制台，由于只有directQueue1里有消息，所以消费者只能看到这个队列消息

![0Qf6hD.png](https://s1.ax1x.com/2020/10/02/0Qf6hD.png)



### 3、topic

>### 生产者

编写pom.xml文件，导入Maven依赖

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

配置文件application.yml

```yml

server:
  port: 8003
spring:
  application:
    name: rabbitmq_springboot_topic_provider
  rabbitmq:
    host: 服务器ip
    port: 5672
    username: shishiyi	#用户名
    password: 123		#密码
    virtual-host: /shishiyi		#虚拟机名称
```

编写配置类

```java


import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConfigRabbitmqTopic {

    @Bean
    public Queue topicQueue1(){
        return QueueBuilder.durable("topicQueue1").build();
    }
    @Bean
    public Queue topicQueue2(){
        return QueueBuilder.durable("topicQueue2").build();
    }
    @Bean
    public Queue topicQueue3(){
        return QueueBuilder.durable("topicQueue3").build();
    }
    @Bean
    public TopicExchange topicExchange(){
        return ExchangeBuilder.topicExchange("topicExchange").build();
    }
    //绑定队列和交换机，以及绑定了不同的路由key值
    @Bean
    public Binding bindingExchangeQueue1(){
        return BindingBuilder.bind(topicQueue1()).to(topicExchange()).with("user.#");
    }
    @Bean
    public Binding bindingExchangeQueue2(){
        return BindingBuilder.bind(topicQueue2()).to(topicExchange()).with("user.save");
    }
    @Bean
    public Binding bindingExchangeQueue3(){
        return BindingBuilder.bind(topicQueue2()).to(topicExchange()).with("user");
    }

}

```

编写接口

```java

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicController {
    @Autowired
    public RabbitTemplate rabbitTemplate;
    @GetMapping("/sendTopic")
    public void sendMessage(){  //路由key为user.save
        rabbitTemplate.convertAndSend("topicExchange","user.save","这是topic类型，路由为user.save的消息");
    }
}
```

启动项目，调用接口

![0QgpqK.png](https://s1.ax1x.com/2020/10/02/0QgpqK.png)

查看rabbitmq控制台，由于发送消息的方法 路由key值为user.save，所以只有路由key为user.#的topicQueue1的队列和路由key值为user.save的topicQueue2队列可以收到消息，而路由key值为user的topicQueue3队列则收不到消息

![0Qfq3Q.png](https://s1.ax1x.com/2020/10/02/0Qfq3Q.png)



>### 消费者方

pom.xml文件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

application.yml文件

```yml

server:
  port: 7003
spring:
  application:
    name: rabbitmq_springboot_topic_consumer
  rabbitmq:
    host: 服务器ip
    port: 5672
    username: shishiyi
    password: 123
    virtual-host: /shishiyi

```

编写监听类，获取消息

```java

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class TopicReceiver {
    @RabbitListener(queues = "topicQueue1")
    public void topicQueue1(Message message){
        System.out.println("topicQueue1 消费者接收到的消息："+message);
    }
    @RabbitListener(queues = "topicQueue2")
    public void topicQueue2(Message message){
        System.out.println("topicQueue2 消费者接收到的消息："+message);
    }
    @RabbitListener(queues = "topicQueue3")
    public void topicQueue3(Message message){
        System.out.println("topicQueue3 消费者接收到的消息："+message);
    }
}
```

启动项目，在控制台查看打印信息，由于只有队列topicQueue1，topicQueue2里有消息，所以消费者只能看到这两个队列消息

![0Q2q39.png](https://s1.ax1x.com/2020/10/02/0Q2q39.png)

# RabbitMQ的高级特性

## 生产者方消息确认机制

在使用RabbitMQ的时候，作为消息发送方希望杜绝任何消息丢失或者投递失败场景。RabbitMQ 为我们提供了两种方式用来控制消息的投递可靠性模式。

- confirm 确认模式
- return 退回模式

RabbitMQ大体流程路径：

![0lGgvd.png](https://s1.ax1x.com/2020/10/02/0lGgvd.png)

- 消息从消费者(Producer) 到 交换机(exchange) 则会调用一个回调方法setConfirmCallback()

  ​	方法参数： **correlationData 相关信息、ack 是否发送成功、cause 失败原因**

- 消息从交换机(exchange)到 队列(queue) 则会调用一个回调方法setReturnCallback()

  ​	方法参数： **message消息内容、 replyCode回应码、replyText回应消息、exchange交换机、routingKey路由key值**

编写配置文件application.yml

```yml

server:
  port: 8004
spring:
  application:
    name: rabbitmq_springboot_provider
  rabbitmq:
    host: 服务器ip
    port: 5672
    username: shishiyi
    password: 123
    virtual-host: /shishiyi

    #消息确认配置项
    #确认消息已发送到交换机(Exchange)
    publisher-confirm-type: correlated
    #确认消息已发送到队列(Queue)
    publisher-returns: true
```



编写配置类

```java

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitmqConfig {
    @Bean
    public RabbitTemplate createRabbitTemplate(ConnectionFactory connectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate();
        rabbitTemplate.setConnectionFactory(connectionFactory);
        /**
         * 从消费者(Producer) 到 交换机(exchange) 则会被调用
         */
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            System.out.println("ConfirmCallback 相关数据: "+correlationData);
            System.out.println("ConfirmCallback 确认情况: "+ack);
            System.out.println("ConfirmCallback 原因: "+cause);
        });

        //设置开启Mandatory,才能触发回调函数,无论消息推送结果怎么样都强制调用回调函数
        rabbitTemplate.setMandatory(true);
        /**
         * 从交换机(exchange)到 队列(queue) 则会被调用
         */
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            System.out.println("ReturnCallback 消息内容: "+message);
            System.out.println("ReturnCallback 回应码: "+replyCode);
            System.out.println("ReturnCallback 回应消息: "+replyText);
            System.out.println("ReturnCallback 交换机: "+exchange);
            System.out.println("ReturnCallback 路由key值: "+routingKey);
        });
        return rabbitTemplate;
    }

    @Bean
    public Queue queue(){
        return QueueBuilder.durable("topic_queue").build();
    }
    @Bean
    public TopicExchange topicExchange(){
        return ExchangeBuilder.topicExchange("topic_exchange").build();
    }
    @Bean
    public Binding binding(){
        return BindingBuilder.bind(queue()).to(topicExchange()).with("yes.#");
    }
}

```

编写接口

```java

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/send")
    public void sendMessage(){
        rabbitTemplate.convertAndSend("topic_exchange","yes","发布了一条消息");
    }

}

```

启动测试，查看控制台信息

![0lZ1AA.png](https://s1.ax1x.com/2020/10/02/0lZ1AA.png)

根据RabbitMQ大体流程路径，从生产者角度，做如下测试：

1、如果发送的交换机没有进行配置，即不存在

```java
//交换机为exchange1111，没有进行配置
@GetMapping("/send")
public void sendMessage(){
    rabbitTemplate.convertAndSend("exchange1111","yes","发布了一条消息");
}
```

启动测试，调用接口，查看控制台

```java

ConfirmCallback 相关数据: null
ConfirmCallback 确认情况: false
ConfirmCallback 原因: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'exchange1111' in vhost '/shishiyi', class-id=60, method-id=40)

```

2、如果交换机配置了，但是没有绑定队列

```java
//配置交换机，但不绑定队列
@Bean
public TopicExchange exchange2(){
    return ExchangeBuilder.topicExchange("topic_exchange_test").build();
}
```

```java
//给没有绑定的交换机发送消息
@GetMapping("/send")
public void sendMessage(){
    rabbitTemplate.convertAndSend("topic_exchange_test","yes","发布了一条消息");
}
```

启动测试，调用接口，查看控制台

```java

ReturnCallback 消息内容: (Body:'发布了一条消息' MessageProperties [headers={}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0])
ReturnCallback 回应码: 312
ReturnCallback 回应消息: NO_ROUTE
ReturnCallback 交换机: topic_exchange_test
ReturnCallback 路由key值: yes
ConfirmCallback 相关数据: null
ConfirmCallback 确认情况: true
ConfirmCallback 原因: null
```

## 消费者方消息确认机制

这里是把消息的自动确认机制，改为手动确认消息，即把 **AcknowledgeMode.NONE** 改为  **AcknowledgeMode.MANUAL** 

配置文件application.yml

```yml

server:
  port: 7004
spring:
  application:
    name: rabbitmq_springboot_consumer
  rabbitmq:
    host: 服务器ip
    port: 5672
    username: shishiyi
    password: 123
    virtual-host: /shishiyi
```

编写开启手动确认，以及自定义监听类

```java

import com.shishiyi.listener.AckListener;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MessageListenerConfig {
    @Autowired
    private AckListener ackListener;
    @Autowired
    private ConnectionFactory connectionFactory;
    @Bean
    public SimpleMessageListenerContainer simpleMessageListenerContainer(){
        SimpleMessageListenerContainer simpleMessageListenerContainer = new SimpleMessageListenerContainer(connectionFactory);
        //消费者改为手动确认，默认为自动确认
        simpleMessageListenerContainer.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        //设置队列，参数可以设置多个队列名称
        simpleMessageListenerContainer.setQueueNames("topic_queue");
        //设置自定义的监听类
        simpleMessageListenerContainer.setMessageListener(ackListener);
        return simpleMessageListenerContainer;
    }
}
```

```java


import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.listener.api.ChannelAwareMessageListener;
import org.springframework.stereotype.Component;

import java.io.IOException;
@Component
public class AckListener implements ChannelAwareMessageListener {
    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            //判断队列，可以针对不同队列，来执行不同的业务内容
            if ("topic_queue".equals(message.getMessageProperties().getConsumerQueue())){
                //转换消息
                System.out.println(new String(message.getBody()));
                //处理业务
                System.out.println("处理业务.........");
            }
            //手动确认，签收消息
            channel.basicAck(deliveryTag,true);
        } catch (IOException e) {
            /**
             *  针对多个消息
             *  第二个参数是否应用于多消息
             *  第三个参数requeue：是否重回队列，如果是true,则重新回到队列，然后重新发给消费者，否者则丢弃或者进入死信队列
             */
            channel.basicNack(deliveryTag,true,false);
            //针对单个消息，第二个参数requeue：是否重回队列
            //channel.basicReject(deliveryTag,true);
        }

    }
}

```

如果执行业务出错，则可以拒绝消息，对此方法有**channel.basicReject(deliveryTag, true)**  和  **channel.basicNack(deliveryTag, false, true)**  两种方式处理

**channel.basicReject(deliveryTag, true)：**   拒绝消费当前消息，如果第二参数传入true，就是将数据重新丢回队列里，那么下次还会消费这消息。设置false，

就是告诉服务器，我已经知道这条消息数据了，因为一些原因拒绝它，而且服务器也把这个消息丢掉就行。 下次不想再消费这条消息了。使用拒绝后重新入列这

个确认模式要谨慎，因为一般都是出现异常的时候，catch异常再拒绝入列，选择是否重入列。但是如果使用不当会导致一些每次都被你重入列的消息一直消费-入

列-消费-入列这样循环，会导致消息积压。

**channel.basicNack(deliveryTag, false, true)：**  第一个参数依然是当前消息到的数据的唯一id；

第二个参数是指是否针对多条消息；如果是true，也就是说一次性针对当前通道的消息的tagID小于当前这条消息的，都拒绝确认。

第三个参数是指是否重新入列，也就是指不确认的消息是否重新丢回到队列里面去。

同样使用不确认后重新入列这个确认模式要谨慎，因为这里也可能因为考虑不周出现消息一直被重新丢回去的情况，导致积压。



## 消费端限流

为防止大量消息全部给到消费端系统，可以对消费端做一个限流操作

开启方式的前提一定是消费端改为手动确认信息，否则就不成功

```java

SimpleMessageListenerContainer simpleMessageListenerContainer = new SimpleMessageListenerContainer(connectionFactory);
//消费者改为手动确认，默认为自动确认
simpleMessageListenerContainer.setAcknowledgeMode(AcknowledgeMode.MANUAL);
//对消费端进行一个限流设置，一次允许多少消息进入
simpleMessageListenerContainer.setPrefetchCount(1);


```

## TTL：消息过期时间

### 1、队列内消息统一过期

```java

@Bean
public Queue queue(){	//只需要在配置队列时，设置一下ttl(int ttl)方法，参数为多少毫秒后消息过期
    return QueueBuilder.durable("topic_queue").ttl(5000).build();
}
```

![03gCtO.png](https://s1.ax1x.com/2020/10/03/03gCtO.png)



### 2、消息单独过期

**单独为某条消息设置过期属性，可以通过发布消息的convertAndSend()重载方法中，MessagePostProcessor消息发布处理器来设置过期属性**

![03gO58.png](https://s1.ax1x.com/2020/10/03/03gO58.png)

![0328PO.png](https://s1.ax1x.com/2020/10/03/0328PO.png)

**通过源码可以发现，MessagePostProcessor是一个函数式接口，而接口的方法有一个Message类型的参数，以此便可以设置消息属性了**

![032Xe1.png](https://s1.ax1x.com/2020/10/03/032Xe1.png)

```java
//发布消息时，单独设置一下消息过期时间
public void sendMessage(){
    rabbitTemplate.convertAndSend("topic_exchange","yes","发布了一条消息",message -> {
        message.getMessageProperties().setExpiration("5000");
        return message;
    });
}
```

**不过，需要注意的是，当队列中有多条消息，有过期时间的消息不会在达到时间后从队列中自动消失，而是只有消息在队列顶端时，才会判断其否过期了**

**而如果同时设置队列过期时间和消息过期时间，则以时间短的为准**

## 死信队列

Rabbitmq的死信队列，其实指的死信交换机(DLX)，但虽然名称是这么一个叫法，但是死信交换机也是一个正常的交换机，和一般的交换机是没有区别的

而进入死信交换机的消息，大致有三种情况：

### 1、消息TTL过期

生产方的配置类

```java


import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitmqConfig {
    
    //声明正常队列
    @Bean
    public Queue queue(){
        return QueueBuilder.durable("topic_queue")
                //正常队列和死信交换机绑定
                .deadLetterExchange("topic_exchange_dlx")
                //和死信交换机的路由key绑定，发给哪个死信队列
                .deadLetterRoutingKey("no.msg")
                //设置队列过期时间，3秒后过期
                .ttl(3000)
                .build();
    }
    //声明正常交换机
    @Bean
    public TopicExchange exchange(){
        return ExchangeBuilder.topicExchange("topic_exchange").build();
    }
    //正常的队列和交换机进行绑定
    @Bean
    public Binding binding(){
        return BindingBuilder.bind(queue()).to(exchange()).with("yes.#");
    }


    //声明死信队列
    @Bean
    public Queue queue_dlx(){
        return QueueBuilder.durable("topic_queue_dlx").build();
    }
    //声明死信交换机
    @Bean
    public TopicExchange exchange_dlx(){
        return ExchangeBuilder.topicExchange("topic_exchange_dlx").build();
    }
    //死信的队列和交换机进行绑定
    @Bean
    public Binding binding_dlx(){
        return BindingBuilder.bind(queue_dlx()).to(exchange_dlx()).with("no.#");
    }

}


```

编写接口

```java

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/send")
    public void sendMessage(){
        rabbitTemplate.convertAndSend("topic_exchange","yes.msg","发布了一条消息");
    }

}
```



启动测试，调用接口发送消息，不消费消息，3秒后查看rabbitmq管理页面，消息已进入死信队列

![03TblV.png](https://s1.ax1x.com/2020/10/03/03TblV.png)



### 2、队列达到最大长度

生产方的配置类

```java

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitmqConfig {
    
    //声明正常队列
    @Bean
    public Queue queue(){
        return QueueBuilder.durable("topic_queue")
                //正常队列和死信交换机绑定
                .deadLetterExchange("topic_exchange_dlx")
                //和死信交换机的路由key绑定，发给哪个死信队列
                .deadLetterRoutingKey("no.msg")
                //指定队列最大接受长度(最大接受消息数)
                .maxLength(5)
                .build();
    }
    //声明正常交换机
    @Bean
    public TopicExchange exchange(){
        return ExchangeBuilder.topicExchange("topic_exchange").build();
    }
    //正常的队列和交换机进行绑定
    @Bean
    public Binding binding(){
        return BindingBuilder.bind(queue()).to(exchange()).with("yes.#");
    }


    //声明死信队列
    @Bean
    public Queue queue_dlx(){
        return QueueBuilder.durable("topic_queue_dlx").build();
    }
    //声明死信交换机
    @Bean
    public TopicExchange exchange_dlx(){
        return ExchangeBuilder.topicExchange("topic_exchange_dlx").build();
    }
    //死信的队列和交换机进行绑定
    @Bean
    public Binding binding_dlx(){
        return BindingBuilder.bind(queue_dlx()).to(exchange_dlx()).with("no.#");
    }

}


```

编写接口

```java
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/send")
    public void sendMessage(){
        //发送10条信息
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend("topic_exchange","yes.msg","发布了一条消息");
        }
    }

}
```

启动测试，调用接口发送消息，不消费消息，查看rabbitmq管理页面，由于设置队列最大长度为5，所以有5条消息进入死信队列

![03qmtJ.png](https://s1.ax1x.com/2020/10/03/03qmtJ.png)

### 3、消息被拒绝

编写生产方配置类

```java

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitmqConfig {
    
    //声明正常队列
    @Bean
    public Queue queue(){
        return QueueBuilder.durable("topic_queue")
                //正常队列和死信交换机绑定
                .deadLetterExchange("topic_exchange_dlx")
                //和死信交换机的路由key绑定，发给哪个死信队列
                .deadLetterRoutingKey("no.msg")
                .build();
    }
    //声明正常交换机
    @Bean
    public TopicExchange exchange(){
        return ExchangeBuilder.topicExchange("topic_exchange").build();
    }
    //正常的队列和交换机进行绑定
    @Bean
    public Binding binding(){
        return BindingBuilder.bind(queue()).to(exchange()).with("yes.#");
    }


    //声明死信队列
    @Bean
    public Queue queue_dlx(){
        return QueueBuilder.durable("topic_queue_dlx").build();
    }
    //声明死信交换机
    @Bean
    public TopicExchange exchange_dlx(){
        return ExchangeBuilder.topicExchange("topic_exchange_dlx").build();
    }
    //死信的队列和交换机进行绑定
    @Bean
    public Binding binding_dlx(){
        return BindingBuilder.bind(queue_dlx()).to(exchange_dlx()).with("no.#");
    }

}


```

编写接口

```java

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/send")
    public void sendMessage(){
        rabbitTemplate.convertAndSend("topic_exchange","yes.msg","发布了一条消息");
    }
}

```

编写 开启手动确认，以及自定义监听类

```java

import com.shishiyi.listener.AckListener;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MessageListenerConfig {
    @Autowired
    private AckListener ackListener;
    @Autowired
    private ConnectionFactory connectionFactory;
    @Bean
    public SimpleMessageListenerContainer simpleMessageListenerContainer(){
        SimpleMessageListenerContainer simpleMessageListenerContainer = new SimpleMessageListenerContainer(connectionFactory);
        //消费者改为手动确认，默认为自动确认
        simpleMessageListenerContainer.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        //设置队列，参数可以设置多个队列名称
        simpleMessageListenerContainer.setQueueNames("topic_queue");
        //设置自定义的监听类
        simpleMessageListenerContainer.setMessageListener(ackListener);
        return simpleMessageListenerContainer;
    }
}
```

执行业务出错，消息被拒绝(basic.reject/ basic.nack）并且requeue=false

```java

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.listener.api.ChannelAwareMessageListener;
import org.springframework.stereotype.Component;

@Component
public class AckListener implements ChannelAwareMessageListener {
    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            //判断队列，可以针对不同队列，来执行不同的业务内容
            if ("topic_queue".equals(message.getMessageProperties().getConsumerQueue())){
                //转换消息
                System.out.println(new String(message.getBody()));
                //处理业务
                System.out.println("处理业务.........");
                //除零出错，模拟执行业务出差错
                int i = 1/0;
            }
            //手动确认，签收消息
            channel.basicAck(deliveryTag,true);
        } catch (Exception e) {
            System.out.println("出现异常，拒绝签收消息");
            //拒绝签收消息，不重回队列requeue=false
            channel.basicNack(deliveryTag,true,false);
        }

    }
}
```

启动测试，调用接口发送消息，查看rabbitmq管理页面，由于消费者方处理业务失败产生错误，拒绝签收消息，且不重回队列，则消息进入死信队列

![03XSte.png](https://s1.ax1x.com/2020/10/03/03XSte.png)



## 延迟队列

RabbitMQ很遗憾不支持原生的延迟队列，但是我们 **可以根据TTL过期时间和死信队列组合实现延迟队列** 功能

生产方的配置类

```java

package com.shishiyi.config;

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitmqConfig {
   
    //声明正常队列
    @Bean
    public Queue queue(){
        return QueueBuilder.durable("topic_queue")
                .deadLetterExchange("topic_exchange_delay")
                .deadLetterRoutingKey("time.msg")
                //设置队列过期时间,即到达指定时间发送消息
                .ttl(5000)
                .build();
    }
    //声明正常交换机
    @Bean
    public TopicExchange exchange(){
        return ExchangeBuilder.topicExchange("topic_exchange").build();
    }
    //正常的队列和交换机进行绑定
    @Bean
    public Binding binding(){
        return BindingBuilder.bind(queue()).to(exchange()).with("yes.#");
    }


    //声明延迟队列
    @Bean
    public Queue queue_delay(){
        return QueueBuilder.durable("topic_queue_delay").build();
    }
    //声明延迟交换机
    @Bean
    public TopicExchange exchange_delay(){
        return ExchangeBuilder.topicExchange("topic_exchange_delay").build();
    }
    //延迟的队列和交换机进行绑定
    @Bean
    public Binding binding_delay(){
        return BindingBuilder.bind(queue_delay()).to(exchange_delay()).with("time.#");
    }


}

```

  编写接口

```java
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/send")
    public void sendMessage(){
        SimpleDateFormat format = new SimpleDateFormat("HH:mm:ss");
        String current_time = format.format(new Date());
        rabbitTemplate.convertAndSend("topic_exchange","yes.msg","这是一条延时的消息,发送时间为："+current_time);
    }

}
```

 编写消费者方

```java

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class DelayReceiver {
    @RabbitListener(queues = "topic_queue_delay")
    public void queue(Message message){
        SimpleDateFormat format = new SimpleDateFormat("HH:mm:ss");
        String current_time = format.format(new Date());
        System.out.println("获取时间："+current_time);
        System.out.println("从延迟队列收到的消息："+new String(message.getBody()));
    }
}
```

启动测试，调用接口发送消息，查看消费者方控制台，5秒后会打印一条消息

![08Fv7j.png](https://s1.ax1x.com/2020/10/03/08Fv7j.png)







## 延迟队列（插件方式）

**注：**`前提是通过docker 安装RabbitMQ-Management`

下载插件

```shell
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.9.0/rabbitmq_delayed_message_exchange-3.9.0.ez
```



复制到docker容器MQ的插件目录下

注：docker cp 文件目录   docker容器id:容器内的路径

```shell
docker cp rabbitmq_delayed_message_exchange-3.9.0.ez 724766fbe93a:/opt/rabbitmq/plugins
```



进入到容器中

```shell
docker exec -it 724766fbe93a /bin/sh
```



开启插件

```shell
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```



查看插件是否安装

```shell
rabbitmq-plugins list
```

![](https://z3.ax1x.com/2021/11/21/IXv0XQ.png)



### 消费者方绑定方式实现延迟队列

创建接收消息类，使用@RabbitListener注解监听消息并同时绑定queue队列、exchange交换机、routing key

```java
@Component
public class MqReceiver {
	/**
	 * rabbitmq_delayed_message_exchange插件版延时队列
	 * @param msg
	 * @param message
	 * @param channel
	 */
	@RabbitListener(bindings= @QueueBinding(value = @Queue(value = "delay.queue", durable = "true"),
			exchange = @Exchange(value = "delay.exchange",type=ExchangeTypes.DIRECT,
			arguments=@Argument(name="x-delayed-type",value="direct"),delayed=Exchange.TRUE),
			key ="delay.key"))
	public void receiveDelay(String msg,Message message,Channel channel) {
		System.out.println(message.toString());
		System.out.println(message.getMessageProperties().getReceivedDelay());
		System.out.println("-----------收到消息:"+msg+",当前时间："+new Date());
	}

}
```



发送消息

```java
@RestController
public class TestContoller {

	@Autowired
	private AmqpTemplate amqpTemplate;
	
	@GetMapping(value = "/send/{exp}")
	public String test(@PathVariable("exp") Integer exp) {
		String msg = "发送时间："+new Date();
		amqpTemplate.convertAndSend(MQConstants.DELAY_EXCHANGE,MQConstants.THEATRE_DELAY_KEY,msg,message -> {
			message.getMessageProperties().setDelay(exp);// 单位 毫秒
			return message;
			});
		return "---------sendTime:"+new Date();
	}
}
```



然后启动项目，可以看到rabbitMQ控制台显示出了定义的exchange，注意type显示为x-delayed-message

![](https://z3.ax1x.com/2021/11/21/IXxqVs.png)

**注**：注意插件版的死信队列Features字段不会显示DLX



调用接口，定义10秒的延迟

![](https://z3.ax1x.com/2021/11/21/IXzEPx.png)





# 连接多个RabbitMq源

`注：以下为springboot方式`

## 自定义配置文件内容

```yml
custom:
  rabbitmq:
    primary:
      host: 192.168.1.101
      port: 3381
      username: admin #用户名
      password: rabbit_3381 #密码
      virtual-host: rabbitMq_1    #虚拟机名称
      #消息确认配置项
      #确认消息已发送到交换机(Exchange)
      publisher-confirm-type: correlated
      #确认消息已发送到队列(Queue)
      publisher-returns: true
    secondary:
      host: 192.168.1.102
      port: 3381
      username: admin #用户名
      password: rabbit_3381 #密码
      virtual-host: rabbitMq_2    #虚拟机名称
      #消息确认配置项
      #确认消息已发送到交换机(Exchange)
      publisher-confirm-type: correlated
      #确认消息已发送到队列(Queue)
      publisher-returns: true
```



## 创建对应的Properties属性类

> 记得加上@EnableConfigurationProperties({CustomRabbitProperties.class})

**主要内容如下：**

```java
import lombok.Data;
import lombok.ToString;
import org.springframework.boot.autoconfigure.amqp.RabbitProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;


/**
 * @Author ShaoYi
 * @Description 自定义Rabbitmq属性
 * @createDate 2022年08月05日 16:35
 **/
@Data
@ToString
@Component
@ConfigurationProperties(prefix = "custom.rabbitmq")
public class CustomRabbitProperties {

	private RabbitProperties primary;

	private RabbitProperties secondary;


}
```



## Rabbitmq配置类

**主要内容如下：**

```java

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

/**
 * @Author ShaoYi
 * @Description rabbitmq配置
 * @createDate 2022年08月05日 17:48
 **/
@Slf4j
@EnableRabbit
@Configuration
public class RabbitmqConfiguration {


    /**
	 * rabbitmq连接工厂
	 * @param customRabbitProperties
	 * @return
	 */
	@Bean(name = "primaryConnectionFactory")
	@Primary
	public ConnectionFactory createPrimaryConnectionFactory(CustomRabbitProperties customRabbitProperties) {
		CachingConnectionFactory primaryConnectionFactory = new CachingConnectionFactory();
		primaryConnectionFactory.setHost(customRabbitProperties.getPrimary().getHost());
		primaryConnectionFactory.setPort(customRabbitProperties.getPrimary().getPort());
		primaryConnectionFactory.setUsername(customRabbitProperties.getPrimary().getUsername());
		primaryConnectionFactory.setPassword(customRabbitProperties.getPrimary().getPassword());
		primaryConnectionFactory.setVirtualHost(customRabbitProperties.getPrimary().getVirtualHost());
		primaryConnectionFactory.setPublisherConfirmType(customRabbitProperties.getPrimary().getPublisherConfirmType());
		primaryConnectionFactory.setPublisherReturns(customRabbitProperties.getPrimary().isPublisherReturns());
		return primaryConnectionFactory;
	}

    /**
	 * rabbitmq连接工厂
	 * @param customRabbitProperties
	 * @return
	 */
	@Bean(name = "secondaryConnectionFactory")
	public ConnectionFactory createSecondaryConnectionFactory(CustomRabbitProperties customRabbitProperties) {
		CachingConnectionFactory secondaryConnectionFactory = new CachingConnectionFactory();
		secondaryConnectionFactory.setHost(customRabbitProperties.getSecondary().getHost());
		secondaryConnectionFactory.setPort(customRabbitProperties.getSecondary().getPort());
		secondaryConnectionFactory.setUsername(customRabbitProperties.getSecondary().getUsername());
		secondaryConnectionFactory.setPassword(customRabbitProperties.getSecondary().getPassword());
		secondaryConnectionFactory.setVirtualHost(customRabbitProperties.getSecondary().getVirtualHost());
		secondaryConnectionFactory.setPublisherConfirmType(customRabbitProperties.getSecondary().getPublisherConfirmType());
		secondaryConnectionFactory.setPublisherReturns(customRabbitProperties.getSecondary().isPublisherReturns());
		return secondaryConnectionFactory;
	}


	/**
	 * primary生产者
	 */
	@Bean(name = "primaryRabbitTemplate")
	@Primary
	public RabbitTemplate primaryRabbitTemplate(@Qualifier("primaryConnectionFactory") ConnectionFactory connectionFactory) {
		RabbitTemplate rabbitTemplate = new RabbitTemplate();
		rabbitTemplate.setConnectionFactory(connectionFactory);
		/**
		 * 从消费者(Producer) 到 交换机(exchange) 则会被调用
		 */
		rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
			log.info("[primaryTemplate]ConfirmCallback 相关数据: "+correlationData);
			log.info("[primaryTemplate]ConfirmCallback 确认情况: "+ack);
			log.info("[primaryTemplate]ConfirmCallback 原因: "+cause);
		});

		//设置开启Mandatory,才能触发回调函数,无论消息推送结果怎么样都强制调用回调函数
		rabbitTemplate.setMandatory(true);
		/**
		 * 从交换机(exchange)到 队列(queue) 则会被调用
		 */
		rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
			log.info("[primaryTemplate]ReturnCallback 消息内容: "+message);
			log.info("[primaryTemplate]ReturnCallback 回应码: "+replyCode);
			log.info("[primaryTemplate]ReturnCallback 回应消息: "+replyText);
			log.info("[primaryTemplate]ReturnCallback 交换机: "+exchange);
			log.info("[primaryTemplate]ReturnCallback 路由key值: "+routingKey);
		});
        // 添加json格式序列化器
		rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
		return rabbitTemplate;
	}

	/**
	 *   secondary生产者
	 */
	@Bean(name = "secondaryRabbitTemplate")
	public RabbitTemplate secondaryRabbitTemplate(@Qualifier("secondaryConnectionFactory") ConnectionFactory connectionFactory) {
		RabbitTemplate rabbitTemplate = new RabbitTemplate();
		rabbitTemplate.setConnectionFactory(connectionFactory);
		/**
		 * 从消费者(Producer) 到 交换机(exchange) 则会被调用
		 */
		rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
			log.info("[secondaryTemplate]ConfirmCallback 相关数据: "+correlationData);
			log.info("[secondaryTemplate]ConfirmCallback 确认情况: "+ack);
			log.info("[secondaryTemplate]ConfirmCallback 原因: "+cause);
		});

		//设置开启Mandatory,才能触发回调函数,无论消息推送结果怎么样都强制调用回调函数
		rabbitTemplate.setMandatory(true);
		/**
		 * 从交换机(exchange)到 队列(queue) 则会被调用
		 */
		rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
			log.info("[secondaryTemplate]ReturnCallback 消息内容: "+message);
			log.info("[secondaryTemplate]ReturnCallback 回应码: "+replyCode);
			log.info("[secondaryTemplate]ReturnCallback 回应消息: "+replyText);
			log.info("[secondaryTemplate]ReturnCallback 交换机: "+exchange);
			log.info("[secondaryTemplate]ReturnCallback 路由key值: "+routingKey);
		});
         // 添加json格式序列化器
		rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
		return rabbitTemplate;
	}


	/**
	 * primary 消费者
	 */
	@Bean(name = "primaryListenerFactory")
	public SimpleRabbitListenerContainerFactory primaryListenerFactory(SimpleRabbitListenerContainerFactoryConfigurer configurer
		, @Qualifier("primaryConnectionFactory") ConnectionFactory connectionFactory) {
		SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
		configurer.configure(factory, connectionFactory);
		factory.setConnectionFactory(connectionFactory);
        //消息序列化
		factory.setMessageConverter(new Jackson2JsonMessageConverter());
		return factory;
	}

	/**
	 * secondary 消费者
	 */
	@Bean(name = "secondaryListenerFactory")
	public SimpleRabbitListenerContainerFactory secondaryListenerFactory(SimpleRabbitListenerContainerFactoryConfigurer configurer
		, @Qualifier("secondaryConnectionFactory") ConnectionFactory connectionFactory) {
		SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
		configurer.configure(factory, connectionFactory);
		factory.setConnectionFactory(connectionFactory);
         //消息序列化
		factory.setMessageConverter(new Jackson2JsonMessageConverter());
		return factory;
	}
    
    
  	/**
	 * primary的AmqpAdmin
	 */
	@Primary
	@Bean(name="primaryAmqpAdmin")
	public AmqpAdmin primaryAmqpAdmin(@Qualifier("primaryConnectionFactory") ConnectionFactory connectionFactory) {
		RabbitAdmin admin = new RabbitAdmin(connectionFactory);
        /**
		 * 防止声明过的所有队列，交换机，绑定三种类型对象，在每个virtual-host上都创建一边，
		 * 除此之外还需要在声明队列，交换机，绑定的地方指定对应的AmqpAdmin 否则还是会创建
		 */
		admin.setExplicitDeclarationsOnly(true);
		admin.setAutoStartup(true);
		return admin;
	}

    /**
	 * secondary的AmqpAdmin
	 */
	@Bean(name="secondaryAmqpAdmin")
	public AmqpAdmin secondaryAmqpAdmin(@Qualifier("secondaryConnectionFactory") ConnectionFactory connectionFactory) {
		RabbitAdmin admin = new RabbitAdmin(connectionFactory);
        /**
		 * 防止声明过的所有队列，交换机，绑定三种类型对象，在每个virtual-host上都创建一边，
		 * 除此之外还需要在声明队列，交换机，绑定的地方指定对应的AmqpAdmin 否则还是会创建
		 */
		admin.setExplicitDeclarationsOnly(true);
		admin.setAutoStartup(true);
		return admin;
	}



}

```



## 发送消息

> 根据配置内容，@Qualifier 指定发送对象

```java
@Autowired
@Qualifier("primaryRabbitTemplate")
private  RabbitTemplate primaryRabbitTemplate;
```



## 接受消息

>根据配置内容，指定监听器工厂 containerFactory = "primaryListenerFactory"  
>
>@RabbitHandler：如果@RabbitListener注解在类上，可通过@RabbitHandler在不同方法上，接受不同对象类型消息
>
>除此之外，需要在声明队列，交换机，绑定的地方指定对应的AmqpAdmin， 否则每个virtual-host上都创建一边

```java
/**
  * 接受消息
  * @param msg 为自定义的消息对象
  */
@RabbitListener(
    containerFactory = "primaryContainerListenerFactory",
    admin = "primaryAmqpAdmin",
    bindings = @QueueBinding(value = @Queue(value = "alarm_rule_queue", durable = "true", admins = "primaryAmqpAdmin"),
            exchange = @Exchange(value = AlarmRuleParamBO.exchange, type = ExchangeTypes.DIRECT, admins = "primaryAmqpAdmin"),
            key = AlarmRuleParamBO.routingKey,
            admins = "primaryAmqpAdmin"))
@RabbitHandler
public void receiveMessage (Message  msg) {
    log.info("接受的消息为: {}", msg);
}
```

