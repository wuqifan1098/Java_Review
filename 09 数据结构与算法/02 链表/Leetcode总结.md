# 面试题

## 1. 给出一个链表和一个数k，链表前k个节点进行翻转（字节跳动）

```java
public ListNode reverseKGroup(ListNode head, int k){
    ListNode dummy = new ListNode(0); //哑节点
    dummy.next = head;
    
}

```

## 2. 两个链表相加求和 （字节跳动）

```java
public class addList(ListNode l1, ListNode l2){
	ListNode dummy = new ListNode(0);//当前节点
	ListNode p = l1, q = l2, cur = dummy;//当前节点为

}
```



# [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

给出两个 **非空** 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 **逆序** 的方式存储的，并且它们的每个节点只能存储 **一位** 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

**示例：**

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

**解析：**

就像你在纸上计算两个数字的和那样，我们首先从最低有效位也就是列表 l1 和 l2 的表头开始相加。由于每位数字都应当处于 0…9 的范围内，我们计算两个数字的和时可能会出现 “溢出”。例如，5 + 7 = 12。在这种情况下，我们会将当前位的数值设置为 2，并将进位 carry = 1 带入下一次迭代。进位 carry 必定是 0 或 1，这是因为两个数字相加（考虑到进位）可能出现的最大和为 9 + 9 + 1 = 19。

**伪代码：**

- 将**当前结点初始化为返回列表的哑结点**。
- 将进位 carry初始化为 00。
- 将 p 和 q 分别初始化为列表 l1 和 l2 的头部。
- **遍历列表 l1 和 l2**直至到达它们的尾端。
  - 将 x 设为结点 p 的值。如果 p 已经到达 l1 的末尾，则将其值设置为 0。
  - 将 y 设为结点 q 的值。如果 q 已经到达 l2 的末尾，则将其值设置为 0。
  - 设定 sum = x + y + carry。
  - 更新进位的值，carry = sum / 10。
  - 创建一个数值为 (sum mod 10) 的新结点，并将其设置为当前结点的下一个结点，然后将当前结点前进到下一个结点。
  - 同时，将 p 和 q 前进到下一个结点。
  - 检查 carry = 1 是否成立，如果成立，则向返回列表追加一个含有数字 1 的新结点。
  - 返回哑结点的下一个结点。

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}
```

复杂度分析

- 时间复杂度：O(max(m, n))，假设 m 和 n 分别表示 l1 和 l2 的长度，上面的算法最多重复 max(m, n) 次。

- 空间复杂度：O(max(m, n))， 新列表的长度最多为 max(m,n) + 1。

# [19. 删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

给定一个链表，删除链表的倒数第 *n* 个节点，并且返回链表的头结点。

**示例：**

```
给定一个链表: 1->2->3->4->5, 和 n = 2.

当删除了倒数第二个节点后，链表变为 1->2->3->5.
```

**思路**：

一次遍历： 使用两个指针，第一个指针fast先开始移动n步，然后第二个指针slow从头开始移动，此时第一个指针也一起移动，这样就可以使得在移动过程中两个指针保持n步的距离，直到fast到达最后一个结点。此时slow将指向从最后一个结点数起的第 n 个结点，我们重新链接slow所引用的结点的 next 指针指向该结点的下下个结点。

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode fast = head;
        while (n-- > 0) {
            fast = fast.next;
        }
        if (fast == null) return head.next;	//防止极端情况
        ListNode slow = head;
        while (fast.next != null) {
            fast = fast.next;
            slow = slow.next;
        }
        slow.next = slow.next.next;
        return head;
        
    }
}
```

# [92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

反转从位置 *m* 到 *n* 的链表。请使用一趟扫描完成反转。

**说明:**
1 ≤ *m* ≤ *n* ≤ 链表长度。

**示例:**

```
输入: 1->2->3->4->5->NULL, m = 2, n = 4
输出: 1->4->3->2->5->NULL
```

实现思路 ：以1->2->3->4->5, m = 2, n=4 为例:

- 定位到要反转部分的头节点 2，head = 2；前驱结点 1，pre = 1；
- 当前节点的下一个节点3调整为前驱节点的下一个节点 1->3->2->4->5,
- 当前结点仍为2， 前驱结点依然是1，重复上一步操作。。。
- 1->4->3->2->5.

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
         ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode pre = dummy;
        for(int i = 1; i < m; i++){
            pre = pre.next;
        }
        head = pre.next;
        for(int i = m; i < n; i++){
            ListNode nex = head.next;
            head.next = nex.next;
            nex.next = pre.next;
            pre.next = nex;
        }
        return dummy.next;
    }
}
```

# [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

反转一个单链表。

**示例:**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

实现思路：

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while(cur != null){
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
}
```

