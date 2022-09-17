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
public ListNode removeNthFromEnd(ListNode head, int k) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;

    ListNode p1 = dummy, p2 = dummy;
    // 前进n步
    for (int i = 0; i < k; i++) {
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
一个链表中包含环，如何找出环的入口结点？从链表的头结点开始沿着next指针进入环的第一个结点为环的入口结点。

例如，在图4.3的链表中，环的入口结点是结点3。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0403.png" alt="图4.3">



> - 快慢双指针找到环中的某个节点
> - 入口节点：指针P1在初始化时指向头节点，先移动环中节点数的步数，然后P2指向头节点，两指针开始以同样的速度移动，直至相遇，相遇的时候就是环的入口节点
> - 该代码见书，下方代码不需要求环中节点数，但思路相同

### 参考代码

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202204292233402.png" alt="image-20220429223351605" style="zoom:67%;" />

```
快指针2步，慢指针1步
假设快慢指针相遇时，
慢指针走了 s 步，则快指针走了f = 2s步
且 f = s + nb
=> f = 2nb, s = nb
```

- 如果让指针从链表头部一直向前走并统计步数 k，那么所有**走到链表入口节点时的步数**是：k=a+nb（先走 a 步到入口节点，之后每绕 1 圈环（ b 步）都会再次到入口节点）。
- 而目前，slow 指针走过的步数为 nb 步。因此，只要让 slow 再走 a 步停下来，就可以到环的入口。
- 但是不知道 a 的值，该怎么办？依然是使用双指针法。我们构建一个指针，此指针需要有以下性质：此指针和slow 一起向前走 a 步后，两者在入口节点重合。那么从哪里走到入口节点需要 a 步？答案是链表头部head。

让指针1指向inLoop

指针2指向头指针，同时同速移动，相遇即为入口

``` java
public ListNode detectCycle(ListNode head) {
    ListNode fast = head, slow = head;
    while (true) {
        if (fast == null || fast.next == null) return null;
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }
    fast = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    return fast;
}
```



## 面试题23：两个链表的第一个重合结点

### 题目
输入两个单向链表，请问如何找出它们的第一个重合结点。例如图4.5中的两个链表的第一个重合的结点的值是4。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0405.png" alt="图4.5">

图4.5：两个部分重合的链表，它们的第一个重合的结点的值是4。

> - 法一：将链表1的尾节点指向链表2的头节点，形成一个带有环的链表3，3的环入口节点就是12的第一个重合点
> - 法二：先分别遍历，得到长度，想让长链表的指针先前进到和短链表相同长度的地方
> - 法三：贼妙～～～～～

### 参考代码

#### 方法二

时间`O(m+n)`

空间`O(1)`

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

#### 方法三***

时间`O(m+n)`

空间`O(1)`

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202205061206757.png" alt="image-20220506120605704" style="zoom: 45%;" />

设 headA 长 a，headB 长 b，公共部分长 c

headA 和 headB 到公共节点的长度分别为 `a - c` 和 `b - c`

headA 走完后继续走 B 的，headB 走完后继续走 A 的

两者相遇时，A 走了 `a + (b - c)`，B 走了 `b + (a - c)`，相同的步数

相遇时有两种情况

- 节点是公共节点
- 没有公共节点的话，`A == B` 只会是 `null == null`

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode A = headA, B = headB;
    while (A != B) {
        A = A != null ? A.next : headB;
        B = B != null ? B.next : headA;
    }
    return A;
}
```



## 4.2 反转链表

## 面试题24：反转链表

### 题目
定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。例如，把图4.8（a）中的链表反转之后得到的链表如图4.8（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0408.png" alt="图4.8">

图4.8：反转一个链表。（a）一个含有5个结点的链表。（b）反转之后的链表。

### 参考代码

#### 迭代

``` java
// i --> j --> k
// 一次断开一个，然后再拼上
public ListNode reverseList(ListNode head) {
    ListNode pre = null;
    ListNode cur = head;    
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = pre;
        pre = cur;
        cur = next;
    }
    return pre;
}
```

#### 递归

```java
 /**
  * 以链表1->2->3->4->5举例
  * @param head
  * @return
  */
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) {
        /*
            直到当前节点的下一个节点为空时返回当前节点
            由于5没有下一个节点了，所以此处返回节点5
         */
        return head;
    }
    //递归传入下一个节点，目的是为了到达最后一个节点
    ListNode newHead = reverseList(head.next);
    /*
        第一轮出栈，head为5，head.next为空，返回5
        第二轮出栈，head为4，head.next为5，执行head.next.next=head也就是5.next=4，
                  把当前节点的子节点的子节点指向当前节点
                  此时链表为1->2->3->4<->5，由于4与5互相指向，所以此处要断开4.next=null
                  此时链表为1->2->3->4<-5
                  返回节点5
        第三轮出栈，head为3，head.next为4，执行head.next.next=head也就是4.next=3，
                  此时链表为1->2->3<->4<-5，由于3与4互相指向，所以此处要断开3.next=null
                  此时链表为1->2->3<-4<-5
                  返回节点5
        第四轮出栈，head为2，head.next为3，执行head.next.next=head也就是3.next=2，
                  此时链表为1->2<->3<-4<-5，由于2与3互相指向，所以此处要断开2.next=null
                  此时链表为1->2<-3<-4<-5
                  返回节点5
        第五轮出栈，head为1，head.next为2，执行head.next.next=head也就是2.next=1，
                  此时链表为1<->2<-3<-4<-5，由于1与2互相指向，所以此处要断开1.next=null
                  此时链表为1<-2<-3<-4<-5
                  返回节点5
        出栈完成，最终头节点5->4->3->2->1
     */
    head.next.next = head;
    head.next = null;
    return newHead;
}
```



## [翻转链表2](https://leetcode.cn/problems/reverse-linked-list-ii/)

### 题目

翻转区间[left, right]的链表

```
输入：head = [1,2,3,4,5], left = 2, right = 4
输出：[1,4,3,2,5]
```



### 题解

![image.png](https://pic.leetcode-cn.com/1615105296-bmiPxl-image.png)

- curr：指向待反转区域的**第一个节点 left**；

- next：永远指向 curr 的下一个节点，循环过程中，curr 变化以后 next 会变化；

- pre：永远指向待反转区域的第一个节点 **left 的前一个节点**，在循环过程中不变。

```java
public ListNode reverseBetween(ListNode head, int left, int right) {
    ListNode dummyNode = new ListNode(-1);
    dummyNode.next = head;
    ListNode pre = dummyNode;
    for (int i = 0; i < left - 1; i++) {
        pre = pre.next;
    }
    ListNode cur = pre.next;
    ListNode next;
    for (int i = 0; i < right - left; i++) {
        next = cur.next;
        cur.next = next.next;
        next.next = pre.next;
        pre.next = next;
    }
    return dummyNode.next;
}

```



## [K个翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

用栈，把 k 个数压入栈中，然后弹出来的顺序就是翻转的！

这里要注意几个问题：

第一，剩下的链表个数够不够 k 个（因为不够 k 个不用翻转）；

第二，已经翻转的部分要与剩下链表连接起来。

```java
public ListNode reverseKGroup(ListNode head, int k) {
    Stack<ListNode> st = new Stack<>();
    ListNode dummy = new ListNode(0);
    // pre是翻转好的前一组的尾节点
    ListNode pre = dummy;
    while (true) {
        int count = 0;
        // head指向的是还没翻转的k个一组的头节点
        ListNode tmp = head;
        while (tmp != null && count != k) {
            st.add(tmp);
            tmp = tmp.next;
            count++;
        } 
        // 此时tmp是下一组的头节点
        if (count != k) {
            pre.next = head;
            break;
        }
        while (!st.isEmpty()) {
            pre.next = st.pop();
            pre = pre.next;
        }
        // head变成下一组的头节点
        head = tmp;
        // pre.next = tmp; 
    }
    return dummy.next;
}
```

方法二

```java
public ListNode reverse(ListNode a,ListNode b){
    ListNode pre,cur,nxt;
    pre=null;cur=a;
    while(cur!=b){
        nxt=cur.next;
        cur.next=pre;
        pre=cur;
        cur=nxt;
    }
    return pre;
}
public ListNode reverseKGroup(ListNode head, int k) {
    if(head==null) return null;
    ListNode a,b;
    a=b=head;
    for(int i=0;i<k;i++){
        if(b==null) return head;
        b=b.next;
    }
    // 此时b是下一组的头节点
    ListNode newhead=reverse(a,b);
    // a已经变成了ab区间链表的尾部
    a.next=reverseKGroup(b,k);
    return newhead;

}
```





## 面试题25：链表中的数字相加

### 题目
给你两个表示非负整数的单向链表，请问如何实现这两个整数的相加并且把和仍然用单向链表表示？链表中的每个结点表示整数十进制的一位，并且头结点对应整数的最高位数而尾结点对应整数的个位数。例如在图4.10（a）和（b）中的两个链表分别表示整数123和531，它们的和为654，对应的链表尾图4.10的（c）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0410.png" alt="图4.10">

图4.10：链表中数字以及它们的和。（a）表示整数123的链表。（b）表示整数531的链表。（c）表示123与531的和654的链表。

### 参考代码

#### 三次反转

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
        carry = sum / 10;
        sum %= 10;         
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



#### 栈

```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        Deque<Integer> stack1 = new ArrayDeque<Integer>();
        Deque<Integer> stack2 = new ArrayDeque<Integer>();
        while (l1 != null) {
            stack1.push(l1.val);
            l1 = l1.next;
        }
        while (l2 != null) {
            stack2.push(l2.val);
            l2 = l2.next;
        }
        int carry = 0;
        ListNode ans = null;
        while (!stack1.isEmpty() || !stack2.isEmpty() || carry != 0) {
            int a = stack1.isEmpty() ? 0 : stack1.pop();
            int b = stack2.isEmpty() ? 0 : stack2.pop();
            int cur = a + b + carry;
            carry = cur / 10;
            cur %= 10;
            ListNode curnode = new ListNode(cur);
            curnode.next = ans;
            ans = curnode;
        }
        return ans;
    }
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
如何判断一个链表是不是回文？要求解法的时间复杂度是O(n)，另外不得使用超过O(1)的辅助空间。如果一个链表是回文，那么链表中结点序列从前往后看和从后往前看是相同的。

例如，图4.13中的链表的结点序列从前往后看和从后往前看都是1、2、3、3、2、1，因此这是一个回文链表。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0413.png" alt="图4.13">



> - slow从head开始，fast从head.next开始的话，奇数总节点：slow到达的是中间；偶数总节点：slow到达的是前半部分最后一个节点

### 参考代码

#### 法一

``` java
// 先分割，再反转
public boolean isPalindrome(ListNode head) {
    if (head == null && head.next == null) return true;
    ListNode slow = head;
    ListNode fast = head.next;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
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

#### 法二

```java
// 一边快慢双指针一边反转
public boolean isPalindrome(ListNode head) {
    ListNode fast = head;
    ListNode slow = head;
    ListNode pre = null;
    ListNode prepre = null;
    while (fast != null && fast.next != null) {
        pre = slow;
        slow = slow.next;
        fast = fast.next.next;
        pre.next = prepre;
        prepre = pre;
    }
	// 若是奇数
    if (fast != null) {
        slow = slow.next;
    }
    return equals(pre, slow);
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

