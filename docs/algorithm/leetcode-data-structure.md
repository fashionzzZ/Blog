# LeetCode - 数据结构

* [链表](#链表)
* [树](#树)
* [栈和队列](#栈和队列)
* [哈希表](#哈希表)
* [字符串](#字符串)
* [数组和矩阵](#数组和矩阵)
* [图](#图)
* [位运算](#位运算)

---

## 链表

### [160. 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

这道题可以用环的思想来做，我们让两条链表分别从各自的开头开始往后遍历，当其中一条遍历到末尾时，我们跳到另一个条链表的开头继续遍历。两个指针最终会相等，而且只有两种情况，一种情况是在交点处相遇，另一种情况是在各自的末尾的空节点处相等。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode l1 = headA, l2 = headB;
        while (l1 != l2) {
            l1 = (l1 == null) ? headB : l1.next;
            l2 = (l2 == null) ? headA : l2.next;
        }
        return l1;
    }
}
```

### [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

这道题，题目让用迭代和递归两种方式实现。`画个图就明白了`
迭代：两个指针，一个指向当前节点的pre节点，一个指向当前节点的next节点，将当前节点的next指针指向pre节点，pre节点更新为当前节点，将之前的next节点作为当前节点，循环。

```java
public ListNode reverseList(ListNode head) {
    ListNode pre = null;
    while (head != null) {
        ListNode next = head.next;
        head.next = pre;
        pre = head;
        head = next;
    }
    return pre;
}
```

递归：递归解法的思路是，不断的进入递归函数，直到head指向倒数第二个节点，因为head指向空或者是最后一个结点都直接返回了，newHead则指向对head的下一个结点调用递归函数返回的头结点，此时newHead指向最后一个结点，然后head的下一个结点的next指向head本身，这个相当于把head结点移动到末尾的操作，因为是回溯的操作，所以head的下一个结点总是在上一轮被移动到末尾了，但head之后的next还没有断开，所以可以顺势将head移动到末尾，再把next断开，最后返回newHead即可

```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode next = head.next;
    ListNode newHead = reverseList(next);
    next.next = head;
    head.next = null;
    return newHead;
}
```



---

## 树

## 栈和队列

## 哈希表

## 字符串

## 数组和矩阵

## 图

## 位运算