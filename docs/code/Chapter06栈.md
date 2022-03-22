# 第六章：栈

## 6.1 应用

## 面试题36：后缀表达式

### 题目

后缀表达式是一种算术表达式，它的操作符在操作数的后面。输入一个用字符串数组表示的后缀表达式，请输出该后缀表达式的计算结果。假设输入的一定是有效的后缀表达式。例如，后缀表达式["2", "1", "3", "*", "+"]对应的算术表达式是“2 + 1 * 3”，因此输出它的计算结果5。

补充：(2+1)\*3 -->21+3\*

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
                // num2是先放进去的，他是计算的首位
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
        // while (!stack.empty() && stack.peek() < -as && as < 0) {
        // 错误：得保证栈顶元素>0，上面这个无法保证栈顶元素>0，下面这个既可以保证栈顶元素>0，又可以保证as<0
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



## 面试题38：每日温度

### 题目

输入一个数组，它的每个数字是某天的温度。请计算在每一天需要等几天才会有更高的温度。例如，如果输入数组[35, 31, 33, 36, 34]，那么输出为[3, 1, 1, 0, 0]。由于第一天的温度是35，要等3天才会有更高的温度36，因此对应的输出为3。第四天的温度是36，后面没有更高的温度，它对应的输出是0。

> 用栈存储数组下标，在遍历数组当前温度比较栈顶温度过程中，若是温度越来越低，就一路存进栈内，碰到当前温度比栈顶温度高的就出栈并把**结果数组**栈顶**对应的下标**位置值设置为 **当前温度的日期-栈顶温度的日期**，一直出栈到栈顶下标的温度大于等于当前温度。
>

### 参考代码

``` java
public int[] dailyTemperatures(int[] temperatures) {
    int[] result = new int[temperatures.length];        
    Stack<Integer> stack = new Stack<>();
    for (int i = 0; i < temperatures.length; i++) {
        while (!stack.empty()
            && temperatures[i] > temperatures[stack.peek()]) {
            int prev = stack.pop();
            result[prev] = i - prev;
        }

        stack.push(i);
    }

    return result;
}
```

## 面试题39：直方图最大矩形面积

### 题目

直方图是由排列在同一基线上的相邻柱子组成的图形。输入一个由非负数组成的数组，数组中的数字是直方图宽为1的柱子的高。求直方图中最大矩形的面积。例如，输入数组[3, 2, 5, 4, 6, 1, 4, 2]，它对应的直方图如图6.3所示。该直方图中最大的矩形的面积为12，如阴影部分所示。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/0603.png" alt="图2.1">

图6.3：柱子高度分别为[3, 2, 5, 4, 6, 1, 4, 2]的直方图，其中最大的矩形的面积是12，如阴影部分所示。

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

``` java
public int largestRectangleArea(int[] heights) {
    Stack<Integer> stack = new Stack<>();
    stack.push(-1);

    int maxArea = 0;
    for (int i = 0; i < heights.length; i++) {
        while (stack.peek() != -1
            && heights[stack.peek()] >= heights[i]) {
            int height = heights[stack.pop()];
            // 以上行栈顶为顶顶最大矩形，宽是左右两侧比他矮的柱子的下标之差再减1
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

#### 解法四：左右指针

```java
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

