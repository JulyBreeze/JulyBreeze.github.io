---
title: Java Thread(线程)
date: 2019-07-14 19:58:57
tags:
---

#### 基本知识

1)进程process和线程thread
Thread is the

2)

#### 多线程创建的方式

1)Thread class  
继承Thread类，重写Thread类中的run()便可以实现多线程。Thread class中的start()用于启动新的线程。线程启动后，系统会自动调用run()

例子：  
第一个代码里，只会运行MyThread里的print语句，因为这是一个单线程的程序：

```java
class MyThread{
    public void run() {
        while(true){
            System.out.println("MyThread class run() is running");
        }
    }
}

public class example {
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        mt.run();
        while(true){
            System.out.println("main class is running");
        }
    }
}
```

为了能打印两个循环里的语句，使用下面这个代码：

```java
class MyThread extends Thread {
    public void run() {
        while(true){
            System.out.println("MyThread class run() is running");
        }
    }
}

public class example {
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        mt.start();
        while(true){
            System.out.println("main class is running");
        }
    }
}
```

2)Runnable class  
由于java是单继承，一个class继承了某父类后就无法继承Thread类，所以有另一种构造方法Thread(Runnable obj)。  
Runnable是一个Interface，有一个run() method。当通过这个构造方法创建一个对象时，只需要在该构造方法里传递一个用于实现Runnable Interface的实例对象。

```java
class MyThread implements Runnable {
    public void run(){
        while(true){
            System.out.println("mythread class run() is running");
        }
    }
}
public class example {
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        Thread thread = new Thread(myThread);
        thread.start();
        while(true){
            System.out.println("main class is running");
        }
    }
}
```

#### 例题

##### 1.Print FooBar Alternately
https://leetcode.com/problems/print-foobar-alternately/description/

##### 2.Print Zero Even Odd
https://leetcode.com/problems/print-zero-even-odd/discuss/