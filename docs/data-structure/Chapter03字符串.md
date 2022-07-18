# 第三章：字符串

## 3.1 双指针

## 面试题14：字符串中的变位词

### 题目

输入两个字符串s1和s2，如何判断s2中是否包含s1的某个变位词？如果s2中包含s1的某个变位词，则s1至少有一个变位词是s2的子字符串。假设两个输入字符串中只包含英语小写字母。例如输入字符串s1为"ab"，s2为"dgcaf"，由于s2中包含s1的变位词"ba"，因此输出是true。如果输入字符串s1为"ac"，s2为"dcgaf"，输出为false。

> - 变位词：stop，tops，pots...
> - counts存放s1中字母出现的次数，然后准备P1、P2双指针，宽度为s1点长度，从s2的开始往右移动，每次P2新指向的字母次数减1，P1失去的字母次数加1（在初始化counts的时候，先将s2的前s1.length()的子字符串的字母次数都减1）
> - 循环的时候，P2、P1上来就右移
> - P1指向子字符串第一个字符，P2指向子字符串最后一个字符

### 参考代码

``` java
public boolean checkInclusion(String s1, String s2) {
    if (s2.length() >= s1.length()) {
        int[] counts = new int[26];
        
        // 从ch1[] == ch2[] --> ch1[] - ch2[] == 0
        for (int i = 0; i < s1.length(); ++i) {
            counts[s1.charAt(i) - 'a']++;
            // 减去s2中前s1.length位，
            // 相当于第一次判断，并且把中间的部分也一并去除
            counts[s2.charAt(i) - 'a']--;
        }
		
        if (areAllZero(counts)) {
            return true;
        }
		
        // i相当于第二个指针P2
        // i - s1.length()相当于第一个指针P1的左侧位
        // 双指针往右移动的过程中
        // P2指向的字母的次数--，P1上一轮指向的字母的次数++
        // 此处开始的是第二轮判断
        for (int i = s1.length(); i < s2.length(); ++i) {
            counts[s2.charAt(i) - 'a']--;
            counts[s2.charAt(i - s1.length()) - 'a']++;
            if (areAllZero(counts)) {
                return true;
            }
        }
    }

    return false;
}

private boolean areAllZero(int[] counts) {
    for (int count : counts) {
        if (count != 0) {
            return false;
        }
    }

    return true;
}
```



## 面试题15：字符串中的所有变位词

### 题目

输入两个字符串s1和s2，如何找出s2的所有变位词在s1中的起始下标？假设两个输入字符串中只包含英语小写字母。例如输入字符串s1为"cbadabacg"，s2为"abc"，s2有两个变位词"cba"和"bac"是s1中的字符串，输出它们在s1中的起始下标0和5。

> 类14

### 参考代码

``` java
public List<Integer> findAnagrams(String s1, String s2) {
    List<Integer> indices = new LinkedList<>();
    if (s1.length() < s2.length()) {
        return indices;
    } 

    int[] counts = new int[26];
    int i = 0;
    for (; i < s2.length(); ++i) {
        counts[s2.charAt(i) - 'a']++;
        counts[s1.charAt(i) - 'a']--;
    }

    if (areAllZero(counts)) {
        indices.add(0);
    }
    // i已经是s2.length()了
    for (; i < s1.length(); ++i) {
        counts[s1.charAt(i) - 'a']--;
        counts[s1.charAt(i - s2.length()) - 'a']++;
        if (areAllZero(counts)) {
            indices.add(i - s2.length() + 1);
        }
    }

    return indices;
}

private boolean areAllZero(int[] counts) {
    for (int count : counts) {
        if (count != 0) {
            return false;
        }
    }

    return true;
}
```



## 面试题16：不含重复字符的最长子字符串

### 题目

输入一个字符串，求该字符串中不含重复字符的最长连续子字符串的长度。例如，输入字符串"babcca"，它最长的不含重复字符串的子字符串是"abc"，长度为3。

> - 指针P1、P2开始都指向第一个字符
> - 若两指针之间的字符没有重复的，则向右移动P2
> - 若有重复，则向右移动P1，直至不重复（有可能会回到P1P2指向同一个字符的状态）

### 参考代码

```java
public int lengthOfLongestSubstring(String s) {
    int maxLen = 0;
    int l = 0, r = 0;
    int[] counts = new int[256];
    while (r < s.length()) {
        counts[s.charAt(r)]++;
        while (counts[s.charAt(r)] > 1) {
            counts[s.charAt(l)]--;
            l++;
        }
        maxLen = Math.max(maxLen, r - l + 1);
        r++;
    }
    return maxLen;
}
```



## 补充：替换后的最长重复字符串

### 题目

给你一个字符串 s 和一个整数 k 。你可以选择字符串中的任一字符，并将其更改为任何其他大写英文字符。该操作最多可执行 k 次。

在执行上述操作后，返回包含相同字母的最长子字符串的长度。

```
输入：s = "ABAB", k = 2
输出：4
解释：用两个'A'替换为两个'B',反之亦然。
```

### 解法

```java
// len - 字母出现最大次数 <= 替换数目
public int characterReplacement(String s, int k) {
    int[] counts = new int[26];
    // int len = 0;
    int res = 0;
    int maxNum = 0;
    int l = 0, r = 0;
    while (r < s.length()) {
        counts[s.charAt(r) - 'A']++;
        // 维持最大的窗口，因为比它小的窗口对题目无意义
        // 窗口大小只会递增，除非数字的个数更大，才会出现更符合要求的更大的窗口
        maxNum = Math.max(maxNum, counts[s.charAt(r) - 'A']);
        // len = r - l + 1;
        if (r - l + 1 - maxNum > k) {
            counts[s.charAt(l) - 'A']--;
            l++;
        }
        r++; 
    }
    // 因为r会多走一步
    return r - l;
}
```



## 面试题17：含有所有字符的最短字符串

### 题目

输入两个字符串s和t，请找出s中包含t的所有字符的最短子字符串。例如输入s为字符串"ADDBANCAD"，t为字符串"ABC"，则s中包含字符'A'、'B'、'C'的最短子字符串是"BANC"。如果不存在符合条件的子字符串，返回空字符串""。如果存在多个符合条件的子字符串，返回任意一个。

> - 若不包含t的所有字符，P1右移，并判断新的字母是不是t里的，如果是，则count--；若全部包含了，记录此时P1P2之间的长度
> - count记录出现在t中但还没出现在s的子字符串中的字符的个数
> - 包含所有后，右移P1，判断是否仍然包含，包含再右移动，直至不包含；

### 参考代码

``` java
public String minWindow(String s, String t) {
    HashMap<Character, Integer> charToCount = new HashMap<>();
    for (char ch : t.toCharArray()) {
        charToCount.put(ch, charToCount.getOrDefault(ch, 0) + 1);
    }
	
    // count记录出现在t中但还没出现在s的子字符串中的字符的个数
    // count = 0时，说明子字符串中已经包含所有t中的字符
    int count = charToCount.size();
    int p1 = 0, p2 = 0, minStart = 0, minEnd = 0;
    int minLength = Integer.MAX_VALUE;
    while (p2 < s.length() || (count == 0 && p2 == s.length())) {
        // 第二个条件指最后一位也是t里面的，而且前面也有t的其他字符
        // eg,"ADOBECODEBANC"，无第二条件输出CODEBA，
        // p2++在if的末尾，count=0之后，p2多加了一次
        if (count > 0) {
            char p2Ch = s.charAt(p2);
            if (charToCount.containsKey(p2Ch)) {
                charToCount.put(p2Ch, charToCount.get(p2Ch) - 1);
                if (charToCount.get(p2Ch) == 0) {
                    count--;
                }
            }
			// 当count减到0的时候，末尾坐标还自加了，
            // 所以最后取子串的时候直接用p2
            p2++;
        } else {
            if (p2 - p1 + 1 < minLength) {
                minLength = p2 - p1 + 1;
                minStart = p1;
                minEnd = p2;
            }

            char p1Ch = s.charAt(p1);
            // p1原本指针移去的如果是
            if (charToCount.containsKey(p1Ch)) {
                charToCount.put(p1Ch, charToCount.get(p1Ch) + 1);
                if (charToCount.get(p1Ch) == 1) count++; 
                // 有可能连续多个都是，但是也只加一次
            }
            p1++;
        }
    }

    return minLength < Integer.MAX_VALUE
        ? s.substring(minStart, minEnd)
        : "";
}
```



## 3.2 回文

## 面试题18：有效的回文

### 题目

给定一个字符串，请判断它是不是一个回文字符串。我们只需要考虑字母或者数字字符，并忽略大小写。例如，"A man, a plan, a canal: Panama"是一个回文字符串，而"race a car"不是。

### 参考代码

``` java
public boolean isPalindrome(String s) {
    int i = 0;
    int j = s.length() - 1;
    while (i < j) {
        char ch1 = s.charAt(i);
        char ch2 = s.charAt(j);
        if (!Character.isLetterOrDigit(ch1)) {
            i++;
        } else if (!Character.isLetterOrDigit(ch2)) {
            j--;
        } else {
            ch1 = Character.toLowerCase(ch1);
            ch2 = Character.toLowerCase(ch2);
            if (ch1 != ch2) {
                return false;
            }

            i++;
            j--;
        }       
    }

    return true;
}
```



## 面试题19：最多删除一个字符得到回文

### 题目

给定一个字符串，请判断如果最多从字符串中删除一个字符能不能得到一个回文字符串。例如，如果输入字符串"abca"，由于删除字符'b'或者'c'就能得到一个回文字符串，因此输出为true。

### 参考代码

``` java
public boolean validPalindrome(String s) {
    int start = 0;
    int end = s.length() - 1;
    for (; start < s.length() / 2; ++start, --end) {
        if (s.charAt(start) != s.charAt(end)) {
            break;
        }
    }
	// start是s的一半，说明，s是回文，因为有可能删除的就是这个位置的字符
    // 分别跳过不等的字符，判断是否为回文
    return start == s.length() / 2
        || isPalindrome(s, start, end - 1)
        || isPalindrome(s, start + 1, end);
}

private boolean isPalindrome(String s, int start, int end) {
    while (start < end) {
        if (s.charAt(start) != s.charAt(end)) {
            break;
        }
        start++;
        end--;
    }

    return start >= end;
}
```

by YiENx
```java
public boolean validPalindrome(String s) {
    int start = 0, end = s.length() - 1;
    while (start < end) {
        if (s.charAt(start) != s.charAt(end)) {
            return isPalindrome(s, start + 1, end) 
                || isPalindrome(s, start, end - 1);
        }
        start++;
        end--; 
    }
    return true;
}

private boolean isPalindrome(String s, int start, int end) {
    while (start < end) {
        if (s.charAt(start) != s.charAt(end)) return false;
        start++ ;
        end--;
    }
    return true;
}
```



## 面试题20：回文子字符串的个数

### 题目

给定一个字符串，请问字符串里有多少回文连续子字符串？例如，字符串里"abc"有3个回文字符串，分别为"a"、"b"、"c"；而字符串"aaa"里有6个回文子字符串，分别为"a"、"a"、"a"、"aa"、"aa"和"aaa"。

> - 从字符串中心向两端延伸：**中心扩展法**
> - 第 i 个字符本身可以构成奇数回文的子字符串的中心
> - 第 i 和 i + 1 个字符可以构成偶数回文的中心

### 参考代码

``` java
    public int countSubstrings(String s) {
        // s = null的时候，没有length
        if (s == null || s.length() == 0) {
            return 0;
        }
        
        int count = 0;
        for (int i = 0; i < s.length(); ++i) {
            count += countPalindrome(s, i, i);
            count += countPalindrome(s, i, i + 1);
        }
        
        return count;
    }
    
    private int countPalindrome(String s, int start, int end) {
        int count = 0;
        while (start >= 0 && end < s.length()
               && s.charAt(start) == s.charAt(end)) {
            count++;
            start--;
            end++;
        }
        
        return count;
    }
```

## 补充：最长公共前缀

### 题目

```
输入：strs = ["flower","flow","flight"]
输出："fl"
```

### 解法一

按字典排序数组，比较第一个和最后一个单词，有多少前缀相同。

```java
public String longestCommonPrefix(String[] strs) {
    Arrays.sort(strs);
    StringBuilder res = new StringBuilder();
    for (int i = 0; i < strs[0].length(); i++) {
        if (strs[0].charAt(i) == strs[strs.length - 1].charAt(i)) {
            res.append(strs[0].charAt(i));
        } else {
            break;
        }
    }
    return res.toString();
}
```

### 解法二

```java
public String longestCommonPrefix(String[] strs) {
    if(strs.length == 0)return "";
    //公共前缀比所有字符串都短，随便选一个先
    String s = strs[0];
    for (String string : strs) {
        while(!string.startsWith(s)){
            if(s.length() == 0)return "";
            //公共前缀不匹配就让它变短！
            s = s.substring(0,s.length()-1);
        }
    }
    return s;
}
```

## 补充：字符串匹配的KMP算法

### 题目

给定文本串S：“BBC_ABCDAB_ABCDABCDABDE”，和模式串P：“ABCDABD”，判断P是否为S的子串。

### 解法

https://blog.csdn.net/v_july_v/article/details/7041827

https://www.zhihu.com/question/21923021

#### 暴力解法O(nm)

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202207171441901.jpg" alt="img" style="zoom: 50%;" />

```java
int ViolentMatch(String s, String p){
	int sLen = s.length();
	int pLen = p.length();
 
	int i = 0;
	int j = 0;
	while (i < sLen && j < pLen){
		if (s.charAt(i) == p.charAt(j)) {
			//①如果当前字符匹配成功（即S[i] == P[j]），则i++，j++    
			i++;
			j++;
		}
		else {
			//②如果失配（即S[i]! = P[j]），令i = i - (j - 1)，j = 0    
			i = i - j + 1;
			j = 0;
		}
	}
	//匹配成功，返回模式串p在文本串s中的位置，否则返回-1
	if (j == pLen)
		return i - j;
	else
		return -1;
}
```

#### KMP算法O(n+m)

##### next数组

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/code/202207171524880.jpg" alt="img" style="zoom:50%;" />

next数组是对于模式串而言的。**每次失配之后，移很多位**，**跳过那些不可能匹配成功的位置**。

- 定义 “k-前缀” 为一个字符串的前 k 个字符； “k-后缀” 为一个字符串的后 k 个字符。
- **next[x]** 定义为： P[0]~P[x] 这一段字符串，使得**k-前缀恰等于k-后缀**的最大的k.

> 特别地，k 必须小于字符串长度。因为这个子串一共才 i+1 个字符，自己肯定与自己相等，就没有意义了。
>
> 使用next并不是真正的移位，只是**相对的移位**
>

```
假如有三个元素ABA，
从A开始考虑，A的前后缀都为空，所以next[0] = 0;
考虑AB，前缀A，后缀B，没有共同字串，所以next[1] = 0;
考虑ABA，前缀A，AB，后缀BA，A，有一个共同字串，并且长度为1，所以next[2] = 1;
所以ABA的部分匹配表next={0,0,1}
```

- 如果next(j) == 0，说明走过的s上，没有路过能和前缀配对上的

```
   0 1 2 3 4 5 6
s: c a b c a b ?
p:   a b c a b d
此时
s6 != p5
对比的路上，s对应p的部分可以跳过一些不匹配的
p的0~4对应s的1~5是完全匹配的
所以下一步，直接比较s6和p2
前缀和后缀相同
   0 1 2 3 4 5 6
s: c a b c a b ?
p:         a b c a b d
```

- next具体流程

求next数组的过程完全可以看成字符串匹配的过程，即以模式字符串为主字符串，以模式字符串的前缀为目标字符串

```
p - i, pp - j
①
        ↆ
p   : a a a b b a b
pp  :   a a a b b a b
        𐌣
if(p[i] == pp[j]) next[i] = j + 1;
next: 0 1
i++, j++;

②
          ↆ
p   : a a a b b a b
pp  :   a a a b b a b
          𐌣
next[i] = j + 1;
next: 0 1 2 
i++, j++;

③
            ↆ
p   : a a a b b a b
pp  :   a a a b b a b
            𐌣
while(p[i] != pp[j]) j = next[j - 1]
until j == 0 or p[i] != pp[j];
next[i] = j;
next: 0 1 2 0

...

p    : a a a b b a b
next : 0 1 2 0 0 1 0
```

- buildNext()

```java
private int[] buildNext(String p) {
    int pLen = p.length();
    int[] next = new int[pLen];
    // next[0] = 0;因为规定自身没有前后缀
    for (int i = 1, j = 0; i < pLen; i++) {
        while (j > 0 && p.charAt(i) != p.charAt(j)) {
            j = next[j - 1];
        }
        
        if (p.charAt(i) == p.charAt(j)) {
            j++;
        }
        // j已经自加1了
        // 或者j == 0了，还是p[i] != p[j]
        next[i] = j;
    }
    return next;   
}
```

- 总体实现

```java
class Solution {
    // KMP 算法
    // ss: 原串(string)  pp: 匹配串(pattern)
    public int strStr(String s, String p) {
        if (p.isEmpty()) return 0;
        int sLen = s.length(), pLen = p.length();
        
        int[] next = buildNext(p);
		// i不回头
        for (int i = 0, j = 0; i < sLen; i++) {
            while (j > 0 && s.charAt(i) != p.charAt(j)) {
                j = next[j - 1];
            }
            if (s.charAt(i) == p.charAt(j)) {
                j++;
            }
            if (j == pLen) {
                return i - pLen + 1;
            }
        }
        return -1;
    } 
}
```

## 最长重复子串LeetCode1044

### [题目](https://leetcode.cn/problems/longest-duplicate-substring)

给你一个字符串 s ，考虑其所有 重复子串 ：即 s 的（连续）子串，在 s 中出现 2 次或更多次。这些出现之间可能存在重叠。

返回 任意一个 可能具有最长长度的重复子串。如果 s 不含重复子串，那么答案为 "" 。

```
输入：s = "banana"
输出："ana"
```

### 解法一（超时）

KMP算法的next数组

```java
public String longestDupSubstring(String s) {
    int ansMax = 0;
    String res = "";
    for (int i = 0; i < s.length(); i++) {
        String p = s.substring(i);
        int subLen = buildNext(p);
        if (subLen > ansMax) {
            ansMax = subLen;
            res = p.substring(0, subLen);
        }
    }
    return res;
}

private int buildNext(String p) {
    int max = 0;
    int[] next = new int[p.length()];
    for (int i = 1, j = 0; i < p.length(); i++) {
        while (j > 0 && p.charAt(i) != p.charAt(j)) {
            j = next[j - 1];
        }
        if (p.charAt(i) == p.charAt(j)) {
            j++;
        }
        next[i] = j;
        max = Math.max(max, next[i]);
    }
    return max;
    // return p.substring(0, max);
}
```

