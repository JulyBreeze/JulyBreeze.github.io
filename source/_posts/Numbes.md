---
title: Numbers类型题
date: 2019-02-11 12:12:41
tags:
---

### Missing Number II
Giving a string with number from 1-n in random order, but miss 1 number. Find that number.

```java
public class Solution {
    int missingNumber;
    public int findMissing2(int n, String str) {
        missingNumber = -1;
        dfs(n, str, 0, new boolean[n + 1], 0);
        
        return missingNumber;
    }
    
    public void dfs(int n, String str, int start, boolean[] visited, int count) {
        if (missingNumber != -1) {
            return;
        }

        if(start == str.length() && count == n - 1){
            for(int i = 1; i <= n; i++){
                if(!visited[i]){
                    missingNumber = i;
                    return;
                }
            }
        }
        
        if(str.charAt(start) == '0'){
            return;
        }
        
        for(int i = 1; i <= 2 && i + start <= str.length(); i++){
            int num = Integer.valueOf(str.substring(start, start + i));
            
            if(num < 1 || num > n || visited[num]){
                continue;
            }
            
            visited[num] = true;
            dfs(n, str, start + i, visited, count + 1);
            visited[num] = false;
        }
    }
}
```


### Find First Positive Number
Given an unsorted integer array, find the smallest missing positive integer.

有n个数，有n个buckets，编号为[0, n-1]，把这些数放在对应的bucket里。  
然后遍历buckets，如果有一个为空，则这个bucket id + 1就是要找的数。  
要求O(1)space，所以用swap在数组上操作。  
当nums[i] != i + 1时，swap(i, nums[i] - 1)，从nums[i] - 1处换来的数还是要判断是否在正确的位置上，所以继续swap，用while。  
注意这里要判断当num[i] != nums[nums[i] - 1]时才swap，是为了防止死循环。  

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int i = 0;
        while(i < nums.length){
            if(nums[i] > 0 && nums[i] < nums.length &&
               nums[i] != i + 1 && nums[i] != nums[nums[i] - 1]){
                swap(nums, i, nums[i] - 1);
            }else{
                i++;
            }
        }
        
        for(i = 0; i < nums.length; i++){
            if(nums[i] != i + 1){
                return i + 1;
            }
        }
        //if all the numbers in the array are in correct place, 
        //then, the first positive number=(length-1+1)+1 = length + 1
        return nums.length + 1;
    }
    
    private void swap(int[] A, int i, int j){
        int temp = A[i];
        A[i] = A[j];
        A[j] = temp;
    }
}
```
### Find Smallest Prime Missing Number
Given an array of prime numbers, find the smallest missing prime number.  
Enumeration: find the smallest prime number, get its next prime number and check if the next is in the array.

### Ugly Number II
Ugly numbers are positive numbers whose prime factors only include 2, 3, 5. Write a program to find the n-th ugly number.


1.O(nlogn)  
因为会有重复的数，所以用treeSet，如果用pq，那么就要手动poll出duplicates。  
只余为什么这个用int就越界，而dp就可以的原因不明白？

```java
public int nthUglyNumber(int n) {
    TreeSet<Long> set = new TreeSet<>();
    set.add(1L);
        
    for(int i = 1; i < n; i++){
        long cur = set.pollFirst();
            
        set.add(cur * 2);
        set.add(cur * 3);
        set.add(cur * 5);
    }
        
    return set.pollFirst().intValue();
}
```

2.O(n)  
Each subsequence is the ugly number * 2, 3, 5, so each step choose the minimum from three candidates.  
当前循环选到谁，那么就把该数对应的指针移动一格。
```java
class Solution {
    public int nthUglyNumber(int n) {
        int[] ugly = new int[n];
        ugly[0] = 1;
        
        int factor2 = 2, factor3 = 3, factor5 =5;
        int index2 = 1, index3 = 1, index5 = 1;
        
        for(int i = 1; i < n; i++){
            int min = Math.min(Math.min(factor2, factor3), factor5);
            ugly[i] = min;
            
            if(factor2 == min)
                factor2 = ugly[index2++] * 2;
            
            if(factor3 == min)
                factor3 = ugly[index3++] * 3;
            
            if(factor5 == min)
                factor5 = ugly[index5++] * 5;
        }
        
        return ugly[n - 1];
    }
}
```