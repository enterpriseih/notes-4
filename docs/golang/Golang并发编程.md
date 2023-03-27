# sync包介绍

sync包提供了基本的并发编程同步原语(concurrency primitives or synchronization primitives)，例如互斥锁sync.Mutex。sync包囊括了以下数据类型：

* sync.Cond
* sync.Locker
* sync.Map
* sync.Mutex
* sync.Once
* sync.Pool
* sync.RWMutex
* sync.WaitGroup

除了sync.Once和sync.WaitGroup这2个类型之外，其它类型主要给一些偏底层的库程序用。业务代码里的goroutine同步，Go设计者是建议通过channel通信来实现。

# sync.WaitGroup

## 定义

WaitGroup是sync包里的一个结构体类型，定义如下

```go
type WaitGroup struct {
    // some fields
}
```

这个结构体有如下3个方法

* Add：

	```go
	func (wg *WaitGroup) Add(delta int)
	```

	

* Done：Done调用会将WiatGroup的计数器减1

	```go
	func (wg *WaitGroup) Done()
	```

	

* Wait：Wait调用会阻塞，直到WaitGroup的计数器为0

	```go
	func (wg *WaitGroup) Wait()
	```

定义一个WaitGroup变量的目的是为了等待若干个goroutine执行完成，主goroutine调用Add方法，指明要等待的子goroutine数量，这些子goroutine执行完成后调用Done方法。同时，主goroutine要调用Wait方法阻塞程序，等WaitGroup的计数器减小到0时，Wait方法不再阻塞。

示例如下：

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

var wg sync.WaitGroup

func worker(id int) {
    /*worker执行完成后，会调用Done将wg计数器减1*/
    defer wg.Done()
    fmt.Printf("worker %d starting\n", id)
    time.Sleep(time.Second)
    fmt.Printf("worker %d done\n", id)
}

func main() {
    /* wg跟踪10个goroutine */
    size := 10
    wg.Add(size)
    /* 开启10个goroutine并发执行 */
    for i:=0; i<size; i++ {
        go worker(i)
    }
    /* Wait一直阻塞，直到wg的计数器变为0 */
    wg.Wait()
    fmt.Println("end")
}
```

* **注意事项**

	* WaitGroup不要拷贝传值，如果要显式地把WaitGroup作为函数参数，**一定要传指针**。

		WaitGroup给函数A传值，在函数A内部这个WaitGroup会是一个局部变量，对WaitGroup的操作只会在函数内部生效。示例如下：

		```go
		package main
		
		import (
		    "fmt"
		    "sync"
		    "time"
		)
		
		
		func worker(id int, wg sync.WaitGroup) {
		    /*worker执行完成后，会调用Done将wg计数器减1*/
		    defer wg.Done()
		    fmt.Printf("worker %d starting\n", id)
		    time.Sleep(time.Second)
		    fmt.Printf("worker %d done\n", id)
		}
		
		func main() {
		    var wg sync.WaitGroup
		    /* wg跟踪10个goroutine */
		    size := 10
		    wg.Add(size)
		    /* 开启10个goroutine并发执行 */
		    for i:=0; i<size; i++ {
		        go worker(i, wg)
		    }
		    /* 这个例子里Wait会一直阻塞，因为函数worker内部的Done调用对外部的wg其实不生效*/
		    wg.Wait()
		    fmt.Println("end")
		}
		
		```

		程序运行时wg.Wait()会报错：fatal error: all goroutines are asleep - deadlock!

		改为下面的传指针就正常了：

		```go
		package main
		
		import (
		    "fmt"
		    "sync"
		    "time"
		)
		
		
		func worker(id int, wg *sync.WaitGroup) {
		    /*worker执行完成后，会调用Done将wg计数器减1*/
		    defer wg.Done()
		    fmt.Printf("worker %d starting\n", id)
		    time.Sleep(time.Second)
		    fmt.Printf("worker %d done\n", id)
		}
		
		func main() {
		    var wg sync.WaitGroup
		    /* wg跟踪10个goroutine */
		    size := 10
		    wg.Add(size)
		    /* 开启10个goroutine并发执行 */
		    for i:=0; i<size; i++ {
		        /*wg传指针给worker*/
		        go worker(i, &wg)
		    }
		    /* Wait会一直阻塞，直到wg的计数器为0*/
		    wg.Wait()
		    fmt.Println("end")
		}
		```


## references

* https://pkg.go.dev/sync@go1.17.2

# sync.Once

## 定义

Once是sync包里的一个结构体类型，Once可以在并发场景下让某个操作只执行一次，比如设计模式里的单例只创建一个实例，比如只加载一次配置文件，比如对同一个channel只关闭一次（对一个已经close的channel再次close会引发panic）等。

定义如下：

```go
type Once struct {
    // some fields
}
```

这个结构体只有1个方法Do，参数是要执行的函数。（**注意**：参数是函数类型，而不是函数的返回值，所以只需要把函数名作为参数给到Do即可）

可以看到Do方法的参数**f**这个函数类型没有参数，所以如果要执行的函数f需要传递参数就要结合Go的闭包来使用。

```go
func(o *Once) Do(f func())
```

参考下面的例子，print函数通过Once执行，只会执行1次

```go
package main

import (
    "fmt"
    "sync"
)

func print() {
    fmt.Println("test once")
}

func main() {
    var wg sync.WaitGroup
    var once sync.Once
    size := 10
    wg.Add(size)
    
    /*启用size个goroutine，每个goroutine都调用once.Do(print)
    最终print只会执行一次
    */
    for i:=0; i<size; i++ {
        go func() {
            defer wg.Done()
            once.Do(print)
        }()
    }
    /*等待所有goroutine执行完成*/
    wg.Wait()
    fmt.Println("end")
}
```



## sync.Once实现并发安全的单例

```go
package main

import (
    "fmt"
    "sync"
)

type Singleton struct {
    member int
}

var instance *Singleton

var once sync.Once

func getInstance() *Singleton {
    /*
    通过sync.Once实现单例，只会生成一个Singleton实例
    */
    once.Do(func() {
        fmt.Println("once")
        instance = &Singleton{}
        instance.member = 100
    })
    fmt.Println(instance.member)
    return instance
}

func main() {
    var wg sync.WaitGroup
    size := 10
    wg.Add(size)
    /*
    多个goroutine同时去获取Singelton实例
    */
    for i:=0; i<size; i++ {
        go func() {
            defer wg.Done()
            instance = getInstance()
        }()
    }
    wg.Wait()
    fmt.Println("end")
}
```



## 注意事项

* Once变量作为函数参数传递时，只能传指针，不能传值。传值给函数A的话，对于函数A而言，参数列表里的once形参会是一个新生成的once局部变量，和外部传入的once实参不一样。

	```go
	package main
	
	import (
	    "fmt"
	    "sync"
	)
	
	func test() {
	    fmt.Println("test once")
	}
	
	func print(once *sync.Once) {
	    once.Do(test)
	}
	
	func main() {
	    var wg sync.WaitGroup
	    var once sync.Once
	    size := 10
	    wg.Add(size)
	    
	    /*启用size个goroutine，每个goroutine都调用once.Do(print)
	    最终print只会执行一次
	    */
	    for i:=0; i<size; i++ {
	        go func() {
	            defer wg.Done()
	            print(&once)
	        }()
	    }
	    /*等待所有goroutine执行完成*/
	    wg.Wait()
	    fmt.Println("end")
	}
	```

	

* 如果once.Do(f)方法调用的函数**f**发生了panic，那Do也会认为函数**f**已经return了。

* 如果多个goroutine执行了都去调用once.Do(f)，只有某次的函数**f**调用返回了，所有Do方法调用才会返回，否则Do方法会一直阻塞等待。如果在f里继续调用同一个once变量的Do方法，就会死锁了，因为Do在等待**f**返回，**f**又在等待Do返回。

# sync.Mutex和sync.RWMutex

## sync.Mutex

### 定义

Mutex是sync包里的一个结构体类型，含义就是互斥锁。Mutex变量的默认值或者说零值是一个没有加锁的mutex，也就是当前mutex的状态是unlocked。

**不要对Mutex使用值传递方式进行函数调用**。

Mutex允许一个goroutine对其加锁，其它goroutine对其解锁，不要求加锁和解锁在同一个goroutine里。

Mutex结构体类型有2个方法

* Lock()加锁。Lock()方法会把Mutex变量m锁住，如果m已经锁住了，如果再次调用Lock()就会阻塞，直到锁释放。

	```go
	func (m *Mutex) Lock()
	```

* Unlock()解锁。Unlock()方法会把Mutex变量m解锁，如果m没有被锁，还去调用Unlock，会遇到runtime error。

	```go
	func (m *Mutex) Unlock()
	```

### 不加锁

* 场景举例：多个 goroutine对共享变量同时执行写操作，并发是不安全的，结果和预期不符。

* 示例代码

	```go
	package main
	
	import (
		"fmt"
		"sync"
	)
	
	var sum int = 0
	
	/*多个goroutine同时访问add
	sum是多个goroutine共享的
	也就是多个goroutine同时对共享变量sum做写操作不是并发安全的
	*/
	func add(i int) {
		sum += i
	}
	
	func main() {
		var wg sync.WaitGroup
		size := 100
		wg.Add(size)
		for i:=1; i<=size; i++ {
			i := i
			go func() {
				defer wg.Done()
				add(i)
			}()
		}
		wg.Wait()
		fmt.Printf("sum of 1 to %d is: %d\n", size, sum)
	}
	```

### 加锁

* 示例代码，通过对共享变量加互斥锁来保证并发安全，结果和预期相符。

	```go
	package main
	
	import (
		"fmt"
		"sync"
	)
	
	var sum int = 0
	var mutex sync.Mutex
	/*多个goroutine同时访问add
	sum是多个goroutine共享的
	通过加互斥锁来保证并发安全
	*/
	func add(i int) {
		mutex.Lock()
		defer mutex.Unlock()
		sum += i
	}
	
	func main() {
		var wg sync.WaitGroup
		size := 100
		wg.Add(size)
		for i:=1; i<=size; i++ {
			i := i
			go func() {
				defer wg.Done()
				add(i)
			}()
		}
		wg.Wait()
		fmt.Printf("sum of 1 to %d is: %d\n", size, sum)
	}
	```




## sync.RWMutex

### 定义

RWMutex是sync包里的一个结构体类型，含义是读写锁。RWMutex变量的零值是一个没有加锁的mutex。

不要对RWMutex变量使用值传递的方式进行函数调用。

RWMutex允许一个goroutine对其加锁，其它goroutine对其解锁，不要求加锁和解锁在同一个goroutine里。

RWMutex结构体类型的定义如下：

```go
type RWMutex struct {
    // some fields
}
```

RWMutex结构体类型有5个方法：

* Lock()，加写锁。某个goroutine加了写锁后，其它goroutine不能获取读锁，也不能获取写锁

	```go
	func (rw *RWMutex) Lock()
	```

* Unlock()，释放写锁。

	```go
	func (rw *RWMutex) Unlock()
	```

* RLock()，加读锁。某个goroutine加了读锁后，其它goroutine可以获取读锁，但是不能获取写锁

	```go
	func (rw *RWMutex) RLock()
	```

* RUnlock()，释放读锁

	```go
	func (rw *RWMutex) RUnlock()
	```

* RLocker()，获取一个类型为Locker的接口，Locker类型定义了Lock()和Unlock()方法

	```go
	func (rw *RWMutex) RLocker() Locker
	```

	类型Locker的定义如下

	```go
	type Locker interface {
	    Lock()
	    Unlock()
	}
	```

	Mutex和RWMutex这2个结构体类型实现了Locker这个interface里的所有方法，因此可以把Mutex和RWMutex变量或者指针赋值给Locker实例，然后通过Locker实例来加锁和解锁，这个在条件变量sync.Cond里会用到，可以参考[sync.Cond](./workspace/lesson24)

### 示例

```go
package main

import (
    "fmt"
    "sync"
)

type Counter struct {
    /*
    成员count:计数器
    成员rw: 读写锁，用于实现count的读写并发安全
    */
    count int
    rw sync.RWMutex
}

func (c *Counter) getCounter() int{
    /*
    读数据的时候加读锁
    */
    c.rw.RLock()
    defer c.rw.RUnlock()
    return c.count
}

func (c *Counter) add() {
    /*
    写数据的时候加写锁
    */
    c.rw.Lock()
    defer c.rw.Unlock()
    c.count++
}

func main() {
    var wg sync.WaitGroup
    size := 100
    wg.Add(size)
    
    var c Counter
    /*
    开启size个goroutine对变量c的数据成员count同时进行读写操作
    */
    for i:=0; i<size; i++ {
        go func() {
            defer wg.Done()
            c.getCounter()
            c.add()
        }()
    }
    wg.Wait()
    fmt.Println("count=", c.count)
}
```



## 注意事项

* Mutex和RWMutex都不是递归锁，不可重入

## References

* https://pkg.go.dev/sync@go1.17.2#Mutex

* https://pkg.go.dev/sync@go1.17.2#RWMutex

# sync.Cond

## 定义

Cond是sync包里的一个结构体类型，表示条件变量。我们知道sync.WaitGroup可以用于等待所有goroutine都执行完成，**sync.Cond可以用于控制goroutine什么时候开始执行**。

Cond结构体类型定义如下：

```go
type Cond struct {
    L Locker
    // some other fields
}
```

Cond结构体类型以下几个方法与其紧密相关：

* NewCond函数，用于创建条件变量，条件变量的成员`L`是NewCond函数的参数`l`

	```go
	func NewCond(l Locker) *Cond
	```

	

* Broadcast，发出广播，**唤醒所有**等待条件变量c的goroutine开始执行。**注意**：在调用Broadcast方法之前，要确保目标goroutine处于Wait阻塞状态，不然会出现死锁问题。

	```go
	func (c *Cond) Broadcast()
	```

	

* Signal，发出信号，**唤醒某一个**等待条件变量c的goroutine开始执行。**注意**：在调用Signal方法之前，要确保目标goroutine处于Wait阻塞状态，不然会出现死锁问题。

	```go
	func (c *Cond) Signal()
	```

	

* Wait，这个方法会解锁c.L以及阻塞当前goroutine往下执行，解锁和阻塞组合在一起构成原子操作。Wait被Broadcast或者Signal唤醒时，会先对c.L加锁，然后Wait才return返回。

	```go
	func (c *Cond) Wait()
	```

	

每个Cond变量都有一个Locker类型的成员L，L通常是\*Mutex或者\*RWMutex类型，**调用Wait方法时要对L加锁**。

不要对Cond变量使用值传递进行函数调用。

## 示例

下面这个示例，先开启了10个goroutine，这10个goroutine都进入Wait阻塞状态，等待被唤醒。

```go
package main

import (
    "fmt"
    "sync"
    "time"
)


func main() {
    var wg sync.WaitGroup
    /**/
    var mutex sync.Mutex
    cond := sync.NewCond(&mutex)
    size := 10
    wg.Add(size+1)
    
    for i:=0; i<size; i++ {
        i := i
        go func() {
            defer wg.Done()
            /*调用Wait方法时，要对L加锁*/
            cond.L.Lock()
            fmt.Printf("%d ready\n", i)
            /*Wait实际上是会先解锁cond.L，再阻塞当前goroutine
            这样其它goroutine调用上面的cond.L.Lock()才能加锁成功，
            才能进一步执行到Wait方法，
            等待被Broadcast或者signal唤醒。
            Wait被Broadcast或者Signal唤醒的时候，会再次对cond.L加锁，
            加锁后Wait才会return
            */
            cond.Wait()
            fmt.Printf("%d done\n", i)
            cond.L.Unlock()
        }()
    }
    
    /*这里sleep 2秒，确保目标goroutine都处于Wait阻塞状态
    如果调用Broadcast之前，目标goroutine不是处于Wait状态，会死锁
    */
    time.Sleep(2*time.Second)
    go func() {
        defer wg.Done()
        cond.Broadcast()
    }()
    wg.Wait()
}
```



## References

https://pkg.go.dev/sync@go1.17.2#Cond

# sync.Map

## 定义

Map是sync包里的一个结构体类型，定义如下

```go
type Map struct {
    // some fields
}
```

Go语言里普通map的读写不是并发安全的，sync.Map的读写是并发安全的。

sync.Map可以理解为类似一个map[interface{}]interface{}的结构，key可以类型不一样，value也可以类型不一样，多个goroutine对其进行读写不需要额外加锁。

**Go官方设计sync.Map主要满足以下2个场景的用途**

1. **每个key只写一次，其它对该key的操作都是读操作**

2. **多个goroutine同时读写map，但是每个goroutine只读写各自的keys**

以上2种场景，相对于对普通的map加Mutex或者RWMutex来实现并发安全，使用sync.Map不用在业务代码里加锁，会大幅减少锁竞争，提升性能。**其它更为常见的场景还是使用普通的Map，搭配Mutex或者RWMutex来使用**。

**不能对sync.Map使用值传递方式进行函数调用。**

sync.Map结构体类型有如下几个方法：

* Delete，删除map里的key，即使key不存在，执行Delete操作也没任何影响

	```go
	func (m *Map) Delete(key interface{})
	```

* Load，从map里取出key对应的value。如果key存在map里，返回值value就是对应的值，ok就是true。如果key不在map里，返回值value就是nil，ok就是false。

	```go
	func (m* Map) Load(key interface{}) (value interface{}, ok bool)
	```

* LoadAndDelete，删除map里的key。如果key存在map里，返回值value就是对应的值，loaded就是true。如果key不在map里，返回值value就是nil，loaded就是false。

	```go
	func (m* Map) LoadAndDelete(key interface{}) (value interface{}, loaded bool)
	```

* LoadOrStore，从map里取出key对应的value。如果key在map里不存在，就把LoadOrStrore函数调用传入的参数<key, value>存储到map里，并返回参数里的value。如果key在map里，那loaded是true，如果key不在map里，那loaded是false。

	```go
	func (m* Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
	```

* Range，遍历map里的所有<key, value>对，把每个<key, value>对，都作为参数传递给**f**去调用，如果遍历执行过程中，**f**返回false，那range迭代就结束了。

	```go
	func (m* Map) Range(f func(key, value interface{}) bool)
	```

* Store，往map里插入<key, vaue>对，即使key已经存在于map里，也没有任何影响

	```go
	func (m* Map) Store(key, value interface{})
	```

Delete, Load, LoadAndDelete, LoadOrStore, Store的均摊时间复杂度是O(1)，Range的时间复杂度是O(N)

## 使用

* 初始化

	```go
	var m1 sync.Map
	m2 := sync.Map{}
	```

	

* 示例1：统计字符串里每个字符出现的次数

	```go
	package main
	
	import (
	    "fmt"
	    "sync"
	)
	
	func main() {
	    /*统计字符串里每个字符出现的次数*/
	    m := sync.Map{}
	    str := "abcabcd"
	    for _, value := range str {
	        temp, ok := m.Load(value)
	        //fmt.Println(temp, ok)
	        if !ok {
	            m.Store(value, 1)
	        } else {
	            /*temp是个interface变量，要转int才能和1做加法*/
	            m.Store(value, temp.(int)+1)
	        }
	    }
	    
	    /*使用sync.Map里的Range遍历map*/
	    m.Range(func(key, value interface{}) bool{
	        fmt.Println(key, value)
	        return true
	    })
	}
	```

* 示例2：多个goroutine并发写sync.Map，不加锁。如果是普通的map，这么来写就会出现运行时错误“fatal error: concurrent map writes”

	```go
	package main
	
	import (
	    "fmt"
	    "sync"
	)
	
	var m sync.Map
	
	/*
	sync.Map里每个key只写一次，属于场景1
	*/
	func changeMap(key int) {
	    m.Store(key, 1)
	}
	
	func main() {
	    var wg sync.WaitGroup
	    size := 2
	    wg.Add(size)
	    
	    for i:=0; i<size; i++ {
	        i := i
	        go func() {
	            defer wg.Done()
	            changeMap(i)
	        }()
	    }
	    wg.Wait()
	    
	    /*使用sync.Map里的Range遍历map*/
	    m.Range(func(key, value interface{}) bool{
	        fmt.Println(key, value)
	        return true
	    })
	}
	```

## 注意事项

* sync.Map不支持len和cap函数

* 在评估要不要使用sync.Map的时候，先考察业务场景是否符合上面描述的场景1和2，符合再考虑用sync.Map，不符合就用普通map+Mutex或者RWMutex。

## References

https://pkg.go.dev/sync@go1.17.2#Map

# 原子操作sync/atomic

## sync/atomic定义

官方文档地址：https://pkg.go.dev/sync/atomic@go1.18.1

Go语言标准库中的`sync/atomic`包提供了偏底层的原子内存原语(atomic memory primitives)，用于实现同步算法，其本质是将底层CPU提供的原子操作指令封装成了Go函数。

使用`sync/atomic`提供的原子操作可以确保在任意时刻只有一个goroutine对变量进行操作，避免并发冲突。

使用`sync/atomic`需要特别小心，Go官方建议只有在一些偏底层的应用场景里才去使用`sync/atomic`，其它场景建议使用`channel`或者`sync`包里的锁。

> Share memory by communicating; don't communicate by sharing memory.

`sync/atomic`提供了5种类型的原子操作和1个`Value`类型。

### 5种类型的原子操作

* swap操作：`SwapXXX`
* compare-and-swap操作：`CompareAndSwapXXX`
* add操作：`AddXXX`
* load操作：`LoadXXX`
* store操作：`StoreXXX`

这几种类型的原子操作只支持几个基本的数据类型。

add操作的`Addxxx`函数只支持`int32`, `int64`, `uint32`, `uint64`, `uintptr`这5种基本数据类型。

其它类型的操作函数只支持`int32`, `int64`, `uint32`, `uint64`, `uintptr`, `unsafe.Pointer`这6种基本数据类型。

### Value类型

由于上面5种类型的原子操作只支持几种基本的数据类型，因此为了扩大原子操作的使用范围，Go团队在1.14版本的`sync/atomic`包中引入了一个新的类型`Value`。`Value`类型可以用来读取(Load)和修改(Store)**任意类型**的值。

Go 1.14版本的`Value`类型只有`Load`和`Store`2个方法，Go 1.17版本又给`Value`类型新增了`CompareAndSwap`和`Swap`这2个新方法。

## sync/atomic实践

### swap操作

swap操作支持`int32`, `int64`, `uint32`, `uint64`, `uintptr`, `unsafe.Pointer`这6种基本数据类型，对应有6个swap操作函数。

```go
func SwapInt32(addr *int32, new int32) (old int32)
func SwapInt64(addr *int64, new int64) (old int64)
func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)
func SwapUint32(addr *uint32, new uint32) (old uint32)
func SwapUint64(addr *uint64, new uint64) (old uint64)
func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
```

swap操作实现的功能是把`addr` 指针指向的内存里的值替换为新值`new`，然后返回旧值`old`，是如下伪代码的原子实现：

```go
old = *addr
*addr = new
return old
```

我们拿`SwapInt32`举个例子：

```go
// swap.go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	var newValue int32 = 200
	var dst int32 = 100
	// 把dst的值替换为newValue
	old := atomic.SwapInt32(&dst, newValue)
	// 打印结果
	fmt.Println("old value: ", old, " new value:", dst)
}
```

上面程序的执行结果如下：

```bash
old value:  100  new value: 200
```

### compare-and-swap操作

compare-and-swap(CAS)操作支持`int32`, `int64`, `uint32`, `uint64`, `uintptr`, `unsafe.Pointer`这6种基本数据类型，对应有6个compare-and-swap操作函数。

```go
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)
func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)
func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)
func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)
func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)
```

compare-and-swap操作实现的功能是先比较`addr` 指针指向的内存里的值是否为旧值`old`相等。

* 如果相等，就把`addr`指针指向的内存里的值替换为新值`new`，并返回`true`，表示操作成功。
* 如果不相等，直接返回`false`，表示操作失败。

compare-and-swap操作是如下伪代码的原子实现：

```go
if *addr == old {
	*addr = new
	return true
}
return false
```

我们拿`CompareAndSwapInt32`举个例子：

```go
// compare-and-swap.go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	var dst int32 = 100
	oldValue := atomic.LoadInt32(&dst)
	var newValue int32 = 200
	// 先比较dst的值和oldValue的值，如果相等，就把dst的值替换为newValue
	swapped := atomic.CompareAndSwapInt32(&dst, oldValue, newValue)
	// 打印结果
	fmt.Printf("old value: %d, swapped value: %d, swapped success: %v\n", oldValue, dst, swapped)
}
```

上面程序的执行结果如下：

```bash
old value: 100, swapped value: 200, swapped success: true
```

### add操作

add操作支持`int32`, `int64`, `uint32`, `uint64`, `uintptr`这5种基本数据类型，对应有5个add操作函数。

```go
func AddInt32(addr *int32, delta int32) (new int32)
func AddInt64(addr *int64, delta int64) (new int64)
func AddUint32(addr *uint32, delta uint32) (new uint32)
func AddUint64(addr *uint64, delta uint64) (new uint64)
func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)
```

add操作实现的功能是把`addr` 指针指向的内存里的值和`delta`做加法，然后返回新值，是如下伪代码的原子实现：

```go
*addr += delta
return *addr
```

我们拿`AddInt32`举个例子：

```go
// add.go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

var wg sync.WaitGroup

// 多个goroutine并发读写sum，有并发冲突，最终计算得到的sum值是不准确的
func test1() {
	var sum int32 = 0
	N := 100
	wg.Add(N)
	for i := 0; i < N; i++ {
		go func(i int32) {
			sum += i
			wg.Done()
		}(int32(i))
	}
	wg.Wait()
	fmt.Println("func test1, sum=", sum)
}

// 使用原子操作计算sum，没有并发冲突，最终计算得到sum的值是准确的
func test2() {
	var sum int32 = 0
	N := 100
	wg.Add(N)
	for i := 0; i < N; i++ {
		go func(i int32) {
			atomic.AddInt32(&sum, i)
			wg.Done()
		}(int32(i))
	}
	wg.Wait()
	fmt.Println("func test2, sum=", sum)
}

func main() {
	test1()
	test2()
}
```

上面程序的执行结果如下：

```bash
func test1, sum= 4857
func test2, sum= 4950
```

**注意**：对于test1函数，你本地运行得到的结果可能和我的不一样，这个值并不是一个固定值。

### load操作

load操作支持`int32`, `int64`, `uint32`, `uint64`, `uintptr`, `unsafe.Pointer`这6种基本数据类型，对应有6个load操作函数。

```go
func LoadInt32(addr *int32) (val int32)
func LoadInt64(addr *int64) (val int64)
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
func LoadUint32(addr *uint32) (val uint32)
func LoadUint64(addr *uint64) (val uint64)
func LoadUintptr(addr *uintptr) (val uintptr)
```

load操作实现的功能是返回`addr` 指针指向的内存里的值，是如下伪代码的原子实现：

```go
return *addr
```

我们拿`LoadInt32`举个例子：

```go
// load.go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	var sum int32 = 100
	result := atomic.LoadInt32(&sum)
	fmt.Println("result=", result)
}
```

上面程序的执行结果如下：

```bash
result= 100
```

### store操作

store操作支持`int32`, `int64`, `uint32`, `uint64`, `uintptr`, `unsafe.Pointer`这6种基本数据类型，对应有6个store操作函数。

```go
func StoreInt32(addr *int32, val int32)
func StoreInt64(addr *int64, val int64)
func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)
func StoreUint32(addr *uint32, val uint32)
func StoreUint64(addr *uint64, val uint64)
func StoreUintptr(addr *uintptr, val uintptr)
```

store操作实现的功能是把`addr` 指针指向的内存里的值修改为`val`，是如下伪代码的原子实现：

```go
*addr = val
```

我们拿`StoreInt32`举个例子：

```go
// store.go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	var sum int32 = 100
	var newValue int32 = 200
	// 将sum的值修改为newValue
	atomic.StoreInt32(&sum, newValue)
	// 读取修改后的sum值
	result := atomic.LoadInt32(&sum)
	// 打印结果
	fmt.Println("result=", result)
}
```

上面程序的执行结果如下：

```bash
result= 200
```

#### Value类型

Go标准库里的`sync/atomic`包提供了`Value`类型，可以用来并发读取和修改任何类型的值。

`Value`类型的定义如下：

```go
// A Value provides an atomic load and store of a consistently typed value.
// The zero value for a Value returns nil from Load.
// Once Store has been called, a Value must not be copied.
//
// A Value must not be copied after first use.
type Value struct {
	v any
}
```

`Value`类型有4个方法：`CompareAndSwap`,  `Load`, `Store`, `Swap`，定义如下：

```go
func (v *Value) CompareAndSwap(old, new any) (swapped bool)
func (v *Value) Load() (val any)
func (v *Value) Store(val any)
func (v *Value) Swap(new any) (old any)
```

源码实现：https://cs.opensource.google/go/go/+/refs/tags/go1.18.1:src/sync/atomic/value.go

下面是一个具体的示例：对`map[string][string]`类型做并发读写，为了避免加锁，使用`value`类型来读取和修改`map[string][string]`。

```go
package main

import (
	"sync/atomic"
	"time"
)

func loadConfig() map[string]string {
	// 从数据库或者文件系统中读取配置信息，然后以map的形式存放在内存里
	return make(map[string]string)
}

func requests() chan int {
	// 将从外界中接收到的请求放入到channel里
	return make(chan int)
}

func main() {
	// config变量用来存放该服务的配置信息
	var config atomic.Value
	// 初始化时从别的地方加载配置文件，并存到config变量里
	config.Store(loadConfig())
	go func() {
		// 每10秒钟定时拉取最新的配置信息，并且更新到config变量里
		for {
			time.Sleep(10 * time.Second)
			// 对应于赋值操作 config = loadConfig()
			config.Store(loadConfig())
		}
	}()
	// 创建协程，每个工作协程都会根据它所读取到的最新的配置信息来处理请求
	for i := 0; i < 10; i++ {
		go func() {
			for r := range requests() {
				// 对应于取值操作 c := config
				// 由于Load()返回的是一个interface{}类型，所以我们要先强制转换一下
				c := config.Load().(map[string]string)
				// 这里是根据配置信息处理请求的逻辑...
				_, _ = r, c
			}
		}()
	}
}
```



## 总结和注意事项

* 原子操作由底层CPU的原子操作指令支持。

* 5种原子操作和`Value`类型的官方文档地址：https://pkg.go.dev/sync/atomic@go1.18.1

* CAS操作会有ABA问题

* 对于386处理器架构，64-bit原子操作函数使用了奔腾MMX或更新处理器型号才支持的CPU指令。对于非Linux的ARM处理器架构，64-bit原子操作函数使用了ARMv6k core或更新处理器型号才支持的CPU指令。对于ARM, 386和32-bit MIPS处理器架构，原子操作的调用者要对进行原子访问的64bit字(word)按照64-bit进行内存对齐。变量或者分配的结构体、数组和切片的第1个字可以认为是64-bit对齐的。(这块涉及到内存对齐，后面抽个专题详解)

	

## References

* https://pkg.go.dev/sync/atomic@go1.18.1
* https://blog.betacat.io/post/golang-atomic-value-exploration/
* https://gfw.go101.org/article/concurrent-atomic-operation.html