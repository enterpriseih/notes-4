# 第四章：链表

`链表结构`

```java
public class ListNode {
    public int value;
    public ListNode next;
    
    public ListNode(int value) {
        this.value = value;
    }
}
```

`使用哨兵节点简化链表的添加操作`

```java
// dummy可以免去输入的链表为空的情况
public ListNode append(LIstNode head, int value) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode newNode = new ListNode(value);
    // if (head == null) {
    //     return newNode;
    // }
    
    ListNode node = dummy;
    // ListNode node = head;
    while(node.next != null) {
        node = node.next;
    }
    node.next = newNode;
    // return head;
    return dummy.next;
}
```

`使用哨兵节点简化链表的删除操作`

```java
public ListNode delete(ListNode head, int value) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    
    ListNode node = dummy;
    while (node.next != null) {
        if (node.next.value == value) {
            node.next = node.next.next;
            break;
        }
        node = node.next;
    }
    
    return dummy.next;
}
```



## 4.1 双指针

> 1. 前后双指针：一个指针先提前移动若干步后，再同时移动两个指针
> 2. 快慢双指针：比如一个指针一次两步，一个指针一次一步

## 面试题21：删除倒数第k个结点
### 题目
给你一个链表，请问如何删除链表中的倒数第k个结点？假设链表中结点的总数为n，那么1≤k≤n。**要求只能遍历链表一次。**
例如输入图4.1中（a）的链表，删除倒数第2个结点之后的链表如图4.1中（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0401.png" alt="图4.1">

图4.1：从链表中删除倒数第2个结点。（a）一个包含6个结点的链表。（b）删除倒数第2个结点（值为5的结点）之后的链表。

> - 找到倒数第 k 个节点：先让第一个指针**从dummy**开始移动 k 步，然后让第二个指针**从dummy节点**，两个指针同时移动，当第一个指针到达尾节点的时候，第二个指针刚好指向倒数第 k + 1 个节点（即倒数 k 的左侧那个）。
> - 该题中删除倒数第 k 个，即将倒数 k + 1 节点指向倒数 k - 1
> - dummy可以避免单独处理删除的是head节点的情况

### 参考代码
``` java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;

    ListNode p1 = dummy, p2 = dummy;
    // 前进n步
    for (int i = 0; i < n; i++) {
        p1 = p1.next; 
    }

    while (p1.next != null) {
        p1 = p1.next;
        p2 = p2.next;
    }

    p2.next = p2.next.next;
    return dummy.next;
}
```



## 面试题22：链表中环的入口结点

### 题目
一个链表中包含环，如何找出环的入口结点？从链表的头结点开始沿着next指针进入环的第一个结点为环的入口结点。例如，在图4.3的链表中，环的入口结点是结点3。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0403.png" alt="图4.3">

图4.3：结点3是链表中环的入口结点

> - 快慢双指针找到环中的某个节点
> - 入口节点：指针P1在初始化时指向头节点，先移动环中节点数的步数，然后P2指向头节点，两指针开始以同样的速度移动，直至相遇，相遇的时候就是环的入口节点
> - 下列代码不需要环中节点步数

### 参考代码
``` java
public ListNode detectCycle(ListNode head) {
    ListNode inLoop = getNodeInLoop(head);
    if (inLoop == null) {
        return null;
    }

    ListNode node = head;
    while (node != inLoop) {
        node = node.next;
        inLoop = inLoop.next;
    }

    return node;
}

private ListNode getNodeInLoop(ListNode head) {
    if (head == null || head.next == null) {
        return null;
    }

    ListNode slow = head.next;
    ListNode fast = slow.next;
    while (slow != null && fast != null) {
        if (slow == fast) return slow;
        // 一步
        slow = slow.next;
        // 两步
        fast = fast.next;
        if (fast != null) fast = fast.next;
    }

    return null;
}
```



## 面试题23：两个链表的第一个重合结点

### 题目
输入两个单向链表，请问如何找出它们的第一个重合结点。例如图4.5中的两个链表的第一个重合的结点的值是4。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0405.png" alt="图4.5">

图4.5：两个部分重合的链表，它们的第一个重合的结点的值是4。

> - 法一：将链表1的尾节点指向链表2的头节点，形成一个带有环的链表3，3的环入口节点就是12的第一个重合点
> - 法二：先分别遍历，得到长度，想让长链表的指针先前进到和短链表相同长度的地方

### 参考代码
``` java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    int count1 = countList(headA);
    int count2 = countList(headB);
    int delta = Math.abs(count1 - count2);
    ListNode longer = count1 > count2 ? headA : headB;
    ListNode shorter = count1 > count2 ? headB : headA;
    ListNode node1 = longer;
    // 循环几次就移动几步
    for (int i = 0; i < delta; ++i) {
        node1 = node1.next;
    }

    ListNode node2 = shorter;
    while (node1 != node2) {
        node2 = node2.next;
        node1 = node1.next;
    }

    return node1;
}

private int countList(ListNode head) {
    int count = 0;
    while (head != null) {
        count++;
        head = head.next;
    }

    return count;
}
```



## 4.2 反转链表

## 面试题24：反转链表

### 题目
定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。例如，把图4.8（a）中的链表反转之后得到的链表如图4.8（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0408.png" alt="图4.8">

图4.8：反转一个链表。（a）一个含有5个结点的链表。（b）反转之后的链表。

### 参考代码

``` java
// i --> j --> k
public ListNode reverseList(ListNode head) {
    ListNode pre = null;
    ListNode cur = head;    
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;// 下一个prev即k的j
        cur = next;// 下一个cur即当前的k
    }

    return pre;
}
```



## 面试题25：链表中的数字相加

### 题目
给你两个表示非负整数的单向链表，请问如何实现这两个整数的相加并且把和仍然用单向链表表示？链表中的每个结点表示整数十进制的一位，并且头结点对应整数的最高位数而尾结点对应整数的个位数。例如在图4.10（a）和（b）中的两个链表分别表示整数123和531，它们的和为654，对应的链表尾图4.10的（c）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0410.png" alt="图4.10">

图4.10：链表中数字以及它们的和。（a）表示整数123的链表。（b）表示整数531的链表。（c）表示123与531的和654的链表。

### 参考代码
``` java
public ListNode addTwoNumbers(ListNode head1, ListNode head2) {
    head1 = reverseList(head1);
    head2 = reverseList(head2);
    ListNode reversedHead = addReversed(head1, head2);
    return reverseList(reversedHead);
}

private ListNode addReversed(ListNode head1, ListNode head2) {
    ListNode dummy = new ListNode(0);
    ListNode sumNode = dummy;
    int carry = 0;
    while (head1 != null || head2 != null) {
        int sum = (head1 == null ? 0 : head1.val)
                + (head2 == null ? 0 : head2.val) + carry;
        carry = sum >= 10 ? 1 : 0;
        sum = sum >= 10 ? sum - 10 : sum;        
        ListNode newNode = new ListNode(sum);

        sumNode.next = newNode;            
        sumNode = sumNode.next;

        head1 = head1 == null ? null : head1.next;
        head2 = head2 == null ? null : head2.next;
    }

    if (carry > 0) {
        sumNode.next = new ListNode(carry);
    }

    return dummy.next;
}

private ListNode reverseList(ListNode head) {
    ListNode pre = null;
    ListNode cur = head;
    while(cur != null) {
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}

```



## 面试题26：重排链表

### 题目
给你一个链表，链表中结点的顺序是L0→ L1→ L2→…→ Ln-1→ Ln，请问如何重排链表使得结点的顺序变成L0→ Ln→ L1→ Ln-1→ L2→ Ln-2→…？例如输入图4.12（a）中的链表，重排之后的链表如图4.12（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0412.png" alt="图4.12">

图4.12：重排链表。（a）一个含有6个结点的链表。（b）重排之后的链表。

> - 中间节点：快慢指针，快指针一次两步，慢指针一次一步，快指针到达尾节点的时候，慢指针到达中间节点
> - 把链表拆开
> - 使用dummy的时候，奇数：slow到达的是中间；偶数：slow到达的是前半部分的最后一个节点
> - 不使用dummy的时候，奇数：slow到达的是中间；偶数：slow到达的是后半部分的第一个节点

### 参考代码
``` java
public void reorderList(ListNode head) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;        
    ListNode fast = dummy;
    ListNode slow = dummy;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next;
        // if的fast.next不是while里面的fast.next
        // 并且需要他找到尾节点
        if (fast.next != null) {
            fast = fast.next;
        }            
    }

    ListNode temp = slow.next;
    slow.next = null;// 这样head的slow后面就无了
    link(head, reverseList(temp), dummy);
}

private void link(ListNode node1, ListNode node2, ListNode dummy) {
    ListNode pre = dummy;
    while (node1 != null && node2 != null) {
        ListNode temp = node1.next;

        pre.next = node1;
        node1.next = node2;
        pre = node2;

        node1 = temp;
        node2 = node2.next;
    }
	// 前半段如果是奇数的话，链表最后是第一段的
    if (node1 != null) {
        pre.next = node1;
    }
}

private ListNode reverseList(ListNode head) {
    ListNode pre = null;
    ListNode cur = head;
    while(cur != null) {
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}
```



## 面试题27：回文链表

### 题目
如何判断一个链表是不是回文？要求解法的时间复杂度是O(n)，另外不得使用超过O(1)的辅助空间。如果一个链表是回文，那么链表中结点序列从前往后看和从后往前看是相同的。例如，图4.13中的链表的结点序列从前往后看和从后往前看都是1、2、3、3、2、1，因此这是一个回文链表。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0413.png" alt="图4.13">

图4.13：一个回文链表。

> - slow从head开始，fast从head.next开始的话，奇数总节点：slow到达的是中间；偶数总节点：slow到达的是前半部分最后一个节点

### 参考代码
``` java
public boolean isPalindrome(ListNode head) {
    if (head == null && head.next == null) return true;
    ListNode slow = head;
    ListNode fast = head.next;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next;
        if (fast.next != null) fast = fast.next;
    }
    ListNode secondHalf = slow.next;
    slow.next = null;

    return equals(head, reverseList(secondHalf));
}

private boolean equals(ListNode node1, ListNode node2) {
    while (node1 != null && node2 != null) {
        if (node1.val != node2.val) return false;

        node1 = node1.next;
        node2 = node2.next;
    }
    return true;
}

private ListNode reverseList(ListNode head) {
    ListNode reversedHead = null;
    ListNode pre = null;
    ListNode cur = head;
    while(cur != null) {
        ListNode next = cur.next;
        if (next == null) reversedHead = cur;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return reversedHead;
}
```



## 4.3 双向链表和循环链表

## 面试题28：展平多级双向链表

### 题目
在一个多级双向链表中节点除了有两个指针分别指向前后两个节点之外，还有一个指针指向它的子链表，并且子链表也是一个双向链表，它的节点也有指向子链表的指针。请将这样的多级双向链表展平成普通的双向链表，即所有节点都没有子链表。例如图4.14（a）是一个多级双向链表，它展平之后如图4.14（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0414.png" alt="图2.1">

图4.14：展平多级双向链表。（a）一个多级双向链表。（b）展平之后的双向链表。

### 参考代码
``` java
class Node {
    public int val;
    public Node prev;
    public Node next;
    public Node child;
}

public Node flatten(Node head) {
    flattenGetTail(head);
    return head;
}
// 递归，并且返回的是尾节点
private Node flattenGetTail(Node head) {
    Node node = head;
    Node tail = null;
    while (node != null) {
        Node next = node.next;
        // 如果有子节点
        if (node.child != null) {
            Node child = node.child;
            Node childTail = flattenGetTail(node.child);
			// node.child要置0
            node.child = null;
            
            node.next = child;
            child.prev = node;
            childTail.next = next;
            if (next != null) {
                next.prev = childTail;
            }

            tail = childTail;
        } else {
            // 没有子节点
            tail = node;                
        }

        node = next;
    }

    return tail;
}
```



## 面试题29：排序的循环链表

### 题目
在一个循环链表中节点的值递增排序，请设计一个算法往该循环链表中插入节点，并保证插入节点之后的循环链表仍然是排序的。例如图4.15（a）是一个排序的循环链表，插入一个值为4的节点之后的链表如图4.15（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0415.png" alt="图2.1">

图4.15：往排序的循环链表中插入节点。（a）一个值分别为1、2、3、5、6的循环链表。（b）往链表中插入值为4的节点。

### 参考代码
``` java
/*
	1、插入节点在区间内，即 node.val <= val <= node.next.val
	2、插入节点大于最大值 val>max
	3、插入节点小于最小值 val<min
*/
/*
// Definition for a Node.
class Node {
    public int val;
    public Node next;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _next) {
        val = _val;
        next = _next;
    }
};
*/

public Node insert(Node head, int insertVal) {
    Node node = new Node(insertVal);
    if (head == null) {
        head = node;
        head.next = head;
    } else if (head.next == null) {
        head.next = node;
        node.next = head;
    } else {
        insertCore(head, node);
    }
    return head;
}

private void insertCore(Node head, Node node) {
    Node cur = head;
    Node next = head.next;
    Node biggest = head;
    // 在区间里
    while (!(cur.val <= node.val && next.val >= node.val) && next != head) {
        cur = next;
        next = next.next;
        // if (next == head) break;//脱离死循环
        if (cur.val >= biggest.val) biggest = cur;
    }
    // 在区间里
    if (cur.val <= node.val && next.val >= node.val) {
        cur.next = node;
        node.next = next;
    }else {
        // 3，5，1；插入0的情况
        node.next = biggest.next;
        biggest.next = node;
    }

}
```

