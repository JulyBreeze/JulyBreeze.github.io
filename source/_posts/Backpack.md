---
title: Backpack
date: 2019-02-08 13:13:57
tags: DP
---
### Description

* 0-1背包问题:  
N个物品，背包大小为V，每种物品仅有一件，要么取要么不取  
* 完全背包问题:  
N个物品，背包大小为V，每种物品有无限件，可以取多次  
* 多重背包问题:  
N个物品，背包大小为V，每种物品做多有M_i件，可以取多次  

按照要求的结果，大概分为3类:  
1）背包里能装进的物品的总价值最大是多少？  
2）装满背包有几种方法？  
3）背包问题的变种

</br>

注意！**背包问题不在乎取得物品的顺序！**

比如climbing stairs这道题，可以跨1,2或3步，问到第n级台阶有多少种方法。这类题需要考虑步数的顺序，因此不能用backpack方法。  
比如n = 3时，正确答案是4，因为{1,2}和{2,1}是两种不同的到达方法。

climbing stairs的代码，和backpack代码的区别是：两个for循环的正好反过来了。  
对于backpack，是从商品的角度来看的。对于每个商品，选还是不选；  
对于climbing这道题，是从台阶角度来看的。对于到达某个台阶，有x种步法可以走到这个台阶。

```java
public int climbStairs2(int n) {
    int[] steps = {1, 2, 3};
    int[] dp = new int[n + 1];

    dp[0] = 1;
    for(int i = 1; i <= n; i++){
        for(int step = 1; step <= 3; step++){
            if(i >= step){
                dp[i] += dp[i - step];
            }
        }
    }
    return dp[n];
}
```

### 一.求总价值的最大值

#### 0-1背包的基础问题

题目：  
Given n items with size S[i], m is the size of a backpack. Return the max size you can fill this backpack.

思路:  
对于每个item，有两种可能性，取或者不取。 
dp[i][j] is the max size when selecting first i items and the total size of these seleted items is <= j

```java
public class Solution {
    public int backPack(int m, int[] A) {
        int[][] dp = new int[A.length + 1][m + 1];

        for(int i = 1; i <= A.length; i++){
            for(int j = 1; j <= m; j++){
                if(j >= A[i - 1]){
                    dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j - A[i-1]] + A[i-1]);
                }else{
                    dp[i][j] = dp[i-1][j];
                }
            }
        }
        return dp[A.length][m];
    }
}
```

换种写法，道理完全一样：

```java
public class Solution {
    public int backPack(int m, int[] A) {
        int[][] dp = new int[A.length + 1][m + 1];

        for(int i = 1; i <= A.length; i++){
            for(int j = 1; j <= m; j++){
                //不取
                dp[i][j] = dp[i - 1][j];
                if(j >= A[i - 1]){
                    //取
                    dp[i][j] = Math.max(dp[i][j], dp[i - 1][j - A[i - 1]] + A[i - 1]);
                }
            }
        }
        return dp[A.length][m];
    }
}
```

空间优化：逆序枚举  
为什么要逆序枚举？  
因为从转移方程可以看出，后面的dp依赖于前面的dp。

 
```java
public int backPack(int m, int[] S) {
    int[] dp = new int[m + 1];

    for(int i = 0; i < S.length; i++) {
        for(int j = m; j >= S[i]; j--) {
            dp[j] = Math.max(dp[j], dp[j - S[i]] + S[i]);
        }
    }
    return dp[m];
}
```

#### 1. 0-1背包问题

题目：
Given n items with size S-i and value V-i, and a backpack with size m. What's the maximum value can you put into the backpack?

Eg: Given 4 items with size [2, 3, 5, 7] and value [1, 5, 2, 4], and a backpack with size 10. The maximum value is 9.

分析:  

```html
dp[i][j]: 从前i件物品(index range[0, i-1])中选择若干件,且这些若干件的size <= j 时的最大value

每件物品要么放，要么不放，所以dp[i][j]有两种情况：
    不把第i件放入包里
    把第i件放入包里(没放前时的物品的总size = 放了后的总size_j - A[i-1], 所以需要j >= A[i-1])

dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j - A[i-1]] + V[i-1])

dp数组的初始化，dp[0][j] = 0, dp[i][0] = 0，因为数组默认就是0，故可以省略
```

```java
public int backPack(int m, int[] S, int[] V) {
    int[][] dp = new int[S.length + 1][m + 1];

    for(int i = 1; i <= S.length; i++){
        for(int j = 1; j <= m; j++){
            if(S[i - 1] > j){
                dp[i][j] = dp[i - 1][j];
            }else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - S[i - 1]] + V[i - 1]);
            }
        }
    }
    return dp[S.length][m];
}
```

优化：
二维数组的填充是逐行从做到右的，如下图所示：
观察发现，每个格子只与前面一行的两个格子的值有关，也就是前i个物品的最优值只和前i-1个物品的最优值有关。因此二维的转移方程公式可以改成：

```html
// 1表示当前，0表示前面
dp[1][j] = Math.max(dp[0][j], dp[0][j - A[i-1]] + V[i-1])
```

因此可以继续把二维的填充变成一维，即：

```html
dp[j] = Math.max(dp[j], dp[j - A[i-1]] + V[i-1]);

等号右边的dp[j]就是前一步得到的值，在这一步更新变成了等号左边的值。
这个左边的值作为下一步等号右边的值
```

确定填充方向：应该从后往前填充，这样在填充过程中前面不会被覆盖掉。

```html
dp[j] = Math.max(dp[j], dp[j - A[i - 1]] + V[i - 1])

m为8，size = {2，3，4，5}，value[4] = {3，4，5，6}
when i = 2, dp = { 0, 0, 3, 4, 4, 7, 7, 7, 7}
when i = 3, dp =  {0, 0, 3, 4, 5, 7, 8, 9, 9}
```

{% asset_img demo.jpg 0-1 %}
空间可以优化成O(M)：

```java
public class Solution {
    public int backPackII(int m, int[] A, int[] V) {
        int n = A.length;
        int[] dp = new int[m + 1];
        
        for(int i = 0; i < n; i++){
            for(int j = m; j >= A[i]; j--){
                dp[j] = Math.max(dp[j], dp[j - A[i]] + V[i]);
            }
        }
        
        return dp[m];
    }
}
```

2)二进制优化  

补充知识：快速幂(快速求x^m)

```html
m = 8
S = {2,   3,   4,   5}
V = {30, 50, 100, 200}

那么第0个物品最多能取4个，4的二进制是100，拆分为1, 10, 100
即十进制中的1,2,4，这3个数可以任意选出几个组合成1,2,3,4

假设第i个物品最多可以取x个，设2^n <= x
拆分为1,2,4,8...2^n，这些数种的任意几个可以组合成1,2...x

然后把1,2,4,8...2^n看成1个物品，2个商品捆绑成1个物品，4个捆绑成1个物品...
相当于有n类新物品。
```

```java

```

#### 2.完全背包问题

```html
一个元素可以取无限次,体现在取第i个元素时,是dp[i][j - A[i - 1]]而不是dp[i-1][..]
dp[i][j] = max(dp[i][j - A[i - 1]] + V[i - 1], dp[i - 1][j])

初始条件：dp[0][j] = 0, dp[i][0] = 0
```

```java
public int backPack(int[] S, int[] V, int m) {
  int[][] dp = new int[S.length + 1][m + 1];

  for(int i = 1; i <= S.length; i++){
    for(int j = 1; j <= m; j++){
        if(j < S[i-1]) {
            dp[i][j] = dp[i - 1][j];
        }else{
            dp[i][j] = Math.max(dp[i-1][j], dp[i][j - S[i-1]] + V[i-1]);
        }
    }
  }
  return dp[S.length][m];
}
```

空间优化：正序枚举

```java
public int backPack(int[] S, int[] V, int m){
    int n = S.length;
    int[] dp = new int[m + 1];

    for(int i = 0; i < n; i++){
        for(int j = S[i]; j <= m; j++){
            dp[j] = Math.max(dp[j], dp[j - S[i]] + V[i]);
        }
    }
    return dp[m];
}
```


#### 3.多重背包问题

可以转化为0-1背包，即把每一类物品的数量展开，就成了0-1背包。  
这里用一维，就是逆向枚举。

```java
public int backPack(int n, int[] size, int[] value, int[] amount){
    int[] dp = new int[n + 1];
    int m = value.length; // m种物品

    //前两个for就是这种展开过程：对于每一类物品的每一件
    for(int i = 0; i < m; i++){
        for(int j = 1; j <= amount[i]; j++){
            for(int k = n; k >= size[i]; k--){
                dp[k] = Math.max(dp[k], dp[k - size[i]] + value[i]);
            }
        }
    }
    return dp[n];
}
````


### 二.求装满书包有几种方法

#### 1.每次只能取一件

```html
dp[i][j]表示从前i件中取若干件且总size为j时的方法个数。

初始化：
  dp[i][0] = 1，但是这个可以合并到转移方程中去
  即dp[0][0] = 1, j starts at 0.
```

```java
public int backPackV(int[] S, int m) {
  int[][] dp = new int[S.length + 1][m + 1];
  dp[0][0] = 1;

  for(int i = 1; i <= S.length; i++){
    for(int j = 0; j <= m; j++){
        //这里不能写成if(j < S[i-1]) .. else ..
        //因为即使j >= S[i-1],dp[i][j]里面需要加上dp[i-1][j]
        dp[i][j] = dp[i-1][j];
        if(j >= S[i-1]){
            dp[i][j] += dp[i-1][j - S[i-1]];
        }
    }
  }
  return dp[S.length][m];
}
```

优化：

```java
public int backPackV(int[] S, int m) {
    int[] dp = new int[m + 1];
    dp[0] = 1;

    for(int i = 0; i < S.length; i++){
        for(int j = m; j >= S[i]; j--){
            dp[j] += dp[j - S[i]];
        }
    }
    return dp[m];
}
```

为什么第二个for用倒叙的循环？
因为每个物品只能使用一次，用倒序循环不会影响之后的操作。

```
例：某item size = 5, package size = 10.
倒叙循环：
f[10] += f[10-5] = f[5];
f[9] += f[4];
f[8] += f[3];
f[7] += f[2];
f[6] += f[1];
f[5] += f[0];
在每次更新f[j]，都是基于这个物品还没有放进去的情况，只有倒序循环才能满足条件.

如果正序循环：
f[5] += f[0];
f[6] +=f[1];
f[7] +=f[2];
f[8] +=f[3];
f[9] +=f[4];
f[10] +=f[5]
计算f[10]时，f[5]在之前已经计算过了，并且是由f[0]得到的，
因此此时的f[10]表示的意思是size = 10的背包里装了两个size为5的物品，是不符合题意的。

倒叙循环的j 是 package_size -> S[i]
```

#### 2.可以取无限次

每个物品的size是S[i]，总共m大小的背包.

```html
初始化，当m = 0时，dp[i][0] = 1,即每个物体都不取。
```

1) 写法1

```java
public int backPackIV(int[] S, int m) {
    int[][] dp = new int[S.length + 1][m + 1];

    for(int i = 0; i <= S.length; i++){
        dp[i][0] = 1;
    }

    for(int i = 1; i <= S.length; i++){
        for(int j = 1; j <= m; j++){
            int cnt = 0;
            while(cnt * S[i-1] <= j){
                dp[i][j] += dp[i-1][j - S[i-1] * cnt];
                cnt++;
            }
        }
    }

    return dp[S.length][m];
}
```

2)写法2，和0-1背包相对应的写法

```java
public int backPack(int[] S, int m){
    int n = S.length;
    int[][] dp = new int[n + 1][m + 1];
    dp[0][0] = 1;

    for(int i = 1; i <= n; i++){
        for(int j = 0; j <= m; j++){
            dp[i][j] = dp[i - 1][j];
            if(j >= S[i - 1]){
                dp[i][j] += dp[i][j - S[i - 1]];//完全背包
                //dp[i][j] += dp[i - 1][j - S[i - 1]];//0-1背包
            }
        }
    }

    return dp[n][m];
}
```

3)优化空间：

```java
public int backPackIV(int[] A, int m) {
    int[] dp = new int[m + 1];
    dp[0] = 1;

    /*j表示总size，只有当总size >= 当前要装的这一件商品的size时，
    才能往里加（也就是要求size=j时有几种方法，假设当前要装的某item size=x
    那么就要知道dp[j-x]有几种装法，dp[j] += dp[j-x]
    或者反过来理解，已知dp[k]，那么更新dp[k+item_size]的方法个数，即加上dp[k]
    这里用正序，因为每个物品可以取多次
    */
    for(int i = 0; i < A.length; i++){
        for(int j = A[i]; j <= m; j++){
            dp[j] += dp[j - A[i]];
        }
    }
    return dp[m];
}
```



#### 3.多重背包

```java

```

### 三. 应用题

#### 题1

题目：超市里有多种大米可以选择。每种大米都是袋装的，必须整袋购买。给出每种大米的重量、价格、数量。给N元，求最多能买多少公斤的大米？

思路：多重背包问题

```java
public class Solution {
    public int backPackVII(int n, int[] prices, int[] weights, int[] amounts) {
        int m = prices.length;
        //从m件商品种选出若干件，花费总和<= n时的最大数量
        int[][] dp = new int[m + 1][n + 1];
    
        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                //k从0开始，表示可以不买。
                for(int k = 0; k <= amounts[i - 1]; k++){
                    if(j >= k * prices[i - 1]){
                        int temp = Math.max(dp[i-1][j], dp[i-1][j - k * prices[i - 1]] + k * weights[i - 1]);
                        dp[i][j] = Math.max(dp[i][j], temp);
                    }else{
                        dp[i][j] = Math.max(dp[i][j], dp[i-1][j]);
                    }
                }
            }
        }
    
        return dp[m][n];
    }
}
```

一维数组的方法：

```java
public class Solution {
    public int backPackVII(int n, int[] prices, int[] weights, int[] amounts) {
        int m = prices.length;
        int[] dp = new int[n + 1];

        for(int i = 0; i < m; i++){
            for(int j = 1; j <= amounts[i]; j++){
                for(int k = n; k >= prices[i]; k--){
                    dp[k] = Math.max(dp[k], dp[k - prices[i]] + weights[i]);
                }
            }
        }

        return dp[n];
    }
}
```



#### 题2

题目：coin change, 硬币可以取无数次，求问使得硬币组合的总价值为amount时的最少硬币数量？

思路：
完全背包问题的优化空间版，正向枚举
```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int n = coins.length;
        int[] dp = new int[amount + 1];
        
        Arrays.fill(dp, amount + 1);
        dp[0] = 0;
        
        for(int i = 0; i < n; i++) {
            for(int j = coins[i]; j <= amount; j++) {
                dp[j] = Math.min(dp[j], dp[j - coins[i]] + 1);
            }
        }
        
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

题目：coin change II，给一些价值和数量的硬币，求有多少种组合方法使得硬币组合的价值总和在[1, n]范围内？  
多重背包问题。  

我用了多重背包的模板写的这题，结果tle，不知道代码本身是不是对的。

```java
public class Solution {
    public int backPackVIII(int n, int[] value, int[] amount) {
        int m = value.length;
        int[] dp = new int[n + 1];
        dp[0] = 1;
        
        int res = 0;
        for(int i = 0; i < m; i++){
            //cnt[k]表示，对于某类物品，在combination总值为k时，用到了多少个此类物品
            int[] cnt = new int[n + 1];
            for(int j = value[i]; j <= n; j++){
                if(dp[j] != 1 && dp[j - value[i]] == 1 && cnt[j - value[i]] < amount[i]){
                    dp[j] = 1;
                    res++;
                    cnt[j] = cnt[j - value[i]] + 1;
                }
            }
        }
        
        return res;
    }
}
```

#### 题3

题目：要申请学校，每个大学的申请费和得到offer的成功率已知，大学数量是m个，总共有n万元。找到获得至少一份大学offer的最高可能性。

思路： 0-1背包
至少一个里面包括有1个，有2个等等。因此可以从反方向来做这道题，即求没有申请到大学的概率。
概率是相乘的。

```java
public double backPack(int n, int[] prices, double[] probability) {
    //表示当花费的钱为i时的不成功的概率
    double[] dp = new double[n + 1];
    //所以初始化为1.0
    for(int i = 0; i <= n; i++){
        dp[i] = 1.0;
    }

    int m = probability.length;
    for(int i = 0; i < m; i++){
        probability[i] = 1 - probability[i];
    }

    for(int i = 0; i < m; i++){
        for(int j = n; j >= prices[i]; j--){
            dp[j] = Math.min(dp[j], dp[j - prices[i]]  * probability[i]);
        }
    }
    return 1 - dp[n];
}
```


#### 题4 Minimum Change After Purchase

题目：有三种商品，价格分别是150，250，350元。三种商品的数量无限多。给N元，购买玩商品后剩下的给商人作为小费，最少要给商人多少小费。

```java
public int fee(int n){
    int[] prices = {150, 250, 350};
    //表示钱为i时，能买到的商品的总价值的最大值
    int[] dp = new int[n + 1];

    for(int i = 0; i < 3; i++){
        for(int j = prices[i]; j <= n; j++){
            dp[j] = Math.max(dp[j], dp[j - prices[i]] + prices[i]);
        }
    }

    return n - dp[n];
}
```

#### 题5 Card Game II

题目：  
You are playing a card game, there are n cards in total. Each card costs cost[i] and inflicts damage[i] damage to the opponent. You have a total of totalMoney dollars and need to inflict at least totalDamage damage to win. And Each card can only be used once. Determine if you can win the game.

思路：0-1背包问题  
要成功，在保证所用的card的总cost <= totalMoney的情况下，使得这些cards的damage总和 >= totalDamage。也就是要使得在所给money的情况下，求出能发挥的最大的damage，如果这个>totalDamage，那么就会赢。

```java 
public boolean cardGame(int[] cost, int[] damage, int totalMoney, int totalDamage) {
    int[] dp = new int[totalMoney + 1];

    int n = cost.length;
    for(int i = 0; i < n; i++){
        for(int j = totalMoney; j >= cost[i]; j--){
            dp[j] = Math.max(dp[j], dp[j - cost[i]] + damage[i]);
        }
    }

    return dp[totalMoney] >= totalDamage ? true : false;
}
```


#### 题6 Cutting a Rod

题目：
Given a rod of length n inches and an array of prices, for piece i, its size = i + 1, price = prices[i]. Determine the maximum value obtainable by cutting up the rod and selling the pieces.

思路：完全背包问题

```java
public class Solution {
    public int cutting(int[] prices, int n) {
        int m = prices.length;
        int[][] dp = new int[m + 1][n + 1];

        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                //for piece index at (i - 1), its size = i
                if(j < i){
                    dp[i][j] = dp[i - 1][j];
                }else{
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - i] + prices[i - 1]);
                }
            }
        }

        return dp[m][n];
    }
}
```

```java
public class Solution {
    public int cutting(int[] prices, int n) {
        int[] dp = new int[n + 1];

        for(int i = 1; i <= prices.length; i++){
            for(int j = i; j <= n; j++){
                dp[j] = Math.max(dp[j], dp[j - i] + prices[i - 1]);
            }
        }
        return dp[n];
    }
}
```

#### 题7. Minimum Adjustment Cost

题目：  
Given an integer array, adjust each integers so that the difference of every adjacent integers are not greater than a given number target.

If the array before adjustment is A, the array after adjustment is B, you should minimize |sum_A - sum_B| and return this min diff.

思路：  

```java
public class Solution {
    public int MinAdjustmentCost(List<Integer> list, int target) {
        int n = list.size();
        int[][] dp = new int[n + 1][101];
        
        //initialization
        for(int[] row : dp){
            Arrays.fill(row, Integer.MAX_VALUE);
        }
        
        for(int i = 0; i <= 100; i++){
            dp[0][i] = 0;
        }
        
        //transfer
        for(int i = 1; i <= n; i++){
            for(int j = 1; j <= 100; j++){
                if(dp[i - 1][j] != Integer.MAX_VALUE){
                    for(int k = 1; k <= 100; k++){
                        if(Math.abs(j - k) <= target){
                            int prev = dp[i - 1][j] + Math.abs(list.get(i - 1) - k);
                            dp[i][k] = Math.min(dp[i][k], prev);
                        }
                    }
                }
            }
        }
        
        int res = Integer.MAX_VALUE;
        for(int i = 0; i <= 100; i++){
            res = Math.min(res, dp[n][i]);
        }
        
        return res;
    }
}
```

空间的优化：
滚动数组，每次滚回来的时候要重新初始化？
能不能变成一维数组？

```java

```

#### 题8. Target Sum

题目：
You are given a list of non-negative integers, a1, a2, ..., an, and a target S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.  
Find out how many ways to assign symbols to make sum of integers equal to target S.

思路：  
对于每个元素，decide add or minus it，所以是0-1背包问题。

```java
dp[i][j] is the number of ways for the first i elements to reach a sum j
dp[i][j] = new int[n + 1][2 * sum + 1] 
//because the range of possible target is [-sum, sum], so new a 2*sum length
dp[i][j] = 
```

```java
class Solution {
    public int findTargetSumWays(int[] nums, int K) {
        int n = nums.length;
        int sum = 0;
        for(int num : nums) {
            sum += num;
        }
        if(sum < Math.abs(K)) return 0;
        int s = 2 * sum;
        
        int[][] dp = new int[n + 1][s + 1];
        if(nums[0] == 0){
            dp[1][sum] = 2;
        }else{
            dp[1][sum - nums[0]] = 1;
            dp[1][sum + nums[0]] = 1;
        }
        
        for(int i = 1; i <= n; i++){
            for(int j = 0; j <= s; j++){
                if(j - nums[i - 1] >= 0){
                    dp[i][j] += dp[i - 1][j - nums[i - 1]];
                }
                if(j + nums[i - 1] <= s){
                    dp[i][j] += dp[i - 1][j + nums[i - 1]];
                }
            }
        }
        
        return dp[n][sum + K];
    }
}
```