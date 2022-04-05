# 变量类型

第一种（按照位置）：局部变量 vs 成员变量（或属性）

- 局部变量

	- 方法内部，生命周期与方法一致
	- 没有默认值，若使用，`必须在使用前显式赋值`
	- 栈内存中

- 成员变量

	- 再分：
		- 类变量/静态变量（static）
			- 准备阶段，默认赋值 —> 初始化阶段，显式赋值
		- 实例变量（无static）
			- 随对象的创建，会在堆空间中分配实例变量空间，并进行默认赋值

	- 类中，方法外
	- 有默认值，规则和数组一致
	- 堆内存中

第二种（按照类型）：基本数据类型 vs 引用数据类型变量（类、数组、接口）



# 常量池

存放

- 字面量
- 符号引用

<br>

1. **(Class)常量池**
  - Class 文件中除了有类的版本、字段、方法、接口等描述等信息外，还有一项信息是常量池（Constant Pool Table），存放编译期生成的各种`字面量`和`符号引用`
  - 每个class文件都有一个class常量池
2. **运行时常量池**（方法区里的重要结构）
  - 运行时常量池存在于内存中，也就是`class常量池被加载到内存之后的版本`，不同之处是：它的字面量可以动态的添加(String.intern())，符号引用可以被解析为直接引用
  - JVM在执行某个类的时候，必须经过加载、连接、初始化，而连接又包括验证、准备、解析三个阶段。而当类加载到内存中后，jvm就会将class常量池中的内容存放到运行时常量池中，由此可知，`运行时常量池也是每个类都有一个`。在解析阶段，会把`符号引用替换为直接引用`，解析的过程会去`查询字符串常量池`，也就是我们上面所说的StringTable，以保证运行时常量池所引用的字符串与字符串常量池中是一致的。

> 字符串常量池
>
> - ≤1.6 方法区中
> 	- String Pool里放的都是字符串常量
> - ≥1.7 移到了堆中
> 	- 由于String#intern()发生了改变，因此String Pool中也可以存放放于堆内的字符串对象的引用
> - 字符串常量池中的字符串只保存一份

## 字面量和符号引用

- 字面量包括：
	- 文本字符串 
	- 八种基本类型的值 
	- 被声明为final的常量等;
- 符号引用包括：
	- 类和方法的全限定名 
	- 字段（属性field）的名称和描述符 
	- 方法的名称和描述符。



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

# final修饰

**final：属性不可变、方法不可覆盖、类不可继承**

final修饰的方法不能重写，但可以继承、重载

重载：方法名字相同，而参数不同。返回类型可以相同也可以不同。

每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。