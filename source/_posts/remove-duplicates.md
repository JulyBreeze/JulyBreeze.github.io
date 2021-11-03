---
title: remove-duplicates
date: 2019-02-12 00:54:15
tags:
---
Given a sorted array nums, remove the duplicates in-place and return the new length.

### Remove Duplicates from Sorted Array
当nums[i] == nums[j], j++  
当nums[i] != nums[j], nums[i + 1] = nums[j], i++, j++  
所以写成for循环即可。

```java
public int removeDuplicates(int[] nums) {
    int i = 0;
    for(int j = 0; j < nums.length; j++){
        if(nums[i] != nums[j]){
            i++;
            nums[i] = nums[j];
        }
    }
    return i + 1;
}
```

### Remove Duplicates II(at Most K)
duplicates appeared at most twice.
Keep the first k elements as same. Start from kth index.  
If nums[i - k] == nums[j], then skip element at j, continue with next element.  
If nums[i - k] != nums[j], then 

```java
public int removeDuplicates(int[] nums, int k) {
	if(nums.length < k){
       return nums.length;
    }
    //i 是符合条件的array的后一位
    int i, j;
    for(i = k, j = k; j < nums.length; j++){
        if(nums[j] != nums[i - k]){
            nums[i] = nums[j];
            i++;
         }
    }
    return i; 
}
```


```java
public int removeDuplicates(int[] nums, int k) {
        if (nums.length == 0) return 0;
        
        int i = 0; //每次赋值完后，i都指向的是符合条件的array的最后一位数
        int count = 1;
        for(int j = 1; j < nums.length; j++){
            if(nums[i] != nums[j]){
                i++;
                nums[i] = nums[j];
                count = 1;
            }else{
            //把这个window里的所有元素都赋值成同一个数
                if(count < k){
                    i++;
                    nums[i] = nums[j];
                    count++;
                }
            }
        }
        
        return i + 1;
    }
```
