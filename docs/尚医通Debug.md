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

