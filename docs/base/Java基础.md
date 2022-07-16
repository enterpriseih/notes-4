# 基础的基础

## Java程序编译解释过程



## java实现多态的机制

靠的是父类或接口定义的引用变量可以指向子类或具体实现类的实例对象，在运行期才动态绑定，就是引用变量所指向的具体实例对象的方法。

## 一、变量类型

第一种（按照位置）：局部变量 vs 成员变量（或属性）

- **局部变量**
	- 方法内部，生命周期与方法一致
	- 没有默认值，若使用，`必须在使用前显式赋值`
	- **栈**内存中
	
- **成员变量**
	- 再分：
		- **类变量**/**静态变量**（static）
			- 准备阶段，默认赋值 —> 初始化阶段，显式赋值
		- **实例变量**（无static）
			- 随对象的创建，会在堆空间中分配实例变量空间，并进行默认赋值
	
	- 类中，方法外
	- 有默认值，规则和数组一致
	- **堆**内存中

第二种（按照类型）：基本数据类型 vs 引用数据类型变量（类、数组、接口）


## 二、字面量和符号引用

- 字面量包括：
	- 文本字符串 
	- 八种基本类型的值 
	- 被声明为final的常量等;
- 符号引用包括：
	- 类和方法的全限定名 
	- 字段（属性field）的名称和描述符 
	- 方法的名称和描述符。

## 三、关键字修饰

### 1、final

**final：属性不可变、方法不可覆盖、类不可继承**

final修饰的方法不能重写，但可以继承、重载

**重载**：方法名字相同，而参数不同。返回类型可以相同也可以不同。

每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。

**重写**：方法名和参数列表必须相同，返回值范围和异常范围必须比父类小

> 基本数据类型，初始化之后不可改变
>
> 引用数据类型，初始化之后**引用的对象不能改变，但是引用的对象的值可以改变**



**Note**：内部类只能访问局部final变量（因为内部类和外部类的生命周期可能不同）

> 1.8之后不用加，默认不能修改，语法糖



### 2、transient

不可序列化

<br>

## 四、接口与抽象类

接口是一种约束，抽象类是为了复用

- 接口是特殊的抽象类
- 抽象类可以存在普通成员函数/变量，而接口只能存在 public abstract 方法/public abstract final 成员变量
- 抽象类只能继承一个，接口可以实现多个

> 接口可以继承接口
>
> 抽象类可以实现接口
>
> 抽象类可以继承具体类
>
> 抽象类可以有静态的main方法

### 1、抽象类

不能有抽象构造方法或抽象静态方法（这两个都不能被继承），如果子类不能实现所有父类抽象方法，那么子类也必须是抽象类。

## 五、Static Nested Class 和 Inner Class

创建内部类的实例对象时，需要先创建外部类的实例对象，然后用外部类的实例对象去创建内部类的实例对象

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
```

在方法外部定义的内部类前面加上static，从而成为Static Nested Class，不再具有内部类的特性，运行时的行为和功能和普通类无区别。

无需创建外部类，就可以直接创建。

```java
Outer.Inner inner = new Outer.Inner();
```

Static Nested Class不依赖于外部类的实例对象，所以可以访问外部类的非static成员变量

# 对象的实例化

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203311610240.png" alt="第10章_对象的实例化" style="zoom:50%;" />

# 异常体系

## 一、error

程序无法处理的错误，一旦出现，程序就会被迫停止。

OutOfMemoryError（内存溢出）

StackOverflowError（栈溢出）

## 二、exception

设计或实现上的问题，代码编写或逻辑上的，可以进行异常处理

- 编译时异常(checked)
	- IOException
		- FileNotFoundException
	- ClassNotFoundException
- 运行时异常(unchecked)
	- NullPointerException
	- ArrayIndexOutOfBoundsException
	- ClassCastException（数据类型转换异常）
	- NumberFormatException
	- InputMismatchException
	- ArithmeticException（数学运算异常）
	- IllegalArgumentException（方法的参数错误）

# String

## 一、String的不可变

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    /* String本质是个char数组，而且用final修饰 */
    private final char value[];
}
```

首先String类是用final关键字修饰，这说明String不可继承。且value也用final修饰value这个引用地址不可变。

引用不可变，如果更改value内容不也是可以的吗？

在源码中没有对value里的元素进行操作；并且string类是final不可继承，避免被别人继承后破坏。

综上，才使得String是不可变的。

## 二、StringTable

### 1、字符串的拼接

1. 常量与常量的拼接结果在字符串常量池，原理是编译期优化
2. 字符串常量池中不会存在相同内容的常量
3. 只要其中有一个是变量，结果就在堆中（但不是字符串常量池所在的空间），原理是StringBuilder；final除外
4. 如果拼接的结果调用intern()方法，则主动将字符串常量池中还没有的字符串对象放入池中，并返回此对象的地址，如存在了就直接返回地址

```java
String s1 = "a" + "b" + "c";
String s2 = "abc";
/*
	编译阶段
	s1 = "abc";
	s2 = "abc";
	编译期就完成了拼接
*/
System.out.println(s1 == s2);// true
System.out.println(s2.equals(s1););// true

/*
	如下的 s1 + s2 的执行细节
	1、StringBuilder s = new StringBuilder();
	2、s.append("Java");
	3、s.append("Go");
	4、s.toString(); ——> 约等于new String("ab");
	JDK5.0之前用的buffer
*/

String s1 = "Java";
String s2 = "Go";
String s3 = "JavaGo";
String s4 = "Java" + "Go";
String s5 = s1 + "Go";
String s6 = "Java" + s2;
String s7 = s1 + s2;
final String s8 = "Java";
final String s9 = "Go";
String s10 = s8 + s9;

System.out.println(s3 == s4);// true
System.out.println(s3 == s5);// false
System.out.println(s3 == s6);// false
System.out.println(s3 == s7);// false
System.out.println(s5 == s6);// false
System.out.println(s5 == s7);// false
System.out.println(s6 == s7);// false
System.out.println(s10 == s3);// true

String s11 = s6.intern();
System.out.println(s3 == s11);// true


```



### 2、intern()

native方法

如果拼接的结果调用intern()方法，查看字符串常量池

- 如存在字符串就直接返回地址
- 如果不存在
	- ≤1.6 放入字符串池
	- ≥1.7 将对象的引用地址复制放入池

Interned String就是确保字符串在内存中只有一份拷贝

调用Intern()可以保证变量s指向的是字符串常量池中的数据

<br>

**题目**

```java
/**
 * 题目：
 * new String("ab")会创建几个对象？
 *     一个对象是：new关键字在堆空间创建的
 *     另一个对象是：字符串常量池中的对象"ab" 
 *
 *
 * 思考：
 * new String("a") + new String("b")呢？
 *  对象1：new StringBuilder()
 *  对象2：new String("a")
 *  对象3：常量池中的"a"
 *  对象4：new String("b")
 *  对象5：常量池中的"b"
 *
 *  深入剖析： StringBuilder的toString():
 *      对象6 ：new String("ab")
 *      强调一下，toString()的调用，在字符串常量池中，没有生成"ab"
 *
 */
```



```java
String s1 = new String("a");
s1.intern(); // 调用之前已经存在了a
String s2 = "a";
System.out.println(s1 == s2);
// jdk1.6 false, jdk7/8 false

String s3 = new String("a") + new String("a");
// s3变量记录的地址：new String("aa")
// 执行完后，字符串常量池中没有"aa"
s3.intern();
// jdk6:创建了一个新的对象"aa"，也有了新的地址
// jdk7:常量池放进堆里了，常量池中并没有创建"aa",
// 而是创建一个指向堆空间中new String("aa")的地址
// 因为堆中已经有了，节省空间，常量池中就不创建了
String s4 = "aa";
// s4变量记录的地址：使用的是intern()后在常量池中生成的"aa"的地址
System.out.println(s3 == s4);
// jdk1.6 false, jdk7/8 true

```



补充：

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203312241480.PNG" alt="IMG_1014" style="zoom:40%;" />

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203312241221.PNG" alt="IMG_1013" style="zoom:40%;" />



---

# 集合框架

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061519804.png" alt="image-20220406151857332" style="zoom: 50%;" />

Note：无序是指存储位置的无序，不是按照索引顺序添加，而是按照hash值添加

threshold：扩容的阈值，=容量*加载因子，超过阈值就扩容

工具类：Collections操作集合、Arrays操作数组

## 一、Collection

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061509495.png" alt="image-20220406150940121" style="zoom:67%;" />

### 1、Arrays.sort()

`Arrays类`

使用的是经过调优的**快速排序**算法

对基本数据类型`byte[] int[] double[] char[]`进行升序排序

对引用数据类型进行降序，如`Integer[]，Double[]，Character[]`等

```java
// 1、对list排序的时候需要保证list中的元素实现了Comparable接口
// 2、或者自定义一个比较器对象Comparator
// [参数1] <、=、> [参数2]的时候，分别返回-1、0、1。
// 表示将[参数1]和[参数2]按照-1、0、1的顺序排序
class cmp implements Comparator<Integer>{
	public int compare(Ingeger n1, Ingeger n2){
		if (n1 < n2){
			return -1;
		}else{
			return 1;
		}
	}
}

```

### 2、集合的remove()

```
关于集合元素的remove
    重点：当集合的结构发生改变时，迭代器必须重新获取，如果还用旧的迭代器，会出现
    异常：java.util.ConcurrentModificationException

    重点：在迭代集合元素的过程中，不能调用集合对象的remove方法，删除元素：
        c.remove(o); 迭代过程中不能这样。
    	会出现：java.util.ConcurrentModificationException

```


> **注**：在迭代元素的过程当中，一定要使用迭代器**Iterator的remove方法**，删除元素，不要使用集合自带的remove方法删除元素。
>
> 集合结构只要发生改变，迭代器必须重新获取。
>
> 通过迭代器删除时，会自动更新迭代器，并且更新集合

```java
public static void main(String[] args) {
    // 创建集合
    Collection c = new ArrayList();

    // 注意：此时获取的迭代器，指向的是那是集合中没有元素状态下的迭代器。
    // 一定要注意：集合结构只要发生改变，迭代器必须重新获取。
    // 当集合结构发生了改变，迭代器没有重新获取时，
    // 调用next()方法时：java.util.ConcurrentModificationException
    Iterator it = c.iterator();

    // 添加元素
    c.add(1); // Integer类型
    c.add(2);
    c.add(3);

    // 获取迭代器
    //Iterator it = c.iterator();
    /*while(it.hasNext()){
            // 编写代码时next()方法返回值类型必须是Object。
            // Integer i = it.next();
            Object obj = it.next();
            System.out.println(obj);
        }*/

    Collection c2 = new ArrayList();
    c2.add("abc");
    c2.add("def");
    c2.add("xyz");

    Iterator it2 = c2.iterator();
    while(it2.hasNext()){
        Object o = it2.next();
        // 删除元素之后，集合的结构发生了变化，应该重新去获取迭代器
        // 但是，循环下一次的时候并没有重新获取迭代器，
        // 所以会出现异常：java.util.ConcurrentModificationException
        // 出异常根本原因是：集合中元素删除了，但是没有更新迭代器（迭代器不知道集合变化了）
        // c2.remove(o); // 直接通过集合去删除元素，没有通知迭代器。（导致迭代器的快照和原集合状态不同。）
        // 使用迭代器来删除可以吗？
        // 迭代器去删除时，会自动更新迭代器，并且更新集合（删除集合中的元素）。
        // *****通过迭代器删除时，会自动更新迭代器，并且更新集合
        it2.remove(); // 删除的一定是迭代器指向的当前元素。
        System.out.println(o);
        // System.out.println(it2.next());
    }

    System.out.println(c2.size()); //0
}
```



## 二、Map

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061510286.png" alt="image-20220406151031801" style="zoom:67%;" />

### 1、HashMap

#### a>添加过程

```
map.put(key1,value1):
1、首先，调用key1所在类的hashCode()计算key1哈希值，
此哈希值经过某种算法计算以后，得到在Entry数组中的存放位置。
2、
如果此位置上的数据为空，此时的key1-value1添加成功。 ----情况1
如果此位置上的数据不为空，(意味着此位置上存在一个或多个数据(以链表形式存在)),
比较key1和已经存在的一个或多个数据的哈希值：
	2.1、如果key1的哈希值与已经存在的数据的哈希值都不相同，
	此时key1-value1添加成功。----情况2
	2.2、如果key1的哈希值和已经存在的某一个数据(key2-value2)的哈希值相同，
	继续比较：调用key1所在类的equals(key2)方法，比较：
		如果equals()返回false:此时key1-value1添加成功。----情况3
		如果equals()返回true:使用value1替换value2。
```

[深入浅出HashMap详解（JDK7）](https://blog.csdn.net/qq_29051413/article/details/107860264)

[深入浅出ConcurrentHashMap详解](https://blog.csdn.net/qq_29051413/article/details/107869427)

1.7 头插

1.8 尾插：可以维护链表原本的顺序

> 头插法的初衷是设计者遵循一个新加进来的元素可能被使用的频率更高，这其实是一个伪命题，因为在hashmap扩容的时候，链表也是会发生颠倒的，因为是先从头节点开始转移掉新的hash表中。
>
> 头插法还有一个致命的缺点，就是在多线程下会出现循环链的情况，导致死循环
>
> 具体是怎么死循环的可以理解为因为扩容是会有一个颠倒的机制，所以多线程操作的时候有可能出现线程1让A->B 而线程2让B->A,导致了循环，
>
> 之所以会出现这个情况，核心在于这样一句代码，e.next = newTable[i];
>
> 这也就是头插法的代码，但是这样会出现一个问题就是，假如在原数组中是a指向b，当线程1完成元素迁移时若是a，b仍在一个索引那么会变成b指向a，但是此时线程二拿到CPU资源，但是在线程2中e指向的是a，那么此时执行e.next = newTable[i];就会变成a指向b，那么就形成了一个循环链表。
>
> jdk1.8之后改为尾插法
> 

#### b>细节

初始容量：16（为了服务于hash函数）

```java
index = HashCode（Key） & （Length - 1）;
```

> 下面我们以值为“book”的Key来演示整个过程：
>
> 1. 计算book的hashcode，结果为十进制的3029737，二进制的101110001110101110 1001。
>
> 2. 假定HashMap长度是默认的16，计算Length-1的结果为十进制的15，二进制的1111。
>
> 3. 把以上两个结果做与运算，101110001110101110 1001 & 1111 = 1001，十进制是9，所以 index=9。
>
> 可以说，Hash算法最终得到的index结果，完全取决于Key的Hashcode值的最后几位。
>
> 长度16或者其他2的幂，Length-1的值是所有二进制位全为1

#### c>扩容机制

1.7版本

> 1. 先⽣成新数组
>
> 2. 遍历⽼数组中的每个位置上的链表上的每个元素
>
> 3. 取每个元素的key，并基于新数组⻓度，计算出每个元素在新数组中的下标
>
> 4. 将元素添加到新数组中去
>
> 5. 所有元素转移完了之后，将新数组赋值给HashMap对象的table属性

1.8版本

> 1. 先⽣成新数组
> 2. 遍历⽼数组中的每个位置上的链表或红⿊树
> 3. 如果是链表，则直接将链表中的每个元素重新计算下标，并添加到新数组中去
> 4. 如果是红⿊树，则先遍历红⿊树，先计算出红⿊树中每个元素对应在新数组中的下标位置
> 	- 统计每个下标位置的元素个数
> 	- 如果该位置下的元素个数超过了8，则⽣成⼀个新的红⿊树，并将根节点的添加到新数组的对应位置
> 	- 如果该位置下的元素个数没有超过8，那么则⽣成⼀个链表，并将链表的头节点添加到新数组的对应位置
>
> 5. 所有元素转移完了之后，将新数组赋值给HashMap对象的table属性

### 2、Map的方法

```java
boolean containsKey(Object key);
// 判断Map中是否包含某个key

boolean containsValue(Object value); 
// 判断Map中是否包含某个value

boolean isEmpty();   
// 判断Map集合中元素个数是否为0

V remove(Object key); 
// 通过key删除键值对

int size(); 
// 获取Map集合中键值对的个数。

Collection<V> values(); 
// 获取Map集合中所有的value，返回一个Collection

Set<K> keySet(); 
// 获取Map集合所有的key（所有的键是一个set集合）

Set<Map.Entry<K,V>> entrySet(); 
// 将Map集合转换成Set集合
```



### 3、Entry

```java
Set<Map.Entry<Integer,String>> set = map.entrySet();
// set里的对象是key=value的形式

// foreach
// 这种方式效率比较高，因为获取key和value都是直接从node对象中获取的属性值。
// 这种方式比较适合于大数据量。
for(Map.Entry<Integer,String> node : set){
    System.out.println(node.getKey() + "--->" + node.getValue());
}
```



### 4、hashCode()和equals()的关系

**hashCode()**的作用是获取哈希码，也称散列码；实际上是一个int整数

哈希码的作用是确定该对象在哈希表中的索引位置。

- 先比较 hashcode，再看需求比较 equals
- 两个对象相等，则 hashcode 一定也是相同的
- 两个对象相等，equals 方法返回 true
- 两个对象拥有相同的 hashcode，也不一定相等
- equals() 重写，hashCode() 也一定要重写
- hashCode() 的默认⾏为是对堆上的对象产⽣独特值。如果没有重写hashCode()，则该class的两个对象⽆论如何都不会相等（即使这两个对象指向相同的数据）



# 集合线程安全问题

## 一、List

```java
// 具体集合类型ArrayList：
// 抛出java.util.ConcurrentModificationException异常

// 具体集合类型Vector：
// 不会抛异常，线程安全，但是这个类太古老

Collections.synchronizedList(new ArrayList<>())：
// 不会抛异常，但是锁定范围大，性能低
public void add(int index, E element) { 
    synchronized (mutex) {
        list.add(index, element);
    } 
}
public E get(int index) { 
    synchronized (mutex) {
        return list.get(index);
    } 
}

// 具体集合类型CopyOnWriteArrayList：
// 使用了写时复制技术，兼顾了线程安全和并发性能
List<String> list = new CopyOnWriteArrayList<>();
```

### 1、写时复制技术

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204072009224.png" alt="image-20220407200925958" style="zoom:80%;" />

使用写时复制技术要向集合对象中写入数据时：

> - 先把整个集合数组复制一份，将新数据写入复制得到的新集合数组
> - 写操作（加锁）在新数组上进行，读操作在旧数组上进行
> - 再让指向集合数组的变量指向新复制的集合数组

优缺点：

- 优点：兼顾了性能和线程安全，允许同时进行读写操作；适合读多写少的情况
- 缺点：
	- 由于需要把集合对象整体复制一份，所以对内存的消耗很大
	- 读到的数据可能不是最新的，不适合实时性很高的场景

对应类中的源代码：

- 所在类：java.util.concurrent.**CopyOnWriteArrayList**

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```



## 二、Set

采用了写时复制技术的Set集合：java.util.concurrent.CopyOnWriteArraySet

```java
// 测试
// 1、创建集合对象
Set<String> set = new CopyOnWriteArraySet<>();
// 2、创建多个线程，每个线程中读写 List 集合
for (int i = 0; i < 5; i++) {
    new Thread(()->{
        for (int j = 0; j < 5; j++) {
            // 写操作：随机生成字符串存入集合
            set.add(UUID.randomUUID().toString().
                    replace("-","").substring(0, 5));
            // 读操作：打印集合整体
            System.out.println("set = " + set);
        }
    }, "thread-"+i).start();
}
```

源码

- 所在类：java.util.concurrent.CopyOnWriteArraySet

```java
public boolean add(E e) {
    return al.addIfAbsent(e);
}
```

- 所在类：java.util.concurrent.CopyOnWriteArrayList

```java
private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] current = getArray();
        int len = current.length;
        if (snapshot != current) {
            // Optimize for lost race to another addXXX operation
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            if (indexOf(e, current, common, len) >= 0)
                return false;
        }
        Object[] newElements = Arrays.copyOf(current, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```



## 三、Map

```java
java.util.concurrent.ConcurrentHashMap
```

```java
// 1、创建集合对象
Map<String, String> map = new ConcurrentHashMap<>();
// 2、创建多个线程执行读写操作
for (int i = 0; i < 5; i++) {
    new Thread(()->{
        for (int j = 0; j < 5; j++) {
            String key = UUID.randomUUID().toString().replace("-","").substring(0, 5);
            String value = UUID.randomUUID().toString().replace("-","").substring(0, 5);
            map.put(key, value);
            System.out.println("map = " + map);
        }
    }, "thread" + i).start();
}
```

### 1、jdk1.7之前Segment

#### 1.1、Segment段

ConcurrentHashMap，它内部细分了若干个小的 HashMap，称之为`段(Segment)`。默认情况下一个 ConcurrentHashMap 被进一步细分为 16 个段，既就是锁的并发度。

如果需要在 ConcurrentHashMap 中添加一个新的表项，并不是将整个 HashMap 加锁，而是首先根据 hashcode 得到该表项应该存放在哪个段中，然后`对该段加锁`，并完成 put 操作。

在多线程环境中，如果多个线程同时进行 put操作，只要被加入的表项不存放在同一个段中，则线程间可以做到真正的并行。

#### 1.2、线程安全（Segment 继承 ReentrantLock 加锁）

简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承ReentrantLock 来进行加锁，所以每次需要加锁的操作**锁住的是一个 segment**，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061546572.png" alt="image-20220406154643937" style="zoom:50%;" />

每个segment也具有红黑树结构

ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成。

Segment 是一种可重入锁 ReentrantLock，在 ConcurrentHashMap 里扮演锁的角色，HashEntry 则用于存储键值对数据。

每个 Segment 守护一个 HashEntry 数组里的元素,当对 HashEntry 数组的数据进行修改时，必须首先获得它对应的 Segment 锁。

> HashTable只能由一个线程操作。 ConcurrentHashMap可以让一个线程操作第一个Segment，另一个线程操作另一个Segment。

#### 1.3、并行度（默认16）

concurrencyLevel：并行级别、并发数、Segment 数。

也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。

`可以初始化赋值，但是初始化后不可扩容`

每个 Segment 很像之前介绍的 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。

### 2、jdk1.8之后Node数组

抛弃了 Segment，改为数组+链表+红黑树的数据结构，乐观锁+synchronized

> 1.7数组存的是HashEntry，1.8数组存的是Node，功能一样

对头节点加锁，从而实现了对每一列数据进行加锁，降低锁粒度

并发控制使用Synchronized和CAS来操作

> 当多线程并发向同一个散列桶添加元素时。
>
> - 若散列桶为空，此时触发乐观锁机制，线程会获取到桶中的版本号，在添加节点之前，判断线程中获取的版本号与桶中实际存在的版本号是否一致，若一致,则添加成功，若不一致，则让线程自旋。
> - 若散列桶不为空，此时使用Synchronized来保证线程安全，先访问到的线程会给桶中的**头节点**加锁，从而保证线程安全。

#### 2.1、为什么是synchronized，而不是ReentrantLock

- **减少内存开销** 

假设使用可重入锁来获得同步支持，那么**每个节点都需要通过继承AQS来获得同步支持**。但并不是每个节点都需要获得同步支持的，**只有链表的头节点（红黑树的根节点）需要同步**，这无疑带来了巨大内存浪费。 

- **获得JVM的支持** 

可重入锁毕竟是API这个级别的，后续的性能优化空间很小。 

synchronized则是JVM直接支持的，JVM能够在运行时作出相应的优化措施：锁粗化、锁消除、锁自旋等等。这就使得**synchronized能够随着JDK版本的升级而不改动代码的前提下获得性能上的提升**。



# 泛型

## 一、泛型中entends和super的区别

1. <? extends T>表示包括T在内的任何T的⼦类 

2. <? super T>表示包括T在内的任何T的⽗类 



# 哈希

## 一、哈希碰撞处理方法

### 1、开放地址

按照一定算法寻找一个空位置存放

- **线性探测再散列**

	依次向后

- **二次探测再散列**

	依次向前后查找，增量为1、2、3的二次方

- **伪随机探测再散列**

	随机产生一个增量位移

### 2、再哈希

出现冲突后采用其他的哈希函数计算，直到不再冲突为止

### 3、链地址

在出现冲突的地方存储一个链表，如 HashMap。

### 4、建立公共溢出区

假设哈希函数的值域是[1,m-1]，则设向量HashTable[0...m-1]为**基本表**，每个分量存放一个记录，另外设向量OverTable[0...v]为**溢出表**，所有关键字和基本表中关键字为同义词的记录，不管它们由哈希函数得到的哈希地址是什么，一旦**发生冲突，都填入溢出表**。

