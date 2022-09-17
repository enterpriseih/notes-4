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

## 跳楼梯

### 题目

假设一共有n阶，同样共有f(n)种跳法，那么这种情况就比较多，最后一步超级蛙可以从n-1阶往上跳，也可以n-2阶，也可以n-3…等等等

### 题解

```
f(n) = f(n-1) + f(n-2) + ... + f(2) + f(1)
f(n-1) = f(n-2) + f(n-3) + ... + f(2) + f(1)
=> f(n) = f(n-1) + f(n-1) = 2 * f(n-1) = 2^(n-1)

```



```java
public int solution(int n) {
    if (n == 1) return 1;
    return 2 * solution(n-1);
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
dp[i]: 以nums[i]结尾的最长子序列长
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

## 补：股票买卖问题Ⅰ

### 题目

给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

```
输入：[7,1,5,3,6,4]
输出：5
解释：
在第 2 天（股票价格 = 1）的时候买入，
在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；
同时，你不能在买入前卖出股票。
```

### 题解一

```java
// 遍历一次，nums[i] - nums[0:i]的最小值
public int maxProfit(int[] prices) {
    int res = 0;
    int min = Integer.MAX_VALUE;
    for (int price : prices) {
        min = Math.min(min, price);
        res = Math.max(res, price - min);
    }
    return res;
}
```

### 题解二：主要为了学dp

```
dp[i][0]: 第i天持有(!=买入)股票所得最多现金
假设一开始现金为0，则第i天买入股票后所得现金为-prices[i]
dp[i][1]: 第i天不持有股票所得最多现金

如果第i天持有股票即dp[i][0]， 那么可以由两个状态推出来
- 第i-1天就持有股票，那么就保持现状，所得现金就是昨天持有股票的所得现金 即：dp[i - 1][0]
- 第i天买入股票，所得现金就是买入今天的股票后所得现金即：-prices[i]
那么dp[i][0]应该选所得现金最大的，所以dp[i][0] = max(dp[i - 1][0], -prices[i]);

如果第i天不持有股票即dp[i][1]， 也可以由两个状态推出来
- 第i-1天就不持有股票，那么就保持现状，所得现金就是昨天不持有股票的所得现金 即：dp[i - 1][1]
- 第i天卖出股票，所得现金就是按照今天股票佳价格卖出后所得现金即：prices[i] + dp[i - 1][0]
同样dp[i][1]取最大的，dp[i][1] = max(dp[i - 1][1], prices[i] + dp[i - 1][0]);

return dp[len][1]
不持有股票才说明已经卖出去了，这样的钱才会多

由递推公式 dp[i][0] = max(dp[i - 1][0], -prices[i]); 
和 dp[i][1] = max(dp[i - 1][1], prices[i] + dp[i - 1][0]);
可以看出，其基础都是要从dp[0][0]和dp[0][1]推导出来。
那么dp[0][0]表示第0天持有股票，此时的持有股票就一定是买入股票了，
因为不可能由前一天推出来，所以dp[0][0] -= prices[0];
dp[0][1]表示第0天不持有股票，不持有股票那么现金就是0，所以dp[0][1] = 0;
```



```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) return 0;
    int length = prices.length;
    // dp[i][0]代表第i天持有股票的最大收益
    // dp[i][1]代表第i天不持有股票的最大收益
    int[][] dp = new int[length][2];
    int result = 0;
    dp[0][0] = -prices[0];
    dp[0][1] = 0;
    for (int i = 1; i < length; i++) {
        
        dp[i][0] = Math.max(dp[i - 1][0], -prices[i]);
        dp[i][1] = Math.max(dp[i - 1][0] + prices[i], dp[i - 1][1]);
    }
    return dp[length - 1][1];
}
```

## 补：股票买卖Ⅱ

```java
// 如果可以购买多次，且同一时间只能持有一张，则
dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] - prices[i]);
// 因为一只股票可以买卖多次，所以当第i天买入股票的时候，
// 所持有的现金可能有之前买卖过的利润。
```



## 补：股票买卖Ⅲ

**在一的基础上最多买两次**

一共五个状态

1、没有操作

2、第一次买入

3、第一次卖出

4、第二次买入

5、第二次卖出

```
dp[i][j]中 i表示第i天，j为 [0 - 4] 五个状态，
dp[i][j]表示第i天状态j所剩最大现金。
```



```java
public int maxProfit(int[] prices) {
    int len = prices.length;
    int[] dp = new int[5];
    dp[1] = -prices[0]; // 第一天买入
    dp[3] = -prices[0]; // 第一次卖出后是0，然后再买入即-prices[0]

    for (int i = 1; i < len; i++) {
        dp[1] = Math.max(dp[1], dp[0] - prices[i]);
        dp[2] = Math.max(dp[2], dp[1] + prices[i]);
        dp[3] = Math.max(dp[3], dp[2] - prices[i]);
        dp[4] = Math.max(dp[4], dp[3] + prices[i]);
    }

    return dp[4];
}
```





## 14.2 双序列问题

## 面试题95：最长公共子序列/不相交的线

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

```
f(i,j)表示s1[0:i]和s2[0:j]能否组成s3[0:i+j+1]
if s3[i+j+1] == s1[i]
    f(i,j) = f(i-1,j)
if s3[i+j+1] == s2[j]
    f(i,j) = f(i,j-1)
if s3[i+j+1] == s1[i] == s2[j]
    f(i,j) = f(i-1,j) || f(i,j-1)

```



``` java
public boolean isInterleave(String s1, String s2, String s3) {
    // 首先判断s1和s2的长度相加是否是s3
    if (s1.length() + s2.length() != s3.length()) {
        return false;
    }
    
    boolean[][] dp = new boolean[s1.length() + 1][s2.length() + 1];
    // dp[0][0] 代表的是s1为“”， s2为“”，s3为“”。
    dp[0][0] = true;
	
    // 单独看s1能否组成s3
    for (int i = 0; i < s1.length(); i++) {
        dp[i + 1][0] = s1.charAt(i) == s3.charAt(i) && dp[i][0];
    }
    // 单独看s2能否组成s3
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



## 补：编辑距离

### 题目

给你两个单词 word1 和 word2， 请返回将 word1 转换成 word2 所使用的最少操作数。

你可以对一个单词进行如下三种操作：

- 插入一个字符

- 删除一个字符
- 替换一个字符

```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```



### 题解

```
dp[i][j]表示以下标i-1为结尾的字符串word1，和以下标j-1为结尾的字符串word2，
最近编辑距离为dp[i][j]。

if (word1[i - 1] == word2[j - 1])
    不操作
    dp[i][j] = dp[i-1][j-1]
if (word1[i - 1] != word2[j - 1])
    1、增
    说明word1[0:i-1]与word2[0:j-2]刚好匹配，
    此时加上一个word2[j-1]，就是word1[0:i-1]与word2[0:j-1]匹配了
    dp[i][j] = dp[i][j-1] + 1;
    
    2、删
    word1删除一个，相当于，不考虑word1[i-1]
    拿word1[0:i-2](dp[i-1][?])与word2[0:j-1](dp[?][j])匹配
    dp[i][j] = dp[i-1][j] + 1;
    
    3、换
    dp[i][j] = dp[i-1][j-1] + 1;

```



```java
public int minDistance(String word1, String word2) {
    int m = word1.length();
    int n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    // 初始化
    for (int i = 1; i <= m; i++) {
        dp[i][0] =  i;
    }
    for (int j = 1; j <= n; j++) {
        dp[0][j] = j;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            // 因为dp数组有效位从1开始
            // 所以当前遍历到的字符串的位置为i-1 | j-1
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = Math.min(Math.min(dp[i - 1][j - 1], dp[i][j - 1]), dp[i - 1][j]) + 1;
            }
        }
    }
    return dp[m][n];
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
int[] dp = new int[bagWeight + 1];
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
            for (int j = target; j >= num; j--) {
            	// dp[i][j] = dp[i - 1][j] -> dp[j] = dp[j];
            	dp[j] = Math.max(dp[j], dp[j - num] + num);
            }
        }
                
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



## 区间DP

模板

```java
// 区间总长n
for (int len = 1; len <= n; len++) {
    // 区间右侧不能超出数组范围
    // 枚举左端点
    for (int left = 1; left + len - 1 <= n; left++) {
        int right = left + len - 1;
        // 断点将区间分为了[left, k-1], k, [k+1, right]
        for (int k = left; k <= right; k++) {
        
        }
    }
}
```





## 加分二叉树

### 题目

设一个 n 个节点的二叉树 tree 的中序遍历为（1,2,3,…,n），其中数字 1,2,3,…,n 为节点编号。

每个节点都有一个分数（均为正整数），记第 i 个节点的分数为 di，tree 及它的每个子树都有一个加分，任一棵子树 subtree（也包含 tree 本身）的加分计算方法如下：

subtree的左子树的加分 × subtree的右子树的加分 ＋ subtree的根的分数

若某个子树为空，规定其加分为 1。

叶子的加分就是叶节点本身的分数，不考虑它的空子树。

试求一棵符合中序遍历为（1,2,3,…,n）且加分最高的二叉树 tree。

要求输出：

（1）tree的最高加分

（2）tree的前序遍历

```
输入：中序遍历
5 
5 7 1 2 10

输出：
145
3 1 2 4 5
```

### 题解

```
因为是中序遍历，从一个断点开始，左右刚好就是左子树和右子树
求max(左子树*右子树+root) => 求左max、右max
dp[l][r]: 区间[l,r]的max
```



```java
int total;
List<Integer> preorderTraversal = new ArrayList<>();

public void scoreTree(int[] scores, int n) {
    int[][] dp = new int[n + 1][n + 1];
    // root存放的是区间[left, right]的最大值对应的根结点
    int[][] root = new int[n + 1][n + 1];
    // 区间dp，三层for，外层区间长度，中层左端点，内层断点取最优
    // len是区间长度
    for (int len = 1; len <= n; len++) {
        // 区间右侧不能超出数组范围
        // 枚举左端点
        for (int left = 1; left + len - 1 <= n; left++) {
            int right = left + len - 1;
            int max = 0;
            // 断点将区间分为了[left, k-1], k, [k+1, right]
            for (int k = left; k <= right; k++) {
                int sum;
                // 断点为左端点，说明没有左子树，则左子树的值为1
                // 否则为[left, k-1]的最大值，即为dp
                int leftValue = (k == left) ? 1 : dp[left][k - 1];
                int rightValue = (k == right) ? 1 : dp[k + 1][right];

                // 长度为1，分值总和就是当前断点自身的分值
                if (len == 1) {
                    sum = scores[k - 1];
                } else {
                    // scores[k-1]是因为k从1开始，scores从0开始
                    sum = scores[k - 1] + leftValue * rightValue;
                }
                // 更新根结点和dp
                if (sum > max) {
                    root[left][right] = k;
                    max = sum;
                    dp[left][right] = max;
                }
            }
        }
    }
    preorderDfs(root, 1, n);
    total = dp[1][n];
}
// 中序节点号为1，2，3，4，5
// 假设3为当前的根，则左子树就是[1,3-1]范围的点
public void preorderDfs(int[][] root, int left, int right) {
    if (left > right) return;
    preorderTraversal.add(root[left][right]);
    // 左子树
    preorderDfs(root, left, root[left][right] - 1);
    // 右子树
    preorderDfs(root, root[left][right] + 1, right);
}

```



## 状态压缩dp

状压 DP，就是专指状态压缩 DP，这一类 DP 的状态一般都特别复杂，所以常常直接用多进制的方式把复杂的状态直接变成一个正整数的形式，达到状态压缩的目的。

常用的位运算

```
x&(1<<(i-1))!=0 查询第i位上的元素是否为1
 
x|=(1<<(i-1)) 把第i位上的数修改为1

x&=~(1<<(i-1)) 把第i位上的数修改为0

x^=(1<<(i-1)) 翻转第i位上的数，0变1，1变0
```



## [最美子字符串](https://leetcode.cn/problems/number-of-wonderful-substrings/)

### 题目

如果某个字符串中 **至多一个** 字母出现 **奇数** 次，则称其为 **最美** 字符串。

- 例如，`"ccjjc"` 和 `"abab"` 都是最美字符串，但 `"ab"` 不是。
- 只有 a - j 小写的10个字母

```
输入：word = "aabb"
输出：9
解释：9 个最美子字符串如下所示：
- "aabb" -> "a"
- "aabb" -> "aa"
- "aabb" -> "aab"
- "aabb" -> "aabb"
- "aabb" -> "a"
- "aabb" -> "abb"
- "aabb" -> "b"
- "aabb" -> "bb"
- "aabb" -> "b"

```



### 题解

如果字符串 word 的某个子串 word[i, j] 是最美字符串，那么其中最多只有一个字符出现奇数次，这说明：

> 对于任意一次字符 c 而言，word 的 i−1 前缀 word[0, i−1] 与 j 前缀 word[0, j] 中字符 c 的出现次数必须同奇偶。
>
> - 奇数 - 奇数 = 偶数 - 偶数 = 偶数
>
> 同时，我们最多允许有一个字符 c，它**在两个前缀中出现次数的奇偶性不同**。
>
> - 奇数 - 偶数 = 奇数

```
使用一个二进制数bit记录原字符串的每个前缀中各个字母的奇偶性
bit的第i位为1说明第i个字母出现了奇数次，0表示偶数次。

记word[0, k]对应的二进制数为bit_k
则word[i, j]是最美字符，当且仅当bit_{i-1}和bit_j最多只有一位不同

如果i = 0, bit_{-1}表示所有字母均未出现

```



```java
public long wonderfulSubstrings(String word) {
    int n = word.length();
    int bit = 0;
    Map<Integer, Integer> map = new HashMap<>();
    // 存(0, 1), 所有字符都具有空前缀
    map.put(bit, 1);
    long res = 0;
    for(int i = 0; i < n; i++){
        // 翻转，当前位从0变1，从1变0，其余位不变 
        bit ^= 1 << (word.charAt(i) - 'a');
        // 10个字母就是10，26个字母就是26，超过32就存不下了
        for(int j = 0; j < 10; j++){
            // 查找前缀有没有和当前bit只有一个字母出现的奇偶性不同的字符二进制
            // 有几个就能组成几组字符
            // bit ^ (1 << j)只是是第j个字母的奇偶性不同
            res += map.getOrDefault(bit ^ (1 << j), 0);
        }
        // 前缀中有和当前bit一样的奇偶性的
        // 如果这题是有且只有一个是奇数次，则该句删除
        res += map.getOrDefault(bit, 0);
        // 将该bit放入
        map.put(bit, map.getOrDefault(bit, 0) + 1);
    }
    return res;
}
```





## 树形dp

https://www.cnblogs.com/wujianxiang/p/15103725.html

## 没有上司的舞会

### 题目

有个公司要举行一场晚会。
为了能玩得开心，公司领导决定：如果邀请了某个人，那么一定不会邀请他的上司
（上司的上司，上司的上司的上司……都可以邀请）。

每个参加晚会的人都能为晚会增添一些气氛，求一个邀请方案，使气氛值的和最大。

```
输入: 
第1行一个整数N（1<=N<=6000）表示公司的人数。
接下来N行每行一个整数。第i行的数表示第i个人的气氛值x(-128<=x<=127)。
接下来每行两个整数L，K。表示第K个人是第L个人的上司。

输出: 气氛最大值

例如
输入: 
7
1
1
1
1
1
1
1
1 3
2 3
6 4
7 4
4 5
3 5

输出: 5
```



### 题解

```
f[p][0/1]: 从p的子节点中选的方案的max，0表示不选p，1表示选p

1- 不选父，则既可以选子，也可以不选子
f[p][0] = sum(max{f[s][0], f[s][1]}

2- 选父，则只能不选子
f[p][1] = w[p] + sum(f[s][0])
```



```java
public class TreeDP {
    static int[][] dp;
    static int[] happy;
    static Map<Integer, List<Integer>> tree = new HashMap<>();
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int N = in.nextInt();
        dp = new int[N][2];
        happy = new int[N];
        boolean[] hasFather = new boolean[N];
        for (int i = 0; i < N; i++) {
            happy[i] = in.nextInt();
            // dp[i][1] = happy[i];
        }
        for (int i = 0; i < N - 1; i++) {
            // - 1 是为了从0开始算节点
            int s = in.nextInt() - 1;
            int p = in.nextInt() - 1;
            buildTree(p, s);
            hasFather[s] = true;
        }

        int root = 0;
        while (root < N) {
            if (!hasFather[root]) break;
            root++;
        }
        // System.out.println(root);
        dfs(root);

        System.out.println(Math.max(dp[root][0], dp[root][1]));
    }

    private static void dfs(int cur) {
        dp[cur][1] = happy[cur];
        List<Integer> children = tree.get(cur);
        if (children == null) return;
        for (int child : children) {
            dfs(child);
            dp[cur][0] += Math.max(dp[child][0], dp[child][1]);
            dp[cur][1] += dp[child][0];
        }
    }

    private static void buildTree(int p, int s) {
        List<Integer> list = tree.get(p);
        if (list == null) list = new ArrayList<>();
        list.add(s);
        tree.put(p, list);
    }
}
```





## 操作子树的节点值

### 题目

n个图顶点，初始值均为1， n-1条边，以1为根顶点，构造一棵树，对一个顶点的操作可以使其子树所有的顶点值加1，问多少次操作可以使所有顶点的值均等于其id

### 题解



```java
public class TreeDP {
    // k: 根结点, v: 子节点的集合
    static Map<Integer, List<Integer>> tree = new HashMap<>();
    static long res = 0;
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        for (int i = 0; i < n-1; i++) {
            int x = sc.nextInt(),y = sc.nextInt();
            // 双向建树，因为不知道谁是根
            buildTree(x, y);
            buildTree(y, x);
        }
        // 用于辨识是否遍历过，可以在不知道谁是根的情况下就dfs
        boolean[] flag = new boolean[n + 1];
        dfs(1, flag, 1);
        System.out.println(res);
    }

    private static void dfs(int root, boolean[] flag,int parent) {
        flag[root] = true;
        List<Integer> list = tree.get(root);
        // 计算该节点与其父节点的数值之差
        // 父节点满了，但是子节点还不够
        res += root-parent;
        for (Integer integer : list) {
            // 
            if(flag[integer])continue;
            dfs(integer,flag,root);
        }
    }
    // 建树
    private static void buildTree(int x, int y) {
        List<Integer> list = tree.get(x);
        if(list==null)list = new ArrayList<>();
        list.add(y);
        tree.put(x,list);
    }
}
```



## Zero Tree

### 题目

# Zero Tree

## 题面翻译

题目描述

一棵树是一个有n个节点与正好n-1条边的图；并且符合以下条件：对于任意两个节点之间有且只有一条简单路径。

我们定义树T的子树为一棵所有节点是树T节点的子集，所有边是T边的子集的树。

给定一颗有n个节点的树，假设它的节点被编号为1到n。每个节点有一个权值，$v_i$表示编号为i的节点的权值。你需要进行一些操作，每次操作符合以下规定：

    - 在给定的这棵树中选择一棵子树，并保证子树中含有节点1
    - 把这棵子树中的所有节点加上或减去1

你需要计算至少需要多少次操作来让所有的节点的权值归零。
输入数据

第一行包含一个整数n，表示树中节点的数量

接下来的n-1行，一行两个整数u,v，表示u和v之间有一条边(u!=v)。

最后一行包含n个整数$v_i$，用空格隔开，表示每个节点的权值
输出数据

一行一个整数，输出最小需要的操作次数。
输入样例
```
3
1 2
1 3
1 -1 1
```
输出样例
```
3
```
数据规模
对于$30\%$的数据，$n\leq100,|vi|\leq1000$

对于$50\%$的数据，$n\leq10^4$

对于$100\%$的数据，$n\leq10^5,|vi|\leq10^9$



### 题解









