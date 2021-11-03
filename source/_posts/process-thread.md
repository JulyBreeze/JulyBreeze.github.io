---
title: Java 进程process和线程Thread
date: 2019-10-18 19:53:56
tags: java
---

### Process & Thread

Process is an executing program, one or more threads run in the context of the process. 这些threads共享所在的process的资源。每个process有独立的地址空间，互相不影响。每一个process对应一个JVM实例，多个thread共享JVM里的堆。(JVM是多线程的)

process是资源分配的最小单位；thread是CPU调度的最小单位。  
A thread is the basic unit to which operating system allocates processor time.  

单核cpu下的"concurrent"其实就是分给每个process的时间片很短，频率高，导致看起来像同时进行多个进程。

thread有自己的堆栈和局部变量，没有独立的地址空间。所以一个thread出问题，其所在的process也会出问题。  
process切换需要耗费的时间和资源比切换thread要大。


### Thread

When a Java program starts up, one thread begins running immediately. This is usually called the **main thread** of your program.
* "child" threads will be spawned from the main thread.主线程产生其他子线程
* It must be the last thread to finish execution. When the main thread stops, your program terminates.主线程最后一个运行

`Thread.currentThread()` return the reference to current thread.

A thread have 3 states: running, ready, and blocked.
Threads share memory, but process is isolated.

run(), start()的区别：  
这两个方法没有可比性。start创建一个新的子线程来执行方法；run只是Thread的一个普通方法

```java


```

```java


```



```java


```



```java


```



```java


```



```java


```