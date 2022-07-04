---
title: "消息中间件之ActiveMQ"
date: 2020-07-03T19:18:23+08:00
lastmod: 2022-07-04T19:18:23+08:00
author: ["shaoyi"]
keywords: 
- ActiveMQ
categories: 
- ActiveMQ
tags: 
- ActiveMQ
description: "ActiveMQ的应用"
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



# ActiveMQ安装（Docker环境下）

首先在docker搜索ActiveMQ，发现**webcenter/activemq**  这个版本使用是最多的 

![wTXzlT.png](https://s1.ax1x.com/2020/09/20/wTXzlT.png)

然后选择版本，下载activeMQ

![wTj63V.png](https://s1.ax1x.com/2020/09/20/wTj63V.png)

查看镜像，配置好activemq容器的后台端口（61616）和前台端口（8161）的映射关系，并启动它

![wTvBrD.png](https://s1.ax1x.com/2020/09/20/wTvBrD.png)

浏览器访问，如下图，便安装完成

![wTzvE4.png](https://s1.ax1x.com/2020/09/20/wTzvE4.png)

# JMS实现消息的生产者和消费者

## 队列的方式

> 生产者：

```java


import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class JmsProduce {
    private static final String ACTIVEMQ_URL="tcp://服务器公网ip:后台映射端口号";
    private static final String QUEUE_NAME="myQueue";

    public static void main(String[] args) throws JMSException {
        //创建连接工厂，指定active的url链接，不指定用户名和密码就是默认的。
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建工厂后获取链接，并启动
        Connection connection = connectionFactory.createConnection();
        connection.start();
        //创建会话，有两个参数，分别是事务和签收机制
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建目的地（这里为队列方式）
        Destination destination = session.createQueue(QUEUE_NAME);
        //创建消息的生产者
        MessageProducer messageProducer = session.createProducer(destination);
        //生产4条消息
        for (int i = 1; i < 5; i++) {
            //创建消息内容
            TextMessage message = session.createTextMessage("第" + i + "条消息");
            //发送消息
            messageProducer.send(message);
        }
        //关闭资源
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发送完成！");
    }
}


```

**执行后的结果：**

![w79mct.png](https://s1.ax1x.com/2020/09/20/w79mct.png)

> 消费者（receive()方法获取信息）：

```java

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;


public class JmsConsumer {
    private static final String ACTIVEMQ_URL="tcp://服务器公网ip:后台映射端口号";
    private static final String QUEUE_NAME="myQueue";
    public static void main(String[] args) throws JMSException {
        //创建连接工厂，指定active的url链接，不指定用户名和密码就是默认的。
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建工厂后获取链接，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //创建会话，有两个参数，分别是事务和签收机制
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建目的地(这里为队列方式)
        Destination destination = session.createQueue(QUEUE_NAME);
        //创建消息的生产者
        MessageConsumer messageConsumer = session.createConsumer(destination);
        while (true){
            //读取消息
            TextMessage textMessage = (TextMessage)messageConsumer.receive();
            if (null !=textMessage){
                //获取消息内容并打印
                System.out.println("消费者接受的消息:  "+textMessage.getText());
            }else {
                break;
            }
        }
        //关闭资源
        messageConsumer.close();
        session.close();
        connection.close();
    }
}

```

**执行结果：**

![w7CjQ1.png](https://s1.ax1x.com/2020/09/20/w7CjQ1.png)



> 消费者（监听器MessageListener方式获取信息）

```java

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

public class JmsConsumer02 {
    private static final String ACTIVEMQ_URL="tcp://服务器公网ip:后台映射端口号";
    private static final String QUEUE_NAME="myQueue";
    public static void main(String[] args) throws JMSException, IOException {
        //创建连接工厂，指定active的url链接，不指定用户名和密码就是默认的。
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建工厂后获取链接，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //创建会话，有两个参数，分别是事务和签收机制
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建目的地(这里为队列方式)
        Destination destination = session.createQueue(QUEUE_NAME);
        //创建消息的生产者
        MessageConsumer messageConsumer = session.createConsumer(destination);
        messageConsumer.setMessageListener((message) -> {
                if (null != message && message instanceof TextMessage){
                    TextMessage textMessage = (TextMessage) message;
                    try {
                        System.out.println("消费者接受的消息:  "+textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
        });
        System.in.read();   //防止消息还没有读取到就关闭资源了
        //关闭资源
        messageConsumer.close();
        session.close();
        connection.close();
    }
}
```





> 消费者获取消息的方式

**receive() 方法**：该方法是同步阻塞的，如果调用者（队列或者订阅方）没有接受到消息，就会一直等待

除此之外它有一个重载方法 **receive(long timeout)** ，参数表示的是 等待的超时时间（毫秒），在等待时间内没有接受到消息就自动停止



**setMessageListener(MessageListener listener) 方法**：该方法是异步非阻塞的，而方法参数类型是MessageListener接口，这个接口里只定义了一个用于处理接收到的消息的onMessage方法，该方法只接收一个Message参数监听器。当消息到达之后，会自动调用onMessage方法获取信息



> 消费者的三种情况

- 先启动生产者，生产消息，然后再启动一个消费者，是否可以正常消费信息 ？   
  - 答案：**可以的**
- 还是先启动生产者，生产消息，再分别启动两个消费者，后一个启动的消费者可以消费到消息吗 ？
  - 答案：**否，后一个启动的消费者是消费不到的消息，消息都被前一个启动的消息者消费完了**

- 先启动两个消费者，再启动生产者生产6条消息，结果如何 ？
  - 答案：**消息被平分，两个消费者一人一半，分别消费3条消息**





## 发布订阅方式

> 生产者：

```java

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class JmsProduce_Topic {
    private static final String ACTIVEMQ_URL="tcp://服务器公网ip:后台映射端口号";
    private static final String TOPIC_NAME="myTopic";
    public static void main(String[] args) throws JMSException {
        //创建连接工厂，指定active的url链接，不指定用户名和密码就是默认的。
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        //创建工厂后获取链接，并启动
        Connection connection = connectionFactory.createConnection();
        connection.start();
        //创建会话，有两个参数，分别是事务和签收机制
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建目的地（这里为发布订阅方式）
        Destination destination = session.createTopic(TOPIC_NAME);
        //创建消息的生产者
        MessageProducer messageProducer = session.createProducer(destination);
        //生产4条消息
        for (int i = 1; i < 5; i++) {
            //创建消息内容
            TextMessage message = session.createTextMessage("第" + i + "条消息");
            //发送消息
            messageProducer.send(message);
        }
        //关闭资源
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发送完成！");
    }
}


```

​		发布订阅的方式和队列方式， 从生产者的代码可以发现，两者是一样的，**唯一不同的就是，创建目的地的方法不一样，队列是createQueue(String queueName)，而发布订阅是createTopic(String topicName)。同样的，消费者的代码和队列也是一样的**

> 消费者

```java


import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.io.IOException;

public class JmsConsumer_Topic {
    private  static final String ACTIVEMQ_URL="tcp://服务器公网ip:后台映射端口号";
    private static final String TOPIC_NAME="myTopic";
    public static void main(String[] args) throws JMSException, IOException {
        //创建连接工厂，指定active的url链接，不指定用户名和密码就是默认的。
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory();
        //创建工厂后获取链接，并启动
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //创建会话，有两个参数，分别是事务和签收机制
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        //创建目的地（这里为发布订阅方式）
        Destination destination = session.createTopic(TOPIC_NAME);
        //创建消息的生产者
        MessageConsumer messageConsumer = session.createConsumer(destination);
        //通过监听器，接受消息
        messageConsumer.setMessageListener((message) -> {
            if (null != message && message instanceof TextMessage) {
                try {
                    TextMessage textMessage = (TextMessage) message;
                    System.out.println("消费者接受的消息:  " + textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
        System.in.read();//阻塞等待
        //关闭资源
        messageConsumer.close();
        session.close();
        connection.close();
    }
}





```



​		不过要注意的是，**发布订阅的模式，要先启用订阅方，也就是消费者方，再启动发布方（生产者）**。否则，就类似微信公众号一样，没有人订阅，生产方发布消息是没有人接受的到，会被丢弃。这种则是废消息。



## 队列和发布订阅两者对比

| 比 较 项 目    | Topic （发布订阅）模式                                       | Queue （队列）模式                                           |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 工 作 模 式    | “订阅-发布”模式，如果当前没有订阅 者，消息将会被丢弃，如果有多个订阅 者，那么这些订阅者都会收到消息。 | 负载均衡 模式，如果当前没有消费者，消息也不会丢弃，如果有多个消费者，那么一条消息 也只会发送给其中一个消费者，并且要求消费者ack信息 |
| 有 无 状 态    | 无状态                                                       | Queue 数据默认会在 mq 服务器上以文件形式 保存，比如 Active MQ 一般保存在 $AMQ_HOME\data\kr-store\data下面，也可 以配置成DB存储 |
| 传 递 完 整 性 | 如果没有订阅者，消息会被丢弃                                 | 消息不会丢弃                                                 |
| 处 理 效 率    | 由于消息要按照订阅者的数量进行复 制，所以处理性能会随着订阅者的增加 而明显降低，并且还要结合不同消息协 议自身的性能差异 | 由于一条消息只发送给一个消费者，所以就算 消费者再多，性能也不会有明显降低。当然不 同消息协议的具体性能也是有差异的。 |



# JMS

​		JMS是Java EE的一个面向消息中间件的一套规范，具体需要靠厂商的产品实现

![wLZGvT.png](https://s1.ax1x.com/2020/09/22/wLZGvT.png)



## 消息的三种部分

### 消息头

- 在生产者方，消息的API看出可以针对每一条消息设置不同的消息头属性



![wHNZE6.png](https://s1.ax1x.com/2020/09/21/wHNZE6.png)

- 如果不想为每一条消息单独设置消息头属性，可以通过Send()方法的重载，批处理消息的消息头属性



![wHNEHx.png](https://s1.ax1x.com/2020/09/21/wHNEHx.png)



- 从批处理消息头属性和单独设置消息头属性两者结合来看，主要介绍五个消息头属性：



**JMSDestination：**  消息发送的目的地，主要是指 Queue 和 Topic

**JMSDeliveryMode：**  持久模式和非持久模式，可设置的参数为**DeliveryMode.NON_PERSISTENT(非持久)**  和  **DeliveryMode.PERSISTENT(持久)**

一条持久性的消息：应该被传送 “一次仅仅一次”，这就意味着如果JMS 提供者出现故障，该消息并不会丢失，它会在服务器恢复之后再次传递消息。
一条非持久的消息：多会传送一次，这意味着服务器出现故障，该消息将永久丢失。

**JMSPriority ：**  消息优先级，从0 ~ 9 十个级别，0 ~ 4 是普通消息，5 ~ 9是加急消息。

JMS 不要求 MQ 严格按照这十个优先级发送消息，但必须保证加急消息要先于普通消息到达，默认是4 级。

**JMSExpiration：**  可以设置消息在一定时间后过期，默认是永不过期。消息过期时间，等于Destination的send方法中的timeToLive值加上发送时刻的GMT时间

值。 如果timeToLive值等于零，则 JMSExpiration 被设为零，表示该消息永不过期。如果发送后，在消息过期时间之后消息还没有被发送到目的地，则该消息被清除。

**JMSMessageID ：**  唯一识别每个消息的标识，由MQ产生。



### 消息体

封装具体的消息数据，发送和接收的消息体类型必须一致对应！

五种消息体格式：

**TextMessage：** 普通字符串消息，包含一个String 。

**MapMessage：** 一个Map类型的消息，key为String类型，而值为Java的基本类型。 

BytesMessage  ： 二进制数组消息，包含一个byte[] 。

StreamMessage ： Java数据流消息，用标准流操作来顺序的填充和读取。

ObjectMessage ：对象消息，包含一个可序列化的 Java 对象。



### 消息属性

如果需要除消息头字段以外的值，那么可以使用消息属性！

识别、去重、重点标注等操作非常有用的方法！

消息属性是以属性名和属性值对的形式指定的。可以将属性是为消息头的扩展，属性指定一些消息头没有包括的附加信息，比如可以在属性里指定消息选择器。

以Property结尾：

![wHsArF.png](https://s1.ax1x.com/2020/09/21/wHsArF.png)





## 消息的可靠性

### 持久性 队列

```java
 //创建目的地（这里为队列方式）
 Destination destination = session.createQueue(QUEUE_NAME);
 //创建消息的生产者
 MessageProducer messageProducer = session.createProducer(destination);

// 非持久化 ：服务器宕机，则消息不能恢复
 messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
// 持久化 ：服务器宕机，消息可以恢复
 messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
```

队列是否设置为持久，可以通过setDeliveryMode(int deliveryMode)方法设置，**默认为持久**

### 持久性  发布订阅 

> 发布方（生产者），顺序与之前有一点不同，需要设定先持久化，在启动连接

测试代码：

```java

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class JmsProduce_Topic_DeliveryMode {
    private static final String ACTIVEMQ_URL="tcp://服务器公网ip:后台映射端口号";
    private static final String TOPIC_NAME="my_topic_persist";
    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = connectionFactory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageProducer messageProducer = session.createProducer(topic);
        //设置发布方为 持久
        messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
        //设定为持久后，然后连接
        connection.start();
        for (int i = 1; i <=3; i++) {
            TextMessage message = session.createTextMessage("第" + i + "条消息");
            messageProducer.send(message);
        }
        //关闭资源
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发送完成！");
    }
}


```

> 订阅方（消费者）测试代码：

```java

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class JmsConsumer_Topic_DeliveryMode {
    private static final String ACTIVEMQ_URL="tcp://服务器公网ip:后台映射端口号";
    private static final String TOPIC_NAME="my_topic_persist";

    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = connectionFactory.createConnection();
        //订阅方的id
        connection.setClientID("001");
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        //创建持久的订阅方
        TopicSubscriber topicSubscriber = session.createDurableSubscriber(topic, "十十乙");
        //持久化后，再进行连接
        connection.start();
        Message message = topicSubscriber.receive();
        while (null != message){
            TextMessage textMessage = (TextMessage)message;
            System.out.println("订阅方收到的消息："+textMessage.getText());
            message = topicSubscriber.receive(3000L);
        }
        topicSubscriber.close();
        session.close();
        connection.close();
    }
}

```

**先启动订阅方，再启动发布方**，到ActiveMQ前台的**Subscribers**查看



> 启动前情况：

![wHIUiT.png](https://s1.ax1x.com/2020/09/21/wHIUiT.png)

> 启动订阅方后情况：

![wHITeI.png](https://s1.ax1x.com/2020/09/21/wHITeI.png)

![wHo9wq.png](https://s1.ax1x.com/2020/09/21/wHo9wq.png)

> 启动生产方 后：

![wHo0tf.png](https://s1.ax1x.com/2020/09/21/wHo0tf.png)

> 订阅方停止后，会被列 离线的一栏去：

![wHo4hT.png](https://s1.ax1x.com/2020/09/21/wHo4hT.png)

> 如果这时，发布者再次发布消息，订阅方重启启动是否能接收到消息

![wHTuvQ.png](https://s1.ax1x.com/2020/09/21/wHTuvQ.png)

> 再次启动订阅方(不停止)后

![wHT7Pf.png](https://s1.ax1x.com/2020/09/21/wHT7Pf.png)
![wHTHG8.png](https://s1.ax1x.com/2020/09/21/wHTHG8.png)

> 小结：

​		**第一次，一定要先启动订阅方，之后启动发布方，发布消息。这时不论订阅方在不在线，都可以接受到消息，不在线，重新启动后会接受没有收到过的消息接受下来**



### 事务

**事务偏向于生产者、签收偏向于消费者**

```java

//创建会话，有两个参数，分别是事务和签收机制
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

```

事务只有两个值，true和false

> 生产者一方

当值为false时，消息会被自动发送到目的地

当值为true时，需要在调用send()方法后，调用commit()方法，手动提交

>消费者一方

当值为false时，消息只会被消费一次

当值为true时，消息会被重复消费，且在目的地中，消息不会出队的，即未被消费

测试：

```java
        //创建会话，开启事务
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
    
        try {
            //OK,就提交
            session.commit();
        } catch (JMSException e) {
           //error,就回滚
            session.rollback();
        }finally {
            //关闭资源
            
        }

```

### 签收机制

**签收针对的是消费者一方，分为以下四种**

```java
public interface Session extends Runnable {
    static final int AUTO_ACKNOWLEDGE = 1;   //自动签收（默认）

    static final int CLIENT_ACKNOWLEDGE = 2;	//手动签收，需要调用 message.acknowledge(); 进行签收，否则也会导致重复消费
 
    static final int DUPS_OK_ACKNOWLEDGE = 3;	//自动批量确认

    static final int SESSION_TRANSACTED = 0;	//这个是配合事务，当事务开启则选择这个，主要是一个语法的一个规范而已
```

**当事务没有开启时 ：**  即非事务，消息的确认情况 则以签收机制为主

**当事务开始时 ：**  即事务情况下，消息的确认以事务为主，即是否commit的为主，不用管签收机制的情况。

**注：常用的签收模式 主要以自动签收（AUTO_ACKNOWLEDGE）和手动签收（CLIENT_ACKNOWLEDGE）为主**



# SpringBoot结合ACtiveMQ

## 队列模式

1、导入maven依赖

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!--activemq的依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>

```

2、编写application.properties配置文件

```properties

server.port=8888

# 配置activemq的链接，用户名和密码
spring.activemq.broker-url=tcp://服务器公网ip:端口号
spring.activemq.user=admin
spring.activemq.password=admin

#配置是否为发布订阅模式
#false为队列（默认）
#ture为发布订阅
spring.jms.pub-sub-domain=false

#队列名称（自定义标签）
myqueue=activemq_springboot_queue

```

3、编写配置类

```java

package com.shishiyi.config;


import org.apache.activemq.command.ActiveMQQueue;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.jms.Queue;

@Configuration
public class ActiveMQConfig {
    //注入队列名称
    @Value("${myqueue}")
    private String myQueue_name;
    //向spring容器导入队列对象
    @Bean
    public Queue queue(){
        return new ActiveMQQueue(myQueue_name);
    }
}

```

4、编写队列的消息生产者Produce

```java

package com.shishiyi.queue;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Component;

import javax.jms.Queue;
@Component
public class QueueProduce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;
    @Autowired
    private Queue queue_name;

    public void produceSend(){
        //发送消息
        jmsMessagingTemplate.convertAndSend(queue_name,"生产者生产消息");
    }
}

```

5、编写controller类，并访问接口

```java

package com.shishiyi.controller;

import com.shishiyi.queue.QueueProduce;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class QueueProduceController {
    @Autowired
    private QueueProduce queueProduce;

    @GetMapping("/queuesend")
    public void queueSend(){
        queueProduce.produceSend();
        System.out.println("消息发送成功");
    }
}

```

查看ActiveMQ的前台，消息已经发送到队列了

![wqCXuR.png](https://s1.ax1x.com/2020/09/21/wqCXuR.png)

6、编写队列的消费者

```java

package com.shishiyi.queue;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Component;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.TextMessage;

@Component
public class QueueConsumer {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    //采用监听器的方式获取消息
    @JmsListener(destination = "${myqueue}")
    public void receive(Message message) throws JMSException {
        if (null !=message && message instanceof TextMessage){
            TextMessage textMessage = (TextMessage)message;
            System.out.println("消费者已经接受到了"+textMessage.getText());
        }
    }
}

```

删除之前入队的信息，然后重新启动 启动类，再次访问接口，并查看ActiveMQ的前台和打印台

![wqkFv4.png](https://s1.ax1x.com/2020/09/21/wqkFv4.png)![wqkfRU.png](https://s1.ax1x.com/2020/09/21/wqkfRU.png)



## 订阅发布模式

1、导入maven依赖

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>

```

2、编写application.properties配置文件

```properties

server.port=7777

# 配置activemq的链接，用户名和密码
spring.activemq.broker-url=tcp://服务器公网ip:端口号
spring.activemq.user=admin
spring.activemq.password=admin

#配置是否为发布订阅模式
#false为队列（默认）
#ture为发布订阅
spring.jms.pub-sub-domain=true

#topic名称
mytopic=activemq_springboot_topic

```

3、编写配置类

```java

package com.shishiyi.config;

import org.apache.activemq.command.ActiveMQTopic;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.jms.Topic;

@Configuration
public class ActiveMQConfig {
    @Value("${mytopic}")
    private String myTopic;
    @Bean
    public Topic topic(){
        return new ActiveMQTopic(myTopic);
    }
}

```

4、编写主题的消息生产者（发布方）

```java

package com.shishiyi.topic;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Component;

import javax.jms.Topic;

@Component
public class TopicProduce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Topic topic;

    public void topicSend(){
        jmsMessagingTemplate.convertAndSend(topic,"发布方向主题发布的消息");
        System.out.println("消息发送成功");
    }
}

```

5、编写controller类

```java

package com.shishiyi.controller;

import com.shishiyi.topic.TopicProduce;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TopicProduceController {

    @Autowired
    private TopicProduce topicProduce;

    @GetMapping("/topicSend")
    public void topicSend(){
        topicProduce.topicSend();
    }
}

```

6、编写主题的消费者（订阅方）

```java

package com.shishiyi.topic;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Component;

import javax.jms.*;

@Component
public class TopicConsumer {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @JmsListener(destination = "${mytopic}")
    public void receive(Message message) throws JMSException {
        if (null != message && message instanceof TextMessage){
            TextMessage textMessage = (TextMessage) message;
            System.out.println("订阅方收到的消息："+textMessage.getText());
        }
    }
}


```

7、启动主启动类，访问接口，让发布方生产消息，并查看ActiveMQ的前台和打印台

![wqeLvT.png](https://s1.ax1x.com/2020/09/21/wqeLvT.png)![wqevb4.png](https://s1.ax1x.com/2020/09/21/wqevb4.png)



> 定时的订阅发布模式

让发布方每过5秒，发送一次消息

```java

//创建定时任务

package com.shishiyi.topic;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import javax.jms.Topic;

@Component
public class TopicProduce {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Topic topic;

    public void topicSend(){
        jmsMessagingTemplate.convertAndSend(topic,"发布方向主题发布的消息");
        System.out.println("消息发送成功");
    }
	//创建定时任务，每隔5秒发布消息
    @Scheduled(fixedRate = 5000L)
    public void scheduled(){
        topicSend();
    }
}

```

添加 @EnableScheduling 注解，发现定时任务让它后台执行

![wLkdpT.png](https://s1.ax1x.com/2020/09/22/wLkdpT.png)

修改主题的消费者（订阅方），添加LocalDateTime.now()方法，方便在控制台查看

```java
 
package com.shishiyi.topic;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Component;

import javax.jms.*;
import java.time.LocalDateTime;


@Component
public class TopicConsumer {
    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @JmsListener(destination = "${mytopic}")
    public void receive(Message message) throws JMSException {
        if (null != message && message instanceof TextMessage){
            TextMessage textMessage = (TextMessage) message;
            System.out.println("订阅方收到的消息："+textMessage.getText()+ "时间为：" +LocalDateTime.now());
        }
    }
}

```

启动程序，查看控制台打印内容，以及（同一时间点内）查看ActiveMQ的前台

![wLVl6O.png](https://s1.ax1x.com/2020/09/22/wLVl6O.png)
![wLVQ1K.png](https://s1.ax1x.com/2020/09/22/wLVQ1K.png)

# ActiveMQ传输协议

在连接activemq时，默认是用TCP连接，那Activemq是否还支持其它协议吗？

查看activemq的conf文件夹下的**activemq.xml**

![wjwPQx.png](https://s1.ax1x.com/2020/09/23/wjwPQx.png)



可以看出，除了支持默认的TCP协议，还支持 amqp ，stomp，mqtt，ws等协议，而且协议对应的端口号也不一样，这就是用TCP连接时，端口号是61616的原因了。

从activemq前台也可以查看出来

![wj03g1.png](https://s1.ax1x.com/2020/09/23/wj03g1.png)

如果想配置其它协议，可以查看官网的文档

http://activemq.apache.org/configuring-version-5-transports

>### 配置NIO协议

![wjySYD.png](https://s1.ax1x.com/2020/09/23/wjySYD.png)



从官网的文档得知，配置NIO协议，只需要修改**activemq.xml**文件

```xml

    <!--添加一段即可-->
  <transportConnectors>
    <transportConnector name="nio" uri="nio://0.0.0.0:61617"/>  
  </<transportConnectors>

```

![wj59Ej.png](https://s1.ax1x.com/2020/09/23/wj59Ej.png)

重启activemq（重启容器），查看activemq前台

![wj5Raj.png](https://s1.ax1x.com/2020/09/23/wj5Raj.png)

修改代码的连接方式，并测试

```java
 //由于安装的activemq是docker容器，添加的nio协议端口是61617，所以端口要提前映射好，否则会连不上
	private static final String ACTIVEMQ_URL="nio://47.115.151.184:61617";
    private static final String QUEUE_NAME="myQueue_NIO";

```

查看前台

![wjbZKx.png](https://s1.ax1x.com/2020/09/23/wjbZKx.png)



# ActiveMQ持久化

持久化不同于之前的持久性，持久化是以防服务器本身挂了，重启服务器也可以恢复消息

ActiveMQ目前默认的持久化方式是KahaDB，此外还有JDBC， LevelDB等，不过它们大体上的逻辑是一样，消息中心接受到消息。会把消息备份存储起来。

## KahaDB(默认)

**KahaDB：这个5.4版本之后出现的默认的持久化方式，** KahaDB的持久化机制是基于日志文件，索引和缓存。

从**activemq.xml**配置文件中可以得知

![wjxVYT.png](https://s1.ax1x.com/2020/09/23/wjxVYT.png)

它是把数据存储在activemq的data目录下的kahadb文件夹中

在这个文件夹中会有四个文件来完成消息持久化

- **db.data** 它是消息的索引文件，本质上是B-Tree（B树），使用B-Tree作为索引指向**db-*.log**里面存储的消息
- **db.redo** 用来进行消息恢复
- **db-*.log** 存储消息内容。新的数据以APPEND的方式追加到日志文件末尾。属于顺序写入，因此消息存储是比较快的。默认是32M，达到阀值会自动递增文 件，文件名按照数字进行编号，如 db-1.log、db-2.log、.....
- **lock** 文件 锁，写入当前获得kahadb读写权限的broker ，用于在集群环境下的竞争处理

**KahaDB配置官网链接：**  http://activemq.apache.org/kahadb



## LevelDB 

这种文件系统是从ActiveMQ 5.8 之后引进的，它和 KahaDB 非常相似，也是基于文件的本地数据库存 储形式，但是它提供比 KahaDB更快的持久性。

但它不使用自定义 B-Tree 实现来索引预写日志，而是使用基于LevelDB的索引。 

可是为什么LeavelDB 更快，并且5.8 以后就支持，为什么还是默认 KahaDB 引擎，因为 activemq 官网本身没有定论，LeavelDB 之后又出了可复制的

LeavelDB 比LeavelDB 更性能更优越，但需要基于 ZooKeeper 所以这些官方还没有定论，仍旧使用 KahaDB。

默认配置：

```xml

  <broker brokerName="broker" ... >
    ...
    <persistenceAdapter>
      <levelDB directory="activemq-data"/>
    </persistenceAdapter>
    ...
  </broker>

```

**LevelDB配置官网链接：**  http://activemq.apache.org/leveldb-store



## JDBC

看名字就知道和数据库有关，所以JDB的持久化就是把消息备份到数据库中

由于用docker比较麻烦，所以直接改为windows本地测试

首先下载activemq的windows版本，http://activemq.apache.org/components/classic/download/

然后下载mysql的驱动包，

把mysql驱动包丢到activemq的lib下

![wv9hxU.png](https://s1.ax1x.com/2020/09/23/wv9hxU.png)

打开activemq的**conf目录下的activemq.xml文件**，配置内容：

```xml
		<!--
            Configure message persistence for the broker. The default persistence
            mechanism is the KahaDB store (identified by the kahaDB tag).
            For more information, see:

            http://activemq.apache.org/persistence.html
        
        <persistenceAdapter>
            <kahaDB directory="${activemq.data}/kahadb"/>
        </persistenceAdapter>
		屏蔽之前的持久化
        -->
		<!--改用mysql方式-->
		<persistenceAdapter>
			<jdbcPersistenceAdapter dataSource="#mysqlDataSource" createTablesOnStartup="true"/>
		</persistenceAdapter>
```

参数解释：

**dataSource：**   表示的是引用的数据源名称

**createTablesOnStartup：**   表示启动的时候是否去创建表，一般第一次为true,之后改为false



接着由于指定了数据源，所以需要在**activemq.xml**配置一个数据源，不过要注意位置

```xml

<bean id="mysqlDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close"> 
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>      
    <property name="url" value="jdbc:mysql://localhost:3306/activemq?relaxAutoCommit=true"/>      
    <property name="username" value="root"/>     
    <property name="password" value="123"/>   
</bean>

```

**然后创建好数据库，数据库名字与数据源链接的要一致**



**测试：**

> Queue



```java
//生产者

public class JmsProduce {
    private static final String ACTIVEMQ_URL="tcp://127.0.0.1:61616";
    private static final String QUEUE_NAME="queue_jdbc_mysql";
    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);     
        Connection connection = connectionFactory.createConnection();
        connection.start();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Destination destination = session.createQueue(QUEUE_NAME);
        MessageProducer messageProducer = session.createProducer(destination);
        //一定要开启持久化
        messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
        for (int i = 1; i < 5; i++) {
            TextMessage message = session.createTextMessage("第" + i + "条消息");
            messageProducer.send(message);
        }
        //关闭资源
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发送完成！");
    }
}
```

启动后，分别查看数据库和前台：

![wvZeVU.png](https://s1.ax1x.com/2020/09/23/wvZeVU.png)
![wvZmaF.png](https://s1.ax1x.com/2020/09/23/wvZmaF.png)



```java
//消费者

public class JmsConsumer {
    private static final String ACTIVEMQ_URL="tcp://127.0.0.1:61616";
    private static final String QUEUE_NAME="queue_jdbc_mysql";
    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Destination destination = session.createQueue(QUEUE_NAME);
        MessageConsumer messageConsumer = session.createConsumer(destination);
        while (true){
            TextMessage textMessage = (TextMessage)messageConsumer.receive();
            if (null !=textMessage){
                //获取消息内容并打印
                System.out.println("消费者接受的消息:  "+textMessage.getText());
            }else {
                break;
            }
        }
        //关闭资源
        messageConsumer.close();
        session.close();
        connection.close();
    }
}

```

启动消费者查看数据库和前台：

![wve3Wj.png](https://s1.ax1x.com/2020/09/23/wve3Wj.png)
![wveGSs.png](https://s1.ax1x.com/2020/09/23/wveGSs.png)

> Queue 小结：

​		在点对点类型中，当**DeliveryMode**设置为**NON_PERSISTENCE** 时，消息被保存在内存中； 当**DeliveryMode**设置为**PERSISTENCE** 时，消息被保存在Broker的相应的文件或者数据库中。而且点对点类型中消息一旦被Consumer消费就从Broker中删除 





> Topic



```java
//发布方

public class JmsProduce_Topic {
    private static final String ACTIVEMQ_URL="tcp://127.0.0.1:61616";
    private static final String TOPIC_NAME="topic_jdbc_mysql";
    
    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = connectionFactory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageProducer messageProducer = session.createProducer(topic);
        //设置发布方为 持久
        messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
        //设定为持久后，然后连接
        connection.start();
        for (int i = 1; i <=3; i++) {
            TextMessage message = session.createTextMessage("第" + i + "条消息");
            messageProducer.send(message);
        }
        //关闭资源
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发送完成！");
    }
}

```



```java
//订阅方

public class JmsConsumer_Topic_DeliveryMode {
    private static final String ACTIVEMQ_URL="tcp://127.0.0.1:61616";
    private static final String TOPIC_NAME="topic_jdbc_mysql";

    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = connectionFactory.createConnection();
        //订阅方的id
        connection.setClientID("001");
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        //创建持久的订阅方
        TopicSubscriber topicSubscriber = session.createDurableSubscriber(topic, "十十乙");
        //持久化后，再进行连接
        connection.start();
        Message message = topicSubscriber.receive();
        while (null != message){
            TextMessage textMessage = (TextMessage)message;
            System.out.println("订阅方收到的消息："+textMessage.getText());
            message = topicSubscriber.receive();
        }
        topicSubscriber.close();
        session.close();
        connection.close();
    }
}
```

先启动订阅方然后再启动发布方，查看数据库和前台：

![wvKJ4U.png](https://s1.ax1x.com/2020/09/23/wvKJ4U.png)
![wvKGNT.png](https://s1.ax1x.com/2020/09/23/wvKGNT.png)
![wvK1H0.png](https://s1.ax1x.com/2020/09/23/wvK1H0.png)
![wvKKjs.png](https://s1.ax1x.com/2020/09/23/wvKKjs.png)

>Topic 小结：

一定要先启动消费订阅然后再启动生产，消息会被保留在 activemq_msgs表中消费完毕也不会删除，而订阅关系在 acks 表 中。