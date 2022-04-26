官方地址: http://mp.baomidou.com

代码发布地址:

Github: https://github.com/baomidou/mybatis-plus 

Gitee: https://gitee.com/baomidou/mybatis-plus 

文档发布地址: https://baomidou.com/pages/24112f

详情见pdf



# 初始化

## 连接地址url

MySQL5.7版本的url:

```yaml
jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf-8&useSSL=false
```

MySQL8.0版本的url:

```yaml
jdbc:mysql://localhost:3306/mybatis_plus?serverTimezone=GMT%2B8&characterEncoding=utf-8&useSSL=false
```

Ps：serverTimezone 是时区

## 添加mapper

```java
public interface UserMapper extends BaseMapper<User> {
}
```

> IDEA在 userMapper 处报错，因为找不到注入的对象，因为类是动态创建的，但是程序可以正确的执行。
>
> 为了避免报错，可以在mapper接口上添加 @Repository 注解

根据范型名和内部的参数，查到对应的表





