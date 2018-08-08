---
title: zookeeper入门教程
date: 2018-08-08 18:54:21
tags:
	- zookeeper
---

## 0x01、Zookeeper安装

1）、下载安装

​	从官网上下载稳定版本，

[zookeeper 下载]: https://archive.apache.org/dist/zookeeper/

2）、目录结构

- bin：主要的运行命令

- conf：存放配置文件

  <!-- more-->

- contrib：附加的一些功能

- dist-maven：mvn编译后的目录

- docs：文档

- lib：需要依赖的jar

- recipes：案列demo代码

- src：源码

3）、配置文件配置

zoo.cfg

```
tickTime：用于计算的时间单元。比如session超时：N*tickTime
initLimit:用于集群，允许从节点同步到主节点的初始化连接时间
syncLimit：用于集群，主节点和从节点消息通信，请求和应答是时间长度，心跳机制
dataDir：必须配置。 
	dataDir=/usr/local/zookeeper/dataDir
	创建DataDir和dataLogDir
dataLogDir：日志目录，如果不配公用dataDir
clientPort: 连接服务器的端口2181
```



4）、启动

```
./zkServer.sh   查看帮助
./zkServer.sh start
```



## 0x02、基本数据模型

树、目录结构

- 每一个节点称为znode，可以有子节点，也可以有数据
- 节点分为临时节点、永久节点，临时节点在客户端断开后消失
- 每一个zk节点都有各自版本号，节点数据发生变化，版本号累加，类似乐观锁
- 删除/修改过时节点，版本号不匹配会报错
- 每个zk节点存储的数据不宜过大，几K即可
- 节点可以设置权限acl，可以通过权限限制用户访问



## 0x03、zookeeper数据模型的基本操作

- 客户端连接
- 查看zk节点
- 关闭客户端连接

```
./zkServer.sh start  启动zk
./zkCli  启动客户端
	[zk:localhsot:2181(Connected) 1]:help/ls/CTRL＋ｃ

```

## 0x04、zk作用体现

- master节点选举，保证集群高可用

- 统一配置文件管理，只需要部署一台服务器，则可以把相同的配置文件同步更新到其他所有服务器（如修改redis统一配置）

- 发布与订阅，类似消息队列，发布者把数据存储在znode上，订阅者读取这个数据。

- 提供分布式锁，分布式环境中不同进程中竞争资源，类似多线程中的锁。

- 集群管理，集群中保证数据的强一致性 

  

  ![](/%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.png)

## 0x05、常用的命令行

```
./zkCli.sh
# ls 和 ls2
ls /
ls /zookeeper
ls2 /   是ls和stat整合的一个命令

# stat 和 get 
stat /   
get  /   取出当前节点存储的数据
	cZxid
	ctime 
	mzxid
	mtime
	pzxid  父节点id
	
create -e  /var value    创建的是临时节点，临时节点不能创建子节点
create -e -s /var value	 创建的是临时有序节点
create /demo	创建的是持久节点
create -s /demo		创建的是持久有序节点

set path data [version(表示操作路径的版本号，如果版本号过时会报错)]

delete path [version]
```



## 0x06、Session基本原理

- 客户端和服务端之间的连接存在会话
- 每个会话可以设置超时时间
- 心跳结束，session过期
- session过期临时节点删除
- 心跳机制：客户端相服务端ping



## 0x07、Watcher机制

##### **1）、简介**

- 针对每个节点的操作，都会有一个监督者 ---> Watch

- 当监控的对象znode发生了变化，则触发watcher事件

- zk中的watcher是一次性的，触发后立即销毁

- 父节点、子节点增删改都能够触发其watcher

- 针对不同的操作，触发的watcher事件也不同：

  ​	（子）节点创建事件

  ​	（子）节点删除事件

  ​	（子）节点数据变化事件

##### **2）、watcher命令行学习**

- 通过get path [watcher] 设置watcher
- 父节点增删改查触发watcher
- 子节点增删改触发watcher

##### 3）、watcher事件类型

```
# 创建watch，watch是一次性的，触发后就销毁
ls path watch
get path watch
set path data watch
```

- 创建父节点： NodeCreaded 
- 修改父节点：NodeDataChanged
- 删除父节点：NodeDelete
- ls为父节点设置watch，创建/删除子节点触发：NodeChildrenChanged
- 修改子节点不会触发任何事件，如需要触发watch，在子Node上创建watch。

##### 4）、watcher使用场景

- 统一资源配置

![](/%E7%BB%9F%E4%B8%80%E9%85%8D%E7%BD%AE.png)

## 0x08、ACL（Access Control Lists）权限控制

##### 1）、简介

- 针对节点可以设置相关的读写权限，保证数据安全性
- 权限permissions可以指定不同的权限范围以及角色

##### 2）、ACL命令行

world:anyone : cdrwa

```
getAcl path
setAcl path world:anyone:crwa    不能删除子节点

addauth digest imooc:imooc
setAcl path auth:imooc:imook:c:cdrwa   imooc为username和pwd,加密存储
setAcl path auth::cdwa  默认使用degist用户名和密码

setAcl path digest:imooc:imooc加密后:cdrwa   设置为密文
addauth digest imooc:imooc  登陆  

setACl path ip:192.168.31.164:cdrwa


```

- getAcl path：获取某个节点的acl权限信息
- setAcl path acl：设置某个节点的acl权限信息
- addauth：输入认证授权信息，注册时输入明文密码，但是在zk系统里，密码是 以加密的形式存在的。

##### 3）、ACL构成

- zk的acl通过[scheme:ip:permissions]来构成权限列表

  - scheme：代表采用搞得权限机制
  - id：代表访问的用户
  - permisssions：权限组合字符串

- **scheme**

  - world:world下只有一个id，即只有一个用户，也就是anyone，那么组合的写法就是 word:anyone:[permissions]
  - auth：代表认证登陆，需要注册用户有权限可以登陆，形式为auth:user:password:[permissions]
  - digest：需要对密码加密才能访问，组合形式为 digest:username:BASE64(SHA1(password)):[permissions]
  - auth与digest的区别是前者是明文，后者是密文；setAcl /path auth:lee:lee:cdrwa 和setAcl /path digest:lee：BASE64(SH1(password))cdrwa是等价的，在通过addauth digest lee:lee后都能操作指定节点的权限

- **ip**：设置ip为指定ip地址，此时限制ip进行访问，比如ip:192.168.1.1:[permisssions]

  - super：代表超级管理员，拥有所有权限

    ```
    修改zkSe rver.sh，添加super用户
    当对某些节点没权限的时候，可以登陆super
    具体设置如下图：
    "-Dzookeeper.DigestAuthenticationProvider.superDigest=durian:3WVkm1FoyH3pOxMjVg5XnhTAAfc="
    重启zkServer.sh
    ```

    ![](/zkServer.png)

- **permissions**：权限字符串缩写crdwa

  - Create ：创建子节点
  - Read : 获取节点/子节点
  - Write：设置节点数据
  - Delete：删除子节点
  - Admin：设置权限



##### 4）、ACL的常用使用场景

- 开发/测试环境分离
- 生产环境上控制指定ip的服务可以访问相关节点，防止混乱

## 0x09、zookeeper的四字命令

- zk可以通过自身的简写命令来和服务器进行交互
- 需要使用nc命令，安装：yum install nc
- echo [commond] | nc ip port

```
stat  查看zk状态
ruok  查看zkServer是否启动，返回imok
dump  列出未经处理的会话和临时节点
conf  查看服务器相关配置
cons  展示连接到服务器的客户端信息
envi  环境变量
mntr  监控zk的健康状况
wchs  展示watch的信息
#  配置：zoo.cfg中添加41w.commands.whitelist=*
wchc 和 wchp  session与path，path和session信息
```



## 0x10、zookeeper集群搭建

- zk集群，主从节点，心跳机制（选举模式）
- 配置数据文件 myid 1/2/3 对应 server.1/2/3
- 通过./zkCli.sh -server ip : port  检测集群是否配置成功

##### 1)、配置本地集群

- 复制三个zookeeper

- 在zoo.cfg中添加一下配置，并更改端口号

  ```
  clientPort=2183   更改端口
  server.1=192.168.31.164:2888:3888
  server.2=192.168.31.164:2889:3889
  server.3=192.168.31.164:2890:3890
  ```

- 在每个dateDir目录下创建一个myid文件内容为对应的1/2/3

2）、连接zookeeper客户端

```
./zkCli.sh -server ip:port
```



## 0x11、zookeeper原生Java api使用

- 会话连接与恢复
- 节点的增删改查
- watch与acl的相关操作

##### 1）、添加jar包依赖

![](/jar.png)

```
 <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.4.1</version>
      <!--type标签要删除，否则可能引用不到该依赖-->
      <!--<type>pom</type>-->
 </dependency>
```



##### 2）、关闭防火墙

```
CentOS7
查看防火墙状态：
firewall-cmd --state
停止防火墙
systemctl stop/start firewalld.service
禁止防火墙开机启动
systemctl disable/enable firewalld.service
开启端口
firewall-cmd --zone=public --add-port=80/tcp --perment
```

```
CentOS7以下
查看防火墙状态：
service iptables status
停止防火墙
service iptables stop/start
禁止防火墙开机启动
chkconfig iptables on/off
```

##### 3、zookeeper的Java Api调用

[java调用zk的Api]: https://www.cnblogs.com/leesf456/p/6028416.html

- 增删改等都有同步和异步操作，同步操作需要try...catch异常，异步调用需要传递一个callback接口，异常或者返回结果在回调接口中执行。
- watch从zk传递到java需要时间，所以在main后面添加了sleep()
- 权限授予，对于有权限设置的节点，必须授权后才能操作。

```
package com.durian.zk.demo;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.util.List;
import java.util.concurrent.CountDownLatch;

public class ZKCreateNodeAsync implements Watcher{
    private static CountDownLatch countDownLatch = new CountDownLatch(1);
    private static ZooKeeper zooKeeper = null;
    private static int num = 1;
    public static void main(String[] args) throws Exception {
        zooKeeper = new ZooKeeper("192.168.31.164:2181", 5000, new ZKCreateNodeAsync());
        System.out.println("客户端连接状态：" + zooKeeper.getState());
        countDownLatch.await();
        System.out.println("客户端连接状态：" + zooKeeper.getState());

        /**
         * 同步或者异步创建节点，都不支持子节点的递归创建，异步有一个callback函数
         * 参数：
         * path：创建的路径
         * data：存储的数据的byte[]
         * acl：控制权限策略
         * 			Ids.OPEN_ACL_UNSAFE --> world:anyone:cdrwa
         * 			CREATOR_ALL_ACL --> auth:user:password:cdrwa
         * createMode：节点类型, 是一个枚举
         * 			PERSISTENT：持久节点
         * 			PERSISTENT_SEQUENTIAL：持久顺序节点
         * 			EPHEMERAL：临时节点
         * 			EPHEMERAL_SEQUENTIAL：临时顺序节点
         * 	异步创建没有返回值，只有回调函数
         */

       //同步创建
//        try {
//            zooKeeper.create("/test", "".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
//        } catch (KeeperException e) {
//            System.out.println("节点已存在");
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }

        //异步创建节点
//       zooKeeper.create("/test", "11".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL,
//                    new IStringCallback(), "{state: success}");
//       Thread.sleep(2000);

//        try {
//            //同步删除
//            zooKeeper.delete("/test", -1);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        } catch (KeeperException e) {
//            System.out.println(e.getPath() + "is not empty");
//        }



        //异步删除
//        zooKeeper.delete("/test", -1, new IVoidCallback(), "ctx");

        //同步获取子节点
        List<String> children = zooKeeper.getChildren("/test", true);
        System.out.println("/test的子节点：" + children);

        zooKeeper.create("/test/var", "var".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        zooKeeper.create("/test/var1", "var1".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        zooKeeper.delete("/test/var1", 0);

        zooKeeper.setData("/test/var", "i change var data".getBytes(), 0);
        Stat stat = new Stat();
        byte[] data = zooKeeper.getData("/test/var", true, stat);
        System.out.println("/test/var的信息为：" + new String(data));
        System.out.println("stat的信息为：" + stat);
        Thread.sleep(2000);
        //授权
        zooKeeper.addAuthInfo("digest", "durian:durian".getBytes());
        String s = zooKeeper.create("/durian", "i am 66".getBytes(), ZooDefs.Ids.CREATOR_ALL_ACL, CreateMode.EPHEMERAL);
        System.out.println("创建durian返回的字符串：" + s);

        zooKeeper = new ZooKeeper("192.168.31.164:2181", 5000, new ZKCreateNodeAsync());
        try {
            byte[] durian = zooKeeper.getData("/durian", true, stat);
            System.out.println("/durian的信息为：" + new String(durian));
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("权限不足,授权...");
        }
        zooKeeper.addAuthInfo("digest", "durian:durian".getBytes());
        zooKeeper.getData("/durian", true, new IDataCallback(), "授权获取durian...");

        Thread.sleep(Integer.MAX_VALUE);

        //异步获取子节点
//        zooKeeper.getChildren("/test", true, new IChildrenCallback(), "ctx");


        //获取节点值
//        Stat stat = new Stat();
//        byte[] data = zooKeeper.getData("/test", true, stat);
//        System.out.println("/test的值为：" + new String(data));

    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        if (Event.KeeperState.SyncConnected == watchedEvent.getState()) {
            if (Event.EventType.None == watchedEvent.getType() && null == watchedEvent.getPath()){
                countDownLatch.countDown();
            } else if(watchedEvent.getType() == Event.EventType.NodeChildrenChanged){
                try {
                    System.out.println("子节点发生变化 child:" + zooKeeper.getChildren(watchedEvent.getPath(), true));
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
        System.out.println("第"+ num++ + "次被监听...");
    }
}



class IDataCallback implements AsyncCallback.DataCallback{

    @Override
    public void processResult(int rc, String path, Object ctx, byte[] data, Stat stat) {
        System.out.println(ctx);
        System.out.println(path + "的值为：" + new String(data));
    }
}

class IChildrenCallback implements AsyncCallback.ChildrenCallback{

    @Override
    public void processResult(int rc, String path, Object ctx, List<String> children) {
        for (String child : children) {
            System.out.println(child);
        }
        System.out.println("传入的CTX：" + ctx);
    }
}

class IStringCallback implements AsyncCallback.StringCallback{

    @Override
    public void processResult(int rc, String path, Object ctx, String name) {
        System.out.println("Create path result: [" + rc + "," + path + "," + ctx + ",real path name:" + name);
        System.out.println("异步创建节点：" + path);
    }
}


class IVoidCallback implements AsyncCallback.VoidCallback {
    public void processResult(int rc, String path, Object ctx) {
        System.out.println(rc + ", " + path + ", " + ctx);
    }
}
```



## 0x12、Zookeeper客户端Curator使用详解

[Zookeeper客户端Curator使用详解]: https://www.jianshu.com/p/70151fc0ef5d



- apache开源
- 解决watcher的注册一次就失效
- 简单高效的Api，提供常用的工具类

##### 1）、添加maven依赖

```
 <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
    <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.4.1</version>
      <!--type标签要删除，否则可能引用不到该依赖-->
      <!--<type>pom</type>-->
    </dependency>

    <!--添加curator依赖-->
    <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
    <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-framework</artifactId>
      <version>4.0.0</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
    <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-recipes</artifactId>
      <version>4.0.0</version>
    </dependency>
```

##### 2)、Curator基础Api

- 基本的增删改查等操作

```
package com.durian.zk.curator.api;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.data.Stat;

import javax.security.auth.callback.Callback;
import java.util.List;
import java.util.concurrent.Executor;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 本类的todo只是为了将模块高亮，便于查看
 */
public class ZK_Curator_Api {
    public static void main(String[] args) throws Exception{
        //重试策略
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        //创建客户端，如果为集群多个主机信息用逗号隔开“ip:port,ip:port”，使用fluent风格创建
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString("192.168.31.164:2181")
                .sessionTimeoutMs(5000)
                .connectionTimeoutMs(5000)
                .retryPolicy(retryPolicy)
                .namespace("base")
                .build();
        //启动客户端
        client.start();


        /**
         * todo 数据节点操作
         * 增加节点
         */

        //如果没有给节点设置名称，该节点默认为持久化，内容为空
//        client.create().forPath("/durian");
//        client.create().forPath("/durian", "66".getBytes());
//        client.create().withMode(CreateMode.EPHEMERAL).forPath("/var", "1".getBytes());
        //如果没有父节点创建父节点
//        client.create().creatingParentContainersIfNeeded()
//                .withMode(CreateMode.EPHEMERAL).forPath("/durian/intro/age", "20".getBytes());




        /**
         * todo 删除节点
         */
        //只能删除叶子节点，否者会抛出异常
//        client.delete().forPath("/base/durian/intro/age");
        //删除一个节点，并且递归删除其所有子节点
//        client.delete().deletingChildrenIfNeeded().forPath("/durian");
        //删除一个节点，强制指定版本号进行删除
//        client.delete().withVersion(10086).forPath("/durian");
        //删除一个节点，强制保证删除
//        client.delete().guaranteed().forPath("/name");


        /**
         * todo 读取节点数据
         */
        //读取该节点内容
//        byte[] bytes = client.getData().forPath("/durian");
//        System.out.println("durian:" + new String(bytes));
        //读取该节点内容，同时获取到该节点的stat
//        Stat stat = new Stat();
//        byte[] bytes1 = client.getData().storingStatIn(stat).forPath("/durian");
//        System.out.println("stat:" + stat);
//        System.out.println("durian:" + new String(bytes1));


        /**
         * todo 检查节点是否存在，并更新节点
         */
//              Stat stat1 = client.checkExists().forPath("/durian");
//        if (stat1 != null) {
//            int aversion = stat1.getAversion();
//            System.out.println("版本号为:" + aversion);
//            client.setData().withVersion(aversion).forPath("/durian", "I am 66.".getBytes());
//        }


        /**
         * todo 获取某个节点的所有子节点路径
         */
//        List<String> strings = client.getChildren().forPath("/durian");


        /**
         * todo zk事务
         * 事务，CuratorFramework的实例包含inTransaction( )接口方法
         * ，调用此方法开启一个ZooKeeper事务. 可以复合create, setData, check, and/or delete
         * 等操作然后调用commit()作为一个原子操作提交。
         */
//        client.inTransaction().check().forPath("path")
//                .and()
//                .create().withMode(CreateMode.EPHEMERAL).forPath("path","data".getBytes())
//                .and()
//                .setData().withVersion(10086).forPath("path","data2".getBytes())
//                .and()
//                .commit();


        /**
         * todo 异步方法调用
         * 上面所有的方法都是同步的，Curator提供异步接口，
         * 引入了BackgroundCallback接口用于处理异步接口调用之后服务端返回的结果信息
         * BackgroundCallback接口中一个重要的回调值为CuratorEvent，里面包含事件类型、响应吗和节点的详细信息。
         */
//        ExecutorService executorService = Executors.newFixedThreadPool(2);
//        client.create()
//                .creatingParentsIfNeeded()
//                .withMode(CreateMode.EPHEMERAL)
//                .inBackground(new BackgroundCallback() {
//                    @Override
//                    public void processResult(CuratorFramework client, CuratorEvent curatorEvent) throws Exception {
//                            System.out.println(String.format("eventType:%s,resultCode:%s",curatorEvent.getType(),curatorEvent.getResultCode()));
//                    }
//                }, executorService)
//                .forPath("/word", "52066".getBytes());

        Thread.sleep(Integer.MAX_VALUE);
    }
}

```



##### 3）、Curator高级Recipes

- ###### Path Cache

```
package com.durian.zk.curator.api;

/**
 * todo Curator-recipes高级特性
 * 强烈推荐使用ConnectionStateListener监控连接的状态，
 * 当连接状态为LOST，curator-recipes下的所有Api将会失效或者过期，
 * 尽管后面所有的例子都没有使用到ConnectionStateListener。
 */


/**
 * todo 缓存
 * Zookeeper原生支持通过注册Watcher来进行事件监听，
 * 但是开发者需要反复注册(Watcher只能单次注册单次使用)。
 * Cache是Curator中对事件监听的包装，可以看作是对事件监听的本地缓存视图，
 * 能够自动为开发者处理反复注册监听。Curator提供了三种Watcher(Cache)来监听结点的变化。
 */

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.ChildData;
import org.apache.curator.framework.recipes.cache.PathChildrenCache;
import org.apache.curator.framework.recipes.cache.PathChildrenCacheEvent;
import org.apache.curator.framework.recipes.cache.PathChildrenCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;

import java.awt.*;

/**
 * todo Path Cache
 * Path Cache用来监控一个ZNode的子节点. 当一个子节点增加， 更新，删除时，
 * Path Cache会改变它的状态， 会包含最新的子节点， 子节点的数据和状态，
 * 而状态的更变将通过PathChildrenCacheListener通知。
 * 主要涉及的类：
 *      PathChildrenCache
 *          public PathChildrenCache(CuratorFramework client, String path, boolean cacheData)
 *      PathChildrenCacheEvent
 *      PathChildrenCacheListener
 *      ChildData
 */
public class Path_Cach_Api {
    private static final String PATH = "/example/pathCache";

    public static void main(String[] args) throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.31.164:2181", new ExponentialBackoffRetry(1000, 3));
        client.start();
        /**
         * 注意：如果new PathChildrenCache(client, PATH, true)中的参数cacheData值设置为false，
         * 则示例中的event.getData().getData()、data.getData()将返回null，cache将不会缓存节点数据。
         */
        //创建缓存，并开启，需要指定客户端和路径
        PathChildrenCache cache = new PathChildrenCache(client, PATH, true);
        cache.start();
        //定义并初始化监听器
        PathChildrenCacheListener cacheListener = new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
                System.out.println("事件类型：" + pathChildrenCacheEvent.getType());
                if (null != pathChildrenCacheEvent.getData()) {
                    System.out.println("节点数据：" + pathChildrenCacheEvent.getData().getPath()
                        + "=" + new String(pathChildrenCacheEvent.getData().getData()));
                }
            }
        };
        //添加监听器
        cache.getListenable().addListener(cacheListener);
        client.create().creatingParentsIfNeeded().forPath(PATH + "/test01", "01".getBytes());
        Thread.sleep(10);
        client.create().creatingParentsIfNeeded().forPath(PATH + "/test02", "02".getBytes());
        Thread.sleep(10);
        client.setData().forPath(PATH + "/test01", "01_V2".getBytes());
        Thread.sleep(10);
        for (ChildData childData : cache.getCurrentData()) {
            System.out.println("getCurrentData:" + childData.getPath() + "=" +
                    new String(childData.getData()));
        }
        client.delete().forPath(PATH + "/test01");
        Thread.sleep(10);
        client.delete().forPath(PATH + "/test02");
        Thread.sleep(1000*5);
        cache.close();
        client.close();
        System.out.println("OK!");
    }
}

```

- ###### Node Cache

```
package com.durian.zk.curator.api;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.ChildData;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;

/**
 * Node Cache与Path Cache类似，Node Cache只是监听某一个特定的节点。它涉及到下面的三个类：
 *   NodeCache - Node Cache实现类
 *   NodeCacheListener - 节点监听器
 *   ChildData - 节点数据
 */
public class Node_Cache_Api {
    private  static final String PATH = "/example/cache";

    public static void main(String[] args) throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.31.164:2181", new ExponentialBackoffRetry(1000, 3));
        client.start();

        client.create().creatingParentsIfNeeded().forPath(PATH);
        final NodeCache cache = new NodeCache(client, PATH);
        NodeCacheListener listener = new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                ChildData currentData = cache.getCurrentData();
                if (null != currentData) {
                    System.out.println("节点被修改，数据为：" + new String(cache.getCurrentData().getData()));
                } else {
                    System.out.println("节点被删除!");
                }
            }
        };
        cache.getListenable().addListener(listener);
        cache.start();
        client.setData().forPath(PATH, "01".getBytes());
        Thread.sleep(10);
        client.setData().forPath(PATH, "02".getBytes());
        Thread.sleep(10);
        client.delete().deletingChildrenIfNeeded().forPath(PATH);
        Thread.sleep(2000);
        cache.close();
        client.close();
        System.out.println("OK!");
    }

}

```

- ###### Tree Cache

```
package com.durian.zk.curator.api;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.PathChildrenCacheListener;
import org.apache.curator.framework.recipes.cache.TreeCache;
import org.apache.curator.framework.recipes.cache.TreeCacheEvent;
import org.apache.curator.framework.recipes.cache.TreeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;

/**
 * Tree Cache可以监控整个树上的所有节点，类似于PathCache和NodeCache的组合，主要涉及到下面四个类：
 *      TreeCache - Tree Cache实现类
 *      TreeCacheListener - 监听器类
 *      TreeCacheEvent - 触发的事件类
 *      ChildData - 节点数据
 */
public class Tree_Cache_Api {
    private static final String PATH = "/example/cache";

    public static void main(String[] args) throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.31.164:2181", new ExponentialBackoffRetry(1000, 3));
        client.start();

        client.create().creatingParentsIfNeeded().forPath(PATH);
        TreeCache cache = new TreeCache(client, PATH);
        TreeCacheListener listener = new TreeCacheListener() {
            @Override
            public void childEvent(CuratorFramework curatorFramework, TreeCacheEvent treeCacheEvent) throws Exception {
                System.out.println("事件类型：" + treeCacheEvent.getType() +
                    " | 路径：" + (null != treeCacheEvent.getData() ? treeCacheEvent.getData().getPath() : null));
                System.out.println(treeCacheEvent.getData().getPath() + "==" + new String(treeCacheEvent.getData().getData()));
            }
        };
        cache.getListenable().addListener(listener);
        cache.start();

        client.setData().forPath(PATH, "01".getBytes());
        Thread.sleep(10);
        client.setData().forPath(PATH, "02".getBytes());
        Thread.sleep(10);
        client.setData().forPath("/example", "hello example".getBytes());
        Thread.sleep(10);
        client.delete().deletingChildrenIfNeeded().forPath(PATH);
        Thread.sleep(1000);
        cache.close();
        client.close();
        System.out.println("OK!");
    }
}

```

##### 4）、基于curator实现分布式锁

```
package com.durian.zk.curator.api;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.PathChildrenCache;
import org.apache.curator.framework.recipes.cache.PathChildrenCacheEvent;
import org.apache.curator.framework.recipes.cache.PathChildrenCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.data.Stat;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class DistributedLock {
    private static CountDownLatch getLockCount = new CountDownLatch(1);
    private static CountDownLatch threadCount = new CountDownLatch(10);
    private final static String PATH = "/lock";
    private final static String DISTRIBUTED_PATH = "/lock/distribute_lock";

    public boolean getLock( CuratorFramework client)  {
        while (true) {
            try {
                client.create().withMode(CreateMode.EPHEMERAL).forPath(DISTRIBUTED_PATH, "yyy".getBytes());
                byte[] bytes = client.getData().forPath(DISTRIBUTED_PATH);
                System.out.println("抢到了锁...,并设置值为：" + new String(bytes));
                return true;
            } catch (Exception e) {
                if (getLockCount.getCount() <= 0) {
                    getLockCount = new CountDownLatch(1);
                }
                try {
                    getLockCount.await();
                } catch (InterruptedException e1) {
                    e1.printStackTrace();
                }
            }
        }
    }


    public boolean releaseLock(CuratorFramework client) {
        try {
            client.delete().forPath(DISTRIBUTED_PATH);
            Stat stat = client.checkExists().forPath(DISTRIBUTED_PATH);
            if (stat == null) {
                getLockCount.countDown();
                System.out.println("释放了锁...");
                return true;
            }
            return false;
        } catch (Exception e) {
           return false;
        }
    }

    public void addWatcherToLock(CuratorFramework client, String path) throws Exception {
        PathChildrenCache cache = new PathChildrenCache(client, path, true);
        cache.start();
        cache.getListenable().addListener(new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, PathChildrenCacheEvent event) throws Exception {
                if (event.getType().equals(PathChildrenCacheEvent.Type.CHILD_REMOVED)) {
//                    System.out.println("上一个节点释放锁,节点路径为：" + event.getData().getPath());
                    if (client.checkExists().forPath(path) == null){
                        getLockCount.countDown();
                    }
                }
            }
        });
        //cache.close();
    }

    public static void main(String[] args) throws Exception {

        ThreadPoolExecutor executor = new ThreadPoolExecutor(100, 1000, 1,
                TimeUnit.SECONDS, new LinkedBlockingDeque<>());
        for (int i = 0; i < 10; i++) {
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    threadCount.countDown();
                    int num = (int) threadCount.getCount();
                    CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.31.164:2181", new ExponentialBackoffRetry(1000, 3));
                    client.start();
                    DistributedLock distributedLock = new DistributedLock();
                    try {
                        distributedLock.addWatcherToLock(client, PATH);
//                        threadCount.countDown();
                        System.out.println("客户端启动成功,等待获取分布式锁..." + num);
                        threadCount.await();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    distributedLock.getLock(client);
                    System.out.println("我抢到了锁--------" + num);
                    distributedLock.releaseLock(client);
                    if (client != null)
                        client.close();
                }
            };
            executor.execute(runnable);
        }
    }
}

```



##### 5)、Leader选举

​	1）、**LeaderSelector** 轮流做Leader

​	2）、**LeaderLatch** 直到挂掉才会重新选举

