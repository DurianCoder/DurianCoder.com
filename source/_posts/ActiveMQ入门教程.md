---
title: ActiveMQ入门教程
date: 2018-08-08 17:08:13
tags:
	- 消息队列
	- ActiveMQ
---

## 0x01、ActiveMQ简介

##### 1、什么是ActiveMQ？

[Apache ActiveMQ](http://activemq.apache.org/) ™ is the most popular and powerful open source messaging and [Integration Patterns](http://activemq.apache.org/enterprise-integration-patterns.html) server.

ActiveMQ是一个MOM，是一个实现了JMS规范的系统间远程通信的消息代理。

##### 2、什么是MOM?

MOM是面向消息中间件（Message-oriented middleware），是用于以分布式应用或系统中的异步、松耦合、可靠、可扩展和安全通信的一类软件。MOM的总体思想是它作为消息发送器和消息接受器之间的消息中介，这种中介提供了一个全新的水平松耦合。![](/897247-20170209130542229-26011702.png)

##### 3、ActiveMQ中一些基本概念

- Provider ：纯Java语言编写的JMS接口实现（比如ActiveMQ）
- Domains：消息传递方式，包括点对点（P2P)、发布订阅（Pub/Sub）两种。
- Connection factory：客户端使用连接工厂来创建与JMS provider的连接
- Destination：消息被寻址、发送以及接受的对象



##### 4、P2P和Pub/Sub的区别

1）、P2P

P2P消息域使用queue作为Destination，消息可以被同步和一部的发送和接收，每个消息只会给一个Consumer传递一次。

Consumer可以使用MessageConsumer.receive()同步接收消息；也可以使用Message.SetMessageListener()注册一个监听器来实现一部接收，在后面的Coding环节中会有代码演示。

多个Consumer可以注册到同一个Queue，但一个消息只能被一个Consumer所接收，然后由该Consumer来确认消息。并且这种情况下，Provider对所有注册的Consumer以轮询的方式发送消息。![](/p2p.png)

2)、Pub/Sub

发布订阅模式的消息域使用topic作为Destination，发布者向topic发送消息，订阅者注册接收来自topic的消息。发送到topic的任何消息都将自动传递给所有订阅者。接收方式有同步和异步与P2P相同。

除非显示指定，否者topic不会为订阅者保留消息。当然可以通过持久化订阅来实现消息的保存。这种情况，当订阅者与发布者断开时，发布者会为他存储消息。当持久化订阅者重新连接时，将会收到所有的断连期间未收到的消息。

![](/topic.png)

##### 5、使用ActiveMQ应用大致步骤

- 获取连接工厂

- 使用连接工厂创建连接

- 启动连接

- 从连接创建会话

- 获取 Destination

- 创建 Producer，或

  - 创建 Producer
  - 创建 message

- 创建 Consumer，或发送或接收message发送或接收 message

  - 创建 Consumer
  - 注册消息监听器（可选）

- 发送或接收 message

- 关闭资源（connection, session, producer, consumer 等)

  

## 0x2、ActiveMQ的存储机制

**1）、KahaDB**

ActiveMQ 5.3 版本起的默认存储方式。KahaDB存储是一个基于文件的快速存储消息，设计目标是易于使用且尽可能快。它使用基于文件的消息数据库意味着没有第三方数据库的先决条件。

要启用 KahaDB 存储，需要在 activemq.xml 中进行以下配置：

```
<broker brokerName="broker" persistent="true" useShutdownHook="false">
        <persistenceAdapter>
                <kahaDB directory="${activemq.data}/kahadb" journalMaxFileLength="16mb"/>
        </persistenceAdapter>
</broker>
```

**2）、AMQ**

与 KahaDB 存储一样，AMQ存储使用户能够快速启动和运行，因为它不依赖于第三方数据库。AMQ 消息存储库是可靠持久性和高性能索引的事务日志组合，当消息吞吐量是应用程序的主要需求时，该存储是最佳选择。但因为它为每个索引使用两个分开的文件，并且每个 Destination 都有一个索引，所以当你打算在代理中使用数千个队列的时候，不应该使用它。

```
<persistenceAdapter>
        <amqPersistenceAdapter
                directory="${activemq.data}/kahadb"
                syncOnWrite="true"
                indexPageSize="16kb"
                indexMaxBinSize="100"
                maxFileLength="10mb" />
</persistenceAdapter>
```

**3）、JDBC**

选择关系型数据库，通常的原因是企业已经具备了管理关系型数据的专长，但是它在性能上绝对不优于上述消息存储实现。事实是，许多企业使用关系数据库作为存储，是因为他们更愿意充分利用这些数据库资源。

```
<beans>
        <broker brokerName="test-broker" persistent="true" xmlns="http://activemq.apache.org/schema/core">
                <persistenceAdapter>
                        <jdbcPersistenceAdapter dataSource="#mysql-ds"/>
                </persistenceAdapter>
        </broker>
        <bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
                <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost/activemq?relaxAutoCommit=true"/>
                <property name="username" value="activemq"/>
                <property name="password" value="activemq"/>
                <property name="maxActive" value="200"/>
                <property name="poolPreparedStatements" value="true"/>
        </bean>
</beans>
```

 **4）、内存存储** 

内存消息存储器将所有持久消息保存在内存中。在仅存储有限数量 Message 的情况下，内存消息存储会很有用，因为 Message 通常会被快速消耗。在 activema.xml 中将 broker 元素上的 persistent 属性设置为 false 即可。 

```
<broker brokerName="test-broker" persistent="false" xmlns="http://activemq.apache.org/schema/core">
        <transportConnectors>
                <transportConnector uri="tcp://localhost:61635"/>
        </transportConnectors>
</broker>
```



## 0x03、ActiveMQ接收和发送消息

![](/Send_Recv.jpg)

##### 0、添加Maven依赖

```
 <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-all</artifactId>
      <version>5.14.5</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.activemq/activemq-spring -->
    <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-spring</artifactId>
      <version>5.14.5</version>
    </dependency>

```

##### 1、基于JMS方式发送和接收消息

Sender.java

```jmssender
package com.durian.jms;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class MessageSender {

    //发送次数
    public static final int SEND_NUM = 5;
    //tcp地址
    public static final String BROKER_URL = "tcp://localhost:61616";
    //目标，在ActiveMQ管理员控制台创建 http://localhost:8161/admin/queues.jsp
    public static final String DESTINATION = "durian.mq.queue";

    public static void sendMessage(Session session, MessageProducer producer) throws Exception {
        for (int i = 0; i < SEND_NUM; i++) {
            String message = "发送消息第" + (i + 1) + "条";
            TextMessage text = session.createTextMessage(message);

            System.out.println(message);
            producer.send(text);
        }
    }

    public static void run() throws Exception {
        Connection connection = null;
        Session session = null;
        try {
            // 创建链接工厂
            ConnectionFactory factory = new ActiveMQConnectionFactory(ActiveMQConnection.DEFAULT_USER, ActiveMQConnection.DEFAULT_PASSWORD, BROKER_URL);
            // 通过工厂创建一个连接
            connection = factory.createConnection();
            // 启动连接
            connection.start();
            // 创建一个session会话
            session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
            // 创建一个消息队列
            Destination destination = session.createQueue(DESTINATION);
            // 创建消息制作者
            MessageProducer producer = session.createProducer(destination);
            // 设置持久化模式
            producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            sendMessage(session, producer);
            // 提交会话
            session.commit();

        } catch (Exception e) {
            throw e;
        } finally {
            // 关闭释放资源
            if (session != null) {
                session.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        MessageSender.run();
    }

}

```

Receiver.java

```receiver.java
package com.durian.jms;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;


public class MessageReceiver {
    // tcp 地址
    public static final String BROKER_URL = "tcp://localhost:61616";
    // 目标，在ActiveMQ管理员控制台创建 http://localhost:8161/admin/queues.jsp
    public static final String DESTINATION = "durian.mq.queue";

    public static void run() throws Exception {

        Connection connection = null;
        Session session = null;
        try {
            // 创建链接工厂
            ConnectionFactory factory = new ActiveMQConnectionFactory(ActiveMQConnection.DEFAULT_USER, ActiveMQConnection
                    .DEFAULT_PASSWORD, BROKER_URL);
            // 通过工厂创建一个连接
            connection = factory.createConnection();
            // 启动连接
            connection.start();
            // 创建一个session会话
            session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
            // 创建一个消息队列
            Destination destination = session.createQueue(DESTINATION);
            // 创建消息制作者
            MessageConsumer consumer = session.createConsumer(destination);

            while (true) {
                // 接收数据的时间（等待） 100 ms
                Message message = consumer.receive(1000 * 100);

                TextMessage text = (TextMessage) message;
                if (text != null) {
                    System.out.println("接收：" + text.getText());
                } else {
                    break;
                }
            }

            // 提交会话
            session.commit();

        } catch (Exception e) {
            throw e;
        } finally {
            // 关闭释放资源
            if (session != null) {
                session.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        MessageReceiver.run();
    }
}

```

##### 2、Queue队列点对点发送接收消息

```sender.java
package com.durian.queue;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class QueueSender {
    //发送次数
    public static final int SEND_NUM = 5;
    // tcp 地址
    public static final String BROKER_URL = "tcp://localhost:61616";
    //目标队列
    public static final String DESTINATION = "durian.mq.queue";

    public static void run() throws Exception{
        QueueConnection connection = null;
        QueueSession session = null;
        try {
            //创建连接工厂
            QueueConnectionFactory factory = new ActiveMQConnectionFactory(ActiveMQConnectionFactory.DEFAULT_USER,
                        ActiveMQConnectionFactory.DEFAULT_PASSWORD,BROKER_URL);
            //通过工厂创建连接
            connection = factory.createQueueConnection();
            //启动连接
            connection.start();
            //创建一个session会话
            session = connection.createQueueSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
            //创建过一个消息队列
            Queue queue = session.createQueue(DESTINATION);
            //创建消息发送者
            javax.jms.QueueSender sender = session.createSender(queue);
            //设置持久化模式
            sender.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            sendMessage(session, sender);
            //提交会话
            session.commit();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            //关闭释放资源
            if (session != null) {
                session.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }


    public static void sendMessage(QueueSession session, javax.jms.QueueSender sender) throws Exception {
        for (int i = 0; i < SEND_NUM; i++) {
            String message = "发送的消息" + i;
            MapMessage mapMessage = session.createMapMessage();
            mapMessage.setString("text", message);
            mapMessage.setLong("time", System.currentTimeMillis());
            System.out.println(mapMessage);
            sender.send(mapMessage);
        }
    }


    public static void main(String[] args) throws Exception {
        QueueSender.run();
    }

}

```

```receiver.java
package com.durian.queue;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class QueueReceiver {
    // tcp 地址
    public static final String BROKER_URL = "tcp://localhost:61616";
    //目标队列
    public static final String DESTINATION = "durian.mq.queue";

    public static void run() throws Exception {
        QueueConnection connection = null;
        QueueSession session = null;
        try {
            //创建连接工厂
            QueueConnectionFactory factory = new ActiveMQConnectionFactory(ActiveMQConnectionFactory.DEFAULT_USER,
                    ActiveMQConnectionFactory.DEFAULT_PASSWORD,BROKER_URL);
            //通过工厂创建连接
            connection = factory.createQueueConnection();
            //启动连接
            connection.start();
            //创建一个session会话
            session = connection.createQueueSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
            //创建过一个消息队列
            Queue queue = session.createQueue(DESTINATION);
            //创建消息接受者
            javax.jms.QueueReceiver receiver = session.createReceiver(queue);

            //设置消息监听器，实现一部监听
            receiver.setMessageListener(new MessageListener() {
                public void onMessage(Message message) {
                    if (message != null) {
                        MapMessage map = (MapMessage) message;
                        try {
                            System.out.println(map.getLong("time") + "接受#" + map.getString("text"));
                        } catch (JMSException e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
            //休眠100ms再关闭
            Thread.sleep(100);
            //提交会话
            session.commit();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            //关闭释放资源
            if (session != null) {
                session.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        QueueReceiver.run();
    }
}

```



##### 3、Topic主题发布和订阅消息模式

```sender.java
package com.durian.topic;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class TopicSender {
    //发送次数
    public static final int SEND_NUM = 5;
    // tcp 地址
    public static final String BROKER_URL = "tcp://localhost:61616";
    //目标队列
    public static final String DESTINATION = "durian.mq.queue";

    public static void run() throws Exception{
        TopicConnection connection = null;
        TopicSession session = null;
        try {
            //创建连接工厂
            TopicConnectionFactory factory = new ActiveMQConnectionFactory(ActiveMQConnectionFactory.DEFAULT_USER,
                    ActiveMQConnectionFactory.DEFAULT_PASSWORD,BROKER_URL);
            //通过工厂创建连接
            connection = factory.createTopicConnection();
            //启动连接
            connection.start();
            //创建一个session会话
            session = connection.createTopicSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
            //创建过一个消息队列
            Topic topic = session.createTopic(DESTINATION);
            //创建消息发送者
            TopicPublisher publisher = session.createPublisher(topic);
            //设置持久化模式
            publisher.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            sendMessage(session, publisher);
            //提交会话
            session.commit();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            //关闭释放资源
            if (session != null) {
                session.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }


    public static void sendMessage(TopicSession session, TopicPublisher publisher) throws Exception {
        for (int i = 0; i < SEND_NUM; i++) {
            String message = "发送的消息:" + i;
            MapMessage mapMessage = session.createMapMessage();
            mapMessage.setString("text", message);
            mapMessage.setLong("time", System.currentTimeMillis());
            System.out.println(mapMessage);
            publisher.send(mapMessage);
        }
    }


    public static void main(String[] args) throws Exception {
        TopicSender.run();
    }
}

```

```receiver.java
package com.durian.topic;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class TopicReceiver {
    // tcp 地址
    public static final String BROKER_URL = "tcp://localhost:61616";
    //目标队列
    public static final String DESTINATION = "durian.mq.queue";

    public static void run() throws Exception{
        TopicConnection connection = null;
        TopicSession session = null;
        try {
            //创建连接工厂
            TopicConnectionFactory factory = new ActiveMQConnectionFactory(ActiveMQConnectionFactory.DEFAULT_USER,
                    ActiveMQConnectionFactory.DEFAULT_PASSWORD,BROKER_URL);
            //通过工厂创建连接
            connection = factory.createTopicConnection();
            //启动连接
            connection.start();
            //创建一个session会话
            session = connection.createTopicSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
            //创建过一个消息队列
            Topic topic = session.createTopic(DESTINATION);
            //创建消息订阅者
            TopicSubscriber subscriber = session.createSubscriber(topic);
            subscriber.setMessageListener(new MessageListener() {
                public void onMessage(Message message) {
                    if (message != null) {
                        MapMessage map = (MapMessage) message;
                        try {
                            System.out.println(map.getLong("time") + ",接受#" + map.getString("text"));
                        } catch (JMSException e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
            //休眠100ms
            Thread.sleep(100);
            //提交会话
            session.commit();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            //关闭释放资源
            if (session != null) {
                session.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    }



    public static void main(String[] args) throws Exception {
        TopicReceiver.run();
    }
}

```

##### 4、ActiveMQ与Spring整合

```springsender.java
package com.durian.spring;

import org.apache.xbean.spring.context.FileSystemXmlApplicationContext;
import org.springframework.context.ApplicationContext;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;

import javax.jms.JMSException;
import javax.jms.MapMessage;
import javax.jms.Message;
import javax.jms.Session;
import java.util.Date;

public class SpringSender {
    public static void main(String[] args) {
        ApplicationContext context = new FileSystemXmlApplicationContext("classpath:spring.xml");
        JmsTemplate jmsTemplate = (JmsTemplate) context.getBean("jmsTemplate");

        jmsTemplate.send(new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                MapMessage mapMessage = session.createMapMessage();
                mapMessage.setString("message", "current system time:" + new Date());
                return mapMessage;
            }
        });
    }
}

```

```springrecv.java
package com.durian.spring;

import org.apache.xbean.spring.context.FileSystemXmlApplicationContext;
import org.springframework.context.ApplicationContext;
import org.springframework.jms.core.JmsTemplate;

import java.util.Map;

public class SpringReceiver {
    public static void main(String[] args) {
        ApplicationContext context = new FileSystemXmlApplicationContext("classpath:spring.xml");
        JmsTemplate jmsTemplate = (JmsTemplate) context.getBean("jmsTemplate");

        while (true) {
            Map<String, Object> map = (Map<String, Object>) jmsTemplate.receiveAndConvert();
            System.out.println("接受到的消息:" + map.get("message"));
        }
    }
}

```

```spring.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.1.xsd">

    <!-- 连接池  -->
    <bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <property name="connectionFactory">
            <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                <property name="brokerURL" value="tcp://localhost:61616"/>
            </bean>
        </property>
    </bean>

    <!-- 连接工厂 -->
    <bean id="activeMQConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://localhost:61616"/>
    </bean>

    <!-- 配置消息目标 -->
    <bean id="destination" class="org.apache.activemq.command.ActiveMQQueue">
        <!-- 目标，在ActiveMQ管理员控制台创建 http://localhost:8161/admin/queues.jsp -->
        <constructor-arg index="0" value="durian.mq.queue"/>
    </bean>

    <!-- 消息模板 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="activeMQConnectionFactory"/>
        <property name="defaultDestination" ref="destination"/>
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>
</beans>

```





##### 参考链接：

[ActiveMQ入门教程]: https://www.jianshu.com/p/8caa6d66b10d
[成小胖学习ActiveMQ·基础篇]: http://www.cnblogs.com/cyfonly/p/6380860.html

