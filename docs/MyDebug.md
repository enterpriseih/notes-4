https://docs.qq.com/doc/DT2JPQUVvb015RHVB



### 缺少swagger2运行环境问题

`[Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException]`

原因，在boot2.6.x版本中，缺少swagger2运行环境，

解决方法：

不可行1、在swagger2点配置类上加上`@EnableWebMvc`

2、配置文件中加上

```
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```



### P76签名问题

医院模拟端发送的sign大家记得改写一下,ApiServiceImpl的103行HttpRequestHelper.getSign(paramMap, this.getSignKey())改成MD5.encrypt(this.getSignKey())，不然和后端比对的时候会一直报错



### 模板状态码的标准问题

utils/request.js 中

response 拦截器的状态码20000，改成200，这是说明，不是20000不能访问



