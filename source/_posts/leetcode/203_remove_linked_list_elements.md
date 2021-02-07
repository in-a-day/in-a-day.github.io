---
title: 移除链表中的元素
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-07 10:11:36
---

## 题目([leetcode-203](https://leetcode.com/problems/remove-linked-list-elements/))
给定一个整数链表和值val, 移除链表中值等于val的元素.  
例:
```
Input:  1->2->6->3->4->5->6, val = 6
Output: 1->2->3->4->5
```

## 题解
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def removeElements(self, head: ListNode, val: int) -> ListNode:
        """
        curr存储当前节点, prev存储上一个值不为val的节点
        1. 当前值 == val时:
            a. 当前节点是头结点, 头结点移动到下一个节点
            b. 当前节点不是头节点, prev节点的next设置为curr的next
        2. 当前值!=val, 将prev节点设置为curr, curr设置为curr.next
        """
        curr = prev = head
        while curr:
            if curr.val == val:
                # 当前节点是头节点
                if curr == head:
                    head = prev = curr.next
                else:
                    prev.next = curr.next
            else:
                prev = curr
            curr = curr.next
        return head

    def remove_with_fake_head(self, head, val):
        """
        添加虚拟节点, 统一处理原来的头结点和其他节点
        """
        fake_head = new ListNode()
        fake_head.next = head
        curr = fake_head
        # 当前节点是fake_head, 所以使用curr.next进行判断
        while curr.next:
            # 当前节点值 == val, 删除
            if curr.next.val == val:
                curr.next = curr.next.next
            else:
                curr = curr.next
        # 返回真正的头结点
        return fake_head.next
```

### java
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        ListNode fakeNode = new ListNode(0);
		fakeNode.next = head;
		ListNode curr = fakeNode;
		while (curr.next != null) {
            if (curr.next.val == val) curr.next = curr.next.next;
            else curr = curr.next;
		}

        return fakeNode.next;
    }
}
```


