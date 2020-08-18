## LeetCode 精选 TOP 面试题

### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

```java
/**
 * 题解：
 *
 * 弄一个HashMap，遍历数组中的元素，每遍历一个元素，判断(target - 当前元素)的值是否存在HashMap中
 * - 如果存在，返回HashMap中元素的下标和当前元素的下标
 * - 如果不存在，将`当前元素`作为Key，下标作为Value存入到HashMap中
 */
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement)) {
                return new int[]{map.get(complement), i};
            } else {
                map.put(nums[i], i);
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

### [2. 两数相加]()

```java
/**
 * 题解：
 * 
 * 两个链表从头部遍历节点，一次相加，记录好进位，返回新生成的链表。
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode node = new ListNode(0);
        ListNode cur = node;
        ListNode c1 = l1;
        ListNode c2 = l2;
        int ca = 0;
        while (c1 != null || c2 != null) {
            int n1 = c1 != null ? c1.val : 0;
            int n2 = c2 != null ? c2.val : 0;
            int n = n1 + n2 + ca;
            ca = n / 10;
            cur.next = new ListNode(n % 10);
            cur = cur.next;
            c1 = c1 != null ? c1.next : null;
            c2 = c2 != null ? c2.next : null;
        }
        if (ca > 0) {
            cur.next = new ListNode(ca);
        }
        return node.next;
    }
}
```

### [3. 无重复字符的最长字串]()

```java
/**
 * 题解：
 *
 * 声明一个128长度的数组，存放每个字符对应的位置
 * 声明一个指针 pre，记录i-1位置结尾的情况下，往左推，推不动的位置是谁
 * 当前i位置的字符情况下，往左推，推不动的是 pre和i位置字符上一次的位置 的最大值
 */
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if (s == null || s.equals("")) {
            return 0;
        }
        int[] map = new int[128];
        for (int i = 0; i < map.length; i++) {
            map[i] = -1;
        }
        char[] chars = s.toCharArray();
        int len = 0;
        int pre = -1;
        int cur = 0;
        for (int i = 0; i < chars.length; i++) {
            pre = Math.max(pre, map[chars[i]]);
            cur = i - pre;
            len = Math.max(len, cur);
            map[chars[i]] = i;
        }
        return len;
    }
}
```

