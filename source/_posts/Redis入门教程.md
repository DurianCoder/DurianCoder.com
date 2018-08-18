---
title: Redis入门教程
date: 2018-08-08 19:05:57
tags:
	- Redis
---

## **0x01、NoSQL介绍**

[Redis博客链接](https://www.cnblogs.com/duriancoder/articles/9383957.html)

要学习Redis首先要了解什么是NoSQL，NoSQL有什么特点等 <!-- more-->

##### **1、什么是NoSQL？**

NoSQL（Not only SQL)，意思是”不仅仅是SQL"，泛指非关系型数据库。像常用的关系型数据库MySQL、Oracle和sqlserver，这些数据库一般用来存储数据信息，应对普通的业务没有问题，但是，随着互联网的迅速发展，传统关系型数据库在应对超大规模、流量以及高并发的时候力不从心。这个时候NoSQL派上了用场，得到迅速发展。

##### **2、NoSQL有哪些特点？**

1）、易扩展。去掉了关系型数据库中间的关系，使得NoSQL非常容易扩展。

2）、大数据量高性能。无关系性，数据库结构建档。

3）、灵活多样的数据模型。NoSQL不需要简历字段，维护表的结构，可以任意扩展。

　　

## **0x02、nosql分类及比较**

##### **1、常用NoSQL有哪些？**

**1）、Redis**

优点：1、支持多种数据结构，如String、list、hash、set、zset、hyperloglog(基数估值)

　　　2、支持持久化操作，rdb（redis database）和aof （append only file)

　　　3、支持主从热备，读写分离等

　　　4、支持pub/sub消息订阅与通知

　　　5、支持简单的事务。

缺点：1、Redis是单线程，性能受限于CPU，QBS取决于数据大小和硬件等

　　　2、只支持简单事务，不能完全保证事务一致性

**2）、Mencache**

优点：1、可以利用多核优势，单实例吞吐量极高（取决key，value大小和硬件等）

　　　2、支持直接配置为session handle

缺点：1、只支持key/value数据结构

　　　2、不能持久化，数据不能备份，只用用于缓存，重启后数据全部丢失。

　　　3、无法进行数据同步，不能将MC中的数据迁移到其他MC实例中

**3）、MangoDB**：是一种文档性数据库，即可以存放xml、json、bson等类型数据

优点：1、支持丰富的数据表达，索引，最类似关系型数据库，支持的查询语言非常丰富

　　　2、支持master-slave,replicaset（内部采用paxos选举算法，自动故障恢复）,auto sharding机制，对客户				端屏蔽了故障转移和切分机制。

　　    3、MongoDB从1.8版本开始采用binlog方式支持持久化的可靠性

缺点：1、合大数据量的存储，依赖操作系统VM做内存管理，吃内存也比较厉害，服务不要和别的服务在一起

　　　2、MongoDB不支持事务

 

##### **2、Redis、Memcache和MongoDB的区别**

**1)、性能**：性能都比较高，总体来讲，TPS方面Redis和Memcache差不多，要大于mongoDB。

**2)、数据结构**：mencache数据结构单一；redis数据结构丰富；mangoDB支持丰富数据表达、索引、存储过程，最类似关系型数据库。

**3)、持久化**：redis和mangoDB支持持久化，memcache所有数据存储在内存。

**4)、数据一致性**：mencache使用cas保持一致性；redis事务支持比较弱；mangoDB不支持事务。



## **0x03、非关系性数据库和关系型数据比较**

**1、关系型数据库和非关系性数据库的区别?**

数据结构的不同。

##### **2、传统数据库的ACID和分布式数据库的CAP**

*关系型数据库遵循ACID原则：*

**Atomicity 原子性**：事务里的操作要么全部完成，要么全部失败。

**Consistency 一致性**：数据库要一直处于一致的状态，事务的运行不会改变原本一致性约束。

**Isolation 独立性**：并发的多线程相互隔离，互不影响。

**Durability 持久性**：事务提交后，数据会永久保存到数据库上。

*分布式数据库的CAP原则：*

分布式存储系统只能实现两点，由于网络丢包、延迟等存在，必须保证P，所以在A和C之间做权衡。

**Consistency 强一致性**：保证数据一致。

**Availablity 可用性**：单点故障或者数据不一致，服务是否可用。

**Partition tolerance** ：分区容忍性。

**CAP关系**：可以通过将数据项复制到多个节点来提高分区容忍性，把数据复制到多个节点就会存在数据一致性，多个节点的数据可能不一致，要保证数据一致，每次写操作就要等待全部节点写成功，而等待就会带来可用性问题。总的来说，节点越多，分区容忍性越高，要复制的数据越多，一致性越难保证。更新节点需要时间，就会导致可用性降低；对于一些对数据一致性要求不是特别高的业务，可以通过降低数据一致性来保证高可用性，最终将内存中数据再持久化来达到最终一致性的方案。

 

## **0x04、redis数据结构**

可以在[http://redisdoc.com](http://redisdoc.com/)中看所有reids数据类型操作命令

##### 1、**key** : redis中对key的基本操作　
![img](/Redis入门教程/key-api.png)

##### 2、**String** （字符串）：string是redis最基本的数据类型，是二进制安全的，可以理解为和memcache一模一样的数据类型一个key对应一个value，redis中一个字符串value最大为512M
![img](/Redis入门教程/string-api.png)
![img](/Redis入门教程/string-api-2.png)

##### 3、**List** （链表）: 底层是一个链表
![img](/Redis入门教程/list-api.png)![img](https://images2018.cnblogs.com/blog/1233188/201807/1233188-20180729173239730-1097707098.png)

##### 4、**Hash** （哈希，类似java中Map)：是string类型的key/value映射表，适合存储对象
![img](/Redis入门教程/hash-api.png)

##### 5、**Set**（集合）：String的无序集合
![](/Redis入门教程/set-api.png)

##### 6、**Zset**（sorted set：有序集合）：在Set的每个元素上添加一个double类型的score，score可以重复，value不能重复
![img](/Redis入门教程/zset-api-1.png)
![img](/Redis入门教程/zset-api-2.png)

## **0x05、Redis配置文件介绍**
对redis的操作和配置主要是对redis.conf配置文件进行操作，哨兵配置文件sentinel.conf
```
参数说明
redis.conf 配置项说明如下：
1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
  daemonize no
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
  pidfile /var/run/redis.pid
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
  port 6379
4. 绑定的主机地址
  bind 127.0.0.1
5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
  timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为 notice
  loglevel notice
7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
  logfile stdout
8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
  databases 16
9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
  save <seconds> <changes>
  Redis默认配置文件中提供了三个条件：
  save 900 1
  save 300 10
  save 60 10000
  分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
 
10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
  rdbcompression yes
11. 指定本地数据库文件名，默认值为dump.rdb
  dbfilename dump.rdb
12. 指定本地数据库存放目录
  dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
  slaveof <masterip> <masterport>
14. 当master服务设置了密码保护时，slav服务连接master的密码
  masterauth <master-password>
15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
  requirepass foobared
16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
  maxclients 128
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
  maxmemory <bytes>
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
  appendonly no
19. 指定更新日志文件名，默认为appendonly.aof
   appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值： 
  no：表示等操作系统进行数据缓存同步到磁盘（快） 
  always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
  everysec：表示每秒同步一次（折衷，默认值）
  appendfsync everysec
 
21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）
   vm-enabled no
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
   vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
   vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
   vm-page-size 32
25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
   vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
   vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
  glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
  hash-max-zipmap-entries 64
  hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
  activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
  include /path/to/local.conf
31. 内存机制
	voltile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

　　volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

　　volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

　　allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

　　allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

　　no-enviction（驱逐）：禁止驱逐数据

```

　　

## **0x06、持久化之rdb、aof**

redis的持久化是通过rdb和aof来实现的，下面将详细介绍rdb和aof。

**RDB介绍：**

**1、RDB（Redis DataBase）**：在指定时间间隔将内存中的数据集写入磁盘，也就是Snapshot快照，存储服务器在恢复的时候默认将快照加载到内存中；

**2、RDB如何持久化**：Redis会fork一个子进程来进行持久化，会将数据写入一个临时文件，等持久化过程结束，再用这个临时文件替换上次持久化好的文件，整个过程中主进程不进行任何IO操作，这就确保极高的性能；如果需要对大规模的数据进行恢复且对数据一致性不是特别敏感，RDB方式要比AOF方式更加高效。

**3、RDB的缺点**：

　1）、最后一次持久化后的数据可能丢失，AOF可以把这个时间缩短到1秒。

​    2）、Fork的时候，内存中的数据被克隆一份，大致2倍的膨胀性需要考虑性能。

 **4、如何配置**：可以在redis.conf中配置 save

\# save ""   关闭rdb

save 900 1   15min至少有一个key被改变时持久化
save 300 10  5min内至少10个key改变时持久化

save 60 10000  1min内10000个key改变时持久化

　　除了出发配置的save参数条件外，在执行save（全部阻塞，不提供请求功能）、bgsave（后台持久化，能同时处理了其他客户端请求）、flushall、flushdb操作时也会执行rdb持久化操作。

**5、rdb恢复：**dump.rdb会生成在CONFIG GET dir 目录下，在服务器启动时会自动将dump.rdb中数据读取到内存中。

**AOF简介：**

**1、AOF （Append Only File）**：以日志的形式记录Redis每个写操作，默认是关闭状态，启动的时候，Redis重启后会根据日志文件的内容将写指令从头执行一次来完成数据恢复工作。

**2、AOF配置：**

```
# redis 默认配置为no，开启aof改为yes
appendonly no 
```

  ##### 配置Rewrite

  auto-aof-rewrite-percentage 100      # 重写的时候大小翻倍
  auto-aof-rewrite-min-size 64mb       # 一般设置为3G+

  ##### 配置记录方式

appendfsync always  # 同步持久化，每发生数据变更，马上记录到磁盘，一致性好性能查。

appendfsync  everysec  # 异步持久化，每秒记录，如果宕机，丢失一秒数据

appendfsync   no    # 从不同步

**3、AOF恢复：**

\# 正常情况Redis启动时直接执行appendonly.aof

\# 在Redis宕机时，appendonly.aof末尾可能出现指令记录不完全导致故障，那么在启动之前先执行

redis-check-aof

**4、AOF重写Rewrite：**

**触发**：Redis会记录上次重写时AOF的大小，默认时当AOF文件大小是上次rewrite大小的一倍且大于64M时触发。

**重写原理**：AOF增长过大时，会fork一条新进程来将文件重写，重写aof并没有读取 旧的aof，而是将整个内存中的内容用命令的方式重写了一个aof，和快照有点类似。

**5、AOF劣势**

　1、相同数据aof文件远大于rdb文件，恢复速度慢于rdb

　2、aof运行效率慢于rdb，每秒同步策略较好，不同步效率和rdb相同



## **0x07、事务、消息订阅**

**1、Redis中的事务：**

Redis中的事务比较弱，不能保证原子一致性，能保证事务中的命令按顺序执行。

悲观锁：每次对表的写操作都将表进行加锁，其请求就阻塞。

乐观锁：每次都不加锁，在表中添加一个version每次写的时候比较当前的version是否大于表中version，如果大于则数据没有被更改，执行写操作，否则重新从数据库读取数据执行业务操作。

Redis中的事务常用命令：

```
WATCH key  　　 #监控锁key，乐观锁,当监控的key有任何变化的时候，事务里的所有命令将全部被放弃
UNWATCH 　　　　# 取消监控所有key
MULTI  # 开启事务
COMMAND   # 命令入栈
EXEC       # 事务执行，执行完会释放所有WATCH
```



**2、Redis中的发布订阅**：发送者 (pub) 发送消息，订阅者（sub）接受消息**

　案例：

```
6397>  SUBSCRIBE c2,c1,b*   #订阅c1,c2频道,b*订阅以b开头的频道
6397>  PUBLIS c1 mesage　　# 往c1频道发送消息
```

 

## **0x08、主从复制、读写分离、哨兵模式**

**主从复制：**主机数据更新后根据配置和策略自动同步到备机的master/slaver机制。

**读写分离：**主机用来写数据，从机用来读操作，减少数据库压力。

**怎么玩**：1、配从不配主，从机执行 SLAVEOF ip port 

　　　　2、修改redis.conf，修改daemonize、filepid、logname

**常用模式**：1、一主二仆：一个Master两Slave；主机down掉，从机原地待命，等待主机恢复，从机重新连接主机；从机down掉变为master，独立程一个节点；

　　　　　2、薪火相传：一个Master连一个从机S1，另一个从机S2连接S1为主机；分解Master压力。

　　　　　3、访客为主：薪火相传当主机down掉，从机 S1执行 SLAVEOF no one 变为主机。

**复制原理**：从机刚启动时会执行一次全量复制，即从机向master发送一个sync命令，master接受命令开启后台的存盘进程，同时收集所有修改数据集命令，后台存盘进程执行完毕后，将整个数据文件传送到slave，来完成同步。

**全量复制：**slave服务在接受到数据文件后，将其存盘并加载到内存。

**增量复制：**master继续将新的所有搜集到的修改命令依次传给slave，完成同步。

 

**哨兵模式：**访客为主的自动化配置，配置好sentinel.confK，当主机down掉时，从机会进行选举投票，票数多的作为主机。配置哨兵配置文件，sentinel.conf中配置

```
sentinel monitor sentinel_name ip port 1
```

**启动哨兵**：Redis-sentinel /sentinel.conf

**哨兵原理：**主机挂掉重新恢复会变成从机，sentinel会每隔一段时间向其他sentinel、master、slaver发送消息（基于socket通信，发送Ping），看是否down掉，如果超过默认时间没有给sentinel回复则标记为SDOWN（主观Down），如果多台sentinel都判断为Down，则SDOWN变成Object Down，那么就会通过执行选举来选出新的Master。

哨兵集群模式：<https://www.cnblogs.com/PatrickLiu/p/8444546.html>

 

## **0x09、java中使用redis、分布式锁**

**java调用redis api**

**在windows下连接Linux上的redis时需要保证一下几点：**

**1、关闭防火墙**  

```
systemctl stop firewalld.service 
```

**2、关闭reids的保护模式，配置redis.conf：protected-mode  no**

**3、加入jedis.jar和commons-pool2.jar**

**创建JedisPool线程安全单例**

```
package com.durian.jedis.util;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class JedisPoolUtil {
    // 私有构造方法
    private JedisPoolUtil() {}

    private volatile static JedisPool jedisPool = null;

    public static JedisPool getJedisPool(String host, int port) {

        if (null == jedisPool){
            // 锁住对象
            synchronized (JedisPoolUtil.class) {
                if (null == jedisPool) {
                    JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
                    // 设置最大连接数
                    jedisPoolConfig.setMaxTotal(1000);
                    // 设置最大空闲数
                    jedisPoolConfig.setMaxIdle(66);
                    // 连接超时时间
                    jedisPoolConfig.setMaxWaitMillis(100 * 1000);
                    // 连接测试
                    jedisPoolConfig.setTestOnBorrow(true);
                    jedisPool = new JedisPool(jedisPoolConfig, host , port);
                }
            }
        }
        return jedisPool;
    }

    public static Jedis getJedis() {
        JedisPool jedisPool = getJedisPool("192.168.31.164", 6379);
        return jedisPool.getResource();
    }

    // 释放连接
    public static void release(Jedis jedis) {
        if (null == jedis) {
            jedis.close();
        }
    }


}
```

**基于Jedis实现分布式锁**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import redis.clients.jedis.Jedis;

import java.util.Collections;

/**
 * 为了确保分布式所可用，必须保证一下四点：
 *      1、互斥性。在任意时刻只有一个客户端能持有锁
 *      2、不会发生死锁。加上过期时间
 *      3、具有容错性。只要大部分redis节点运行正常即可。
 *      4、解铃还须系铃人。加锁解锁必须为同一个客户端。
 */
public class DistributeLock {

    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIT = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";
    private static final Long RELEASE_SUCCESS = 1L;

    /**
     *
     * @param jedis  jedis客户端
     * @param lockKey 要加锁的key
     * @param requestId  key对应客户端id，可以使用UUID
     * @param expireTime 过期时间
     * @return
     */
    public static boolean getDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime){

        /**
         * 当key不存在时加锁，并设置过期时间，避免死锁
         */
        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIT , SET_WITH_EXPIRE_TIME, expireTime);
        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }

    /**
     *  使用lua脚本，保证操作原子性，java在执行lua代码时，lua代码将被当成一个命令去执行，并且直到eval命令执行完成，redis才会执行其他命令。
     * @param jedis
     * @param lockKey
     * @param requestId
     * @return
     */
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId){
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }


    public static void main(String[] args) {
        Jedis jedis = JedisPoolUtil.getJedis();
        boolean name = getDistributedLock(jedis, "name", "66", 10);
        System.out.println("给name加锁：" + name);
        System.out.println(jedis.get("name"));
        jedis.set("name", "liuliu");
        System.out.println(jedis.get("name"));
    }
}
```

**Jedis事务、乐观锁**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

import javax.sound.midi.Soundbank;
import java.util.List;

public class TestLock {
    public static void main(String[] args) throws InterruptedException {
        Jedis jedis = JedisPoolUtil.getJedis();
        int price = 10;

        System.out.println(jedis.keys("*"));


        String balance = jedis.get("balance");

        /**
         * 当你对一个key 进行watch之后，如果key改变了，那么后面的事务不会执行。
         */
        jedis.watch("balance");
        int b = new Integer(balance);
        if (b < 10) {
            System.out.println("余额不足.");
            jedis.unwatch();
            return;
        }

        System.out.println("before:" + jedis.mget("balance", "debt"));
        Thread.sleep(6 * 1000);
        System.out.println("after:" + jedis.mget("balance", "debt"));

        Transaction transaction = jedis.multi();
        transaction.decrBy("balance" , price);
        transaction.incrBy("debt" , price);
//        System.out.println("please change balance...");
//        Thread.sleep(6 * 1000);
        List<Object> exec = transaction.exec();
        System.out.println(exec);

//        System.out.println(jedis.mget("balance", "debt"));
    }
}
```

**Jedis使用key**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import oracle.jrockit.jfr.events.JavaEventDescriptor;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

import java.util.Set;

/**
 * 在windows上连接WMware 虚拟机上的redis时，需要配置好一下两点：
 *      1、关闭防火墙： systemctl stop firewalld.service
 *      2、将redis.conf 中bind:127.0.0.1注释
 *      3、将reids.conf 中protected-mod 配置为 no, CONFIG set protected-mod no
 */
public class TestKeyApi {
    public static void main(String[] args) throws InterruptedException {
        JedisPool jedisPool = JedisPoolUtil.getJedisPool("192.168.31.164" , 6379);
        Jedis jedis = jedisPool.getResource();


        Set<String> keys = jedis.keys("*");
        System.out.println(keys);

        // 清除数据库
        String s = jedis.flushDB();
        System.out.println("flushDB:" + s);

        // 设置key
        jedis.set("k1", "v1");
        System.out.println(jedis.get("k1"));

        // 设置超时
        jedis.setex("k1", 20 , jedis.get("k1"));
        Thread.sleep(200);
        Long k1 = jedis.ttl("k1");
        System.out.println("k1还有" + k1 + "秒过期.");

        // 查看key是否存在、查看key类型
        System.out.println("k1是否存在：" + jedis.exists("k1"));
        System.out.println("k1的数据类型为：" + jedis.type("k1"));

    }
}
```

**Jedis使用String**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.exceptions.JedisException;
import sun.applet.Main;

public class TestStringApi {

    public static void main(String[] args) {
        Jedis jedis = JedisPoolUtil.getJedis();

        // 选择数据库
        String select = jedis.select(1);
        System.out.println("使用1号数据库?:" + select );

        // set、get、strlen、append、getrange、setrange、incr、decr、mset、mget
        jedis.set("k1" , "v1");
        String k1 = jedis.get("k1");
        System.out.println("k1的值为：" + k1);

        System.out.println("k1的长度为:" + jedis.strlen("k1"));
        jedis.append("k1", "1234");
        String k1range = jedis.getrange("k1", 0, -1);
        System.out.println("getrange k1 0 -1 :" + k1range);

        Long k11 = jedis.setrange("k1", 0, "66520");
        System.out.println("setrange k1 0 66520:" + k11);

        String age = jedis.set("age", "22");
        System.out.println("before incr age:" + jedis.get("age"));
        Long age1 = jedis.incr("age");
        System.out.println("after incr age:" + age1);
    }

}
```

**Jedis使用List**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import redis.clients.jedis.BinaryClient;
import redis.clients.jedis.Jedis;

public class TestListApi {
    public static void main(String[] args) {
        Jedis jedis = JedisPoolUtil.getJedis();

        jedis.flushDB();

        // lpush、rpush、lrange、lpop、rpop、llen、lindex、lrem、ltrim、lset key index value 、linsert key after/before v1 v2
        jedis.lpush("list01", "v1" , "v2", "v3", "v4");
        jedis.rpush("list01", "v5" , "v6");
        System.out.println("list01 :" + jedis.lrange("list01" , 0 , -1));
        System.out.println("rpush list01 (v6) :" + jedis.rpop("list01"));
        System.out.println("lpush list01 (v1):" + jedis.lpop("list01"));
        System.out.println("llen list01 (6): " + jedis.llen("list01"));

        jedis.lrem("list01", 1, "1");
        System.out.println("删除第一个元素后list01 :" + jedis.lrange("list01", 0 , -1));

        jedis.linsert("list01", BinaryClient.LIST_POSITION.AFTER, "v5", "66");
        String list01 = jedis.ltrim("list01", 0, -1);
        System.out.println("update后的list01 ： " + jedis.lrange("list01" , 0 , -1));
        System.out.println("i want " + jedis.lindex("list01" , 4));


    }
}
```

**Jedis使用Hash**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import redis.clients.jedis.Jedis;

import java.util.HashMap;
import java.util.Map;

public class TestHashApi {
    public static void main(String[] args) {
        Jedis jedis = JedisPoolUtil.getJedis();

        // hash  key  k:v
        // hget,hset,hlen,hkeys,hvals,hmget,hmset,hexits,hincr

        jedis.hset("hs01", "k1", "v1");
        Map map = new HashMap();
        map.put("k1", "v1");
        map.put("k2", "v2");

        jedis.hmset("hs02", map);
        System.out.println("hs01 :" + jedis.hget("hs01" , "k1"));
        System.out.println("hs02 :" + jedis.hget("hs02" , "k2"));
        System.out.println("hs02 length :" + jedis.hlen("hs02"));
        System.out.println("hs02是否存在k1 : " + jedis.hexists("hs02" , "k1"));

    }
}
```

**Jedis使用Set**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import redis.clients.jedis.Jedis;

public class TestSetApi {
    public static void main(String[] args) {
        Jedis jedis = JedisPoolUtil.getJedis();

        // sadd,spop,srem,scard,srandomkey,smenber,srandmember,sdiff,sinter,sunion
        jedis.sadd("s01", "v1", "v2", "v3", "v1");
        Long s01 = jedis.scard("s01");
        System.out.println(s01);
        jedis.sadd("s01", "66");
        System.out.println("s01 : " + jedis.smembers("s01"));
        System.out.println("随机取一个和三个：");
        System.out.println(jedis.srandmember("s01"));
        System.out.println(jedis.srandmember("s01" , 3));

    }
}
```

**Jedis使用Zset**

```
package com.durian.jedis.api;

import com.durian.jedis.util.JedisPoolUtil;
import redis.clients.jedis.Jedis;

import java.util.Set;

public class TestZSetApi {
    public static void main(String[] args) {
        Jedis jedis = JedisPoolUtil.getJedis();

        // zadd,zrem,zrange,zcard,zcount,zrangbyscore
        jedis.zadd("z01", 66, "ll");
        jedis.zadd("z01", 77, "qq");
        jedis.zadd("z01", 88, "&&");
        System.out.println(jedis.zrange("z01" , 0 , -1));
        jedis.zrem("z01" , "&&");
        System.out.println(jedis.zrange("z01" , 0 , -1));
        jedis.zadd("z01" , 52 , "jiangying");
        Set<String> z01 = jedis.zrangeByScore("z01", 1, 77);
        System.out.println(z01);


    }
}
```



 
