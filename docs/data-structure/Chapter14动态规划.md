# 第十四章：动态规划

1. 确定dp数组（dp table）以及下标的含义
2. 确定递推公式
3. dp数组如何初始化
4. 确定遍历顺序
5. 举例推导dp数组

## 补充：最大连续子数组之和

给你一个整数数组 nums ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

### 题解

`dp[i]以i结尾的最大连续子序列和`

```java
// dp[i]以i结尾的最大连续子序列和
// 1
dp[i] = max{nums[i], dp[i - 1] + nums[i]};
// 2，可以合并成1
if (dp[i - 1] > 0) {
    dp[i] = dp[i - 1] + nums[i];
} else {
    dp[i] = nums[i];
}
```



```java
public int maxSubArray(int[] nums) {
    int res = Integer.MIN_VALUE;
    int pre = 0;
    for (int num : nums) {
        // dp[i] = max{nums[i], dp[i - 1] + nums[i]};
        pre = Math.max(num, pre + num);
        // 求max，一起球了
        res = Math.max(res, pre);
    }
    return res;
}
```

```java
public int maxSubArray(int[] nums) {
    int len = nums.length;
    int[] dp = new int[len];
    dp[0] = nums[0];
    int res = dp[0];
    for (int i = 1; i < len; i++) {
        if (dp[i - 1] > 0) {
            dp[i] = dp[i - 1] + nums[i];
        } else {
            dp[i] = nums[i];
        }
        res = Math.max(res, dp[i]);
    }
    return res;
}
```

## 补充：最大连续子数组乘积

```
输入: nums = [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```
要维护两个值，一个最大值，一个最小值，

这个最小值是可能为负数的，负负得正

```java
public int maxProduct(int[] nums) {
    int len = nums.length;
    int[] fmin = new int[len];
    int[] fmax = new int[len];
    int res = nums[0];
    fmin[0] = nums[0];
    fmax[0] = nums[0];
    for (int i = 1; i < len; i++) {
        int t1 = fmax[i-1] * nums[i], t2 = fmin[i-1] * nums[i];
        fmax[i] = Math.max(t1, Math.max(t2, nums[i]));
        fmin[i] = Math.min(t1, Math.min(t2, nums[i]));
        res = Math.max(fmax[i], res);
    }
    return res;
}
```





## 不同的二叉搜索树

[代码随想录](https://programmercarl.com/0096.%E4%B8%8D%E5%90%8C%E7%9A%84%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91.html#%E6%80%9D%E8%B7%AF)

### 题目

给定一个整数 n，求以 1 ... n 为节点组成的二叉搜索树有多少种？

### 题解

**dp[i] ： 1到i为节点组成的二叉搜索树的个数为dp[i]**。

`dp[i] = \sum_1^i(dp[j-1] * dp[i-j])`

`j-1 为 j 为头结点左子树节点数量，i-j 为以 j 为头结点右子树节点数量`

```java
public int numTrees(int n) {
    //初始化 dp 数组
    int[] dp = new int[n + 1];
    //初始化0个节点和1个节点的情况
    dp[0] = 1;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            //对于第i个节点，需要考虑1作为根节点直到i作为根节点的情况，所以需要累加
            //一共i个节点，对于根节点j时,左子树的节点个数为j-1，右子树的节点个数为i-j
            dp[i] += dp[j - 1] * dp[i - j];
        }
    }
    return dp[n];
}
```



## 面试题88：爬楼梯的最少成本

### 题目

一个数组cost的所有数字都是正数，它的第i个数字表示在一个楼梯的第i级台阶往上爬的成本，在支付了成本cost[i]之后我们可以从第i级台阶往上爬1级或者2级。假设台阶至少有两级，我们可以从第0级台阶出发，也可以从第1级台阶出发，请计算爬上该楼梯的最少成本。例如输入数组[1, 100, 1, 1, 100, 1]，则爬上该楼梯的最少成本是4，分别经过下标为0、2、3、5这四级台阶，如图14.1所示。 

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1401.png" alt="图14.1" width=300px>

图14.1： 一个楼梯爬上每级台阶的成本用数组[1, 100, 1, 1, 100, 1]表示，爬上该台阶的最少成本为4，分别经过下标为0、2、3、5四级台阶。

### 参考代码

#### 解法一

``` java
public int minCostClimbingStairs(int[] cost) {
    int len = cost.length;
    int[] dp = new int[len];
    helper(cost, len - 1, dp);
    return Math.min(dp[len - 2], dp[len - 1]);
}

private void helper(int[] cost, int i, int[] dp) {
    if (i < 2) {
        dp[i] = cost[i];
    } else if (dp[i] == 0) {
        helper(cost, i - 2, dp);
        helper(cost, i - 1, dp);
        dp[i] = Math.min(dp[i - 2], dp[i - 1]) + cost[i];
    }
}
```

#### 解法二

``` java
public int minCostClimbingStairs(int[] cost) {
    int len = cost.length;
    int[] dp = new int[len];
    dp[0] = cost[0];
    dp[1] = cost[1];

    for (int i = 2; i < len; i++) {
        dp[i] = Math.min(dp[i - 2], dp[i - 1]) + cost[i];
    }

    return Math.min(dp[len - 2], dp[len - 1]);
}
```

#### 解法三

``` java
public int minCostClimbingStairs(int[] cost) {
    int[] dp = new int[]{cost[0], cost[1]};
    for (int i = 2; i < cost.length; i++) {
        dp[i % 2] = Math.min(dp[0], dp[1]) + cost[i];
    }
    
    return Math.min(dp[0], dp[1]);
}
```

## 14.1 单序列问题

## 面试题89：房屋偷盗

### 题目

输入一个数组表示某条街上的一排房屋内财产的数量。如果这条街道上相邻的两家被盗就会自动触发报警系统。一个小偷打算到给街去偷窃，请计算该小偷最多能偷到多少财产。例如，街道上五家的财产用数组[2, 3, 4, 5, 3]表示，如果小偷到下标为1、2和4的房屋内偷窃，那么他能偷取到价值9的财物，这是他在不触发报警系统情况下能偷取到的最多的财物，如图14.3所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1403.png" alt="图2.1">

图14.3： 一条街道上有5个财产数量分别为2、3、4、5、3的家庭。一个小偷到这条街道上偷东西，如果他不能到相邻的两家盗窃，那么他最多只能偷到价值为9的财物。被盗的房屋上方用特殊符号标出。

### 参考代码

```java
dp[i]:前i个房子能偷到的最大钱财
dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
```



#### 解法一

``` java
public int rob(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }

    int[] dp = new int[nums.length];
    Arrays.fill(dp, -1);

    helper(nums, nums.length - 1, dp);
    return dp[nums.length - 1];
}

private void helper(int[]nums, int i, int[] dp) {
    if (i == 0) {
        dp[i] = nums[0];
    } else if (i == 1) {
        dp[i] = Math.max(nums[0], nums[1]);
    } else if (dp[i] < 0) {
        helper(nums, i - 2, dp);
        helper(nums, i - 1, dp);
        dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
    }
}
```

#### 解法二

``` java
public int rob(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }

    int[] dp = new int[nums.length];
    dp[0] = nums[0];

    if (nums.length > 1) {
        dp[1] = Math.max(nums[0], nums[1]);
    }

    for (int i = 2; i < nums.length; i++) {
        dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
    }

    return dp[nums.length - 1];
}
```

#### 解法三

``` java
public int rob(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }

    int[] dp = new int[2];
    dp[0] = nums[0];

    if (nums.length > 1) {
        dp[1] = Math.max(nums[0], nums[1]);
    }

    for (int i = 2; i < nums.length; i++) {
        dp[i%2] = Math.max(dp[(i-1) % 2], dp[(i-2) % 2] + nums[i]);
    }

    return dp[(nums.length-1)%2];
}
```

#### 解法四

``` java
public int rob(int[] nums) {
    int len = nums.length;
    if (len == 0) {
        return 0;
    }

    int[][] dp = new int[2][2];
    dp[0][0] = 0;
    dp[1][0] = nums[0];

    for (int i = 1; i < len; i++) {
        dp[0][i % 2] = Math.max(dp[0][(i-1) % 2], dp[1][(i-1) % 2]);
        dp[1][i % 2] = nums[i] + dp[0][(i-1) % 2];
    }

    return Math.max(dp[0][(len - 1) % 2], dp[1][(len - 1) % 2]);
}
```

## 面试题90：环形房屋偷盗

### 题目

一个环形街道上有若干房屋。输入一个数组表示该街上的房屋内财产的数量。如果这条街道上相邻的两家被盗就会自动触发报警系统。一个小偷打算到给街去偷窃，请计算该小偷最多能偷到多少财产。例如，街道上五家的财产用数组[2, 3, 4, 5, 3]表示，如果小偷到下标为1和3的房屋内偷窃，那么他能偷取到价值8的财物，这是他在不触发报警系统情况下能偷取到的最多的财物，如图14.4所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1404.png" alt="图2.1">

图14.4： 一条环形街道上有5个财产数量分别为2、3、4、5、3的家庭。一个小偷到这条街道上偷东西，如果他不能到相邻的两家盗窃，那么他最多只能偷到价值为8的财物。被盗的房屋上方用特殊符号标出。

### 参考代码

``` java
public int rob(int[] nums) {
    if (nums.length == 0) {
        return 0;
    }

    if (nums.length == 1) {
        return nums[0];
    }
	// 0~n-2
    int result1 = helper(nums, 0, nums.length - 2);
    // 1~n-1
    int result2 = helper(nums, 1, nums.length - 1);
    return Math.max(result1, result2);        
}

private int helper(int[] nums, int start, int end) {
    int[] dp = new int[2];
    dp[0] = nums[start];

    if (start < end) {
        dp[1] = Math.max(nums[start], nums[start + 1]);
    }

    for (int i = start + 2; i <= end; i++) {
        int j = i - start;
        dp[j%2] = Math.max(dp[(j-1) % 2], dp[(j-2) % 2] + nums[i]);
    }

    return dp[(end - start) % 2];
}
```

## 面试题91：粉刷房子

### 题目

一排n幢房子要粉刷成红、绿、蓝三种颜色，不同房子粉刷成不同颜色的成本不同。用一个n×3的数组表示n幢房子分别用三种颜色粉刷的成本。要求任意相邻的两幢房子的颜色都不一样，请计算粉刷这n幢的最少成本。

例如，粉刷3幢房子的成本分别为[[17, 2, 16], [15, 14, 5], [13, 3, 1]]，如果分别将这3幢房子粉刷成绿色、蓝色和绿色，那么粉刷的成本是10，是最小的成本。

### 参考代码

```java
r(i)是将i号房粉刷成r的成本;
r(i) = min(g(i-1),b(i-1)) + costs[i][0];
g(i) = min(r(i-1),b(i-1)) + costs[i][1];
b(i) = min(g(i-1),r(i-1)) + costs[i][2];
```



``` java
public int minCost(int[][] costs) {
    if (costs.length == 0) {
        return 0;
    }

    int[][] dp = new int[3][2];
    for (int j = 0; j < 3; j++) {
        dp[j][0] = costs[0][j];
    }

    // i是房号
    for (int i = 1; i < costs.length; i++) {
        for (int j = 0; j < 3; j++) {
            // j+1和j+2是与j不同颜色的房子
            int prev1 = dp[(j + 2) % 3][(i - 1) % 2];
            int prev2 = dp[(j + 1) % 3][(i - 1) % 2];
            dp[j][i % 2] = Math.min(prev1, prev2) + costs[i][j];
        }
    }

    int last = (costs.length - 1) % 2;
    return Math.min(dp[0][last], Math.min(dp[1][last], dp[2][last]));
}
```

## 面试题92：翻转字符

### [题目](https://leetcode.cn/problems/flip-string-to-monotone-increasing/)

输入一个只包含和'0'的'1'字符串，我们可以将其中的'0'的翻转成'1'，可以将'1'翻转成'0'。请问至少需要翻转几个字符，使得翻转之后的字符串中所有的'0'位于'1'的前面？翻转之后的字符串可能只含有'0'或者'1'。

例如，输入字符串"00110"，至少需要翻转1个字符才能使所有的'0'位于'1'的前面。我们可以将最后一个字符'0'的翻转成'1'，得到字符串"00111"。

### 参考代码

#### 解法一：动态规划

```
f0(i): s[0...i]翻转成以0为结尾的最小次数 => dp[0][i]
f1(i): s[0...i]翻转成以1为结尾的最小次数 => dp[1][i]
dp[0][i] = dp[0][i - 1] + (ch == '0' ? 0 : 1);
dp[1][i] = Math.min(dp[0][i - 1], dp[1][i - 1]) + (ch == '1' ? 0 : 1);
```



``` java
public int minFlipsMonoIncr(String S) {
    int len = S.length();
    if (len == 0) {
        return 0;
    }

    int[][] dp = new int[2][2];
    char ch = S.charAt(0);
    dp[0][0] = ch == '0' ? 0 : 1;
    dp[1][0] = ch == '1' ? 0 : 1;

    for (int i = 1; i < len; i++) {
        ch = S.charAt(i);
        int prev0 = dp[0][(i - 1) % 2];
        int prev1 = dp[1][(i - 1) % 2];
        dp[0][i % 2] = prev0 + (ch == '0' ? 0 : 1);
        dp[1][i % 2] = Math.min(prev0, prev1) + (ch == '1' ? 0 : 1);
    }

    return Math.min(dp[0][(len - 1) % 2], dp[1][(len - 1) % 2]);
}
```

#### 解法二***

```java
// 统计每个位置的前面有多少个1和后面有多少个0
// 只要改变左边的1和右边的0，改变次数之和就是结果
public int minFlipsMonoIncr(String s) {
    int ans = Integer.MAX_VALUE;
    int len = s.length();
    int[] zero = new int[len];
    int[] one = new int[len];
    int cnt0 = 0, cnt1 = 0;
    
    for (int i = 1; i < len; i++) {
        // i的左边有多少个1
        if (s.charAt(i - 1) == '1') cnt1++;
        one[i] = cnt1;
        // i的右边有多少个0
        if (s.charAt(len - i) == '0') cnt0++;
        zero[len - i - 1] = cnt0;
    }
    for (int i = 0; i < len; i++) {
        ans = Math.min(ans, one[i] + zero[i]);
    }
    return ans;
}
// 还可以直接计算前缀和for in 0:len
// 设前缀1为one，则i的前面有one[i-1]个1，
// 后面有one[len] - one[i]个1
// 只是最后判断的时候for in 1:len
// 单独判断i=0的情况。
```

## 补：最长递增子序列

### 题目

给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。

子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

```
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
```

### 解法一：dp - O(N^2)

```
dp[i]: 0~i的最长子序列长
dp[i]和dp[j]有关系，j < i

dp[i] = max(dp[i], dp[j] + 1) for j in [0, i)
```

```java
public int lengthOfLIS(int[] nums) {
    int[] dp = new int[nums.length];
    int res = 0;
    // 初始化为1，至少有递增子序列长度为1
    Arrays.fill(dp, 1);
    for (int i = 0; i < nums.length; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                // max是取dp[i]的最大，dp[i]是会变的
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        res = Math.max(dp[i], res);
    }
    return res;
}
```



### 解法二：dp+二分查找 - O(NlogN)

**降低复杂度切入点**： 解法一中，遍历计算 dp 列表需 O(N)，计算每个 dp[k] 需 O(N)。

- 动态规划中，通过线性遍历来计算 dp 的复杂度无法降低；

- 每轮计算中，需要通过线性遍历 [0,k) 区间元素来得到 dp[k] 。**考虑**：是否可以通过重新设计状态定义，**使整个 dp 为一个排序列表**；这样在计算每个 dp[k] 时，就可以通过二分法遍历 [0,k) 区间元素，将此部分复杂度由 O(N) 降至 O(logN)。

> tails[k]为长度为k+1的子序列最小的末尾元素
>
> 遍历nums，查找tails数组中第一个比num大元素的下标，即可插入的位置

```
比如
tails[] = 1 3 7
此时来了个5
3 < 5 < 7
5会覆盖掉7
1 3 5
再来个2
1 < 2 < 5

```



```java
public int lengthOfLIS(int[] nums) {
    int len = nums.length;
    int[] tails = new int[len];
    int res = 0;
    for (int num : nums) {
        int i = 0, j = res;
        while (i < j) {
            int mid = (i + j) / 2;
            if (tails[mid] < num) i = mid + 1;
            else j = mid;
        }
        // tails是每个长度的子序列最小的末尾元素
        // 为了使得后面的元素尽量都可以插进来
        // 如果没有到达j==res这个条件 
        // 就说明tail数组里只有部分比这个num要小 
        // 那么就把num插入到tail数组合适的位置即可 
        // 但是由于这样的子序列长度肯定是没有res长的 
        // 因此res不需要更新
        tails[i] = num;
        // 如果该元素比tails里存在的元素还大，
        // 说明有个新的更长的子序列
        // i == j，都一样
        // res是长度，比下标都大
        if (i == res) res++;
    }
    return res;
} 
```



## 面试题93：最长斐波那契数列

### 题目

输入一个没有重复数字的单调递增的数组，数组里至少有三个数字，请问数组里最长的斐波那契序列的长度是多少？例如，如果输入的数组是[1, 2, 3, 4, 5, 6, 7, 8]，由于其中最长的斐波那契序列是1、2、3、5、8，因此输出是5。

> 一种最长递增子序列的问题

### 参考代码

``` java
public int lenLongestFibSubseq(int[] A) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < A.length; i++) {
        map.put(A[i], i);
    }

    int[][] dp = new int[A.length][A.length];
    int result = 2;
    for (int i = 1; i < A.length; i++) {
        for (int j = 0; j < i; j++) {
            int k = map.getOrDefault(A[i] - A[j], -1);
            dp[i][j] = k >= 0 && k < j ? dp[j][k] + 1 : 2;

            result = Math.max(result, dp[i][j]);
        }
    }

    return result > 2 ? result : 0;
}
```

## 面试题94：最少回文分割

### 题目

输入一个字符串，请问至少需要分割几次使得分割出的每一个子字符串都是回文？例如，输入字符串"aaba"，至少需要分割1次，从两个相邻字符'a'中间切一刀将字符串分割成2个回文子字符串"a"和"aba"。

### 参考代码

``` java
public int minCut(String s) {
    int len = s.length();
    boolean[][] isPal = new boolean[len][len];
    for (int i = 0; i < len; i++) {
        for (int j = 0; j <= i; j++) {
            char ch1 = s.charAt(i);
            char ch2 = s.charAt(j);
            if (ch1 == ch2 && (i <= j + 1 || isPal[j + 1][i - 1])) {
                isPal[j][i] = true;
            }
        }
    }

    int[] dp = new int[len];
    for (int i = 0; i < len; i++) {
        if (isPal[0][i]) {
            dp[i] = 0;
        } else {
            dp[i] = i;
            for (int j = 1; j <= i; j++) {
                if (isPal[j][i]) {
                    dp[i] = Math.min(dp[i], dp[j - 1] + 1);
                }
            }
        }
    }

    return dp[len - 1];
}
```

## 14.2 双序列问题

## 面试题95：最长公共子序列

### [题目](https://leetcode.cn/problems/longest-common-subsequence/)

输入两个字符串，求出它们的最长公共子序列的长度。

如果从字符串s1中删除若干个字符之后能得到字符串s2，那么s2就是s1的一个子序列。例如，从字符串"abcde"中删除两个字符之后能得到字符串"ace"，因此"ace"是"abcde"的一个子序列。但字符串"aec"不是"abcde"的子序列。

```
如果输入字符串"abcde"和"badfe"，
它们的最长公共子序列是"bde"，
因此输出3。
```



### 参考代码

#### 解法一

```
dp[i][j]是s1的0~i-1和s2的0~j-1的最长公共子序列长度
int[][] dp = new int[len1 + 1][len2 + 1];
这样初始化简单
s1[i]==s2[j]: dp[i+1][j+1] = dp[i][j] + 1;
s1[i]!=s2[j]: 说明s1[i]和s2[j]不可能同时出现在s1_i-1和s2_j-1的公共子序列中。
```

``` java
public int longestCommonSubsequence(String text1, String text2) {
    int len1 = text1.length();
    int len2 = text2.length();
    int[][] dp = new int[len1 + 1][len2 + 1];
    for (int i = 0; i < len1; i++) {
        for (int j = 0; j < len2; j++) {
            if (text1.charAt(i) == text2.charAt(j)) {
                dp[i+1][j+1] = dp[i][j] + 1;
            } else {
                dp[i+1][j+1] = Math.max(dp[i][j+1], dp[i+1][j]);
            }
        }
    }

    return dp[len1][len2];
}
```

#### 解法二

``` java
public int longestCommonSubsequence(String text1, String text2) {
    int len1 = text1.length();
    int len2 = text2.length();
    if (len1 < len2) {
        return longestCommonSubsequence(text2, text1);
    }

    int[][] dp = new int[2][len2 + 1];
    for (int i = 0; i < len1; i++) {
        for (int j = 0; j < len2; j++) {
            if (text1.charAt(i) == text2.charAt(j)) {
                dp[(i+1)%2][j+1] = dp[i%2][j] + 1;
            } else {
                dp[(i+1)%2][j+1] = Math.max(dp[i%2][j+1],
                                            dp[(i+1)%2][j]);
            }
        }
    }

    return dp[len1%2][len2];
}
```

#### 解法三

``` java
public int longestCommonSubsequence(String text1, String text2) {
    int len1 = text1.length();
    int len2 = text2.length();
    if (len1 < len2) {
        return longestCommonSubsequence(text2, text1);
    }

    int[] dp = new int[len2 + 1];
    for (int i = 0; i < len1; i++) {
        int prev = dp[0];
        for (int j = 0; j < len2; j++) {
            int cur;
            if (text1.charAt(i) == text2.charAt(j)) {
                cur = prev + 1;
            } else {
                cur = Math.max(dp[j], dp[j + 1]);
            }

            prev = dp[j + 1];
            dp[j + 1] = cur;
        }
    }

    return dp[len2];        
}
```

## 面试题96：字符串交织

### 题目

输入三个字符串s1、s2、s3，请判断s3能不能由s1和s2交织而成，即s3的所有字符都是s1或s2的字符，s1和s2的字符都出现在s3中且相对位置不变。例如"aadbbcbcac"可以由"aabcc"和"dbbca"交织而成，如图14.5所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1405.png" alt="图2.1">

图14.5：一种交织"aabcc"和"dbbca"得到"aadbbcbcac"的方法。

### 参考代码

#### 解法一

``` java
public boolean isInterleave(String s1, String s2, String s3) {
    if (s1.length() + s2.length() != s3.length()) {
        return false;
    }

    boolean[][] dp = new boolean[s1.length() + 1][s2.length() + 1];
    dp[0][0] = true;

    for (int i = 0; i < s1.length(); i++) {
        dp[i + 1][0] = s1.charAt(i) == s3.charAt(i) && dp[i][0];
    }

    for (int j = 0; j < s2.length(); j++) {
        dp[0][j + 1] = s2.charAt(j) == s3.charAt(j) && dp[0][j];
    }

    for (int i = 0; i < s1.length(); i++) {
        for (int j = 0; j < s2.length(); j++) {
            char ch1 = s1.charAt(i);
            char ch2 = s2.charAt(j);
            char ch3 = s3.charAt(i + j + 1);
            dp[i + 1][j + 1] = (ch1 == ch3 && dp[i][j + 1])
                || ( ch2 == ch3 && dp[i + 1][j]);
        }
    }

    return dp[s1.length()][s2.length()];
}
```

#### 解法二

``` java
public boolean isInterleave(String s1, String s2, String s3) {
    if (s1.length() + s2.length() != s3.length()) {
        return false;
    }

    if (s1.length() < s2.length()) {
        return isInterleave(s2, s1, s3);
    }

    boolean[] dp = new boolean[s2.length() + 1];
    dp[0] = true;

    for (int j = 0; j < s2.length(); j++) {
        dp[j + 1] = s2.charAt(j) == s3.charAt(j) && dp[j];
    }

    for (int i = 0; i < s1.length(); i++) {
        dp[0] = dp[0] && s1.charAt(i) == s3.charAt(i);

        for (int j = 0; j < s2.length(); j++) {
            char ch1 = s1.charAt(i);
            char ch2 = s2.charAt(j);
            char ch3 = s3.charAt(i + j + 1);
            dp[j + 1] = (ch1 == ch3 && dp[j + 1])
                || ( ch2 == ch3 && dp[j]);
        }
    }

    return dp[s2.length()];
}
```

## 面试题97：子序列的数目

### [题目](https://leetcode.cn/problems/distinct-subsequences/)

输入字符串S和T，请计算S有多少个子序列等于T。例如，在字符串"appplep"中，有三个子序列等于字符串"apple"，如图14.6所示。 

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1406.png" alt="图2.1">

图14.6：字符串"appplep"中有三个子序列等于"apple"。

### 参考代码

#### 解法一

```
dp[i][j]: s[0:i-1]中t[0:j-1]个数，记作s_i, t_j
前i个，前j个

dp[i][j]显然要从dp[i-1][?]递推而来。
立即思考dp[i-1][j], dp[i-1][j-1]分别与dp[i][j]的关系。

若s[i]!=t[j]：
说明s[i]这个数没用，
即s_i-1和s_i中t_j的数量是一样的
dp[i][j] = dp[i-1][j]

若s[i]==t[j]：
1、s[i]选择与t[j]配对，
如果配对了，结果就和dp[i-1][j-1]一样
2、不配对
dp[i-1][j]
所以: dp[i][j] = dp[i-1][j-1] + dp[i-1][j]

综上
s[i]==t[j]: dp[i][j] = dp[i-1][j-1] + dp[i-1][j];
s[i]!=t[j]: dp[i][j] = dp[i-1][j];

```



``` java
public int numDistinct(String s, String t) {
    
    int[][] dp = new int[s.length() + 1][t.length() + 1];
    // ""和""匹配
    dp[0][0] = 1;

    for (int i = 0; i < s.length(); i++) {
        dp[i + 1][0] = 1;
        for (int j = 0; j <= i && j < t.length(); j++) {
            if (s.charAt(i) == t.charAt(j)) {
                dp[i + 1][j + 1] = dp[i][j] + dp[i][j + 1];
            } else {
                dp[i + 1][j + 1] = dp[i][j + 1];
            }
        }
    }

    return dp[s.length()][t.length()];
}
```

#### 解法二

``` java
public int numDistinct(String s, String t) {
    int[] dp = new int[t.length() + 1];
    if (s.length() > 0) {
        dp[0] = 1;
    }

    for (int i = 0; i < s.length(); i++) {
        for (int j = Math.min(i, t.length() - 1); j >= 0; j--) {
            if (s.charAt(i) == t.charAt(j)) {
                dp[j + 1] += dp[j];
            }
        }
    }

    return dp[t.length()];
}
```



## 14.3 矩阵的路径问题

## 面试题98：路径的数目

### 题目

一个机器人从m×n的格子的左上角出发，它每一步要么向下要么向右直到抵达格子的右下角。请计算机器人从左上角到达右下角的路径的数目。例如，如果格子的大小是3×3，那么机器人有6中符合条件的不同路径从左上角走到右下角，如图14.7所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1407.png" alt="图2.1">

图14.7：机器人在3×3的格子每一步只能向下或者向右，它从左上角到右下角有6条不同的路径。

### 参考代码

#### 解法一：递归

``` java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    return helper(m - 1, n - 1, dp);
}

private int helper(int i, int j, int[][] dp) {
    if (dp[i][j] == 0) {
        if (i == 0 || j == 0) {
            dp[i][j] = 1;
        } else {
            dp[i][j] = helper(i - 1, j, dp) + helper(i, j - 1, dp);
        }
    }

    return dp[i][j];
}
```

#### 解法二：迭代

``` java
public int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];
    Arrays.fill(dp[0], 1);
    for (int i = 1; i < m; i++) {
        dp[i][0] = 1;
    }

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i][j - 1] + dp[i-1][j];
        }
    }

    return dp[m - 1][n - 1];
}
```

#### 解法三

``` java
public int uniquePaths(int m, int n) {
    int[] dp = new int[n];
    Arrays.fill(dp, 1);

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[j] += dp[j - 1];
        }
    }

    return dp[n - 1];
}
```

## 面试题99：最小路径之和

### 题目

在一个m×n（m、n均大于0）的格子里每个位置都有一个数字。一个机器人每一步只能向下或者向右，请计算它从格子的左上角到右下角的路径的数字之和的最小值。例如，从图14.8中3×3的格子的左上角到右下角的路径的数字之和的最小值是8。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1408.png" alt="图2.1">

图14.8：机器人在3×3的格子中每一步只能向下或者向右，它从左上角右下角的路径的数字之和为8。数字之和最小的路径用灰色背景表示。

### 参考代码

#### 解法一

``` java
public int minPathSum(int[][] grid) {
    int[][] dp = new int[grid.length][grid[0].length];
    dp[0][0] = grid[0][0];
    for (int j = 1; j < grid[0].length; j++) {
        dp[0][j] = grid[0][j] + dp[0][j - 1];
    }

    for (int i = 1; i < grid.length; i++) {
        dp[i][0] = grid[i][0] + dp[i - 1][0];
        for (int j = 1; j < grid[0].length; j++) {
            int prev = Math.min(dp[i - 1][j], dp[i][j - 1]);
            dp[i][j] = grid[i][j] + prev;
        }
    }

    return dp[grid.length - 1][grid[0].length - 1];
}
```

#### 解法二

``` java
public int minPathSum(int[][] grid) {
    int[] dp = new int[grid[0].length];
    dp[0] = grid[0][0];
    for (int j = 1; j < grid[0].length; j++) {
        dp[j] = grid[0][j] + dp[j - 1];
    }

    for (int i = 1; i < grid.length; i++) {
        dp[0] += grid[i][0];
        for (int j = 1; j < grid[0].length; j++) {
            dp[j] = grid[i][j] + Math.min(dp[j], dp[j - 1]);
        }
    }

    return dp[grid[0].length - 1];
}
```

## 面试题100：三角形中最小路径之和

### 题目

在一个由数字组成的三角形中，第一行有1个数字，第二行有2个数字，以此类推第n行有n个数字。例如图14.9是一个包含4行数字的三角形。如果每一步我们只能前往下一行中相邻的数字，请计算从三角形顶部到底部的路径经过的数字之和的最小值。例如，图14.9中三角形从顶部到底部的路径数字之和的最小值为11，对应的路径经过的数字用阴影表示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/1409.png" alt="图2.1">

图14.9：一个包含4行数字的三角形。从三角形的顶部到底部的路径数字之和的最小值为11，对应的路径经过的数字用阴影表示。



### 参考代码

#### 解法一

``` java
public int minimumTotal(List<List<Integer >> triangle) {
    int size = triangle.size();
    int[][] dp = new int[size][size];
    for (int i = 0; i < size; ++i) {
        for (int j = 0; j <= i; ++j) {
            dp[i][j] = triangle.get(i).get(j);
            if (i > 0 && j == 0) {
                dp[i][j] += dp[i - 1][j];
            } else if (i > 0 && i == j) {
                dp[i][j] += dp[i - 1][j - 1];
            } else if (i > 0) {
                dp[i][j] += Math.min(dp[i - 1][j], dp[i - 1][j - 1]);
            }
        }
    }

    int min = Integer.MAX_VALUE;
    for (int num : dp[size - 1]) {
        min = Math.min(min, num);
    }

    return min;
}
```

#### 解法二

``` java
public int minimumTotal(List<List<Integer>> triangle) {
    int[] dp = new int[triangle.size()];
    for (List<Integer> row : triangle) {
        for (int j = row.size() - 1; j >= 0; --j) {
            if (j == 0) {
                dp[j] += row.get(j);
            } else if (j == row.size() - 1) {
                dp[j] = dp[j - 1] + row.get(j);
            } else {
                dp[j] = Math.min(dp[j], dp[j - 1]) + row.get(j);
            }
        }
    }

    int min = Integer.MAX_VALUE;
    for (int num : dp) {
        min = Math.min(min, num);
    }

    return min;
}
```



## 14.4 背包问题

## 14.4.1 01背包

有n件物品和一个最多能背重量为w 的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

### 例子

背包最大重量为4，物品为：

|       | 重量 | 价值 |
| :---- | ---- | :--- |
| 物品1 | 1    | 15   |
| 物品2 | 3    | 20   |
| 物品3 | 4    | 30   |

问背包能背的物品都最大价值是多少？

### 解法

```
dp[i][j] 表示从下标为[0-i]的物品里任意取，放进容量为j的背包，价值总和最大是多少。
```

| 背包重量-> | 0    | 1    | 2    | 3    | 4    |
| ------- | ---- | ---- | ---- | ---- | ---- |
| 物品0（不放） |      |      |      |      |      |
| 物品1         |      |      |      |      |      |
| 物品2         |      |      |      |      |      |
| 物品3         |      |      |      |      |      |




- **不放物品i**

```
由dp[i - 1][j]推出，即背包容量为j，
里面不放物品i的最大价值，此时dp[i][j]就是dp[i - 1][j]。
不放物品有两种情况：1、重量不足；2、主动不放
```

- **放物品i**

```
由dp[i - 1][j - weight[i]]推出，
dp[i - 1][j - weight[i]] 为背包容量为j - weight[i]的时候不放物品i的最大价值，
那么dp[i - 1][j - weight[i]] + value[i] （物品i的价值），
就是背包放物品i得到的最大价值
```

- 综上

`dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])`

```java
//遍历顺序：先遍历物品，再遍历背包容量
for (int i = 1; i <= wlen; i++){
    for (int j = 1; j <= bagsize; j++){
        if (j < weight[i - 1]){
            // 重量不足
            dp[i][j] = dp[i - 1][j];
        }else{
            // 主动不放和放之间选一个最大的
            // weight[i - 1]和value[i - 1]都是第i个的意思
            dp[i][j] = Math.max(dp[i - 1][j], 
                   dp[i - 1][j - weight[i - 1]] + value[i - 1]);
        }
    }
}
```
- `dp[j]：容量为j的背包，所背的物品价值可以最大为dp[j]。`

```
dp[j]可以通过dp[j - weight[i]]推导出来，
dp[j - weight[i]]表示容量为j - weight[i]的背包所背的最大价值。

dp[j - weight[i]] + value[i] 表示 
容量为 j - 物品i重量 的背包 加上 物品i的价值。
（也就是容量为j的背包，放入物品i了之后的价值即：dp[j]）
```



```java
// 一维，物品从0开始就可以，数组遍历只和 j 有关
for(int i = 0; i < weight.size(); i++) { // 遍历物品
    // >=是为了确保可以放进去
    // 倒序是因为，用到了左上角的数据，从右往左才不会覆盖掉左上角的数据
    for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量
        // 取自己dp[j] 相当于 二维dp数组中的dp[i-1][j]
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
    }
}
```





## 面试题101：分割等和子集

### 题目

给你一个非空的正整数数组，请判断能否将这些数字分成和相等的两部分。例如，如果输入数组为[3, 4, 1]，这些数字分成[3, 1]和[4]两部分，因此输出true；如果输入数组为[1, 2, 3, 5]，则不能将这些数字分成和相等的两部分，因此输出false。

> - 背包体积：sum / 2
> - 背包要放入的商品（集合里的元素）重量为**元素的数值**，价值也是
> - 正好装满，则找到了sum / 2 的子集
> - 每一个元素都不可重复放入

### 参考代码

### 模板-最大的价值

```java
// dp[j]表示 背包总容量是j，最大可以凑成j的子集总和为dp[j]
// dp[0] = 0;
// 如果如果题目给的价值都是正整数那么非0下标都初始化为0就可以了，
// 如果题目给的价值有负数，那么非0下标就要初始化为负无穷。
for (int num : nums) {
    for (int j = target; j >= num; j--) {
        dp[j] = Math.max(dp[j], dp[j - num] + num);
    }
}
```



#### 传统01背包解法

```java
// dp[i][j]表示 从前 i 个（下标为[0,.,i - 1]）的物品里任意取，放进容量为j的背包，价值总和最大是多少。
// 二维数组
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        if (sum % 2 == 1) return false;
        int target = sum / 2;
        int len = nums.length;
        int[][] dp = new int[len + 1][target + 1];
        // nums[i - 1]就是第i个
        for (int i = 1; i <= len; i++) {
            for (int j = 1; j <= target; j++) {
                // 不放（放不下）
                dp[i][j] = dp[i - 1][j];
                // 可以放（再考虑放还是不放）
                if (j >= nums[i - 1]) {
                    dp[i][j] = Math.max(dp[i][j], 
                            dp[i - 1][j - nums[i - 1]] + nums[i - 1]);
                } 
            }
        }
        return dp[len][target] == target;
    }
}
```


```java
// dp[j]表示 背包总容量是j，最大可以凑成j的子集总和为dp[j]
// 一位数组
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        if (sum % 2 == 1) return false;
        int target = sum / 2;
        int len = nums.length;
        int[] dp = new int[target + 1];

        for (int num : nums) {
            for (int j = target; j >= 1; j--) {
                // dp[i][j] = dp[i - 1][j] -> dp[j] = dp[j];
                if (j >= num) {
                    dp[j] = Math.max(dp[j], dp[j - num] + num);
                }
            }
        }
        /*
        // 这样也是对的，
        for (int num : nums) {
            for (int j = target; j >= num; j--) {
            	dp[j] = Math.max(dp[j], dp[j - num] + num);
            }
        }
        */
        
        return dp[target] == target;
    }
}
```



#### 解法二

``` java
public boolean canPartition(int[] nums) {
    int sum = 0;
    for (int num : nums) {
        sum += num;
    }

    if (sum % 2 == 1) {
        return false;
    }

    return subsetSum(nums, sum / 2);
}
// 二维
private boolean subsetSum(int[] nums, int target) {
    boolean[][] dp = new boolean[nums.length + 1][target + 1];
    for (int i = 0; i <= nums.length; i++) {
        dp[i][0] = true;
    }

    for (int i = 1; i <= nums.length; i++) {
        for (int j = 1; j <= target; j++) {
            dp[i][j] = dp[i - 1][j];
            if (!dp[i][j] && j >= nums[i - 1]) {
                dp[i][j] =  dp[i - 1][j - nums[i - 1]];
            }
        }
    }

    return dp[nums.length][target];
}
```


```java
// 一维
private boolean subsetSum(int[] nums, int target) {
    boolean dp[] = new boolean[target + 1];
    dp[0] = true;

    for (int i = 1; i <= nums.length; i++) {
        for (int j = target; j > 0; --j) {
            if (!dp[j] && j >= nums[i - 1]) {
                dp[j] = dp[j - nums[i - 1]];
            }
        }
    }

    return dp[target];
}
```



## 面试题102：加减的目标值（目标和）

### 题目

给你一个非空的正整数数组和一个目标值S，如果给每个数字添加‘+’或者‘-’运算符，请计算有多少种方法使得这个整数的计算结果为S。

例如，如果输入数组[2, 2, 2]并且S等于2，有三种添加‘+’或者‘-’的方法，使得结果为2，它们分别是2+2-2=2、2-2+2=2以及-2+2+2=2。

> - 令 ‘+’ 的数字和为p，‘-’ 的数字和为q， `p - q = S, p + q = sum`
> - `p = (S + sum) / 2`
> - 即找出数组中和为`(S + sum) / 2`的数字，**装满容量为p的背包，有几种方法**

### 模板-装满背包有多少种方法

> 例如：dp[j]，j 为5，
>
> - 已经有一个1（nums[i]） 的话，有 dp[4]种方法 凑成 dp[5]。
> - 已经有一个2（nums[i]） 的话，有 dp[3]种方法 凑成 dp[5]。
> - 已经有一个3（nums[i]） 的话，有 dp[2]中方法 凑成 dp[5]
> - 已经有一个4（nums[i]） 的话，有 dp[1]中方法 凑成 dp[5]
> - 已经有一个5 （nums[i]）的话，有 dp[0]中方法 凑成 dp[5]
>
> 累加就是`dp[j] += dp[j - num];`

```java
// dp[j]表示 填满 j 容量的背包，有dp[j]种方法 
for (int num : nums) {
    for (int j = target; j >= num; --j) {
        dp[j] += dp[j - num];
    }
}
```



### 参考代码

**二维状态转移方程**

<img src = "https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/MommyTalk1648001536829.png">

**一维解法**

``` java
public int findTargetSumWays(int[] nums, int S) {
    int sum = 0;
    for (int num : nums) {
        sum += num;
    }

    if ((sum + S) % 2 == 1 || sum < S) {
        return 0;
    }

    return subsetSum(nums, (sum + S) / 2);
}

private int subsetSum(int[] nums, int target) {
    int dp[] = new int[target + 1];
    // 装满 0 容量的背包有一种方法，即啥也不加
    dp[0] = 1;

    for (int num : nums) {
        for (int j = target; j >= num; --j) {
            dp[j] += dp[j - num];
        }
    }

    return dp[target];
}
```



## [二维01背包问题](https://programmercarl.com/0474.%E4%B8%80%E5%92%8C%E9%9B%B6.html#_474-%E4%B8%80%E5%92%8C%E9%9B%B6)

### 题目

给你一个二进制字符串数组 strs 和两个整数 m 和 n 。

请你找出并返回 strs 的最大子集的大小，该子集中 最多 有 m 个 0 和 n 个 1 。

如果 x 的所有元素也是 y 的元素，集合 x 是集合 y 的 子集 。

```
示例 1：
输入：strs = ["10", "0001", "111001", "1", "0"], m = 5, n = 3 
输出：4
解释：最多有 5 个 0 和 3 个 1 的最大子集是 {"10","0001","1","0"} ，
因此答案是 4 。 
其他满足题意但较小的子集包括 {"0001","1"} 和 {"10","1","0"} 。
{"111001"} 不满足题意，因为它含 4 个 1 ，大于 n 的值 3 。

示例 2： 
输入：strs = ["10", "0", "1"], m = 1, n = 1 
输出：2 
解释：最大的子集是 {"0", "1"} ，所以答案是 2 。

```



> **本题中strs 数组里的元素就是物品，每个物品都是一个！**
>
> **而m 和 n相当于是一个背包，两个维度的背包**。

```java
// dp[i][j]：最多有i个0和j个1的strs的最大子集的大小为dp[i][j]。
public int findMaxForm(String[] strs, int m, int n) {
    //dp[i][j]表示i个0和j个1时的最大子集
    int[][] dp = new int[m + 1][n + 1];
    int oneNum, zeroNum;
    for (String str : strs) {
        oneNum = 0;
        zeroNum = 0;
        for (char ch : str.toCharArray()) {
            if (ch == '0') {
                zeroNum++;
            } else {
                oneNum++;
            }
        }
        //倒序遍历
        for (int j = m; j >= zeroNum; j--) {
            for (int k = n; k >= oneNum; k--) {
                dp[j][k] = Math.max(dp[j][k], 
                         dp[j - zeroNum][k - oneNum] + 1);
            }
        }
    }
    return dp[m][n];
}
```



## 14.4.2 完全背包

> **如果求组合数就是外层for循环遍历物品，内层for遍历背包**。
>
> **如果求排列数就是外层for遍历背包，内层for循环遍历物品**。

```java
// 物品是可以添加多次的，所以要从小到大去遍历
// 与01背包相反，01背包正序遍历的话会多次存放，相当于用了覆盖之后的数据
// 覆盖之后的数据就是添加了前面元素之后
// 而倒序，使用的数据都是遍历这一层之前的

// 先遍历物品，再遍历背包（组合） 
for(int i = 0; i < weight.size(); i++) { // 遍历物品
    for(int j = weight[i]; j <= bagWeight ; j++) { // 遍历背包容量
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
    }
}
// 只有 j >= weight[i]才能放进去，不够放的话，之前的数值不会改变

// （排列）
for (int i = 1; i <= target; ++i) {
    for (int num : nums) {
        if (i >= num) {
            dp[i] += dp[i - num];
        }
    }
}
```



**遍历顺序的区别**

```java
// 组合：先遍历物品再遍历背包
for (int i = 0; i < nums.length; i++) {
	for (int j = nums[i]; j <= target; j++) {
        dp[j] += dp[j - nums[i]];
    }
}
```

假设：nums[0] = 1，nums[1] = 5。

那么就是先把1加入计算，然后再把5加入计算，得到的方法数量只有{1, 5}这种情况。而不会出现{5, 1}的情况。

```java
// 排列：先遍历背包再遍历物品
for (int j = 1; j <= target; j++) {
    for (int i = 0; i < nums.length; i++) {
        if (j - nums[i] >= 0) {
            dp[j] += dp[j - nums[i]];
        }
    }
}
```

背包容量的每一个值，都是经过 1 和 5 的计算，包含了{1, 5} 和 {5, 1}两种情况。



## 面试题103：最少的硬币数目

### 题目

给你正整数数组coins表示硬币的面额和一个目标总额t，请计算凑出总额t至少需要的硬币数目。每种硬币可以使用任意多枚。如果不能用输入的硬币凑出给定的总额，则返回-1。例如，如果硬币的面额为[1, 3, 9, 10]，总额t为15，那么至少需要3枚硬币，即2枚面额为3的硬币以及1枚面额为9的硬币。

> - `dp[j] = min(dp[j - coins[i]] + 1, dp[j]);`

### 参考代码

``` java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    // 凑出amount不可能用到 amount + 1 个硬币，币值最低为1
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    for (int coin : coins) {
        for (int j = coin; j <= amount; j++) {
            dp[j] = Math.min(dp[j], dp[j - coin] + 1);
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```



## 最少的硬币数目2

给定不同面额的硬币和一个总金额。写出函数来计算可以凑成总金额的硬币组合数。假设每一种面额的硬币有无限个。

示例 1:输入: amount = 5, coins = [1, 2, 5] 输出: 4 解释: 有四种方式可以凑成总金额: 

5=5 

5=2+2+1 

5=2+1+1+1 

5=1+1+1+1+1

> 组合 + 满包

```java
public int change(int amount, int[] coins) {
    //递推表达式
    int[] dp = new int[amount + 1];
    //初始化dp数组，表示金额为0时只有一种情况，也就是什么都不装
    dp[0] = 1;
    for (int i = 0; i < coins.length; i++) {
        for (int j = coins[i]; j <= amount; j++) {
            dp[j] += dp[j - coins[i]];
        }
    }
    return dp[amount];
}
```



## 面试题104：排列的数目

### 题目

给你一个非空的正整数数组nums和一个目标值t，数组中所有数字都是唯一的，请计算数字之和等于t的所有排列的数目。数组中的数字**可以在排列中出现任意次**。

```
例如，输入数组[1, 2, 3]并且t为3，
那么总共由4个排序的数字之和等于3，
它们分别为{1, 1, 1}、{1, 2}、{2, 1}以及{3}。
```



### 参考代码

``` java
public int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;

    for (int i = 1; i <= target; ++i) {
        for (int num : nums) {
            if (i >= num) {
                dp[i] += dp[i - num];
            }
        }
    }

    return dp[target];
}
```



## 14.4.3 多重背包

```java
public void testMultiPack1(){
    // 版本一：改变物品数量为01背包格式
    List<Integer> weight = new ArrayList<>(Arrays.asList(1, 3, 4));
    List<Integer> value = new ArrayList<>(Arrays.asList(15, 20, 30));
    List<Integer> nums = new ArrayList<>(Arrays.asList(2, 3, 2));
    int bagWeight = 10;

    for (int i = 0; i < nums.size(); i++) {
        while (nums.get(i) > 1) { // 把物品展开为i
            weight.add(weight.get(i));
            value.add(value.get(i));
            nums.set(i, nums.get(i) - 1);
        }
    }

    int[] dp = new int[bagWeight + 1];
    for(int i = 0; i < weight.size(); i++) { // 遍历物品
        for(int j = bagWeight; j >= weight.get(i); j--) { // 遍历背包容量
            dp[j] = Math.max(dp[j], dp[j - weight.get(i)] + value.get(i));
        }
        System.out.println(Arrays.toString(dp));
    }
}

public void testMultiPack2(){
    // 版本二：改变遍历个数
    int[] weight = new int[] {1, 3, 4};
    int[] value = new int[] {15, 20, 30};
    int[] nums = new int[] {2, 3, 2};
    int bagWeight = 10;

    int[] dp = new int[bagWeight + 1];
    for(int i = 0; i < weight.length; i++) { // 遍历物品
        for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量
            // 以上为01背包，然后加一个遍历个数
            for (int k = 1; k <= nums[i] && (j - k * weight[i]) >= 0; k++) { // 遍历个数
                dp[j] = Math.max(dp[j], dp[j - k * weight[i]] + k * value[i]);
            }
            System.out.println(Arrays.toString(dp));
        }
    }
}
```



