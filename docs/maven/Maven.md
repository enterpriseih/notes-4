# Maven是什么

Maven 本身的产品定位是一款『**项目**管理工具』，包含『**构建**管理』和『**依赖**管理』。

『项目管理』的角度来看，Maven 提供了如下这些功能：

- 项目对象模型（POM）：将整个项目本身抽象、封装为应用程序中的一个对象，以便于管理和操作。
- 全局性构建逻辑重用：Maven 对整个构建过程进行封装之后，程序员只需要指定配置信息即可完成构建。让构建过程从 Ant 的『编程式』升级到了 Maven 的『声明式』。
- 构件的标准集合：在 Maven 提供的标准框架体系内，所有的构件都可以按照统一的规范生成和使用。
- 构件关系定义：Maven 定义了构件之间的三种基本关系，让大型应用系统可以使用 Maven 来进行管理
	- 继承关系：通过从上到下的继承关系，将各个子构件中的重复信息提取到父构件中统一管理
	- 聚合关系：将多个构件聚合为一个整体，便于统一操作
	- 依赖关系：Maven 定义了依赖的范围、依赖的传递、依赖的排除、版本仲裁机制等一系列规范和标准，让大型项目可以有序容纳数百甚至更多依赖
- 插件目标系统：Maven 核心程序定义抽象的生命周期，然后将插件的目标绑定到生命周期中的特定阶段，实现了标准和具体实现解耦合，让 Maven 程序极具扩展性
- 项目描述信息的维护：我们不仅可以在 POM 中声明项目描述信息，更可以将整个项目相关信息收集起来生成 HTML 页面组成的一个可以直接访问的站点。这些项目描述信息包括：
	- 公司或组织信息
	- 项目许可证
	- 开发成员信息
	- issue 管理信息
	- SCM 信息

# Maven的三大坐标

- groupId：公司或组织域名的倒序，通常也会加上项目名称
  - 例如：com.atguigu.maven
- artifactId：模块的名称，将来作为 Maven 工程的工程名
- version：模块的版本号，根据自己的需要设定
  - 例如：SNAPSHOT 表示快照版本，正在迭代过程中，不稳定的版本
  - 例如：RELEASE 表示正式版本

# 配置

[下载地址](https://maven.apache.org/download.cgi)

解压至无中文目录下

Maven 的核心配置文件：**conf/settings.xml**

## 1、指定本地仓库

```xml
<!-- localRepository
| The path to the local repository maven will use to store artifacts.
|
| Default: ${user.home}/.m2/repository
<localRepository>/path/to/local/repo</localRepository>
-->
<localRepository>D:\maven-repository</localRepository>
```

**记住**：一定要把 localRepository 标签**从注释中拿出来**。

**注意**：本地仓库本身也需要使用一个**非中文、没有空格**的目录。

## 2、镜像文件

原有的注释掉，加入下面的

```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```



## 3、基础JDK版本

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```



# POM的层次

单继承关系

`当前pom` -继承-> `父pom` -继承-> `超级pom`

- 超级 POM：所有 POM 默认继承，只是有直接和间接之分。
- 父 POM：这一层可能没有，可能有一层，也可能有很多层。
- 当前 pom.xml 配置的 POM：我们最多关注和最多使用的一层。

> 有效 POM：隐含的一层，但是实际上真正生效的一层。
>
> 查看：mvn help:effective-pom



# 基础概念

## 1、依赖范围

**compile**

- 通常使用的第三方框架的 jar 包这样在项目实际运行时真正要用到的 jar 包都是以 compile 范围进行依赖的。比如 SSM 框架所需jar包。

**test**

- 测试过程中使用的 jar 包，以 test 范围依赖进来。比如 junit。

**provided**

- 在开发过程中需要用到的“服务器上的 jar 包”通常以 provided 范围依赖进来。
- 比如 servlet-api、jsp-api。而这个范围的 jar 包之所以不参与部署、不放进 war 包，就是避免和服务器上已有的同类 jar 包产生冲突，同时减轻服务器的负担。说白了就是：“**服务器上已经有了，你就别带啦！**”

**import**

管理依赖最基本的办法是继承父工程，但是和 Java 类一样，Maven 也是单继承的。如果不同体系的依赖信息封装在不同 POM 中了，没办法继承多个父工程怎么办？这时就可以使用 import 依赖范围。

典型案例是在项目中引入 SpringBoot、SpringCloud 依赖。

import 依赖范围使用要求：

- 打包类型必须是 pom
- 必须放在 dependencyManagement 中

**system**

以 Windows 系统环境下开发为例，假设现在 D:\tempare\atguigu-maven-test-aaa-1.0-SNAPSHOT.jar 想要引入到我们的项目中，此时我们就可以将依赖配置为 system 范围。

但是很明显：这样引入依赖完全不具有可移植性，所以**不要使用**。如果需要引入体系外 jar 包后面会有专门的办法。

**runtime**

专门用于编译时不需要，但是运行时需要的 jar 包。比如：编译时我们根据接口调用方法，但是实际运行时需要的是接口的实现类。典型案例是：

```xml
<!--热部署 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```



## 2、依赖传递

在 A 依赖 B，B 依赖 C 的前提下，C 是否能够传递到 A，取决于 B 依赖 C 时使用的依赖范围。

- B 依赖 C 时使用 compile 范围：可以传递
- B 依赖 C 时使用 test 或 provided 范围：不能传递，所以需要这样的 jar 包时，就必须在需要的地方明确配置依赖才可以。

## 3、依赖排除

依赖传递的时候，A 依赖 B 和 C，B 和 C 依赖的 jar 版本不一致，会导致冲突，需要在 A 和 B 之间设置阻断 X-1.0.jar，**意思是不让 X-1.0.jar 传递到 A**，此时不影响 B 和 C 之间原本的依赖关系。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202203291311262.png" alt="img027.2faff879"  />

所以配置依赖的排除其实就是阻止某些 jar 包的传递。因为这样的 jar 包传递过来会和其他 jar 包冲突。



```xml
pro02-maven-web -> pro01-maven-java -> spring-core -> commons-logging 
在pro02的pom.xml的依赖里排除掉commns-logging

<dependency>
	<groupId>com.yienx.maven</groupId>
	<artifactId>pro01-maven-java</artifactId>
	<version>1.0-SNAPSHOT</version>
	<scope>compile</scope>
	<!-- 使用excludes标签配置依赖的排除	-->
	<exclusions>
		<!-- 在exclude标签中配置一个具体的排除 -->
		<exclusion>
			<!-- 指定要排除的依赖的坐标（不需要写version） -->
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

> 后续会有专题讲解 jar 包冲突

## 4、继承

Maven工程之间，A 工程继承 B 工程

- B 工程：父工程
- A 工程：子工程

**本质上是 A 工程的 pom.xml 中的配置继承了 B 工程中 pom.xml 的配置**。

### 作用

在父工程中统一管理项目中的依赖信息，具体来说是管理依赖信息的版本。

它的背景是：

- 对一个比较大型的项目进行了模块拆分。
- 一个 project 下面，创建了很多个 module。
- 每一个 module 都需要配置自己的依赖信息。

它背后的需求是：

- 在每一个 module 中各自维护各自的依赖信息很容易发生出入，不易统一管理。
- 使用同一个框架内的不同 jar 包，它们应该是同一个版本，所以整个项目中使用的框架版本需要统一。
- 使用框架时所需要的 jar 包组合（或者说依赖信息组合）需要经过长期摸索和反复调试，最终确定一个可用组合。这个耗费很大精力总结出来的方案不应该在新的项目中重新摸索。

通过在父工程中为整个项目维护依赖信息的组合既**保证了整个项目使用规范、准确的 jar 包**；又能够将**以往的经验沉淀**下来，节约时间和精力。

**将所有版本信息统一在父工程中进行管理**，子工程配置的时候不写版本号，只要在父工程中可以找到就可以。

> **TIP**
>
> - 只有打包方式为 pom 的 Maven 工程能够管理其他 Maven 工程。打包方式为 pom 的 Maven 工程中不写业务代码，它是专门管理其他 Maven 工程的工程。
> - 如果子工程坐标中的 groupId 和 version 与父工程一致，那么可以省略
> - 使用 **dependencyManagement** 标签配置在父工程中对依赖进行管理
> - 子工程中也要配置依赖，只是**可以**省略版本号（也可以写）
> 	- 如果写了，且版本不同，会按照子工程的版本来执行

```xml
<!-- 通过自定义属性，统一指定Spring的版本 -->
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	
	<!-- 自定义标签，维护Spring版本数据 -->
	<atguigu.spring.version>4.3.6.RELEASE</atguigu.spring.version>
</properties>

<!-- 在依赖里配置的时候，可以用${} -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${atguigu.spring.version}</version>
</dependency>
```



## 5、聚合

部分组成整体

使用一个“总工程”将各个“模块工程”汇集起来，作为一个整体对应完整的项目。

### 优点

- 一键执行 Maven 命令：很多构建命令都可以在“总工程”中一键执行。

	以 mvn install 命令为例：Maven 要求有父工程时先安装父工程；有依赖的工程时，先安装被依赖的工程。我们自己考虑这些规则会很麻烦。但是工程聚合之后，在总工程执行 mvn install 可以一键完成安装，而且会自动按照正确的顺序执行。

- 配置聚合之后，各个模块工程会在总工程中展示一个列表，让项目中的各个模块一目了然。

## 6、可选依赖

```xml
<optional>true</optional>
```

其核心含义是：Project X 依赖 Project A，A 中一部分 X 用不到的代码依赖了 B，那么对 X 来说 B 就是『可有可无』的。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204191324834.png" alt="image-20220419132420155" style="zoom: 33%;" />

## 7、版本仲裁

### a>最短路径优先

对pro25来说，Maven 会采纳 1.2.12 版本

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204191325080.png" alt="image-20220419132520830" style="zoom:50%;" />

### b>路径相同时先声明者优先

此时 Maven 采纳哪个版本，取决于在 pro29-module-x 中，对 pro30-module-y 和 pro31-module-z 两个模块的依赖哪一个先声明。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204191326793.png" alt="image-20220419132650890" style="zoom: 33%;" />

# jar包冲突***

## 1、面对者

编订依赖列表的程序员。

## 2、表现形式

一般来说，由于我们自己编写代码、配置文件写错所导致的问题通常能够在异常信息中看到我们自己类的全类名或配置文件的所在路径。如果整个错误信息中完全没有我们负责的部分，全部是框架、第三方工具包里面的类报错，这往往就是 jar 包的问题所引起的。

而具体的表现形式中，主要体现为**找不到类**或**找不到方法**、**没报错但结果不对**

## 3、本质

- 同一 jar 包的不同版本
- 不同 jar 包中包含同名类

具体例子是有的同学在实际工作中遇到过：项目中部分模块使用 log4j 打印日志；其它模块使用 logback，编译运行都不会冲突，但是会引起日志服务降级，让你的 log 配置文件失效。比如：你指定了 error 级别输出，但是冲突就会导致 info、debug 都在输出。

## 4、解决方法

- 第一步：把彼此冲突的 jar 包找到
- 第二步：在冲突的 jar 包中选定一个。具体做法无非是通过 exclusions 排除依赖，或是明确声明依赖。

1. IDEA 的 Maven Helper 插件
2. Maven 的 enforcer 插件

# 外部 jar 包导入

使用的 jar 包不是 maven 导入的，maven 也没有。

- 将该 jar 包安装到 Maven 仓库

```
mvn install:install-file -Dfile=[体系外 jar 包路径] \
-DgroupId=[给体系外 jar 包强行设定坐标] \
-DartifactId=[给体系外 jar 包强行设定坐标] \
-Dversion=1 \
-Dpackage=jar
```

例如（Windows 系统下使用 ^ 符号换行；Linux 系统用 \）：

```
mvn install:install-file -Dfile=D:\idea2019workspace\atguigu-maven-outer\out\artifacts\atguigu_maven_outer\atguigu-maven-outer.jar ^
-DgroupId=com.atguigu.maven ^
-DartifactId=atguigu-maven-outer ^
-Dversion=1 ^
-Dpackaging=jar
```

查看本地仓库是否存在，存在就可以使用了。

 

# dependencyManagement和dependencies

> 父pom中依赖若是无法导入，先注释掉dependencyManagement，导入完成后再解除

Maven 使用dependencyManagement 元素来提供了一种管理依赖版本号的方式。 

通常会在一个组织或者项目的最顶层的父POM 中看到dependencyManagement 元素。

使用pom.xml 中的dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。 

Maven 会沿着父子层次向上走，直到找到一个拥有dependencyManagement 元素的项目，然后它就会使用这个 dependencyManagement 元素中指定的版本号。

这样做的好处就是：

- 如果有**多个子项目都引用同一样依赖**，则可以避免在每个使用的子项目里都声明一个版本号，这样当想升级或切换到另一个版本时，**只需要在顶层父容器里更新**，而不需要一个一个子项目的修改 ；
- 另外如果某个子项目需要另外的一个版本，只需要声明version就可。  

> dependencyManagement里只是声明依赖， **并不实现引入** ，因此子项目需要显示的声明需要用的依赖。 
