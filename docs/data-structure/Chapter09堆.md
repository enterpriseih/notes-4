# 第九章：堆

## 完全二叉树

- 在**填满**的情况下

每一层元素数都是上一层的二倍，所以**最后一层的节点数量是之前所有层节点总数 + 1**

**最后一层的第一个节点的索引就是节点总数/2**（根节点索引为0），该节点也是第一个叶子结点

则**最后一个非叶子结点的索引就是第一个叶子结点的索引 - 1**

即最后一个非叶子结点的索引 = 节点总数 / 2

- 在不填满的情况下也适用

```
       9
     /   \
    5     8
   / \   / \
  2   3 4   7
 /
1
总共8个节点
第一个叶子结点的的索引 = 8 / 2 = 4，是节点3
则最后一个非叶子结点的索引 = 4 - 1 = 3，是节点2
```



## 堆

堆是一种完全二叉树：除了最底层，每层都填满

最大堆：每个节点的值总是大于等于其任意子节点；arr[i] >= arr[2i + 1] && arr[i] >= arr[2i + 2]

最小堆：每个节点的值总是小于等于其任意子节点；arr[i] <= arr[2i + 1] && arr[i] <= arr[2i + 2]

```
大顶堆
       9
     /   \
    5     8
   / \   / \
  2   3 4   7
 /
1
```

> - 最小堆 --> 找k个最大的元素
> - 最大堆 --> 找k个最小的元素

用数组表示堆，堆中一节点下标设为i，则（从0开始）

```
   (i-1)/2
     / 
    i   
  /   \ 
2i+1  2i+2 
```

**最大堆添加**：

1、从上到下，从左到右，找到空位，添加

2、判断新节点是否比其父节点大，是则交换，直到小于其父节点或到顶

**最大堆删除**：通常指删除堆顶元素

1、删除堆顶元素，然后将最底层最右侧的节点移动到堆顶

2、如果小于其左或右节点，则将该节点与其左右子节点中**较大**的节点交换

3、重复，直至大于或等于子节点，或到底

`堆中有n个节点，则插入或删除的时间复杂度都是O(logn)`

```java
// 默认最小堆
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
// 传入Comparator可以改变比较规则
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(
    (e1, e2) -> e2 - e1
);
// 如果存的是Double这样的非int类型，
// 要用e1 - e2 => e1.compareTo(e2)
// 使用函数和Queue一样，offer、poll、peek
```

## 补充：滑动窗口的最大值

### 题目

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7

```



### 解一：最大堆

时间复杂度`O(nlongn)`

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int len = nums.length;
    int[] res = new int[len - k + 1];
    // 堆中存放的是 值和下标
    PriorityQueue<int[]> maxHeap = new PriorityQueue<int[]>(
        (e1, e2) -> e1[0] != e2[0] ? e2[0] - e1[0] : e2[1] - e1[1]
    );
    /*
    PriorityQueue<int[]> maxHeap = new PriorityQueue<int[]>(new Comparator<int[]>() {
        public int compare(int[] e1, int[] e2) {
            return e1[0] != e2[0] ? e2[0] - e1[0] : e2[1] - e1[1];
        }
    });
	*/
    for (int i = 0; i < k; i++) {
        maxHeap.offer(new int[]{nums[i], i});
    }

    res[0] = maxHeap.peek()[0];
    for (int i = k; i < len; i++) {
        maxHeap.offer(new int[]{nums[i], i});
        // 添完新数，就要删除旧数
        // 如果max在窗口的最左侧或者已经不在窗口中了，则去掉
        while (maxHeap.peek()[1] <= i - k) {
            maxHeap.poll();
        }
        res[i - k + 1] = maxHeap.peek()[0];
    }

    return res;
}

```

### 解二：双向队列

时间复杂度`O(n)`

```java
// 队列中存放坐标
public int[] maxSlidingWindow(int[] nums, int k) {
    if(nums == null || nums.length == 1) return nums;
    // 双向队列 保存当前窗口最大值的坐标 保证队列中数组位置的数值按从大到小排序
    LinkedList<Integer> queue = new LinkedList();
    int[] result = new int[nums.length-k+1];

    for(int i = 0;i < nums.length;i++){
        // 从大到小排，要求队首对应的数是最大的
        while (!queue.isEmpty() && nums[queue.peekLast()] < nums[i]) {
            queue.pollLast();
        }
        queue.addLast(i);
        // 队首的下标超出了窗口的界，就要去除
        while (queue.peekFirst() <= i - k) {
            queue.pollFirst();
        }
        // 当窗口长度为k时 保存当前窗口中最大值
        if(i+1 >= k){
            result[i-k+1] = nums[queue.peek()];
        }
    }
    return result;
}
```



## 面试题59：数据流的第k大数值

### 题目

请设计一个类型KthLarges，它每次从一个数据流中读取一个数字，并得出数据流已经读取的数字中第k（k≥1）大的数值。该类型的构造函数有两个参数，一个是整数k，另一个是包含数据流中最开始数值的整数数组nums（假设数组nums的长度大于k）。该类型还有一个函数add用来添加数据流中的新数值并返回数据流中已经读取的数字的第k大数值。

例如，

```
当k=3、nums为数组[4, 5, 8, 2]时，
调用构造函数创建除类型KthLargest的实例之后，
第一次调用add函数添加数字3，
此时已经从数据流里读取了数值4、5、8、2和3，第3大的数值是4；
第二次调用add函数添加数字5时，则返回第3大的数值5。
```



### 参考代码

``` java
class KthLargest {
    private PriorityQueue<Integer> minHeap;
    private int size;

    public KthLargest(int k, int[] nums) {
        // 让堆中只存k个，则最小堆堆顶就是第k大元素
        size = k;
        minHeap = new PriorityQueue<>();
        for (int num : nums) {
            add(num);
        }
    }

    public int add(int val) {
        if (minHeap.size() < size) {
            minHeap.offer(val);
        } else if (val > minHeap.peek()) {
            minHeap.poll();
            minHeap.offer(val);
        }

        return minHeap.peek();
    }
}
```

## 面试题60：出现频率最高的k个数字

### 题目

请找出数组中出现频率最高的k个数字。

例如

当`k=2`时输入数组`[1, 2, 2, 1, 3, 1]`，由于数字1出现3次，数字2出现2次，数字3出现1，那么出现频率最高的2个数字是1和2。

### 参考代码

``` java
public int[] topKFrequent(int[] nums, int k) {
    // 存数字及其次数
    Map<Integer, Integer> numToCount = new HashMap<>();
    for (int num : nums) {
        numToCount.put(num, numToCount.getOrDefault(num, 0) + 1);
    }

    Queue<Map.Entry<Integer, Integer>> minHeap = new PriorityQueue<>(
        (e1, e2) -> e1.getValue() - e2.getValue()
    );
    for (Map.Entry<Integer, Integer> entry : numToCount.entrySet()) {
        if (minHeap.size() < k) {
            minHeap.offer(entry);
        } else {
            if (entry.getValue() > minHeap.peek().getValue()) {
                minHeap.poll();
                minHeap.offer(entry);
            }
        }
    }

    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) {
        result[i] = minHeap.poll().getKey();
    }

    return result;
}
```

## 面试题61：和最小的k个数对

### 题目

给你两个递增排序的整数数组，分别从两个数组中各取一个数字u、v形成一个数对(u, v)，请找出和最小的k个数对。例如输入两个数组[1, 5, 13, 21]和[2, 4, 9, 15]，和最小的3个数对为(1, 2)、(1, 4)和(2, 5)。

### 参考代码

#### 解法一：最大堆

> 数组是递增排序，则第一个数组中取第k+1个数和第二个数组中取任意数字组成和p
>
> 其一定不是和最小的k个之一

``` java
public List<List<Integer>> kSmallestPairs(int[] nums1,
    int[] nums2, int k) {
    Queue<int[]> maxHeap = new PriorityQueue<>((p1, p2)
        -> p2[0] + p2[1] - p1[0] - p1[1]);
    
    for (int i = 0; i < Math.min(k, nums1.length); ++i) {
        for (int j = 0; j < Math.min(k, nums2.length); ++j) {
            // 装满了
            if (maxHeap.size() >= k) {
                // 堆中和最大的数对
                int[] root = maxHeap.peek();
                if (root[0] + root[1] > nums1[i] + nums2[j]) {
                    maxHeap.poll();
                    maxHeap.offer(new int[] {nums1[i], nums2[j]});
                }
            } else {
                maxHeap.offer(new int[] {nums1[i], nums2[j]});
            }
        }
    }

    List<List<Integer>> result = new LinkedList<>();
    while (!maxHeap.isEmpty()) {
        int[] vals = maxHeap.poll();
        result.add(Arrays.asList(vals[0], vals[1]));
    }

    return result;
}
```

#### 解法二：最小堆

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202205201616274.png" alt="image-20220520161600269" style="zoom:50%;" />

先放进去三个候选，A0，A1，A2与B0的组合

第一对最小组合肯定是A0B0

第二对不是A0B1，就是A1B0，但A1B0已经在堆中，所以需要放入A0B1，然后堆顶就是第二个需要的数对

下一对的第二个坐标是堆顶弹出的数对的B的下标+1

``` java
public List<List<Integer>> kSmallestPairs(
    int[] nums1,int[] nums2, int k) {
    Queue<int[]> minHeap = new PriorityQueue<>((p1, p2)
        -> nums1[p1[0]] + nums2[p1[1]] - nums1[p2[0]] - nums2[p2[1]]);
    // 先把第一个
    if (nums2.length > 0) {
        for (int i = 0; i < Math.min(k, nums1.length); ++i) {
            minHeap.offer(new int[] {i, 0});
        }
    }

    List<List<Integer>> result = new ArrayList<>();        
    while (k-- > 0 && !minHeap.isEmpty()) {
        int[] ids = minHeap.poll();
        result.add(Arrays.asList(nums1[ids[0]], nums2[ids[1]]));

        if (ids[1] < nums2.length - 1) {
            minHeap.offer(new int[] {ids[0], ids[1] + 1});
        }
    }

    return result;
}
```

