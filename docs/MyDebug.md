https://docs.qq.com/doc/DT2JPQUVvb015RHVB

- 

`[Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException]`

原因，在boot2.6.x版本中，缺少swagger2运行环境，

不可行1、在swagger2点配置类上加上`@EnableWebMvc`

2、配置文件中加上

```
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```





P76

医院模拟端发送的sign大家记得改写一下,ApiServiceImpl的103行HttpRequestHelper.getSign(paramMap, this.getSignKey())改成MD5.encrypt(this.getSignKey())，不然和后端比对的时候会一直报错