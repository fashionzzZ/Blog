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



---

## 树

## 栈和队列

## 哈希表

## 字符串

## 数组和矩阵

## 图

## 位运算