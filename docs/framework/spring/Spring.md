# Spring、Spring MVC和Spring Boot

- spring是⼀个IOC**容器框架**，⽤来**管理Bean** (Java 对象)，使⽤依赖注⼊实现控制反转，可以很⽅便的整合各种框架，提供AOP机制弥补OOP (面向对象程序设计，`Object Oriented Programming`) 的代码重复问题、更⽅便将不同类不同⽅法中的共同处理抽取成切⾯、⾃动注⼊给⽅法执⾏，⽐如⽇志、异常等 。
- springmvc是spring**对web框架的⼀个解决⽅案**，提供了⼀个总的`前端控制器DispatcherServlet`，⽤来接收请求，然后定义了⼀套路由策略（url到handle的映射）及适配执⾏handle，将handle结果使⽤视图解析技术⽣成视图展现给前端 。
- springboot是spring提供的⼀个快速开发⼯具包，让程序员能更⽅便、更快速的开发spring+springmvc应⽤，**简化了配置**（约定了默认配置），整合了⼀系列的解决⽅案（starter机制）、redis、mongodb、es，可以开箱即⽤。



# Spring特点

- 轻量级
	- 大小和开销都是轻量级
	- 非侵入式：典型的，Spring应用中的对象不依赖于Spring的特定类
- **控制反转IOC**
	- **转移创建对象的控制权**，将创建对象的控制权从开发者转移到了Spring框架
	- 目的：**降低耦合度**
	- 底层原理：xml解析、工厂模式、反射
	- **依赖注入DI**：IOC容器（K-V）在运行期间，动态地将某种依赖关系注入到对象中
		- set方法进行注入
		- 有参构造进行注入
		- p空间：set方法，需要加入p命名空间
		- c空间：有参构造，需要加入c命名空间
	- 通过**配置文件**或**配置类+注解**描述Bean和Bean之间的依赖关系
- **面向切面AOP**
	- 将程序中的交叉业务逻辑（如记录日志，权限校验等），封装成一个切面（可重用模块），然后注入到目标对象（具体业务逻辑）中。可以将对象的功能进行增强。
	- 通俗描述：不通过修改源代码的方式，在主干功能里面添加新功能
	- 底层原理：**动态代理**
- **容器**
	- Spring包含并管理应用对象的配置和生命周期
- **框架**
	- Spring可以将简单的组件配置、组合成复杂的应用

# Spring的三大核心

beans、context、core

1、**beans**：beans 又是核心中最重要的一个，因为其它两个都是围绕它的。Spring通过IOC管理的就是Beans，没有beans，Spring就没有存在的必要了。

2、**context**：上下文，这里术语 “上下文” 中的文，其实就是 Spring 管理的 bean。我们将 Spring 容器看成是一片文章，而 bean 就是每个段落或者每句文字，而 “上下” 我们可以理解成 Java 中 bean 与 bean 之间的依赖（引用）。context 作用就是负责管理 Spring 中 bean 与 bean 之间关系的。

3、**core**：作用主要是为 context 在管理 Spring 中 bean 与 bean 之间关系时为其服务的。其实直白一点就是为 Spring 管理 bean 提交工具的一个工具(类)。

# Spring的启动流程

https://blog.csdn.net/a745233700/article/details/113761271

Spring的启动流程可以归纳为三个步骤：

- 1、初始化Spring容器，注册内置的BeanPostProcessor的BeanDefinition到容器中
- 2、将配置类的BeanDefinition注册到容器中
- 3、调用refresh()方法刷新容器

# IOC

## 一、容器

实际上就是个map（key，value），存放配置文件里配置的各种对象；容器底层是对象工厂。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241057549.jpg" alt="IMG_1008" />

Bean缓存池为HashMap实现

在项目启动的时候读取配置的bean节点，根据全限定类名使用反射创建对象放到map里。

## 二、实现方式（两个接口）

- **BeanFactory**

	- Bean工厂，生成bean，维护bean，BeanFactory是接口，**提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范**

	- 加载配置文件的时候不会创建对象，在获取对象（使用）才去创建对象

- **ApplicatiContext**

	- 继承了BeanFactory，面向使用Spring框架的开发者（几乎都是使用这个），提供更强大的功能

	- 加载配置文件的时候就会把在配置文件中的对象进行创建
	- 实现类
		- FileSystemXmlApplicationContext：绝对路径
		- ClassPathXmlApplicationContext：相对路径



## 三、Bean管理

指的是两个操作

1. Spring创建对象（管理对象的生命周期）
2. Spring注入属性

实现方式：**xml配置文件**或**注解**



### 1、Bean的作用域（Scope）

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

#### 单例模式和单例bean

单例模式是指在一个**JVM进程中**仅有一个实例，而单例bean是指在一个**Spring Bean容器** (ApplicationContext) 中仅有一个实例。一个JVM进程可以有多个SpringBean容器。

```java
//  第一个Spring Bean容器
ApplicationContext context_1 = new FileSystemXmlApplicationContext("classpath:/ApplicationContext.xml");
Person person1 = context_1.getBean("person", Person.class);
//  第二个Spring Bean容器
ApplicationContext context_2 = new FileSystemXmlApplicationContext("classpath:/ApplicationContext.xml");
Person person2 = context_2.getBean("person", Person.class);
//  这里绝对不会相等，因为创建了多个实例
System.out.println(person1 == person2);
```

Spring的单例是用单例注册表实现的。

### 2、Bean的生命周期***

> 构造函数 --> 依赖注入 --> init-method

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202209071107125.png" alt="图片" style="zoom:67%;" />

1. **实例化**一个bean，new
2. **依赖注入**，根据Spring上下文对实例化的bean进行配置、**填充属性**
3. 处理Aware回调（可选）
	- BeanFactoryAware接口实现和ApplicationContextAware接口实现
	- **就是获取Spring上下文**
4. 后置处理器，若实现了BeanPostProcess接口
	- postProcessBeforeInitialization方法，初始化前执行的方法
	- 初始化前处理@PostConstruct注解
5. 初始化，init-method
6. 后置处理器，若实现了BeanPostProcess接口
	- postProcessAfterInitialization方法，初始化后执行的方法
	- 初始化后进行AOP
7. 获取创建bean实例对象
8. destroy销毁



### 3、注解方式

#### 3.1、创建对象

- @Component
- @Service
- @Controller
- @Repository（DAOImpl使用）

```java
@Component(value = "userService")
// 默认是类名的首字母小写
```



#### 3.2、自动装配

#### 3.2.1、xml文件中autowire属性

- no - 缺省的情况下，通过ref属性手动设定
- **byName** - 根据bean的属性名称进行自动装配
- **byType** - 根据bean的类型进行自动装配
- constructor - 类似byType，不过是应用于构造起的参数；如果一个bean于构造器参数的类型相同，则进行自动装配，否则导致异常。
- autodetect - 如果有默认的构造器，则通过constructor，否则通过byType

#### 3.2.2、注解方式

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

	默认byName，无法通过名称注入的时候，转为byType

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

> @Autowired底层原理
>
> 1. 默认优先按照类型去容器中找对应的组件：applicationContext.getBean()
> 2. 如果找到多个相同类型的组件，再将属性的名称作为组件的ID去容器中查找
>
> https://blog.csdn.net/X18406432083/article/details/125967348

#### 3.2.3、配置类

```java
// 作为配置类，替代xml配置文件
@Configuration
@ComponentScan(basePackages = {"com.yienx"})
public class SpringConfig {
    
}
```

> @Component注解表明一个类会作为组件类，并告知Spring要为这个类创建bean。
>
> @Bean注解告诉Spring这个方法将会返回一个对象，这个对象要注册为Spring应用上下文中的bean。通常方法体中包含了最终产生bean实例的逻辑。

## 四、Bean是线程安全的吗？

无状态 -- 线程安全

有状态 -- 线程不安全

> 如果单例Bean，是一个无状态Bean，也就是线程中的操作不会对Bean的成员执行查询以外的操作，那么这个单例Bean是线程安全的。比如Spring mvc 的 Controller、Service、Dao等，这些Bean大多是无状态的，只关注于方法本身。singleton

**有状态对象**(Stateful Bean) ：就是**有实例变量的对象**，可以保存数据，是非线程安全的。每个用户有自己特有的一个实例，在用户的生存期内，bean保持了用户的信息，即“有状态”；一旦用户灭亡（调用结束或实例结束），bean的生命期也告结束。即每个用户最初都会得到一个初始的bean。

**无状态对象**(Stateless Bean)：就是**没有实例变量的对象**，不能保存数据，是不变类，是线程安全的。bean一旦实例化就被加进会话池中，各个用户都可以共用。即使用户已经消亡，bean 的生命期也不一定结束，它可能依然存在于会话池中，供其他用户调用。由于没有特定的用户，那么也就不能保持某一用户的状态，所以叫无状态bean。但无状态会话bean 并非没有状态，如果它有自己的属性（变量），那么这些变量就会受到所有调用它的用户的影响，这是在实际应用中必须注意的。

> “ DAO对象必须包含一个数据库的连接Connection，而这个Connection不是线程安全的，所以每个DAO都要包含一个不同的Connection对象实例，这样一来DAO对象就不能是单实例的了。”
>
> 上述观点对了一半。对的是“每个DAO都要包含一个不同的Connection对象实例”这句话，错的是“DAO对象就不能是单实例”。其实Spring在实现Service和DAO对象时，使用了**ThreadLocal**这个类，这个是一切的核心！
>
> Dao会操作数据库Connection，Connection是带有状态的，比如说数据库事务，Spring的事务管理器使用Threadlocal为不同线程维护了一套独立的connection副本，保证线程之间不会互相影响（**Spring是如何保证事务获取同一个Connection的**）
>
> 对于有状态的bean，Spring官方提供的bean，一般提供了通过ThreadLocal去解决线程安全的方法

# AOP

Spring中的事务管理就用到AOP

## 一、核心概念

- 目标（target）：被通知的对象
- 连接点（joinpoint）
	- 可以被增强的方法：目标对象的所属类中，定义的所有方法都是连接点
	- 被拦截的点，被拦截到的方法
- 切入点（pointcut）
	- 实际**被增强的方法**，有的方法可能没被增强
	- 对连接点进行拦截的定义
	- 被增强的连接点：切入点一定是连接点，连接点不一定是切入点
- 通知（advice，又称增强）
	- 实际**增强的逻辑部分**
	- **拦截到连接点之后要执行的代码**
	- 类型
		- 前置通知Before
		- 后置通知After Returning（方法返回值之后执行）
		- 环绕通知Around
		- 异常通知After Throwing
		- 最终通知After（方法之后执行）
- 切面（aspect）：切入点+通知
	- 类是对物体特征的抽象，**切面就是对横切关注点的抽象（模块化）**
	  - 横切关注点：对哪些方法进行拦截，拦截之后怎么处理，这些关注点称之为横切关注点
	- **通知和切入点共同定义了切面的全部内容** — 它是什么，在何时和在何处完成其功能
- 织入（weave）
	- 将切面/通知应用到代理的目标对象并导致代理对象创建的过程
- 引入（introduction）
	- 在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241614714.jpeg" alt="AOP核心概念" />

> **多个切面的执行顺序**
>
> 1. 通过@Order注解
>
> 	@Order(3)值越小，优先级越高
>
> 2. 实现Ordered接口重写getOrder方法

## 二、AOP中的代理

> [动态代理](./动态代理.md)

- JDK动态代理：有接口的情况
- CGLIB动态代理，没有接口的情况

### 1、JDK动态代理

- 创建接口实现类代理对象，增强类的方法
- 只能为接口创建代理实例

### 2、CGLIB动态代理

- **创建子类的代理对象，增强类的方法**
- 生成目标类的子类，而子类是通过增强过的，这个子类对象就是代理对象。
- 所以，使用cglib生成动态代理，要求目标类必须能够被继承，即不能是final的类。

### 3、AspectJ

- Spring一般基于AspectJ实现AOP操作，其不是 Spring 的组成，是独立AOP框架
- Spring会自行判断并调用JDK还是CGLIB

> **Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。
>
> Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，
>
> 如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。



# Spring中的事务管理

- 编程式
- **声明式**（使用）@Transactional
	- **注解**（使用）
	- xml配置文件
	- **底层：AOP**



提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241853683.png" alt="image-20220324185244188" />



## 一、步骤

1. 配置文件中配置事务管理器
2. 配置文件中开启事务注解（先引入名称空间tx）
3. 在service类上添加事务注解@Transactional
	- 注解加在类上，这个类里的所有方法都添加事务
	- 注解加在方法上，为这个方法添加事务



## 二、事务的底层原理

1. Spring事务底层是基于**数据库事务**和**AOP**机制的

2. 首先对于使用了@Transactional注解的Bean， Spring会创建一个代理对象作为Bean

3. 当调用代理对象的方法时，会先判断该方法上是否加了@Transactional注解

4. 如果加了，那么则利用事务管理器创建一个数据库连接

5. 并且修改数据库连接的autocommit属性为false，禁止此连接的自动提交

6. 然后执行当前方法，方法中会执行sql

7. 执行完当前方法后，如果没有出现异常就直接提交事务

8. 如果出现了异常，并且这个异常是需要回滚的就会回滚事务，否则仍然提交事务

9. Spring事务的隔离级别对应的就是数据库的隔离级别



> Spring事务的传播机制是基于数据库连接来做的，一个数据库连接一个事务，如果传播机制配置为需要新开一个事务，那么实际上就是先建立一个数据库连接，在此新数据库连接上执行sql。



## 三、@Transactional中的参数配置

### 1、propagation：事务传播行为

- 事务方法：对数据库表数据进行改变的操作
- **多事务方法直接进行调用，这个过程中事务是如何进行管理的**
- 比如A方法加了事务（@Transactional），B方法没有加事务，A调用B的行为就叫事务传播行为；反之亦是。
- 事务的传播行为可以由传播属性指定，Spring中定义了7类传播行为
	1. REQUIRED（Spring默认）：如果有事务在运行，当前方法就加入这个事务；如果没有，则自己新建一个事务，并在自己的事务内运行。
	2. REQUIRED_NEW：创建一个新事务；如果有事务在运行，则挂起。
	3. SUPPORTS：如果有事务，则加入；如果没有，就以非事务方法执行。
	4. NOT_SUPPORTED：以非事务方法执行；如果有事务，则挂起。
	5. NEVER：以非事务方法执行；如果有事务，则抛出异常。
	6. NESTED：如果有事务在运行，就在这个事务的嵌套事务内运行；如果没有，则自己新建一个事务，并在自己的事务内运行。
	7. MANDATORY：如果有事务，则加入；如果没有，就抛出异常。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203241958645.png" alt="事务传播行为" style="zoom:80%;" />

- 举例

	<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203242139692.png" alt="image-20220324213941231" style="zoom:40%" />

	- REQUIRED
		- 如果add方法本身有事务，调用update方法之后，update使用当前add方法里面的事务；如果add方法本身没有事务，调用update方法后，创建新事务
	- REQUIRED_NEW
		- 如果add方法调用update方法，无论add有没有事务，都创建新的事务

### 2、isolation：事务隔离级别

### 3、timeout：超时时间

- 事务需要再一定时间内进行提交，如果不提交就进行回滚
- 默认-1，设置时间以秒为单位

### 4、readOnly：是否只读

- 默认false，可以查询，可以添加修改删除操作
- 设置true后，只能查询

### 5、rollbackFor：回滚

- 设置哪些异常进行回滚

### 6、noRollbackFor：不回滚

- 设置哪些异常不进行回滚



## 四、事务的隔离级别

与数据库的相同，如果数据库和spring都设置了，以spring的配置为准，但如果spring配置的隔离级别数据库不支持，再以数据库为准。



## 五、事务失效

spring事务的原理是AOP，进行了切面增强，失效的原因就是aop不起作用了。

1. **发生自调**，类里面使用this调用本类的方法（this通常省略），此时这个this对象不是代理类，而是UserService本身。

	解决：让this变成UserService的代理类即可`AopContext.currentProxy()`。

2. **方法不是public**，@Transactional只能用于public的方法上，否则事务不会生效；要用在非public方法上，需要开启AspectJ代理模式。

3. **数据库不支持事务**。

4. **没有被spring管理**。

5. **异常被吃掉**（被try...catch了），事务不会回滚

6. 抛出的**异常没有被定义**，**默认为RuntimeException及其子类与Error**；使用rollbakcFor指定非运行时异常。

### 补充：

#### 1、同类自调

- 1.1、A方法中无事务，B方法中有事务 => 无事务

在同一个类中的不同方法调用中，A方法调用B方法，A中无事务，B中有事务，此时事务不会生效。Spring采用动态代理（AOP）实现对bean的管理和切片，它为每个class都会生成一个代理对象。只有在代理对象之间调用时，可以触发切面逻辑。

而**同一个class中，方法A调用方法B，且方法A无事务，调用的是原对象this的方法，而不是通过代理对象**，因此Spring无法切换到这次调用，也就无法通过注解保证事务性了。

```
使用xml配置方式暴露代理对象/或者代理类上注解
然后在service中通过代理对象AopContext.currentProxy()去调用方法。
```

```xml
<aop:aspectj-autoproxy proxy-target-class="true" expose-proxy="true"/>
```

```java
@EnableAspectJAutoProxy(proxyTargetClass = true, exposeProxy = true)  
被代理类{
    ((被代理对象) AopContext.currentProxy()).xxx;
}

```



- 1.2、A方法中有事务，B方法中无事务 => 有事务

- 1.3、A方法中有事务，B方法中有事务 => 有事务：A的事务

#### 2、不同类

A方法中无事务，B方法中有事务 => B会回滚

# Spring中的循环依赖

https://blog.csdn.net/weixin_44129618/article/details/122839774

先讲实例化依赖注入：setter注入和构造器注入

再讲生命周期：实例化、属性赋值、初始化、销毁

## 三种情况

1. 构造器的循环依赖：这种依赖spring是处理不了的，直接抛出BeanCurrentlylnCreationException异常。
2. 单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖，能处理。
3. 非单例循环依赖：无法处理。原型(Prototype)的场景是不支持循环依赖的，通常会走到AbstractBeanFactory类中下面的判断，抛出异常。

```java
if (isPrototypeCurrentlyInCreation(beanName)) {  
    throw new BeanCurrentlyInCreationException(beanName);
}
```

> 只能解决setter注入的单例的bean。

## 三级缓存解决循环依赖

一级缓存：用于存放完全初始化好的bean

```java
/** Cache of singleton objects: bean name -> bean instance */
private final Map<String, Object> singletonObjects 
    = new ConcurrentHashMap<String, Object>(256);
```

二级缓存：存放原始的bean 对象（**尚末填充属性**的半成品），用于解决循环依赖

```java
/** Cache of early singleton objects: bean name -> bean instance */
private final Map<String, Object> earlySingletonObjects 
    = new HashMap<String, Object>(16);
```

三级缓存：存放bean工厂对象，用于解决循环依赖

```java
/** Cache of singleton factories: bean name -> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories 
    = new HashMap<String, ObjectFactory<?>>(16);
```

## 解决方式

### 二级缓存

假设一级缓存叫做map1，二级缓存叫做map2

首先实例化A，将还未填充的A对象的**引用**放入map2，然后A去属性填充B，发现B没有实例化，于是B同样实例化后，将半成品放入map2。

B开始填充，发现map1中没有A，就去map2中找，使用map2中的A填充自己，然后将自己B放入map1，并把map2中的半成品删除了。

回到A的阶段，A发现map1中有了B，那么A就完成了属性填充。

### 三级缓存

主要因为Spring的AOP产生的代理对象问题。

> Spring的代理对象产生阶段是在填充属性后才进行的，原理通过后置处理器BeanPostProcessor来实现。
>
> 如果 A 的原始对象注入给 B 的属性之后，A 的原始对象进行了 AOP 产生了一个代理对象，此时就会出现，对于 A 而言，它的 Bean 对象其实应该是 AOP 之后的代理对象，而 B 的 a 属性对应的并不是 AOP 之后的代理对象，这就产生了冲突。
>
> B 依赖的 A 和最终的 A 不是同一个对象。
>

三级缓存存放的是bean工厂对象，通过工厂在真正需要动态代理对象的时候才来获取。

1. 实例化 A，此时 A 还未完成属性填充和初始化方法（@PostConstruct）的执行，A 只是一个半成品。
2. 为 A 创建一个 Bean工厂，并放入到 singletonFactories 中。
3. 发现 A 需要注入 B 对象，但是一级、二级、三级缓存均为发现对象 B。
4. 实例化 B，此时 B 还未完成属性填充和初始化方法（@PostConstruct）的执行，B 只是一个半成品。
5. 为 B 创建一个 Bean工厂，并放入到 singletonFactories 中。
6. 发现 B 需要注入 A 对象，此时在一级、二级未发现对象 A，但是在三级缓存中发现了对象 A，从三级缓存中得到对象 A，并将对象 A 放入二级缓存中，同时删除三级缓存中的对象 A。（注意，此时的 A 还是一个半成品，并没有完成属性填充和执行初始化方法）
7. 将对象 A 注入到对象 B 中。
8. 对象 B 完成属性填充，执行初始化方法，并放入到一级缓存中，同时删除二级缓存中的对象 B。（此时对象 B 已经是一个成品）
9. 对象 A 得到对象B，将对象 B 注入到对象 A 中。（对象 A 得到的是一个完整的对象 B）
10. 对象 A完成属性填充，执行初始化方法，并放入到一级缓存中，同时删除二级缓存中的对象 A。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202208011304365.png" alt="在这里插入图片描述" style="zoom:67%;" />

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 一级缓存
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            // 二级缓存
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                // 三级缓存
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    // Bean 工厂中获取 Bean
                    singletonObject = singletonFactory.getObject();
                    // 放入到二级缓存中
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}

```

## AOP问题的解决

从 singletonFactories 根据 beanName 得到一个 ObjectFactory ，然后执行 ObjectFactory ，也就是执行 getEarlyBeanReference 方法，此时会得到一个 **A 原始对象经过 AOP 之后的代理对象**，然后把该代理对象放入 earlySingletonObjects 中。

> 如果有AOP就返回AOP的代理对象，没有就返回原始对象。

我们只得到了 A 原始对象的代理对象，这个对象还不完整，因为 A 原始对象还没有进行属性填充，所以此时不能直接把A的代理对象放入 singletonObjects 中，所以只能把代理对象放入earlySingletonObjects 。

假设现在有其他对象依赖了 A，那么则可以从 earlySingletonObjects 中得到 A 原始对象的代理对象了，并且是A的同一个代理对象。

当 B 创建完了之后，A 继续进行生命周期，而 A 在完成属性注入后，会按照它本身的逻辑去进行AOP，而此时我们知道 A 原始对象已经经历过了 AOP ，所以对于 A 本身而言，不会再去进行 AOP了。

## 总结

总结一下生命周期

spring容器进行扫描->反射后封装成beanDefinition对象->放入beanDefinitionMap->遍历map->验证（是否单例、是否延迟加载、是否抽象）->推断构造方法->准备开始进行实例->去单例池（一级缓存）中查，没有->去二级缓存中找，没有提前暴露->生成一个objectFactory对象暴露到二级缓存中->属性注入，发现依赖Y->此时Y开始它的生命周期直到属性注入，发现依赖X->X又走一遍生命周期，当走到去二级缓存中找的时候找到了->往Y中注入X的objectFactory对象->完成循环依赖。

## 问题

1、为什么要使用X的objectFacory对象而不是直接使用X对象？

利于拓展，程序员可以通过beanPostProcess接口操作objectFactory对象生成自己想要的对象

2、是不是只能支持单例(scope=singleton)而不支持原型(scope=prototype)？

是。因为单例是spring在启动时进行bean加载放入单例池中，在依赖的bean开始生命周期后，可以直接从二级缓存中取到它所依赖的bean的objectFactory对象从而结束循环依赖。而原型只有在用到时才会走生命周期流程，但是原型不存在一个已经实例化好的bean，所以会无限的创建->依赖->创建->依赖->...。

3、循环依赖是不是只支持非构造方法？

是。类似死锁问题



# Spring中的设计模式

- 工厂模式：BeanFactory、FactoryBean
	- FactoryBean是一种bean，在Spring内部广泛使用，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean，它的实现与设计模式中的工厂模式和修饰器模式类似 

- 代理模式：AOP
	- 事务管理中使用

- 模版模式：JdbcTemplate

- 观察者模式：事件监听机制
- 适配器模式：AdvisorAdapter接口，对Advisor进行了适配
- 访问者模式：PropertyAccessor接口，属性访问，用来访问和设置某个对象的某个属性
- 装饰器模式：BeanWrapper
- 策略模式：InstantiationStrategy，根据不同的情况进行实例化



# Spring MVC原理

MVC 是一种软件架构的思想，将软件按照模型、视图、控制器（MVC）来划分，围绕前端控制器 DispatcherServlet 来设计的。

## 一、MVC的工作流程

1. 浏览器发送请求至前端控制器 DispatcherServlet
2. DispatcherServlet收到请求，调用 HandlerMapping 处理器映射器
3. HandlerMapping 找到具体的处理器（根据xml配置、注解进行查找），生成处理器及处理器拦截器（如果有则生成）一并返回给 DispatcherServlet
4. DispatcherServlet 调⽤ HandlerAdapter 处理器适配器。 
5. HandlerAdapter 经过适配调⽤具体的处理器(Controller，也叫后端控制器) 
6. Controller 执⾏完成返回 ModelAndView。 
7. HandlerAdapter 将 controller 执⾏结果 ModelAndView 返回给 DispatcherServlet。
8. DispatcherServlet 将 ModelAndView 传给 ViewReslover 视图解析器。 
9. ViewReslover 解析后返回具体 View。 
10. DispatcherServlet 根据 View 进⾏渲染视图（即将模型数据填充⾄视图中）
4. DispatcherServlet 响应⽤户

> 浏览器访问、映射到前端控制器、读取配置文件、组件扫描找到控制器、根据mapping转发、控制器返回、视图解析、视图渲染

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203251523555.png" alt="image-20220325152345234" />



## 二、主要组件

Handler：也就是处理器。它直接应对着MVC中的C也就是Controller层，它的具体表现形式有很多，可以是类，也可以是方法。在Controller层中@RequestMapping标注的所有方法都可以看成是一个Handler，只要可以实际处理请求就可以是Handler

### 1、 HandlerMapping

initHandlerMappings(context)，处理器映射器，**根据用户请求的资源uri来查找Handler的**。

在SpringMVC中会有很多请求，每个请求都需要一个Handler处理，具体接收到一个请求之后使用哪个Handler进行，这就是HandlerMapping需要做的事。

### 2、HandlerAdapter

initHandler Adapters(context)，适配器。**调用 Handler 处理请求**。

因为SpringMVC中的Handler可以是任意的形式，只要能处理请求就ok，但是Servlet需要的处理方法的结构却是固定的，都是以request和response为参数的方法。

如何让固定的Servlet处理方法调用灵活的Handler来进行处理呢？这就是HandlerAdapter要做的事情。

> Handler是用来干活的工具；
>
> HandlerMapping用于根据需要干的活找到相应的工具；
>
> HandlerAdapter是**使用工具干活的人**。

### 3、HandlerExceptionResolver

initHandlerExceptionResolvers(context)，其它组件都是用来干活的。在干活的过程中难免会出现问题，出问题后怎么办呢？

这就需要有一个专门的角色**对异常情况进行处理**，在SpringMVC中就是HandlerExceptionResolver。具体来说，此组件的作用是根据异常设置ModelAndView，之后再交给render方法进行渲染。

### 4、ViewResolver

initViewResolvers(context)， ViewResolver用来**将String类型的视图名和Locale解析为View类型的视图**。

View是用来渲染页面的，也就是将程序返回的参数填入模板里，生成html（也可能是其它类型）文件。

这里就有两个关键问题：使用哪个模板？用什么技术（规则）填入参数？

这就是ViewResolver主要要做的工作，ViewResolver需要找到渲染所用的模板和所用的技术（也就是视图的类型）进行渲染，具体的渲染过程则交由不同的视图自己完成。

### 5、RequestToViewNameTranslator

initRequestToViewNameTranslator(context)， ViewResolver是根据ViewName查找View，但有的Handler处理完后并没有设置View也没有设置ViewName，这时就需要从request获取ViewName了。

如何**从request中获取ViewName**就是RequestToViewNameTranslator要做的事情了。

RequestToViewNameTranslator在Spring MVC容器里**只可以配置一个**，所以所有request到ViewName的转换规则**都要在一个Translator里面全部实现**。

### 6、LocaleResolver

initLocaleResolver(context)，解析视图需要两个参数：一是视图名，另一个是Locale。视图名是处理器返回的，Locale是从哪里来的？这就是LocaleResolver要做的事情。 

LocaleResolver用于**从request解析出Locale**， Locale就是zh-cn之类，表示一个区域，有了这个就可以对不同区域的用户显示不同的结果。

SpringMVC主要有两个地方用到了Locale：一是ViewResolver视图解析的时候；二是用到国际化资源或者主题的时候。

### 7、ThemeResolver

initThemeResolver(context)，用于**解析主题**。SpringMVC中一个主题对应一个properties文件，里面存放着跟当前主题相关的所有资源、如图片、css样式等。

SpringMVC的主题也支持国际化，同一个主题不同区域也可以显示不同的风格。SpringiMVC中跟主题相关的类有 ThemeResolver、ThemeSource和Theme。

主题是通过一系列资源来具体体现的，要得到一个主题的资源，首先要得到资源的名称，这
是ThemeResolver的工作。然后通过主题名称找到对应的主题（可以理解为一个配置）文件，这是ThemeSource的工作。最后从主题中获取资源就可以了。

### 8、MultipartResolver

initMultipartResolver(context)，用于**处理上传请求**。

处理方法是将普通的request包装成MultipartHttpServletRequest， 后者可以直接调用getFile方法获取File，如果上传多个文件，还可以调用getFileMap得到FileName->File结构的Map。

此组件中一共有三个方法，作用分别是判断是不是上传请求、将request包装成MultipartHttpServletRequest、处理完后清理上传过程中产生的临时资源。

### 9、FlashMapManager

initFlashMapManager(context)，用来管理FlashMap的，FlashMap主要用在redirect中传递参数。

### 10、HandlerInterceptor

Spring MVC 的拦截器（Interceptor）与 Java Servlet 的过滤器（Filter）类似，它主要用于拦截用户的请求并做相应的处理，通常应用在权限验证、记录请求信息的日志、判断用户是否登录等功能上。

[拦截器](https://blog.csdn.net/qq_45515182/article/details/124739633?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-124739633-blog-123307319.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-124739633-blog-123307319.pc_relevant_aa&utm_relevant_index=1)

# SpringBoot

## 启动流程

### 一、SpringBoot启动的时候，会构造一个SpringApplication的实例

构造SpringApplication的时候会进行初始化的工作，初始化的时候会做以下几件事：

1、把参数sources设置到SpringApplication属性中，这个sources可以是任何类型的参数.

2、判断是否是web程序，并设置到webEnvironment的boolean属性中.

3、创建并初始化ApplicationInitializer，设置到initializers属性中 。

4、创建并初始化ApplicationListener，设置到listeners属性中 。

5、初始化主类mainApplicatioClass。

### 二、SpringApplication构造完成之后调用run方法，

启动SpringApplication，run方法执行的时候会做以下几件事：

1、构造一个StopWatch计时器，用来记录SpringBoot的启动时间 。

2、初始化监听器，获取SpringApplicationRunListeners并启动监听，用于监听run方法的执行。

3、创建并初始化ApplicationArguments,获取run方法传递的args参数。

4、创建并初始化ConfigurableEnvironment（环境配置）。封装main方法的参数，初始化参数，写入到 Environment中，发布 ApplicationEnvironmentPreparedEvent（环境事件），做一些绑定后返回Environment。

5、打印banner和版本。

6、构造Spring容器(ApplicationContext)上下文。先填充Environment环境和设置的参数，如果application有设置beanNameGenerator（bean）、resourceLoader（加载器）就将其注入到上下文中。调用初始化的切面，发布ApplicationContextInitializedEvent（上下文初始化）事件。

7、SpringApplicationRunListeners发布finish事件。

8、StopWatch计时器停止计时，日志打印总共启动的时间。

9、发布SpringBoot程序已启动事件(started())

10、调用ApplicationRunner和CommandLineRunner

11、最后发布就绪事件ApplicationReadyEvent，标志着SpringBoot可以处理就收的请求了(running())



## 一、自动配置原理

@Import + @Configuration + Spring spi

自动配置类由各个starter提供，使用@Configuration + @Bean**定义配置类**，放到META-
INF/spring.factories 下

使用Spring spi**扫描**META-INF/spring.factories 下的配置类

使用@lmport**导入自动配置类**

### Note：SPI

#### 1、spi是什么？

SPI（service provider interface）机制是JDK内置的一种**服务发现机制**，可以**动态的发现服务**，即服务提供商，它通过在ClassPath路径下的META-INF/services文件夹查找文件，自动加载文件里所定义的类。目前这种大部分都利用SPI的机制进行服务提供，比如：dubbo、spring、JDBC等;

#### 2、spi解决了什么问题？

由于classLoader加载类的时候采用是【双亲委托模式】，意思是：首先委托父类去加载器获取，若父类加载器存在则直接返回，若加载器无法完成此加载任务，自己才去加载。该加载存在的弊端就是**上层的类加载永远无法加载下层的类加载器所加载的类，所以通过spi解决了该问题。**

spi是一种将服务接口与服务实现分离以达到解耦、大大提升了程序可扩展性的机制。引入服务提供者就是引入了spi接口的实现者，通过本地的注册发现获取到具体的实现类，轻松可插拔spi实现了动态加载，插件化，

#### 3、弊端

资源浪费:由于 spi 是通过循环加载实现类，会导致所有的类全部一起加载！


## 二、SpringBoot的Starter

starter 就是定义了一个 starter 的 jar 包，写一个 @Configuration 配置类，将这些 bean 定义在里面，然后在 starter 包的 META-INF/spring.factories 中写入该配置类，springboot 会按照约定来加载该配置类。

开发者只需要导入相应的 starter 依赖包，比如 mybatis-spring-boot-starter



## 三、嵌入式服务器

不需要下载 tomcat，springboot 已经内置了 tomcat.jar，运行 main 方法时会去启动 tomcat，并利用 tomcat 的 spi 机制加载 springmvc

### 启动流程

1. springboot 启动的时候会创建一个 spring 容器
2. 在创建 spring 容器的过程中，会利用 @ConditionalOnClass 技术来判断当前 classpath 是否存在 tomcat 依赖，若存在则会生成一个启动 tomcat 的 bean
3. Spring 容器创建完之后，会获取启动 tomcat 的 bean，并创建 tomcat 对象，并绑定端口等，然后启动 tomcat。

## 四、[常用注解](./spring-common-annotations.md)





## 五、配置文件的加载

**bootstrap.yml** (bootstrap.properties) **先加载** 

- bootstrap.yml 用于应用程序上下文的引导阶段。

- bootstrap.yml 由父Spring ApplicationContext加载。 

- 父ApplicationContext 被加载到使用 application.yml 的之前。
- 定义系统级别的参数

**application.yml** (application.properties) **后加载** 

- 如果application里写了`spring.profiles.active=dev`，还回去加载application-dev.properties
- 其中定义应用级别的配置

### 优先级从高到低

1、命令行参数。所有的配置都可以在命令行上进行指定。

2、Java系统属性（`System.getProperties()`）

3、操作系统环境变量。

4、properties和yml

5、@configuration注解类上的@PropertySource



## 六、读取配置文件的方法

```properties
profile.name=Spring Boot Profile
profile.desc=Spring Boot Profile Desc.
```

#### 1、@Value读取单个配置项

```java
@Value("${profile.name}")
private String name;

```

#### 2、@ConfigurationProperties 加实体类读取一组配置项

prefix 表示读取一组配置项的根 name，相当于 Java 中的类名，最后再把此配置类，注入到某一个类中就可以使用了

```java
@ConfigurationProperties(prefix = "profile")
@Data
public class Profile {
    private String name;
    private String desc;
}
```

#### 3、@PropertySource 注解可以用来指定读取某个配置文件

比如指定读取 application.properties 配置文件的配置内容

```java
@SpringBootApplication
@PropertySource("classpath:application.properties")
public class DemoApplication implements InitializingBean {
    @Value("${profile.name}")
    private String name;
 
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
 
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Name：" + name);
    }
}
```

#### 4、原生方式

```java
@SpringBootApplication
public class DemoApplication implements InitializingBean {
 
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
 
    @Override
    public void afterPropertiesSet() throws Exception {
        Properties props = new Properties();
        try {
            InputStreamReader inputStreamReader = new InputStreamReader(
                    this.getClass().getClassLoader().getResourceAsStream("application.properties"),
                    StandardCharsets.UTF_8);
            props.load(inputStreamReader);
        } catch (IOException e1) {
            System.out.println(e1);
        }
        System.out.println("Properties Name：" + props.getProperty("profile.name"));
    }
}
```





# 替换Spring容器中已经存在的Bean

https://blog.csdn.net/fu_huo_1993/article/details/124313557

在系统中根据 @Bean或通过 @Component 定义的Bean对象在Spring中都会转换成一个个的BeanDefinition对象，如果我们在Spring创建这些对象加入到Spring容器之前，将不想要的BeanDefinition对象删除，而加入我们自己想要的BeanDefinition对象是不是就可以实现了？而Spring提供的BeanDefinitionRegistryPostProcessor接口正好可以帮助我们实现这个功能。

BeanDefinitionRegistryPostProcessor 是在系统加载完所有的BeanDefinition对象来进行回调。



# 拦截器和过滤器

https://zhuanlan.zhihu.com/p/157715642

1. **实现原理**：拦截器是基于java的反射机制（动态代理）的，而过滤器是基于函数回调。　
2. **使用范围**：过滤器实现的是 javax.servlet.Filter 接口，而这个接口是在Servlet规范中定义的，也就是说**过滤器Filter的使用要依赖于Tomcat等容器**，导致它只能在web程序中使用。而**拦截器**(Interceptor) 它是一个Spring组件，并由Spring容器管理，**并不依赖Tomcat等容器，是可以单独使用的**。不仅能应用在web程序中，也可以用于Application、Swing等程序中。　　
3. 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
4. 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。　
5. **在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。**
6. 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。
7. **触发时机**：过滤器Filter是在请求进入容器后，但在进入servlet之前进行预处理，请求结束是在servlet处理完以后。拦截器 Interceptor 是在请求进入servlet后，在进入Controller之前进行预处理的，Controller 中渲染了对应的视图之后请求结束。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202210121114024.webp" alt="img" style="zoom:67%;" />
