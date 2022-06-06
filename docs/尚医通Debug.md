# P88

点击市区不显示

```javascript
// 需要在searchObj中定义一下cityCode
searchObj: {provinceCode:'', cityCode:''}, // 查询表单对象
```



# P90

获取路由中存放的医院的id

```js
const id = this.$route.params.id
```



路由跳转

```js
this.$router.push({ path: '/hospSet/hosp/list' })
```



前端hosname报错：hospital未加载成，vue就渲染完成了

在根标签加个!=null，就好了

```html
<div class="app-container" v-if="hospital!=null">
```



# P96

Cannot read properties of undefined (reading '0')

```
注意respongse里面的key是bookingScheduleRuleVoList
直接复制的前端中是bookingScheduleRuleList
所以会查不到
```



# P100

使用gateway后前端503

```properties
#发现是lb找不到，版本依赖的问题，改成端口就好了
#spring.cloud.gateway.routes[0].uri=lb://service-hosp
spring.cloud.gateway.routes[0].uri=http://localhost:8201
```

或者

```xml
<!-- 依赖中加上loadbalancer -->
<!-- 没找到依赖导致的 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



