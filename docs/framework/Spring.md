## Spring、Spring MVC和Spring Boot

- spring是⼀个IOC容器，⽤来管理Bean，使⽤依赖注⼊实现控制反转，可以很⽅便的整合各种框架，提供AOP机制弥补OOP的代码重复问题、更⽅便将不同类不同⽅法中的共同处理抽取成切⾯、⾃动注⼊给⽅法执⾏，⽐如⽇志、异常等 。
- springmvc是spring对web框架的⼀个解决⽅案，提供了⼀个总的`前端控制器DispatcherServlet`，⽤来接收请求，然后定义了⼀套路由策略（url到handle的映射）及适配执⾏handle，将handle结果使⽤视图解析技术⽣成视图展现给前端 。
- springboot是spring提供的⼀个快速开发⼯具包，让程序员能更⽅便、更快速的开发spring+springmvc应⽤，简化了配置（约定了默认配置），整合了⼀系列的解决⽅案（starter机制）、redis、mongodb、es，可以开箱即⽤。





## 1、Spring特点

- 轻量级
	- 大小和开销都是轻量级
	- 非侵入式：典型的，Spring应用中的对象不依赖于Spring的特定类
- 控制反转IOC
	- 转移创建对象的控制权，将创建对象的控制权从开发者转移到了Spring框架
	- 目的：降低耦合度
	- 底层原理：xml解析、工厂模式、反射
	- **依赖注入DI**：IOC容器（K-V）在运行期间，动态地将某种依赖关系注入到对象中
		- set方法进行注入
		- 有参构造进行注入
		- p空间：set方法，需要加入p命名空间
		- c空间：有参构造，需要加入c命名空间
	- 通过**配置文件**或**配置类+注解**描述Bean和Bean之间的依赖关系
- 面向切面AOP
	- 将公共行为（如记录日志，权限校验等）封装到可重用的模块中，而使原本的模块内只需关注自身的个性化行为
	- 对业务逻辑的各个部分机型隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，提高了开发效率
	- 通俗描述：不通过修改源代码的方式，在主干功能里面添加新功能
	- 底层原理：动态代理
- 容器
	- Spring包含并管理应用对象的配置和生命周期
- 框架集合
	- Spring可以将简单的组件配置、组合成复杂的应用



## 2、IOC

### 容器

实际上就是个map（key，value），存放配置文件里配置的各种对象；容器底层是对象工厂。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241057549.jpg" alt="IMG_1008" />

Bean缓存池为HashMap实现

#### 实现方式（两个接口）

- **BeanFactory**

	- Spring框架的基础设施，面向Spring本身，BeanFactory是接口，**提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范**

	- 加载配置文件的时候不会创建对象，在获取对象（使用）才去创建对象

- **ApplicatiContext**

	- 继承了BeanFactory，面向使用Spring框架的开发者（几乎都是使用这个），提供更强大的功能

	- 加载配置文件的时候就会把在配置文件中的对象进行创建
	- 实现类
		- FileSystemXmlApplicationContext：绝对路径
		- ClassPathXmlApplicationContext：相对路径



### Bean管理

指的是两个操作

1. Spring创建对象（管理对象的生命周期）
2. Spring注入属性

实现方式：**xml配置文件**或**注解**



#### Bean的作用域（Scope）

> **singleton**、**prototype**
>
> 仅WebApplicationContext环境：request、session、global-session

- **singleton**

	- 单实例，只创建一次

	- 默认，每个容器中只有一个bean实例，由BeanFactory自身来维护

- **prototype**

	- 多实例，每次使用时创建

	- 为每一个bean请求创建一个新的bean实例

> 一般，对于有状态的bean使用prototype，无状态的bean使用singleton
>
> - 有状态：有实例变量，线程不安全
> - 无状态：无实例变量，如Service层、DAO层，线程安全
>
> 线程安全与否还是跟Bean对象本身有关，作用域只是表示Bean的生命周期范围

- **request**
	- 一次request一个实例
	
	- 仅在当前Http Request内有效
	
- **session**

	- 类似request

- **global-session**

	- 一般应用于portlet应用环境



#### Bean的生命周期

1. 实例化一个bean，new
2. 依赖注入，根据Spring上下文对实例化的bean进行配置
3. 处理Aware回调（可选）
	- BeanFactoryAware接口实现和ApplicationContextAware接口实现
	- 就是获取Spring上下文
4. 后置处理器，若实现了BeanPostProcess接口
	- postProcessBeforeInitialization方法，初始化前执行的方法
5. 初始化，init-method
6. 后置处理器，若实现了BeanPostProcess接口
	- postProcessAfterInitialization方法初始化后执行的方法
7. 获取创建bean实例对象
8. destroy销毁



#### 注解方式

##### 创建对象

- @Component
- @Service
- @Controller
- @Repository（DAOImpl使用）

```java
@Component(value = "userService")
// 默认是类名的首字母小写
```



##### 自动装配

xml文件中autowire属性

- no - 缺省的情况下，通过ref属性手动设定
- **byName** - 根据bean的属性名称进行自动装配
- **byType** - 根据bean的类型进行自动装配
- constructor - 类似byType
- autodetect - 如果有默认的构造器，则通过constructor，否则通过byType

注解方式

- @Autowired：根据属性类型进行装配
	- 接口有时会有多个实现类，此时单独用@Autowired会报错，所以常和@Qualifier一起使用
	- @Qualifier根据名称进行注入

```java
@Service
public class UserService {
    @Autowired
    @Qualifier(value = "userDaoImpl01")
    private UserDao userDao;
    
    public void add() {
        userDao.add();
    }
}
```

- @Resource：可以根据类型也可以根据名称

```java
// 注：是javax中的，不是spring中的
// @Resource
@Resource(name = "userDaoImpl01")
```

- @Value：注入普通类型属性

```java
@Value(value = "abc")
private String name;
```



**配置类**

```java
// 作为配置类，替代xml配置文件
@Configuration
@ComponentScan(basePackages = {"com.yienx"})
public class SpringConfig {
    
}
```



## 3、AOP

Spring中的事务管理就用到AOP

### 核心概念

- 连接点（joinpoint）
	- 可以被增强的方法
	- 被拦截的点，被拦截到的方法
- 切入点（pointcut）
	- 实际被增强的方法，有的方法可能没被增强
	- 对连接点进行拦截的定义
- 通知（advice，又称增强）
	- 实际增强的逻辑部分
	- 拦截到连接点之后要执行的代码
	- 类型
		- 前置通知Before
		- 后置通知After Returning（方法返回值之后执行）
		- 环绕通知Around
		- 异常通知After Throwing
		- 最终通知After（方法之后执行）
- 切面（aspect）
	- 是动作，把通知应用到切入点的过程
	- 类是对物体特征的抽象，切面就是对横切关注点的抽象
		- 横切关注点：对哪些方法进行拦截，拦截之后怎么处理，这些关注点称之为横切关注点
- 织入（weave）
	- 将切面应用到代理的目标对象并导致代理对象创建的过程
- 引入（introduction）
	- 在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241614714.jpeg" alt="AOP核心概念" />

### AOP中的动态代理

> [动态代理](../base/动态代理.md)

- JDK动态代理：有接口的情况
- CGLIB动态代理，没有接口的情况

#### JDK动态代理

- 创建接口实现类代理对象，增强类的方法
- 只能为接口创建代理实例

#### CGLIB动态代理

- 创建子类的代理对象，增强类的方法



#### AspectJ

- Spring一般基于AspectJ实现AOP操作，其不是Spring 的组成，是独立AOP框架
- Spring会自行判断并调用JDK还是CGLIB



## 4、Spring中的事务管理

- 编程式
- **声明式**（使用）
	- **注解**（使用）
	- xml配置文件
	- **底层：AOP**



提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241853683.png" alt="image-20220324185244188" />



**步骤**

1. 配置文件中配置事务管理器
2. 配置文件中开启事务注解（先引入名称空间tx）
3. 在service类上添加事务注解@Transactional
	- 注解加在类上，这个类里的所有方法都添加事务
	- 注解加在方法上，为这个方法添加事务



### @Transactional中的参数配置

1. propagation：**事务传播行为**

	- 事务方法：对数据库表数据进行改变的操作
	- 多事务方法直接进行调用，这个过程中事务是如何进行管理的
	- 比如A方法加了事务（@Transactional），B方法没有加事务，A调用B的行为就叫事务传播行为；反之亦是。
	- 事务的传播行为可以由传播属性指定，Spring中定义了7类传播行为

	<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241958645.png" alt="事务传播行为" style="zoom:80%;" />
	
	- 举例
	
		<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203242139692.png" alt="image-20220324213941231" style="zoom:40%" />
	
		- REQUIRED
			- 如果add方法本身有事务，调用update方法之后，update使用当前add方法里面的事务；如果add方法本身没有事务，调用update方法后，创建新事务
		- REQUIRED_NEW
			- 如果add方法调用update方法，无论add有没有事务，都创建新的事务


2. isolation：事务隔离级别
3. timeout：超时时间
	- 事务需要再一定时间内进行提交，如果不提交就进行回滚
	- 默认-1，设置时间以秒为单位
4. readOnly：是否只读
	- 默认false，可以查询，可以添加修改删除操作
	- 设置true后，只能查询
5. rollbackFor：回滚
	- 设置哪些异常进行回滚
6. noRollbackFor：不回滚
	- 设置哪些异常不进行回滚



## 5、Spring中的设计模式

- 工厂模式：BeanFactory、FactoryBean
	- FactoryBean是一种bean，在Spring内部广泛使用，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似 

- 代理模式：AOP
	- 事务管理中使用

- 模版模式：JdbcTemplate



## 6、Spring MVC原理

MVC 是一种软件架构的思想，将软件按照模型、视图、控制器（MVC）来划分，围绕前端控制器 DispatcherServlet 来设计的。

**MVC的工作流程**: 

1. 浏览器发送请求至前端控制器 DispatcherServlet
2. DispatcherServlet 收到请求调用 HandlerMapping 处理器映射器
3. HandlerMapping 找到具体的处理器（根据xml配置、注解进行查找），生成处理器及处理器拦截器（如果有则生成）一并返回给 DispatcherServlet
4. DispatcherServlet 调⽤ HandlerAdapter 处理器适配器。 
4. HandlerAdapter 经过适配调⽤具体的处理器(Controller，也叫后端控制器) 
4. Controller 执⾏完成返回 ModelAndView。 
4. HandlerAdapter 将 controller 执⾏结果 ModelAndView 返回给 DispatcherServlet。
4. DispatcherServlet 将 ModelAndView 传给 ViewReslover 视图解析器。 
4. ViewReslover 解析后返回具体 View。 
4. DispatcherServlet 根据 View 进⾏渲染视图（即将模型数据填充⾄视图中）
4. DispatcherServlet 响应⽤户

> 浏览器访问、映射到前端控制器、读取配置文件、组件扫描找到控制器、根据mapping转发、控制器返回、视图解析、视图渲染

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203251523555.png" alt="image-20220325152345234" />

SpringMVC 是 Spring 的一个后续产品，是 Spring 的一个子项目。
