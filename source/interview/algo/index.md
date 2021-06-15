---
title: 常见算法题思路
comments: false
---


# 一、链表

## 1. 反转链表

用三个变量 prev、curr、next 分别暂存上一个结点、当前结点、下一个结点，然后依次反转

```java
while (curr != null){
    next = curr.next;

    // 反转当前结点
    curr.next = prev;

    prev = curr;
    curr = next;
}
```

递归解法

```java
/**
* 输入一个节点 head，将「以 head 为起点」的链表反转，并返回反转之后的头结点
**/
private static ListNode reverseByRecursion(ListNode head){
    if (head == null || head.next == null){
        return head;
    }
    ListNode last = reverseByRecursion(head.next);
    head.next.next = head;
    head.next = null;
    return last;
}
```

## 2. 从尾到头打印链表

逆向输出链表，自然会想到用后进先出的栈。先遍历链表，把遍历的值依次写入栈，然后再出栈即可。

事实上，递归本身就是栈结构，这道题可以直接用递归代替栈。


```java
// 递归实现
private static void reversePrint(ListNode node){

    if (node.next != null) {
        reversePrint(node.next);
    }

    System.out.println(node.value);

}
```

> 《剑指offer》题目6

## 3. 删除链表的节点

删除链表节点的两种方法(比如要删除 i 节点)：

- 方法一: 遍历到 i 的前一个节点 h，将 h 指向 i 的下一个节点 j，然后删除 i
- 方法二：把节点 j 的内容复制到 i，将 i.next 指向 j 的下一节点 k，然后删除 j

> 《剑指offer》题目18

## 4. 链表中倒数第 k 个节点

两个指针，如求倒数第 3 个节点，A指针先走3步，然后B指针开始走，当A指针到尾的时候，B指针所指就是倒数第 k 个节点。（注意判空，例如链表只有2个节点）


```
2 → 88 → 60 → 21 → 6 → 13 → 78
↑（B）    ↑（A）

2 → 88 → 60 → 21 → 6 → 13 → 78
                   ↑（B）    ↑（A）
```

>  《剑指offer》题目22

## 5. 链表中环的入口节点

如果一个链表中包含环，如何找出环的入口节点？

两个指针，一个一次走一步，另一个一次走两步，如果快的指针追上了慢的指针，那链表就包含环。否则如果走得快的到了末尾（node.next = null）都没追上慢的，就不包括环。

相遇节点就是入口节点。