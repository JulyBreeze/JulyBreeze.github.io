---
title: Quick Select
date: 2019-02-09 10:50:34
tags:
- quick select
---
### Quick Select
比如说find kth largest element in an unsorted array; find median; find kth smallest, etc. 用quick select方法，其中需要用到quickSort的partition部分，基本思想是一样的。先找pivot，然后从大到小排序；如果k在前半部分，就继续在前半部分quickSort，反之在后半部分。

时间复杂度是O(n)：
T(n) = O(n) + T(n/2) = O(n) + O(n/2) +T(n/4) = O(n) + O(n/2) + O(n/4) + … + O(1) = O(n)  
Quickselect only recurses into one side – the side with the element it is searching for. This reduces the average complexity from O(nlogn) to O(n), with a worst case of O(n^2).

有五点需要注意。
```java
//quickselsect
class Solution {
    public int findKthLargest(int[] nums, int k) {
        if(nums == null || nums.length == 0)
            return -1;
        return quickSelect(nums, 0, nums.length - 1, k);
    }
    
    public int quickSelect(int[] nums, int start, int end, int k) {    
      //1.注意这里的返回情况
        if(start == end)
            return nums[start];
        
        int left = start, right = end;
        int pivot = nums[(left + right)/2];
        
        //2.排序要从大到小排序，所以比pivot大的放在左边
        while(left <= right){
            while(left <= right && nums[left] > pivot)
                left++;
            while(left <= right && nums[right] < pivot)
                right--;
            
            if(left <= right){
                int temp = nums[left];
                nums[left] = nums[right];
                nums[right] = temp;
                left++;
                right--;
            }
        }

        //3.执行到这，可能有两种情况，j,i 或者 j,x,i。
        //后者因为经过if后，left和right相等后还有个left++，right--的情况
        if(start + k - 1 <= right)
            return quickSelect(nums, start, right, k);
        if(start + k - 1 >= left)
        //在后半部找，去掉前半部[start, left)
        //4.比如本来要找第10个最大的数，去掉前半部分5个数，那么就是要在后半部找第5个最大的数。
            return quickSelect(nums, left, end, k - (left - start));
            
        //5.如果出现了right, xx, left的情况，经过前面的return，剩下的情况就是第k个最大的数就是中间这个xx
        return nums[right + 1];
    }
}
```

### Example: Kth Smallest Element in BST
Follow up：  
What if the BST is often modified (insert/delete operations) and you need to find the kth smallest number frequently? How would you optimize the kth Smallest routine?

使用quick select，如果多次查询的话，可以给每个节点统计其子节点个数，这个过程只需要做一次。查询可以很快。  
O(n)
```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        Map<TreeNode, Integer> numOfChildren = new HashMap<>();//子节点个数包括自身。
        countNodes(root, numOfChildren);
        
        return quickSelectOnTree(root, k, numOfChildren);
    }
    
    public int countNodes(TreeNode root, Map<TreeNode, Integer> map){
        if(root == null){
            return 0;
        } 
        
        int left = countNodes(root.left, map);
        int right = countNodes(root.right, map);
        
        map.put(root, left + right + 1);
        return left + right + 1;
    }
    
    public int quickSelectOnTree(TreeNode root, int k, Map<TreeNode, Integer> numOfChildren){
        if(root == null){
            return -1;
        }
        
        int left = root.left == null ? 0 : numOfChildren.get(root.left);
        
        if(k <= left){
            return quickSelectOnTree(root.left, k, numOfChildren);
        }
        
        if(left + 1 == k){
            return root.val;
        }
     
        return quickSelectOnTree(root.right, k - left - 1, numOfChildren);
    }
}
```