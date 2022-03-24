## 1、Session、Cookie和Token

**Session**

- 在**服务器端保存**（客户端只有Session ID）的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中
- 可以保存在：内存、Cookie中、redis或memcached等缓存中、数据库中

**Cookie**

- **客户端**（浏览器）保存用户信息的一种机制，用来记录用户的一些信息，通常在Cookie中**记录Session ID**。

**Token**

- token是用户身份的验证方式，我们通常叫它：令牌。
- 最简单的token组成:uid(用户唯一的身份标识)、time(当前时间的时间戳)、sign(签名，由token的前几位+以哈希算法压缩成一定长的十六进制字符串，可以防止恶意第三方拼接token请求服务器)。还可以把不变的参数也放进token，避免多次查库。
- 是否授权给软件
- 服务器端会有校验机制，校验token是否合法



**Cookie和Session的区别**

1）cookie数据存放在客户的浏览器上，session数据放在服务器上；

2）cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗,考虑到安全应当使用session；

3）session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能,考虑到减轻服务器性能方面，应当使用cookie；

4）单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

> 将登陆信息等重要信息存放为session；
>
> 其他信息如果需要保留，可以放在cookie中。



