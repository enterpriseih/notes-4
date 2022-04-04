- groupId：公司或组织域名的倒序，通常也会加上项目名称
	- 例如：com.atguigu.maven
- artifactId：模块的名称，将来作为 Maven 工程的工程名
- version：模块的版本号，根据自己的需要设定
	- 例如：SNAPSHOT 表示快照版本，正在迭代过程中，不稳定的版本
	- 例如：RELEASE 表示正式版本

# 基础概念

## 1、依赖范围

**compile**

- 通常使用的第三方框架的 jar 包这样在项目实际运行时真正要用到的 jar 包都是以 compile 范围进行依赖的。比如 SSM 框架所需jar包。

**test**

- 测试过程中使用的 jar 包，以 test 范围依赖进来。比如 junit。

**provided**

- 在开发过程中需要用到的“服务器上的 jar 包”通常以 provided 范围依赖进来。
- 比如 servlet-api、jsp-api。而这个范围的 jar 包之所以不参与部署、不放进 war 包，就是避免和服务器上已有的同类 jar 包产生冲突，同时减轻服务器的负担。说白了就是：“**服务器上已经有了，你就别带啦！**”

system/runtime/import

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
