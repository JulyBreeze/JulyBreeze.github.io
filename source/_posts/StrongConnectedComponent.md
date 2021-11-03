---
title: Strongly Connected Component
date: 2019-09-15 23:28:52
tags: data structure
---

### Strongly Connected Component

1.Definition  
Given a directed graph G, a strongly connected component is a maximal set of vertices that for each pair of vertices u and v, there are always a path from u to v and v to u.

总结为两点：

* Each vertex is reachable from every other vertex within a strongly connected component.
* The set is maximal.

2.Property  
1) Given two strongly connected components A and B, it is impossible to have edges from A to B and B to A at the same time.  
Because if so, A and B can be combined into a larger connected component.

2) If each strongly connected component is collapse into a single vertex, and construct an edge from A to B if there exists an edge from a vertex of A to a vertex of B. The new graph is a directed **acyclic** graph(DAG).

3) The SSC is **cyclic**.  从某点出发，总能回到出发点。

### 相关Graph知识

#### 基本概念

1) Articulation Point  
A vertex is called Articulation point if removing it and all the edges associated with it will increase the number of connected components in the graph.

2) Back Edge  
Edge connects a vertex to a vertex that is discovered before its parents.

The meaning of back edge:  
Presence of a back edge means there is an alternative path in case the parent of the vertex is rmoved.  
假设一个点u有一个child v，v及其子树上的所有点都没有back edge to any vertex discovered before u，所以如果u被removed，那么subtree rooted at v will be get disconnected from the whole graph. 因此connected component的数量会增加，因此u is a articulation point.

如果v的子树上的某点x有一条到达vertex discovered before u, say y的back edge, 那么there will be a path for any vertex in subtree rooted at v to reach y even after removal of u, 所以u不是articulation point.

3) Bridge  
If an edge between u and v is removed, then there is no path between u and v, this edge is a bridge.  
和articulation point的含义是一样的。

#### 算法解释  

所以寻找articulation points可以从brute force转变成find a back edge exisit or not for every vertex.  
So we need to keep track of the discovery time of each vertex v, and the earliest discovered vertex that can be reached from any of the vertices in the subtree rooted at v. 

如果v及其子树上的某点x的back edge能到的最早的已经被discovered的点的discovery time >= discovery(u)，那么v就没有back edge， u就是articulation point.

Q：如果u是tree的root，没有ancestors，这时该怎么判断root是不是articulation point呢？  
A：If root has more than one child, then root is an articulation point; otherwise, it is not.  
原因：如果root有两个child，那么这两个child及其子树之间肯定不会有edge，否则这两个child就可以合并成一个child了。所以这种条件下把root remove，那么connected component的数量就会增加。因此上述判断方法成立。

所以我们至少需要两个数组，分别存放每个点的discovery time(disc[]) and earliest discovered time of vertex can be reached(low[]):
至于判断vertex又没有被visited过，可以和discovery time 数组合并，比如初始化disc array = -1. 当遇到一个点，先看其discovery time是否等于-1，如果是，就表示没有visited过。

low[] array的用法解释：the discovery time of the earliest discovered vetex reachable from subtree rooted at v. 就是v的子树上的点的back edge所能到达的disc time最早的点的disc time，存放的是discovery time!

另外对于没有给现成graph结构的题，还要自己创建一个List< Integer >[] 数组来表示graph结构。  
还需要一个global variable to keep track of  current discovery time.

O(V + E)

```java
class Solution {
    public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
        List<List<Integer>> res = new ArrayList<>();
        
        List<Integer>[] graph = new ArrayList[n];
        for(int i = 0; i < n; i++){
            graph[i] = new ArrayList<>();
        }
        
        for(List<Integer> con : connections) {
            graph[con.get(0)].add(con.get(1));
            graph[con.get(1)].add(con.get(0));
        }
        
        int[] disc = new int[n];
        int[] low = new int[n];
        
        Arrays.fill(disc, -1);
        
        for(int i = 0; i < n; i++){
            if(disc[i] == -1){
                dfs(graph, i, 0, disc, low, res);
            }
        }
        
        return res;
    }
    
    int time = 0;
    public void dfs(List<Integer>[] graph, int u, int parent, 
                    int[] disc, int[] low, List<List<Integer>> res) {
        
        disc[u] = low[u] = time++;
        
        int cnt = graph[u].size();
        if(cnt == 0) return;
        
        for(int i = 0; i < cnt; i++) {
            int v = graph[u].get(i);
            
            if(v == parent) continue;
            
            //unvisited
            if(disc[v] == -1) {
                dfs(graph, v, u, disc, low, res);
                
                low[u] = Math.min(low[u], low[v]);
                
                //if u is deleted, then there is no way for v to back to u
                //so found a critical edge
                if(low[v] > disc[u]){
                    res.add(Arrays.asList(u, v));
                }
                
            }else{
                //u is not in the subtree of v, so cannot use low[v] here
                low[u] = Math.min(low[u], disc[v]);
            }
        }
    }
}
```

### Kosaraju Algorithm

3.How to find a strongly connected component?  
Let G be a directed graph, G* denotes the graph with the direction of all edges reversed.

a) call DFS(G) and record finish time f(v) of vertex v.  
b) call DFS(G*) but the vertices are considered in a descending order of finish time.

### Tarjan's Algorithm

This algorithm only requires one DFS traversal.

维护两个存放vertex访问顺序的数组，访问时间都是该SCC的出发点的访问时间

```html

```

O(V+E) for a graph represented using adjacency list

### Example

```java

```
