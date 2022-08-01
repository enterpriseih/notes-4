# 第十一章：二分查找

```java
// 第一种写法：左闭右闭区间
public int search(int[] nums, int target) {
    // 避免当 target 小于nums[0] nums[nums.length - 1]时多次循环运算
    if (target < nums[0] || target > nums[nums.length - 1]) {
        return -1;
    }
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) >> 1;
        if (nums[mid] == target)
            return mid;
        else if (nums[mid] < target)
            left = mid + 1;
        else if (nums[mid] > target)
            right = mid - 1;
    }
    return -1;
}

// 第二种写法：左闭右开
public int search(int[] nums, int target) {
    // 此处右边界是长度
    int left = 0, right = nums.length;
    while (left < right) {
        int mid = left + ((right - left) >> 1);
        if (nums[mid] == target)
            return mid;
        else if (nums[mid] < target)
            // 下一轮搜索的区间是 [mid + 1..right)
            left = mid + 1;
        else if (nums[mid] > target)
            // 下一轮搜索的区间是 [left..mid)
            right = mid;
    }
    return -1;
}
```



## 11.1 排序数组的二分查找

## 面试题68：查找插入位置

### 题目

输入一个排序的整数数组nums和一个目标值t，如果nums中包含t，返回t在数组中的下标；如果nums中不包含t，则返回如果将t添加到nums里时t在nums中的下标。假设数组中的没有相同的数字。

例如，输入数组nums为[1, 3, 6, 8]，如果目标值t为3，则输出1；如果t为5，则返回2。

### 参考代码

``` java
public int searchInsert(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    while (left <= right) {
        int mid = (left + right) / 2;
        if (nums[mid] >= target) {
            if (mid == 0 || nums[mid - 1] < target) {
                return mid;
            }
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }

    return nums.length;
}
```

```java
// 第二种写法：左闭右开
// while (left < right)，
// 这里使用 < ,因为left == right在区间[left, right)是没有意义的
public int searchInsert(int[] nums, int target) {
    int len = nums.length;
    // 特殊判断，好像可以不需要，下面的程序包括了这种情况
    if (nums[len - 1] < target) {
        return len;
    }

    // 程序走到这里一定有 nums[len - 1] >= target，
    // 插入位置在区间 [0..len - 1]即[0..len)
    int left = 0, right = len;
    // 在区间 nums[left..right) 里查找第 1 个大于等于 target 的元素的下标
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target){ 
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    return left;
}
```





## 面试题69：山峰数组的顶部

### 题目

在一个长度大于或等于3的数组里，任意相邻的两个数都不相等。该数组的前若干个数字是递增的，之后的数字是递减的，因此它的值看起来像一座山峰。请找出山峰顶部即数组中最大值的位置。例如，在数组[1, 3, 5, 4, 2]中，最大值是5，输出它在数组中的下标2。

### 参考代码

``` java
public int peakIndexInMountainArray(int[] nums) {
    int left = 1;
    int right = nums.length - 2;
    while (left <= right) {
        int mid = (left + right) / 2;
        if (nums[mid] > nums[mid + 1] && nums[mid] > nums[mid - 1]) {
            return mid;
        }
        if (nums[mid] > nums[mid - 1]) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;
}
```

## 面试题70：排序数组中只出现一次的数字

### 题目

在一个排序的数组中，除了一个数字只出现一次之外其他数字数字都出现了两次，请找出这个唯一只出现一次的数字。例如，在数组[1, 1, 2, 2, 3, 4, 4, 5, 5]中，数字3只出现一次。

### 参考代码

- 两两一组，单独的那个数会导致后面的组别里的数都不相同

```
(1,1)、(2,2)、(3,4)、(4,5)、(5)
```

- 所以只出现一次的数字正好是`第一个两两一组不相同`的分组的`第一个数字`
- `n(奇数)`个数可以分成`n/2+1`组，最后一组只有一个数字
- 从`0`开始编号，为`0～n/2`，`left`是查找范围内第一个分组的编号，`right`是查找范围内第二个分组的编号，查找编号为`mid`的分组，分组的第一个数字的编号为`i`

``` java
public int singleNonDuplicate(int[] nums) {
    int left = 0;
    int right = nums.length / 2;
    while (left <= right) {
        int mid = (left + right) / 2;
        int i = mid * 2;
        if (i < nums.length - 1 && nums[i] != nums[i + 1]) {
            if (mid == 0 || nums[i - 2] == nums[i - 1]) {
                return nums[i];
            }
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
	// 直到最后都没找到两个数字不同的分组，说明单独的数字在数组尾部
    return nums[nums.length - 1];
}
```

## 面试题71：按权重生成随机数

### 题目

输入一个正整数数组w，数组中的每个数字w[i]表示下标i的权重，请实现一个函数pickIndex根据权重比例随机选择一个下标。

例如，如果权重数组w为[1, 2, 3, 4]，这pickIndex将有10%的概率选择0、20%的概率选择1、30%的概率选择2、40%的概率选择3。

### 题解

先计算权重和total。

sums[i]就是权重数组前i个元素和。

找sums中第一个大于随机数的值的对应的下标就是求的答案。

```
[3,4,1,2]的权重为[3,7,8,10]
0 1 2 3 4 5 6 7 8 9 10
[0,3) - 3 在权重数组sums中的下标为0
[3,7) - 7
[7,8) - 8
[8,10) - 10
```



``` java
class Solution {
    private int[] sums;
    private int total;
    
    public Solution(int[] w) {
        sums = new int[w.length];
        for (int i = 0; i < w.length; ++i) {
            total += w[i];
            sums[i] = total;
        }
    }
    
    public int pickIndex() {
        Random random = new Random();
        // 返回一个[0, total)的随机数，0～total-1
        int p = random.nextInt(total);
        int left = 0;
        int right = sums.length;
        // 找第一个大于p的元素，对应的下标就是
        while (left <= right) {
            int mid = (left + right) / 2;
            if (sums[mid] > p) {
                if (mid == 0 || (sums[mid - 1] <= p)) {
                    return mid;
                }
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        
        return -1;
    }
}
```

## 补：寻找两个正序数组的中位数

给定两个大小分别为 m 和 n 的正序（从小到大）数组 nums1 和 nums2。请你找出并返回这两个正序数组的 中位数 。

算法的时间复杂度应该为 O(log (m+n)) 。

### 思路

```
/* 要找到第 k (k>1) 小的元素，那么就取 pivot1 = nums1[k/2-1] 和 pivot2 = nums2[k/2-1] 进行比较
 * 这里的 "/" 表示整除
 * nums1 中小于等于 pivot1 的元素有 nums1[0 .. k/2-2] 共计 k/2-1 个
 * nums2 中小于等于 pivot2 的元素有 nums2[0 .. k/2-2] 共计 k/2-1 个
 * 取 pivot = min(pivot1, pivot2)，两个数组中小于等于 pivot 的元素共计不会超过 (k/2-1) + (k/2-1) <= k-2 个
 * 这样 pivot 本身最大也只能是第 k-1 小的元素
 * 如果 pivot = pivot1，那么 nums1[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums1 数组
 * 如果 pivot = pivot2，那么 nums2[0 .. k/2-1] 都不可能是第 k 小的元素。把这些元素全部 "删除"，剩下的作为新的 nums2 数组
 * 由于 "删除" 了一些元素（这些元素都比第 k 小的元素要小），因此需要修改 k 的值，减去删除的数的个数
 */
```



```
A：1，2，4，9
B：1，2，3，4，5，6，7，8，9

长度4和9，中位数是第7个，因此寻找第k=7个元素
A: 1 3 4 9
       ↑
B: 1 2 3 4 5 6 7 8 9
       ↑

A: 1 3 4 9
     ↑
B: [1 2 3] 4 5 6 7 8 9
             ↑

A: [1 3] 4 9
         ↑
B: [1 2 3] 4 5 6 7 8 9
           ↑

A: [1 3 4] 9
           ↑
B: [1 2 3] 4 5 6 7 8 9
           ↑
```

### 解法

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int len1 = nums1.length;
    int len2 = nums2.length;
    int lens = len1 + len2;
    if (lens % 2 == 1) {
        return getKthElement(nums1, nums2, lens/2 + 1) * 1.0;
    } else {
        return (getKthElement(nums1, nums2, lens/2) 
        + getKthElement(nums1, nums2, lens/2 + 1)) * 0.5;
    }
}

private int getKthElement(int[] nums1, int[] nums2, int k) {
    // 下标起点，相当于删除了前面那些
    // 比如抛弃了0 1 2下标的nums1.那么下标为3的实际就是新的首位数组元素。
    int index1 = 0;
    int index2 = 0;
    int len1 = nums1.length;
    int len2 = nums2.length;
    while (true) {
        // nums1删没了，那就从nums2中找当前的第k小，就是第k个
        if (index1 == len1) {
            return nums2[index2 + k - 1];
        }
        if (index2 == len2) {
            return nums1[index1 + k - 1];
        }
        if (k == 1) {
            return Math.min(nums1[index1], nums2[index2]);
        }

        int half = k / 2;
        int newIndex1 = Math.min(index1 + half, len1) - 1;
        int newIndex2 = Math.min(index2 + half, len2) - 1;
        if (nums1[newIndex1] <= nums2[newIndex2]) {
            k -= (newIndex1 - index1 + 1);// 删去的子数组长度
            index1 = newIndex1 + 1;
        } else {
            k -= (newIndex2 - index2 + 1);
            index2 = newIndex2 + 1;
        }
    }
}
```



## 补充：搜索二维矩阵Ⅰ

### 题

编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

- 每行中的整数从左到右按升序排列。
- 每行的第一个整数大于前一行的最后一个整数。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202205251552931.jpg" alt="img" style="zoom:67%;" />

### 解

```java
public static boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length;
    int n = matrix[0].length;
    int l = 0, r = m-1;
    
    // 二分查找：在第0列中二分，找到小于target的最接近的元素，记录行号row
    // 即找到第一个小于target的元素
    int row = 0;
    while (l <= r) {
        int mid = l + (r-l)/2;
        int cur = matrix[mid][0];
        if (cur == target) return true;
        else if (cur < target) {
            row = mid;
            l = mid + 1;
        } else { // cur > target
            r = mid - 1;
        }
    }
    
    // 二分查找：在第row行中二分，找target是否存在
    l = 1;
    r = n-1;
    while (l <= r) {
        int mid = l + (r-l)/2;
        int cur = matrix[row][mid];
        if (cur == target) return true;
        else if (cur < target) l = mid + 1;
        else r = mid - 1;
    }

    return false;
}
```



## 补充：搜索二维矩阵Ⅱ

### 题

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202205251608933.jpg" alt="img" style="zoom:50%;" />

### 解

**方法一**：针对每行都进行二分查找

略

**方法二**：右上角看作是 BST 二叉搜索树的根节点，左是左节点，右是右节点

该方法前题也能使用

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202205251646634.png" alt="image-20220525164605256" style="zoom: 33%;" />

```java
public boolean searchMatrix(int[][] matrix, int target) {
    int m = matrix.length, n = matrix[0].length;
    int r = 0, c = n - 1;
    while (r < m && c >= 0) {
        int cur = matrix[r][c];
        if (target > cur) r++;
        else if (target < cur) c--;
        else return true;
    } 
    return false;
}
```







## 11.2 在数值范围内二分查找

## 面试题72：求平方根

### 题目

输入一个非负整数，请计算它的平方根。正数的平方根有两个，只输出其中正数平方根。如果平方根不是整数，只需要输出它的整数部分。例如，如果输入4则输出2；如果输入18则输出4。

### 参考代码

``` java
public int mySqrt(int n) {
    int left = 1;
    int right = n;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (mid <= n / mid) {
            if ((mid + 1) > n / (mid + 1)) {
                return mid;
            }

            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return 0;
}
```

## 补充：子序列最小值的最大值

### 题目

有m个节点，k个任务，每个任务都有执行时长，将这k个任务在m个节点上执行，每个节点执行的任务序列，必须是连续的，如1、2、3，不可以是1、3，求最短的执行时间。

```
输入：
3 5
1 5 3 4 2
输出：
6
说明：3个节点，5个任务，每个任务的执行时长是1、5、3、4、2
分成三个序列，{1、5} {3} {4、2}，最长时长是6

分成的子序列的最大值中的最小时间
```



### 解法

```
二分法，初始化时，

最短执行时间在[ Max(1、5、3、4、2) ，1+5+3+4+2 ]之间，

使用二分法，mid=（left+end)/2，

判断执行时间是mid时，可以分成多少个子序列，使每个子序列的执行时长都小于mid，

如果计算结果是k个子序列，若k>mid，right=mid，

否则，left=mid+1，继续下层循环，直至left==right，返回结果。

```

当m=3、dataArray=[1,5,3,4,2]，执行流程如下：

- left=5，right=15，mid=10，当前执行时间是10，可以分成{1,5,3}、{4,2}两个序列，即k=2，k<(m=3)，所以下一步 ：right=mid=10
- left=5，right=10，mid=7，当前执行时间是7，可以分成{1,5}、{3,4}、{2}三个序列，即k=3，k<=(m=3)，所以下一步 right=mid=7  
- left=5，right=7，mid=6，当前执行时间是6，可以分成{1,5}、{3}、{4,2}三个序列，即k=3，k<=(m=3)，所以下一步 right=mid=6
- left=5，right=6，mid=5，当前执行时间是5，可以分成{1}、{5}、{3}、{4}、{2}五个序列，即k=5，k>(m=3)，所以下一步： left=mid+1=6
- left=6，right=6，循环结束，返回执行时间6



```java
import java.util.*;
 
public class Main {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int m = in.nextInt();
        int size = in.nextInt();
        int[] dataArray = new int[size];
        for (int i = 0; i < size; i++) {
            dataArray[i] = in.nextInt();
        }
        System.out.println(schedule(m, dataArray));
    }
 
    static int schedule(int m, int[] dataArray) {
        int left = 0, right = 0;
        for (int i = 0; i < dataArray.length; i++) {
            left = Math.max(left, dataArray[i]);
            right += dataArray[i];
        }
        int mid;
        while (left < right) {
            mid = (left + right) >> 1;
            // 判断当每个子序列最大执行时间是mid，
            // 可以分成多少个子序列，子序列的数量是t
            int t = 1, sum = dataArray[0];
            for (int i = 1; i < dataArray.length; i++) {
                if (sum + dataArray[i] <= mid) {
                    sum += dataArray[i];
                } else {
                    ++t;
                    if (t > m) {
                        break;
                    }
                    sum = dataArray[i];
                }
            }
            //已获得子序列的数量t
            if (t > m) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return right;
    }
}
```








## 面试题73：狒狒吃香蕉

### 题目

狒狒很喜欢吃香蕉。一天它发现了n堆香蕉，第i堆有piles[i]个香蕉。门卫刚好走开了要H小时后才会回来。狒狒吃香蕉喜欢细嚼慢咽，但又想在门卫回来之前吃完所有的香蕉。请问狒狒每小时至少吃多少根香蕉？如果狒狒决定每小时吃k根香蕉，而它在吃的某一堆剩余的香蕉的数目少于k，那么它只会将这一堆的香蕉吃完，下一个小时才会开始吃另一堆的香蕉。

### 参考代码

``` java
public int minEatingSpeed(int[] piles, int H) {
    int max = Integer.MIN_VALUE;
    for (int pile : piles) {
        max = Math.max(max, pile);
    }

    int left = 1;
    int right = max;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int hours = getHours(piles, mid);
        if (hours <= H) {
            if (mid == 1 || getHours(piles, mid - 1) > H) {
                return mid;
            }

            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }

    return -1;
}

private int getHours(int[] piles, int speed) {
    int hours = 0;
    for (int pile : piles) {
        // hours += (pile + speed - 1) / speed;
        hours += pile/speed + pile%speed == 0 ? 0 : 1;
    }

    return hours;
}
```

