---
title: System Design 基本概念
date: 2019-06-19 20:25:03
tags: system design
---

### Basics

#### 1. web server

Web server有时候又称为Application server，提供HTTP/HTTPs服务。  
HTTP/HTTPs是超文本传输协议，用于browser和server之间的协议。一台较好的server一般每s可以服务1k次访问请求。

#### 2. Database

数据库一般和server打交道，private network才能访问到，支持CRUD增删查改。  
合理的架构中，web server和database server是不同的机器。

**常见数据库:**

* MySQL，PostgreSQL：关系型，存储查询较为复杂的数据表单，更高稳定性
* Memcached：最常用的key-value缓存系统。value不支持set/list的结构，不支持数据持久化；适合存储耗时的计算结果，或者缓存数据库中不经常改动的数据，或者经常被访问的数据  
memcache是一个cache db，可以由多台机器组成，并且可以多个server同时访问  
* Redis：最常用的key-value NoSQL数据库之一。value支持set/list这种结构；支持数据持久化；内存级访问速度，但低于memcached；可以用作cache/message queue/database
* Cassandra, HBase: NoSQL, key-value, key分为row-key和column-key，适合放查询请求简单的数据
* MongoDB: Document Based NoSQL，适合写多读少的数据
* Rocksdb：key-value NoSQL，value不支持set/list结构。常用于大公司的k-v storage的底层。

**SQL vs NoSQL:**  
1)SQL的column是在Schema中预先指定好的，不能随意添加。一个row是一条数据。  
NoSQL的column是动态的，可以随意添加。

2)SQL没有sharding功能，NoSQL大多自带sharding  
3)NoSQL不支持Transaction

**数据库选择原则:**

* 通常情况下SQL, NoSQL都可以
* 如果需要支持Transaction，就不能使用NoSQL  
Transaction是指对一些内容的操作同时成功或失败，比如A.money += 10, B.money -= 10.
* NoSQL在一些事上比如serialization, Multiple Indexes上需要自己做  
Serialization是指不同类型的数据存储时需要先serialize
* 想获得更快的速度等高性能，用NoSQL
* 如果需要sequential ID，用SQL，因为NoSQL的ID不是连续的

**NoSQL原理**  
是分布式数据库，便于scalability。  
可能存在的未解决问题是Query Language, Secondary Index, ACID transaction, Trust and Confidence

##### 1) Sharding

Sharding is a type of database partitioning that separates very large data into smaller, faster, more easily managed parts called data shards.  
按照一定的规则把数据拆分成不同的部分，放在不同的机器上。  

* Vertical Sharding  
不同的table放在不同的服务器上，但是依然存在单点失效问题
* Horizontal Sharding  
consistent Hashing algorithms

consistent hashing的介绍见：  
https://blog.csdn.net/u014708700/article/details/92759621

##### 2) Cassandra

是三层结构的NoSQL数据库。insert(row_key, column_key, value)

* row_key  
也就是hash-key，对应某台机器，cassandra根据这个决定把整条数据存放到哪里。   
任何查询都要带上这个key; 无法用这个key进行range query
* column_key  
是排序的，可以进行range query;可以是复合值  
* value  
通常是string。如果需要存很多信息，需要自己做serialization

#### 3. File System

是操作系统的组成部分之一，一般是目录系统。数据库系统是基于file system存在的，也就是说数据库的数据最终都是要存在file system上的。

file system的接口比较单一：

* 读某个file的从某个位置开始的多少字节的数据
* 在某个file的某个位置开始写入多少自己的数据
断电后数据依然存在  
非结构化数据适合直接存储在file system中，比如.jpg, .mp4这些  

#### 4. Cache

**Cache，缓存，是相对的概念，可以在内存、磁盘、CPU、服务器、客户端**。比如file system相对于network来说可以是一个cache，因为从本地获取html文件比从远程server获取要快。

cache可以理解为一个hashTable，key-value pair结构。  
在系统设计中默认其为mem cache，即内存中的cache。常见的Cache软件是Memcached/Redis。

通常把经常访问的数据放在Cache里来加速访问速度。Cache因为空间限制，经常需要淘汰一些不常用的数据，使用的策略比如LRU。

但是不能把数据全放入cache中，因为**cache不是持久化保存**，断电就会消失；而且cache比disk贵。
</br>

#### 5. DB & Cache

1.例子  
假设key是一个user_id，value是userInfo。cache和db里都有某user的信息。怎么保证cache和db里的数据一致？
以这个代码为例子分析：

```javascript
cache.delete(key); db.set(userInfo); // Eg.1
db.set(userInfo); cache.delete(key); // Eg.2
```

Eg.1中，如果前一个代码操作出问题了，后面的代码就不会执行，cache和db里的内容还是一致的；如果第二个操作出问题了，但cache里的数据已经删掉了，所以下次读数据会去db里得到，至少保证了db is the source of truth.

Eg.2中，如果第一个代码出问题了，后面的代码就不会运行。此时db中的userInfo没有更新成功，所以需要重新load代码运行，用户会收到提示信息，选择重试保存数据到db中。

2.**Cache-Through & Cache-Aside**：  
1)Cache-Through是server只与cache沟通，cache再和db沟通，把数据持久化。  
Redis是cache-through，读写都很快。可以理解为Redis里包括了一个cache和一个db。

2)Cache-Aside就是由server来进行DB和Cache的沟通，DB和Cache之间不直接沟通。  
Memcached/MySQL是cache-aside。


#### 6. Message Queue

进程间通信，或者同一进程之间的不同线程的通信方式。异步。  
任务比较慢或任务如果失败可能需要重试几次这种，需要消息队列。  
最常用的是RabbitMQ，Redis(可以作为cache，db等多用)，Amazon SQS

#### 7. Producer-Consumer solution using threads in Java

又称为bounded-buffer有界缓存区问题.

以下面多线程同步为例题，简单介绍一下：  
The multi-processes synchronization Description:  
Two processes, the producer and the consumer, which share a common, fixed-size buffer used as a queue. The producer generates data, put it into the buffer, and start again. At the same time, consumer is consuming the data(i.e. removing it from data) one piece at a time.

Problem:  
To make sure that the producer won't try to add data into the buffer if it's full and the consumer won't try to remove the data from an empty buffer.

Solution:  
The producer is either go to sleep or discard data if the buffer is full.  
The next time the consumer removes an item from the buffer, it notifies the producer who starts to fill the buffer again. If consumer find the buffer to be empty, it can go to sleep.  
The next time the producer puts data into the buffer, it wakes up the sleeping consumer.

An inadequate solution could result in a deadlock where both processed are waiting to be awakened.

答案代码：

```java
import java.util.LinkedList;

public class ThreadExample {
    public static void main(String[] args) throws InterruptedException{
        //object of a class that has both produce() and consumer() methods
        final PC pc = new PC();

        Thread t1 = new Thread(new Runnable()) {
            @Override
            public void run() {
                try{
                    pc.produce();
                }
                catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
        };

        Thread t2 = new Thread(new Runnable()){
            @Override
            public void run(){
                try{
                    pc.consum();
                }
                catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
        };

        //start both threads
        t1.start();
        t2.start();

        //t1 finishes befor t2
        t1.join();
        t2.join();
    }


    //this class has a list, producer and consumer
    public static class PC {
        //list is shared by producer and consumer
        LinkedList<Integer> list = new LinkedList<>();
        //size of the list is 2
        int capacity = 2;

        public void produce() throws InterruptedException {
            int value = 0;
            while(true) {
            //have a synchronized block so that 
            //only a producer or a consumer thread runs at a time.
                synchronized(this){
                    while(list.size() == capacity){
                        wait();
                    }
                    System.out.println("producer produced: " + value);

                    list.add(value++);
                    notify();//notify consumer, consumer can start consuming
                    //make the working of program easier to understand
                    //not display everything all at once
                    Thread.sleep(1000);
                }
            }
        }

        public void consume() throws InterruptedException {
            while(true) {
                synchronized(this){
                    while(list.size() == 0){
                        wait();
                    }

                    int value = list.removeFirst();
                    System.out.println("consumer consumed: " + value);

                    notify();
                    Thread.sleep(1000);
                }
            }
        }
    }
}
```

Output:

```html
Producer produced: 0
Producer produced: 1
Consumer consumed: 0
Consumer consumed: 1
Producer produced: 2
...
```

#### 8.Master-Slave

Master-Slave is a communication model where one device or process (known as the master) controls one or more other devices or processes (known as slaves).

##### 1) In Database

In Database, SQL uses master-slave to do replica:

* Master for Write/Read？？write
* Slave for Read only

Write Ahead Log
SQL数据库的任何Write操作，都会以log形式把这个操作做一份记录。Master每次有任何操作，就通知slave来读log，因此slave上的数据是有延迟的。  
如果master breaks down，就将某台slave升级为master。
