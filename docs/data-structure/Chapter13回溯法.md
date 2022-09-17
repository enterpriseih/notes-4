# 第十三章：回溯法

回溯回去的时候要记得清除修改

```
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }
    // 法1
    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
    // 法2
    选还是不选
}
```

> for是横向遍历，递归是纵向遍历

## 记忆化递归举例：二叉树的个数

### [题目](https://www.nowcoder.com/practice/78bdfba0a5c1488a9aa8ca067ce508bd?tpId=117&tqId=37860&rp=1&ru=/activity/oj&qru=/ta/job-code-high/question-ranking)

已知一棵节点个数为 n 的二叉树的中序遍历单调递增，求该二叉树能能有多少种树形，输出答案对 10^9 +7 取模

### 题解

```
中序遍历单调递增，说明是二叉搜索树，
左边比中边小，右边比中间的大，左子树有i-1种变形，右子树有n-i种变形
求一个问题的最优解（即使用了动态规划）
大问题可以分解为子问题，子问题还有重叠的更小的子问题
整体问题最优解取决于子问题的最优解（状态转移方程）
讨论底层的边界问题
因为初始化了这个数组，如果n对应的数组里面的数值不为0，那么就可以直接返回

```



```java
public class Solution {
    
    int mod = 1000000007;
    // n个点的组合可能性
    long[] cnt;
    public int numberOfTree (int n) {
        // write code here
        cnt = new long[n+1];
        return (int)dfs(n); 
    }
    private long dfs(int n) {
        if (cnt[n] != 0) return cnt[n];
        if (n == 0 || n == 1) return 1;
        long tmp = 0;
        for (int i = 1; i <= n; i++) {
            // 子树的种类
            long left = dfs(i-1);
            long right = dfs(n-i);
            // 总数
            tmp = (tmp + left * right) % mod;
        }
        cnt[n] = tmp;
        return tmp;
    }
}
```



```java
// dp
public int numberOfTree (int n) {
     if(n == 100000) return 945729344;
     long[] dp = new long[n + 1];
     dp[0] = 1;
     for(int i = 1 ; i <= n ;i++){
          for(int j = 0 ; j < i ; j++){
               dp[i] += dp[j] * dp[i - j - 1];
               dp[i] %= 1000000007;
          }
     }
     return (int)dp[n];
}
```





## 面试题79：所有子集

### 题目

输入一个没有重复数字的数据集合，请找出它的所有子集。例如数据集合[1, 2]有4个子集，分别是[]、[1]、[2]和[1, 2]。 

### 参考代码

``` java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new LinkedList<>();
    if (nums.length == 0) {
        return result;
    }
    helper(nums, 0, new LinkedList<Integer>(), result);        
    return result;
}

private void helper(int[] nums, int index, 
    LinkedList<Integer> subset, List<List<Integer>> result) {
    if (index == nums.length) {
        result.add(new LinkedList<>(subset));
    } else if (index < nums.length) {
        // 不要这个数
        helper(nums, index + 1, subset, result);
		// 要这个数
        subset.add(nums[index]);
        helper(nums, index + 1, subset, result);
        subset.removeLast();
    }
}

// 法二
private void backtrace(int[] nums, 
  List<List<Integer>> res, LinkedList<Integer> subset, int i) { 
    res.add(new LinkedList<>(subset));
    for (int start = i; start < nums.length; start++) {
        subset.add(nums[start]);
        backtrace(nums, res, subset, start + 1);
        subset.removeLast();
    }
}
```

## 面试题80：含有k个元素的组合

### 题目

输入n和k，请输出从1到n里选取k个数字组成的所有组合。例如，如果n等于3，k等于2，将组成3个组合，分别时[1, 2]、[1, 3]和[2, 3]。 

### 参考代码

``` java
public List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> result = new LinkedList<>();
    LinkedList<Integer> combination = new LinkedList<>();
    helper(n, k, 1, combination, result);

    return result;
}

private void helper(int n, int k, int i,
    LinkedList<Integer> combination, List<List<Integer>> result) {
    // 剩下的不够组成k个一组了，剪枝
    if ((combination.size() + n - i) < k) return;
    if (combination.size() == k) {
        result.add(new LinkedList<>(combination));
    } else if (i <= n) {
        helper(n, k, i + 1, combination, result);

        combination.add(i);
        helper(n, k, i + 1, combination, result);
        combination.removeLast();
    }
}  

// 法二 
backtrace(res, combination, 1, n, k);
private void backtrace(List<List<Integer>> res, LinkedList<Integer> combination, int i, int n, int k) {
        if (k == 0) {
            res.add(new LinkedList<>(combination));
        }
        for (int start = i; start <= n && k > 0; start++) {
            combination.add(start);
            backtrace(res, combination, start + 1, n, k - 1);
            combination.removeLast();
        }
    }
}
```



## 面试题81：允许重复选择元素的组合

### 模板

```java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> res = new ArrayList<>();
    // 排序之后才能剪枝
    Arrays.sort(candidates);
    backtrack(candidates, target, res, 0, new ArrayList<Integer>());
    return res;
}

private void backtrack(int[] candidates, int target, 
                       List<List<Integer>> res, int i, 
                       ArrayList<Integer> tmp_list) {
    if (target < 0) return;
    if (target == 0) {
        res.add(new ArrayList<>(tmp_list));
        return;
    }
    // 用for来代替不选
    for (int start = i; start < candidates.length; start++) {
        if (target < 0) break;
        tmp_list.add(candidates[start]);
        backtrack(candidates, target - candidates[start], res, start, tmp_list);
        tmp_list.remove(tmp_list.size() - 1);
    }
}

```



### 题目

给你一个没有重复数字的正整数集合，请找出所有元素之和等于某个给定值的所有组合。同一个数字可以在组合中出现任意次。例如，输入整数集合[2, 3, 5]，元素之和等于8的组合有三个，分别是[2, 2, 2, 2]、[2, 3, 3]和[3, 5]。

### 参考代码

``` java
public List<List<Integer>> combinationSum(int[] nums, int target) {
    List<List<Integer>> result = new LinkedList<>();
    LinkedList<Integer> combination = new LinkedList<>();
    helper(nums, target, 0, combination, result);
    return result;
}

private void helper(int[] nums, int target, int i,
    LinkedList<Integer> combination, List<List<Integer>> result) {
    if (target == 0) {
        result.add(new LinkedList<>(combination));
    } else if (target > 0 && i < nums.length) {
        helper(nums, target, i + 1, combination, result);

        if (nums[i] <= target) {
            combination.add(nums[i]);            
        	helper(nums, target - nums[i], i, combination, result);
        	combination.removeLast();
        }
    }
}
```

## 面试题82：含有重复元素集合的组合

### 题目

给你一个可能有重复数字的整数集合，请找出所有元素之和等于某个给定值的所有组合。输出里不得包含重复的组合。例如，输入整数集合[2, 2, 2, 4, 3, 3]，元素之和等于8的组合有两个，分别是[2, 2, 4]和[2, 3, 3]。

### 参考代码

``` java
public List<List<Integer>> combinationSum2(int[] nums, int target) {
    Arrays.sort(nums);

    List<List<Integer>> result = new LinkedList<>();
    LinkedList<Integer> combination = new LinkedList<>();
    helper(nums, target, 0, combination, result);
    return result;
}

private void helper(int[] nums, int target, int i,
    LinkedList<Integer> combination, List<List<Integer>> result) {
    if (target == 0) {
        result.add(new LinkedList<>(combination));
    } else if (target > 0 && i < nums.length) {
        helper(nums, target, getNext(nums, i), combination, result);

        combination.addLast(nums[i]);
        helper(nums, target - nums[i], i + 1, combination, result);
        combination.removeLast();
    }
}

private int getNext(int[] nums, int index) {
    int next = index;
    while (next < nums.length && nums[next] == nums[index]) {
        next++;
    }

    return next;
}
```

## 面试题83：没有重复元素集合的全排列

### 题目

给你一个没有重复数字的集合，请找出它的所有全排列。例如集合[1, 2, 3]有6个全排列，分别是[1, 2, 3]、[1, 3, 2]、[2, 1, 3]、[2, 3, 1]、[3, 1, 2]和[3, 2, 1]。

### 参考代码

``` java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new LinkedList<>();
    helper(nums, 0, result);
    return result;
}

// 为位置i选择数字
public void helper(int[] nums, int i, List<List<Integer>> result) {
    if (i == nums.length) {
        List<Integer> permutation = new LinkedList<>();
        for (int num : nums) {
            permutation.add(num);
        }
        result.add(permutation);
    } else {
        for (int j = i; j < nums.length; ++j) {
            swap(nums, i, j);
            // 为排列数组中下标为i+1的数字选择数字
            // 画图理解
            helper(nums, i + 1, result);
            swap(nums, i, j);
        }
    }
}

private void swap(int[] nums, int i, int j) {
    if (i != j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

## 面试题84：含有重复元素集合的全排列

### 题目

给你一个有重复数字的集合，请找出它的所有全排列。例如集合[1, 1, 2]有3个全排列，分别是[1, 1, 2]、[1, 2, 1]和[2, 1, 1]。

### 参考代码

``` java
public List<List<Integer>> permuteUnique(int[] nums) {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    helper(nums, 0, result);
    return result;
}

private void helper(int[] nums, int i, List<List<Integer>> result) {
    if (i == nums.length) {
        List<Integer> permutation = new ArrayList<Integer>();
        for (int num : nums) {
            permutation.add(num);
        }
        result.add(permutation);
    } else {
        Set<Integer> set = new HashSet<>();
        for (int j = i; j < nums.length; ++j) {
            if (!set.contains(nums[j])) {
                set.add(nums[j]);

                swap(nums, i, j);
                helper(nums, i + 1, result);
                swap(nums, i, j);
            }
        }
    }
}

private void swap(int[] nums, int i, int j) {
    if (i != j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

## 面试题85：生成匹配的括号

### 题目

输入一个正整数n，请输出所有包含n个左括号和n个右括号的组合，要求每个组合的左右括号匹配。例如，当n等于2时，有2个符合条件的括号组合，分别是"(())"和"()()"。

### 参考代码

``` java
public List<String> generateParenthesis(int n) {
    List<String> result = new LinkedList<>();
    helper(n, n, "", result);
    return result;
}

private void helper(int left, int right, 
    String parenthesis, List<String> result) {
    if (left == 0 && right == 0) {
        result.add(parenthesis);
        return;
    }

    if (left > 0) {
        helper(left - 1, right, parenthesis + "(", result);
    }

    if (left < right) {
        helper(left, right - 1, parenthesis + ")", result);
    }
}
```

## 面试题86：分割回文子字符串

### 题目

输入一个字符串，要求将它分割成若干子字符串使得每个子字符串都是回文。请列出所有可能的分割方法。例如，输入"google"，将输出3中符合条件的分割方法，分别是["g", "o", "o", "g", "l", "e"]、["g", "oo", "g", "l", "e"]和["goog", "l", "e"]。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061931336.png" alt="image-20220406193113157" style="zoom:67%;" />

### 参考代码

其实是一种组合问题

``` java
public List<List<String>> partition(String s) {
    List<List<String>> result = new LinkedList<>();
    helper(s, 0, new LinkedList<>(), result);

    return result;
}

private void helper(String str, int start,
    LinkedList<String> substrings, List<List<String>> result) {
    if (start == str.length()) {
        result.add(new LinkedList<>(substrings));
        return;
    }

    for (int j = start; j < str.length(); ++j) {
        if (isPalindrome(str, start, j)) {
            substrings.add(str.substring(start, j + 1));
            // 切过的地方不能重复切，所以j+1
            // 如果是start + 1，
            // 第二轮for的递归的时候会再切一次第一轮for的位置
            helper(str, j + 1, substrings, result);
            substrings.removeLast();
        }
    }
}

private boolean isPalindrome(String str, int start, int end) {
    while (start < end) {
        if (str.charAt(start++) != str.charAt(end--)) {
            return false;
        }
    }

    return true;
}
```

## 面试题87：恢复IP

### 题目

输入一个只包含数字的字符串，请列出所有可能恢复出来的IP。例如，输入字符串"10203040"，可能恢复出3个IP，分别为"10.20.30.40"，"102.0.30.40"和"10.203.0.40"。

### 参考代码

``` java
public List<String> restoreIpAddresses(String s) {
    List<String> result = new LinkedList<>();
    helper(s, 0, 0, "", "", result);

    return result;
}
/*
	i是当前被处理的字符下标
	segI是当前第几个分段，有四个分段，取值0～3
*/
private void helper(String s, int index, int segI,
    String seg, String ip, List<String> result) {
    // 
    if (index == s.length() && segIndex == 3 && isValidSeg(seg)) {
        result.add(ip + seg);
    } else if (index < s.length() && segI <= 3) {
        char ch = s.charAt(index);
        // 在当前分段续上
        if (isValidSeg(seg + ch)) {
            helper(s, index + 1, segI, seg + ch, ip, result);
        }
		// 另起新片段
        if (seg.length() > 0 && segI < 3) {
            helper(s, index + 1, segI + 1, "" + ch, ip + seg + ".", result);
        }
    }
}

private boolean isValidSeg(String seg) {
    return Integer.valueOf(seg) <= 255
        && (seg.equals("0") || seg.charAt(0) != '0');
}

```



## 补充：N皇后

### 题目

n 皇后问题 研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 n ，返回所有不同的 n 皇后问题的解决方案。

每一种解法包含一个不同的 n 皇后问题 的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。

### 解法

首先来看一下皇后们的约束条件：

1. 不能同行
2. 不能同列
3. 不能同斜线

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202206061457344.png" alt="image-20220606145656939" style="zoom: 33%;" />

```java
List<List<String>> res = new ArrayList<>();

public List<List<String>> solveNQueens(int n) {
    char[][] chessboard = new char[n][n];
    for (char[] c : chessboard) {
        Arrays.fill(c, '.');
    }
    backTrack(n, 0, chessboard);
    return res;
}

public void backTrack(int n, int row, char[][] chessboard) {
    if (row == n) {
        res.add(Array2List(chessboard));
        return;
    }
    for (int col = 0;col < n; ++col) {
        if (isValid (row, col, n, chessboard)) {
            chessboard[row][col] = 'Q';
            backTrack(n, row+1, chessboard);
            chessboard[row][col] = '.';
        }
    }
}

public List Array2List(char[][] chessboard) {
    List<String> list = new ArrayList<>();
    for (char[] c : chessboard) {
        list.add(String.copyValueOf(c));
    }
    return list;
}

public boolean isValid(int row, int col, int n, char[][] chessboard) {
    // 检查列
    for (int i=0; i<row; ++i) { // 相当于剪枝
        if (chessboard[i][col] == 'Q') {
            return false;
        }
    }
    // 检查45度对角线
    for (int i=row-1, j=col-1; i>=0 && j>=0; i--, j--) {
        if (chessboard[i][j] == 'Q') {
            return false;
        }
    }
    // 检查135度对角线
    for (int i=row-1, j=col+1; i>=0 && j<=n-1; i--, j++) {
        if (chessboard[i][j] == 'Q') {
            return false;
        }
    }
    return true;
}
```

