# ThreadLocal线程本地存储

> ThreadLocal线程本地变量，是Java中所提供的**线程本地存储机制**，可以利⽤该机制**将数据缓存在某个线程内部**，该线程可以在任意时刻、任意⽅法中获取缓存的数据 
>
> 如果你创建了一个`ThreadLocal`变量，那么访问这个变量的每个线程都会有这个变量的一个**本地拷贝**，多个线程操作这个变量的时候，实际是在**操作**自己**本地内存**里面的变量拷贝，从而起到**线程隔离**的作用，避免了并发场景下的线程安全问题。
>
> **线程间变量隔离**

**ThreadLocal**

1. 每个线程中都有一个自己的ThreadLocalMap类对象，可以将线程自己的对象保持到其中，各管各的，线程可以正确的访问到自己的对象。
2. 将一个共用的ThreadLocal静态实例作为key，将不同对象的引用保存到不同线程的ThreadLocalMap中，然后在线程执行的各处通过这个静态ThreadLocal实例的get()方法取得自己线程保存的那个对象，避免了将这个对象作为参数传递的麻烦。

[参考](https://blog.csdn.net/u010445301/article/details/111322569)

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202208010859053.png" alt="图片" style="zoom: 50%;" />

```java
//创建一个ThreadLocal变量
static ThreadLocal<String> localVariable = new ThreadLocal<>();
```

**为什么使用ThreadLocal？**

解决线程安全问题：加锁；本地线程变量

加锁会导致系统变慢。

## 例子

```java
/**
 * 日期工具类
 */
public class DateUtil {
	/* 会出现线程不安全的现象，报错
    private static final SimpleDateFormat simpleDateFormat =
            new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	*/
    private static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal =
            ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    public static Date parse(String dateString) {
        Date date = null;
        try {
            date = dateFormatThreadLocal.get().parse(dateString);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        for (int i = 0; i < 10; i++) {
            executorService.execute(()->{
                System.out.println(DateUtil.parse("2022-07-24 16:34:30"));
            });
        }
        executorService.shutdown();
    }
}
```

这是因为`SimpleDateFormat`不是线性安全的，它以共享变量出现时，并发多线程场景下即会报错。

## 底层原理

**ThreadLocal底层是通过ThreadLocalMap来实现的**

每个Thread对象（注意不是ThreadLocal对象）中都存在⼀个ThreadLocalMap，维护了一个Entry数组。

**Map的key为ThreadLocal对象，Map的value为需要缓存的值** 

- 并发多线程场景下，每个线程`Thread`，在往`ThreadLocal`里设置值的时候，都是往自己的`ThreadLocalMap`里存，读也是以某个`ThreadLocal`作为引用，在自己的`map`里找对应的`key`，从而可以实现了**线程隔离**。

> **不同的线程之间threadlocal这个key值是一样**，
>
> 不同的线程所拥有的ThreadLocalMap是独一无二的，
>
> 不同的线程间同一个ThreadLocal（key）**对应存储的值(value)不一样**，
>
> 同一个线程中这个value变量地址是一样的。



## ThreadLocal与Synchronized的区别

ThreadLocal\<T>其实是与线程绑定的一个变量。ThreadLocal和Synchonized都用于解决多线程并发访问。

1. Synchronized用于线程间的**数据共享**，而ThreadLocal则用于线程间的**数据隔离**。

2. Synchronized是利用锁的机制，使变量或代码块在某一时该只能被一个线程访问。

	而ThreadLocal为每一个线程都提供了变量的副本，使得每个线程在某一时间访问到的并不是同一个对象，这样就隔离了多个线程对数据的数据共享。

<br>

## 为什么不直接用线程id作为ThreadLocalMap的key

```java
public class TianLuoThreadLocalTest {
    private static final ThreadLocal<String> threadLocal1 
        = new ThreadLocal<>();
    private static final ThreadLocal<String> threadLocal2 
        = new ThreadLocal<>();
}
```

如果用id作为key，无法区分是哪个ThreadLocal变量

## 使⽤ThreadLocal会造成内存泄漏

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202208010930313.png" alt="图片" style="zoom:50%;" />

`ThreadLocalMap`使用`ThreadLocal`的**弱引用**作为`key`，当`ThreadLocal`变量被手动设置为`null`，即一个`ThreadLocal`对象没有外部强引用来引用它，当系统GC时，`ThreadLocal`一定会被回收。即ThreadLocal被回收掉。

这样的话，`ThreadLocalMap`中就会出现`key`为`null`的`Entry`，就没有办法访问这些`key`为`null`的`Entry`的`value`，如果当前线程再迟迟不结束的话(比如线程池的核心线程)，这些`key`为`null`的`Entry`的`value`就会一直存在一条**强引用链**：Thread变量 -> Thread对象 -> ThreaLocalMap -> Entry -> value -> Object 永远无法回收，造成内存泄漏。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202208010941512.png" alt="图片" style="zoom:50%;" />

**解决办法**：在`ThreadLocal`的`get`,`set`,`remove`方法，都会清除线程`ThreadLocalMap`里所有`key`为`null`的`Entry`。

## Entry的key为什么是弱引用

- 如果`Key`使用强引用：当`ThreadLocal`的对象被回收了，但是`ThreadLocalMap`还持有`ThreadLocal`的强引用的话，如果没有手动删除，ThreadLocal就不会被回收，会出现Entry的内存泄漏问题。
- 如果`Key`使用弱引用：当`ThreadLocal`的对象被回收了，因为`ThreadLocalMap`持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。`value`则在下一次`ThreadLocalMap`调用`set,get，remove`的时候会被清除。

使用弱引用作为`Entry`的`Key`，可以**多一层保障**：弱引用`ThreadLocal`不会轻易内存泄漏，对应的`value`在下一次`ThreadLocalMap`调用`set,get,remove`的时候会被清除。

## 场景

> - 每个线程需要有自己单独的实例
> - 实例需要在多个方法中共享，但不希望被多线程共享

1. 使用日期工具类，当用到`SimpleDateFormat`，使用ThreadLocal保证线性安全
2. 全局存储用户信息（用户信息存入`ThreadLocal`，那么当前线程在任何地方需要时，都可以使用）
3. 保证同一个线程，获取的数据库连接`Connection`是同一个，使用`ThreadLocal`来解决线程安全的问题
4. 使用`MDC`保存日志信息。

比如：数据库连接、session管理

```java
private static final ThreadLocal threadSession = new ThreadLocal(); 
public static Session getSession() throws InfrastructureException { 
    Session s = (Session) threadSession.get(); 
    try { 
        if (s == null) { 
            s = getSessionFactory().openSession(); 
            threadSession.set(s); 
        } 
    } catch (HibernateException ex) { 
        throw new InfrastructureException(ex); 
    } 
    return s; 
}
```

## 使用

```java
public class ThreadLocaDemo {
 
    private static ThreadLocal<String> localVar 
        = new ThreadLocal<String>();
 
    static void print(String str) {
        //打印当前线程中本地内存中本地变量的值
        System.out.println(str + " :" + localVar.get());
        //清除本地内存中的本地变量
        localVar.remove();
    }
    public static void main(String[] args) throws InterruptedException {
 
        new Thread(new Runnable() {
            public void run() {
                ThreadLocaDemo.localVar.set("local_A");
                print("A");
                //打印本地变量
                System.out.println("after remove : " + localVar.get());
               
            }
        },"A").start();
 
        Thread.sleep(1000);
 
        new Thread(new Runnable() {
            public void run() {
                ThreadLocaDemo.localVar.set("local_B");
                print("B");
                System.out.println("after remove : " + localVar.get());
              
            }
        },"B").start();
    }
}
 
A :local_A
after remove : null
B :local_B
after remove : null
 
```

## 源码分析

`Thread`类源码，可以看到成员变量`ThreadLocalMap`的初始值是为`null`

```java
public class Thread implements Runnable {
   //ThreadLocal.ThreadLocalMap是Thread的属性
   ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

ThreadLocalMap的关键源码

```java
static class ThreadLocalMap {
    
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    //Entry数组
    private Entry[] table;
    
    // ThreadLocalMap的构造器，ThreadLocal作为key
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }
}
```

ThreadLocal中的set()方法

```java
public void set(T value) {
    Thread t = Thread.currentThread(); //获取当前线程t
    ThreadLocalMap map = getMap(t);  //根据当前线程获取到ThreadLocalMap
    if (map != null)  //如果获取的ThreadLocalMap对象不为空
        map.set(this, value); //K，V设置到ThreadLocalMap中
    else
        createMap(t, value); //创建一个新的ThreadLocalMap
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals; //返回Thread对象的ThreadLocalMap属性
}

void createMap(Thread t, T firstValue) { //调用ThreadLocalMap的构造函数
    t.threadLocals = new ThreadLocalMap(this, firstValue); this表示当前类ThreadLocal
}
```

ThreadLocal中的get()方法

```java
public T get() {
    Thread t = Thread.currentThread();//获取当前线程t
    ThreadLocalMap map = getMap(t);//根据当前线程获取到ThreadLocalMap
    if (map != null) { //如果获取的ThreadLocalMap对象不为空
        //由this（即ThreadLoca对象）得到对应的Value，即ThreadLocal的泛型值
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value; 
            return result;
        }
    }
    return setInitialValue(); //初始化threadLocals成员变量的值
}

private T setInitialValue() {
    T value = initialValue(); //初始化value的值
    Thread t = Thread.currentThread(); 
    ThreadLocalMap map = getMap(t); //以当前线程为key，获取threadLocals成员变量，它是一个ThreadLocalMap
    if (map != null)
        map.set(this, value);  //K，V设置到ThreadLocalMap中
    else
        createMap(t, value); //实例化threadLocals成员变量
    return value;
}
```





