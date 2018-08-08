---
title: RabbitMQ入门教程
date: 2018-08-08 18:48:06
tags:
	- 消息队列
	- RabbitMQ
---

## 0x01、MQ简介

##### 1、有哪些常用消息队列？

ActiveMQ、RabbitMQ、Kafka、RocketMQ

![](/1226944-20180625122436309-378812433.png)

<!-- more -->

##### 2、RabbitMQ是什么？

​	RabbitMQ is a message broker: it accepts and forwards messages. 

##### 3、RabbitMQ适用哪些场景？

​	适合比较耗时且不急着需要返回响应的场景，在实际项目中遇到性能瓶颈才考虑使用MQ，否则会减低系统的可用性，增加维护难度和开发成本。

##### 4、RabbitMQ的ACK机制？

[应答和确认机制]: https://blog.csdn.net/vbirdbest/article/details/78699913

1）、设置是否自动应答：

```
// 第二个参数为设置是否自动应答
boolean autoAck = true;
channel.basicConsume(QUEUE_NAME, autoAck, consumer);
```

2）、拒绝应答

```
// requeue：重新入队列，false：直接丢弃，相当于告诉队列可以直接删除掉
channel.basicReject(envelope.getDeliveryTag(), false);
```

3)、重新投递

```
// If true, messages will be requeued and possibly delivered to a different consumer. If false, messages will be  redelivered to the same consumer.
channel.basicRecover(true);
```

4）、手动应答

```
/** true to acknowledge all messages up to and
     * including the supplied delivery tag; false to acknowledge just
     * the supplied delivery tag.
     */
channel.basicAck(envelope.getDeliveryTag(), false);
```



## 0x02、RabbitMQ环境搭建

#### 1、安装GCC 等模块

```
yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel 
```

#### 2、安装ncurses

```
yum -y install ncurses-devel 
```

#### 3、安装erlang环境

```
# yum install erlang  版本不对出现错误，需要移除erlang
# yum remove erlang

-- 下载解压配置erlang
# wget http://erlang.org/download/otp_src_20.3.tar.gz
# tar -xf otp_src_20.3.tar.gz  

-- 安装相关依赖项 
# yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel unixODBC-devel libtool libtool-ltdl-devel

-- 编译erlang
# cd oto_src_20.3
# ./configure --prefix=/opt/erlang
# make && make install

-- 验证erlang安装成功
# cd /opt/erlang
1>
表示安装成功
```

#### 4、下载安装RabbitMQ

```
-- 下载
# weget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-generic-unix-3.6.1.tar.xz
-- 解压
# xz -d rabbitmq-server-generic-unix-3.6.1.tar.xz
# tar -xvf rabbitmq-server-generic-unix-3.6.1.tar

--配置环境变量
# vi /etc/profile
# export PATH=$PATH:/opt/rabbitmq/sbin
#  source  /etc/profile   使得文件生效
```

#### 5、启动rabbitmq

```
cd rabbitmq/sbin
./rubbitmq-server start
```

#### 6、访问rabbitmq后台管理

```
localhost:15672
```

#### 7、添加用户

```
cd rabbitmq/sbin/
./rabbitmq-server start   # 开启rabbitmq服务
./rabbitmqctl add_user admin admin  # 创建用户
./rabbitmqctl set_user_tags admin administrator  # 设置新账号为超级管理员
./rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"   #设置用户权限
./rabbitmq-plugins enable rabbitmq_management   #开启web界面管理工具
```



## 0x03、RabbitMQ的6中模式

#### 1、添加RabbitMQ依赖

```
<dependencies>
<!-- https://mvnrepository.com/artifact/com.rabbitmq/amqp-client -->
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.1.2</version>
        </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.25</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.3.2</version>
    </dependency>
    <dependency>
      <groupId>org.glassfish.web</groupId>
      <artifactId>javax.servlet.jsp.jstl</artifactId>
      <version>1.2.2</version>
    </dependency>
  </dependencies>
```



#### 2、模式一Hello-World

 a producer that sends a single message, and a consumer that receives messages and prints them out.  

![](/python-one.png)

Hello-World模式简介：生产者发送一个消息到MQ中，消费者从MQ中获取消息。

```send
package com.durian.hello;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Send {
    private final static String QUEUE = "hello";

    public static void main(String[] args) throws Exception {
        // 创建连接
        Connection connection = ConnectionUtil.getConnection();

        //创建通道
        Channel channel = connection.createChannel();

        //声明队列
        /**
         * param1:队列名字
         * param2:是否开启持久化
         * param3:是否排他
         * param4：参数
         */
        channel.queueDeclare(QUEUE, false, false, false, null);
        String message = "Hello World";
        channel.basicPublish("", QUEUE, null, message.getBytes());
        System.out.println("[X] Sent'" + message + "'");

        //关闭连接
        channel.close();
        connection.close();
    }
}
```

```recv
package com.durian.hello;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv {
    private final static String QUEUE = "hello";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        /**
         * 申明队列，如果队列存在则什么都不做，如果不存在才创建
         * 参数1：队列名字
         * 参数2：是否持久化队列，我们的队列模式是在内存中的，如果rabbimq重启会丢失，如果设置为true则会保存到erlang自带的数据库中，重启会重新读取数据
         * 参数3：是否排外，有两个作用，第一个当我们的连接关闭后是否会自动删除队列，作用二：私有当前队列，如果私有了其他通道不可以访问当前队列，如果为true,一般是一个队列只适用于一个消费者使用
         * 参数4：是否自动删除
         * 参数5：我们的一些其他参数
         */
        channel.queueDeclare(QUEUE, false, false, false, null);
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [x] Received '" + message + "'");
            }
        };

        //参数二自动确认
        channel.basicConsume(QUEUE, true, consumer);

//        channel.close();
//        connection.close();

    }

}


```

#### 3、模式二Work Queues

we wrote programs to send and receive messages from a named queue. In this one we'll create a *Work Queue* that will be used to distribute time-consuming tasks among multiple workers. 

![](/python-two.png)

Work Queues模式简介：生产者将消息发送到工作队列中，消息队列可以将消息分发给多个消费者来处理耗时任务。

Tips:

​	可以使用 channel.basicQos(1);  //告诉服务器在我确认之前不要给我发新的消息

```send
package com.durian.workqueues;


import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;

public class Send {
    private final static String QUEUE ="task_work";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE, false, false, false, null);
        for (int i = 0; i < 100; i++) {
            channel.basicPublish("" , QUEUE, null, ("消息" + i ).getBytes());
        }
        //关闭连接
        channel.close();
        connection.close();
    }

}
```

```recv1
package com.durian.workqueues;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv1 {
    private final static String QUEUE = "task_work";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE, false, false, false, null);
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [Consumer1] Received '" + message + "'");
                try {
                    Thread.sleep(4);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 参数二，false为确认收到消息，true为拒绝收到消息
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };

        // 注册消费者，参数2false为手动确认，代表我们收到消息后需要手动告诉服务器，我收到消息了
        channel.basicConsume(QUEUE, false, consumer);
    }

}
```

```recv2
package com.durian.workqueues;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv2 {
    private final static String QUEUE = "task_work";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE, false, false, false, null);
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");
        channel.basicQos(1);  //告诉服务器在我确认之前不要给我发新的消息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [Consumer2] Received '" + message + "'");
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 参数二，false为确认收到消息，true为拒绝收到消息
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };

        // 注册消费者，参数2为手动确认，代表我们收到消息后需要手动告诉服务器，我收到消息了
        channel.basicConsume(QUEUE, false, consumer);
    }

}
```

#### 4、模式三Publish/Subscribe

In the [previous tutorial](http://www.rabbitmq.com/tutorials/tutorial-two-java.html) we created a work queue. The assumption behind a work queue is that each task is delivered to exactly one worker. In this part we'll do something completely different -- we'll deliver a message to multiple consumers. This pattern is known as "publish/subscribe". 

![](/python-three-overall.png)

发布者和订阅者模式简介：该模式生产者绑定一个交换机，将消息发送到交换机，一个交换机可以对应多个队列，消费者创建队列并绑定发布者对应的交换机，当发布者发布消息时，订阅者可以接受到对应的消息。

```send
package com.durian.publish;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;

import javax.swing.event.ChangeListener;

public class Send {
    private final static String EXCHANGE_NAME = "Exchange";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        //声明一个交换机，类型为fanout时，表示为发布订阅模式
        channel.exchangeDeclare(EXCHANGE_NAME , "fanout");
        //发布订阅模式，消息先发送到交换机中，交换机没有保存功能，如果没有消费者，消息会丢失
        channel.basicPublish(EXCHANGE_NAME, "", null, "发布者发布的消息...".getBytes());
        channel.close();
        connection.close();
    }

}
```

```recv1
package com.durian.publish;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import javax.swing.event.ChangeListener;
import java.io.IOException;

public class Recv1 {
    //定义交换机名字,一个交换机可以绑定多个队列
    private final static String EXCHANGE_NAME = "Exchange";
    //声明队列
    private final static String QUEUE_NAME = "PublishQueue1";
    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();

        //申明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //绑定队列到交换机当中
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
        channel.basicQos(1);
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("消费者1:" + new String(body));
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

```recv2
package com.durian.publish;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv2 {
    //定义交换机名字
    private final static String EXCHANGE_NAME = "Exchange";
    //声明队列
    private final static String QUEUE_NAME = "PublishQueue2";
    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();

        //申明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //绑定队列到交换机当中
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
        channel.basicQos(1);
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                System.out.println("消费者2:" + new String(body));
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);
    }
}
```

#### 5、模式四Routing

In this tutorial we're going to add a feature to it - we're going to make it possible to subscribe only to a subset of the messages. For example, we will be able to direct only critical error messages to the log file (to save disk space), while still being able to print all of the log messages on the console. 

![](/direct-exchange.png)

Routin模式简介：该模式在P/S模式上加了一个RoutingKey，生产者在发布消息的时候会指定一个routingKey，消费者会在队列上绑定一个routingKey（一个队列可以绑定多个key），只有队列上绑定的key包含生产者发布消息指定的key时才能接受消息。

```send
package com.durian.router;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;

public class Send {
    private final static String EXCHANGE_NAME = "router";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();

        //定义路由形式的交换机,
        //消费者只有绑定的交换机且key值相等才能收到发送者的消息
        channel.exchangeDeclare(EXCHANGE_NAME, "direct");
        channel.basicPublish(EXCHANGE_NAME, "key1", null, "路由消息".getBytes());

        channel.close();
        connection.close();
    }
}

```

```recv1
package com.durian.router;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv1 {
    private final static String EXCHANGE_NAME = "router";
    private final static String QUEUE_NAME = "routerQueue";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //绑定队列到交换机中
        // 参数3：标记，绑定交换机会指定一个标记，必须和发送者绑定的标记一致才能接受到消息
        //可接受多个标记，多次绑定即可
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "key1");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "key2");

        channel.basicQos(1);

        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                System.out.println("消费者1" + new String(body));
                //手动确定接受到消息
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);

    }
}

```

```recv2
package com.durian.router;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv2 {
    private final static String EXCHANGE_NAME = "router";
    private final static String QUEUE_NAME = "routerQueue";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //绑定队列到交换机中
        // 参数3：标记，绑定交换机会指定一个标记，必须和发送者绑定的标记一致才能接受到消息
        //可接受多个标记，多次绑定即可
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "key3");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "key2");

        channel.basicQos(1);

        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                System.out.println("消费者1" + new String(body));
                //手动确定接受到消息
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);

    }
}

```

#### 6、模式五Topic

In the [previous tutorial](http://www.rabbitmq.com/tutorials/tutorial-four-java.html) we improved our logging system. Instead of using a fanout exchange only capable of dummy broadcasting, we used a direct one, and gained a possibility of selectively receiving the logs. 

![](/python-five.png)

Topic模式简介：该模式相对Routing模式添加了占位符匹配Key，# 可以匹配任意个字符，*只能匹配一个字符。

```send
package com.durian.topic;


import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;

public class Send {
    private final static String EXCHANGE_NAME="topic";//定义交换机的名称
    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        //声明为topic模式
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        //尝试绑定key:  abc.1,abc.1.2,key.1,key.1.2
        channel.basicPublish(EXCHANGE_NAME, "abc.1", null, "topic模式消息".getBytes());
        channel.close();
        connection.close();
    }
}

```

```recv1
package com.durian.topic;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv1 {
    private final static String EXCHANGE_NAME = "topic";
    private final static String QUEUE_NAME = "topicQueue1";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();

        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //绑定队列到交换机中
        // 参数3：标记，绑定交换机会指定一个标记，必须和发送者绑定的标记一致才能接受到消息
        //可接受多个标记，多次绑定即可
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "Key.*");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "abc.#");

        channel.basicQos(1);
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                System.out.println("消费者1:" + new String(body));
                //手动确定接受到消息
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);

    }
}

```

```recv2
package com.durian.topic;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv2 {
    private final static String EXCHANGE_NAME = "topic";
    private final static String QUEUE_NAME = "topicQueue2";

    public static void main(String[] args) throws Exception {
        Connection connection = ConnectionUtil.getConnection();
        final Channel channel = connection.createChannel();
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //绑定队列到交换机中
        // 参数3：标记，绑定交换机会指定一个标记，必须和发送者绑定的标记一致才能接受到消息
        //可接受多个标记，多次绑定即可
        //# 可以匹配多个字符，*号只能匹配一个字符
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "Key.#");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "abc.#");

        channel.basicQos(1);

        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                System.out.println("消费者2:" + new String(body));
                //手动确定接受到消息
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };
        channel.basicConsume(QUEUE_NAME, false, consumer);

    }
}

```

#### 7、模式六RPC

But what if we need to run a function on a remote computer and wait for the result? Well, that's a different story. This pattern is commonly known as *Remote Procedure Call* or *RPC*. 

![](/python-six.png)

RPC(Remote Programing Call)简介：客户端将请求发送到队列中，参数中会携带一个ID；Server从队列中获取请求并处理，将结果response和ID一并传到reply队列，客户端冲reply队列中获取和ID对应的response消息。

```rpcclient
package com.durian.rpc;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Envelope;

import java.io.IOException;
import java.util.UUID;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeoutException;

public class RPCClient {

    private Connection connection;
    private Channel channel;
    private String requestQueueName = "rpc_queue";
    private String replyQueueName;

    public RPCClient() throws Exception {
//        ConnectionFactory factory = new ConnectionFactory();
//        factory.setHost("192.168.31.164");

//        connection = factory.newConnection();
        //创建连接和通道
        connection = ConnectionUtil.getConnection();
        channel = this.connection.createChannel();
        //声明队列
        replyQueueName = channel.queueDeclare().getQueue();
    }

    public String call(String message) throws IOException, InterruptedException {
        //获取一个唯一请求ID,用于结果返回判断
        final String corrId = UUID.randomUUID().toString();
        //设置参数
        AMQP.BasicProperties props = new AMQP.BasicProperties
                .Builder()
                .correlationId(corrId)
                .replyTo(replyQueueName)
                .build();

        channel.basicPublish("", requestQueueName, props, message.getBytes("UTF-8"));

        final BlockingQueue<String> response = new ArrayBlockingQueue<String>(1);

        channel.basicConsume(replyQueueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                if (properties.getCorrelationId().equals(corrId)) {
                    response.offer(new String(body, "UTF-8"));
                }
            }
        });

        return response.take();
    }

    public void close() throws IOException {
        connection.close();
    }

    public static void main(String[] argv) {
        RPCClient fibonacciRpc = null;
        String response = null;
        try {
            fibonacciRpc = new RPCClient();

            System.out.println(" [x] Requesting fib(6)");
            response = fibonacciRpc.call("6");
            System.out.println(" [.] Got '" + response + "'");
        }
        catch  (IOException | TimeoutException | InterruptedException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (fibonacciRpc!= null) {
                try {
                    fibonacciRpc.close();
                }
                catch (IOException _ignore) {}
            }
        }
    }
}

```

```rpcservler
package com.durian.rpc;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Envelope;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class RPCServer {

    private static final String RPC_QUEUE_NAME = "rpc_queue";

    private static int fib(int n) {
        if (n ==0) return 0;
        if (n == 1) return 1;
        return fib(n-1) + fib(n-2);
    }

    public static void main(String[] argv) {
//        ConnectionFactory factory = new ConnectionFactory();
//        factory.setHost("192.168.31.164");
//
            Connection connection = null;
        try {
            connection      = ConnectionUtil.getConnection();
            final Channel channel = connection.createChannel();

            channel.queueDeclare(RPC_QUEUE_NAME, false, false, false, null);

            channel.basicQos(1);

            System.out.println(" [x] Awaiting RPC requests");

            Consumer consumer = new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    AMQP.BasicProperties replyProps = new AMQP.BasicProperties
                            .Builder()
                            .correlationId(properties.getCorrelationId())
                            .build();

                    String response = "";

                    try {
                        String message = new String(body,"UTF-8");
                        int n = Integer.parseInt(message);

                        System.out.println(" [.] fib(" + message + ")");
                        response += fib(n);
                    }
                    catch (RuntimeException e){
                        System.out.println(" [.] " + e.toString());
                    }
                    finally {
                        channel.basicPublish( "", properties.getReplyTo(), replyProps, response.getBytes("UTF-8"));
                        channel.basicAck(envelope.getDeliveryTag(), false);
                        // RabbitMq consumer worker thread notifies the RPC server owner thread
                        synchronized(this) {
                            this.notify();
                        }
                    }
                }
            };

            channel.basicConsume(RPC_QUEUE_NAME, false, consumer);
            // Wait and be prepared to consume the message from RPC client.
            while (true) {
                synchronized(consumer) {
                    try {
                        consumer.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (connection != null)
                try {
                    connection.close();
                } catch (IOException _ignore) {}
        }
    }
}

```

#### 8、持久化

简介：生产者发送的消息会持久化到erlang的数据库中，当生产者宕机后，消息可以从磁盘中加载到队列中，以便消费者来消费。

```send
package com.durian.persistence;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.MessageProperties;

public class Send {
    private static String EXCHANGE_NAME="testpersist";
    public static void main(String[] args) throws Exception {
        Connection connection= ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        //申明一个持久的交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "direct",true,false,null);
        // MessageProperties.PERSISTENT_TEXT_PLAIN初久化消息
        //申明持久化消息
        channel.basicPublish(EXCHANGE_NAME, "abc", MessageProperties.PERSISTENT_TEXT_PLAIN, "持久化的消息".getBytes());
        channel.close();
        connection.close();
    }
}

```

```recv
package com.durian.persistence;

import com.durian.utils.ConnectionUtil;
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv {
    private static String EXCHANGE_NAME="testpersist";
    private static String QUEUE_NAME="testpersistqueue";
    public static void main(String[] args) throws Exception {
        Connection connection= ConnectionUtil.getConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "direct",true,false,null);
        //申明持久化的队列
        channel.queueDeclare(QUEUE_NAME,true,false,false,null);
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME,"abc");
        Consumer consume=new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                System.out.println("收到消息:"+new String(body));
            }
        };
        channel.basicConsume(QUEUE_NAME, true,consume);

    }
}

```

#### 9、生产者确认

当生产者发布消息到RabbitMQ中，生产者需要知道是否真的已经发送到RabbitMQ中，需要RabbitMQ告诉生产者。

- Confirm机制 
  channel.confirmSelect(); 
  channel.waitForConfirms();
- 事务机制 
  channel.txSelect(); 
  channel.txRollback();

注意：事务机制是非常非常非常消耗性能的，最好使用Confirm机制，Confirm机制相比事务机制性能上要好很多。

```
//Confirm机制 
channel.confirmSelect();
String message = "Hello RabbitMQ:";
for (int i = 0; i < 5; i++) {
    channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, null, (message + i).getBytes("UTF-8"));
}
boolean isAllPublished = channel.waitForConfirms();123456


//事务机制
String message = "Hello RabbitMQ:";
try {
    channel.txSelect();
    for (int i = 0; i < 5; i++) {
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, null, (message + i).getBytes("UTF-8"));
    }
    channel.txCommit();
} catch (Exception e) {
    channel.txRollback();
}
```



## 0x04、Spring整合RabbitMQ

#### 1、Pom.xml

```pom.xml
	<dependencies>
		<!--rabbitmq依赖-->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.7</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.rabbitmq/amqp-client -->
		<!--使用稳定版本，之前使用5.3.0报错，可能存在版本不一致-->
		<dependency>
			<groupId>com.rabbitmq</groupId>
			<artifactId>amqp-client</artifactId>
			<version>4.5.0</version>
		</dependency>

		<!--rabbitmq与spring整合需要的依赖-->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>5.0.6.RELEASE</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.springframework.amqp/spring-rabbit -->
		<dependency>
			<groupId>org.springframework.amqp</groupId>
			<artifactId>spring-rabbit</artifactId>
			<version>1.7.5.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>4.3.7.RELEASE</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.25</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
```

#### 2、Spring.xml配置文件

```spring.xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/rabbit
                           http://www.springframework.org/schema/rabbit/spring-rabbit-1.7.xsd
                           http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
    <!-- 1、定义连接工厂 -->
    <rabbit:connection-factory id="connectionFactory" host="192.168.31.164" port="5672" username="admin"
                               password="admin" virtual-host="/durian"/>
    <!--2、定义rabbitmq的模版
            如果发送到队列，添加queue=""
            如果发送到交换机，添加exchange=""
            定义路由绑定routing-key=""         -->
    <rabbit:template id="template" connection-factory="connectionFactory" exchange="fanoutExchange"/>
    <rabbit:admin connection-factory="connectionFactory"/>

    <!--3、消息发送到交换机还是消息队列-->
    <!--4、定义队列-->
    <rabbit:queue name="myQueue" auto-declare="true"/>
    <!--5、如果发送到交换机，定义交换机-->
    <rabbit:fanout-exchange name="fanoutExchange" auto-declare="true">
        <!--将队列绑定到交换机-->
        <rabbit:bindings>
            <rabbit:binding queue="myQueue"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:fanout-exchange>

    <!--6、定义一个消费者-->
    <bean id="consumer" class="com.durian.spring.MyConsumer"/>
    <!--7、定义监听器容器，当收到消息的时候就会执行内部配置-->
    <rabbit:listener-container connection-factory="connectionFactory">
        <!--定义哪个类里哪个方法用于处理接受到的消息-->
        <rabbit:listener ref="consumer" method="test" queue-names="myQueue"/>
    </rabbit:listener-container>


    <!-- 扩展：路由模式
    <rabbit:direct-exchange name="directExchange" durable="true" auto-delete="false">
        <rabbit:bindings>
            <rabbit:binding queue="myQueue" key="key1"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:direct-exchange>-->
    <!-- 扩展：topic模式
    <rabbit:topic-exchange name="topicExchange" durable="true" auto-delete="false">
        <rabbit:bindings>
            <rabbit:binding pattern="dfasfsd.*" queue="myQueue"/>
        </rabbit:bindings>
    </rabbit:topic-exchange>-->

</beans>
```

#### 3、消费者类

```consumer
package com.durian.spring;

public class MyConsumer {

    public void test(String message) {
        System.out.println("消费者接受到的消息：");
        System.out.println("================" + message);
    }
}

```



#### 4、测试类

```test
package com.durian.spring;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {

    public static void main(String[] args) throws Exception{
        ApplicationContext context=new ClassPathXmlApplicationContext("classpath:spring-autoconfirm.xml");
        RabbitTemplate template = context.getBean(RabbitTemplate.class);
        template.convertAndSend("Spring的消息");
        ((ClassPathXmlApplicationContext)context).destroy();
    }
}

```



