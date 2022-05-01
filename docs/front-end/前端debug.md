# 跨域问题

`No 'Access-Control-Allow-Origin'`

跨域问题，三个地方，任何一个不相同都会产生跨域，不能访问

- 访问协议：http 访问 https
- 访问地址：192.128.1.1 访问 172.11.1.1
- 访问端口：9528 访问 8201

解决方式：

在Controller类上加注解`@CrossOrigin`

# 模板状态码的标准问题

utils/request.js 中

response 拦截器的状态码20000，改成200，这是说明，不是20000不能访问



