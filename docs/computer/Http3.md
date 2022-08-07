[参考1](https://blog.csdn.net/weixin_45583158/article/details/106132325?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165839094116782350812518%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165839094116782350812518&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-4-106132325-null-null.142^v33^new_blog_pos_by_title,185^v2^control&utm_term=http3&spm=1018.2226.3001.4187)

[参考2](https://zhuanlan.zhihu.com/p/431672713)

> 为了实现可靠性，QUIC 也有个序列号，是递增的。任何一个序列号的包只发送一次，下次就要加一了。即使丢包重传，序列号也会加一，并且QUIC 定义了一个 offset 概念。QUIC 既然是面向连接的，也就像 TCP 一样，是一个数据流，发送的数据在这个数据流里面有个偏移量 offset，可以通过 offset 查看数据发送到了哪里，这样只要这个 offset 的包没有来，就要重发；如果来了，按照 offset 拼接，还是能够拼成一个流。

> 为了防止网络上的中间设备识别协议的细节，QUIC 全面采用加密通信，而且QUIC 直接应用了 TLS1.3，顺便也就获得了 0-RTT、1-RTT 连接的好处。但 QUIC 并不是建立在 TLS 之上，而是内部“包含”了 TLS。它使用自己的帧“接管”了TLS 里的“记录”，握手消息、警报消息都不使用 TLS 记录，直接封装成 QUIC 的帧发送，省掉了一次开销。

> TCP 的流量控制是通过滑动窗口协议。而QUIC 的流量控制也是通过 window_update，来告诉对端它可以接受的字节数。但是 QUIC 的窗口是适应自己的多路复用机制的，不但在一个连接上控制窗口，还在一个连接中的每个 stream 控制窗口。

> HTTP3 没有指定默认的端口号，也就是说不一定非要在 UDP 的 80 或者 443 上提供 HTTP/3 服务。
> HTTP3 通过 HTTP2 里的“扩展帧”了。浏览器需要先用 HTTP2 协议连接服务器，然后服务器可以在启动 HTTP2 连接后发送一个“Alt-Svc”帧，包含一个“h3=host:port”的字符串，告诉浏览器在另一个端点上提供等价的 HTTP/3 服务。浏览器收到“Alt-Svc”帧，会使用 QUIC 异步连接指定的端口，如果连接成功，就会断开HTTP2 连接，改用新的HTTP3 收发数据。