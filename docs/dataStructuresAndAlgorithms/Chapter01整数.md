<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [第一章：整数](#%E7%AC%AC%E4%B8%80%E7%AB%A0%E6%95%B4%E6%95%B0)
  - [面试题1：整数除法](#%E9%9D%A2%E8%AF%95%E9%A2%981%E6%95%B4%E6%95%B0%E9%99%A4%E6%B3%95)
    - [题目](#%E9%A2%98%E7%9B%AE)
    - [参考代码](#%E5%8F%82%E8%80%83%E4%BB%A3%E7%A0%81)
  - [面试题2：二进制加法](#%E9%9D%A2%E8%AF%95%E9%A2%982%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%8A%A0%E6%B3%95)
    - [题目](#%E9%A2%98%E7%9B%AE-1)
    - [参考代码](#%E5%8F%82%E8%80%83%E4%BB%A3%E7%A0%81-1)
  - [面试题3：前n个数字二进制中1的个数](#%E9%9D%A2%E8%AF%95%E9%A2%983%E5%89%8Dn%E4%B8%AA%E6%95%B0%E5%AD%97%E4%BA%8C%E8%BF%9B%E5%88%B6%E4%B8%AD1%E7%9A%84%E4%B8%AA%E6%95%B0)
    - [题目](#%E9%A2%98%E7%9B%AE-2)
    - [参考代码](#%E5%8F%82%E8%80%83%E4%BB%A3%E7%A0%81-2)
      - [解法一](#%E8%A7%A3%E6%B3%95%E4%B8%80)
      - [解法二](#%E8%A7%A3%E6%B3%95%E4%BA%8C)
      - [解法三](#%E8%A7%A3%E6%B3%95%E4%B8%89)
  - [面试题4：只出现一次的数字](#%E9%9D%A2%E8%AF%95%E9%A2%984%E5%8F%AA%E5%87%BA%E7%8E%B0%E4%B8%80%E6%AC%A1%E7%9A%84%E6%95%B0%E5%AD%97)
    - [题目](#%E9%A2%98%E7%9B%AE-3)
    - [参考代码](#%E5%8F%82%E8%80%83%E4%BB%A3%E7%A0%81-3)
  - [面试题5：单词长度的最大乘积](#%E9%9D%A2%E8%AF%95%E9%A2%985%E5%8D%95%E8%AF%8D%E9%95%BF%E5%BA%A6%E7%9A%84%E6%9C%80%E5%A4%A7%E4%B9%98%E7%A7%AF)
    - [题目](#%E9%A2%98%E7%9B%AE-4)
    - [参考代码](#%E5%8F%82%E8%80%83%E4%BB%A3%E7%A0%81-4)
      - [解法一：哈希表记录字符串中出现的字符](#%E8%A7%A3%E6%B3%95%E4%B8%80%E5%93%88%E5%B8%8C%E8%A1%A8%E8%AE%B0%E5%BD%95%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E5%87%BA%E7%8E%B0%E7%9A%84%E5%AD%97%E7%AC%A6)
      - [解法二：用整数的二进制位数记录字符串中出现的字符](#%E8%A7%A3%E6%B3%95%E4%BA%8C%E7%94%A8%E6%95%B4%E6%95%B0%E7%9A%84%E4%BA%8C%E8%BF%9B%E5%88%B6%E4%BD%8D%E6%95%B0%E8%AE%B0%E5%BD%95%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E5%87%BA%E7%8E%B0%E7%9A%84%E5%AD%97%E7%AC%A6)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 第一章：整数

## 面试题1：整数除法

### 题目

输入两个int型整数，求它们除法的商，要求不得使用乘号'*'、除号'/'以及求余符号'%'。当发生溢出时返回最大的整数值。假设除数不为0。例如，输入15和2，输出15/2的结果，即7。

### 参考代码

``` java
public int divide(int dividend, int divisor) {
    int res = 0;
    // 判断符号
    boolean sign = (dividend ^ divisor) < 0;

    if (dividend == Integer.MIN_VALUE) {
        if (divisor == 1) return Integer.MIN_VALUE;
        if (divisor == -1) return Integer.MAX_VALUE;
    }

    dividend = dividend > 0 ? -dividend : dividend;
    divisor = divisor > 0 ? -divisor : divisor;

    while(dividend <= divisor) {
        int quotient = 1;
        int value = divisor;
        while (value >= 0xc0000000 && dividend <= value + value) {
            value += value;
            quotient += quotient;
        }
        res += quotient;
        dividend -= value;
    }
    return sign ? -res : res;
}
```

`时间复杂度`O(log*n*)



## 面试题2：二进制加法

### 题目

输入两个表示二进制的字符串，请计算它们的和，并以二进制字符串的形式输出。例如输入的二进制字符串分别是"11"和"10"，则输出"101"。

### 参考代码

``` java
public String addBinary(String a, String b) {
    StringBuilder result = new StringBuilder();
    int aL = a.length() - 1;
    int bL = b.length() - 1;
    // 进位数
    int carry = 0;
    while (aL>=0 || bL>=0) {
        // aL后自减
        int digtalA = aL >= 0 ? a.charAt(aL--) - '0' : 0;
        int digtalB = bL >= 0 ? b.charAt(bL--) - '0' : 0;
        int sum = digtalA + digtalB + carry;
        carry = sum >= 2 ? 1 : 0;
        sum = sum >= 2 ? sum - 2 : sum;
        /*
        * sum = sum >= 2 ? 1 : 0;
        * 错误：sum进位后也可能是0
        * 不进位的话就是1或0
        * 1 + 1 + 1 = 3
        * */
        result.append(sum);
    }

    // 如果都加完了，进位还有的话
    if (carry == 1) {
        result.append(1);
    }
	// 最低位保存到了最左端，最后需要反转
    return result.reverse().toString();
}
```

`Note:`

```java
StringBuilder // 非线程安全，性能更高
StringBuffer // 线程安全
charAt() // 根据索引，输出字符串指定位置的字符
```



## 面试题3：前n个数字二进制中1的个数

### 题目

输入一个非负数n，请计算0到n之间每个数字的二进制表示中1的个数，并输出一个数组。例如，输入n为4，由于0、1、2、3、4的二进制表示的1的个数分别为0、1、1、2、1，因此输出数组[0, 1, 1, 2, 1]。

### 参考代码

#### 解法一

``` java
public int[] countBits(int num) {
    int[] result = new int[num + 1];
    for (int i = 0; i <= num; ++i) {
        int count = 0;
        int j = i;
        while (j != 0) {
            result[i]++;
            j = j & (j - 1);
        }
    }

    return result;
}
```

`时间复杂度`O(nk)，每个整数有k位，就可能有O(k)个1

#### 解法二

`Note:`整数i的二进制形式中1的个数比"i&(i-1)"的二进制中的1的个数多1个

``` java
public int[] countBits(int num) {
    int[] result = new int[num + 1];
    for (int i = 1; i <= num; ++i) {
        // 前面的个数已经计算得出，
        // 最右边的1变为0后，该数将会是整数的前1，2，4，8...的数
        result[i] = result[i & (i - 1)] + 1;
    }

    return result;
}
```

`时间复杂度`O(n)

#### 解法三

`Note:`

1. 偶数：i相当于是"i/2"左移一位的结果，并且二进制中1的个数一样。
2. 奇数：i相当于是"i/2"左移一位后再将二进制最右位设为1；因此二进制中1的个数比"i/2"多1个。

``` java
public int[] countBits(int num) {
    int[] result = new int[num + 1];
    for (int i = 1; i <= num; ++i) {
        result[i] = result[i >> 1] + (i & 1);
        // i>>1 计算i/2
        // i&1 计算i%2
    }

    return result;
}
```

`一维数组的遍历`

```java
// 1、foreach
for (int count : countBits(num)) {
    System.out.println(count);
}
// 2、Arrays.toString()
// 输出形式为：[1,2,3,4]
System.out.println(Arrays.toString(countBits(num)));
```



## 面试题4：只出现一次的数字

### 题目

输入一个整数数组，数组中除一个数字只出现一次之外其他数字都出现三次。请找出那个唯一只出现一次的数字。例如，如果输入的数组为[0, 1, 0, 1, 0, 1, 100]，则只出现一次的数字时100。

> 1. 整数 a 的偶数次异或运算&的结果还是 a（与此题无关）
> 1. 如果所有第 i 位所有位数之和能够被 3 整除，那么只出现一次的数的第 i 位就是 0；否则为 1
> 2. " (num >> (32 - i)) & 1 " 计算的是从左数第i位的位数；eg：左 1，右移 31 位，左 1 跑到了最右边，然后与0000 0000 ... 0001 进行&运算，（都是 1 才是 1 ）如果左 1 是 1 ，则&后的结果就是 1 ，否则 0

### 参考代码

``` java
public int singleNumber(int[] nums) {
    int[] bitSums = new int[32];
    for (int num : nums) {
        for (int i = 0; i < 32; i++) {
            // 数组中所有数的二进制的第i位的位数之和
            bitSums[i] += (num >> (31 - i)) & 1;
        }
    }

    int result = 0;
    for (int i = 0; i < 32; i++) {
        result = (result << 1) + bitSums[i] % 3;
    }

    return result;
}
```

`举一反三`

**题目：**输入一个整数数组，数组中只有一个数字出现m次，其他数字都出现 n 次。请找出那个唯一出现m次的数宇。假设m不能被n整除。（1不能被3整除）
**分析：**解决面试题 4 的方法可以用来解决同类型的问题。如果数组中所有数字的第 i 个数位相加之和能被 n 整除，那么出现m 次的数字的第i个数位一定是0；否则出现机次的数宇的第之个数位一定是 1。



## 面试题5：单词长度的最大乘积

### 题目

输入一个字符串数组words，请计算当两个字符串words[i]和words[j]不包含相同字符时它们长度的乘积的最大值。如果没有不包含相同字符的一对字符串，那么返回0。假设字符串中只包含英语的小写字母。例如，输入的字符串数组words为["abcw", "foo", "bar", "fxyz","abcdef"]，数组中的字符串"bar"与"foo"没有相同的字符，它们长度的乘积为9。"abcw"与" fxyz "也没有相同的字符，它们长度的乘积是16，这是不含相同字符的一对字符串的长度乘积的最大值。

### 参考代码

#### 解法一：哈希表记录字符串中出现的字符

``` java
public int maxProduct(String[] words) {
    // 用长度为26的布尔型数组模拟哈希表
    boolean[][] flags = new boolean[words.length][26];
    for (int i = 0; i < words.length; i++) {
        for(char c: words[i].toCharArray()) {
            // 下标0的值表示字符a是否出现，下标1的值表示字符b是否出现，以此类推
            flags[i][c - 'a'] = true;
        }
    }

    int result = 0;
    for (int i = 0; i < words.length; i++) {
        for (int j = i + 1; j < words.length; j++) {
            int k = 0;
            for (; k < 26; k++) {
                if (flags[i][k] && flags[j][k]) {
                    break;
                }
            }

            if (k == 26) {
                int prod = words[i].length() * words[j].length();
                result = Math.max(result, prod);
            }
        }
    }

    return result;
}
```

#### 解法二：用整数的二进制位数记录字符串中出现的字符

> 整数的二进制有32位，刚好可以表示26个字母的出现与否，1为true，0为false
>
> 如果字符串出现a，则右 1 为 1 ；如果字符串出现b，则右 2 为 1 ...

```java
public int maxProduct(String[] words) {
    int[] flags = new int[words.length];
    for (int i = 0; i < words.length; i++) {
        for (char ch : words[i].toCharArray()) {
            flags[i] |= 1 << (ch - 'a');
        }
    }

    int result = 0;
    for (int i = 0; i < words.length; i++) {
        for (int j = i+1; j < words.length; j++) {
            if ((flags[i] & flags[j]) == 0) {
                int prod = words[i].length() * words[j].length();
                result = Math.max(prod, result);
            }
        }
    }
    
    return result;
}
```
