---
title: Recursion of Binary Tree
date: 2019-02-12 20:48:17
tags:
---
### Validate Binary Search Tree

top-bottom, 先处理node，再recursive;  
bottom-top, 先recursive，再处理node.  

1.联想到inorder
```java
class Solution {
    TreeNode lastNode;
    boolean isValid;
    public boolean isValidBST(TreeNode root) {
        lastNode = null;
        isValid = true; //找到不满足bst条件的就返回false，所以初始为true
        
        inorder(root);
        return isValid;
    }
    
    public void inorder(TreeNode root){
        if(root == null){
            return;
        }
        
        inorder(root.left);
        if(lastNode != null && lastNode.val >= root.val){
            isValid = false;
            return;
        }
        
        lastNode = root;
        inorder(root.right);
    }
}
```

2.top-bottom, O(height)
```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return helper(root, null, null);
    }
    
    //upper and lower is the range of the current node
    //type设为Integer很好！
    public boolean helper(TreeNode root, Integer upper, Integer lower){
        if(root == null){
            return true;
        }
        if(upper != null && root.val >= upper){
            return false;
        }
        if(lower != null && root.val <= lower){
            return false;
        }
        //left node, its range should in (lower, root.val), here lower == null, 
        //right node's range should in (root.val, upper), here upper = null
        return helper(root.left, root.val, lower) && helper(root.right, upper, root.val);
    }
}
```
3.
root.val > max(node in left subtree).
root.val < min(node in right subtree)
```java
class Solution {
    class Result {
        boolean isBST;
        int max, min;
        public Result(int max, int min, boolean isBST){
            this.max = max;
            this.min = min;
            this.isBST = isBST;
        }
    }
    public boolean isValidBST(TreeNode root) {
        Result res = traverse(root);
        return res.isBST;
    }
    
    public Result traverse(TreeNode root){
        if(root == null){
            return new Result(Integer.MIN_VALUE, Integer.MAX_VALUE, true);
        }
        Result left = traverse(root.left);
        Result right = traverse(root.right);
        
        if(!left.isBST || !right.isBST || root.val <= left.max || root.val >= right.min){
            return new Result(0, 0, false);
        }
        
        return new Result(Math.max(root.val, right.max), Math.min(root.val, left.min), true);
    }
}
```

### Largest BST Tree
Given a binary tree, find the largest subtree which is a BST, where largest means subtree with largest number of nodes in it.

要求用O(n)做出来，也就是说每个点遍历一次。可以从下往上遍历，backtracking时保存每个点是否是BST和size。  
利用了validate BST tree的第三种做法
```java
class Solution {
    class Node {
        int max, min;
        int size;
        public Node(int size, int max, int min){
            this.size = size;
            this.max = max;
            this.min = min;
        }
    }
    
    int maxCnt;
    public int largestBSTSubtree(TreeNode root) {
        maxCnt = 0;
        traverse(root);
        return maxCnt;
    }
    
    public Node traverse(TreeNode root){
        if(root == null){
            return new Node(0, Integer.MIN_VALUE, Integer.MAX_VALUE);
        }
        
        Node left = traverse(root.left);
        Node right = traverse(root.right);
        
        //if the current subtree is not BST
        if(left.size == -1 || right.size == -1 || root.val <= left.max || root.val >= right.min){
            //max和min都设为0，这样往上面一步的root不可能同时满足> 0 && < 0
            //所以自然还是非BST(我是这样理解的)
            return new Node(-1, 0, 0);
        }
        
        int size = left.size + right.size + 1;
        maxCnt = Math.max(maxCnt, size);
        
        return new Node(size, Math.max(root.val, right.max), Math.min(root.val, left.min));
    }
}
```