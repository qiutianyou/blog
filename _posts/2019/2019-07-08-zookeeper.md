---
layout: post
title: ZooKeeper
category: ZooKeeper
tags: [ZooKeeper]
---

让我们来一起认识一下这位强大的动物管理员吧


## ZooKeeper：分布式应用程序的分布式协调服务
  ZooKeeper是apache软件基金会下面的顶级项目，属于hadoop生态项目之一；
  从字面意思可以看出，意思是 **动物管理员** ，这里的 **动物** 可不是真的动物，在分布式架构中，随着业务量的增加，微服务系统越来越多，系统之间的调用页越来越复杂，而这里的 **动物** 指的就是分布式系统的各个子系统，**ZooKeeper** 则是管理员。

### Zookeeper的特点
  * Zookeeper的主要作用是为分布式系统提供协调服务,包括但不限于:分布式锁,统一命名服务,配置管理,负载均衡,主控服务器选举以及主从切换等。
  * ZooKeeper允许分布式进程通过共享的分层命名空间相互协调，该命名空间的组织方式与标准文件系统类似。专为存储而设计的典型文件系统不同，**ZooKeeper数据保存在内存** 中，这意味着ZooKeeper可以实现高吞吐量和低延迟数量。ZooKeeper实现非常重视 **高性能，高可用性**，严格有序的访问。ZooKeeper的性能方面意味着它可以在大型分布式系统中使用。

  * **ZooKeeper同样支持日志备份**，在每次写入内存中的同时都会 **将记录记录到事务日志** 中，当事务日志记录的次数达到一定数量后(默认10W次)，就会将内存数据库序列化一次，使其 **持久化** 保存到磁盘上，序列化后的文件称为 **"快照文件"**。每次拍快照都会生成新的 **事务日志**。


  * **支持集群，支持主从复制**，故而在 **读多写少的** 场合有非常好的性能表现。每个服务器节点每次接收到写操作请求时，都会先将这次请求发送给leader，leader将这次写操作转换为带有状态的事务，然后leader会对这次写操作广播出去以便进行协调。当协调通过(大多数节点允许这次写)后，leader通知所有的服务器节点，让它们将这次写操作应用到内存数据库中，并将其记录到事务日志中。
  ![/assets/images/zkservice.jpg](/assets/images/zkservice.jpg)

  更加详细的介绍：[https://zookeeper.apache.org/doc/current/zookeeperOver.html](https://zookeeper.apache.org/doc/current/zookeeperOver.html)

### Zookeeper的应用
  作为一名web服务端开发的Java程序员，并没有大数据hadoop生态开发经验，项目中能应用到zookeeper的就是注册中心了。使用zookeeper来管理各个子系统的服务发布、服务调用、服务监控。
  下一章节，讲dubbo时，介绍zookeeper作为dubbo的注册中心的使用
