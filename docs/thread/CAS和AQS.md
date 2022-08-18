# CAS（比较并交换-乐观锁机制-锁自旋）

## 一、概念

CAS（Compare-And-Swap）是`比较并交换`的意思，它是一条 CPU 并发原语，用于判断内存中某个值是否为预期值，如果是则更改为新的值，这个过程是`原子`的。

> **可以保证原子吗？可以**
>
> CAS 是一种`系统原语`，原语属于操作系统用语，原语由若干指令组成，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说 CAS 是一条 CPU 的原子指令，由操作系统硬件来保证。

CAS机制当中使用了3个基本操作数：

- 需要更新的变量（内存值）V
- 旧的预期值E
- 计算后要修改后的新值N

当且仅当 V 值于 E 值时，才会将 V 的值设为 N，如果 V 值和 E 值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后，CAS 返回当前 V 的真实值。

```java
return V = V == E ? N : V;
```

## 二、CAS在Java中的应用

**原子包 java.util.concurrent.atomic（锁自旋）**

这个包里面提供了一组原子类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，<strong style="color:#03468F">即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择一个另一个线程进入</strong>，这只是一种逻辑上的理解。

相对于对于 synchronized 这种阻塞算法，CAS 是非阻塞算法的一种常见实现。由于一般 CPU 切换时间比 CPU 指令集操作更加长， 所以 J.U.C 在性能上有了很大的提升。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204072036985.png" style="zoom: 67%;" />

比如说 AtomicInteger 类就可以解决 i++ 非原子性问题，通过查看源码可以发现主要是靠 volatile 关键字和 CAS 操作来实现。

```java
public class AtomicInteger extends Number implements java.io.Serializable { 
	private volatile int value; 
	public final int get() {  
        return value; 
 	} 
	public final int getAndIncrement() { 
    	for (;;) { //CAS 自旋，一直尝试，直达成功
            int current = get(); 
            int next = current + 1; 
            if (compareAndSet(current, next)) 
                return current; 
        } 
 	} 
 	public final boolean compareAndSet(int expect, int update) { 
 		// JNI
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update); 
 	} 
}
```

## 三、CAS的问题***

### 1、 典型 ABA 问题

ABA 是 CAS 操作的一个经典问题，假设有一个变量初始值为 A，修改为 B，然后又修改为 A，这个变量实际被修改过了，但是 CAS 操作可能无法感知到。

如果是整形还好，不会影响最终结果，但如果是对象的引用类型包含了多个变量，引用没有变实际上包含的变量已经被修改，这就会造成大问题。

解决方法：在变量前加版本号，每次变量更新了就把版本号加一，结果如下：

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204072041052.png" style="zoom: 33%;" />

最终结果都是 A 但是版本号改变了。

从 JDK 1.5 开始提供了`AtomicStampedReference`类，这个类的 `compareAndSet `方法首先检查`当前引用`是否等于`预期引用`，并且`当前标志`是否等于`预期标志`，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

### 2、自旋开销问题

CAS 出现冲突后就会开始`自旋`操作，如果资源竞争非常激烈，自旋长时间不能成功就会给 CPU 带来非常大的开销。

解决方案：可以考虑限制自旋的次数，避免过度消耗 CPU；另外还可以考虑延迟执行。

### 3、只能保证单个变量的原子性

当对一个共享变量执行操作时，可以使用 CAS 来保证原子性，但是如果要对多个共享变量进行操作时，CAS 是无法保证原子性的，比如需要将 i 和 j 同时加 1：

```java
i++;
j++;
```

这个时候可以使用 synchronized 进行加锁，有没有其他办法呢？有，将多个变量操作合成一个变量操作。从 JDK1.5 开始提供了`AtomicReference` 类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

<br>

# AQS（抽象的队列同步器）

[参考](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

AbstractQueuedSynchronizer

AQS 定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的 ReentrantLock、Semaphore、CountDownLatch。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202208111012317.png" alt="img"  />



**AQS核心思想**：如果被请求的共享资源空闲，那么就将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中。

它维护了一个**信号量 volatile int state**（代表共享资源）和一个 **FIFO 线程等待队列**（双向链表队列，多线程争用资源被阻塞时会进入此队列）。这里 volatile 是核心关键词。state 的访问方式有三种：

- getState()
- setState()
- compareAndSetState()

AQS定义两种资源共享方式：

- Exclusive（独占，只有一个线程能执行，如 ReentrantLock）
- Share（共享，多个线程可同时执行，如 Semaphore、CountDownLatch ）。

**AQS 只是一个框架，具体资源的获取/释放方式交由自定义同步器去实现**，AQS 这里只定义了一个接口，具体资源的获取交由自定义同步器去实现了（通过 state 的 get/set/CAS）。

之所以没有定义成abstract ，是因为**独占模式**下只用实现 tryAcquire-tryRelease ，而**共享模式**下只用实现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。不同的自定义同步器争用共享资源的方式也不同。

## 一、实现方式

自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS 已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

1．isHeldExclusively()：该线程是否正在独占资源。只有用到 condition 才需要去实现它。

2．tryAcquire(int)：独占方式。尝试获取资源，成功则返回 true，失败则返回 false。 

3．tryRelease(int)：独占方式。尝试释放资源，成功则返回 true，失败则返回 false。 

4．tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0 表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。

5．tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回 true，否则返回 false。

> **ReentrantRead Write Lock 实现独占利共享两种方式**
>
> 一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现 tryAcquire-tryRelease、tryAcquire Shared-tryReleaseShared 中的一种即可。
>
> 但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock.

## 二、原理

**同步器是实现 AQS 的核心（state 资源状态计数）**

> 以**ReentrantLock**为例，state初始化为0，表示未锁定状态。
>
> A线程lock时，会调用tryAcquire()独占该锁并将state+1。
>
> 此后，其他线程再tryAquire()时就会失败，直到A线程unlock()到state=0(即释放锁)为止，其他线程才有机会获取到该锁。
>
> 释放锁之前，A线程可以重复获取此锁（state++），即可重入。

> 以**CountDownLatch**为例，任务分为 N 个子线程去执行，state也初始化为N（注意N要与线程个数一致）。
>
> 这 N 个子线程是并行执行的，每个子线程执行完后 countDown() 一次，state会 CAS 减 1。
>
> 等到所有子线程都执行完后(即 state =0)，会unpark()主调用线程，然后主调用线程
> 就会从 await() 函数返回，继续后余动作。

## 三、应用场景

- **ReentrantLock**

	使用AQS保存锁重复持有的次数。当一个线程获取锁时，ReentrantLock记录当前获得锁的线程标识，用于检测是否重复获取，以及错误线程试图解锁操作时异常情况的处理。

- **Semaphore**

	使用AQS同步状态来保存信号量的当前计数。tryRelease会增加计数，acquireShared会减少计数。

- **CountDownLatch**

	使用AQS同步状态来表示计数。计数为0时，所有的Acquire操作（CountDownLatch的await方法）才可以通过。

<br>