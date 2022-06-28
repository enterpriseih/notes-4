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



# Kaptcha验证码

1、kaptcha如何使用:

   - 添加jar
   - 在web.xml文件中注册KaptchaServlet，并设置验证码图片的相关属性
   - 在html页面上编写一个img标签，然后设置src等于KaptchaServlet对应的url-pattern

2、kaptcha验证码图片的各个属性在常量接口：Constants中

3、KaptchaServlet在生成验证码图片时，会同时将验证码信息保存到session中

因此，在注册请求时，首先将用户文本框中输入的验证码值和session中保存的值进行比较，相等，则进行注册



# 单点登录SSO(Single Sign On)

三种常用方式

1、session广播机制实现

2、cookie+redis实现

3、token实现

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206281706291.png" alt="02 单点登录三种方式介绍" style="zoom:75%;" />

# JWT

## 简介

JWT 就是一种生成 token 的规则。

JWT（Json Web Token）是为了在网络应用环境间传递声明而执行的一种基于 JSON 的开放标准。

JWT 最重要的作用就是对 token信息的**防伪**作用。 

## JWT 的原理

一个 JWT 由三个部分组成：**公共部分**、**私有部分**、**签名部分**。

最后由这三者组合进行 base64 编码得到 JWT。

### 1、 公共部分

主要是该 JWT 的相关**配置参数**，比如签名的**加密算法**、**格式类型**、**过期时间**等等。

Key=YIENX

### 2、 私有部分

用户自定义的内容，根据实际需要真正要封装的信息。

userInfo{用户的Id，用户的昵称nickName}

### 3、 签名部分

SaltiP: 当前服务器的Ip地址!{linux 中配置代理服务器的ip}

主要用户对JWT生成字符串的时候，进行加密{盐值}

最终组成 key+salt+userInfo è token!

base64编码，并不是加密，只是把明文信息变成了不可见的字符串。但是其实只要用一些工具就可以把base64编码解成明文，所以不要在JWT中放入涉及私密的信息。

## 优缺点

- JWT不仅可用于认证，还可用于信息交换。善用JWT有助于减少服务器请求数据库的次数。

- 生产的token可以包含基本信息，比如id、用户昵称、头像等信息，避免再次查库
- 存储在客户端，不占用服务端的内存资源
- JWT默认不加密，但可以加密。生成原始令牌后，可以再次对其进行加密。
- 当JWT未加密时，一些私密数据无法通过JWT传输。
- JWT的最大缺点是服务器不保存会话状态，所以在使用期间不可能取消令牌或更改令牌的权限。也就 是说，一旦JWT签发，在有效期内将会一直有效。
- JWT本身包含认证信息，token是经过base64编码，所以可以解码，因此token加密前的对象不应该 包含敏感信息，一旦信息泄露，任何人都可以获得令牌的所有权限。为了减少盗用，JWT的有效期不 宜设置太长。对于某些重要操作，用户在使用时应该每次都进行进行身份验证。
- 为了减少盗用和窃取，JWT不建议使用HTTP协议来传输代码，而是使用加密的HTTPS协议进行传 输。
