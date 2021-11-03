---
title: Dijkstra & Bellman-Ford 最短路径算法
date: 2019-02-09 18:12:19
tags: Dijkstra greedy
---
是贪心算法。
最短路径：从图中的某顶点出发到达另一点所经过的edge的weight sum最小的一条路径。

### Dijkstra

Given a connected direct graph G = (V, E), with positive weight on each edge (u, v), denoted as w(u, v). If no edge, w(u, v) = + Infinity.

1.Given a vertex v_0, find the shortest paths from v_0 to all other vertics.  
For each vertex v, d(v) = the current best shortest distance from v_0 to v.  
At the end, d(v) is the best shortest path distance from v_0 to v. Use a set S which initially is {v_0}, it will be enlarged during each iteration until becomes V at the end.

把图中顶点集合V分成两组：  
第一组：已求出最短路径的点集合(用S表示，初始时S中只有一个源点，以后每求得一条最短路径, 就将其加入到S中，直到全部点都加入到S中，算法就结束了)  
第二组：其余未确定最短路径的点集合(用U表示)，按最短路径长度的递增次序依次把U中的点加入S中。

步骤：

```html
1) Initialize
  S = {V_0}
  d(v_0) = 0
  for each vertex v except v_0:
    d(v) = w(v_0, v);
2) 
While(S != V) {
  从U中选取一个距v_0最近的点u，加入S中。
  let u be the vertex in U=V-S such that d(u) is the minimum in all d(x)s.
  S = S + {u}
  以u为新的中间点，更新U中各点x距v_0的距离d(x)
  for each vertex x in U:
    d(x) = min(d(x), d(u) + w(u, x));即v_0 - u - x的path距离
}
```

第2步可以通过priority/minheap来实现，poll()的就是U中有最小距离的点
O(|V|^2)?

这个方法的例题见Example 2.

### Bellman-Ford

**Dijkstra doesn’t work for Graphs with negative weight edges, while Bellman-Ford works for such graphs.**
Bellman-Ford is also simpler than Dijkstra, and suites well for distributed systems.  
But time complexity of Bellman-Ford is O(VE), which is more than Dijkstra.
还没完全弄懂

Steps:  
Let n be the number of vertices, n = |V|.

1)Initialize distances from source to all vertices as infinite, and the distance to source itself is 0.

```java
int INF = 0x3f3f3f3f;
int[] dist = new int[n];
Arrays.fill(dist, INF);
dist[src] = 0;
```

补充：  
0x3f3f3f3f的十进制是1061109567，也就是10^9级别的，和0x7fffffff一个数量级。一般场合下的数据都是小于10^9的，所以可以把这个数作为无穷大使用，而不致出现数据大于无穷大的情形。

2)Calculate shortest distances  
(n - 1) iterations, for each iteration, do:

```java
for(edge(u, v) : edges){
    if(dist[u] + w(u, v) < dist[v]){
        dist[v] = dist[u] + w(u, v);
    }
}
```

3)If there is a negative weight cycle in graph, do:

```java
for(edge(u, v) : edges){
    if(dist[u] + w(u, v) < dist[v]){
        graph has negative weight cycles
    }
}
```

This step iterates through all edges one more time to guarantee the shortest distances if graph doesn't contain negative weight cycle.

</br>

### Example

#### 1.Cheapest Flights With K Stops

There are n cities[0, n-1] connected by m flights. Each fight starts from city u and arrives at v with a price w. The format of each flight will be (src, dst, price).There will not be any duplicated flights or self cycles.

Now given all the cities and flights, together with starting city src and the destination dst, your task is to find the cheapest price from src to dst with up to k stops. k is in the range of [0, n - 1]. If there is no such route, output -1.
</br>

方法1, Minheap:  
从source city开始，下一步有很多flights。假设下一站为(B,C,D), 从中选取accumulative cost最小的flight，需要一个函数或者一个数据结构来选出这堆里累计cost最小的city，用minheap。minheap中需要存放的信息包括，下一个飞机的id，累计cost，和这条path上有多少站。

假设最小累计cost为B，把B的下一步所有能到达的city(X,Y,Z)都放入minheap中，X,Y,Z和原来heap中剩下的C,D一起比较cost，从中选出最小的累计cost。简单说就是，每一步都挑选最小累计cost的city。

```java
class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int K) {
        //start : <end, price>
        Map<Integer, Map<Integer, Integer>> map = new HashMap<>();
        for(int[] path : flights){
            map.putIfAbsent(path[0], new HashMap<>());
            map.get(path[0]).put(path[1], path[2]);
        }
        
        //int[]格式：city, 到达该city的accumulative cost, cnt(path上的city stops个数)
        //minheap on cost
        Queue<int[]> queue = new PriorityQueue<>((a,b) -> (a[1] - b[1]));
        queue.offer(new int[]{src, 0, 0});
        
        while(!queue.isEmpty()) {
            int[] node = queue.poll();
            int city = node[0], cost = node[1], cnt = node[2];
            
            if(city == dst){
                return cost;
            }
            
            //最多k个stops，cnt初始为0，所以最后到达的city的编号最大为k
            //所以是cnt <= K
            if(cnt <= K){
                //获取从当前city A到下一个city Bs的map<cityB, cost>
                if(!map.containsKey(city)) continue;
                Map<Integer, Integer> adj = map.get(city);
                for(int nextCity : adj.keySet()){
                    queue.offer(new int[]{nextCity, cost + adj.get(nextCity), cnt + 1});
                }
            }
        }
        
        return -1;
    }
}
```

</br>

方法2，Bellman-Ford算法：  
K stops, so (k + 1) iterations
这个代码没看懂
```java
class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int K) {
        int INF = 0x3f3f3f3f;
    
        int[] dist = new int[n];//the shortest cost between city i and src
        Arrays.fill(dist, INF);
        dist[src] = 0;
       
        int res = dist[dst];
        for(int i = 0; i <= K; i++){
            int[] temp = new int[n];
            Arrays.fill(temp, INF);
            for(int j = 0; j < flights.length; j++){
                int start = flights[j][0], end = flights[j][1];
                int price = flights[j][2];
                
                temp[end] = Math.min(temp[end], dist[start] + price);
            }
            dist = temp;
            res = Math.min(res, dist[dst]);
        }
        
        return res == INF ? -1 : res;
    }
}
```

#### 2.Network Delay Time

题目：  
N个network nodes，labelled 1 to N.  
Given times array, times[i] = (u, v, w) where from u to v take w time. Send a signal from a certain node K. How long will it take for all nodes to receive the signal at the earliest? If impossible, return -1.

思路：  
如果知道了K到所有点的最短路径，那么这些最短路径中的max值就是earliest time. 既然是求最短路径，就联想到了Dijkstra方法。 To speed things up, at each visited node, we'll consider signals exiting the node that are faster first. 所以可以用minheap，就可以选择下一步时选择最近的点了。所以pq里需要存放的信息有server的id，和K到该server的dist/time，因此`PriorityQueue<int[]>`。为了确保能到所有的点，所以可以用一个set来存放从K出发到过的点。
如果最后set.size() == N，就表示所有的server都到达过了，此时就可以返回最后更新的time值了。

每到一个点，需要知道下面可以通往哪些点，以及两点之间的edge，所以用一个`Map<Integer, List<Integer>>`来构建graph。

这道题非常适合加深理解该算法。把上面的步骤复制过来了，按照这题对步骤进行了具体描述。

下面这段其实可以不用看，想清楚上面的思路，就可以直接做题了。

```html
1) 初始化
做了一个Map<Integer, List<int[]>> map
key: start node, value: end node and weight
  v_0 = K, S is a HashSet<Integer>
  d(K) = 0
  for each vertex v except K:
    d(v) = w(K, v);

2) PriorityQueue<int[]> pq,
   always pick the next smallest cost
int[]: node Id, updated d(K, nodeId)

从pq中弹出的元素是此时accumulated weight最小的点，放入set中
假设弹出的点名为u，以u为新的中间点，更新它的临近各点(没有visited，即不在set中)x距v_0的距离，然后把这些edge放入pq中。
```

```java
class Solution {
    public int networkDelayTime(int[][] times, int N, int K) {
        //startNode, <endNode, cost>
        Map<Integer, List<int[]>> map = new HashMap<>();
        for(int[] path : times) {
            map.putIfAbsent(path[0], new ArrayList<>());
            map.get(path[0]).add(new int[]{path[1], path[2]});
        }
       
        Set<Integer> set = new HashSet<>();
        //int[]: node v, dist(v_0, v)
        Queue<int[]> pq = new PriorityQueue<>((a, b) -> (a[1] - b[1]));
        pq.offer(new int[]{K, 0});
        
        int time = 0;
        while(!pq.isEmpty()) {
            int[] node = pq.poll();
            /*只有从pq中弹出，表示可以到达这个点，才标记为visited
            而不是像以前那样做bfs在往pq里加入新元素时就中把新元素标记了
            
            但由于有可能被弹出的点之前已经被别的路径先抵达了，
            因此如果从现在的路径后抵达它，所费的总time肯定会要大
            所以还是要加一个if来检查是否已经被visited过
            eg[[1,2,1],[2,3,2],[1,3,4]],从1出发
            */
            if(set.contains(node[0])) continue;
            set.add(node[0]);
            time = node[1];
            
            if(!map.containsKey(node[0])) continue;
            for(int[] next : map.get(node[0])) {
                if(set.contains(next[0])) continue;
                pq.offer(new int[] {next[0], next[1] + node[1]});
            }
        }
       
        return set.size() == N ? time : -1;
    }
}
```

Time:  
O(ElogE + N), E是length of times array, O(ElogE) is the minheap implementation, N is the number of network servers. We traverse the entire graph and take O(N + E) time.

法2)Bellman-Ford algorithm  
如果用一个dist[]数组存放K到每个点的最短距离。那么min time就是所有dist中的max值.
如果是N个点，那么就要使用N-1个iteration来得到最后的dist数组

Time: O(N*E)
我觉得这个代码好像是错的，只是侥幸通过所有的test，应为每次iteration貌似应该只使用还没有被更新过的点进行松弛操作？
```java
class Solution {
    public int networkDelayTime(int[][] times, int N, int K) {
        int[] dist = new int[N];
        
        int INF = 0x3f3f3f3f; //4个3f
        Arrays.fill(dist, INF);
        dist[K - 1] = 0;
        
        //n - 1 iterations
        for(int i = 1; i < N; i++) {
            for(int[] edge : times) {
                int u = edge[0] - 1, v = edge[1] - 1, w = edge[2];
                dist[v] = Math.min(dist[u] + w, dist[v]);
            }
        }
        System.out.println(Arrays.toString(dist) );
        int res = 0;
        for(int num : dist) {
            res = Math.max(num, res);
        }
        
        return res == INF ? -1 : res;
    }
}
```