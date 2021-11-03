---
title: House RobberI II III
date: 2019-02-08 10:34:53
tags: DP
---

### 1.I
Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.

Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.

```java
public int rob(int[] nums) {
  //dp[i][j] the max cash of first i houses
  //j = 0, don't robb house(i-1), j = 1, robb
  int[][] dp = new int[nums.length + 1][2];

  for(int i = 1; i <= nums.length; i++){
      dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1]);
      dp[i][1] = dp[i-1][0] + nums[i - 1];
  }
  return Math.max(dp[nums.length][0], dp[nums.length][1]);
}

//at each step i, the max value of choose or not choose i influence next step
//2D array -> 1D array
public int rob(int[] nums) {
  if(nums == null || nums.length == 0){
     return 0;
  }

  //the max cash of robbing first i houses
  int[] dp = new int[nums.length + 1];
  dp[0] = 0;
  dp[1] = nums[0];

  for(int i = 2; i <= nums.length; i++){
      dp[i] = Math.max(dp[i-1], dp[i-2] + nums[i-1]);
  }

  return dp[nums.length];
}
```

进一步优化空间：对于每一步i，只需要知道i-1和i-2这两家，就相当于三个指针不断移动。第三个指针可以做成temp指针

```java
//1. 滚动数组
public int rob(int[] nums) {
    if(nums == null || nums.length == 0){
        return 0;
    }

    int[] dp = new int[2];
    dp[0] = 0;
    dp[1] = nums[0];

    for(int i = 2; i <= nums.length; i++){
        dp[i % 2] = Math.max(dp[(i-1) % 2], dp[(i-2) % 2] + nums[i - 1]);
    }

    return dp[nums.length % 2];
}

2.
public int rob(int[] nums) {
    int preMax = 0;
    int curMax = 0;

    for(int num : nums){
        int temp = curMax;
        curMax = Math.max(curMax, preMax + num);
        preMax = temp;
    }

    return curMax;
}
```

### 2.II
All houses at this place are arranged in a circle.
Let the house be [1, n]. The problem can be divided to two cases:  
1) choose 1: [1, n-1]
2) not choose 1: [2, n]
For each case, the problem degenerated to the Question I.

```java
public int rob(int[] nums) {
    if(nums.length == 0) return 0;
    if(nums.length == 1) return nums[0];

    int[] start1 = new int[nums.length + 1];
    int[] start2 = new int[nums.length + 1];

    start1[0] = 0;
    start1[1] = nums[0];

    start2[0] = 0;
    start2[1] = 0;

    for(int i = 2; i <= nums.length; i++){
        start1[i] = Math.max(start1[i - 1], start1[i - 2] + nums[i - 1]);
        start2[i] = Math.max(start2[i - 1], start2[i - 2] + nums[i - 1]);
    }

    return Math.max(start1[nums.length - 1], start2[nums.length]);
}
```

### 3.III
All houses forms a binary tree. The entrance is the root. It will automatically contact the police if two directly-linked houses were broken into on the same night.

Analyze:  
Recursion方法 + DP  
For each node:  
 If rob, then no rob its children, max = curVal + no_rob left + no_rob right;  
 If no rob, then max = the max of left + the max of right

```java
class Solution {
    public int rob(TreeNode root) {
        int[] res = robSubtree(root);
        //0: no rob;  1: rob
        return Math.max(res[0], res[1]);
    }

    //for each node, two cases: rob and no rob
    public int[] robSubtree(TreeNode root) {
        if(root == null) return 0;

        int[] left = robSubtree(root.left);
        int[] right = robSubtree(root.right);

        int[] res = new int[2];
        res[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
        res[1] = root.val + left[0] + right[0];

        return res;
    }
}
```

```java
class Solution {
    public int rob(TreeNode root) {
        if(root == null) return 0;
        return Math.max(robInclude(root), robExclude(root));
    }
    
    public int robInclude(TreeNode node) {
        if(node == null) return 0;
        return robExclude(node.left) + robExclude(node.right) + node.val;
    }
    
    public int robExclude(TreeNode node) {
        if(node == null) return 0;
        return rob(node.left) + rob(node.right);
    }
}
```
