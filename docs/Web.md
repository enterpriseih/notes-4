# Java 的三层架构

**业务层（逻辑层、service层）**

采用事务脚本模式。将一个业务中所有的操作封装成一个方法，同时保证方法中所有的数据库更新操作，即保证同时成功或同时失败。避免部分成功部分失败引起的数据混乱操作。

**表述层**

采用MVC模式。  

M称为模型，也就是实体类。用于数据的封装和数据的传输。 

V为视图，也就是GUI组件，用于数据的展示。  

C为控制，也就是事件，用于流程的控制。

**持久层（DAO）**

采用DAO模式，建立实体类和数据库表映射（ORM映射，Object Relationship Mapping）。也就是哪个类对应哪个表，哪个属性对应哪个列。

持久层的目的就是，完成对象数据和关系数据的转换。

>TIP
>
>**SSM框架**
>
>- 业务层——Spring
>- 表现层——SpringMVC
>- 持久层——MyBatis
>
>**SSH框架**
>
>- 业务层——Spring
>- 表现层——Struts
>- 持久层——Hibernate





# Session、Cookie和Token

### Session

- 在**服务器端保存**（客户端只有Session ID）的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中
- 可以保存在：内存、Cookie中、redis或memcached等缓存中、数据库中

**钝化**：服务器关闭，但是浏览器没关闭，会话仍继续，需要将session中的内容序列化保存在磁盘上

**活化**：服务器又重启了，将磁盘中的内容反序列化至session

### Cookie

- **客户端**（浏览器）保存用户信息的一种机制，用来记录用户的一些信息，通常在Cookie中**记录Session ID**。

### Token

- token是用户身份的验证方式，我们通常叫它：令牌。
- 最简单的token组成:uid(用户唯一的身份标识)、time(当前时间的时间戳)、sign(签名，由token的前几位+以哈希算法压缩成一定长的十六进制字符串，可以防止恶意第三方拼接token请求服务器)。还可以把不变的参数也放进token，避免多次查库。
- 是否授权给软件
- 服务器端会有校验机制，校验token是否合法



### Cookie和Session的区别

1）cookie数据存放在客户的浏览器上，session数据放在服务器上；

2）cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗,考虑到安全应当使用session；

3）session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能,考虑到减轻服务器性能方面，应当使用cookie；

4）单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

> 将登陆信息等重要信息存放为session；
>
> 其他信息如果需要保留，可以放在cookie中。
>
> 第一次请求的响应中会生成一个对应session的cookie：JSESSIONID，下一次请求的响应不会生成，但是请求会将JSESSIONID传过去，寻找相应的session



# JavaWeb的三大规范

Servlet程序、Listener监听器、Filter过滤器

初始化顺序：listener -> filter -> servlet

## 1、Servlet



## 2、Listener



## 3、Filter





# Servlet的三大域对象

| 对象名称    | 对应类型           |
| ----------- | ------------------ |
| request     | HttpServletRequest |
| session     | HttpSession        |
| Application | ServletContext     |

## 1、request

**生命周期**

- 创建：客户端向服务器发送一次请求,服务器就会创建request对象
- 销毁：服务器对这次请求作出响应后就会销毁request对象
- 有效：仅在当前请求中有效

**作用**

1. 获取表单提交参数： request.getParameter()

2. 传值到表单： request.setAttribute()

## 2、session

**生命周期**

- 创建：服务器端第一次调用getSession();(保存在服务器内存中)

- 销毁：

	1. 非正常关闭服务器(正常关闭session会序列化，再次启动服务器session会被反序列化)；
	2. session过期了默认30分钟.
	3. 手动调用session.invalidate();

	- 注意：关闭浏览器再次访问会找不到session的会话id而不是session被销毁了。

- 有效：用户打开浏览器会话开始，直到关闭浏览器会话才会结束。一次会话期间只会创建一个session对象。

**作用**

1. 读取生成的验证码信息

```java
// 图片的验证码
String imageMsg = (String) request.getSession().getAttribute(“imageMsg”);
```

2. 用户保持登录状态

```java
//登录成功 保存用户登录状态
request.getSession().setAttribute(“user”, user)；
```

3. 购物车物品保存

```java
//将cart放入session中
request.getSession().setAttribute(“cart”, cart);
```



## 3、application

生命周期

- 创建：服务器启动的时候,服务器为每个WEB应用创建一个属于该web项目的对象ServletContext类.
- 销毁：服务器关闭或者项目从服务器中移除的时候.
- 有效：此信息在整个服务器上被保留。



## 4、区别

**request**：

每一次请求都是一个新的request对象，如果在web组件之间需要共享同一个请求中的数据，只能使用请求转发.

**session**：

每一次会话都是一个新的session对象，如果如果需要在一次会话中的多个请求之间需要共享数据，只能使用session.

**application**：

 应用对象，Tomcat启动到关闭，表示一个应用，在一个应用中有且只有一个application对象，作用于整个Web应用，可以实现多次会话之间的数据共享.

## 5、补充：page

对应pageContext，仅jsp有效

生命周期

- 当对JSP的请求开始，当相应结束时销毁。
- jsp页面被执行，声明周期开始；
- jsp页面执行完毕，声明周期结束；

作用范围：整个JSP页面，是四大作用域中最小的一个。



# 服务器内部转发和客户端重定向

## 1、服务器内部转发 

```java
request.getRequestDispatcher("...").forward(request,response);
```

- 一次请求响应的过程，对于客户端而言，内部经过了多少次转发，客户端是不知道的

- 由...响应

- 地址栏没有变化
- 同一个request和response

## 2、客户端重定向

```java
 response.sendRedirect("....");
```

- 两次请求响应的过程。客户端肯定知道请求URL有变化

- 地址栏有变化
- 不同的request和response

