---
title: 反转链表
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-07 16:10:30
---
## 题目([leetcode-206](https://leetcode.com/problems/reverse-linked-list/))
反转单链表.  
例:
```
Input: 1->2->3->4->5->NULL
Output: 5->4->3->2->1->NULL
```

## 题解

### python
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
		prev = None
		while head:
			next = head.next
			head.next = prev
            prev = head
            head = next

    def recursiveRevese(self, head: ListNode) -> ListNode:
        return self.recursion(head, None)

    def recursion(self, head, pre) -> ListNode:
        if head is None:
            return pre
        
        next = head.next
        head.next = pre
        pre = head
        return self.recursion(next, pre)
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
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        while (head != null) {
            ListNode next = head.next;
            head.next = prev;
            prev = head;
            head = next;
        }
        return prev;
    }
}
```
