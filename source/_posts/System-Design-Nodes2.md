---
title: System Design II
date: 2019-08-26 19:28:46
tags: system design
---

 
### I. Design User System

#### Scenario

* Features  
Register, Login, Search, Edit User Profile.  
查询的需求量最大
* Computation  
DAU = 100M.  
QPS for register/login/edit = 100M * 0.1 / 86400 = 100  
Peak QPS = 100 * 3 = 300

#### Service

* Auth Service  
负责register and login
* User Service  
负责用户信息的存储和查询
* FriendShip Service  
负责好友关系的存储

#### Storage  

**一般来说，给人用的系统都是读多写少，给机器用的系统是写多读少。一个读多写少的系统，Read多就一定要用cache进行优化！**

1)对于User service, 用户系统的特点是 **读多写少**，所以应该放在cache里进行优化。但是user info应该放在可以数据持久化的存储系统中，所以不能用memcached。

2)对于Auth Service，需要一个session会话表。  
User Login后，创建一个session object，把session key作为cookie返回给浏览器，浏览器将这个值保存在浏览器的cookie中。User每次向server发送访问请求，都会带上这个网站的所有cookie。server检查到cookie中的session_key是有效的，就认为用户登录了。  
**(Session存放在server端；Cookie存放在client端)**

User Logout后，session_table里会把他的数据删掉。

{%asset_image sessionTable.png%}

Session table的存放位置：  
一般来说db和cache里都可以，如果cache断电了，让所有的users都logout是可以的，放在db中肯定更好，但由于用户系统是读多的，应该存放在db + cache里，用cache来优化。

3)FriendShip Service  
单向好友关系：twitter，ins，微博等  
双向好友关系：whatsapp, fb，微信等

用cassandra来存储friendship service，见下图：

{%asset_image cassandra_frienship.png%}

#### Scale

为了防止Single Point Failure，需要两件事：Sharding & Replica。  

1)Sharding
按照一定的规则把数据拆分成不同的部分，放在不同的机器上。

* Vertical Sharding  
不同的table放在不同的服务器上，但是依然存在单点失效问题
* Horizontal Sharding

2)Replica  
通常一式三份；同时还能做到分摊read请求。是实时的，数据写入的时候就会复制了。这和Backup不同，backup是周期性的备份。

* SQL，用master-slave来实现
* NoSQL，在consistent hashing环上顺时针找3台virtual nodes对应的机器来实现

</br>

### II. Design TinyURL

**1. Scenario**  
Features:  
TinyURL负责把long url and short url来回转换返回给user。注意不是用户通过TinyURL来访问短网址。  
Long和short是一对多的关系。

Computation:  
1)Create a tiny url  
DAU = 100M  
QPS = 100M * 0.1 / 86400 = 100(假设每个用户平均每天产生一条url)  
Peak QPS = 100 * 2 = 200

2)Click a tiny url  
DAU = 100M
Average Read QPS = 100M * 1 / 86400 = 1K
Peak Read QPS = 2K

综上，2K个QPS，用一台MySQL数据库可以支持

3)New URL Storage  
100M * 0.1 = 10M  
10M * 100(100 is the length of url) = 1GB

**2. Service**  
URL Service  
1)逻辑设计：encode(long_url), decode(short_url)  
2)接口设计：  
想要访问short_url: GET /<short_url>, return a redirect response.  
想要把long转换成short：POST/data/shorten, Data={long_url}, return a short url

**3. Storage**  
NoSQL和SQL取决于选择的算法是什么

**算法：**  
1)随机生成short url，用数据库去重。如果没有用过，就绑定给long url.  
应用：比如定的票，发给用户的查询码就是随机生成的。这个查询码不像验证码可以相同，
而是必须不同，也不能有规律生成，否则容易被别人破解而暴露信息。

需要分别对shortKey和longKey分别进行索引Index。
如果用SQL，就一张表可以；如果用NoSQL，需要两张表

2)进制转换Base62  
将6位的short url看成是一个62位的进制数，即这6个位置上，每一个位置可以放0-9, a-z, A-Z的任意数字。  
每个short url对应到一个整数ID，该ID对应于数据库中的Primary Key - Sequential ID.

缺点，依赖于全局的自增ID，容易被破解。

由于需要自增ID，所以只能用SQL。表单里是ID和long url。因为short url可以和ID换算，所以可以不存储在表单里。

**4. Scale**  
A) How to reduce response time?   
1)Cache  
因为读多写少，可以提高Web server与db server之间的访问效率，利用Cache Aside:

{%asset_image tinyURLScale.png%}

2)地理位置信息提速

* 优化server的访问速度  
不同的地区使用不同的web server, 这样可以通过DNS解析不同地区的user到不同的server
* 优化data的访问速度  
使用Centrailized MySQL + distributed memcahced  
一个MySQL配多个memcached，memcached跨地区分布

B) How to scale?  
当cache不够，或者write操作越来越多时，会需要越多的db servers。增加多台DB server可以优化storage和QPS。对于TinyURL来说，更多的是为了解决QPS的问题。  
那么就需要将data sharding到不同的数据库上。这里有点乱，建议看PPT

C) 扩展Short key from 6bits to 7bits  
在short key前面加一个前置位，该前置位的值由hash(long_url) % 62得到。由这个前置位作为sharding key，来判断去哪台机器上获取数据。  
这样就可以同时通过short url和long url得到sharding key，而不用广播，直接找到所在的数据库。

D) 按照网站地区来进行sharding，比如美国网站的数据放在美国的DB中

最终架构图：  
{%asset_image tinyURLWorkSolution.png%}

**5. 其他可能的问题**  
custom url功能，需要另外创建一个custom url table，而不是在原有的url table上增加一个新的列。

当创建custom short url时，去custom table里查询是否已经存在，不存在就插入新数据。  
当要创建普通的短链接时，先去custom table里查询是否存在，再去URL table里查询是否存在和插入新数据。  
当要得到某short url对应的long url时，先查询custom table，再去查询URL table。
