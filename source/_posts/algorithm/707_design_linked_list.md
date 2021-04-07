---
title: 设计链表
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-07 14:30:02
---
## 题目([leetcode-707](https://leetcode.com/problems/design-linked-list/))
设计一个单向或双向链表.  
实现MyLinkedList类:
- MyLinkedList() 实例化对象.
- get(int index) 返回`index`位置的值. 如果index不合法, 返回 -1.
- addAtHead(int val) 在头部添加节点.
- addAtTail(int val) 在尾部添加节点.
- addAtIndex(int index, int val) 在指定位置添加节点. 若index等于链表长度, 节点添加至最后, 若index大于链表长度, 不添加节点.
- deleteAtIndex(int index) 如果index合法, 删除index节点元素.

## 题解
### python
```python
class Node:
	def __init__(self, val):
		self.val = val
        self.next = None
        self.prev = None

class MyLinkedList:

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self._head = None
        self._tail = None
        self._size = 0
        
    def _get_node(self, index):
        if index == 0:
            return self._tail
        if index == self._size:
            return self._tail
        if index > self._size or index < 0:
            return None
        count = 1
        curr = self._tail
        while count < index:
            curr = curr.next
        return curr


    def get(self, index: int) -> int:
        """
        Get the value of the index-th node in the linked list. If the index is invalid, return -1.
        """
        node = self._get_node(index)
        if node:
            return node.val
        return -1
        

    def addAtHead(self, val: int) -> None:
        """
        Add a node of value val before the first element of the linked list. After the insertion, the new node will be the first node of the linked list.
        """
        node = Node(val)
        if self._head:
            node.next = self._head
            self._head.prev = node
        else:
            self._head = self._tail = node
        self._size += 1
        

    def addAtTail(self, val: int) -> None:
        """
        Append a node of value val to the last element of the linked list.
        """
        node = Node(val)
        if self._tail:
            self._tail.next = node
            node.prev = self._tail
        else:
            self._head = self._tail = node
        self._size += 1
            
        

    def addAtIndex(self, index: int, val: int) -> None:
        """
        Add a node of value val before the index-th node in the linked list. If index equals to the length of linked list, the node will be appended to the end of linked list. If index is greater than the length, the node will not be inserted.
        """
        if index == 0:
            self.addAtHead(val)
        if index == self._size:
            self.addAtTail(val)
        if index > self._size or index < 0:
            return
        insert = Node(val)
        node = self._get_node(index)
        next = node.next
        next.prev = insert
        insert.next = next
        insert.prev = node
        node.next = insert
        self._size += 1

    def deleteAtIndex(self, index: int) -> None:
        """
        Delete the index-th node in the linked list, if the index is valid.
        """
        if index > self._size or index <= 0 or self._size <= 0:
            return
        
        if index == 1:
            self._head = self._head.next
        if index == self._size:
            if self._size == 1:
                self._head = self._tail = None
            else:
                self._tail = self._tail.prev
                self._tail.next = None
        node = self._get_node(index)

        prev = node._prev
        next = node._next

        self._size -= 1
        
```


