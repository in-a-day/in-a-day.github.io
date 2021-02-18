---
title: 链表中环的入口
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-18 12:25:26
---
## 题目([leetcode-142](https://leetcode.com/problems/linked-list-cycle-ii/))
给定一个链表, 返回链表环的起始位置节点. 如果没有环, 返回null.  
例:
![q1](https://cdn.jsdelivr.net/gh/in-a-day/cdn@main/images/leetcode/142/q_1.png)
```
Input: head = [3,2,0,-4], pos = 1
Output: tail connects to node index 1
Explanation: 链表的第二个节点是环的入口, 返回第二个节点.
```

## 题解
首先判断链表是否有环, 无环返回null即可.
下面来看有环的情况:
假设有两个人在环形操场跑步, a 速度是b 速度的2倍, 若两人同时同地出发, 则a跑完两圈, b刚好跑完一圈相遇. 若a在b的前面位置出发, 则b尚未跑完一圈时, 两人相遇.  
则可使用双指针判断是否有环, 慢指针slow一次走一步, 快指针fast一次走两步.若两个指针相遇代表链表有环.  
设链表头结点到环起始位置的距离为s1, 环起始位置到指针相遇位置的距离为s2, 相遇位置到环的起始位置距离为s3.  
由上面分析可知, 当slow进入环后, slow指针最多只可能绕环一圈, 所以$s_{slow} = s1 + s2$. 由于fast的速度是slow的两倍, 所以$s_{fast} = 2 * (s1 + s2)$.  
链表长度未知,且环的位置位置, 所以在slow指针尚未进入环中时, fast指针可能已经绕环n圈.则$s_{fast} = s1 + n * (s2 + s3) + s2$.  
综上:
1. $s_{slow} = s1 + s2$
2. $s_{fast} =  2 * (s1 + s2)$
3. $s_{fast} = s1 + n * (s2 + s3) + s2$  

则: 
1. $2 * (s1 + s2) = s1 + n * (s2 + s3) + s2$
2. $s1 = (n - 1) * (s2 + s3) + s3$  

由上, 当fast和slow相遇时, 此时slow指针再前进长度为s3完成一次环, 由最后一个公式可得: 将fast指针指向head节点, 且slow与fast每次都走1步,则再次相遇的位置即环的起始位置(slow相遇后再走的距离刚好是$(n - 1) * (s2 + s3) + s3$).

### python
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def detectCycle(self, head: ListNode) -> ListNode:
```

### java
```java
```
