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
class Solution {
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
class Solution {
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
}
```

递归：递归解法的思路是，不断的进入递归函数，直到head指向倒数第二个节点，因为head指向空或者是最后一个结点都直接返回了，newHead则指向对head的下一个结点调用递归函数返回的头结点，此时newHead指向最后一个结点，然后head的下一个结点的next指向head本身，这个相当于把head结点移动到末尾的操作，因为是回溯的操作，所以head的下一个结点总是在上一轮被移动到末尾了，但head之后的next还没有断开，所以可以顺势将head移动到末尾，再把next断开，最后返回newHead即可。

```java
class Solution {
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
}
```

### [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

这道题我才用的是递归写法，当某个链表为空了，就返回另一个。然后比较当前两个节点值大小，如果 l1 的小，那么对于 l1 的下一个节点和 l2 调用递归函数，将返回值赋值给 l1.next，然后返回 l1；否则就对于 l2 的下一个节点和 l1 调用递归函数，将返回值赋值给 l2.next，然后返回 l2。

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        if (l2 == null) {
            return l1;
        }
        if (l1.val <= l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```

### [83. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

这道题让我们移除给定有序链表的重复项，那么可以遍历这个链表，每个结点和其后面的结点比较，如果结点值相同了，只要将前面结点的 next 指针跳过紧挨着的相同值的结点，指向后面一个结点。这样遍历下来，所有重复的结点都会被跳过，留下的链表就是没有重复项的了。

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode cur = head;
        while (cur != null && cur.next != null) {
            if (cur.val == cur.next.val) {
                cur.next = cur.next.next;
            } else {
                cur = cur.next;
            }
        }
        return head;
    }
}
```

递归写法：

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null || head.next == null) return head;
        head.next = deleteDuplicates(head.next);
        return head.val == head.next.val ? head.next : head;
    }
}
```

### [19. 删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

这道题我的第一想法是用递归来做，并且代码一次成功。然后看了其它题解都是用的双指针搞定的，嗯，也行吧。

递归思路：声明一个全局变量记录当前节点是链表中的倒数第几位，如何等于n，说明需要删除（或者说跳过）当前节点，只需要返回当前节点的下一个节点就行了。

```java
class Solution {
    private int num = 0;

    public ListNode removeNthFromEnd(ListNode head, int n) {
        if (head == null) {
            return head;
        }
        head.next = removeNthFromEnd(head.next, n);
        num++;
        if (num == n) {
            return head.next;
        }
        return head;
    }
}
```

### [24. 两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

迭代：迭代方式其实和反转链表的迭代方式类似，这里就不做描述了，画图会比较方便理解。

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode node = new ListNode(-1);
        node.next = head;
        ListNode pre = node;
        while (pre.next != null && pre.next.next != null) {
            ListNode l1 = pre.next;
            ListNode l2 = pre.next.next;
            l1.next = l2.next;
            l2.next = l1;
            pre.next = l2;
            pre = l1;
        }
        return node.next;
    }
}
```

递归：递归的写法就更简洁了，实际上利用了回溯的思想，递归遍历到链表末尾，然后先交换末尾两个，然后依次往前交换。

`一下代码参考LeetCode官方题解，看注释很好理解`

```java
class Solution {
    public ListNode swapPairs(ListNode head) {

        // If the list has no node or has only one node left.
        if ((head == null) || (head.next == null)) {
            return head;
        }

        // Nodes to be swapped
        ListNode firstNode = head;
        ListNode secondNode = head.next;

        // Swapping
        firstNode.next  = swapPairs(secondNode.next);
        secondNode.next = firstNode;

        // Now the head is the second node
        return secondNode;
    }
}
```

### [445. 两数相加 II](https://leetcode-cn.com/problems/add-two-numbers-ii/)

这道题题目不让修改原链表，我们可以利用栈来保存所有的元素，然后利用栈的后进先出的特点就可以从后往前取数字了，其它的和`两数相加I`相似。这里有个`头插法`的概念需要注意一下。

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        Stack<Integer> stack1 = buildStack(l1);
        Stack<Integer> stack2 = buildStack(l2);
        ListNode head = new ListNode(-1);
        int carry = 0;
        while (!stack1.isEmpty() || !stack2.isEmpty()) {
            int x = stack1.isEmpty() ? 0 : stack1.pop();
            int y = stack2.isEmpty() ? 0 : stack2.pop();
            int sum = x + y + carry;
            carry = sum / 10;
            ListNode node = new ListNode(sum % 10);
            node.next = head.next;
            head.next = node;
        }
        if (carry == 1) {
            ListNode node = new ListNode(1);
            node.next = head.next;
            head.next = node;
        }
        return head.next;
    }

    public Stack<Integer> buildStack(ListNode head) {
        Stack<Integer> stack = new Stack<>();
        while (head != null) {
            stack.push(head.val);
            head = head.next;
        }
        return stack;
    }
}
```

### [234. 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)

题目要求O(n) 时间复杂度和 O(1) 空间复杂度

找到链表的中点，反转后半部分链表，顺序比较前半部分和后半部分的节点。

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        if (head == null) {
            return true;
        }
        ListNode firstHalfEnd = endOfFirstHalf(head);
        ListNode secondHalfStart = reverseList(firstHalfEnd.next);
        ListNode l1 = head, l2 = secondHalfStart;
        while (l2 != null) {
            if (l1.val != l2.val) {
                return false;
            }
            l1 = l1.next;
            l2 = l2.next;
        }
        return true;
    }

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

    public ListNode endOfFirstHalf(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;
        while (fast.next != null && fast.next.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

### [725. 分割链表](https://leetcode-cn.com/problems/split-linked-list-in-parts/solution/fen-ge-lian-biao-by-leetcode-2/)

算法：`参考LeetCode官方题解`

如果链表有 N 个结点，则分隔的链表中每个部分中都有 n/k 个结点，且前 N%k 部分有一个额外的结点。我们可以用一个简单的循环来计算 N。
现在对于每个部分，我们已经计算出该部分有多少个节点：width + (i < remainder ? 1 : 0)。我们创建一个新列表并将该部分写入该列表。

```java
class Solution {
    public ListNode[] splitListToParts(ListNode root, int k) {
        ListNode[] nodes = new ListNode[k];
        int n = listLength(root);
        int width = n / k, rem = n % k;
        ListNode cur = root;
        for (int i = 0; i < k; i++) {
            ListNode head = new ListNode(0), nCur = head;
            for (int j = 0; j < width + (i < rem ? 1 : 0); j++) {
                nCur.next = new ListNode(cur.val);
                nCur = nCur.next;
                if (cur != null) {
                    cur = cur.next;
                }
            }
            nodes[i] = head.next;
        }
        return nodes;
    }

    public int listLength(ListNode head) {
        int l = 0;
        while (head != null) {
            l++;
            head = head.next;
        }
        return l;
    }
}
```

### [328. 奇偶链表](https://leetcode-cn.com/problems/odd-even-linked-list/)

我们可以使用两个指针来做，pre指向奇节点，cur指向偶节点，然后把偶节点cur后面的那个奇节点提前到pre的后面，然后pre和cur各自前进一步，此时cur又指向偶节点，pre指向当前奇节点的末尾，以此类推直至把所有的偶节点都提前了即可。

```java
class Solution {
    public ListNode oddEvenList(ListNode head) {
        if (head == null) {
            return head;
        }
        ListNode pre = head;
        ListNode cur = head.next;
        while (cur != null && cur.next != null) {
            ListNode tmp = pre.next;
            pre.next = cur.next;
            cur.next = pre.next.next;
            pre.next.next = tmp;
            pre = pre.next;
            cur = cur.next;
        }
        return head;
    }
}
```

---

## 树

## 栈和队列

## 哈希表

## 字符串

### [242. 有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/)

这不算一道难题，核心点就在于使用哈希表映射，我们还是用一个数组来代替哈希表。我们先判断两个字符串长度是否相同，不相同直接返回false。若相同，则初始化26个字母哈希表，遍历字符串s和t，s 负责在对应位置增加，t 负责在对应位置减少，如果哈希表的值都为 0，则二者是字母异位词。

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }
        int[] map = new int[26];
        for (int i = 0; i < s.length(); i++) {
            map[s.charAt(i) - 'a']++;
            map[t.charAt(i) - 'a']--;
        }
        for (int i : map) {
            if (i != 0) {
                return false;
            }
        }
        return true;
    }
}
```

### [409. 最长回文串](https://leetcode-cn.com/problems/longest-palindrome/)

使用长度为 128 的整型数组来统计每个字符出现的个数，每个字符有偶数个可以用来构成回文字符串。因为回文字符串最中间的那个字符可以单独出现，所以如果有单独的字符就把它放到最中间。

```java
class Solution {
    public int longestPalindrome(String s) {
        int[] map = new int[128];
        for (char c : s.toCharArray()) {
            map[c]++;
        }
        int palindrome = 0;
        for (int count : map) {
            palindrome += (count / 2) * 2;
        }
        if (palindrome < s.length()) {
            palindrome++;
        }
        return palindrome;
    }
}
```

### [205. 同构字符串](https://leetcode-cn.com/problems/isomorphic-strings/)

记录一个字符上次出现的位置，如果两个字符串中的字符上次出现的位置一样，那么就属于同构。

```java
class Solution {
    public boolean isIsomorphic(String s, String t) {
        int[] sMap = new int[128];
        int[] tMap = new int[128];
        char[] sChars = s.toCharArray();
        char[] tChars = t.toCharArray();
        for (int i = 0; i < s.length(); i++) {
            char sc = sChars[i], tc = tChars[i];
            if (sMap[sc] != tMap[tc]) {
                return false;
            }
            sMap[sc] = i + 1;
            tMap[tc] = i + 1;
        }
        return true;
    }
}
```

### [647. 回文字串](https://leetcode-cn.com/problems/palindromic-substrings/)

枚举每一个可能的回文中心，然后用两个指针分别向左右两边拓展，当两个指针指向的元素相同的时候就拓展，否则停止拓展。

```java
class Solution {
    private int count = 0;

    public int countSubstrings(String s) {
        char[] chars = s.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            extend(chars, i, i);     // 奇数情况
            extend(chars, i, i + 1); // 偶数情况
        }
        return count;
    }

    public void extend(char[] chars, int start, int end) {
        while (start >= 0 && end < chars.length && chars[start] == chars[end]) {
            start--;
            end++;
            count++;
        }
    }
}
```



## 数组和矩阵

## 图

## 位运算