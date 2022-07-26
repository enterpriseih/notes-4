# 权限管理

不同用户登陆后台管理系统拥有不同的彩蛋权限和功能权限

权限管理包含三个功能模块：菜单管理，角色管理，用户管理

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206021518699.png" alt="07 权限管理需求" style="zoom:67%;" />

1、创建权限管理模块service-acl

2、引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>com.yienx</groupId>
        <artifactId>spring_security</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
    </dependency>
</dependencies>
```



## SpringSecurity

- **用户认证**(**Authentication**)

	验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。

- **用户授权**(**Authorization**)

	验证某个用户是否有权限执行某个操作。

> **Spring Security**其实就是用**filter**，多请求的路径进行过滤。

1. 如果是基于Session，那么Spring-security会对cookie里的sessionid进行解析，找到服务器存储的sesion信息，然后判断当前用户是否符合请求的要求。

2. 如果是token，则是解析出token，然后将当前请求加入到Spring-security管理的权限信息中去

[参考](https://blog.csdn.net/lendsomething/article/details/119251123)

### 实现原理

如果系统的模块众多，每个模块都需要就行授权与认证，所以我们选择基于 token 的形式进行授权与认证。

1、用户根据用户名密码认证成功，然后从数据库获取当前用户角色的一系列**权限值**，

2、并以用户名为 key，权限列表为 value 的形式存入 redis 缓存中。

3、根据用户名相关信息生成 token 返回，

4、浏览器将 token 记录到 cookie 中， 每次调用api接口都默认将 token 携带到 header 请求头中。

5、Spring-security 从 header 中解析获取 token 信息，解析 token 获取当前用户名，根据用户名就可以从 **redis** 中获取权限列表，

6、这样 Spring-security 就能够判断当前请求是否有权限访问，给用户赋予权限

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206022017607.png" alt="01 Spring Security授权过程" style="zoom:75%;" />

### 实现

```xml
<dependencies>
    <dependency>
        <groupId>com.atguigu</groupId>
        <artifactId>common_utils</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
	<!-- Spring Security依赖 --> 
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
    </dependency>
</dependencies>
```

代码结构

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206022014018.png" alt="image-20220602201435579" style="zoom:50%;" />

Spring Security 的核心配置就是继承 WebSecurityConfigurerAdapter 并注解 @EnableWebSecurity 的配置。

这个配置指明了用户名密码的处理方式、请求路径的开合、登录登出控制等和安全相关的配置

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206022118689.png" alt="02 Spring Security代码执行过程" style="zoom:75%;" />

# 两个重要接口

## UserDetailsService

查询数据库用户名和密码的过程。

* 创建类维承UsermamepasswordAuthenticationFiter，重写三个方法
* 创建类实现UserDetailService，编写查询数据库过程，返回User对象，这个User对象是安全框架提供的。

## PasswordEncoder

密码编码过程



