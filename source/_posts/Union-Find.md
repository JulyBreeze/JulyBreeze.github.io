---
title: Union Find
date: 2019-02-09 13:30:13
tags: union find
---

### Union Find
三个部分：
* 建立father数组和初始化，一般都是father[id] = id;
* find Father()  
* Union()，即判断两个相连点的father是否相等，如果不相等，就把两者归到一个father。

一般群组类问题，很适合使用Union Find。

### Examples
#### Number of Connected Components in an Undirected Graph
Given n nodes labeled from 0 to n - 1 and a list of undirected edges (each edge is a pair of nodes), write a function to find the number of connected components in an undirected graph.求的是连通图的个数.
```java
class Solution {
    public int countComponents(int n, int[][] edges) {
        int[] father = new int[n];
        
        for(int i = 0; i < n; i++){
            father[i] = i;
        }
        
        for(int[] edge : edges){
            int root1 = find(father, edge[0]);
            int root2 = find(father, edge[1]);
            
            //A和B是同一条edge上的两点，如果它们不在同一个set中，就让它们归属于一个set
            //比如让A进入B所在的组，这样n - 1就是现在的所有组的组数
            if(root1 != root2){
                father[root1] = root2;
                n--;
            }
        }
        
        return n;
    }
    
    public int findFather(int[] father, int id){
        if(father[id] == id){
            return id;
        }
        father[id] = findFather(father, father[id]);
        return father[id];
    }
}
```

#### Number of Islands II
A 2d grid map of m rows and n columns is initially filled with water. We may perform an addLand operation which turns the water at position (row, col) into a land.

Given a list of positions to operate, count the number of islands after each addLand operation. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

要求出每次operation后的岛屿个数，岛屿个数在变化，为了解决这种之间会合并的情况，最好能够将每个陆地都标记出其属于哪个岛屿，这样就会方便统计岛屿个数。

可以是把所有涉及union find相关的函数放在一个class里，建立grid二维数组。或者不需简历二维数组，直接借助father数组的初始化来判断一个格子是否是0还是1，即把father全部初始化为-1，如果father[i] == -1，表示这个格子是海水。

```java
class UnionFind{
    int count; // the number of islands
    int[] father;
    
    //initialize
    public UnionFind(int n){
        father = new int[n];
        for(int i = 0; i < n; i++){
            father[i] = i;
        }
    }

    //union
    public void connect(int a, int b){
        int root1 = find(a);
        int root2 = find(b);

        if(root1 != root2){
            father[root1] = root2;
            count--;
        }
    }

    //find
    public int findFather(int id){
        if(father[id] == id){
            return id;
        }

        father[id] = findFather(father[id]);
        return father[id];
    }

    public int queryCount(){
        return count;
    }

    public void setCount(int total){
        count = total;
    }
}

class Solution {
    int[] dx = {0, 0, 1, -1};
    int[] dy = {1, -1, 0, 0};
    
    public List<Integer> numIslands2(int n, int m, Point[] operators) {
        List<Integer> res = new ArrayList<>();
        
        if(n == 0 || m == 0 || operators == null || operators.length == 0){
          return result;
        }

        int[][] grid = new int[n][m];
        UnionFind uf = new UnionFind(n * m);
        
        int cnt = 0;
        for(Point point : operators){
            //考虑到operations可能有重复的操作
            if(grid[point.x][point.y] == 1){
                res.add(cnt);
                continue;
            }

            grid[point.x][point.y] = 1;
            cnt++;
            uf.setCount(cnt);

            for(int i = 0; i < 4; i++){
                int x = point.x + dx[i];
                int y = point.y + dy[i];

                if(outbound(n, m, x, y) || grid[x][y] == 0){
                    continue;
                }

                uf.connect(point.x * m + point.y, x * m + y);
            }

            cnt = uf.queryCount();
            res.add(cnt);
        }
        return res;
    }

    public boolean outbound(int n, int m, int x, int y){
        return x < 0 || y < 0 || x >= n || y >= m;
    }
}
```

#### Graph is Valid Tree
Given n nodes labeled from 0 to n - 1 and a list of undirected edges (each edge is a pair of nodes), write a function to check whether these edges make up a valid tree. 判断一个graph是否是tree

要验证是否是树：1.全连通；2.没有环。

edges = nodes - 1不能保证是一棵树，因为有可能存在好几个环，比如ABCD四个点，ABC相连，也满足4个点3条边。

每个点初始都是自己孤立的存在，一共n个集合。每次取一条边，如果这条边的两端不在同一个集合中，就将其所在的集合合并成一个。如果在一个集合中，那么就说明存在环。在取完所有的n-1条边之后，应该所有的点都在一个集合中了，否则就不是树。
```java
public boolean validTree(int n, int[][] edges){
    int[] father = new int[n];
    for(int i = 0; i < n; i++){
        father[i] = i;
    }

    for(int[] edge : edges){
      int root1 = findFather(father, edge[0]);
      int root2 = findFather(father, edge[1]);
          
      if(root1 == root2){
          return false;
      }
      
      father[root1] = root2;
      n--;
    }

    return n == 1;
}
    
public int findFather(int[] father, int id){
    if(father[id] == id){
        return id;
    }

    father[id] = findFather(father, father[id]);
    return father[id];
}
```
