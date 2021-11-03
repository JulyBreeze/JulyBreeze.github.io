---
title: Quick Sort & Merge Sort
date: 2019-02-09 10:50:06
tags: 
- sort
- Divide and Conquer
---
### Quick Sort 从整体到局部
选取一个pivot，>的放右边，<放左边，=的情况左右都可以
注意 = 号这个是为了避免极端情况，比如几乎全是1，只有几个数不一样，如果严格的把=归于左边或右边，partition的线可能就不在中间。希望partition的线在中间，是均匀的划分。
体现在code里的那两个while里的条件，就是 < 和 > 。

```java
public class Solution {
    public void sortIntegers(int[] nums) {
        if(nums == null || nums.length == 0){
            return;
        }
        quickSort(nums, 0, nums.length - 1);
    }
    
    public void quickSort(int[] nums, int start, int end) {
        if(start >= end){
            return;
        }
        //1. pivot is value, not index
        int pivot = nums[(start + end) / 2];
        int left = start, right = end;
        
        //2. left <= right, not <
        //否则的话会造成overflow，比如[3,1,2,5,4]，确保递归的时候两个区间不要有重合。
        while(left <= right) {
            //3.nums[left] < pivot not <=
            while(left <= right && nums[left] < pivot){
                left++;
            }
            while(left <= right && nums[right] > pivot){
                right--;
            }

            if(left <= right){
                int temp = nums[left];
                nums[left] = nums[right];
                nums[right] = temp;
                left++;
                right--;
            }
        }
        //while 后，left在右边，right在左边
        quickSort(nums, start, right);
        quickSort(nums, left, end);
    }
}
```

### Merge Sort 从局部到整体
将数据分成两部分，将两个子部分进行递归的merge排序；然后将已经有序的两个子部分进行合并，最终完成排序。
```java
//temp array不能放在mergeSort 和 merge函数中，否则当n数组很大的时候，会导致mle。
public class Solution {
    public void sortIntegers(int[] nums) {
        if(nums == null || nums.length == 0){
            return;
        }
        int[] temp = new int[nums.length];
        mergeSort(nums, 0, nums.length - 1, temp);
    }
    
    public void mergeSort(int[] nums, int start, int end, int[] temp) {
        if(start >= end){
            return;
        }
        
        mergeSort(nums, start, (start + end)/2, temp);
        mergeSort(nums, (start + end)/2 + 1, end, temp);
        merge(nums, start, end, temp);
    }
    
    public void merge(int[] nums, int start, int end, int[] temp) {
        int mid = (start + end)/2;
        int leftIndex = start;
        int rightIndex = mid + 1;
        int i = start;//注意这里，遍历temp的指针是从start开始的
        
        while(leftIndex <= mid && rightIndex <= end){
            if(nums[leftIndex] < nums[rightIndex]){
                temp[i++] = nums[leftIndex++];
            }else{
                temp[i++] = nums[rightIndex++];
            }
        }
        
        while(leftIndex <= mid) {
            temp[i++] = nums[leftIndex++];
        }
        while(rightIndex <= end) {
            temp[i++] = nums[rightIndex++];
        }
        
        for(int k = start; k <= end; k++){
            nums[k] = temp[k];
        }
    }
}
```

### Compare
|   | time  | space  | other  |
|---|---|---|---|
|quickSort  |O(nlogn) 极端时O(n^2)|O(1)   |   |
|mergeSort  |O(nlogn)   |O(n)   | stable  |

MergeSort是一种稳定的排序算法，quickSort不然。(稳定排序：duplicate的数，1和1'，如果排序结束后，1还是在1'前，保证原来顺序就是稳定排序)

### Example: Sort List
因为是List，用mergesort比较好，quickSort还要找tail点。 
```java
class Solution {
    public ListNode sortList(ListNode head) {
        //base case
        if(head == null)
            return head;
        if(head.next == null)
            return head;

        //slow指向的是中间偏左的点
        //if fast= head, 则slow指向的是中间偏右的点
        //因为找到中间点后，需要把两个子list分割开，所以选择找中间偏左的点
        ListNode slow = head, fast = head.next;
        while(fast != null && fast.next != null){
            slow = slow.next;
            fast = fast.next.next;
        }

        ListNode secondHead = slow.next;
        slow.next = null;

        ListNode h1 = sortList(head);
        ListNode h2 = sortList(secondHead);

        return merge(h1, h2);
    }

    //用while来写merge函数
    public ListNode merge(ListNode h1, ListNode h2) {
        ListNode dummyhead = new ListNode(0), p = dummyhead;

        while(h1 != null && h2 != null){
            if(h1.val < h2.val){
                p.next = h1;
                h1 = h1.next;
            }else{
                p.next = h2;
                h2 = h2.next;
            }
            p = p.next; 
        }

        //如果上面的while结束后，h2那部分已经都比较完了，就把h1剩下的部分都贴到p的后面
        if(h1 != null)
            p.next = h1;
        //同理
        if(h2 != null)
            p.next = h2;

        return dummyhead.next; 
    }   
}
```