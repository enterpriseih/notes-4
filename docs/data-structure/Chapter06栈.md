# 第六章：栈

Stack

push(e)，pop，peek



```java
Deque<?> stack = new ArrayDeque<>();
Deque<?> stack = new LinkedList<>();
pollFirst();
pollLast();
offerFirst();
offerLast();

// LinkedList实现了List和Deque接口，但是创建的时候，注意多态
List<?> stack = new LinkedList<>();
addFirst();
removeFirst();
getFirst();
```



## 6.1 应用

## 补充：用栈实现队列

A是逆序，将A中的数挪到B中，然后删除B的顶部，就是队列的首部元素

```java
class CQueue {
    LinkedList<Integer> A, B;
    public CQueue() {
        A = new LinkedList<Integer>();
        B = new LinkedList<Integer>();
    }
    public void appendTail(int value) {
        A.addLast(value);
    }
    public int deleteHead() {
        if(!B.isEmpty()) return B.removeLast();
        if(A.isEmpty()) return -1;
        while(!A.isEmpty())
            B.addLast(A.removeLast());
        return B.removeLast();
    }
}

```



## 面试题36：后缀表达式

### 题目

后缀表达式是一种算术表达式，它的操作符在操作数的后面。输入一个用字符串数组表示的后缀表达式，请输出该后缀表达式的计算结果。假设输入的一定是有效的后缀表达式。例如，后缀表达式["2", "1", "3", "*", "+"]对应的算术表达式是“2 + 1 * 3”，因此输出它的计算结果5。

补充：(2+1)\*3 -->21+3\*

```
9+(3-1)*3+10/2
9 3 1 - 3 * + 10 2 / +
```



### 参考代码

``` java
public int evalRPN(String[] tokens) {
    Stack<Integer> stack = new Stack<Integer>();
    for (String token : tokens) {
        switch (token) {
            case "+":
            case "-":
            case "*":
            case "/":
                int num1 = stack.pop();
                int num2 = stack.pop();
                // num2是先放进去的，是计算的首位
                stack.push(calculate(num2, num1, token));
                break;
            default:
                stack.push(Integer.parseInt(token));
        }
    }

    return stack.pop();
}

private int calculate(int num1, int num2, String operator) {
    switch (operator) {
        case "+":
            return num1 + num2;
        case "-":
            return num1 - num2;
        case "*":
            return num1 * num2;
        case "/":
            return num1 / num2;
        default:
            return 0;
    }
}
```



## 面试题37：小行星碰撞

### 题目

输入一个表示小行星的数组，数组中每个数字的绝对值表示小行星的大小，数字的正负号表示小行星运动的方向，正号表示向右飞行，负号表示向左飞行。如果两个小行星相撞，体积较小的小行星将会爆炸最终消失，体积较大的小行星不受影响。如果相撞的两个小行星大小相同，它们都会爆炸。飞行方向相同的小行星永远不会相撞。求最终剩下的小行星。例如，假如有六个小行星[4, 5, -6, 4, 8, -5]（如图6.2所示），它们相撞之后最终剩下三个小行星[-6, 4, 8]。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0602.png" alt="图2.1">

图6.2：用数组[4, 5, -6, 4, 8, -5]表示的六个小行星。箭头表示飞行的方向。

### 参考代码

``` java
// 以正数向右为基准，只有右会和左撞，左不会和右撞
// 所有向左的都会和向右的相撞
// 负数都在正数左边，该题的栈是部分排序
// 只有栈顶是向右的才会发生相撞
public int[] asteroidCollision(int[] asteroids) {
    Stack<Integer> stack = new Stack<>();
    for (int as : asteroids) {
        // 既可以保证栈顶元素>0，又可以保证as<0
        while (!stack.empty() && stack.peek() > 0 && stack.peek() < -as) {
            stack.pop();
        }

        if (!stack.empty() && as < 0 && stack.peek() == -as) {
            stack.pop();
        } else if (as > 0 || stack.empty() || stack.peek() < 0) {
            stack.push(as);
        }
    }

    return stack.stream().mapToInt(i->i).toArray();
}
```

## 补：有效的括号

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

- 左括号必须用相同类型的右括号闭合。
- 左括号必须以正确的顺序闭合。
- 注意空字符串可被认为是有效字符串。

示例 :

- "()[]{}" or "{[]}"：true
- "([)]"：false

```java
private static final Map<Character,Character> map = 
    new HashMap<Character,Character>(){{
    	put('{','}'); put('[',']'); put('(',')'); put('?','?');
	}};
// 加?是为了pop不报错
public boolean isValid(String s) {
    if(s.length() > 0 && !map.containsKey(s.charAt(0))) return false;
    LinkedList<Character> stack = new LinkedList<Character>() {{ 
        add('?'); 
    }};
    for(Character c : s.toCharArray()){
        // 如果是左括号直接放进去
        if(map.containsKey(c)) stack.addLast(c);
        else if(map.get(stack.removeLast()) != c) return false;
    }
    return stack.size() == 1;
}
```



## 6.2 单调栈

> 通常是一维数组，要寻找任一个元素的右边或者左边第一个比自己大或者小的元素的位置，此时我们就要想到可以用单调栈了。

在使用单调栈的时候首先要明确如下几点：

1. 单调栈里存放的元素是什么？

	`单调栈里只需要存放元素的下标i`就可以了，如果需要使用对应的元素，直接T[i]就可以获取。

2. 单调栈里元素是递增呢？ 还是递减呢？（**从底到头的顺序**）

	**要找大的，就按照递减顺序**（底大头小），这样才知道遍历的元素`T[i] > T[stack.peek()]`

3. 单调栈的过程有如下三种

	- 当前遍历的元素T[i]小于栈顶元素T[st.top()]的情况
	- 当前遍历的元素T[i]等于栈顶元素T[st.top()]的情况
	- 当前遍历的元素T[i]大于栈顶元素T[st.top()]的情况

## 面试题38：每日温度

### 题目

输入一个数组，它的每个数字是某天的温度。请计算在每一天需要等几天才会有更高的温度。例如，如果输入数组[35, 31, 33, 36, 34]，那么输出为[3, 1, 1, 0, 0]。由于第一天的温度是35，要等3天才会有更高的温度36，因此对应的输出为3。第四天的温度是36，后面没有更高的温度，它对应的输出是0。

> **求右边第一个大于当前值的下标与当前下标差**
>
> 用栈存储数组下标，在遍历数组当前温度比较栈顶温度过程中，
>
> - 若是温度越来越低或等于，就一路存进栈内，
> - 碰到当前温度比栈顶温度高的就出栈，并把**结果数组**中栈顶**对应的下标**位置值设置为 **当前温度的日期 - 栈顶温度的日期**，一直出栈到栈顶下标的温度大于等于当前温度。

### 参考代码

``` java
/**
 * 单调栈，栈内顺序要么从大到小 要么从小到大,本题从大到小
 * 入站元素要和当前栈内栈首元素进行比较
 * 若大于栈首则与元素下标做差
 * 若小于等于则放入
 * 放入的是下标
 */
public int[] dailyTemperatures(int[] temperatures) {
    int[] ans = new int[temperatures.length];
    Stack<Integer> st = new Stack<>();
    for (int i = 0; i < temperatures.length; i++) {
        while (!st.isEmpty() && temperatures[st.peek()] < temperatures[i]) {
            int prev = st.pop();
            ans[prev] = i - prev;
        }
        st.push(i);
    }
    return ans;
}
```

## 补1：下一个更大元素

### 题目

给你两个**没有重复元素**的数组 nums1 和 nums2 ，其中nums1 是 nums2 的子集。

请你找出 nums1 中每个元素在 nums2 中的下一个比其大的值。

nums1 中数字 x 的下一个更大元素是指 x 在 nums2 中**对应位置的右边**的第一个比 x 大的元素。如果不存在，对应位置输出 -1 。

示例 1:

输入: nums1 = [4,1,2], nums2 = [1,3,4,2].
输出: [-1,3,-1]
解释:
对于 num1 中的数字 4 ，你无法在第二个数组中找到下一个更大的数字，因此输出 -1 。
对于 num1 中的数字 1 ，第二个数组中数字1右边的下一个较大数字是 3 。
对于 num1 中的数字 2 ，第二个数组中没有下一个更大的数字，因此输出 -1 。

> 遍历nums2
>
> 数组中没有重复元素，可以用map来做映射。根据数值快速找到下标，还可以判断nums2[i]是否在nums1中出现过。

```java
class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        Stack<Integer> st = new Stack<>();
        int[] res = new int[nums1.length];
        Arrays.fill(res, -1);
        Map<Integer, Integer> map = new HashMap();
        for (int i = 0; i < nums1.length; i++) {
            map.put(nums1[i], i);
        }
        for (int i = 0; i < nums2.length; i++) {
            while (!st.isEmpty() && nums2[i] > nums2[st.peek()]) {
                if (map.containsKey(nums2[st.peek()])) {
                    res[map.get(nums2[st.peek()])] = nums2[i];
                }
                st.pop();
            }
            st.push(i);
        }
        return res;
    }
}
```

## 补2：循环数组中的下一个更大元素

### 题目

给定一个循环数组（最后一个元素的下一个元素是数组的第一个元素），输出每个元素的下一个更大元素。数字 x 的下一个更大的元素是按数组遍历顺序，这个数字之后的第一个比它更大的数，这意味着你应该循环地搜索它的下一个更大的数。如果不存在，则输出 -1。

示例 1:

- 输入: [1,2,1]
- 输出: [2,-1,2]
- 解释: 第一个 1 的下一个更大的数是 2；数字 2 找不到下一个更大的数；第二个 1 的下一个最大的数需要循环搜索，结果也是 2。

```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int[] res = new int[nums.length];
        Arrays.fill(res, -1);
        Stack<Integer> st = new Stack<>();
        for (int i = 0; i < nums.length * 2; i++) {
            while (!st.isEmpty() && nums[i % nums.length] > nums[st.peek()]) {
                res[st.peek()] = nums[i % nums.length];
                st.pop();
            }
            // 第二轮的时候就不用放进去了，否则会造成冗余弹栈操作
            if(i < nums.length) st.push(i);
        }
        return res;
    }
}
```



## 补3：[接雨水](https://leetcode.cn/problems/trapping-rain-water/)

### 题目

给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

示例：

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/06b3.png" alt="图2.1" style="zoom: 50%;" >

- 输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
- 输出：6
- 解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。

### 解法一：动态规划

可以看出每一列雨水的高度，取决于，该列`左侧最高的柱子和右侧最高的柱子`中`最矮`的那个柱子的高度。

如：

列4 左侧最高的柱子是列3，高度为2（以下用leftMax[4]表示）。

列4 右侧最高的柱子是列7，高度为3（以下用rightMax[4]表示）。

列4 柱子的高度为1（以下用height[4]表示）

那么列4的雨水高度为`列3和列7的高度最小值减列4高度`，即：` min(leftMax[4], rightMax[4]) - height[4]`。

列4的雨水高度求出来了，宽度为1，相乘就是列4的雨水体积了。

基于双指针会出现重复计算，所以动态规划：

当前位置，左边的最高高度是前一个位置的左边最高高度和本高度的最大值。

即从左向右遍历：maxLeft[i] = max(height[i], maxLeft[i - 1]);

从右向左遍历：maxRight[i] = max(height[i], maxRight[i + 1]);

```java
public int trap(int[] height) {
    int length = height.length;
    if (length <= 2) return 0;
    int[] maxLeft = new int[length];
    int[] maxRight = new int[length];

    // 记录每个柱子左边柱子最大高度
    maxLeft[0] = height[0];
    for (int i = 1; i< length; i++) 
        maxLeft[i] = Math.max(height[i], maxLeft[i-1]);

    // 记录每个柱子右边柱子最大高度
    maxRight[length - 1] = height[length - 1];
    for(int i = length - 2; i >= 0; i--) 
        maxRight[i] = Math.max(height[i], maxRight[i+1]);

    // 求和
    int sum = 0;
    for (int i = 0; i < length; i++) {
        int count = Math.min(maxLeft[i], maxRight[i]) - height[i];
        if (count > 0) sum += count;
    }
    return sum;
}
```

复杂度分析

- 时间复杂度：O(n)，其中 n 是数组 height 的长度。计算数组 leftMax 和 rightMax 的元素值各需要遍历数组 height 一次，计算能接的雨水总量还需要遍历一次。

- 空间复杂度：O(n)，其中 n 是数组 height 的长度。需要创建两个长度为 n 的数组 leftMax 和 rightMax。

### 解法二：单调栈

1、**按照行方向计算雨水**

2、从栈头（元素从栈头弹出）到栈底的顺序应该是从小到大的顺序，存的依旧是下标

- 因为一旦发现添加的柱子高度大于栈头元素了，此时就出现凹槽了，栈头元素就是凹槽底部的柱子，栈头第二个元素就是凹槽左边的柱子，而添加的元素就是凹槽右边的柱子。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/06b3-2.png" alt="图2.1" width=300px  >

3、`遇到相同的元素，更新栈内下标`，就是将栈里元素（旧下标）弹出，将新元素（新下标）加入栈中。

**因为我们要求宽度的时候 如果遇到相同高度的柱子，需要使用最右边的柱子来计算宽度**。

那么雨水高度是 min(凹槽左边高度, 凹槽右边高度) - 凹槽底部高度，代码为：`int h = min(height[st.peek()], height[i]) - height[mid];`

雨水的宽度是 凹槽右边的下标 - 凹槽左边的下标 - 1（因为只求中间宽度），代码为：`int w = i - st.peek() - 1 ;`

TIP：如果当前遍历的元素（柱子）高度等于栈顶元素的高度，要跟更新栈顶元素，因为遇到相相同高度的柱子，需要使用最右边的右边的柱子来计算宽度。

```java
public int trap(int[] height) {
    int res = 0;
    Stack<Integer> st = new Stack<>();
    for (int i = 0; i < height.length; i++) {
        // 这个if不加的话不影响，只是处理高度相同的列的时候的逻辑不一样
        if (!st.isEmpty() && height[i] == height[st.peek()]) {
            st.pop();
        }
        // 如果mid和peek高度一样，取min的时候肯定是取peek了，
        // 这样计算的高度是0，存不了水，
        // while继续往下算
        while (!st.isEmpty() && height[i] > height[st.peek()]) {
            int mid = st.pop();
            if(!st.isEmpty()) {
                int h = Math.min(height[st.peek()], height[i]) 
                        - height[mid];
                int w = i - st.peek() - 1; 
                res += h * w;
            }
        }
        st.push(i);
    }
    return res;
}
// 时间复杂度O(n)
// 空间复杂度O(n)
```



### 解法三：双指针

按照列求雨水

求每一列的水，我们只需要关注当前列，以及左边最高的墙，右边最高的墙就够了。

```java
public int trap(int[] height) {
    int sum = 0;
    for (int i = 0; i < height.length; i++) {
        // 第一个柱子和最后一个柱子不接雨水
        if (i==0 || i== height.length - 1) continue;

        int rHeight = height[i]; // 记录右边柱子的最高高度
        int lHeight = height[i]; // 记录左边柱子的最高高度
        for (int r = i+1; r < height.length; r++) {
            if (height[r] > rHeight) rHeight = height[r];
        }
        for (int l = i-1; l >= 0; l--) {
            if(height[l] > lHeight) lHeight = height[l];
        }
        int h = Math.min(lHeight, rHeight) - height[i];
        if (h > 0) sum += h;
    }
    return sum;

}
```





## 面试题39：直方图最大矩形面积

### 题目

直方图是由排列在同一基线上的相邻柱子组成的图形。输入一个由非负数组成的数组，数组中的数字是直方图宽为1的柱子的高。求直方图中最大矩形的面积。

例如，输入数组[3, 2, 5, 4, 6, 1, 4, 2]，它对应的直方图如图6.3所示。该直方图中最大的矩形的面积为12，如阴影部分所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0603.png" alt="图2.1">

图6.3：柱子高度分别为[3, 2, 5, 4, 6, 1, 4, 2]的直方图，其中最大的矩形的面积是12，如阴影部分所示。

> 找到左右第一个小于栈顶元素的柱子

### 参考代码

#### 解法一：蛮力法

``` java
public int largestRectangleArea(int[] heights) {
    int maxArea = 0;
    for (int i = 0; i < heights.length; i++) {
        int min = heights[i];
        for (int j = i; j < heights.length; j++) {
            min = Math.min(min, heights[j]);
            int area = min * (j - i + 1);
            maxArea = Math.max(maxArea, area);
        }
    }

    return maxArea;
}
```

#### 解法二：分治法

``` java
// 找到起点到终点之间最矮的，其高度乘以起点终点间的长度就是最大面积，
// 也可能是最矮的左侧和右侧里面
// 递归求最矮的左侧和右侧里面的最大面积
public int largestRectangleArea(int[] heights) {
    if (heights.length == 0) {
        return 0;
    }

    return helper(heights, 0, heights.length);
}

private int helper(int[] heights, int start, int end) {
    if (start == end) {
        return 0;
    }

    if (start + 1 == end) {
        return heights[start];
    }

    int minIndex = start;
    for (int i = start + 1; i < end; i++) {
        if (heights[i] < heights[minIndex]) {
            minIndex = i;
        }
    }
	// 不用end-start+1是因为start是0开始，end是长度 
    int area = (end - start) * heights[minIndex];
    int left = helper(heights, start, minIndex);
    int right = helper(heights, minIndex + 1, end);

    area = Math.max(area, left);
    return Math.max(area, right);
}
```

#### 解法三：单调栈法

一种思路

``` java
public int largestRectangleArea(int[] heights) {
    Stack<Integer> stack = new Stack<>();
    // -1 是为了防止栈顶元素左侧的第一个小于的柱子不存在
    stack.push(-1);
    int maxArea = 0;
    for (int i = 0; i < heights.length; i++) {
        while (stack.peek() != -1
            && heights[stack.peek()] >= heights[i]) {
            int height = heights[stack.pop()];
            // 以上行栈顶为顶的最大矩形，宽是左右两侧比他矮的柱子的下标之差再减1
            // i是右侧矮的柱子，peek是左侧比他矮的柱子（栈中是递增排序的）
            int width = i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }

        stack.push(i);
    }
    // 到此时为止，说明右侧没有比栈顶还矮的柱子了
    while (stack.peek() != -1) {
        int height = heights[stack.pop()];
        int width = heights.length - stack.peek() - 1;
        maxArea = Math.max(maxArea, height * width);
    }

    return maxArea;
}
```

```java
public int largestRectangleArea(int[] heights) {
    // 引入哨兵
    // 哨兵的作用是 将最后的元素出栈计算面积 以及 将开头的元素顺利入栈
    // len为引入哨兵后的数组长度
    int len = heights.length + 2;
    int[] newHeight = new int[len];
    newHeight[0] = newHeight[len - 1] = 0;
    // [1,2,3]->[0,1,2,3,0]
    for(int i = 1; i < len - 1; i++) {
        newHeight[i] = heights[i - 1];
    }
    Stack<Integer> stack = new Stack<>();
    int res = 0;
    for(int i = 0; i < len; i++) {
        while(!stack.empty() 
              && newHeight[i] < newHeight[stack.peek()]) {
            int h = newHeight[stack.pop()];
            // 有哨兵在
            int w = i - stack.peek() - 1;
            res = Math.max(res, h * w);
        }
        stack.push(i);
    }
    return res;
}
```

#### 解法四：左右指针

```java
// 超时
public int largestRectangleArea(int[] heights) {
    int max = 0;
    int left = 0;
    int right = 0;
    for(int i =0;i<heights.length;i++){
        left = i-1;
        right = i+1;
        if(heights.length * heights[i]>max){
            while(left>=0 && heights[left]>=heights[i]){
                left--; 
            }
            while(right<heights.length && heights[right]>=heights[i]){
                right++;
            }
            max = Math.max(max,(right - left - 1)*heights[i]);
        }
    }
    return max;
}
```





## 面试题40：矩阵中最大的矩形

### 题目

请在一个由0、1组成的矩阵中找出最大的只包含1的矩形并输出它的面积。例如在图6.6的矩阵中，最大的只包含1的矩阵如阴影部分所示，它的面积是6。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0606.png" alt="图2.1">

图6.6：一个由0、1组成的矩阵，其中只包含1的最大矩形的面积为6，如阴影部分所示。

> 以每行为基准组成直方图

### 参考代码

``` java
public int maximalRectangle(char[][] matrix) {
    if (matrix.length == 0 || matrix[0].length == 0) {
        return 0;
    }

    int[] heights = new int[matrix[0].length];
    int maxArea = 0;
    for(char[] row : matrix) {
        for (int i = 0; i < row.length; i++) {
            if (row[i] == '0') {
                heights[i] = 0;
            } else {
                heights[i]++;
            }
        }

        maxArea = Math.max(maxArea, largestRectangleArea(heights));
    }

    return maxArea;
}    

public int largestRectangleArea(int[] heights) {
    Stack<Integer> stack = new Stack<>();
    stack.push(-1);

    int maxArea = 0;
    for (int i = 0; i < heights.length; i++) {
        while (stack.peek() != -1
            && heights[stack.peek()] >= heights[i]) {
            int height = heights[stack.pop()];
            int width = i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }

        stack.push(i);
    }

    while (stack.peek() != -1) {
        int height = heights[stack.pop()];
        int width = heights.length - stack.peek() - 1;
        maxArea = Math.max(maxArea, height * width);
    }

    return maxArea;
}
```

