---
title: Greedy例题
date: 2020-05-12 21:54:40
tags: greedy
---

### 1. Set Intersection Size At Least Two  

题目：  
给一个数组intervals[][]，每一个[a, b]表示consecutive integers from a to b, inclusive. 求一个set, 每个interval里至少有两个元素在这个set里，求这个set的最短长度。注意要求的Set中的元素没有要求是连续的。

思路：  
因为希望set越小越好，所以如果有两个interval a and b部分重合，那么对于interval a肯定选择最大的两个数。
如果interval b和a的重合部分不够两个，那么为了能使得S最小，在b中再选择最大的那个数，这个最大的数可以在当b与后面的interval重合时派上用场。  
如果一个interval包含了另一个interval，那么这个大的interval可以无需考虑。

因此需要对array排序，只需要按照end的值来由小到大的排序，无需考虑start值
比如[3, 5], [2, 10]，如果按照start来排序，就是[2,10] [3,5]
但按照end排序，[3,5] [2,10]。 [3,5]选完元素后，[2,10]自然就有两个元素已经被选了。
所以这个排序的目的是eliminate the intervals which fully overlap another interval.

Greedy
每次拿set中的最大的两个元素和当前的interval进行比较：
1)If there is no elements of current interval in set, choose two biggest number in this interval
2)If there is one number picked up, pick up the biggest number
3)If there are two numbers already picked up, skip

```java
class Solution {
    public int intersectionSizeTwo(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> (a[1] - b[1]));
        
        int res = 0;
        int max1 = -1, max2 = -1; //表示set中最大的两个数，再次注意set中的元素不一定是连续的
        for(int[] interval : intervals) {
            int start = interval[0], end = interval[1];
            //情况1
            if(start > max1) {
                max1 = end;
                max2 = end - 1;
                res += 2;
            } 
            //情况2，表示该interval中只有val == max的元素在set中
            //所以需要选择第二个元素放入set中，这个元素越大越好
            else if(start > max2) {
                //对于倒数第二大的max2,如果当前interval end > max1
                //那么set中的最大值就变成了end，此时max2 = max1
                if(end > max1) max2 = max1;
                //如果end == max1，表示end已经存在set中了
                //因此选择interval中倒数第二大的元素放入set中
                else if(end == max1) max2 = end - 1;
                max1 = end;
                res++;
            }
        }
        return res;
    }
}
```