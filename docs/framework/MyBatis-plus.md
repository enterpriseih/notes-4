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



# 常用注解

## @TableName

### 1、通过@TableName解决问题

在实体类类型上添加@TableName("t_user")，标识实体类对应的表，即可成功执行SQL语句

### 2、通过全局配置解决问题

实体类所对应的表都有固定的前缀，例如 t 或 tbl 

此时，可以使用MyBatis-Plus提供的全局配置，为实体类所对应的表名设置默认的前缀，那么就不需要在每个实体类上通过@TableName标识实体类对应的表

```yaml
mybatis-plus:
  global-config:
	db-config:
		# 配置MyBatis-Plus操作表的默认前缀 
		table-prefix: t_
```



## @TableId

若实体类和表中表示主键的不是id，而是其他字段，例如uid，MyBatis-Plus会自动识别uid为主键列吗?

### 1、通过@TableId解决问题

在实体类中uid属性上通过@TableId将其标识为主键，即可成功执行SQL语句

### 2、@TableId的value属性

> 若实体类中主键对应的属性为id，而表中表示主键的字段为uid，此时若只在属性id上添加注解 @TableId，则抛出异常Unknown column 'id' in 'field list'，即MyBatis-Plus仍然会将id作为表的主键操作，而表中表示主键的是字段uid
>
> 此时需要通过@TableId注解的value属性，指定表中的主键字段，@TableId("uid")或 @TableId(value="uid")

### 3、@TableId的type属性

#### 通过 type 自定义主键策略

| 值                       | 描述                                                         |
| :----------------------- | :----------------------------------------------------------- |
| IdType.ASSIGN_ID（默认） | 基于雪花算法的策略生成数据id，与数据库id是否设置自增无关     |
| IdType.AUTO              | 使用数据库的自增策略，注意，该类型请确保数据库设置了id自增， 否则无效 |

```yaml
mybatis-plus:
  global-config:
	db-config:
		# 配置MyBatis-Plus的主键策略 
		id-type: auto
```



#### 雪花算法

##### 背景

需要选择合适的方案去应对数据规模的增长，以应对逐渐增长的访问压力和数据量。

数据库的扩展方式主要包括:业务分库、主从复制，数据库分表。

##### 数据库分表

垂直分表

水平分表

**核心思想**

一共64位，long 型

首先是一个符号位，1bit标识，由于long基本类型在Java中是带符号的，最高位是符号位，正数是0，负 数是1，所以id一般是正数，最高位是0。

41bit时间截(毫秒级)，存储的是时间截的差值(当前时间截 - 开始时间截)，结果约等于69.73年。 

10bit作为机器的ID(5个bit是数据中心，5个bit的机器ID，可以部署在1024个节点)。 

12bit作为毫秒内的流水号(意味着每个节点在每毫秒可以产生 4096 个 ID)。



## @TableField

### 1、情况1

若实体类中的属性使用的是驼峰命名风格，而表中的字段使用的是下划线命名风格 

例如实体类属性userName，表中字段user_name 

此时MyBatis-Plus会自动将下划线命名风格转化为驼峰命名风格 

相当于在MyBatis中配置

### 2、情况2

若实体类中的属性和表中的字段不满足情况1 

例如实体类属性name，表中字段username 

此时需要在实体类属性上使用 @TableField("username") 设置属性所对应的字段名



## @TableLogic

### 1、逻辑删除

物理删除:真实删除，将对应数据从数据库中删除，之后查询不到此条被删除的数据 

逻辑删除:假删除，将对应数据中代表是否被删除字段的状态修改为“被删除状态”，之后在数据库中仍旧能看到此条数据记录

使用场景:可以进行数据恢复

### 2、实现

step1：数据库中创建逻辑删除状态列（例如 is_deleted），设置默认值为0

step2：实体类中添加逻辑删除属性 @TableLogic

step3：测试

测试删除功能，真正执行的是修改

UPDATE t_user SET is_deleted=1 WHERE id=? AND is_deleted=0 

测试查询功能，被逻辑删除的数据默认不会被查询

SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE is_deleted=0



# 条件构造器和常用接口

Wrapper : 条件构造抽象类，最顶端父类

- AbstractWrapper ：用于查询条件封装，生成 sql 的 where 条件
	- QueryWrapper ：查询条件封装 
	- UpdateWrapper ：Update 条件封装 
	- AbstractLambdaWrapper ：使用Lambda 语法
		- LambdaQueryWrapper ：用于Lambda语法使用的查询 Wrapper 
		- LambdaUpdateWrapper ：Lambda 更新封装 Wrapper

## QueryWrapper

优先级，and 比 or 大

lambda表达式内的逻辑优先运算

## UpdateWrapper



## LambdaQueryWrapper

```java
@Test
public void test09() {
    //定义查询条件，有可能为null(用户未输入)
    String username = "a";
    Integer ageBegin = 10;
    Integer ageEnd = 24;
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>(); 
    //避免使用字符串表示字段，防止运行时错误
    queryWrapper
        .like(StringUtils.isNotBlank(username), User::getName, username)
        .ge(ageBegin != null, User::getAge, ageBegin)
        .le(ageEnd != null, User::getAge, ageEnd);
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```



## LambdaUpdateWrapper



# 插件

## 1、分页插件

配置类

```java
@Configuration
@MapperScan("com.atguigu.mybatisplus.mapper") //可以将主类中的注解移到此处 
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    } 
}

```

分页插件默认是查询所有

```java
@Test
public void testSelectPage(){ 
    //设置分页参数
	Page<User> page = new Page<>(1, 5); 
    userMapper.selectPage(page, null);
	//获取分页数据
	List<User> list = page.getRecords(); 
    list.forEach(System.out::println); 
    System.out.println("当前页:"+page.getCurrent()); 
    System.out.println("每页显示的条数:"+page.getSize()); 
    System.out.println("总记录数:"+page.getTotal()); 
    System.out.println("总页数:"+page.getPages()); 
    System.out.println("是否有上一页:"+page.hasPrevious()); 
    System.out.println("是否有下一页:"+page.hasNext());
}
```



## 2、xml 自定义分页

```java
/**
* 根据年龄查询用户列表，分页显示
* @param page 分页对象,xml中可以从里面进行取值,
* 传递参数 Page 即自动分页,必须放在第一位 * @param age 年龄
* @return
*/
IPage<User> selectPageVo(@Param("page") Page<User> page, @Param("age")
Integer age);
```



```xml
<!--IPage<User> selectPageVo(Page<User> page, Integer age);-->
<select id="selectPageVo" resultType="User">
    SELECT id,username,age,email FROM t_user WHERE age > #{age}
</select>
```



```yaml
mybatis-plus:
	# 配置类型别名所对应的包
	type-aliases-package: com.atguigu.mybatis.plus.pojo
```



测试

```java
@Test
public void testSelectPageVo(){ 
    //设置分页参数
	Page<User> page = new Page<>(1, 5); 
    // 仅此处不同
    userMapper.selectPageVo(page, 20);
	//获取分页数据
	List<User> list = page.getRecords(); 
    list.forEach(System.out::println); 
    System.out.println("当前页:"+page.getCurrent()); 
    System.out.println("每页显示的条数:"+page.getSize()); 
    System.out.println("总记录数:"+page.getTotal()); 
    System.out.println("总页数:"+page.getPages()); 
    System.out.println("是否有上一页:"+page.hasPrevious()); 
    System.out.println("是否有下一页:"+page.hasNext());
}
```





## 3、乐观锁

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor(){ 
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor(); 
    //添加分页插件
	interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL)); 
    //添加乐观锁插件
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    return interceptor;
}
```



```java
@Data
public class Product {
    private Long id;
    private String name;
    private Integer price;
    @Version
    private Integer version;
}
```



### 乐观锁实现流程

数据库中添加version字段

取出记录时，获取当前version

```sql
SELECT id,`name`,price,`version` FROM product WHERE id=1
```

> ``是为了区分字段名，一般不需要加

更新时，version + 1，如果where语句中的version版本不对，则更新失败

```sql
UPDATE product SET price=price+50, `version`=`version` + 1 WHERE id=1 AND `version`=1
```



# 通用枚举

表中的有些字段值是固定的，例如性别(男或女)，

此时我们可以使用MyBatis-Plus的通用枚举 来实现

```java
@Getter
public enum SexEnum { 
    MALE(1, "男"),
	FEMALE(2, "女");
	@EnumValue
    private Integer sex;
    private String sexName;
    SexEnum(Integer sex, String sexName) {
        this.sex = sex;
		this.sexName = sexName;
    }
}
```



```yaml
mybatis-plus:
    # 配置扫描通用枚举
	type-enums-package: com.atguigu.mybatisplus.enums
```



```java
@Test
public void testSexEnum(){
    User user = new User();
    user.setName("admin");
    user.setAge(20); 
    //设置性别信息为枚举项，会将@EnumValue注解所标识的属性值存储到数据库 
    user.setSex(SexEnum.MALE);
    //INSERT INTO t_user ( username, age, sex ) VALUES ( ?, ?, ? )
    //Parameters: admin(String), 20(Integer), 1(Integer)
    userMapper.insert(user);
}
```



# 配置参数总结

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  # 设置MyBatis-Plus的全局配置
  global-config:
    db-config:
      # 设置实体类所对应的表的统一前缀
      table-prefix: t_
      # 设置统一的主键生成策略
      id-type: auto
  # 配置类型别名所对应的包
  type-aliases-package: com.atguigu.mybatisplus.pojo
  # 扫描通用枚举的包
  type-enums-package: com.atguigu.mybatisplus.enums
```



# 代码生成器



```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.1</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>
```



```java
public static void main(String[] args) {
    FastAutoGenerator.create("jdbc:mysql://118.178.135.86:3306/mybatis_plus?serverTimezone=GMT%2B8&characterEncoding=utf-8&useSSL=false", "root", "root1205")
        .globalConfig(builder -> {
            builder.author("yienx") // 设置作者
                //.enableSwagger() // 开启 swagger 模式
                .fileOverride() // 覆盖已生成文件
                .outputDir("./"); // 指定输出目录，从工程所在文件夹内开始算
        })
        .packageConfig(builder -> {
            builder.parent("com.yienx") // 设置父包名
                .moduleName("mybatisplus") // 设置父包模块名
                .pathInfo(Collections.singletonMap(OutputFile.mapperXml, "./")); // 设置mapperXml生成路径
        })
        .strategyConfig(builder -> {
            builder.addInclude("user") // 设置需要生成的表名
                .addTablePrefix("t_", "c_"); // 设置过滤表前缀
        })
        .templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板
        .execute();
}

```





# 多数据源

适用于多种场景:纯粹多库、 读写分离、 一主多从、 混合模式等 

模拟一个纯粹多库的一个场景，其他场景类似。

场景说明:

创建两个库，

分别为:mybatis_plus(以前的库不动)与mybatis_plus_1(新建)，将 mybatis_plus 库的product表移动到 mybatis_plus_1 库，

这样每个库一张表，通过一个测试用例分别获取用户数据与商品数据，如果获取到说明多库模拟成功

## 配置

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.5.0</version>
</dependency>
```



说明：注释掉之前的数据库连接，添加新配置

```yaml
spring:
  # 配置数据源信息
  datasource:
    dynamic:
      # 设置默认的数据源或者数据源组,默认值即为master
      primary: master
      # 严格匹配数据源,默认false.true未匹配到指定数据源时抛异常,false使用默认数据源
      strict: false
      datasource:
        master:
          url: jdbc:mysql://localhost:3306/mybatis_plus01?characterEncoding=utf-8&useSSL=false
          driver-class-name: com.mysql.cj.jdbc.Driver
          username: root
          password: 123456
        slave_1:
          url: jdbc:mysql://localhost:3306/mybatis_plus02?characterEncoding=utf-8&useSSL=false
          driver-class-name: com.mysql.cj.jdbc.Driver
          username: root
          password: 123456
          
```



## service 中配置注解

```java
@DS("master")
// 在XXXServiceImpl上，指定所操作的数据源
@DS("slave_1")
```



# MyBatisX插件

在真正开发过程中，MyBatis-Plus并不能为我们解决所有问题，例如一些复杂的SQL，多表 联查，我们就需要自己去编写代码和SQL语句，我们该如何快速的解决这个问题呢，这个时候可 以使用MyBatisX插件

MyBatisX插件用法:https://baomidou.com/pages/ba5b24/



需要在idea的database里右键相应的表来生成