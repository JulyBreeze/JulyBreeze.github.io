---
title: Advanced Data Structure
date: 2019-10-01 15:28:28
tags: data structure
---

关于search的进化之路：  
逐个寻找O(n) -> binary search O(logn)(元素排列是sorted) -> hashMap O(1) -> balanced BST(Red-Black tree..)

Hash方法会存在冲突，因此hash地址相同的数会存放在一个linkelist中，采用linkelist是因为链表在内存中不是连续存储，因此可以充分利用内从。但是如果hash的buckets比较少，而数据量很大时，会导致每个bucket里的linkedlist很长。这样就有点趋向于O(n)线性搜索了。  
其实hashMap = linkedlist + red-black tree

Most of BST operations take O(h) time, but for a skewed binary tree, the operations may become O(n).

### Red–black tree红黑树

Red-Black Tree is a self-balancing BST, the height of RBT is always O(logN).

Five Properties:

* every node has a color either red or black
* root is black
* NULL leaf node is black
* red node must have differen color adjacent nodes
* every path froma node to any of its descendant Null node has the same number of black nodes  
也就是说没有一条路径会比其它路径长两倍，所以红黑树是接近平衡的bst

相比AVL，RBT允许局部比较小的不完全平衡。
Basic operation: add, delete, rotation(to keep tree remain red-black)

```java
public class RBTree<T extends Comparable<T>> {
    private RBTNode<T> mRoot;

    public RBTree() {
        this.mRoot = null;
    }
    private static final boolean RED = false;
    private static final boolean BLACK = true;

    public class RBTNode<T extends Comparable<T>> {
        boolean color;
        T key;

        RBTNode<T> parent;//为了方便，所以也定义了父节点
        RBTNode<T> left, right;
        
        public RBTNode<T key, boolean color, RBTNode<T> parent, RBTNode<T> left, RBTNode<T> right) {
            this.key = key;
            this.color = color;
            
            this.parent = parent;
            this.left = left;
            this.right = right;
        }

        public T getKey(){
          return key;
        }

        public String toString() {
          return key + (this.color == RED ? "R" : "B");
        }
    }
    
    //这些method在这里就省略不写block code了
    private boolean isRed(RBTNode<T> node){}
    private boolean isBlack(RBTNode<T> node){}

    private void setRed(RBTNode<T> node){}
    private void setBlack(RBTNode<T> node){}
    private void setColor(RBTNode<T> node, boolean color){}

    private RBTNode<T> search(T key){}
    private void insert(RBTNode<T> node){}
    private void remove(RBTNode<T> node){}

    //分别是insert和remove操作后，需要保持tree仍然是红黑树
    private void insertFix(RBTNode<T> node){}
    private void removeFix(RBTNode<T> node){}

    private void leftRotation(RBTNode<T> node){}
    private void rightRotation(RBTNode<T> node){}
}
```

左旋和右旋：

```java
private void leftRotation(RBTNode<T> x) {
    RBTNode<T> y = x.right;

    x.right = y.left;
    if(y.left != null){
        y.left.parent = x;
    }

    y.parent = x.parent;
    if(x.parent != null){
        this.mRoot = y;
    }else{
        if(x.parent.left == x){
            x.parent.left = y;
        }else{
            x.parent.right = y;
        }
    }

    y.left = x;
    x.parent = y;
}

public void rightRotation(RBTNode<T> x) {
    RBTNode<T> x = y.left;
    y.left = x.right;
    if(x.right != null){
        x.right.parent = x;
    }

    x.parent = y.parent;
    if(y.parent == null){
        this.mRoot = x;
    }else{
        if(y.parent.right == y){
            y.parent.right = x;
        }else{
            y.parent.left = x;
        }
    }

    x.right = y;
    y.parent = x;
}
```

insert()操作，插入的node颜色为Red.  
为什么是红色呢？因为红色可以尽可能介绍对5点property的violation。
1) If the insert node is the new root, we need to consider the color; otherwise, insert red node won't affect root.
2) won't affect the property that NULL node is black.
3) Since all path from one node to NULL node have the same number of black nodes, insert a red node won't affect the number of black nodes.

```java

```

### AVL tree

AVL tree is a self-balancing BST where the difference of height between left subtree and right subtree cannot be more than one.

To make sure that the tree remains AVL after insertion, two re-balancing operations:
* left rotation
* right rotation

### SkipList跳表

SkipList has the same asymptotic expected time bounds as balanced trees and are simpler, faster and use less space. Redis中的核心就是SkipList

insert: 从最底层开始；然后抛硬币来判断是否要向上层扩展。  
delete: 先search到后，从下往上删除

```java

```