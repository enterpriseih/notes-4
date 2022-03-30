# Java源程序编译运行过程

Java 源文件 → 编译器 → 字节码文件 .class → JVM →机器码

> 字节码文件
>
> - 不专对特定机器，无需重新编译就可以在不同的机器上运行
> - 以一个字节 8bit 为最小单位储存

# JVM运行机制

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203291127022.png" alt="jvm机制" style="zoom:67%;" />

## 一、线程

线程时进程中执行运算（资源分配）的最小单位，是进程的一个实体

## 二、JVM内存区域

> 栈管运行，堆管存储

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203301347028.png" alt="image-20220330134754626" style="zoom:40%;" />

**线程私有区**【程序计数器、虚拟机栈（Java栈）、本地方法栈】

- 生命周期与线程相同, 依赖用户线程的启动/结束，而创建/销毁在 Hotspot VM 内

**线程共享区**【Java堆（堆空间）、方法区】

- 随虚拟机的启动/关闭而创建/销毁

**直接内存**【不受 JVM GC 管理】

- 不是 JVM 运行时数据区的一部分，但 I/O 可以使用 Native 函数直接分配堆外内存
- 直接内存并不是虚拟机运行时数据区的一部分，也不是Java 虚拟机规范中定义的内存区域。在JDK1.4 中新加入了NIO(New Input/Output)类，引入了一种基于通道（Channel）与缓冲区（Buffer）的 I/O 方式，它可以使用native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆中来回复制数据。 本机直接内存的分配不会受到 Java 堆大小的限制，受到本机总内存大小限制。 配置虚拟机参数时，不要忽略直接内存防止出现 OutOfMemoryError 异常。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203301115606.png" alt="image-20220330111510231" style="zoom: 33%;" />

<br>

### 1、程序计数器 PC寄存器

- 当前线程所执行的字节码的行号指示器，方便时间片结束后再次回来可以继续执行
- 唯一一个没有规定 OOM（OutOfMemoryError）的区域

<br>

### 2、虚拟机栈（Java栈）

> 存放`局部变量`
>
> - 基本数据类型
>
> - 引用数据类型的话存它的`引用`

#### 总体结构

描述java方法执行的内存模型，每个方法在执行的时候都会创建一个栈帧（Stack Frame）

参与方法的调用与返回

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203301118092.png" alt="image-20220330111823493" style="zoom: 33%;" />

#### 栈帧

- 栈中存储数据的基本单位，一个栈帧对应一个方法

##### 栈帧存储的数据

方法在本次执行过程中所用到的局部变量、动态链接、方法出口等信息。栈帧中主要保存3 类数据：

- 本地变量（Local Variables）：输入参数和输出参数以及方法内的变量，**局部变量**。
- 栈操作（Operand Stack）：记录出栈、入栈的操作。
- 栈帧数据（Frame Data）：包括类文件、方法等等。

##### 结构

- 局部变量表：方法执行时的参数、方法体内声明的`局部变量`
- 操作数栈：存储中间运算结果，是一个临时存储空间
- 帧数据区：保存访问**常量池指针**，异常处理表

<br>

### 3、本地方法栈

>虚拟机栈为 Java 方法服务，而本地方法栈为 Native 方法服务。

`HotSpot VM 将本地方法栈和虚拟机栈合二为一`

<br>

### 4、堆

> 创建的`实例对象`和`数组`都保存在堆中，也是 GC 的重要区域

1.8后加上`字符串池`和`静态变量`

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203301316327.png" alt="image-20220330131644568" style="zoom:40%;" />

> - Eden区主要是生命周期很短的对象来来往往
>
> - 老年代主要是生命周期很长的对象，例如：IOC容器对象、线程池对象、数据库连接池对象等等
>
> - 幸存者区作为二者之间的过渡地带

#### 新生代young

- **Eden**：Java 新对象的出生地
	- 如果新创建的对象占用内存很大，则直接分配到老年代
	- 当 Eden 内存不够的时候会触发 MinorGC，对新生代区进行 GC
- **ServivorFrom**：上一次 GC 的幸存者，作为这一次 GC 的被扫描者
- **ServicorTo**：保留了一次 MinorGC 过程中的幸存者
- **MinorGC** 的过程（复制 -> 清空 -> 互换，复制算法）
	- eden、from 中存活的对象复制到 to，年龄 + 1
		- to 满了就直接进 old 
		- 年龄到了15，即进行了15次 GC，进 old
	- 清空 eden、from 中的对象
	- to 和 from 互换指针，即原 to 成为下一次 GC 的 from

> 复制必交换，谁空谁为 to

#### 老年代old

old 中的对象比较稳定，Major GC 不会频繁执行

MajorGC 采用标记清除算法：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。MajorGC 的耗时比较长，因为要扫描再回收。MajorGC 会产生内存碎片，为了减少内存损耗，我们一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候，就会抛出 OOM（Out of Memory）异常。

> **TIP**
>
> Minor GC：清理 young
>
> Major GC：清理 old
>
> Full GC：清理整个堆空间

<br>

### 5、方法区

#### 存放内容

- 类信息：类中定义的构造器、接口定义
- 静态变量（类变量）
- 常量
- 运行时常量池
	- Class 文件中除了有类的版本、字段、方法、接口等描述等信息外，还有一项信息是常量池（Constant Pool Table）
	- 存放编译期生成的各种字面量和符号引用
- 类中方法的代码

#### 概念

- 标准层面：方法区（Method Area）
- 具体实现层面：
	- ≤1.7 永久代（PermGen）：名义上属于堆，实现上不属于堆
		- 1.7 中就已经将字符串池和静态变量移入堆中了
	- ≥1.8 元空间（Meta Space）
		- 元空间使用本地内存
		- 类的元数据（描述代码间关系联系的数据）存入 native memory，`字符串池和类的静态变量放入堆中`

<br>

### Java栈、堆和方法区的关系

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203301255028.png" alt="image-20220330125507742" style="zoom: 33%;" />



<br>

## 三、垃圾回收机制

### 1、如何判断对象死亡

- **引用计数法**

	- 引用一次计数+1，解除引用-1；可能会循环引用，导致计数器无法归零

- **可达性分析**、**根搜索法** GC Roots

	- 核心原理：判断一个对象，是否存在从『堆外』到『堆内』的引用。

	- GC Root 对象：就是`作为根节点`出发，顺着引用路径一直查找到堆空间内，找到堆空间中的对象。
		- Java 栈中的局部变量
		- 本地方法栈中的局部变量
		- 方法区中的类变量、常量

	- 如果在 GC Roots 和一个对象之间没有可达路径，则称该对象不可达
	- 不可达对象经过至少两次标记后，如果仍是可回收对象，则将面临回收



<br>

# JVM类加载机制

加载、验证、准备、解析、初始化

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203301024818.png" alt="image-20220330102400457" style="zoom:67%;" />

1. 加载

	- 这个阶段会在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的入口

2. 验证

	- 确保 Class 文件的字节流中包含的信息是否符合当前虚拟机的要求

3. 准备

	- 在方法区中分配这些变量所使用的内存空间

	```java
	public static int i = 8080;
	// 变量在准备阶段的初始化值为0，在编译阶段才会赋值8080
	public static final int i =8080;
	// final在准备阶段就会赋值
	```

4. 解析

	- 虚拟机将常量池中的符号引用替换为直接引用的过程
		- 符号引用：引用的目标不一定要加载在内存中
		- 直接引用：引用的目标必定已经在内存中了

5. 初始化

	TIP：不会执行初始化的情况

	- 通过子类引用父类的静态字段，只会触发父类的初始化
	- 定义对象数组
	- 通过类名获取 Class 对象
	- 通过 Class.forName 加载指定类时，指定参数 initialize = false
	- 通过 ClassLoader 默认的 loadClass 方法


## 类加载
**启动类加载器 Bootstrap ClassLoader**

- 加载 **$JAVA_HOME/jre/lib** 下的 jar 包，如 rt.jar

**扩展累加载器 Extension ClassLoader**

- 加载\$JAVA_HOME/jre/lib/*.jar 、-Djava.ext.dirs 参数指定目录下的 jar 包、**$JAVA_HOME/jre/lib/ext/classes** 目录下的 class

**应用程序类加载器 Application ClassLoader**

- 加载用户路径 classpath 中指定的 jar 包及目录中的 class

- 自定义类加载器：程序员自己开发一个类继承 java.lang.ClassLoader， 定制类加载方式
	- 加载用户指定目录下的，挂载应用类加载器

### 双亲委派机制

- 当一个类收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求**委派给父类**去完成，
- 每一个层次类加载器都是如此，因此所有的加载请求都应该传送到**启动类加载器**中，
- 只有当父类加载器反馈自己无法完成这个请求的时候（在它的加载路径下没有找到所需加载的Class），子类加载器才会尝试自己去加载。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203301035804.png" alt="image-20220330103519857" />

> **TIP**
>
> - **避免类的重复加载**：父加载器加载了一个类，就不必让子加载器再去查找了。同时也保证了在整个 JVM 范围内全类名是类的唯一标识。
> 	- 比如加载位于 rt.jar 包中的类 java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个 Object 对象。
> - **安全机制**：避免恶意替换 JRE 定义的核心 API
> 	- 比如自定义一个 java.lang.String 类，最终加载的还是启动类加载器加载的String



