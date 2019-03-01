---
title: MongoDB
abbrlink: 51937
date: 2018-12-14 16:22:30
categories:
  - DB
tags: [MongoDB]
---

### 一. MongoDB是什么，有什么优点？
1. MongoDB是什么？
	MongoDB是一个基于分布式文件存储的NoSQL非关系数据库，是非关系数据库中功能最丰富，最像关系数据库的。
	NoSQL是非关系型数据库， NoSQL = Not Only SQL， 采用键值对的方式存储数据。
2. MongoDB优点？
	- 面向文件的
	- 高性能
	- 高可用性
	- 易扩展性
	- 丰富的查询语言

### 二. MongoDB与SQL型关系数据库的比较
1. 数据存储结构不同，关系数据库采用结构化的数据，NoSQL采用的是键值对的方式存储数据。因此，MySQL和MongoDB有许多基本差别包括数据的表示(data representation)，查询，关系，事务，schema的设计和定义，标准化(normalization)，速度和性能。
2. 因此，在处理非结构化/半结构化的大数据时；在水平方向上进行扩展时；随时应对动态增加的数据项时可以优先考虑使用NoSQL数据库； 在考虑数据库的成熟度，支持，分析和商业智能，管理及专业性等问题时，应优先考虑关系型数据库。

### 三. MongoDB与Redis的比较
1. 共同点：都是NoSQL,采用结构型数据存储。 
2. 一般称“Redis缓存，MongoDB数据库”。因为Redis主要把数据存储在内存中，其“缓存”性质远大于其“数据存储”的性质，其中数据的增删改查也只是像变量操作一样简单；而MongoDB却是一个“存储数据”的系统，增删改查可以添加很多条件，就像SQL数据库一样灵活。
2. 应用场景中存在区别：主要由于二者在内存映射的处理过程，持久化的处理方法不同。MongoDB建议集群部署，更多的考虑到集群方案，Redis则更偏重于进程顺序写入，虽然支持集群，但也仅限于主-从模式。
![](/images/MongoDB1.png)
![](/images/MongoDB2.png)

> 参考： 
MongoDB与Redis的比较： https://yq.aliyun.com/ziliao/64031
28个MongoDB经典面试题： https://blog.csdn.net/shmnh/article/details/42833291
MongoDN与关系型数据库相比的优缺点：https://blog.csdn.net/qq_32364939/article/details/79654377












