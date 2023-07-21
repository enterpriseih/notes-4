[CF中的参考文档](https://cf.jd.com/pages/viewpage.action?pageId=245185287)

# 调用方式

```xml
<jsf:consumer id="noticeService" interface="com.jd.up.portal.export.api.article.AritcleProviderApiJsfService"
 alias="${noticeRpc.jsf.alias}" timeout="2000" protocol="jsf" serialization="hessian" />

```





# 配置方式



**请注意：服务匹配是依靠 interface（接口） 以及 alias（别名）两个配置来完成的；**

也就是说调用端（Service Consumer）必须与服务端（Service Provider）的interface 与alias 属性配置必须完全一致，否则调用时会收到一个No Provider（没有服务提供者）的错误！



# 服务注册与订阅





