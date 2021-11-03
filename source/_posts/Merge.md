---
title: Merge
date: 2019-02-14 18:32:24
tags: merge
---

关于Merge Array, List, Interval问题的总结

### Merge Array

#### Merge one array to another
已知array排好了序，假设merge B to A. 双指针  
如果从头开始比较，如果A[i] > B[j]，那么此时A[i]应题换成B[j]，有关原A[i]的处理就有点麻烦。  
所以从数组的尾部开始比较:  
i = A.length - 1, j = B.length - 1, index = A.length + B.length - 1.  
A[index--] = compare A[i] with B[j]

#### Merge two arrays to a new array
两个数组没有排序，可以先排序然后用双指针做。  
但如果两个数组中有一个特别大，该怎么优化？  
遍历大的数组1，对于每个num1，在小的数组2中做binary search，找到第一个 >= num1的数的pos。  
然后就可以把数组2中 < pos的所有数都放入res中，再把num1放入res中。

```java
public class Solution {
    public int[] mergeSortedArray(int[] A, int[] B) {
        if(A.length < B.length){
            return mergehelper(A, B);
        }
        return mergehelper(B, A);
    }
    //do binary search in nums1(smaller array)
    public int[] mergehelper(int[] nums1, int[] nums2){
        int len1 = nums1.length, len2 = nums2.length;
        int[] ans = new int[len1 + len2];
        
        int index = 0;
        int p1 = 0;
        for(int p2 = 0; p2 < len2; p2++){
            int pos = binarySearch(nums1, nums2[p2]);
            //find index of the first number >= target
            while(p1 < pos){
                ans[index++] = nums1[p1++];
            }
            ans[index++] = nums2[p2];
        }
        
        while(p1 < len1){
            ans[index++] = nums1[p1++];
        }
        return ans;
    }
    
    //return the position of target in arr 或者说 target不存在时就返回下一位数
    public int binarySearch(int[] arr, int target){
        int start = 0, end = arr.length - 1;
        while(start <= end){
            int mid = start + (end - start)/2;
            
            if(target < arr[mid]){
                end = mid - 1;
            }else if(target > arr[mid]){
                start = mid + 1;
            }else{
                return mid;
            }
        }
        
        //当while结束时，end start的位置是end在前，所以返回start
        return start;
    }

    //用binary search模板做的
    public int binarySearch(int[] nums, int target){
        int start = 0, end = nums.length - 1;
        
        while(start + 1 < end){
            int mid = start + (end - start) / 2;
            if(nums[mid] > target){
                end = mid;
            }else{
                start = mid;
            }
        }
        
        if(nums[start] >=  target) return start;
        if(nums[end] >= target) return end;
        //这里是因为如果target比数组中的所有数都大,得到的end = nums.length - 1
        //所以需要返回end + 1
        return end + 1;
    }
}
```

#### Merge K Sorted Arrays
注意所给的是已经排好序的。PriorityQueue
```java
public class Solution {
    class Pair{
        int row, col;
        public Pair(int row, int col){
            this.row = row;
            this.col = col;
        }
    }

    public int[] mergekSortedArrays(int[][] arrays) {
        int k = arrays.length;
        //pair是坐标，但queue里是按照value来排序的
        Queue<Pair> queue = new PriorityQueue<>(k, new Comparator<Pair>(){
            public int compare(Pair o1, Pair o2){
                return arrays[o1.row][o1.col] - arrays[o2.row][o2.col];
            }
        });
        
        int totalsize = 0;
        for(int i = 0; i < k; i++){
            if(arrays[i].length > 0){
                queue.add(new Pair(i, 0));
                totalsize += arrays[i].length;
            }
        }
        
        int[] results = new int[totalsize];
        int index = 0;
        while(!queue.isEmpty()){
            Pair pair = queue.poll();
            results[index++] = arrays[pair.row][pair.col];
            pair.col++;
            
            if(pair.col < arrays[pair.row].length){
                queue.offer(pair);
            }
        }
        return results;
    }
}
```

### Merge List
#### Merge K Sorted List
```java
public class Solution {
    
    public ListNode mergeKLists(List<ListNode> lists) {  
        if(lists.size() == 0){
            return null;
        }
        
        return helper(lists, 0, lists.size() - 1);
    }
    
    public ListNode helper(List<ListNode> lists, int start, int end){
        if(start == end){
            return lists.get(start);
        }
        
        int mid = (start + end)/2;
        ListNode left = helper(lists, start, mid);
        ListNode right = helper(lists, mid+1, end);
        
        return mergeTwoLists(left, right);
    }
```