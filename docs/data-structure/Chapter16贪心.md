# 第十六章贪心

## 跳跃游戏

### [题目](https://leetcode.cn/problems/jump-game/)

给定一个非负整数数组 nums ，你最初位于数组的 **第一个下标** 。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标。

```
示例1
输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 
然后再从下标 1 跳 3 步到达最后一个下标。

示例2
输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。
但该下标的最大跳跃长度是 0 ， 
所以永远不可能到达最后一个下标。
```

> **贪心算法局部最优解：每次取最大跳跃步数（取最大覆盖范围），整体最优解：最后得到整体最大覆盖范围，看是否能到终点**。
>
> 局部最优推出全局最优，找不出反例，试试贪心！

### 贪心题解

```java
public boolean canJump(int[] nums) {
    int range = 0;
    for (int i = 0; i <= range; i++) {
        range = Math.max(range, i + nums[i]);
        if (range >= nums.length - 1) return true;
    }
    return false;
}
```

> i 每次移动只能在cover的范围内移动，每移动一个元素，cover得到该元素数值（新的覆盖范围）的补充，让i继续移动下去。
>
> 而cover每次只取 max(该元素数值补充后的范围, cover本身范围)。
>
> 如果cover大于等于了终点下标，直接return true就可以了。

### 动态规划题解

动态规划一遍循环，判断能否到达最后一个元素，就是判断倒数第二个能否到达倒数第一个元素，如果不能就是判断倒数第三个元素能否到达倒数第一个元素，如果能到达就是判断倒数第三个元素能否到达倒数第二个元素

```java
 public boolean canJump(int[] nums) {
     int length = nums.length;
     if (length == 1) return true;
     int minStep = 1;        
     //定义一个数，为达到最后最后一个结点最小需要的步数
     for (int i = length-2; i >0; i--) {          
         //从倒数第二个往第二个开始遍历
         if (nums[i]<minStep){            
             // 如果当前元素的值小于最小步数,则到达最后一个元素的最小步数+1;
             minStep++;
         }else {
             minStep = 1;              
             // 如果当前元素的值大于或等于最小步数，则一定能到达最后一个元素，
             // 此时可以就当前元素认为是最后一个元素，并对于前一个元素来说最小步数为1;
         }
     }

     return nums[0] >= minStep;       
     //此时minStep为达到"最后一个元素"(并不是nums[length-1])的最小步数，
     //只要判断第一个元素的值是否大于或等于最小步数就可以了;
 }
```



## 跳跃游戏2

### 题目

给定一个非负整数数组，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用**最少的跳跃次数**到达数组的最后一个位置。

说明: 假设你总是可以到达数组的最后一个位置。

```
输入: nums = [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

### 题解

> **要从覆盖范围出发，不管怎么跳，覆盖范围内一定是可以跳到的，以最小的步数增加覆盖范围，覆盖范围一旦覆盖了终点，得到的就是最小步数！**
>
> **这里需要统计两个覆盖范围，当前当前能走到的最大区域点和当前区域里的最大的覆盖区域**。

```java
public int jump(int[] nums) {
    if (nums == null || nums.length == 0 || nums.length == 1) {
        return 0;
    }
    //记录跳跃的次数
    int count = 0;
    //当前能走到的最大区域点
    int curEnd = 0;
    //当前区域里的最大的覆盖区域
    int maxDistance = 0;
    for (int i = 0; i < nums.length; i++) {
        //在可覆盖区域内更新最大的覆盖区域
        maxDistance = Math.max(maxDistance, i + nums[i]);
        //说明当前一步，再跳一步就到达了末尾
        if (maxDistance>=nums.length - 1){
            count++;
            break;
        }
        //走到当前覆盖的最大区域时，更新下一步可达的最大区域
        if (i == curEnd){
            curEnd = maxDistance;
            count++;
        }
    }
    return count;
}
```

