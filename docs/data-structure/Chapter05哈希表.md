# 第五章：哈希表

## 5.1 哈希表的设计

## 面试题30：插入、删除和随机访问都是O(1)的容器

### 题目

设计一个数据结构，使得如下三个操作的时间复杂度都是O(1)：

+ insert(value)：如果数据集不包含一个数值，则把它添加到数据集；
+ remove(value)：如果数据集包含一个数值，则把它删除；
+ getRandom()：随机返回数据集中的一个数值，要求数据集里每个数字被返回的概率都相同。

> - 用HashMap存数值和数值对应的数组下标
> - 用数组存放数值
> - 这样找数的时候，先在哈希表中找下标
> - 数组中随意删除中间数值的时候会产生空缺，可以将需要删除的数值与末尾的数值交换（覆盖中间数值），然后删除末尾的数值

### 参考代码

``` java
class RandomizedSet {
    public RandomizedSet() {
        numToLocation = new HashMap<>();
        nums = new ArrayList<>();
    }
    
    public boolean insert(int val) {
        if (numToLocation.containsKey(val)) {
            return false;
        }

        numToLocation.put(val, nums.size());
        nums.add(val);
        return true;
    }
    
    public boolean remove(int val) {
        if (!numToLocation.containsKey(val)) {
            return false;
        }
        
        int location = numToLocation.get(val);
        
        numToLocation.put(nums.get(nums.size() - 1), location);
        numToLocation.remove(val);
        
        nums.set(location, nums.get(nums.size() - 1));
        nums.remove(nums.size() - 1);
        return true;
    }
    
    public int getRandom() {
        Random random = new Random();
        int r = random.nextInt(nums.size());
        return nums.get(r);
    }
}
```



## 面试题31：最近最少使用缓存LRU

### 题目

请设计实现一个最近最少使用（Least Recently Used，LRU）缓存，要求如下两个操作的时间复杂度都是O(1)：

+ get(key)：如果缓存中存在键值key，则返回它对应的值；否则返回-1。
+ put(key, value)：如果缓存中之前包含键值key，将它的值设为value；否则添加键值key及对应的值value。在添加一个键值时如果缓存容量已经满了，则在添加新键值之前删除最近最少使用的键值（缓存里最长时间没有被使用过的元素）。

> - map查找元素时间`O(1)`，双向链表增删元素时间`O(1)`
> - 哈希表和双向链表的结合
> - 键就是缓存的键，值是双向链表的节点：每个节点都是一对键值
> - 链表尾部的元素是最近使用的（包括查询和添加），头部元素是最近最少使用的

### 参考代码

#### 方法一：哈希表和双向链表结合

``` java
class ListNode {
    public int key;
    public int val;
    public ListNode prev;
    public ListNode next;

    public ListNode(int key, int val) {
        this.key = key;
        this.val = val;
    }

}

class LRUCache {
    private ListNode head;
    private ListNode tail;
    private int capacity;
    private Map<Integer, ListNode> map;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new HashMap<>();
        // 哨兵节点，位于头尾
        head = new ListNode(-1, -1);
        tail = new ListNode(-1, -1);
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        if (map.containsKey(key)) {
            moveToTail(map.get(key));
            return map.get(key).val;
        } else {
            return -1;
        }
    }
    
    public void put(int key, int value) {
        if (map.containsKey(key)) {
            map.get(key).val = value;
            moveToTail(map.get(key));
        } else {
            if (map.size() == capacity) {
                ListNode toBeDel = head.next;
                delNode(toBeDel);
                map.remove(toBeDel.key);
            }
            ListNode newNode = new ListNode(key, value); 
            addToTail(newNode);
            map.put(key, newNode);
        }
    }

    public void delNode(ListNode node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    public void moveToTail(ListNode node) {
        delNode(node);
        addToTail(node);
    }

    public void addToTail(ListNode node) {
        tail.prev.next = node;
        node.prev = tail.prev;
        node.next = tail;
        tail.prev = node;
    }
    
}
```

#### 方法二：直接使用LinkedHashMap

```
super(实参)的作用是：初始化当前对象的父类型特征。
并不是创建新对象。实际上对象只创建了1个。
```

```java
class LRUCache extends LinkedHashMap<Integer, Integer>{
    private int capacity;
    
    public LRUCache(int capacity) {
        // loadFactor默认0.75
        // accessOrder: 
        // - false表示链表按照插入顺序排列，
        // - true表示按照读取顺序排列
        super(capacity, 0.75F, true);
        this.capacity = capacity;
    }

    public int get(int key) {
        return super.getOrDefault(key, -1);
    }

    // 这个可不写，和父类相同
    public void put(int key, int value) {
        super.put(key, value);
    }
    
	// 该方法的返回值是删除最近最少使用的条件 
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity; 
    }
}

```

```java
// 在插入一个新元素之后，如果是按插入顺序排序，即调用newNode()中的linkNodeLast()完成
// 如果是按照读取顺序排序，即调用afterNodeAccess()完成
// 这个就是著名的 LRU 算法啦
// 在插入完成之后，需要回调函数判断是否需要移除某些元素！
// LinkedHashMap 函数部分源码

/**
 * 插入新节点才会触发该方法，因为只有插入新节点才需要内存
 * 根据 HashMap 的 putVal 方法, evict 一直是 true
 * removeEldestEntry 方法表示移除规则, 在 LinkedHashMap 里一直返回 false
 * 所以在 LinkedHashMap 里这个方法相当于什么都不做
 */
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    // 根据条件判断是否移除最近最少被访问的节点
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

// 移除最近最少被访问条件之一，通过覆盖此方法可实现不同策略的缓存
// LinkedHashMap是默认返回false的，我们可以继承LinkedHashMap然后复写该方法即可
// 例如 LeetCode 第 146 题就是采用该种方法，直接 return size() > capacity;
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}

```

## 补充：最不常使用缓存LFU

### 题目

一个缓存结构需要实现如下功能。

- set(key, value)：将记录(key, value)插入该结构
- get(key)：返回key对应的value值

但是缓存结构中最多放K条记录，如果新的第K+1条记录要加入，就需要根据策略删掉一条记录，然后才能把新记录加入。

这个策略为：在缓存结构的K条记录中，哪一个key从进入缓存结构的时刻开始，被调用set或者get的**次数最少**，就删掉这个key的记录；

如果调用次数最少的key有多个，上次调用发生最早的key被删除

### 解法

```java
class Node {
    int key;
    int value;
    int freq = 1;
    Node pre;
    Node next;
    public Node() {}
    public Node(int key, int value) {
        this.key = key;
        this.value = value;
    }
}

class DoublyLinkedList {
    Node head;
    Node tail;

    public DoublyLinkedList() {
        head = new Node();
        tail = new Node();
        head.next = tail;
        tail.pre = head;
    }

    void removeNode(Node node) {
        node.pre.next = node.next;
        node.next.pre = node.pre;
    }

    void addNode(Node node) {
        node.next = head.next;
        head.next.pre = node;
        head.next = node;
        node.pre = head;
    }
}
class LFUCache {
    // 存储缓存的内容
    Map<Integer, Node> cache; 
    // 存储每个频次对应的双向链表
    Map<Integer, DoublyLinkedList> freqMap; 
    int size;
    int capacity;
    int min; // 存储当前最小频次

    public LFUCache(int capacity) {
        cache = new HashMap<> (capacity);
        freqMap = new HashMap<>();
        this.capacity = capacity;
    }
    
    public int get(int key) {
        if (cache.containsKey(key)) {
            Node node = cache.get(key);
            freqInc(node);
            return node.value;
        } else {
            return -1;
        }
    }
    
    public void put(int key, int value) {
        // 不存储
        if (capacity == 0) {
            return;
        }
        if (cache.containsKey(key)) {
            Node node = cache.get(key);
            node.value = value;
            freqInc(node);
        } else {
            if (size == capacity) {
                DoublyLinkedList minFreqLinkedList = freqMap.get(min);
                cache.remove(minFreqLinkedList.tail.pre.key);
                // 这里不需要维护min, 因为下面add了newNode后min肯定是1.
                minFreqLinkedList.removeNode(minFreqLinkedList.tail.pre); 
                size--;
            }
            Node newNode = new Node(key, value);
            cache.put(key, newNode);
            DoublyLinkedList linkedList = freqMap.get(1);
            if (linkedList == null) {
                linkedList = new DoublyLinkedList();
                freqMap.put(1, linkedList);
            }
            linkedList.addNode(newNode);
            size++;  
            min = 1;   
        }
    }
    // freq增加
    void freqInc(Node node) {
        // 从原freq对应的链表里移除, 并更新min
        int freq = node.freq;
        DoublyLinkedList linkedList = freqMap.get(freq);
        linkedList.removeNode(node);
        // 如果是最低的频次，并且该频次的链表为空
        if (freq == min && linkedList.head.next == linkedList.tail) { 
            min = freq + 1;
        }
        // 加入新freq对应的链表
        node.freq++;
        linkedList = freqMap.get(freq + 1);
        if (linkedList == null) {
            linkedList = new DoublyLinkedList();
            freqMap.put(freq + 1, linkedList);
        }
        linkedList.addNode(node);
    }
}


```



## 5.2 应用

## 面试题32：有效的变位词

### 题目

给定两个字符串s和t，请判断它们是不是一组变位词。在一组变位词中，如果它们中的字符以及每个字符出现的次数都相同，但字符的顺序不能。例如"anagram"和"nagaram"就是一组变位词。

字符串完全相同不是变为词

### 参考代码

#### 解法一

``` java
public boolean isAnagram(String str1, String str2) {
    if (str1.length() != str2.length() || s.equals(t))
        return false;

    int[] counts = new int[26];
    for (char ch : str1.toCharArray()) {
        counts[ch - 'a']++;
    }

    for (char ch : str2.toCharArray()) {
        if (counts[ch - 'a'] == 0) {
            return false;
        }

        counts[ch - 'a']--;
    }

    return true;
}
```

#### 解法二

``` java
public boolean isAnagram(String str1, String str2) {
    if (str1.length() != str2.length() || s.equals(t)) {
        return false;
    }

    Map<Character, Integer> counts = new HashMap<>();
    for (char ch : str1.toCharArray()) {
        counts.put(ch, counts.getOrDefault(ch, 0) + 1);
    }

    for (char ch : str2.toCharArray()) {
        if (counts.getOrDefault(ch, 0) == 0)
            return false;

        counts.put(ch, counts.get(ch) - 1);
    }

    return true;
}
```



## 面试题33：变位词组

### 题目

给定一组单词，请将它们按照变位词分组。例如输入一组单词["eat", "tea", "tan", "ate", "nat", "bat"]，则可以分成三组，分别是["eat", "tea", "ate"]、["tan", "nat"]和["bat"]。假设单词中只包含小写的英文字母。

### 参考代码

#### 解法一

> - 每个字母都映射到一个质数上，这样变位词的字母的质数乘积是相同的
> - 哈希表的key存字母乘积，value存变位词的组合
> - 但是乘积可能会溢出

``` java
public List<List<String>> groupAnagrams(String[] strs) {
    int hash[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 
        43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101};

    Map<Long, List<String>> groups = new HashMap<>();
    for (String str : strs) {
        long wordHash = 1;
        for(int i = 0; i < str.length(); ++i) {
            wordHash *= hash[str.charAt(i) - 'a'];
        }

        groups.putIfAbsent(wordHash, new LinkedList<String>());
        groups.get(wordHash).add(str);
    }

    return new LinkedList<>(groups.values());
}
```

#### 解法二

> - 把一组变位词映射到同一个单词
> - 排序一个单词时间复杂度O(mlogm)，n个单词就是O(nmlogm)
> - 时间换不会溢出的可能

``` java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String str : strs) {
        char[] charArray = str.toCharArray();
        Arrays.sort(charArray);
        String sorted = new String(charArray);
        groups.putIfAbsent(sorted, new LinkedList<String>());
        groups.get(sorted).add(str);
    }

    return new LinkedList<>(groups.values());
    // return new LinkedList<List<String>>(groups.values());
}
```



## 面试题34：外星语言是否排序

### 题目

有一门外星语言，它的字母表刚好包含所有的英文小写字母，只是字母表的顺序不同。给定一组单词和字母表顺序，请判断这些单词是否按照字母表的顺序排序。

例如，输入一组单词["offer", "is", "coming"]，以及字母表顺序"zyxwvutsrqponmlkjihgfedcba"，由于字母'o'在字母表中位于'i'的前面，所以单词"offer"排在"is"的前面；同样由于字母'i'在字母表中位于'c'的前面，所以单词"is"排在"coming"的前面。因此这一组单词是按照字母表顺序排序的，应该输出true。

### 参考代码

``` java
public boolean isAlienSorted(String[] words, String order) {
    int[] orderArray = new int[order.length()];
    for (int i = 0; i < order.length(); ++i) {
        orderArray[order.charAt(i) - 'a'] = i;
    }

    for (int i = 0; i < words.length - 1; ++i) {
        if (!isSorted(words[i], words[i + 1], orderArray)) {
            return false;
        }
    }

    return true;
}

private boolean isSorted(String word1, String word2, int[] orderArray) {
    int i = 0;
    for (; i < word1.length() && i < word2.length(); ++i) {
        char ch1 = word1.charAt(i);
        char ch2 = word2.charAt(i);
        
        if (orderArray[ch1 - 'a'] < orderArray[ch2 - 'a']) {
            return true;
        }
        if (orderArray[ch1 - 'a'] > orderArray[ch2 - 'a']) {
            return false;
        }
    }
	// apple和app
    return i == word1.length();
}
```



## 面试题35：最小时间差

### 题目

给你一组范围在00:00至23:59的时间，求它们任意两个时间之间的最小时间差。例如，输入时间数组["23:50", "23:59", "00:00"]，"23:59"和"00:00"之间只有1分钟间隔，是最小的时间差。

### 参考代码

``` java
public int findMinDifference(List<String> timePoints) {
    if (timePoints.size() > 1440) {
        // 大于1440，说明有重复的时间
        return 0;
    }

    boolean minuteFlags[] = new boolean[1440];
    for (String time : timePoints) {
        String t[] = time.split(":");// 从":"分割成两个
        int minute = Integer.parseInt(t[0]) * 60 + Integer.parseInt(t[1]);
        if (minuteFlags[minute]) {
            // 存在，说明有重复的时间
            return 0;
        }

        minuteFlags[minute] = true;
    }

    return helper(minuteFlags);
}

private int helper(boolean minuteFlags[]) {
    int minDiff = minuteFlags.length - 1;// 1439
    int prev = -1;// prev是前一个时间，i是当前时间
    // 最小的时间取最大
    int min = minuteFlags.length - 1;
    int max = -1;
    for (int i = 0; i < minuteFlags.length; ++i) {
        if (minuteFlags[i]) {
            if (prev >= 0) {
                minDiff = Math.min(i - prev, minDiff);
            }
            prev = i;
            
            min = Math.min(i, min);
            max = Math.max(i, max);
        }
    }
	// min最小的时间，max最大的时间，
    // min + minuteFlags.length - max，加上一天的时间1440
    // 比如00:03和23:56，间隔7
    minDiff = Math.min(min + minuteFlags.length - max, minDiff);
    return minDiff;
}
```

