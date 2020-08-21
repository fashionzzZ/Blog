# LeetCode - 算法

* [哈希](#id=hash)
* [双指针](#id=双指针)

## Hash

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

### [387. 字符串中的第一个唯一字符](https://leetcode-cn.com/problems/first-unique-character-in-a-string/)

```java
/**
 * 题解：
 *
 * 声明一个26长度的int数组，存放26个英文字母出现的个数
 * 遍历两遍原始字符串
 * 第一遍统计字符串中每个字母的个数存放在数组中，下标是（当前字母 - 'a'）
 * 第二遍根据每个字母的下标（当前字母 - 'a'），返回第一个元素为1的下标
 */
class Solution {
    public int firstUniqChar(String s) {
        int[] count = new int[26];
        int n = s.length();
        for (int i = 0; i < n; i++) {
            count[s.charAt(i) - 'a']++;
        }
        for (int i = 0; i < n; i++) {
            if (count[s.charAt(i) - 'a'] == 1) {
                return i;
            }
        }
        return -1;
    }
}
```


---

## 双指针

`双指针主要用于遍历数组，两个指针指向不同的元素，从而协同完成任务。`

### [26. 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

```java
/**
 * 题解：
 *
 * 数组完成排序后，我们可以放置两个指针i和j，其中i是慢指针，而j是快指针。只要 nums[i] = nums[j] ，我们就增加j以跳过重复项。
 * 当我们遇到 nums[i] != nums[j] 时，跳过重复项的运行已经结束，因此我们必须把它 nums[j] 的值复制到 nums[i + 1] 。
 * 然后递增i，接着我们将再次重复相同的过程，直到j到达数组的末尾为止。
 */
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) {
            return 0;
        }
        int i = 0;
        for (int j = 1; j < nums.length; j++) {
            if (nums[i] != nums[j]) {
                i++;
                nums[i] = nums[j];
            }
        }
        return i + 1;
    }
}
```

### [167. 两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/)

给定一个已按照升序排列的有序数组，找到两个数使得它们相加之和等于目标数。
函数应该返回这两个下标值 index1 和 index2，其中 index1 必须小于 index2。

#### 说明

- 返回的下标值（index1 和 index2）不是从零开始的。
- 你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。

#### 示例

> **输入: ** numbers = [2, 7, 11, 15], target = 9
> **输出:** [1,2]
> **解释:** 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。

#### 思路

使用双指针，一个指针(index1)指向值较小的元素，一个指针(index2)指向值较大的元素。指向较小元素的指针从头向尾遍历，指向较大元素的指针从尾向头遍历。

> - 如果`sum == target`，直接return结果。
> - 如果`sum < target`，index1++。
> - 如果`sum > target`，index2--。

#### 代码

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        if (numbers == null)
            return null;
        int index1 = 0, index2 = numbers.length - 1;
        while (index1 < index2) {
            int sum = numbers[index1] + numbers[index2];
            if (sum == target) {
                return new int[] { index1 + 1, index2 + 1 };
            } else if (sum < target) {
                index1++;
            } else {
                index2--;
            }
        }
        return null;
    }
}
```

---

### [633. 平方数之和](https://leetcode-cn.com/problems/sum-of-square-numbers/)

给定一个非负整数 `c` ，你要判断是否存在两个整数 `a` 和 `b`，使得`a2 + b2 = c`。

#### 示例1:

> **输入:** 5
> **输出: ** True
> **解释:** 1 * 1 + 2 * 2 = 5

#### 示例2:

> **输入:** 3
> **输出:** False

#### 思路

本题和 167. Two Sum II - Input array is sorted 类似，只有一个明显区别：一个是和为 target，一个是平方和为 target。本题同样可以使用双指针得到两个数，使其平方和为 target。需要注意右指针的取值问题，可以将其取为sqrt(target)。

#### 代码

```java
class Solution {
    public boolean judgeSquareSum(int c) {
        if (c < 0)
            return false;
        int i = 0, j = (int) Math.sqrt(c);
        while (i <= j) {
            int sum = i * i + j * j;
            if (sum == c) {
                return true;
            } else if (sum > c) {
                j--;
            } else {
                i++;
            }
        }
        return false;
    }
}
```

---

### [345. 反转字符串中的元音字母](https://leetcode-cn.com/problems/reverse-vowels-of-a-string/)

编写一个函数，以字符串作为输入，反转该字符串中的元音字母。

#### 示例1

> **输入:** "hello"
> **输出:** "holle"

#### 示例2

> **输入:** "leetcode"
> **输出:** "leotcede"

#### 说明

元音字母不包含字母"y"。

#### 思路

使用双指针，一个指针从头向尾遍历，一个指针从尾到头遍历，当两个指针都遍历到元音字符时，交换这两个元音字符。

#### 代码

```java
class Solution {
    public String reverseVowels(String s) {
        char[] chars = s.toCharArray();
        int i = 0, j = chars.length - 1;
        while (i < j) {
            if (isVowel(chars[i]) && isVowel(chars[j])) {
                char t = chars[i];
                chars[i++] = chars[j];
                chars[j--] = t;
            } else if (!isVowel(chars[i])) {
                i++;
            } else {
                j--;
            }
        }
        return new String(chars);
    }

    private boolean isVowel(char c) {
        return 'a' == c || 'A' == c ||
            'e' == c || 'E' == c || 
            'i' == c || 'I' == c || 
            'o' == c || 'O' == c ||
            'u' == c || 'U' == c;
    }
}
```

---

### [680. 验证回文字符串 Ⅱ](https://leetcode-cn.com/problems/valid-palindrome-ii/)

给定一个非空字符串 `s`，**最多**删除一个字符。判断是否能成为回文字符串。

#### 示例1

> **输入:** "aba"
> **输出:** True

#### 示例2

> **输入:** "abca"
> **输出:** True
> **解释:** 你可以删除c字符。

#### 注意

1. 字符串只包含从 a-z 的小写字母。字符串的最大长度是50000。

#### 思路

使用双指针可以很容易判断一个字符串是否是回文字符串：令一个指针从左到右遍历，一个指针从右到左遍历，这两个指针同时移动一个位置，每次都判断两个指针指向的字符是否相同，如果都相同，字符串才是具有左右对称性质的回文字符串。

本题的关键是处理删除一个字符。在使用双指针遍历字符串时，如果出现两个指针指向的字符不相等的情况，我们就试着删除一个字符，再判断删除完之后的字符串是否是回文字符串。

在判断是否为回文字符串时，我们不需要判断整个字符串，因为左指针左边和右指针右边的字符之前已经判断过具有对称性质，所以只需要判断中间的子字符串即可。

在试着删除字符时，我们既可以删除左指针指向的字符，也可以删除右指针指向的字符。

#### 代码

```java
class Solution {
    public boolean validPalindrome(String s) {
        for (int i = 0, j = s.length() - 1; i < j; i++, j--) {
            if (s.charAt(i) != s.charAt(j)) {
                return isPalindrome(s, i, j - 1) || isPalindrome(s, i + 1, j);
            }
        }
        return true;
    }

    private boolean isPalindrome(String s, int i, int j) {
        while (i < j) {
            if (s.charAt(i++) != s.charAt(j--)) {
                return false;
            }
        }
        return true;
    }
}
```

#### 结果

---

### [88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

```java
/**
 * 题解：
 *
 * 两个指针，左指针指向nums1的第一个元素；右指针指向nums2的第一个元素。
 * 声明一个新数组，长度为上面两个数组长度的和，并声明一个指针指向0号元素。
 * 循环判断，两个数组中的元素，哪个小就拷贝到新数组，新数组和元素小的数组的指针向右移动一位
 * 其中一个数组已经遍历完成，拷贝另一个数组剩下的元素到新数组
 * 将新数组中的元素拷贝到nums1中
 */
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = 0, l = 0, r = 0;
        int[] newArr = new int[m + n];
        while (l < m && r < n) {
            newArr[i++] = nums1[l] <= nums2[r] ? nums1[l++] : nums2[r++];
        }
        while (l < m) {
            newArr[i++] = nums1[l++];
        }
        while (r < n) {
            newArr[i++] = nums2[r++];
        }
        for (int k = 0; k < newArr.length; k++) {
            nums1[k] = newArr[k];
        }
    }
}
```

---

### [141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

给定一个链表，判断链表中是否有环。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

#### 思路

使用双指针，一个指针每次移动一个节点，一个指针每次移动两个节点，如果存在环，那么这两个指针一定会相遇。

#### 代码

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null) {
            return false;
        }
        ListNode l1 = head, l2 = head.next;
        while (l1 != null && l2 != null && l2.next != null) {
            if (l1 == l2) {
                return true;
            }
            l1 = l1.next;
            l2 = l2.next.next;
        }
        return false;
    }
}
```

---

### [通过删除字母匹配到字典里最长单词](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/)

给定一个字符串和一个字符串字典，找到字典里面最长的字符串，该字符串可以通过删除给定字符串的某些字符来得到。如果答案不止一个，返回长度最长且字典顺序最小的字符串。如果答案不存在，则返回空字符串。

#### 示例1

> **输入:**
> s = "abpcplea", d = ["ale","apple","monkey","plea"]
>
> **输出:** 
> "apple"

#### 示例2

> **输入:**
> s = "abpcplea", d = ["a","b","c"]
>
> **输出:** 
> "a"

#### 说明

1. 所有输入的字符串只包含小写字母。
2. 字典的大小不会超过 1000。
3. 所有输入的字符串长度不会超过 1000。

#### 思路

通过删除字符串 s 中的一个字符能得到字符串 t，可以认为 t 是 s 的子序列，我们可以使用双指针来判断一个字符串是否为另一个字符串的子序列。

#### 代码

```java
class Solution {
    public String findLongestWord(String s, List<String> d) {
        String longest = "";
        for (String target : d) {
            int l1 = longest.length(), l2 = target.length();
            if (l1 > l2 || (l1 == l2 && longest.compareTo(target) < 0)) {
                continue;
            }
            if (isChildStr(s, target)) {
                longest = target;
            }
        }
        return longest;
    }

    public boolean isChildStr(String father, String son) {
        int i = 0, j = 0;
        while (i < father.length() && j < son.length()) {
            if (father.charAt(i) == son.charAt(j)) {
                j++;
            }
            i++;
        }
        return j == son.length();
    }
}
```

---


