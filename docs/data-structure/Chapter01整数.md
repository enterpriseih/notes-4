- [面试题1：整数除法](#%E9%9D%A2%E8%AF%95%E9%A2%981%E6%95%B4%E6%95%B0%E9%99%A4%E6%B3%95)
- [面试题2：二进制加法](#%E9%9D%A2%E8%AF%95%E9%A2%982%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%8A%A0%E6%B3%95)
- [面试题3：前n个数字二进制中1的个数](#%E9%9D%A2%E8%AF%95%E9%A2%983%E5%89%8Dn%E4%B8%AA%E6%95%B0%E5%AD%97%E4%BA%8C%E8%BF%9B%E5%88%B6%E4%B8%AD1%E7%9A%84%E4%B8%AA%E6%95%B0)
- [面试题4：只出现一次的数字](#%E9%9D%A2%E8%AF%95%E9%A2%984%E5%8F%AA%E5%87%BA%E7%8E%B0%E4%B8%80%E6%AC%A1%E7%9A%84%E6%95%B0%E5%AD%97)
- [面试题5：单词长度的最大乘积](#%E9%9D%A2%E8%AF%95%E9%A2%985%E5%8D%95%E8%AF%8D%E9%95%BF%E5%BA%A6%E7%9A%84%E6%9C%80%E5%A4%A7%E4%B9%98%E7%A7%AF)

# 第一章：整数

## 数学

[数学](./数学.md)

## 位运算原理

[位运算题解](./位运算.md)

**原补反**

对于正数来讲都是一个

主要是针对负数的加减运算产生的

```
负数
反码：原码的 0->1,1->0;符号位不变
补码：反码 + 1

补码转原值：补码绝对值的补码
```

**计算机数值一律采用补码来存储和表示。**

```
-8的在内存上存储形式: 1...1000

-128存储形式: 1000 0000

```



**基本原理** 

0s 表示一串 0，1s 表示一串 1。

```
x ^ 0s = x      x & 0s = 0      x | 0s = x
x ^ 1s = ~x     x & 1s = x      x | 1s = 1s
x ^ x = 0       x & x = x       x | x = x
```

利用 x ^ 1s = \~x 的特点，可以将一个数的位级表示翻转；利用 x ^ x = 0 的特点，可以将三个数中重复的两个数去除，只留下另一个数。

```
1^1^2 = 2
```

利用 x & 0s = 0 和 x & 1s = x 的特点，可以实现掩码操作。一个数 num 与 mask：00111100 进行位与操作，只保留 num 中与 mask 的 1 部分相对应的位。

```
01011011 &
00111100
--------
00011000
```

利用 x | 0s = x 和 x | 1s = 1s 的特点，可以实现设值操作。一个数 num 与 mask：00111100 进行位或操作，将 num 中与 mask 的 1 部分相对应的位都设置为 1。

```
01011011 |
00111100
--------
01111111
```

**位与运算技巧** 

`n&(n-1)` 去除 n 的位级表示中最低的那一位 1。例如对于二进制表示 01011011，减去 1 得到 01011010，这两个数相与得到 01011010。

```
01011011 &
01011010
--------
01011010
```

`n&(-n)` 得到 n 的位级表示中最低的那一位 1。-n 得到 n 的反码加 1，也就是 `-n=\~n+1`。例如对于二进制表示 10110100，-n 得到 01001100，相与得到 00000100。

```
10110100 &
01001100
--------
00000100
```

`n-(n&(-n))` 则可以去除 n 的位级表示中最低的那一位 1，和 n&(n-1) 效果一样。

**移位运算** 

`>> n` 为算术右移，相当于除以 2n，例如 -7 \>\> 2 = -2。

```
11111111111111111111111111111001  >> 2
--------
11111111111111111111111111111110
```

`>>> n` 为无符号右移，左边会补上 0。例如 -7 \>\>\> 2 = 1073741822。

```
11111111111111111111111111111001  >>> 2
--------
00111111111111111111111111111111
```

`<< n` 为算术左移，相当于乘以 2n。-7 \<\< 2 = -28。

```
11111111111111111111111111111001  << 2
--------
11111111111111111111111111100100
```

**常用操作**

```java
i >> 1 => i / 2;
i & 1 => i % 2; 
```

**mask 计算** 

要获取 111111111，将 0 取反即可，\~0。

要得到只有第 i 位为 1 的 mask，将 1 向左移动 i-1 位即可，1\<\<(i-1) 。例如 1\<\<4 得到只有第 5 位为 1 的 mask ：00010000。

要得到 1 到 i 位为 1 的 mask，(1\<\<i)-1 即可，例如将 `(1<<4)-1 = 00010000-1 = 00001111`。

要得到 1 到 i 位为 0 的 mask，只需将 1 到 i 位为 1 的 mask 取反，即 \~((1\<\<i)-1)。

**Java 中的位操作**  

```html
static int Integer.bitCount();           // 统计 1 的数量
static int Integer.highestOneBit();      // 获得最高位
static String toBinaryString(int i);     // 转换为二进制表示的字符串
```



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
	// 因为转为负数计算了
    while(dividend <= divisor) {
        int quotient = 1;
        int value = divisor;
        // 0xc0000000是MIN_VALUE的一半
        // 0x80000000是MIN_VALUE
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

## 补：Pow(x,n)

```java
public double myPow(double x, int n) {
    if(x == 0.0f) return 0.0d;
    // 防止n为负取整后越界
    long b = n;
    double res = 1.0;
    if (b < 0) {
        x = 1 / x;
        b = - b;
    }
    while (b > 0) {
        // b是奇数的话，乘入res
        if ((b & 1) == 1) res *= x;
        x *= x;
        b >>= 1;
    }
    return res;
}
```



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

输入一个整数数组，数组中除一个数字只出现一次之外其他数字都出现三次。请找出那个唯一只出现一次的数字。例如，如果输入的数组为[0, 1, 0, 1, 0, 1, 100]，则只出现一次的数字是100。

> 1. 如果所有第 i 位所有位数之和能够被 3 整除，那么只出现一次的数的第 i 位就是 0；否则为 1
> 2. " (num >> (32 - i)) & 1 " 计算的是从左数第i位的位数；eg：左 1，右移 31 位，左 1 跑到了最右边，然后与0000 0000 ... 0001 进行&运算，（都是 1 才是 1 ）如果左 1 是 1 ，则&后的结果就是 1 ，否则 0

### 参考代码

**方法一**

``` java
// 方法一
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
**方法二**

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202206292126573.png" alt="Picture3.png" style="zoom: 50%;" />

> 针对1bit
>
> - 异或运算：`x ^ 0 = x` ， `x ^ 1 = ~x`
> - 与运算：`x & 0 = 0` ， `x & 1 = x`

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202206292139687.png" alt="Picture4.png" style="zoom: 40%;" />

```java
// 计算one的方法
// 设当前状态为two one，此时输入二进制位n。
// 对上表拆分
if two == 0:
  if n == 0:
    one = one
  if n == 1:
    one = ~one
if two == 1:
    one = 0

// 引入异或，简化
if two == 0:
    one = one ^ n
if two == 1:
    one = 0
// 引入与，简化
one = one ^ n & ~two
        
// 同理two，用计算之后对one来计算
two = two ^ n & ~one

```
> 遍历完所有数字后，各二进制位都处于状态 00 和状态 01 （取决于 “只出现一次的数字” 的各二进制位是 1 还是 0，即count[i]%3 == 0还是1 ），而此两状态是由 one 来记录的（此两状态下 twos 恒为 0 ），因此返回 ones 即可。

```java
// 方法二
public int singleNumber(int[] nums) {
    int a = 0, b = 0;
    for (int num : nums) {
        a = (a^num) & ~b;
        b = (b^num) & ~a;
    }
    return a;
}
```

`举一反三`

**题目：**输入一个整数数组，数组中只有一个数字出现 m 次，其他数字都出现 n 次。请找出那个唯一出现m次的数宇。假设 m 不能被n整除。（1不能被3整除）
**分析：**解决面试题 4 的方法可以用来解决同类型的问题。如果数组中所有数字的第 i 个数位相加之和能被 n 整除，那么出现 m 次的数字的第 i 个数位一定是0；否则出现机次的数宇的第之个数位一定是 1。

## 补充：只有一个数出现奇数次，其余出现偶数次

```java
异或的知识
a^a = 0;
a^0 = a;
异或满足交换律和结合律
```

可以用来解决，只有一个数出现奇数次，其余出现偶数次的题目

## 补充：只有两个出现一次，其余出现两次

```java
public int[] singleNumber(int[] nums) {
    int xorsum = 0;
    for (int num : nums) {
        xorsum ^= num;
    }
    // xorsum是出现一次的两个数的异或
    // 防止溢出
    // 用xorsum & (-xorsum)取出xorsum二进制中最低位的那个1，记为第l位
    // 假设num1的第l位是1则，num2的第l位就是0
    // 分类，一类是l位为0，一类是l位为1
    // 出现两次的数会被分到同一类中
    // num1和num2会出现在不同类中
    int lsb = 
        xorsum == Integer.MIN_VALUE ? xorsum : xorsum & (-xorsum);
    int num1 = 0, num2 = 0;
    for (int num : nums) {
        if ((num & lsb) != 0) {
            num1 ^= num;
        } else {
            num2 ^= num;
        }
    }
    return new int[]{num1, num2};
}
```







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

## 补充1：不需要额外变量交换两个整数

```java
a = a ^ b;
b = a ^ b;
a = a ^ b;
```

