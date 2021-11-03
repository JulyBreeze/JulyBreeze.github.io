---
title: Rabin Karp
date: 2019-02-09 00:06:50
tags:
---

### Introduction
Rabin-Karp算法类似于hashing function的原理。Rabin-Karp的复杂度通常是O(n)。

为了避免逐个char对source和pattern进行比较，可以尝试一次性判断两者是否相等，因此需要一个好的hash function。通过hash function，可算出pattern的hash code，然后将它和source的子串的hash code进行比较。

唯一的问题在于需要找到一个hash function ，它需要能够对不同的字符串返回不同的hash code。一般magic number选31是因为根据经验，选择比较大的素数可以减少hash冲突。

以"hello world"为例，假设它的哈希值hash('hello world')=12345。如果hash('he')=1，就可以说pattern "he" 包含在"hello world"中。由此，可以每次从source中取出长度为m = pattern.length()的子串，将该子串进行哈希，并将其hash code与pattern的hash code进行比较。

注意相同string的hashcode一定相同，但是反过来不一定，即有哈希冲突的现象存在。

所以当找到两个相同的hashcode后，还需要对这两个长度为m的字符串进行额外的比对（当然，如果不相等也就不用比对了，其实大部分的时间省在这上面），这时比对的开销是O(m)。
最坏情况下，文本中所有长度为m的子串(一共n-m+1个)都和pattern匹配，所以算法复杂度为O((n-m+1)*m)。然而实际情况下，需要进一步比对的子串个数总是有限的（假设为c个），那么算法的期望匹配时间就变成O((n-m+1)+cm)=O(n+m)。

mod的作用是让所有的运算结果的范围都在一定范围以内，因为如果不加mod，hash值的大小可能会因为字符串很长变得很大，所以要使用一个mod将其映射到一定范围之内，mod后的结果对于加、乘都是不变的。
```
(a + b) % mod = a % mod + b % mod
(a * b) % mod = [(a % mod) * (b % mod)] % mod

对于减法，可能会变成负数，所以需要再加上一个mod
(a - b) % mod = [(a % mod) - (b % mod) + mod] % mod
对于除法，% mod之后可能无法整除
```


例题：
### 1.Find First Index of pattern in String
Implement function in O(n + m) time.  
return the first index of the pattern string in a source string. The length of the pattern string is m and the length of the source string is n. If pattern does not exist in source, just return -1.


```java
public class Solution {
   //hash function: (a*31^2 + b*31 + c) % BASE = ((a*31^2 % BASE + b*31) % BASE + c) % BASE
    public int BASE = 1000000;//要取得mod，希望这个值越大越好，以让31^(n-1)不越界
  
    public int strStr2(String source, String target) {
        if(source == null || target == null){
            return -1;
        }
        
        int tlen = target.length();
        if(tlen == 0){
            return 0;
        }
        
        //calculate 31^tlen，而不是31^(tlen-1)
        //因为要移除的是前一个window的start，此时该处的值*31^tlen加入到hashCode里了
        int power = 1;
        for(int i = 0; i < tlen; i++){
            power = (power * 31) % BASE;
        }
        
        int targetCode = 0;
        for(int i = 0; i < tlen; i++){
            targetCode = (targetCode * 31 + target.charAt(i)) % BASE;
        }
        
        int hashCode = 0;
        for(int i = 0; i < source.length(); i++){
            hashCode = (hashCode * 31 + source.charAt(i)) % BASE;
            
            if(i < tlen - 1){
                continue;
            }
            
           //当i到tlen时，只需要abc，此时却是abcd
           //相当于sliding window，i-len是前一个window的start index
            if(i >= tlen){
                hashCode = hashCode - (source.charAt(i - tlen) * power) % BASE;
                if(hashCode < 0){
                    hashCode += BASE;
                }
            }
            
            //当i = n-1或者i >= n时，到这里的substring都已满足长度为n
            if(hashCode == targetCode){
                if(source.substring(i - tlen + 1, i + 1).equals(target)){
                    return i - tlen + 1;
                }
            }
        }
        
        return -1;
    }
}
```

上面的hash叫滚动hash，因为对于每一个字符串，其hash值都能由它的上一个前缀递推过来。
```
hash(abc) = ((a * seed) + b) * seed + c
hash(abcd)= (((a * seed) + b) * seed + c) * seed + d
          = hash(abc) * seed + d
hash(cd) = hash(abcd) - hash(ab) * seed^2

hash(s[x,y]) = hash(s[1, y]) - hash(s[1, x-1])*seed^(len(s[x,y]))
```
由上可以观察得到，只要知道了某string的前缀的hashcode，那么该string的任意部分都可以在O(1)内计算出来

附上O(n^2)做法：
```java
public class Solution {
    public int strStr(String source, String target) {
        if(source == null || target == null){
            return -1;
        }
        if(target.length() == 0){
            return 0;
        }
        
        int slen = source.length();
        int tlen = target.length();
        if(slen < tlen){
            return -1;
        }
        
        for(int i = 0; i <= slen - tlen; i++){
            if(source.charAt(i) == target.charAt(0)){
                if(source.substring(i, i + tlen).equals(target)){
                    return i;
                }
            }
        }
        return -1;
    }
}
```

### 2.Shortest Palindrome
Given a string s, you are allowed to convert it to a palindrome by adding characters in front of it. Find and return the shortest palindrome you can find by performing this transformation.

思路:  
Find the longest palindrome starting from the index 0.   
Add the reverse of the remaining part to the front.

```
对于某个string，从左往右求hashcode已经知道怎么做.

nums array = {1 2 3 4}
从左往右求hash code:
0
0 * 10 + 1 = 1
1 * 10 + 2 = 12
12 * 10 + 3 = 123
123 * 10 + 4 = 1234

因为希望指针移动的过程中，同时计算reverse array的hash code:
0
0 + 1 * 1 = 1
1 + 2 * 10 = 21
21 + 3 * 100 = 321
321 + 4 * 1000 = 4321
所以hash2 = hash2 + digit * power
power = power * b;
```
O(n)
```java
class Solution {
    public String shortestPalindrome(String s) {
        if(s == null || s.length() <= 1){
            return s;
        }
        
        int MOD = 1000000, B = 31;
        int hash1 = 0, hash2 = 0;
        int pow = 1;
        int pos = 0;

        for(int i = 0; i < s.length(); i++){
            char c = s.charAt(i);
          
            hash1 = (c + hash1 * B) % MOD;
            hash2 = (c * pow + hash2) % MOD;
            
            if(hash1 == hash2){
                pos = i;
            }
            
            pow = pow * B % MOD;
        }
        
        return new StringBuilder().append(s.substring(pos + 1)).reverse().append(s).toString();
    }
}
```
当然也有可能hashcode相同，但string不同，但是这个概率很低。可以通过增大MOD、调整magic code来降低概率。

### 3. Repeated String Match
