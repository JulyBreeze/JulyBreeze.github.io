---
title: System Design 例题集合
date: 2019-07-06 15:46:43
tags: system design
---


### I.Design Twitter

```java

```

</br>

### II.User System - Database & Cache

#### 1.Friendship Service

题目：  
Support follow & unfollow, getFollowers, getFollowings.

Eg:  

```java
follow(1, 3)
getFollowers(1) // return [3]
getFollowings(3) // return [1]
follow(2, 3)
getFollowings(3) // return [1,2]
unfollow(1, 3)
getFollowings(3) // return [2]
```

</br>

思路：  
两个map，分别维护关注的对象和粉丝。

```java
public class FriendshipService {
	//用treeset，这样返回粉丝或者关注者列表就是按id排序的。
    Map<Integer, TreeSet<Integer>> followers, followings;
    public FriendshipService() {
        followers = new HashMap<>();//关注者
        followings = new HashMap<>();//粉丝
    }

    public List<Integer> getFollowers(int user_id) {
        if(!followers.containsKey(user_id)){
            return new ArrayList<Integer>();
        }
        //convert set to array
        return new ArrayList<Integer>(followers.get(user_id));
    }

   
    public List<Integer> getFollowings(int user_id) {
        if(!followings.containsKey(user_id)){
            return new ArrayList<Integer>();
        }

        return new ArrayList<Integer>(followings.get(user_id));
    }

    //A关注了B
    public void follow(int A, int B) {
        followers.putIfAbsent(A, new TreeSet<>());
        followers.get(A).add(B);

        followings.putIfAbsent(B, new TreeSet<>());
        followings.get(B).add(A);
    }

    public void unfollow(int A, int B) {
        if(followers.containsKey(A)){
            followers.get(A).remove(B);
        }

        if(followings.containsKey(B)){
            followings.get(B).remove(A);
        }
    }
}
```

</br>

#### 2.Friendship Service II

题目：

```java

```

</br>

#### 3.Memcache

题目：  
Implement a memcache which support the following features:

```html
get(curTime, key)
set(curTime, key, value, ttl)
delete(curTime, key)
incr(curTime, key, delta)
decr(curTime, key, delta)
```

1) get(). Get the key's value, return 2147483647 if key does not exist.
2) set(). Set the key-value pair in memcache with a time to live (ttl). The key will be valid from curtTime to curtTime + ttl - 1, and it will expire after ttl seconds. If ttl = 0, the key lives forever until out of memory.  
3) delete(). Delete the key.  
4) incr(). Increase the key's value by delta, return the new value. Return 2147483647 if key does not exist.  
5) decr(). Decrease the key's value by delta, return the new value. Return 2147483647 if key does not exist.

It's guaranteed that the input is given with increasing curtTime.

```java
class Resource {
    public int val;
    public int exp;
    public Resource(int value, int expired){
        this.val = value;
        this.exp = expired;
    }
}

public class Memcache {
    Map<Integer, Resource> map;
    public Memcache() {
        map = new HashMap<>();
    }

    public int get(int curtTime, int key) {
        if(!map.containsKey(key)){
            return Integer.MAX_VALUE;
        }
        
        Resource res = map.get(key);
        if(res.exp >= curtTime || res.exp == -1){
            return res.val;
        }
        
        map.remove(key);
        return Integer.MAX_VALUE;
    }

    public void set(int curtTime, int key, int value, int ttl) {
        int expired;
        if(ttl == 0) expired = -1;
        else expired = curtTime + ttl - 1;
        
        map.put(key, new Resource(value, expired));
    }

    public void delete(int curtTime, int key) {
        if(!map.containsKey(key)) return;
        map.remove(key);
    }

    public int incr(int curtTime, int key, int delta) {
        //用get或者map.containsKey()都可以
        //我觉得调用本class的函数来判断更好些，因为顺便把过期的内容清除了
        if(get(curtTime, key) == Integer.MAX_VALUE){
            return Integer.MAX_VALUE;
        }
        
        map.get(key).val += delta;
        return map.get(key).val;
    }

    public int decr(int curtTime, int key, int delta) {
        if(!map.containsKey(key)) return Integer.MAX_VALUE;
        
        map.get(key).val -= delta;
        return map.get(key).val;
    }
}
```

</br>

#### 4.Mini Cassandra

题目：  
Cassandra is a NoSQL database (a.k.a key-value storage). One individual data entry in cassandra constructed by 3 parts:  
1)row_key. (a.k.a hash_key, partition key or sharding_key)  
2)column_key.  
3)value  

row_key is used to hash and can not support range query. Let's simplify this to a string.  
column_key is sorted and support range query. Let's simplify this to integer.  
value is a string. You can serialize any data into a string and store it in value.  

Implement the following methods:

```html
1)insert(row_key, column_key, value)
2)//return a list of entries
query(row_key, column_start, column_end)
```

```java
/**
 * Definition of Column:
 * public class Column {
 *     public int key;
 *     public String value;
 *     public Column(int key, String value) {
 *         this.key = key;
 *         this.value = value;
 *    }
 * }
 */
public class MiniCassandra {
    private Map<String, Map<Integer, String>> map;
    public MiniCassandra() { 
        map = new HashMap<>();
    }

    public void insert(String row_key, int column_key, String column_value) {
        if(!map.containsKey(row_key)) {
            map.put(row_key, new HashMap<Integer, String>());
        }
        map.get(row_key).put(column_key, column_value);
    }

    public List<Column> query(String row_key, int column_start, int column_end) {
        List<Column> res = new ArrayList<>();
        if(!map.containsKey(row_key)){
            return res;
        }

        Map<Integer, String> temp = map.get(row_key);
        for(int i = column_start; i <= column_end; i++){
            if(temp.containsKey(i)){
                res.add(new Column(i, temp.get(i)));
            }
        }

        return res;
    }
}
```

</br>

### III. Database Sharding & Consistent Hashing

#### 1.Consistent Hashing I & II

##### I

题目：  

```java

```

##### II

题目：  
In Consistent Hashing I, we introduced a relatively simple consistency hashing algorithm. This simple version has two defects：  
1)After adding a machine, the data comes from one of the machines. The read load of this machine is too large, which will affect the normal service.  
2)When adding to 3 machines, the load of each server is not balanced, it is 1:1:2.

In order to solve this problem, the concept of micro-shards was introduced, and a better algorithm is like this：

From 0~359 to a range of 0 ~ n-1, the interval is connected end to end and connected into a circle.  
When joining a new machine, randomly choose to sprinkle k points in the circle, representing the k micro-shards of the machine.
Each data also corresponds to a point on the circumference, which is calculated by a hash function.
Which machine belongs to which data is to be managed is determined by the machine to which the first micro-shard point that is clockwise touched on the circle is corresponding to the point on the circumference of the data.
n and k are typically 2^64 and 1000 in a real NoSQL database.

Implement these methods of introducing consistent hashing of micro-shard.

create(int n, int k)
addMachine(int machine_id) // add a new machine, return a list of shard ids.
getMachineIdByHashCode(int hashcode) // return machine id.

```java

```

</br>

#### 2.Load Balancer

介绍：  
把任务分散到不同的服务器上。  
A load balancer is a device that acts as a reverse proxy and distributes network or application traffic across a number of servers. Load balancers are used to increase capacity (concurrent users) and reliability of applications.  
</br>

题目：  
Implement a load balancer for web servers. It provide the following functionality:  
1）Add a new server to the cluster => add(server_id).  
2）Remove a bad server from the cluster => remove(server_id).  
3）Pick a server in the cluster randomly with equal probability => pick().  

At beginning, the cluster is empty. When pick() is called, you need to randomly return a server_id in the cluster.
</br>

三个操作都是O(1)

```java
public class LoadBalancer {
    List<Integer> servers;
    //the serverId and its positon in the list
    Map<Integer, Integer> map;
    Random rand;
    
    public LoadBalancer() {
        servers = new ArrayList<>();
        map = new HashMap<>();
        rand = new Random();
    }

    public void add(int server_id) {
        map.put(server_id, servers.size());
        servers.add(server_id);
    }

    public void remove(int server_id) {
        int pos = map.get(server_id);
        
        if(pos != servers.size() - 1){
            int lastServerId = servers.get(servers.size() - 1);
            servers.set(pos, lastServerId);
            map.put(lastServerId, pos);
        }
        
        map.remove(server_id);
        servers.remove(servers.size() - 1);
    }

    public int pick() {
        return servers.get(rand.nextInt(servers.size()));
    }
}
```

</br>

#### 3.Hit Counter

题目：  
Design a hit counter which counts the number of hits received in the past 5 minutes.  
Each function accepts a timestamp parameter (in seconds granularity) and you may assume that calls are being made to the system in chronological order (ie, the timestamp is monotonically increasing). You may assume that the earliest timestamp starts at 1.  
It is possible that several hits arrive roughly at the same time.

思路：  
One bucket for one second, 考虑到可能同时间会出现多个hit。  

有一种做法是，把新的timestamp放入queue的后端。当需要get hits时，就不断poll直到timestamp和queue首端的timestamp相差在300s。此时queue的size就是hit数。  
但是这个方法不好之处在于，如果同一时刻有很多hit，each element in queue is a single hit，那么会占用很多内存；另外如果要求返回很长时间内的hit数，那么queue的长度会很长。如果要求返回很短时间内的hit数，那么每隔很短的时间就要重新poll一次queue。所以这个方法不是很scalable。

下面的方法是复用了数组元素：

```java
class HitCounter {
    int[] time;
    int[] hits;
    
    public HitCounter() {
        time = new int[300];
        hits = new int[300];
    }
    
    public void hit(int timestamp) {
        int i = timestamp % 300;
        if(time[i] != timestamp){
            time[i] = timestamp;
            hits[i] = 1;
        }else{
            hits[i]++;
        }
    }
    
    public int getHits(int timestamp) {
        int count = 0;
        for(int i = 0; i < 300; i++){
            if(timestamp - time[i] < 300){
                count += hits[i];
            }
        }
        
        return count;
    }
}
```

</br>

#### 4.Tiny URL I && II

##### I.encode & decode

题目：  
Given a long url, make it shorter. Implement two methods:  
* longToShort(url)   
Convert a long url to a short url which starts with http://tiny.url/.  
* shortToLong(url)   
Convert a short url to a long url.  

You can design any shorten algorithm. The short key's length should equal to 6 (without domain and slash). And the acceptable characters are [a-zA-Z0-9]. For example: abcD9E.  
No two long urls mapping to the same short url and no two short urls mapping to the same long url.

</br>

思路：  

法1)  
随机一个6位的shortURL，如果没有用过，就绑定到LongURL。
两个map，一个是短网址映射到长网址, 一个是长网址映射到短网址。随机生成的基础上用一个集合来记录是否使用过.

```java
public class TinyUrl {
    private Map<String, String> long2short, short2long;
    
    public TinyUrl() {
        long2short = new HashMap<>();
        short2long = new HashMap<>();
    }
    
    public String longToShort(String url) {
        if(long2short.containsKey(url)) return long2short.get(url);
        
        while(true){
            String shortURL = generate();
            
            if(!short2long.containsKey(shortURL)){
                short2long.put(shortURL, url);
                long2short.put(url, shortURL);
                return shortURL;
            }
        }
    }

    public String shortToLong(String url) {
        if(short2long.containsKey(url)) {
            return short2long.get(url);
        }

        return null;
    }
    
    private String generate() {
        //一共26+26+10 = 62
        String allows = "0123456789" + "ABCDEFGHIJKLMNOPQRSTUVWXYZ" + "abcdefghijklmnopqrstuvwxyz";
        
        Random rand = new Random();
        String shortURL = "http://tiny.url/";
        for(int i = 0; i < 6; i++){
            int index = rand.nextInt(62);
            shortURL += allows.charAt(index);
        }

        return shortURL;
    }
}
```

</br>

法2) 进制转换base62  
给每个url都分配一个id，id自增，是全局变量。然后把url的id转换成一个shortKey，和前缀合并后就是对应的shortURL了，所以需要两个map，存放id和url的映射关系。

要获得shortURL，就先得到url的id，将其转换字母形式shortKey，不满6位数就补0(补0的原因是把shortKey转换回id时，char c = '0'不会对原id的计算有影响)。
要获得原url，把shortKey转回id，和前面的计算公式反过来就行。

优点是效率高。
缺点是依赖于全局自增的id

```java
public class TinyUrl {
    private Map<String, Integer> url2id;
    private Map<Integer, String> id2url;
    int Global_Id;
    
    String prefix = "http://tiny.url/";
    
    public TinyUrl() {
        id2url = new HashMap<>();
        url2id = new HashMap<>();
        this.Global_Id = 0;
    }
    
    public String longToShort(String url) {
        if(url2id.containsKey(url)) {
            return prefix + idToShortKey(url2id.get(url));
        }
        
        Global_Id++;
        url2id.put(url, Global_Id);
        id2url.put(Global_Id, url);
        
        return prefix + idToShortKey(Global_Id);
    }

    public String shortToLong(String url) {
        String shortKey = getShortKey(url);
        int id = shortKeyToId(shortKey);
        return id2url.get(id);
    }
    
    /*   self-defined func  */
    //把id转换成一个shortURL
    private String idToShortKey(int id){
        String chars = "0123456789" + "abcdefghijklmnopqrstuvwxyz" + "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
        StringBuilder sb = new StringBuilder();
        
        while(id > 0){
            sb.append(chars.charAt(id % 62));
            id = id / 62;
        }
        
        while(sb.length() < 6){
            sb.append("0");
        }
        
        return sb.reverse().toString();
    }
    
    private int shortKeyToId(String key){
        int id = 0;
        for(int i = 0; i < key.length(); i++){
            char c = key.charAt(i);
            id = id * 62 + toBase(c);
        }

        return id;
    }
    
    private String getShortKey(String url) {
        return url.substring(prefix.length());
    }
    
    //convert a character to an int
    //注意当c是小写字母时，返回的值要 + 10;大写则 + 36.
    //否则当char为小写字母c - 'a'的值可能和char为某数字时返回的值一样。
    private int toBase(char c) {
        if(c >= '0' && c <= '9') return c - '0';
        else if(c >= 'a' && c <= 'z') return c - 'a' + 10;
        return c - 'A' + 36;
    }
}
```

##### II

题目：  

思路：  


```java

```

</br>

#### 5.GeoHash I && II

介绍：  

##### I.编码

题目：  
Geohash is a hash function that convert a location coordinate pair into a base32 string.Convert a (latitude, longitude) pair into a geohash string.

思路：  
1)把经纬度按照binary search进行二进制编码  
2)奇数位放纬度，偶数位放经度，把2串编码交叉组合生成新串  
3)把编码(每五位是一个code)转化为十进制(基底是2)，找到这个十进制数在base32中对应的char
4)根据base32编码得到最后的结果

```java
public class GeoHash {
    public String encode(double latitude, double longitude, int precision) {
        //0-9 + 小写字母，但是没有abil四个，共10 + 22 = 32个chars
        String _base32 = "0123456789bcdefghjkmnpqrstuvwxyz";
        
        String lat = narrow(latitude, -90, 90);
        String lon = narrow(longitude, -180, 180);
        
        StringBuilder sb = new StringBuilder();
        //奇数位放纬度latitude,偶数位放经度longitude，
        for(int i = 0; i < 30; i++){
            sb.append(lon.charAt(i));
            sb.append(lat.charAt(i));
        }
        
        StringBuilder res = new StringBuilder();
        //每5bits转换成一个base32中的char
        for(int i = 0; i < 60; i += 5){
            int index = convert(sb.substring(i, i + 5));
            res.append(_base32.charAt(index));
        }
        
        return res.substring(0, precision);
    }
    
    private int convert(String s){
        //parseInt()将字符串参数作为有符号的十进制整数进行解析
        //如果有两个参数,使用第二个参数指定的基数
        return Integer.parseInt(s, 2);
    }
    
    //在[left, right]中定位val的位置
    private String narrow(double val, double left, double right){
        StringBuilder sb = new StringBuilder();
        
        for(int i = 0; i < 30; i++){
            double mid = (left + right) / 2;
            if(val > mid){
                left = mid;
                sb.append("1");
            }else{
                right = mid;
                sb.append("0");
            }
        }
        
        return sb.toString();
    }
}
```

</br>

##### II.解码

题目：  
Convert a Geohash string to latitude and longitude.

```java
public class GeoHash {
    public double[] decode(String geohash) {
        String _base32 = "0123456789bcdefghjkmnpqrstuvwxyz";
        int[] mask = {16, 8, 4, 2, 1};
        double[] lon = {-180, 180};
        double[] lat = {-90, 90};
        
        boolean isEven = true;
        for(int i = 0; i < geohash.length(); i++){
            int val = _base32.indexOf(geohash.charAt(i));
            //将每个index转化为5位的binary
            //然后对二进制的每一bit来缩小lat，lon的范围
            for(int j = 0; j < 5; j++){
                if(isEven){
                    refine(lon, val, mask[j]);
                }else{
                    refine(lat, val, mask[j]);
                }
                
                isEven = !isEven;
            }
        }
        //上面重新定位了两个interval的左右端点后
        //这块还没有想明白
        double latitude = (lat[0] + lat[1]) / 2.0;
        double longitude = (lon[0] + lon[1]) / 2.0;
        return new double[]{latitude, longitude};
    }
    
    private void refine(double[] interval, int val, int mask){
        //val在5bit中的某个bit上为1
        if((val & mask) > 0){
            interval[0] = (interval[0] + interval[1]) / 2.0;
        }else{
            interval[1] = (interval[0] + interval[1]) / 2.0;
        }
    }
}
```

</br>

#### 6.Rate Limiter

介绍：  
Rate limiting is used to control the rate of traffic sent or received by a network interface controller and is used to prevent DoS attacks.
</br>

##### 1) Logger Rate Limiter

题目：  
Design a logger system that receive stream of messages along with its timestamps, each message should be printed, if and only if it is not printed in the last 10 seconds.

Given a message and a timestamp (in seconds granularity), return true if the message should be printed in the given timestamp, otherwise returns false.

It is possible that several messages arrive roughly at the same time.
</br>

思路: 和Hit Counter很相似.  
corner case: many different messages at the same timestamp.  
注意只有当前时间打印了，才往数据结构里放入data。

有一个简单的解法是用map存储message和对应的timestamp，但可能会问follow up，关于map size grow up，所以这个解法并不是很好。

方法1)

```java
//self defined class
class Log {
    int t;
    String message;
    public Log(int ts, String mes){
        this.t = ts;
        this.message = mes;
    }
}

class Logger {
    Queue<Log> queue;
    Set<String> set;
    public Logger() {
        queue = new LinkedList<>();
        set = new HashSet<>();
    }
    
    /*
    只有当前时间打印了，才往数据结构里放入该数据，所以10s内打印的message有且只有一个
    首先把10s之前的数据都弹出去了，这里还要使用set.remove()，
    因为10s内的打印的message有且只有一个，所以可以使用set.remove，
    而不需考虑如果出现相同messeage但不同timestamp的情况。
    */
    public boolean shouldPrintMessage(int timestamp, String message) {
        while(!queue.isEmpty() && timestamp - queue.peek().t >= 10){
            Log data = queue.poll();
            set.remove(data.message);
        }
        
        if(!set.contains(message)){
            queue.offer(new Log(timestamp, message));
            set.add(message);
            return true;
        }
        
        return false;
    }
}
```

</br>

方法2)   
类似Hit Counter。该方法limited memory usage and little concurrency issue

```java
class Logger {
    private int[] buckets;//存放time
    private Set[] sets; //存放当前time下的所有message
    public Logger() {
        buckets = new int[10];
        sets = new Set[10];
        for(int i = 0; i < sets.length; i++){
            sets[i] = new HashSet<String>();
        }
    }
    
    public boolean shouldPrintMessage(int timestamp, String message) {
        int id = timestamp % 10;
        //bucket[id]里面的时间和当前timestamp相差10，即在10s开外
        //所以原来sets[id]的数据可以舍弃
        if(timestamp != buckets[id]){
            sets[id].clear();
            buckets[id] = timestamp;
        }
        
        for(int i = 0; i < buckets.length; i++){
            if(timestamp - buckets[i] < 10){
                if(sets[i].contains(message)){
                    return false;
                }
            }else{//save more space
                sets[i].clear();
            }
        }
        
        //只有打印了的数据才放入
        sets[id].add(message);
        return true;
    }
}

```

</br>

##### 2) Rate Limiter

题目：  
Implement a rate limiter, provide one method: is_ratelimited(timestamp, event, rate, increment).

Parameters:  
a) timestamp:  
The current timestamp, which is an integer and in second unit.  
b) event:  
The string to distinct different event. for example, "login" or "signup".  
c) rate:  
The rate of the limit. Eg.1/s, 2/m, 10/h, 100/d. The format is [integer]/[s/m/h/d].  
d) increment:  
Whether we should increase the current event' count.

The method should return true if the event is limited.

这个题的description有点混乱。如果某个操作没有limited，且increment为true时，才把这个是event和time记录下来。
而且the occurrence of an event before current timestamp && in the duration >= limit cnt时，此次操作就无法执行。  
Eg.

```html
is_ratelimited(1, "login", "3/m", true)    false
is_ratelimited(11, "login", "3/m", true)   false
is_ratelimited(21, "login", "3/m", true)   false
is_ratelimited(30, "login", "3/m", true)   true
is_ratelimited(65, "login", "3/m", true)   false
is_ratelimited(300, "login", "3/m", true)  false
```

思路:  


```java
public class Solution {
    //event, list of time
    Map<String, List<Integer>> map = new HashMap<>();
    
    public boolean isRatelimited(int timestamp, String event, String rate, boolean increment) {
        if(!map.containsKey(event)){
            map.put(event, new ArrayList<>());
        }
        
        int[] temp = getCntAndSecond(rate);
        int limit = temp[0], duration = temp[1];
         
        List<Integer> list = map.get(event);
        int cnt = 0;
        for(int i = list.size() - 1; i >= 0; i--){
            if(timestamp - list.get(i) < duration){
                cnt++;
            }
        }
        
        boolean res = cnt >= limit;
        //只有当increment = true，且当前行为没有被limit时，
        //才更新该event的time list
        if(increment && !res) map.get(event).add(timestamp);
        
        return res;
    }

    public int[] getCntAndSecond(String rate){
        int index = rate.indexOf("/");
        
        String cntStr = rate.substring(0, index);
        String unit = rate.substring(index + 1);
        
        int cntInt = Integer.parseInt(cntStr);
        
        int seconds = 1;
        switch (unit) {
            case "m" :
                seconds = 60;
                break;
            case "h" :
                seconds = 60 * 60;
                break;
            case "d" :
                seconds = 60 * 60 * 24;
                break;
            default :
        }
        
        return new int[]{cntInt, seconds};
    }
}
```

