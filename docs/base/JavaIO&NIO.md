# 传统IO操作流程

传统的IO模式，主要包括 read 和 write 过程：

- read：把数据从磁盘读取到内核缓冲区，再拷贝到用户缓冲区
- write：先把数据写入到 socket缓冲区，最后写入网卡设备

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207130940484.png" alt="img" style="zoom:80%;" />

1. 用户空间的应用程序通过read()函数，向操作系统发起IO调用，上下文从用户态到切换到内核态，然后再通过 DMA 控制器将数据从磁盘文件中读取到内核缓冲区
2. 接着CPU将内核空间缓冲区的数据拷贝到用户空间的数据缓冲区，然后read系统调用返回，而系统调用的返回又会导致上下文从内核态切换到用户态
3. 用户空间的应用程序通过write()函数向操作系统发起IO调用，上下文再次从用户态切换到内核态；接着CPU将数据从用户缓冲区复制到内核空间的 socket 缓冲区（也是内核缓冲区，只不过是给socket使用），然后write系统调用返回，再次触发上下文切换
4. 最后异步传输socket缓冲区的数据到网卡，也就是说write系统调用的返回并不保证数据被传输到网卡

在传统的数据 IO 模式中，读取一个磁盘文件，并发送到远程端的服务，就共有四次用户空间与内核空间的上下文切换，四次数据复制，包括两次 CPU 数据复制，两次 DMA 数据复制。

> DMA（Direct Memory Access，直接内存访问）：DMA 本质上是一块主板上独立的芯片，允许外设设备直接与内存存储器进行数据传输，并且不需要CPU参与的技术

# 基础IO模型

## 一、阻塞IO模型

最传统的一种 IO 模型，即`在读写数据过程中会发生阻塞现象`。

当用户线程发出 IO 请求之后，内核会去查看数据是否就绪，如果`没有就绪就会等待数据就绪`，而用户线程就会处于阻塞状态，用户线程交出 CPU。

当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除 block 状态。典型的阻塞 IO 模型的例子为：

```java
data = socket.read();
```

如果数据没有就绪，就会一直阻塞在 read 方法。

## 二、非阻塞IO模型

当用户线程发起一个 read 操作后，并不需要等待，而是马上就得到了一个结果。如果结果是一个 error 时，它就知道数据还没有准备好，于是它可以再次发送 read 操作。一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。

所以事实上，在非阻塞 IO 模型中，**用户线程**需要`不断地询问内核数据是否就绪`，也就说非阻塞 IO 不会交出 CPU，而会一直占用 CPU。典型的非阻塞 IO 模型一般如下：

```java
while(true){
    data = socket.read();
    if(data!= error){
        // 处理数据
        break;
    }
}
```

但是对于非阻塞 IO 就有一个非常严重的问题，在 while 循环中需要不断地去询问内核数据是否就绪，这样会**导致 CPU 占用率非常高**，因此一般情况下很少使用 while 循环这种方式来读取数据

## 三、多路复用IO模型

多路复用 IO 模型是目前使用得比较多的模型。

Java NIO 实际上就是多路复用 IO。

在多路复用 IO 模型中，`会有一个线程不断去轮询多个 socket 的状态，只有当 socket 真正有读写事件时，才真正调用实际的 IO 读写操作`。**单独一个线程**轮询

非阻塞IO中是通过用户线程去不断询问，而多路复用IO是单独的线程去轮询

因为在多路复用 IO 模型中，只需要使用一个线程就可以管理多个socket，系统不需要建立新的进程或者线程，也不必维护这些线程和进程，并且只有在真正有socket 读写事件进行时，才会使用 IO 资源，所以它大大减少了资源占用。

在 Java NIO 中，是通过 selector.select()去查询每个通道是否有到达事件，如果没有事件，则一直阻塞在那里，因此这种方式会导致用户线程的阻塞。

多路复用 IO 模式，通过一个线程就可以管理多个 socket，只有当socket 真正有读写事件发生才会占用资源来进行实际的读写操作。因此，多路复用 IO 比较适合连接数比较多的情况。

不过要注意的是，多路复用 IO 模型是通过轮询的方式来检测是否有事件到达，并且对到达的事件逐一进行响应。因此对于多路复用 IO 模型来说，`一旦事件响应体很大，那么就会导致后续的事件迟迟得不到处理，并且会影响新的事件轮询`。

## 四、信号驱动IO模型

在信号驱动 IO 模型中，当用户线程发起一个 IO 请求操作，会给对应的 socket 注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用 IO 读写操作来进行实际的 IO 请求操作。

## 五、异步IO模型 - asynchronous

异步 IO 模型才是最理想的 IO 模型，在异步 IO 模型中，**当用户线程发起 read 操作之后，立刻就可以开始去做其它的事**。

内核受到一个 asynchronous read 之后，会立刻返回，说明 read 请求已经成功发起了，因此不会对用户线程产生任何 block。然后，**内核等待数据准备完成，将数据拷贝到用户线程后**，内核会给用户线程**发送信号**，**告诉它 read 操作完成了**。**用户线程就可以使用数据了**。

也就说用户线程完全不需要知道实际的整个 IO 操作是如何进行的，`只需要先发起一个请求，当接收内核返回的成功信号时表示 IO 操作已经完成，可以直接去使用数据了`。

IO 操作的两个阶段都不会阻塞用户线程，这两个阶段都是由内核自动完成，然后发送一个信号告知用户线程操作已完成。

> 用户线程中不需要再次调用 IO 函数进行具体的读写。这点是和信号驱动模型有所不同的。
>
> 在信号驱动模型中，当用户线程接收到信号表示数据已经就绪，然后需要用户线程调用 IO 函数进行实际的读写操作；而在异步 IO 模型中，收到信号表示 IO 操作已经完成，不需要再在用户线程中调用 IO 函数进行实际的读写操作，直接可以使用数据。
>
> 异步 IO 需要操作系统的底层支持，在 Java 7 中，提供了Asynchronous IO

<br>

# I/O复用

select/poll/epoll 都是 I/O 多路复用的具体实现，select 出现的最早，之后是 poll，再是 epoll。

### 文件描述符

[文件描述符](https://blog.csdn.net/weixin_43202123/article/details/121008441)

Linux 系统中，把一切都看做是文件，当**进程打开现有文件**或创建新文件时，**内核向进程返回一个文件描述符**，文件描述符就是内核为了高效**管理已被打开的文件所创建的索引**，用来**指向被打开的文件**，所有执行I/O操作的系统调用都会通过文件描述符。

### select

```c
int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

select 允许应用程序监视一组文件描述符，等待一个或者多个描述符成为就绪状态，从而完成 I/O 操作。

- fd_set 使用数组实现，数组大小使用 FD_SETSIZE 定义，所以只能监听少于 FD_SETSIZE 数量的描述符。有三种类型的描述符类型：readset、writeset、exceptset，分别对应读、写、异常条件的描述符集合。

- timeout 为超时参数，调用 select 会一直阻塞直到有描述符的事件到达或者等待的时间超过 timeout。

- 成功调用返回结果大于 0，出错返回结果为 -1，超时返回结果为 0。

```c
fd_set fd_in, fd_out;
struct timeval tv;

// Reset the sets
FD_ZERO( &fd_in );
FD_ZERO( &fd_out );

// Monitor sock1 for input events
FD_SET( sock1, &fd_in );

// Monitor sock2 for output events
FD_SET( sock2, &fd_out );

// Find out which socket has the largest numeric value as select requires it
int largest_sock = sock1 > sock2 ? sock1 : sock2;

// Wait up to 10 seconds
tv.tv_sec = 10;
tv.tv_usec = 0;

// Call the select
int ret = select( largest_sock + 1, &fd_in, &fd_out, NULL, &tv );

// Check if select actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    if ( FD_ISSET( sock1, &fd_in ) )
        // input event on sock1

    if ( FD_ISSET( sock2, &fd_out ) )
        // output event on sock2
}
```

### poll

```c
int poll(struct pollfd *fds, unsigned int nfds, int timeout);
```

poll 的功能与 select 类似，也是等待一组描述符中的一个成为就绪状态。

poll 中的描述符是 pollfd 类型的数组，pollfd 的定义如下：

```c
struct pollfd {
               int   fd;         /* file descriptor */
               short events;     /* requested events */
               short revents;    /* returned events */
           };
```


```c
// The structure for two events
struct pollfd fds[2];

// Monitor sock1 for input
fds[0].fd = sock1;
fds[0].events = POLLIN;

// Monitor sock2 for output
fds[1].fd = sock2;
fds[1].events = POLLOUT;

// Wait 10 seconds
int ret = poll( &fds, 2, 10000 );
// Check if poll actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    // If we detect the event, zero it out so we can reuse the structure
    if ( fds[0].revents & POLLIN )
        fds[0].revents = 0;
        // input event on sock1

    if ( fds[1].revents & POLLOUT )
        fds[1].revents = 0;
        // output event on sock2
}
```

### 比较

#### 1. 功能

select 和 poll 的功能基本相同，不过在一些实现细节上有所不同。

- select 会修改描述符，而 poll 不会；
- select 的描述符类型使用数组实现，FD_SETSIZE 大小默认为 1024，因此默认只能监听少于 1024 个描述符。如果要监听更多描述符的话，需要修改 FD_SETSIZE 之后重新编译；而 poll 没有描述符数量的限制；
- poll 提供了更多的事件类型，并且对描述符的重复利用上比 select 高。
- 如果一个线程对某个描述符调用了 select 或者 poll，另一个线程关闭了该描述符，会导致调用结果不确定。

#### 2. 速度

select 和 poll 速度都比较慢，每次调用都需要将全部描述符从应用进程缓冲区复制到内核缓冲区。

#### 3. 可移植性

几乎所有的系统都支持 select，但是只有比较新的系统支持 poll。

### epoll

```c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

epoll_ctl() 用于向内核注册新的描述符或者是改变某个文件描述符的状态。已注册的描述符在内核中会被维护在一棵红黑树上，通过回调函数内核会将 I/O 准备好的描述符加入到一个链表中管理，进程调用 epoll_wait() 便可以得到事件完成的描述符。

从上面的描述可以看出，epoll 只需要将描述符从进程缓冲区向内核缓冲区拷贝一次，并且进程不需要通过轮询来获得事件完成的描述符。

epoll 仅适用于 Linux OS。

epoll 比 select 和 poll 更加灵活而且没有描述符数量限制。

epoll 对多线程编程更有友好，一个线程调用了 epoll_wait() 另一个线程关闭了同一个描述符也不会产生像 select 和 poll 的不确定情况。

```c
// Create the epoll descriptor. Only one is needed per app, and is used to monitor all sockets.
// The function argument is ignored (it was not before, but now it is), so put your favorite number here
int pollingfd = epoll_create( 0xCAFE );

if ( pollingfd < 0 )
 // report error

// Initialize the epoll structure in case more members are added in future
struct epoll_event ev = { 0 };

// Associate the connection class instance with the event. You can associate anything
// you want, epoll does not use this information. We store a connection class pointer, pConnection1
ev.data.ptr = pConnection1;

// Monitor for input, and do not automatically rearm the descriptor after the event
ev.events = EPOLLIN | EPOLLONESHOT;
// Add the descriptor into the monitoring list. We can do it even if another thread is
// waiting in epoll_wait - the descriptor will be properly added
if ( epoll_ctl( epollfd, EPOLL_CTL_ADD, pConnection1->getSocket(), &ev ) != 0 )
    // report error

// Wait for up to 20 events (assuming we have added maybe 200 sockets before that it may happen)
struct epoll_event pevents[ 20 ];

// Wait for 10 seconds, and retrieve less than 20 epoll_event and store them into epoll_event array
int ready = epoll_wait( pollingfd, pevents, 20, 10000 );
// Check if epoll actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    // Check if any events detected
    for ( int i = 0; i < ready; i++ )
    {
        if ( pevents[i].events & EPOLLIN )
        {
            // Get back our connection pointer
            Connection * c = (Connection*) pevents[i].data.ptr;
            c->handleReadEvent();
         }
    }
}
```


#### 工作模式

epoll 的描述符事件有两种触发模式：LT（level trigger）和 ET（edge trigger）。

##### 1. LT 模式

当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking。

##### 2. ET 模式

和 LT 模式不同的是，通知之后进程必须立即处理事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知。

很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

### 应用场景

很容易产生一种错觉认为只要用 epoll 就可以了，select 和 poll 都已经过时了，其实它们都有各自的使用场景。

#### 1. select 应用场景

select 的 timeout 参数精度为微秒，而 poll 和 epoll 为毫秒，因此 select 更加适用于实时性要求比较高的场景，比如核反应堆的控制。

select 可移植性更好，几乎被所有主流平台所支持。

#### 2. poll 应用场景

poll 没有最大描述符数量的限制，如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select。

#### 3. epoll 应用场景

只需要运行在 Linux 平台上，有大量的描述符需要同时轮询，并且这些连接最好是长连接。

需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且 epoll 的描述符存储在内核，不容易调试。

### 三者对比

1. select，使用的是**数组**来存储Socket连接文件描述符，容量是固定的，需要通过轮询来判断是否发生了IO事件
2. poll，使用的是**链表**来存储Socket连接文件描述符，容量是不固定的，同样需要通过轮询来判断是否发生了IO事件
3. epoll，epol和poll是完全不同的，epoll是一种**事件通知模型**，当发生了IO事件时，应用程序才进行IO操作，不需要像poll模型那样主动去轮询

# Java IO

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061117929.png" alt="image-20220406111706468" style="zoom:50%;" />

# Java NIO

NIO 主要有三大核心部分：Channel(通道)，Buffer(缓冲区)，Selector(选择区)。

传统 IO 基于字节流和字符流进行操作，而 NIO 基于 Channel 和 Buffer进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061139740.png" alt="image-20220406113858769" style="zoom:67%;" />

> IO是面向流，NIO是面向缓冲区的

**NIO的缓冲区**

JavaIO面向流意味着每次从流中读一个多多个字节，直至读取所有字节；不能前后移动流中中的数据；要移动，必须先缓冲到一个缓冲区。

NIO会将数据读取到一个缓冲区，需要时可以进行前后移动，增加了处理的灵活性；

- 但是，需要检查该缓冲区是否包含所有需要处理的数据；
- 而且，需确保当更多的数据读入缓冲区时，不要覆盖尚未处理的数据

**NIO的非阻塞**

IO 的各种流是阻塞的。这意味着，当一个线程调用 read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。

NIO 的非阻塞模式

- 非阻塞读：使一个线程从某通道发送请求读取数据，但是它**仅能得到目前可用的数据**，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以**直至数据变的可以读取之前，该线程可以继续做其他的事情**。

- 非阻塞写：一个线程请求写入一些数据到某通道，但**不需要等待它完全写入**，这个线程同时可以去做别的事情。

线程通常将非阻塞 IO 的空闲时间用于在其它通道上执行 IO 操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）

## 1、Channel通道

Channel 和 IO 中的 Stream(流)是差不多一个等级的。只不过` Stream 是单向的`，譬如：InputStream, OutputStream，而` Channel 是双向的`，既可以用来进行读操作，又可以用来进行写操作。

主要实现及其对应

- FileChannel：文件IO
- DatagramChannel：UDP
- SocketChannel：TCP（Client）
- ServerSocketChannel：TCP（Server）

## 2、Buffer缓冲区

缓冲区，实际上是一个容器，是一个连续数组。Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061133636.png" alt="image-20220406113328926" style="zoom: 50%;" />

从一个客户端向服务端发送数据，然后服务端接收数据的过程。客户端发送数据时，必须先将数据存入 Buffer 中，然后将 Buffer 中的内容写入通道。服务端这边接收数据必须通过 Channel 将数据读入到 Buffer 中，然后再从 Buffer 中取出数据来处理。 

NIO 中，Buffer 是一个顶层父类，它是一个抽象类，常用的 Buffer 的子类有：ByteBuffer、IntBuffer、 CharBuffer、 LongBuffer、 DoubleBuffer、FloatBuffer、ShortBuffer

## 3、Selector选择区

Selector 类是 NIO 的核心类

`Selector 能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理`。

这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。



# 零拷贝

[零拷贝](https://blog.csdn.net/a745233700/article/details/122660332)

- 避免频繁的上下文切换与系统调用。
- 需要拷贝的数据可以先被缓存起来，减少没必要的 CPU 数据拷贝操作。
- 对数据进行处理尽量让硬件来做。

零拷贝指在进行数据 IO 时，数据在用户态下经历了零次 CPU 拷贝，并非不拷贝数据。通过减少数据传输过程中**内核缓冲区和用户进程缓冲区间不必要的CPU数据拷贝**与**用户态和内核态的上下文切换次数**，降低 CPU 在这两方面的开销，释放 CPU 执行其他任务，更有效的利用系统资源，提高传输效率，同时还减少了内存的占用，也提升应用程序的性能。

由于零拷贝在**内核空间中完成所有的内存拷贝**，可以**最大化使用 socket 缓冲区的可用空间**，从而提高了一次系统调用中处理的数据量，进一步降低了上下文切换次数。零拷贝技术基于 **PageCache**，而 PageCache 缓存了最近访问过的数据，提升了访问缓存数据的性能，同时，为了解决机械磁盘寻址慢的问题，它还协助 IO 调度算法实现了 IO 合并与预读（这也是顺序读比随机读性能好的原因），这进一步提升了零拷贝的性能。



## 方式

### 1、内存映射mmap + write

```c
#include <sys/mman.h>
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset)
```

- addr：指定映射的虚拟内存地址
- length：映射的长度
- prot：映射内存的保护模式
- flags：指定映射的类型
- fd:进行映射的文件句柄
- offset:文件偏移量

在传统 IO 模式的4次内存拷贝中，与物理设备相关的2次拷贝（把磁盘数据拷贝到内存 以及 把数据从内存拷贝到网卡）是必不可少的。**但与用户缓冲区相关的2次拷贝都不是必需的**，如果内核在读取文件后，直接把内核缓冲区中的内容拷贝到 Socket 缓冲区，待到网卡发送完毕后，再通知进程，这样就可以减少一次 CPU 数据拷贝了。

**内存映射mmap**：核心就是操作系统**把内核缓冲区与应用程序共享**，将一段用户空间内存映射到内核空间，当映射成功后，用户对这段内存区域的修改可以直接反映到内核空间；同样地，内核空间对这段区域的修改也直接反映用户空间。正因为有这样的映射关系, 就不需要在用户态与内核态之间拷贝数据。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207131026655.png" alt="img" style="zoom:80%;" />

1. 用户应用程序通过 mmap() 向操作系统发起 IO调用，上下文从用户态切换到内核态；然后通过 DMA 将数据从磁盘中复制到内核空间缓冲区
2. mmap 系统调用返回，上下文从内核态切换回用户态（这里不需要将数据从内核空间复制到用户空间，因为用户空间和内核空间共享了这个缓冲区）
3. 用户应用程序通过 write() 向操作系统发起 IO调用，上下文再次从用户态切换到内核态。接着 CPU 将数据从内核空间缓冲区复制到内核空间 socket 缓冲区；write 系统调用返回，导致内核空间到用户空间的上下文切换
4. DMA 异步将 socket 缓冲区中的数据拷贝到网卡

**mmap 的零拷贝 I/O 进行了4次用户空间与内核空间的上下文切换，以及3次数据拷贝；其中3次数据拷贝中包括了2次 DMA 拷贝和1次 CPU 拷贝。**所以 mmap 通过内存地址映射的方式，节省了数据IO过程中的一次CPU数据拷贝以及一半的内存空间

### 2、sendfile

```c
#include <sys/sendfile.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

- out_fd:为待写入内容的文件描述符，一个socket描述符。，
- in_fd:为待读出内容的文件描述符，必须是真实的文件，不能是socket和管道。
- offset：指定从读入文件的哪个位置开始读，如果为NULL，表示文件的默认起始位置。
- count：指定在fdout和fdin之间传输的字节数。

只要我们的代码执行 read 或者 write 这样的系统调用，一定会发生 2 次上下文切换：首先从用户态切换到内核态，当内核执行完任务后，再切换回用户态交由进程代码执行。因此，如果想减少上下文切换次数，就一定要减少系统调用的次数，解决方案就是**把 read、write 两次系统调用合并成一次**，在内核中完成磁盘与网卡的数据交换。在 Linux 2.1 版本内核开始引入的 sendfile 就是通过这种方式来实现零拷贝的：

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207131032822.png" alt="img" style="zoom:80%;" />

1. 用户应用程序发出 sendfile 系统调用，上下文从用户态切换到内核态；然后通过 DMA 控制器将数据从磁盘中复制到内核缓冲区中
2. 然后CPU将数据从内核空间缓冲区复制到 socket 缓冲区
3. sendfile 系统调用返回，上下文从内核态切换到用户态
4. DMA 异步将内核空间 socket 缓冲区中的数据传递到网卡

**通过 sendfile 实现的零拷贝I/O使用了2次用户空间与内核空间的上下文切换，以及3次数据的拷贝。其中3次数据拷贝中包括了2次DMA拷贝和1次CPU拷贝。**那能不能将CPU拷贝的次数减少到0次呢？答案肯定是有的，那就是 带 DMA 收集拷贝功能的 sendfile。

### 3、带 DMA 收集拷贝功能的 sendfile 实现的零拷贝

Linux 2.4 版本之后，对 sendfile 做了升级优化，引入了 SG-DMA技术，其实就是对DMA拷贝加入了 scatter/gather 操作，它可以直接从内核空间缓冲区中将数据读取到网卡，无需将内核空间缓冲区的数据再复制一份到 socket 缓冲区，从而省去了一次 CPU拷贝。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207131052131.png" alt="img" style="zoom:80%;" />

1. 用户应用程序发出 sendfile 系统调用，上下文从用户态切换到内核态；然后通过 DMA 控制器将数据从磁盘中复制到内核缓冲区中
2. 接下来不需要CPU将数据复制到 socket 缓冲区，而是将相应的文件描述符信息复制到 socket 缓冲区，该描述符包含了两种的信息：①内核缓冲区的内存地址、②内核缓冲区的偏移量
3. sendfile 系统调用返回，上下文从内核态切换到用户态
4. DMA 根据 socket 缓冲区中描述符提供的地址和偏移量直接将内核缓冲区中的数据复制到网卡

**带有 DMA 收集拷贝功能的 sendfile 实现的 I/O 使用了2次用户空间与内核空间的上下文切换，以及2次数据的拷贝，而且这2次的数据拷贝都是非CPU拷贝，这样就实现了最理想的零拷贝I/O传输了，不需要任何一次的CPU拷贝，以及最少的上下文切换**

> Note：需要注意的是，零拷贝有一个缺点，就是不允许进程对文件内容作一些加工再发送，比如数据压缩后再发送。

## 使用场景

### 1、Kafka、RocketMQ

数据文件使用 sendfile 方式

索引文件使用 mmap + write 方式

### 2、Java的NIO

#### 2.1、mmap + write

FileChannel 的 map() 方法产生的 MappedByteBuffer：FileChannel 提供了 map() 方法，该方法可以在一个打开的文件和 MappedByteBuffer 之间建立一个虚拟内存映射，MappedByteBuffer 继承于 ByteBuffer；该缓冲器的内存是一个文件的内存映射区域。map() 方法底层是通过 mmap 实现的，因此将文件内存从磁盘读取到内核缓冲区后，用户空间和内核空间共享该缓冲区。mmap的小demo如下：

```java
public class MmapTest {
 
    public static void main(String[] args) {
        try {
            FileChannel readChannel = FileChannel.open(Paths.get("./jay.txt"), StandardOpenOption.READ);
            MappedByteBuffer data = readChannel.map(FileChannel.MapMode.READ_ONLY, 0, 1024 * 1024 * 40);
            FileChannel writeChannel = FileChannel.open(Paths.get("./siting.txt"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);
            //数据传输
            writeChannel.write(data);
            readChannel.close();
            writeChannel.close();
        }catch (Exception e){
            System.out.println(e.getMessage());
        }
    }
}
```



#### 2.2、sendfile

FileChannel 的 transferTo、transferFrom 如果操作系统底层支持的话，transferTo、transferFrom也会使用 sendfile 零拷贝技术来实现数据的传输

```java
@Override
public long transferFrom(FileChannel fileChannel, long position, long count) throws IOException {
   return fileChannel.transferTo(position, count, socketChannel);
}
```

```java
public class SendFileTest {
    public static void main(String[] args) {
        try {
            FileChannel readChannel = FileChannel.open(Paths.get("./jay.txt"), StandardOpenOption.READ);
            long len = readChannel.size();
            long position = readChannel.position();
            
            FileChannel writeChannel = FileChannel.open(Paths.get("./siting.txt"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);
            //数据传输
            readChannel.transferTo(position, len, writeChannel);
            readChannel.close();
            writeChannel.close();
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

