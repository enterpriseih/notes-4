# 基础的基础

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

# StringTable

## 一、字符串的拼接

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



## 二、intern()

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

### 4、ConcurrentHashMap
线程安全

多线程并发中详细



### 5、hashCode()和equals()的关系

**hashCode()**的作用是获取哈希码，也称散列码；实际上是一个int整数

哈希码的作用是确定该对象在哈希表中的索引位置。

- 先比较 hashcode，再看需求比较 equals
- 两个对象相等，则 hashcode 一定也是相同的
- 两个对象相等，equals 方法返回 true
- 两个对象拥有相同的 hashcode，也不一定相等
- equals() 重写，hashCode() 也一定要重写
- hashCode() 的默认⾏为是对堆上的对象产⽣独特值。如果没有重写hashCode()，则该class的两个对象⽆论如何都不会相等（即使这两个对象指向相同的数据）



# 泛型

## 一、泛型中entends和super的区别

1. <? extends T>表示包括T在内的任何T的⼦类 

2. <? super T>表示包括T在内的任何T的⽗类 
