# 三大类

1.   servlet的过滤器Filter
2.   springmvc的intercepter：`HandlerIntercepter`
3.   spring aop：`MethodIntercepter`



# 区别

|                        |                                                              |
| ---------------------- | ------------------------------------------------------------ |
| servlet的过滤器Filter  | 可以拿到原始的http请求和响应的信息，但是拿不到真正处理这个请求的方法的信息 |
| springmvc的interceptor | 在Filter中是不知道这个请求是哪个控制器的哪个方法来处理的。如果你需要这个信息的话，那么需要使用springmvc的interceptor。 拦截器可以拿到原始的**http请求和响应的信息**，也能拿到真正处理这个请求的方法的信息，但是其拿不到这个方法被调用的时候真正调用的参数的值 |
| spring的切片Aspect     | 可以拿到方法被调用的时候真正调用的**参数的值**，但是拿不到原始的http请求和响应的信息 |

