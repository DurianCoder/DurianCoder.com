---
title: Redis多master集群管理
date: 2018-08-21 08:54:25
tags:
	- Redis
---

前面我们简单入门学习了解了Redis，包括Redis特性、安装、数据结构、持久化、事务、简单的主备等。

[Redis基础](https://duriancoder.github.io/2018/08/08/Redis%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/)

[Redis集群官网文档](http://www.redis.cn/topics/cluster-tutorial.html) <!-- more -->

今天我们来学习使用Redis搭建多master多slave主备模式。

## 0x01、搭建6台Redis主机

- 创建一个redis-cluster文件夹，在其下面简历6个文件夹分别命名为6001,6002,6003,7001,7002,7003
- 复制redis.conf文件到上述六个文件夹
- 修改redis.conf配置文件，将配个redis.conf的port和对应的file值更改。

```
port 7001   //六个节点配置文件分别是7001-7003
bind 192.168.1.109  //默认ip为127.0.0.1 需要改为其他节点机器可访问的ip 否则创建集群时无法访，和单机集群有区别

daemonize yes        //redis后台运行

pidfile /var/run/redis_7001.pid   //pidfile文件对应7001-7003

cluster-enabled yes   //开启集群

cluster-config-file nodes_7001.conf  //保存节点配置，自动创建，自动更新对应7001-7003

cluster-node-timeout 5000    //集群超时时间，节点超过这个时间没反应就断定是宕机

appendonly yes   //存储方式，aof，将写操作记录保存到日志中
```

- 运行六台Redis服务器



## 0x02、搭建集群

##### 1、搭建ruby环境，[ruby环境安装](http://www.runoob.com/ruby/ruby-installation-unix.html)

```
# 切换到root用户下安装，如报错参考上面链接安装
$ yum install -y ruby rubygems
$ gem install redis
```

##### 2、创建集群，运行redis/src目录下./redis-trib.rb

```
# --replicas 1 表示给每个master分配一个slave，即6001、6002、6003为master,7001、7002、7003为slave
$ ./redis-trib.rb create --replicas 1 192.168.163.66:6001 127.0.0.1:6002 127.0.0.1:6003 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003
```



## 0x03、集群管理

[Redis命令](http://www.redis.cn/commands.html)

##### 1、基本的集群命令简介

```
CLUSTER INFO 打印集群的信息
CLUSTER NODES 列出集群当前已知的所有节点（node），以及这些节点的相关信息。 
//节点
CLUSTER MEET <ip> <port> 将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
CLUSTER FORGET <node_id> 从集群中移除 node_id 指定的节点。
CLUSTER REPLICATE <node_id> 将当前节点设置为 node_id 指定的节点的从节点。
CLUSTER SAVECONFIG 将节点的配置文件保存到硬盘里面。
CLUSTER ADDSLOTS <slot> [slot ...] 将一个或多个槽（slot）指派（assign）给当前节点。
CLUSTER DELSLOTS <slot> [slot ...] 移除一个或多个槽对当前节点的指派。
CLUSTER FLUSHSLOTS 移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。
CLUSTER SETSLOT <slot> NODE <node_id> 将槽 slot 指派给 node_id 指定的节点。
CLUSTER SETSLOT <slot> MIGRATING <node_id> 将本节点的槽 slot 迁移到 node_id 指定的节点中。
CLUSTER SETSLOT <slot> IMPORTING <node_id> 从 node_id 指定的节点中导入槽 slot 到本节点。
CLUSTER SETSLOT <slot> STABLE 取消对槽 slot 的导入（import）或者迁移（migrate）。 
//键
CLUSTER KEYSLOT <key> 计算键 key 应该被放置在哪个槽上。
CLUSTER COUNTKEYSINSLOT <slot> 返回槽 slot 目前包含的键值对数量。
CLUSTER GETKEYSINSLOT <slot> <count> 返回 count 个 slot 槽中的键。 
//新增
CLUSTER SLAVES node-id 返回一个master节点的slaves 列表
```

##### 2、连接Redis集群客户端

```
# -c 表示连接集群，-p 表示端口， -h表示主机
$ ./redis-cli -c -p 6001
```

