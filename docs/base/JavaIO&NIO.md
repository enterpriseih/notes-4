# Java IO

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204061117929.png" alt="image-20220406111706468" style="zoom:50%;" />



# Java NIO（New IO）

其实就是IO多路复用

NIO 主要有三大核心部分：Channel(通道)，Buffer(缓冲区)，Selector(选择区)。

传统 IO 基于字节流和字符流进行操作，而 NIO 基于 Channel 和 Buffer进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector用于监听多个通道的事件（比如：连接打开，数据到达）。因此，**单个线程可以监听多个数据通道**。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207191651377.png" alt="640 (4)" style="zoom:67%;" />

> IO是面向流，NIO是面向缓冲区

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



