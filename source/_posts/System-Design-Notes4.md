---
title: System Design IV - MapReduce
date: 2019-08-27 22:44:32
tags: system design
---

### Introduction

MapReduce is a processing technique and a program model for distributed computing based on java. 用于大规模数据集(大于1TB)的并行运算.  
The MapReduce algorithm contains two important tasks, Map and Reduce:

* Map takes a set of data and converts it into another set of data, where individual elements are broken down into tuples(key/value pairs).
* Reduce takes the output from a map as an input, combines those data tuples.  
Map和Reduce的输入和输出都是key-value形式。

The major advantage of MapReduce is that it is easy to scale data processing over multiple computing nodes.

### Examples

#### 1.Word count

从一台机器for循环计算word的count，到多台机器并行分别for循环处理word的count，然后再合并结果。这两个方法对于大数据来说不合适。

MapReduce方法：一些机器负责把文章拆分成一个个的单词；另一些机器负责把word的数量合并。  
MapReduce是一个实现分布式运算的框架，自己需要学的就是map和reduce的逻辑，至于谁来拆分文章、中间传输问题由MR的框架实现，不需要考虑。

{%asset_image .png%}

```java
public class WordCount {
    public static class Map {
        public void map(String key, String value, OutputCollector<String, Integer> output) {
            // Write your code here
            // Output the results into output buffer.
            // Ps. output.collect(String key, int value);
        }
    }

    public static class Reduce {
        public void reduce(String key, Iterator<Integer> values,
                           OutputCollector<String, Integer> output) {
            // Write your code here
            // Output the results into output buffer.
            // Ps. output.collect(String key, int value);
        }
    }
}
```


#### 2.

{%asset_image .png%}

```java

```


