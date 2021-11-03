---
title: System Design I
date: 2019-08-26 12:04:32
tags: system design
---

### Introduction

#### 1.常见System Design问题

* 设计某系统  
Design Twitter, Uber, Whatsapp, TinyURL, NoSQL...
* 设计某系统中的某功能  
Design Rate Limiter, Delete Twitter, Mark email as read...

#### 2.与OOD的异同

* OOD  
Class, Interface, Inheritance...
* System Design  
Database, Cache, File System, Scalability, Master Slave, Load Balancer, Web Server, Sharding, Consistent Hashing, QPS...

#### 3.4S分析法

* Scenario  
Ask: Features, Interfaces. 即需要哪些功能  
Analyze: QPS(quries per second), Peak QPS, DAU(daily active users)
* Service
将大系统拆分成小的service  
Merge similar into one service  
把service之间的关系图画出来
* Storage 这一步非常重要!!!  
数据的存储和访问: Schema/Data/SQL or NoSQL/File System  
Choose a proper db for each service，画图
* Scale  
Optimize and Maintenance.  
解决可能遇到的问题, Special case, Sharding

前三步就是work solution，最后的scale一般是工作经验比较久的。  
先设计一个基本能工作的work solution，然后再逐步优化。
</br>

### Design Twitter

#### 1.Scenario

1)首先把所有可能的功能罗列出来，从中挑选出核心功能。

常见的twitter功能有：  
Register/Login, User Profile Display/Edit, Upload Image/Video, Search, Post/share a twitter, Timeline/News Feed, Follow/Unfollow a user.

核心功能：  
Register/Login, Post a twitter, News Feed(自己关注的信息流的集合), Timeline(自己发的信息的集合), Follow/Unfollow a user

2)Computations  
一个TPS(transactions per second)里有多个database queries，一般就考虑QPS就行。

* QPS = DAU * querie per user / 一天的秒数(86400)
* Peak QPS = QPS * 3
* Read QPS & Write QPS: 对database的操作频率

</br>

根据QPS的值来考虑db and server:  
a) For server

* QPS = 100  
普通电脑做服务器即可
* QPS = 1K  
一台好点的服务器，需要考虑single point failure, 解决方法Sharding & Replica
* QPS = 1M  
web server cluster, need to consider maintainance.

b) For DB

* MySQL/PosgreSQL DB = 1k QPS
* NoSQL Cassandra/MongoDB = 10k QPS
* NoSQL Redis/Memchached = 1M QPS

#### 2.Service

为每个feature添加一个service  
Merge same and similar services

{%asset_image twitterService.png%}

#### 3.Storage

* 为每个service选择存储结构  
* Schema细化表结构

1)Database

* SQL: User Table  
因为user的信息比较多
* NoSQL: Twitter Service and Friend Service  
因为查询比较简洁，由于推文增加比user要快，所以用Nosql自带分布式属性。而且NoSQL扩展比较容易，而放在sql里就需要自己做sharding  

2)File System  
Media and Video files  
3)Cache

如果Read Operation很多，一定要用cache来进行优化！
如果Write Operation很多，可以用sharding  
如果读写都多，可以使用更多的db来分摊，或者使用Redis这种读写多快的数据库。

{%asset_image twitterStorage.png%}

#### 4.Scale

主要分成两个部分：

* Optimize  
解决设计缺陷，比如pull and push model(模型介绍见后面);  
more features;  
special cases;

* Maintenance  
Robust: If one server/db breaks down;  
Scalability.
</br>

**1) Optimize**  

- 由于DB read很费时间，所以可以在db前加上cache。  
比如cache每个user的timeline, cache每个user的news feed.
- Deal with inactive users.
- Special case  
Q1：If followers >> following, fanout will take a lot of time. How to solve it?  
A1：For ordinary user, use push model; For celebrity, use pull model.  
Q2：How to check a user a celebrity or not?  
A2：If a user is marked as celebrity, he will not be unmarked even if he lost a lot fans.

**2) Maintenance**  
见其他文章，链接：

</br>

### Pull Model & Push Model

以News Feed为例

#### 1.Pull Model

当user查看news feed时，需要获取其好友的前100条tweets。可以使用Merge K Sorted Arrays算法，合并出来得到news feed。这个merge算法因为是在内存中进行的，所以相比db的操作，其时间可以忽略。

假设该user关注了N个人，那么在获取news feed的过程中，会读取DB N次，每次读取200条tweet。这个N次DB reads会很慢。所以pull model不是很合理。
</br>

#### 2.Push Model

为每个user建立一个list存储其news feed信息。当某user发一个tweet后，server将该tweet推送(FAN OUT)到该用户的好友的news feed列表之中。  
当user需要查看news feed时，只需从自己的list中读取最新的100条即可。

复杂度：  
Fanout a tweet needs N DB writes, 这步是异步，可以在后台进行，无需用户等待.  
Get NewsFeed needs 1 DB read.

Push模型的缺陷：
1)粉丝数目可能很大，导致 Fanout 过程很长，从而导致用户刷到新鲜事有延迟  
2)浪费系统资源去为很多僵尸粉创建新鲜事记录  
3)明星发帖会在短时间内为系统带来很大的处理压力  

但不会因为在数据库中创建的记录太多，而说它浪费disk，因为disk is cheap.

#### 3.如何选择

* Push  
双向好友关系，比如朋友圈
发帖比较少，实用性不高？？？

* Pull  
实时性比较高  
用户发帖很多  
单向好友关系

</br>

### 其他问题

#### 1.Follow and Unfollow

follow后，将关注对象的timeline合并到自己的news feed中。  
unfollow后，将它的tweets从自己的news feed中删除。

以上两步都是异步操作。

#### 2.存储likes、转发次数等

**De-normalize**
Denormalized是通过在不同的Table中存储同一份数据的(也就是说至少一份是冗余数据)的形式，来加速数据的查询。  
因为当数据只存储在一个固定的Table A的时候，如果访问其他TableB时需要同时取得关联的Table A的数据，则需要进行join之类的操作，这样会比较慢。

比如统计有多少人点赞了一个帖子，可以通过select count(*) from like_table where post_id=< id >的方式来获取。

但是也可以在post table中新增一个like_count列。每次点赞就+1。这里like_count就是一个 denormalized field，因为是可以通过 select count(*) 直接在 like_table 中获得的。

#### 3.Thundering Herd 惊群现象

在高并发的情况下，如果因为缓存过期或者淘汰算法等原因，使得某条很热的记录被从缓存中删除后，会出现短时间内大量的数据请求，从而导致**cache miss**现象。数据从DB到cache需要时间，数据请求会都去访问DB，导致DB崩溃。

