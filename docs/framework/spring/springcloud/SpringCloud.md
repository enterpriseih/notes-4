# 简介

SpringCloud 是一套微服务技术的集合体

- 服务注册与发现
	- Eureka:x:
	- Zookeeper
	- Consul
	- **Nacos**

- 服务调用
	- **Ribbon**
	- LoadBalancer
	- 服务调用2
		- Feign:x:
		- **OpenFeign**

- 负载均衡
- 服务熔断和降级
	- Hystrix:x:
	- resilience4j
	- **sentienl**

- 服务消息队列

- 配置中心管理
	- Config:x:
	- apollo
	- **Nacos**

- 服务网关
	- Zuul:x:
	- **gateway**

- 服务总线
	- Bus:x:
	- **Nacos**
- 服务监控

- 全链路追踪

- 自动化构建部署

- 服务定时任务调度操作

# 版本选择

https://start.spring.io/actuator/info

[SpringCloud.H资料](https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/)

[SpringBoot2.2.2资料](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/)

# SpringCloud组件

Spring Cloud 在接口调用上，大致会经过如下几个组件配合:

调用者（消费者）—>

`接口化请求调用 —> Feign —> Hystrix —> Ribbon —> Http Client(apache http components 或者 Okhttp) `

—>被调用者（生产者）

(1)**接口化请求调用**：当调用被@FeignClient注解修饰的接口时，在框架内部，将请求转换成Feign的请求 实例feign.Request，交由Feign框架处理。

(2)**Feign**：转化请求Feign是一个http请求调用的轻量级框架，可以以Java接口注解的方式调用Http请求，封装了Http调用流程。

(3)**Hystrix熔断器**：熔断处理机制 Feign的调用关系，会被Hystrix代理拦截，对每一个Feign调用请 求，Hystrix都会将其包装成HystrixCommand,参与Hystrix的流控和熔断规则。如果请求判断需要熔断， 则Hystrix直接熔断，抛出异常或者使用FallbackFactory返回熔断 Fallback 结果;如果通过，则将调用请求传递给Ribbon组件。

(4)**Ribbon负载均衡**：服务地址选择，当请求传递到Ribbon之后,Ribbon会根据自身维护的服务列表，根据服务的服务质量，如平均响应时间，Load等，结合特定的规则，从列表中挑选合适的服务实例，选择好机器之后，然后将机器实例的信息请求传递给Http Client客户端，HttpClient客户端来执行真正的Http接口调用;

(5)**HttpClient**：Http客户端，真正执行Http调用根据上层Ribbon传递过来的请求，已经指定了服务地址，则HttpClient开始执行真正的Http请求



# 服务网关Gateway

不同的微服务一般会有不同的网络地址，而外部客户端可能需要调用多个服务的接口才能完成一个业务需求，如果让客户端直接与各个微服务通信会加重负载。

服务网关 = 路由转发 + 过滤器

网关的功能

- **统一进行认证和鉴权**

- **服务路由**、**负载均衡**

	配合服务注册与发现，网关对请求代理后，还可以把请求分发到运转正常的服务消费端

- 请求限流

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202205211159827.png" alt="未命名文件" style="zoom:67%;" />
