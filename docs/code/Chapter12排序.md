# 第十二章：排序

排序[leetcode](https://leetcode-cn.com/problems/sort-an-array/)

## 面试题74：合并区间

### 题目

输入一个区间的集合，请将重叠的区间合并。每个区间用两个数字比较，分别表示区间的起始和结束位置。例如，输入区间[[1, 3], [4, 5], [8, 10], [2, 6], [9, 12], [15, 18]]，合并重叠的区间之后得到[[1, 6], [8, 12], [15, 18]]。

### 参考代码

``` java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (i1, i2) -> i1[0] - i2[0]);

    List<int[]> merged = new LinkedList<>();
    int i = 0;
    while (i < intervals.length) {
        int[] temp = new int[] {intervals[i][0], intervals[i][1]};
        int j = i + 1;
        while (j < intervals.length && intervals[j][0] <= temp[1]) {
            temp[1] = Math.max(temp[1], intervals[j][1]);
            j++;
        }

        merged.add(temp);
        i = j;
    }

    int[][] result = new int[merged.size()][];
    return merged.toArray(result);
}
```

## 12.1 计数排序O(nlogn)

数组长度n，数字范围k（最大值和最小值的差值），时间复杂度`O(n+k)`

适合k远小于n的场景，比如员工的年龄排序

出现数字范围的时候使用

```java
public int[] sortArray (int[] nums) {
    int max = Integer.MIN_VALUE;
    int min = Integer.MAX_VALUE;
    for(int num : nums) {
        max = Math.max(max, num);
        min = Math.min(min, num);
    }
    // count记录数组中每个数出现的次数
    int counts = new int[max - min + 1];
    for(int num : nums) {
        count[num - min]++;
    }
    
    int i = 0;
    for (int num = min; num <= max; num++) {
        // 如果有这个数
        while(counts[num - min] > 0) {
            nums[i++] = num;
            counts[num - min]--;
        }
    }
    2
    return nums;
}
```



## 面试题75：数组相对排序

### 题目

输入两个数组arr1和arr2，其中arr2里的每个数字都唯一并且也都是arr1中的数字。请将arr1中的数字按照arr2中数字的相对顺序排序。

如果arr1中的数字在arr2中没有出现，那么将这些数字按递增的顺序排在后面。假设数组中的所有数字都在0到1000的范围内。

例如，输入的数组arr1和arr2分别是[2, 3, 3, 7, 3, 9, 2, 1, 7, 2]和[3, 2, 1]，则arr1排序之后为[3, 3, 3, 2, 2, 2, 1, 7, 7, 9]。

### 参考代码

``` java
public int[] relativeSortArray(int[] arr1, int[] arr2) {
    int[] counts = new int[1001];
    for (int num : arr1) {
        counts[num]++;
    }

    int i = 0;
    for (int num : arr2) {
        while (counts[num] > 0) {
            arr1[i++] = num;
            counts[num]--;
        }
    }

    for (int num = 0; num < counts.length; num++) {
        while (counts[num] > 0) {
            arr1[i++] = num;
            counts[num]--;
        }
    }

    return arr1;
}
```

## 12.2 快速排序O(nlogn)

时间复杂度`O(nlogn)`

分治思想，过程如下

1. 在输入数组中随机选取一个元素作为中间值（pivot）
2. 对数组进行分区（partition），使得，所有比中间值小的挪到数组左侧，比中间值大的挪到数组右侧
3. 对中间值左右侧的数组用同样的步骤，直至子数组中只有一个数

**分区步骤**

1. 先将中间值换到末尾去
2. p1初始化为-1的位置，p2初始化为0的位置；p1指向的是发现的最后一个小于中间值的位置
3. 向右移动p2，当p2指向一个小于中间值的数时，移动p1，并交换p1和p2指向的数字，继续；
4. 最后将中间值与p1++互换，此时p1指向的是本次的中间值 

```
  4 1 5 3 6 2 7 8
随机选中了3,移到末尾

  4 1 5 6 2 7 8｜3
↑ ↑
p1p2

  1 4 5 6 2 7 8｜3
  ↑ ↑
  p1p2

  1 2 5 6 4 7 8｜3
    ↑     ↑
    p1    p2

  1 2 3 6 4 7 8 5
      ↑         ↑
      p1        p2

```

​                                                                                                                                                                                                                                                            

```java
public int[] sortArray(int[] nums) {
    quicksort(nums, 0, nums.length - 1);
    return nums;
}

private void quicksort(int[] nums, int start, int end) {
    if (end > start) {
        int pivotIndex = partition(nums, start, end);
        quicksort(nums, start, pivotIndex - 1);
        quicksort(nums, pivotIndex + 1, end);
    }
}

private int partition(int[] nums, int start, int end) {
    int pivot = new Random().nextInt(end - start + 1) + start;
    swap(nums, pivot, end);
    
    int p1 = start - 1;
    for(int p2 = start; p2 < end; p2++) {
        if(nums[p2] < nums[end]) {
            p1++;
            swap(nums, p1, p2);
        }
    }
    p1++;
    swap(nums, p1, end);
    return p1;
}

private void swap(int[] nums, int index1, int index2){
    if(index1 != index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```



## 面试题76：数组中第k大的数字

### 题目

请从一个乱序的数组中找出第k大的数字。例如数组[3, 1, 2, 4, 5, 5, 6]中第3大的数字是5。

### 参考代码

``` java
public int findKthLargest(int[] nums, int k) {
    int target = nums.length - k;
    int start = 0;
    int end = nums.length - 1;
    int index = partition(nums, start, end);
    while (index != target) {
        if (index > target) {
            end = index - 1;
        } else {
            start = index + 1;
        }

        index = partition(nums, start, end);
    }

    return nums[index];
}

private int partition(int[] nums, int start, int end) {
    int random = new Random().nextInt(end - start + 1) + start;
    swap(nums, random, end);

    int small = start - 1;
    for (int i = start; i < end; ++i) {
        if (nums[i] < nums[end]) {
            small++;
            swap(nums, i, small);
        }
    }

    small++;
    swap(nums, small, end);

    return small;
}

private void swap(int[] nums, int index1, int index2) {
    if (index1 != index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}
```



## 12.3 归并排序O(nlogn)

为了排序 n 组，先分成两个 n/2 组，然后合并这两个排序的子数组，依次排序。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202205020938540.png" alt="iShot_2022-05-02_09.36.44" style="zoom: 33%;" />

归并排序需要创建一个和输入数组大小相同的数组，用来保存合并两个排序子数组的结果

数组 src 存放合并之前的数字

数组 dst 存放合并之后的数字

```java
// 迭代
public int[] mergeSort(int[] nums) {
    int[] src = nums;
    int[] dst = new int[nums.length];
	int numsLen = nums.length;
    for (int seg = 1; seg < numsLen; seg += seg) {
        // seg每次每组的大小 1,2,4...
        // 两个一组，四个一组
        for(int start = 0; start < numsLen; start += seg * 2) {
            // start是两两比较的第一组的头
            // start += seg * 2 下一个两两比较的头
            // 和numsLen比较，是因为可能后面不够分成seg一组
            // nextStart下一组的头，第一组的下标要小于它
            int nextStart = Math.min(start + seg, numsLen);
            // nnextStart下下一组的头，第二组的下标j要小于它
            int nnextStart = Math.min(start + seg * 2, numsLen);
            int i = start, j = nextStart, k = start;
            while(i < nextStart || j < nnextStart) {
                // 有可能比较大时候，某一组全是最小的，先放完了
                if (j == nnextStart 
                    || (i < nextStart && src[i] < src[j])) {
                    dst[k++] = src[i++];
                }else {
                    dst[k++] = src[j++];
                }
            }
        }
        
        // 每次的组大小的比较结束后，将分组排序后的dst作为下一次的src
        int[] temp = src;
        src = dst;
        dst = temp;
    }
    // 最后一次的代排src就是排序完成的数组
    return src;
}
```

- 

```java
// 递归
// 排序长度为n的数组，只需分别排序n/2的数组，然后合并即可
public int[] sortArray(int[] nums) {
    int[] dst = new int[nums.length];
    dst = Arrays.copyOf(nums, nums.length);
    mergeSort(nums, dst, 0, nums.length);
    return dst;
}
private void mergeSort(int[] src, int[] dst, int start, int nnextStart) {
    // 说明每组都只有一个了，不用排序
    if(start >= nnextStart - 1) return;
    
    int nextStart = (start + nnextStart) / 2;
    // 将每次排序后的dst是下一次的src
    mergeSort(dst, src, start, nextStart);
    mergeSort(dst, src, nextStart, nnextStart);
    int i = start, j = nextStart, k = start;
    while(i < nextStart || j < nnextStart) {
        // 有可能比较大时候，某一组全是最小的，先放完了
        if (j == nnextStart 
            || (i < nextStart && src[i] < src[j])) {
            dst[k++] = src[i++];
        }else {
            dst[k++] = src[j++];
        }
    }
}

```



## 面试题77：链表排序

### 题目

输入一个链表的头节点，请将该链表排序。例如，输入图12.4（a）中的链表，该链表排序后如图12.4（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1204.png" alt="图2.1">

图12.4：链表排序。（a）一个有6个结点的链表。（b）排序后的链表。

### 参考代码

#### 方法一：递归

空间复杂度`O(logn)`，递归调用栈的开销

``` java
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) {
        return head;
    }

    ListNode head1 = head;
    ListNode head2 = split(head);

    head1 = sortList(head1);
    head2 = sortList(head2);

    return merge(head1, head2);
}
// 快慢双指针，分割链表成两份
private ListNode split(ListNode head) {
    ListNode slow = head;
    ListNode fast = head.next;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    ListNode second = slow.next;
    slow.next = null;

    return second;
}

private ListNode merge(ListNode head1, ListNode head2) {
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;
    while (head1 != null && head2 != null) {
        if (head1.val < head2.val) {
            cur.next = head1;
            head1 = head1.next;
        } else {
            cur.next = head2;
            head2 = head2.next; 
        }
        cur = cur.next;
    }
    cur.next = head1 == null ? head2 : head1;
    return dummy.next;
}
```

#### 方法二：迭代

空间复杂度`O(1)`

```java
public ListNode sortList(ListNode head) {
    int len = 0;
    ListNode node = head;
    while (node != null) {
        node = node.next;
        len++;
    }
    ListNode dummy = new ListNode(0);
    dummy.next = head;

    for (int seg = 1; seg < len; seg += seg) {
        ListNode cur = dummy.next;
        ListNode prev = dummy;

        while (cur != null) {
            ListNode head1 = cur;
            for (int i = 1; i < seg && cur.next != null; i++) {
                cur = cur.next;
            }
            ListNode head2 = cur.next;
            // 将head1所在链表和head2所在链表断开
            cur.next = null;
            cur = head2;
            // 有可能这组只有一条链表
            for (int i = 1; i < seg && cur != null && cur.next != null; i++) {
                cur = cur.next;
            }
            // 再次断开,next相当于第三个链表头
            ListNode next = null;
            if (cur != null) {
                next = cur.next;
                cur.next = null;
            }

            ListNode merged = merge(head1, head2);
            // 然后把链表连起来
            prev.next = merged;
            // 将prev移动到合并好的第一组的尾部
            // 下一次的prev.next = merged后
            // 就会将合并好的链表连起来
            while (prev.next != null) {
                prev = prev.next;
            }

            cur = next;
        }
    }
    return dummy.next;
}

// 合并两个有序链表
private ListNode merge(ListNode head1, ListNode head2) {
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;
    while (head1 != null && head2 != null) {
        if (head1.val < head2.val) {
            cur.next = head1;
            head1 = head1.next;
        } else {
            cur.next = head2;
            head2 = head2.next; 
        }
        cur = cur.next;
    }
    cur.next = head1 == null ? head2 : head1;
    return dummy.next;
}

```



## 面试题78：合并排序链表

### 题目

输入k个排序的链表，请将它们合并成一个排序的链表。例如，输入三个排序的链表如图12.5（a）所示，将它们合并之后得到的排序的链表如图12.5（b）所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1205.png" alt="图2.1">

图12.5：合并排序链表。（a）三个排序的链表。（b）合并后的链表。

### 参考代码

#### 解法一：最小堆

``` java
// 将节点放入minHeap，每次都从中选取最小的那个，然后挨个拼接
public ListNode mergeKLists(ListNode[] lists) {
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;

    PriorityQueue<ListNode> minHeap = new PriorityQueue<>((n1, n2) 
        -> n1.val - n2.val);
    for (ListNode list : lists) {
        if (list != null) {
            minHeap.offer(list);
        }
    }

    while (!minHeap.isEmpty()) {
        ListNode least = minHeap.poll();
        cur.next = least;
        cur = least;

        if (least.next != null) {
            minHeap.offer(least.next);
        }
    }

    return dummy.next;
}
```

#### 解法二：归并排序

``` java
/*
	eg：123456
	start = 0，end = 6
	mid = 3，是后半段的头，也是前半段的长度
	eg：1234567
	start = 0，end = 7
	mid = 3，是后半段的头，也是前半段的长度
*/

public ListNode mergeKLists(ListNode[] lists) {
    if (lists.length == 0) {
        return null;
    }
	// start和end举例更好理解
    return mergeLists(lists, 0, lists.length);
}

private ListNode mergeLists(ListNode[] lists, int start, int end) {
    // 分到只剩单独的链表节点
    if (start + 1 == end) {
        return lists[start];
    }

    int mid = (start + end) / 2;
    ListNode head1 = mergeLists(lists, start, mid);
    ListNode head2 = mergeLists(lists, mid, end);
    return merge(head1, head2);
}

// 合并两个排序链表，一道单独的题目
private ListNode merge(ListNode head1, ListNode head2) {
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;
    while (head1 != null && head2 != null) {
        if (head1.val < head2.val) {
            cur.next = head1;
            head1 = head1.next;
        } else {
            cur.next = head2;
            head2 = head2.next; 
        }

        cur = cur.next;
    }

    cur.next = head1 == null ? head2 : head1;
    return dummy.next;
}
```



## 12.4 冒泡排序

- 一共`[0,n-1]`依次两两交换，最终第一轮会把最大的放到末尾
- 第二轮在`[0, n-2]`中从头两两交换

```java
public bubble(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = 0; j + 1 < nums.length - i; j++) {
            if(nums[j] > nums[j+1]) {
                int temp = nums[j];
                nums[j] = nums[j+1];
                nums[j+1] = temp;
            }
        }
    }
}
```



## 12.5 选择排序

1. 遍历全部，选择最小（最大）的与第一个节点交换
2. 从第二个节点开始遍历后面，选择最小（最大）的，与第二个交换，这样前两个是排序的
3. 以此类推

```java
void select_sort(int a[], int n)
{
    int i;        // 有序区的末尾位置
    int j;        // 无序区的起始位置
    int min;    // 无序区中最小元素位置

    for(i=0; i<n; i++)
    {
        min=i;
        //找"a[i+1]..a[n]"之间最小元素，并赋给min
        for(j=i+1; j<n; j++)
        {
            if(a[j] < a[min])
                min=j;
        }
        //若min!=i，则交换 a[i] 和 a[min]。
        //交换后，保证了a[0]..a[i]之间元素有序。
        if(min != i)
            swap(a[i], a[min]);
    }
}
```

## 12.6 插入排序

时间复杂度`O(n^2)`

每次迭代中，插入排序只从输入数据中移除一个待排序的元素，找到它在序列中适当的位置，并将其插入。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202205011937522.gif" alt="Insertion-sort-example-300px"  />

```java
public void insertSort(int arr[]){
    for(int i = 1; i < arr.length; i++){
        int rt = arr[i];
        for(int j = i - 1; j >= 0; j--){
            if(rt < arr[j]){
                arr[j + 1] = arr[j];
                arr[j] = rt;
            }else{
                break;
            }
        }
    }
}
```



## 12.7 希尔排序

插入排序的改进版。

为了减少数据的移动次数，在初始序列较大时取较大的步长，通常取序列长度的一半，此时只有两个元素比较，交换一次；之后步长依次减半直至步长为1，即为插入排序，由于此时序列已接近有序，故插入元素时数据移动的次数会相对较少，效率得到了提高。

第一趟：gap = len / 2 = 4

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202205011948122.png" alt="image-20220501194841078" style="zoom: 33%;" />

第二趟：gap = gap >> 1 = 2

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202205011950396.png" alt="image-20220501195041892" style="zoom: 33%;" />

第三趟：gap = 1

就是普通插入排序

```java
void shellSort(int arr[]){
    int gap = arr.length >> 1;
    while(gap >= 1){
        for(int i = gap; i < arr.length; i++){
            int rt = arr[i];
            for(int j = i - gap; j >= 0; j -= d){
                if(rt < arr[j]){
                    arr[j + gap] = arr[j];
                    arr[j] = rt;
                }else break;
            }
        }
        gap >>= 1;
    }
}
```

