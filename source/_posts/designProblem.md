---
title: File System design例题集合
date: 2019-06-20 15:49:25
tags: design
---

### 1. Design File System

题目：  
You are asked to design a file system which provides two functions:

* create(path, value)  
Creates a new path and associates a value to it if possible and returns True. Returns False if the path already exists or its parent path doesn't exist.
* get(path)  
Returns the value associated with a path or returns -1 if the path doesn't exist.

The format of a path is one or more concatenated strings of the form: / followed by one or more lowercase English letters.  
For example, /leetcode and /leetcode/problems are valid paths, while an empty string and / are not.

思路：  

method 1: only to check the nearest parent root existence.

```java
class FileSystem {
    Map<String, Integer> map;
    public FileSystem() {
        map = new HashMap<>();
        //for path like "/a", it is valid, but it parent is "" 
        // and the requirment says an empty string and / are not valid
        //so we make the value of "" is -1.
        //这样代码就都适用于"/a" 和其他有多个/的path了
        map.put("", -1);
    }
    
    public boolean create(String path, int value) {
        int index = path.lastIndexOf("/");
        String parent = path.substring(0, index);
        
        if(!map.containsKey(parent) || map.containsKey(path)) return false;
        map.put(path, value);
        return true;
    }
    
    public int get(String path) {
        if(map.containsKey(path)) return map.get(path);
        
        return -1;
    }
}
```

</br>

method2: use trie to store path  
但是这个trie的每个node不是像之前那样是一个char，而是一个path的部分，比如/a/bc/def，那么在trie上体现为a是一个节点，bc是一个，def是一个，这三个节点相连。所以需要把题中给的path split by /.

time: O(n), n is strs.length

```java
class FileSystem {
    class TrieNode {
        Map<String, TrieNode> children;
        int val;
        
        public TrieNode() {
            children = new HashMap<>();
            val = -1;//help to check path existence
        }
    }
    
    TrieNode root;
    public FileSystem() {
        root = new TrieNode(); //represents ""
    }
    
    public boolean createPath(String path, int value) {
        TrieNode cur = root;
        String[] arrs = path.split("/");
        
        //arrs[0] = "", so we skip it
        for(int i = 1; i < arrs.length; i++) {
            if(!cur.children.containsKey(arrs[i])){
                //the path closest parent exists
                if(i == arrs.length - 1){
                    cur.children.put(arrs[i], new TrieNode());
                }else{
                    return false;
                }
            }
            
            cur = cur.children.get(arrs[i]);
        }
        
        //the path already exists
        if(cur.val != -1){
            return false;
        }
        
        cur.val = value;
        return true;
    }
    
    public int get(String path) {
        TrieNode cur = root;
        String[] arrs = path.split("/");
        
        for(int i = 1; i < arrs.length; i++) {
            if(!cur.children.containsKey(arrs[i])){
                return -1;
            }
            
            cur = cur.children.get(arrs[i]);
        }
        
        return cur.val;
    }
}
```

### 2. Design Search Autocomplete System

题目：  
https://leetcode.com/problems/design-search-autocomplete-system/description/

简要概括就是：用户一个一个的char输入，当输入#号，表示输入结束; 有效char是小写字母或space 
每输入一个有效char，都要以list形式返回前三个hot sentences that have the same prefix as part of the sentence already typed(因此我们需要一个变量来不断更新本次输入的内容)。当输入#，返回空list

关于如何定义hot degree: Hot degree is the number of times a user typed the exactly sentence before.  
返回的3个hottest sentences should sorted by hot degree, if same, then sort by ascii order.  
If less than 3 hottest sentences, return as many as you can.  

constructor function的定义：

```java
AutocompleteSystem(String[] sentences, int[] times)
sentences array consists of previously typed sentences
times is corresponding times of a sentence
```

思路：  
用sentences建立trie，当遇到sentence的结尾char所对应的node时，就更新times += sentencen.times
查找一个current sentence，把所有可能符合前缀的结果放入一个list中，sort list后返回前3个最大times的

至于查找的具体步骤是：  
指针停下的node的times如果 > 0，表示sentences中正好有一个对应该current input的，就放入list中。  
如果此时该点的children中有非null的，就从这个点出发做dfs，得到该点以下所有的sentences和times，放入list中。

time：  
1)create trie: O(n * k), n个sentence，平均长度为k  
2)input() function,
  O(p + q + mlogm) p是curSent的长度，q是以curSent的终点node作为root的下面的所有node数目。  
  m是放入hot sentences的list的length，mlogm是sort list的time。 

```java
class AutocompleteSystem {
    class TrieNode {
        TrieNode[] children;
        int times;
        String str; //the cur string until current node so far 
        public TrieNode(String s) {
            children = new TrieNode[27];
            this.times = 0;
            this.str = s;
        }
    }
    
    TrieNode root;
    //p as pointer
    //so far new input char, don't need to search from tree root again
    TrieNode p; 
    
    public AutocompleteSystem(String[] sentences, int[] times) {
        root = new TrieNode("");
        p = root;
        
        buildTree(sentences, times);
    }
    
    public List<String> input(char c) {
        List<String> res = new ArrayList<>();
        
        if(c == '#') {
            if(p != root) {
                p.times++;
            }
            
            p = root;
            return res;
        }
          
        int id = getId(c);    
        if(p.children[id] == null) {
            p.children[id] = new TrieNode(p.str + c);
            p = p.children[id];
            return res;
        }
        
        p = p.children[id];
        //minheap
        Queue<TrieNode> pq = new PriorityQueue<>((a, b) -> (a.times == b.times ? b.str.compareTo(a.str) : a.times - b.times));
        
        getHotSentences(p, pq);
        while(!pq.isEmpty()){
            res.add(pq.poll().str);
        }
        Collections.reverse(res);
        return res;
    }
    
    private void buildTree(String[] sentences, int[] times) {
        for(int k = 0; k < sentences.length; k++) {
            String word = sentences[k];
            TrieNode cur = root;
            
            for(int i = 0; i < word.length(); i++) {
                char c = word.charAt(i);
                int id = getId(c);
                
                if(cur.children[id] == null) {
                    cur.children[id] = new TrieNode(cur.str + c);
                }
                cur = cur.children[id];
            }
            cur.times += times[k];
        }
    }
```

### 3.Design In-Memory File System

题目：  
https://leetcode.com/problems/design-in-memory-file-system/description/

实现4个methods:  
ls(path), mkdir(dirPath), addContentToFile(filePath), readContentFromFile(filePath).所有传入的parameter都是valid和lowercase

思路:  
Each directory contains two maps, dirs & files.  
dirs map < subdirName, subdir-structure>  
files map < fileName, fileContent>

```java
class FileSystem {
    class Dir{
        //当前路径下的所有<subDir name, subDir structure>
        Map<String, Dir> dirs = new HashMap<>();
        //当前路径中的file及其content
        Map<String, String> files = new HashMap<>();
    }
    
    Dir root;
    public FileSystem() {
        root = new Dir();
    }
    
    public List<String> ls(String path) {
        Dir p = root;
        List<String> res = new ArrayList<>();
        
        //这里考虑到path为"/"时，root dir下可能有东西，
        //这时候要找到所给的dir下所有的dirs和files，
        //所以用一个 if(!)来寻找子dir和file
        if(!path.equals("/")){ 
            String[] arrs = path.split("/");

            for(int i = 1; i < arrs.length - 1; i++){
                p = p.dirs.get(arrs[i]);
            }

            String lastStr = arrs[arrs.length - 1];
            //if is a file path
            if(p.files.containsKey(lastStr)){
                res.add(lastStr);
                return res;
            }else{
                p = p.dirs.get(lastStr);
            }
        }
        
        res.addAll(new ArrayList<>(p.dirs.keySet()));
        res.addAll(new ArrayList<>(p.files.keySet()));
        
        Collections.sort(res);
        return res;
    }
    
    //path is the directory path
    public void mkdir(String path) {
        Dir p = root;
        String[] arrs = path.split("/");
        
        for(int i = 1; i < arrs.length; i++) {
            if(!p.dirs.containsKey(arrs[i])){
                p.dirs.put(arrs[i], new Dir());
            }
            p = p.dirs.get(arrs[i]);
        }
    }
    
    public void addContentToFile(String filePath, String content) {
        Dir p = root;
        String[] arrs = filePath.split("/");
        
        for(int i = 1; i < arrs.length - 1; i++){
            p = p.dirs.get(arrs[i]);
        }
        
        String fileName = arrs[arrs.length - 1];
        p.files.put(fileName, p.files.getOrDefault(fileName, "") + content);
    }
    
    public String readContentFromFile(String filePath) {
        Dir p = root;
        String[] arrs = filePath.split("/");
        
        for(int i = 1; i < arrs.length - 1; i++){
            p = p.dirs.get(arrs[i]);
        }
        
        return p.files.get(arrs[arrs.length - 1]);
    }
}
```

Advantages: 把files和dirs放在不同的map中，it is expandable to include more commands.  
For example, `rmdir`, rename files/dirs, relocate a sub from one to the other, extract only dirs or files list on any path.
</br>

另一种做法，code style超好！
仍然是建立tree，每个node有一个member叫content，用content的length() > 0来判断是否是file节点。

```java
//Used TreeMap at insertion takes O(logN). GetList method takes O(N)
class FileSystem {
    
    private class Node {
        TreeMap<String, Node> children;
        String name; //dir name or file name
        StringBuilder content; //file content
        
        public Node(String name) {
            children = new TreeMap<>();
            this.name = name;
            this.content = new StringBuilder();
        }
        
        public String getContent() {
            return content.toString();
        }
        
        public String getName() {
            return name;
        }
        
        public void addContent(String s) {
            content.append(s);
        }
        
        public boolean isFile() {
            return content.length() > 0;
        }
        
        public List<String> getList() {
            List<String> list = new ArrayList<>();
            
            if(isFile()) {
                list.add(getName());
            }else{
                list.addAll(children.keySet());
            }
            
            return list;
        }
    }
    
    Node root;
    public FileSystem() {
        root = new Node("");
    }
    
    public List<String> ls(String path) {
        Node node = helper(path);
        return node.getList();
    }
    
    public void mkdir(String path) {
        helper(path);
    }
    
    public void addContentToFile(String filePath, String content) {
        Node node = helper(filePath);
        node.addContent(content);
    }
    
    public String readContentFromFile(String filePath) {
        Node node = helper(filePath);
        return node.getContent();
    }
    
    //兼顾查找node和build tree的功能
    //由于ls()中的path一定是valid，所以调用该函数时，putIfAbsent()不会起作用
    private Node helper(String path) {
        String[] arrs = path.split("/");
        
        Node cur = root;
        for(String arr : arrs) {
            if(arr.length() == 0) continue;
            
            cur.children.putIfAbsent(arr, new Node(arr));
            cur = cur.children.get(arr);
            
            if(cur.isFile()) break;
        }
        
        return cur;
    }
}
```

### 4.Design Log Storage System

题目：  
给一些logs，写两个method

https://leetcode.com/problems/design-log-storage-system/description/

Year ranges from [2000,2017]. Hour ranges from [00,23].  
Output for Retrieve has no order required.

思路：  
to get the lower bound, append a suffix of "2000:01:01:00:00:00" to the prefix of s.  
to get the upper bound, append a suffix of "2017:12:31:23:59:59" to the prefix of e.

```java
class LogSystem {
    String min = "2000:01:01:00:00:00", max = "2017:12:31:23:59:59";
    Map<String, Integer> indices;//the granularity's pos in the string，分号的位置
    TreeMap<String, List<Integer>> logs; //<timestamp, list of id>
    
    public LogSystem() {
        indices = new HashMap<>();
        logs = new TreeMap<>();
        
        indices.put("Year", 4);
        indices.put("Month", 7);
        indices.put("Day", 10);
        indices.put("Hour", 13);
        indices.put("Minute", 16);
        indices.put("Second", 19);
    }
    
    public void put(int id, String timestamp) {
        logs.putIfAbsent(timestamp, new ArrayList<>());
        logs.get(timestamp).add(id);
    }
    
    public List<Integer> retrieve(String s, String e, String gra) {
        List<Integer> res = new ArrayList<>();
        
        int pos = indices.get(gra);
        String start = s.substring(0, pos) + min.substring(pos);
        String end = e.substring(0, pos) + max.substring(pos);
        
        Map<String, List<Integer>> subMap = logs.subMap(start, true, end, true);
        
        for(List<Integer> list : subMap.values()){
            res.addAll(list);
        }
        
        return res;
    }
}
```