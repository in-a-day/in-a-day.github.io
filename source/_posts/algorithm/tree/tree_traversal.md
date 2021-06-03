---
title: 树的遍历
tags:
  - 树的遍历
categories: tree
date: 2021-06-03 10:41:41
---

树的遍历主要分为两种: 深度优先和层次遍历.

### **深度优先**

![BinaryTreeTraversal](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/algorithm/binary_tree_traversal.svg.png)_**二叉树深度优先遍历**_

树的深度优先遍历主要分为以下三种:

- 前序遍历: 先访问根节点, 再访问左子树, 最后访问右子树, 图中红色点的遍历顺序
- 中序遍历: 先访问左子树, 再访问根节点, 左后访问右子树, 图中绿色点的遍历顺序
- 后序遍历: 先访问左子树, 再访问右子树, 最后访问根节点, 图中蓝色点的遍历顺序

#### **前序遍历**

访问顺序: 根 --> 左 --> 右

递归方式:

```java
public void preOrder(TreeNode root) {
    if (root == null) return;
    visit(root);
    preOrder(root.left);
    preOrder(root.right);
}
```

迭代方式, 使用栈进行模拟, 由于栈是先进后出, 所以先将右子树放入栈中, 再放入左子树:

```java
public void preOrder(TreeNode root) {
    if (root == null) return;
    Stack<TreeNode> s = new Stack<>();
    s.push(root);
    TreeNode node;
    while (!stack.isEmpty()) {
        node = s.pop();
        // 访问节点
        visit(node);
        if (node.right != null) s.push(node.right);
        if (node.left != null) s.push(node.left);
    }
}
```



#### **中序遍历**

访问顺序: 左 --> 根 --> 右

递归方式:

```java
public void inOrder(TreeNode root) {
    if (root == null) return;
    inOrder(root.left);
    visit(root);
    inOrder(root.right);
}
```

迭代方式, 由于需要先访问左子树, 所以先一直遍历左子树, 放入栈中, 不存在时, 遍历右子树:

```java
public void inOrder(TreeNode root) {
    Stack<TreeNode> s = new Stack<>();
    TreeNode node = root;
    while (!s.isEmpty() || root != null) {
        if (node != null) {
			s.push(node);
            node = node.left;
        } else {
            node = s.pop();
            visit(node);
            node = node.right;
        }

    }
}
```



#### **后续遍历**

遍历顺序: 左 --> 右 --> 根

递归方式:

```java
public void postOrder(TreeNode root) {
    if (root == null) return;
    postOrder(root.left);
    postOrder(root.right);
    visit(root);
}
```

迭代方式, 由于后序遍历需要最后访问根节点, 所以会有重复访问的问题, 需要设置一个变量保存上次访问的节点, 避免重复访问:

```java
public void postOrder(TreeNode root) {
    Stack<TreeNode> s = new Stack<>();
    s.push(root);
    TreeNode node = root, peek;
    while (!s.isEmpty() || node != null) {
        if (node != null) {
            s.push(node);
            node = node.left;
        } else {
            peek = s.peek();
            // 如果当前节点存在右子节点, 且并没有访问过该右子节点, 将该节点加入栈中
            
            if (peek.right != null && peek.right != lastVisited) {
                node = node.peek;
            } else {
                // 否则不存在右子节点, 或者是上次访问的是该右子节点, 直接访问当前节点
                // 同时缓存访问节点
                visit(peek);
                lastVisited = s.pop();
            }
        }
    }
}
```



### **层次遍历**

![BinaryTreeLevelTraversal](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/algorithm/binary_tree_level_traversal.svg.png)_**二叉树层次遍历**_

遍历顺序: 按照每一层依次遍历.

迭代方式: 

```java
public void levelOrder(TreeNode root) {
    if (root == null) return;
    // 使用队列保存节点, 先进先出
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    TreeNode node = root;
    while (!q.isEmpty()) {
        node = q.poll();
        visit(node);
        if (node.left != null) q.offer(node.left);
        if (node.right != null) q.offer(node.right);
    }
}
```

迭代且使用层级:

```java
public void levelOrder(TreeNode root) {
    if (root == null) return;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    TreeNode node = root;
    int levelLength;
    while (!q.isEmpty()) {
        // levelLength: 每层节点个数
        levelLength = q.size();
        while (levelLength > 0) {
            node = q.poll();
            visit(node);
            if (node.left != null) q.offer(node.left);
            if (node.right != null) q.offer(node.right);
            levelLength--;
        }
    }
}
```

递归方式: 并不是真正按照层次遍历, 而是使用 DFS模拟层次(出自https://leetcode.com/problems/binary-tree-level-order-traversal/discuss/33445/Java-Solution-using-DFS):

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new LinkedList<>();
    levelHelper(res, root, 0);
    return res;
}

public void levelHelper(List<List<Integer>> res, TreeNode root, int height) {
    if (root == null) return;
    // height代表当前节点所属的层级, size()用于判断当前节点层级是否已经存在于res中
    // 例如树为[1,2,3], 根节点1进入后, 递归left和right传入的height为1, 在left中会添加LinkedList, 此时size()变为2, 所以之后遍历right时, 不会继续添加新的LinkedList
    if (height >= res.size()) {
        res.add(new LinkedList<Integer>());
    }
    res.get(height).add(root.val);
    levelHelper(res, root.left, height+1);
    levelHelper(res, root.right, height+1);
}
```



### **参考**

- wiki: https://en.wikipedia.org/wiki/Tree_traversal
- leetcode: https://leetcode.com/problems/binary-tree-level-order-traversal/discuss/33445/Java-Solution-using-DFS