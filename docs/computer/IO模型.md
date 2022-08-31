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

# 用户态和内核态

通过**系统调用**将Linux整个体系分为内核态和用户态(或者说内核空间和用户空间)。

内核态从本质上说就是我们所说的内核，它是一种**特殊的软件程序**。控制计算机的硬件资源，例如协调CPU资源，分配内存资源，并且提供稳定的环境供应用程序运行。

**用户态就是提供应用程序运行的空间**，为了使应用程序访问到内核管理的资源例如CPU，内存，I/O。

内核必须提供一组通用的访问接口，这些接口就叫**系统调用**。

**从用户态到内核态切换可以通过三种方式**:

1. 系统调用：其实系统调用本身就是中断，但是软件中断，跟硬中断不同。
2. 异常：如果当前进程运行在用户态，如果这个时候发生了异常事件，就会触发切换。例如：缺页异常。
3. 外设中断：当外设完成用户的请求时，会向CPU发送中断信号。

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



# 基础IO模型

https://blog.csdn.net/qq_44377709/article/details/123724519

- 同步阻塞IO（Blocking IO，BIO）：即传统的IO模型。

- 同步非阻塞IO（Non-blocking IO，NIO）：默认创建的socket都是阻塞的，非阻塞IO要求socket被设置为NONBLOCK。注意这里所说的NIO并非Java的NIO（New IO）库。

- 多路复用IO（IO Multiplexing）：即经典的Reactor设计模式，有时也称为异步阻塞IO，Java中的Selector和Linux中的epoll都是这种模型。

- 异步IO（Asynchronous IO，AIO）：即经典的Proactor设计模式，也称为异步非阻塞IO。

### 阻塞和非阻塞

> 阻塞：需要内核 IO 操作彻底完成后才返回到用户空间执行用户程序的操作指令。
>
> 非阻塞：户进程不需要等待内核 IO 操作彻底完成，即可返回用户空间执行后续指令。与此同时，内核会立即返回给用户一个 IO 状态值。
>
> 而**内核缓冲区与用户缓冲区之间的读写操作肯定是阻塞的**

### 同步和异步

> 同步：调用者主动发起请求，调用者主动等待这个结果返回，**一但调用就必须有返回值**
>
> 异步：调用发出后直接返回，所以**没有返回结果**。被调用者处理完成后通知回调、通知等机制来通知调用者

**阻塞和非阻塞关注的是调用方进程状态，而同步与异步更关注调用双方进程通信方式。**

## 一、BIO同步阻塞

最传统的一种 IO 模型，即`在读写数据过程中会发生阻塞现象`。

- 用户进程调用read()系统函数，用户进程进入**阻塞**状态
- 系统内核收到read()系统调用，网卡开始准备接收数据，在一开始内核缓冲区数据为空，内核在等待接收数据，用户进程同步阻塞等待
- 内核缓冲区中有完整的数据后，内核会将内核缓冲区中的数据复制到用户缓冲区
- 直到用户缓冲区中有数据，用户进程才能解除阻塞状态继续执行

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207191643408.png" alt="640" style="zoom:67%;" />

## 二、NIO同步非阻塞

不断进行IO系统调用，轮询数据是否准备好。

- 用户进程发起请求调用read()函数，系统内核收到read()系统调用，网卡开始准备接收数据
- 内核缓冲区数据没有准备好，**请求立即返回error**，用户进程**不断的重试查询**内核缓冲区数据有没有准备好
- 当内核缓冲区**数据准备好了之后**，用户**进程阻塞**，内核开始将内核缓冲区数据复制到用户缓冲区
- 复制完成后，用户进程解除阻塞，读取数据继续执行

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207191643268.png" alt="640 (1)" style="zoom:67%;" />

所以事实上，在非阻塞 IO 模型中，**用户线程**需要**不断地询问内核数据是否就绪**，也就说非阻塞 IO 不会交出 CPU，而会一直占用 CPU。

缺点：空轮询

- 假如现在有1万个客户端连接，但只有1个客户端发送数据过来，为了获取这个1个客户端发送的消息，我需要循环向内核发送1万遍recv()系统调用，而这其中有9999次是无效的请求，浪费CPU资源

## 三、多路复用IO模型---异步阻塞

> Java NIO 实际上就是多路复用 IO。

IO多路复用模型是**建立在内核提供的多路分离函数select基础之上**的，使用select函数可以**避免同步非阻塞IO模型中轮询等待**的问题，即**一次性将N个客户端socket连接传入内核**（把事件都收集，传入内核），然后阻塞，**交由内核去轮询**，当某一个或多个socket连接有事件发生时，解除阻塞并返回事件列表，用户进程**再使用IO资源**，循环遍历处理有事件的socket连接。这样就避免了多次调用recv()系统调用，**避免了用户态到内核态的切换**。

> **Linux多路复用技术，就是多个进程的IO可以注册到同一个管道上，这个管道会统一和内核进行交互。当管道中的某一个请求需要的数据准备好之后，进程再把对应的数据拷贝到用户空间中。**

通过select调用而不是read调用，并且通过内核监控。

**非阻塞IO中是通过用户线程去不断询问，而多路复用IO是单独的内核线程去轮询。**

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207191644300.png" alt="640 (2)" style="zoom:67%;" />

实现方式有select、poll、epoll

## 四、信号驱动IO模型

在信号驱动 IO 模型中，当用户线程发起一个 IO 请求操作，会给对应的 socket 注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用 IO 读写操作来进行实际的 IO 请求操作。

## 五、AIO异步非阻塞

asynchronous

在异步 IO 模型中，**当用户线程发起 read 操作之后，立刻就可以开始去做其它的事**。

内核受到一个 asynchronous read 之后，会立刻返回，说明 read 请求已经成功发起了，因此不会对用户线程产生任何 block。然后，**内核等待数据准备完成，将数据拷贝到用户线程后**，内核会给用户线程**发送信号**，**告诉它 read 操作完成了**。**用户线程就可以使用数据了**。

也就说用户线程完全不需要知道实际的整个 IO 操作是如何进行的，**只需要先发起一个请求，当接收内核返回的成功信号时表示 IO 操作已经完成，可以直接去使用数据了**。

IO 操作的两个阶段都不会阻塞用户线程，这两个阶段都是由内核自动完成，然后发送一个信号告知用户线程操作已完成。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207191644075.png" alt="640 (3)" style="zoom:67%;" />

> **用户线程中不需要再次调用 IO 函数进行具体的读写**。这点是和信号驱动模型有所不同的。
>
> 在信号驱动模型中，当用户线程接收到信号表示数据已经就绪，然后需要用户线程调用 IO 函数进行实际的读写操作；而在异步 IO 模型中，收到信号表示 IO 操作已经完成，不需要再在用户线程中调用 IO 函数进行实际的读写操作，直接可以使用数据。
>
> 异步 IO 需要操作系统的底层支持，在 Java 7 中，提供了Asynchronous IO

<br>

# select、poll、epoll

select/poll/epoll 都是 I/O 多路复用的具体实现，select 出现的最早，之后是 poll，再是 epoll。

https://blog.csdn.net/shawntime/article/details/115089947

### 文件描述符

[文件描述符](https://blog.csdn.net/weixin_43202123/article/details/121008441)

Linux 系统中，把一切都看做是文件，当**进程打开现有文件**或创建新文件时，**内核向进程返回一个文件描述符**，文件描述符就是内核为了高效**管理已被打开的文件所创建的索引**，用来**指向被打开的文件**，所有执行I/O操作的系统调用都会通过文件描述符。用fd表示。

### select

```c
int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

select 允许应用程序监视一组文件描述符，等待一个或者多个描述符成为就绪状态，从而完成 I/O 操作。

> select函数仅仅知道有几个I/O事件发生了，但并不知道具体是哪几个socket连接有I/O事件，还需要轮询去查找，时间复杂度为O(n)，处理的请求数越多，所消耗的时间越长。

- fd_set 使用**数组**实现，数组**大小**使用 FD_SETSIZE（**默认1024**）定义，所以只能监听少于1024个描述符。有三种类型的描述符类型：readset、writeset、exceptset，分别对应读、写、异常条件的描述符集合。
- timeout 为超时参数，调用 select 会一直阻塞直到有描述符的事件到达或者等待的时间超过 timeout。
- 成功调用返回结果大于 0，出错返回结果为 -1，超时返回结果为 0。

缺点

- 需要维护一个用来存放大量fd的数据结构，
- 每次调用select时把fd集合**从用户态拷贝到内核态**，**复制开销大**。

> 1024是因为FD_SETSIZE配置成了1024，而且超过1024还可能会导致数组越界

### poll

```c
int poll(struct pollfd *fds, unsigned int nfds, int timeout);
```

poll 的功能与 select 类似，也是等待一组描述符中的一个成为就绪状态。

**采用链表存放，没有最大连接数的限制**   

poll 中的描述符是 pollfd 类型的数组，pollfd 的定义如下：

```c
struct pollfd {
    int   fd;         /* file descriptor */
    short events;     /* requested events */
    short revents;    /* returned events */
};
```

### epoll

```c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

- 调用epoll_create()创建一个ep对象，即红黑树的根节点，即声明一个事件收集器的内存空间
- 调用epoll_ctl()向这个ep对象（红黑树，事件收集器）中注册事件
- 调用epoll_wait()等待、监听是否有事件出现，将就绪的事件放入就绪列表，返回

epoll可以理解为event pool，不同与select、poll的轮询机制，epoll采用的是**事件驱动机制**，每个fd上有注册有**回调函数**，当网卡接收到数据时会回调该函数，同时将该fd的引用放入rdlist就绪列表中。**不需要轮询**得到就绪的事件描述符。

epoll相当于没有最大描述符的限制，并且使用红黑树来存。

> 内存拷贝，利用**mmap()文件映射内存**加速与内核空间的消息传递，减少复制开销。
>
> 每次select和poll的时候都需要拷贝fd，但是epoll在第一次epoll_ctl()的时候就拷贝并保存到了内核。


#### 工作模式

epoll 的描述符事件有两种触发模式：LT（level trigger）和 ET（edge trigger）。

##### 1、LT 模式

水平触发

当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking。

##### 2、ET 模式

边缘触发

和 LT 模式不同的是，通知之后进程必须**立即处理**事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知。

很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

### 三者比较

- epoll支持的最大文件描述符上限是整个系统最大可打开的文件数目，1G内存理论上最大创建10万个文件描述符；select有限制，poll没限制。

- select数组、poll链表、epoll红黑树

- **epoll每个文件描述符上都有一个callback函数**，当socket有事件发生时会回调这个函数将该fd的引用添加到就绪列表中，**select和poll并不会明确指出是哪些文件描述符就绪**，**需要轮询**，而 epoll 能直接得到。

	即系统调用返回后，调用select和poll的程序需要**轮询**整个文件描述符找到是谁处于就绪，而 epoll 采用**回调机制**则直接处理即可

	造成的结果就是，随着fd的增加，select 和 poll 的效率会线性降低，而 epoll 不会受到太大影响，除非所有 socket 都很活跃的情况下，可能会有性能问题

- fd拷贝

	select/poll 内核需要将消息传递到用户空间，都需要内核拷贝动作。

	epoll通过内核和用户空间共享一块内存来实现的。mmap

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207191853736.png" alt="image-20210322173910834" style="zoom:80%;" />

### 应用场景

很容易产生一种错觉认为只要用 epoll 就可以了，select 和 poll 都已经过时了，其实它们都有各自的使用场景。

#### 1. select 应用场景

select 的 timeout 参数精度为微秒，而 poll 和 epoll 为毫秒，因此 select 更加适用于**实时性**要求比较高的场景，比如核反应堆的控制。

select 可移植性更好，几乎被所有主流平台所支持。

#### 2. poll 应用场景

poll 没**有最大描述符数量的限制**，如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select。

#### 3. epoll 应用场景

只需要运行在 Linux 平台上，有大量的描述符需要同时轮询，并且这些连接最好是长连接。

需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且 epoll 的描述符存储在内核，不容易调试。

