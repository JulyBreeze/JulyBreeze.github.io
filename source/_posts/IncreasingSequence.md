---
title: Longest Increasing Sequence例题集合
date: 2019-02-10 18:12:26
tags: 
- binarysearch
- DP
---
Given an unsorted array of integers.

### Find the length of LIS

1.DP O(n^2)
Let lis(i) = the length of the lis with its last element = nums[i].  
lis(i) = 1 + max(lis(j)), if nums[j] < nums[i], 0 < j < i; otherwise, lis(i) = 1.  
So this problem has many subproblems -> overlapping substructure property.


```java
public int lengthOfLIS(int[] nums) {
    //以i为结尾的lst的长度
    int[] dp = new int[nums.length];
    int max = 0;

    for(int i = 0; i < nums.length; i++){
        dp[i] = 1;
        for(int j = 0; j < i; j++){
            if(nums[j] < nums[i]){
                dp[i] = Math.max(dp[j] + 1, dp[i]);
            }
        } 
        max = Math.max(max, dp[i]);
    }
        
    return max;
}
```

2.Binary Search O(nlogn)
新建一个数组f，把nums[0]放入。
遍历nums数组，如果nums[i] > f数组中的最后一个数(或者说nums[i] > f中的最大值)，就放入f中；否则，替换f中第一个 >= nums[i]的数。
这个f数组的顺序不一定是LIS，但有数值的部分的长度 = LIS length。

例如：

```html
[4,10,4,3,8,9] LIS length = 3.
遍历nums每一步的f: 
n = 4, f = [4]
n = 10, f = [4, 10]
n = 4, f = [4, 10]
n = 3, f = [3, 10] 此时的f中的顺序不是LIS，但是长度没有变化，始终表示当前为止的LIS的长度
n = 8, f = [3, 8]
n = 9, f = [3, 8, 9]
```

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] res = new int[nums.length];
        int index = 0;
        
        for(int num : nums){
          //每到新的一补，此时的index指向的是res数组中的第一个空位
            if(index == 0 || num > res[index - 1]){
                res[index] = num;
                index++;
                continue;
            }
            
            int pos = binarySearch(res, index - 1, num);
            res[pos] = num;
        }
        
        return index;
    }
    
    //find the first num larger than target in [0, end]
    public int binarySearch(int[] nums, int end, int target){
        int start = 0;
        while(start + 1 < end){
            int mid = start + (end - start) / 2;
            if(nums[mid] >= target){
                end = mid;
            }else{
                start = mid;
            }
        }
        return nums[start] >= target ? start : end;
    }
}
```

### Find the number of LIS

0 <= j < i, only when nums[i] > nums[j], i depends on j.  
lens[i]代表以第 i 个数结尾的LIS的length。cnts[i]代表以第 i 个数结尾的LIS的count。
if lengths[j] + 1 == lengths[i], then counts[i] += counts[j]
if lengths[j] + 1 > lengths[i], counts[i] = counts[j], lengths[i] = lengths[j] + 1;

then find the longest in lengths array, sum all the counts[i] if lengths[i] == longest.

```java
class Solution {
    public int findNumberOfLIS(int[] nums) {
        int n = nums.length;
        int[] lengths = new int[n];
        int[] counts = new int[n];
        
        int longest = 0;
        for(int i = 0; i < n; i++){
            lengths[i] = 1;
            counts[i] = 1;
            for(int j = 0; j < i; j++){
                if(nums[i] > nums[j]){
                    if(lengths[j] + 1 == lengths[i]){
                        counts[i] += counts[j];
                    }else if(lengths[j] + 1 > lengths[i]){
                        lengths[i] = lengths[j] + 1;
                        counts[i] = counts[j];
                    }
                }
            }
            
            longest = Math.max(longest, lengths[i]);
        }
        
        int res = 0;
        for(int i = 0; i < n; i++){
            if(lengths[i] == longest){
                res += counts[i];
            }
        }
        
        return res;
    }
}
```

### Convert to strictly increasing array with min changes

LIS变种题目。  
给一个数组，可以改变其中的元素的值，使得array称为严格递增。求最少要改动多少？

思路:  
The numbers which are already a part of LIS need not to be changed.  
So the minimum elements to change = size of array - number of elements in LIS.

We do not consider those elements as part of LIS which can't form LIS by inserting elements in middle.  
Eg. { 1, 2, 5, 3, 4 }, we consider length of LIS as three {1, 2, 5}, not as {1, 2, 3, 4}.  Because we cant make a strictly increasing array of integers with this LIS.

```java
public int minRemove(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];

    int maxLen = 0;//the length of lis
    for(int i = 0; i < n; i++){
        dp[i] = 1;
        for(int j = 0; j < i; j++){
            //第二个判断条件就表明nums[j]和nums[i]之间可以插入新的元素，仍然是LIS
            // >= 的 = 是如果这个lis是position consecutive也是可以的
            if(nums[j] < nums[i] && (nums[i] - nums[j]) >= (i - j)){
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.max(maxLen, dp[i]);
    }

    return n - maxLen;
}
```

### Russian Doll Envelopes

先对envelops排序。先按照width从小到大排，如果width相同，则按height由大到小排序。  
然后对height做longest increasing sequences。

因为这样可以保证依次遍历数组的时候，后面的width始终比前面的大; 当width相同时，取最大的height。  
如果height由小到大排序，当width相同时，height大的也被放入到了res数组中，这种情况并不符合。

例：

``` html
[4,5],[4,6],[6,7],[2,3],[1,1], output = 4

如果width和height都按照从小到大排序：
[1,1],[2,3],[4,5],[4,6],[6,7] 
对height做LIS: 1 3 5 6 7

如果按照当widht相同对height由大到小排序:
[1,1],[2,3],[4,6],[4,5],[6,7] 
对height做LIS: 1 3 6 -> 1 3 5 7
```

```java
public int maxEnvelopes(int[][] envelopes) {
    Arrays.sort(envelopes, new Comparator<int[]>(){
        public int compare(int[] a, int[] b){
            if(a[0] == b[0]){
                return b[1] - a[1];
            }
                
            return a[0] - b[0];
        }
    });
        
    int[] res = new int[envelopes.length];
    int index = 0;
        
    for(int[] item : envelopes){
        int pos = Arrays.binarySearch(res, 0, index, item[1]);
        if(pos < 0){
            pos = -pos - 1;
        }
        res[pos] = item[1];
        if(pos == index){
            index++;
        }
    }
        
    return index;
}
```

follow up：信封可以旋转，怎么求最长序列？  
预处理，把旋转之后的信封也加入到原数组中，再按照本题的方法进行求解。

补充：

```html
Arrays.binarySearch(object[], fromIndex, toIndex, target),
search range is [fromIndex, toIndex), return index of the search key if found;
otherwise, return -insertion point - 1

Insertion point: 数组中第一个 > target的数的index。
如果该区间内的所有数都小于target，insert point = toIndex.
```