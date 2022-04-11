- [乐观锁和悲观锁](#乐观锁和悲观锁)
- [独占锁和共享锁](#独占锁和共享锁)
- [互斥锁和读写锁](#互斥锁和读写锁)
- [公平锁和非公平锁](#公平锁和非公平锁)
- [可重入锁](#可重入锁)
- [自旋锁](#自旋锁)
- [分段锁](#分段锁)
- [锁升级（无锁|偏向锁|轻量级锁|重量级锁）](#锁升级无锁偏向锁轻量级锁重量级锁)
- [锁优化技术（锁粗化、锁消除）](#锁优化技术锁粗化锁消除)

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/%E9%82%A3%E4%BA%9B%E9%94%81202204051537100.png" alt="Java中那些眼花缭乱的锁" style="zoom:50%;" />

# 乐观锁和悲观锁

**悲观锁**

一个共享数据加了悲观锁，那线程每次想操作这个数据前都会假设其他线程也可能会操作这个数据，所以每次操作前都会上锁，这样其他线程想操作这个数据拿不到锁只能阻塞了。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051543754.png" alt="20210606232504-2021-06-06-23-25-04" style="zoom:40%;" >

在 Java 语言中 `synchronized` 和 `ReentrantLock`等就是典型的悲观锁，还有一些使用了 synchronized 关键字的容器类如 `HashTable` 等也是悲观锁的应用。

AQS框架下的锁则是先尝试cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如RetreenLock。

**乐观锁**

乐观锁操作数据时不会上锁，在更新的时候会判断一下在此期间是否有其他线程去更新这个数据。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051543803.png" alt="20210606232434-2021-06-06-23-24-35" style="zoom:40%;" >

乐观锁可以使用`版本号机制`和`CAS算法`实现。在 Java 语言中 `java.util.concurrent.atomic`包下的原子类就是使用CAS 乐观锁实现的。

**两种锁的使用场景**

悲观锁和乐观锁没有孰优孰劣，有其各自适应的场景。

乐观锁适用于写比较少（冲突比较小）的场景，因为不用上锁、释放锁，省去了锁的开销，从而提升了吞吐量。

如果是写多读少的场景，即冲突比较严重，线程间竞争激励，使用乐观锁就是导致线程不断进行重试，这样可能还降低了性能，这种场景下使用悲观锁就比较合适。

# 独占锁和共享锁

**独占锁**

`独占锁`是指锁一次只能被一个线程所持有。如果一个线程对数据加上排他锁后，那么其他线程不能再对该数据加任何类型的锁。获得独占锁的线程即能读数据又能修改数据。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051543655.png" alt="20210606232544-2021-06-06-23-25-45" style="zoom:40%;" >

JDK中的`synchronized`和`java.util.concurrent(JUC)`包中Lock的实现类就是独占锁。

**共享锁**

`共享锁`是指锁可被多个线程所持有。如果一个线程对数据加上共享锁后，那么其他线程只能对数据再加共享锁，不能加独占锁。获得共享锁的线程只能读数据，不能修改数据。 

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051542018.png" alt="20210606232612-2021-06-06-23-26-13" style="zoom:40%;" >

在 JDK 中 `ReentrantReadWriteLock` 就是一种共享锁。


# 互斥锁和读写锁

**互斥锁**

`互斥锁`是独占锁的一种常规实现，是指某一资源同时只允许一个访问者对其进行访问，具有唯一性和排它性。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051542734.png" alt="20210606232634-2021-06-06-23-26-35" style="zoom:40%;" >

互斥锁一次只能一个线程拥有互斥锁，其他线程只有等待。

**读写锁**

`读写锁`是共享锁的一种具体实现。读写锁管理一组锁，一个是只读的锁，一个是写锁。

读锁可以在没有写锁的时候被多个线程同时持有，而写锁是独占的。写锁的优先级要高于读锁，一个获得了读锁的线程必须能看到前一个释放的写锁所更新的内容。

读写锁相比于互斥锁并发程度更高，每次只有一个写线程，但是同时可以有多个线程并发读。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051542152.png" alt="20210606232658-2021-06-06-23-26-59" style="zoom:40%;" >

在 JDK 中定义了一个读写锁的接口：`ReadWriteLock`

```java
public interface ReadWriteLock {
    /**
     * 获取读锁
     */
    Lock readLock();

    /**
     * 获取写锁
     */
    Lock writeLock();
}
```

`ReentrantReadWriteLock` 实现了`ReadWriteLock`接口，具体实现这里不展开，后续会深入源码解析。

# 公平锁和非公平锁

**公平锁**

`公平锁`是指多个线程按照申请锁的顺序来获取锁，这里类似排队买票，先来的人先买，后来的人在队尾排着，这是公平的。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051541251.png" alt="20210606232716-2021-06-06-23-27-17" style="zoom:40%;" >

在 java 中可以通过构造函数初始化公平锁

```java
/**
* 创建一个可重入锁，true 表示公平锁，false 表示非公平锁。默认非公平锁
*/
Lock lock = new ReentrantLock(true);
```

**非公平锁**

`非公平锁`是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁，在高并发环境下，有可能造成优先级翻转，或者饥饿的状态（某个线程一直得不到锁）。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051540864.png" alt="20210606232737-2021-06-06-23-27-38" style="zoom: 40%;" >

在 java 中 synchronized 关键字是非公平锁，ReentrantLock默认也是非公平锁。
```java
/**
* 创建一个可重入锁，true 表示公平锁，false 表示非公平锁。默认非公平锁
*/
Lock lock = new ReentrantLock(false);
```

# 可重入锁

`可重入锁`又称之为`递归锁`，是指同一个线程在外层方法获取了锁，在进入内层方法会自动获取锁。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051540621.png" alt="20210606232755-2021-06-06-23-27-56" style="zoom: 40%;" >

对于Java ReentrantLock而言, 他的名字就可以看出是一个可重入锁。对于Synchronized而言，也是一个可重入锁。

敲黑板：可重入锁的一个好处是可一定程度避免死锁。

以 synchronized 为例，看一下下面的代码：

```java
public synchronized void mehtodA() throws Exception{
	// Do some magic tings
	mehtodB();
}

public synchronized void mehtodB() throws Exception{
	// Do some magic tings
}
```
上面的代码中 methodA 调用 methodB，如果一个线程调用methodA 已经获取了锁再去调用 methodB 就不需要再次获取锁了，这就是可重入锁的特性。如果不是可重入锁的话，mehtodB 可能不会被当前线程执行，可能造成死锁。

# 自旋锁

> 如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞挂起状态，它们只需要等一等（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就避免用户线程和内核的切换的消耗。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051539264.png" alt="20210606232809-2021-06-06-23-28-09" style="zoom: 40%;" >

自旋锁的目的是为了减少线程被挂起的几率，因为线程的挂起和唤醒也都是耗资源的操作。

线程自旋是需要消耗 cup 的，说白了就是让 cup 在做无用功，如果一直获取不到锁，那线程也不能一直占用 cup 自旋做无用功，所以需要设定一个**自旋等待的最大时间**。

如果持有锁的线程执行的**时间超过自旋等待的最大时间**仍没有释放锁，就会导致其它争用锁的线程在最大等待时间内还是获取不到锁，这时争用线程会停止自旋**进入阻塞状态**。

在 Java 中，`AtomicInteger` 类有自旋的操作：
```java
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));
    return v;
}
```

CAS 操作如果失败就会一直循环获取当前 value 值然后重试。

**自适应自旋锁**

在JDK1.6又引入了自适应自旋，这个就比较智能了，自旋时间不再固定，`由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定`。如果虚拟机认为这次自旋也很有可能再次成功那就会次序较多的时间，如果自旋很少成功，那以后可能就直接省略掉自旋过程，避免浪费处理器资源。

# 分段锁

`分段锁` 是一种锁的设计，并不是具体的一种锁。

分段锁设计目的是将锁的粒度进一步细化，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204051539351.png" alt="20210606232830-2021-06-06-23-28-31" style="zoom:50%;" />

在 Java 语言中 CurrentHashMap 底层就用了分段锁，使用Segment，就可以进行并发使用了。


# 锁升级（无锁|偏向锁|轻量级锁|重量级锁）

JDK1.6 为了提升性能减少获得锁和释放锁所带来的消耗，引入了4种锁的状态：`无锁`、`偏向锁`、`轻量级锁`和`重量级锁`，它会随着多线程的竞争情况逐渐升级，但不能降级。

**无锁**

`无锁`状态其实就是上面讲的乐观锁，这里不再赘述。

**偏向锁**

第一个尝试加锁的线程，不会真的加锁，而是进入偏向锁状态（很轻量的标记），直到其他线程也来竞争这把锁，才会取消偏向锁状态，真正进行加锁。

`目的是在某个线程获得锁之后，消除这个线程锁重入CAS (Compare and Swap) 的开销。`

偏向锁的实现是通过控制对象`Mark Word`的标志位来实现的，如果当前是`可偏向状态`，需要进一步判断对象头存储的线程 ID 是否与当前线程 ID 一致，如果一致直接进入。

轻量级锁的获取及释放依赖多次 CAS 原子指令，而偏向锁只需要在置换ThreadID 的时候依赖一次 CAS 原子指令



**轻量级锁**

当线程竞争变得比较激烈时，偏向锁就会升级为`轻量级锁`，轻量级锁认为虽然竞争是存在的，但是理想情况下竞争的程度很低，通过`自旋方式`等待上一个线程释放锁。
> - 轻量级锁是为了在没有多线程竞争的前提（只是线程交替执行同步块时）下，减少传统的重量级锁的性能消耗
> - 偏向锁是在只有一个线程执行同步块时进一步提高性能

**重量级锁**

如果线程并发进一步加剧，线程的自旋超过了一定次数，或者一个线程持有锁，一个线程在自旋，又来了第三个线程访问时（反正就是竞争继续加大了），轻量级锁就会膨胀为`重量级锁`，重量级锁会使除了此时拥有锁的线程以外的线程都阻塞。

升级到重量级锁其实就是互斥锁了，一个线程拿到锁，其余线程都会处于阻塞等待状态。

在 Java 中，synchronized 关键字内部实现原理就是锁升级的过程：无锁 --> 偏向锁 --> 轻量级锁 --> 重量级锁。


# 锁优化技术（锁粗化、锁消除）

**锁粗化**

`锁粗化`就是将多个同步块的数量减少，并将单个同步块的作用范围扩大，本质上就是将多次上锁、解锁的请求合并为一次同步请求。

举个例子，一个循环体中有一个代码同步块，每次循环都会执行加锁解锁操作。

```java
private static final Object LOCK = new Object();

for(int i = 0;i < 100; i++) {
    synchronized(LOCK){
        // do some magic things
    }
}
```

经过`锁粗化`后就变成下面这个样子了：

```java
 synchronized(LOCK){
     for(int i = 0;i < 100; i++) {
        // do some magic things
    }
}
```

**锁消除**

`锁消除`是指虚拟机编译器在运行时检测到了共享数据没有竞争的锁，从而将这些锁进行消除。

举个例子让大家更好理解。
```java
public String test(String s1, String s2){
    StringBuffer stringBuffer = new StringBuffer();
    stringBuffer.append(s1);
    stringBuffer.append(s2);
    return stringBuffer.toString();
}
```

上面代码中有一个 test 方法，主要作用是将字符串 s1 和字符串 s2 串联起来。

test 方法中三个变量s1, s2, stringBuffer， 它们都是局部变量，局部变量是在栈上的，栈是线程私有的，所以就算有多个线程访问 test 方法也是线程安全的。

我们都知道 StringBuffer 是线程安全的类，append 方法是同步方法，但是 test 方法本来就是线程安全的，为了提升效率，虚拟机帮我们消除了这些同步锁，这个过程就被称为`锁消除`。


```java
StringBuffer.class

// append 是同步方法
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```



