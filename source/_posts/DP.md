---
title: DP
date: 2019-02-08 12:24:50
tags: DP
---

### 1.动态规划常见题型
1.通常用于以下几种：
最优解/max,min
存不存在/yes,no
多少个可行解

六大问题：  
1）坐标型 dp[i]表示从起点到坐标i  
2）序列行 dp[i]表示前i个元素  
3）背包型  
4）区间型  
5）划分型  
6）双序列行

### 2.滚动数组
需要多少个状态，就new多少个。  
例如：如果某状态i只与i - 1有关，就new int[2]

```html
f[i] = max(f[i-1], f[i-2])

f[i] = max(f[(i-1) % 2], f[(i-2) % 2])
```

### 3.循环数组的解决办法：

a) 取反
b) 分裂
c) 倍增

例题：
[House Robber II](House-RobberI-II-III.md)