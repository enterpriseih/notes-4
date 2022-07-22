# Vue的目录结构

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204282214173.png" alt="image-20220428221413687" style="zoom: 50%;" />

```
src
├── api // 各种接口
├── assets // 图片等资源
├── components // 各种公共组件，非公共组件在各自 view 下维护 
├── icons //svg icon
├── router // 路由表
├── store // 存储
├── styles // 各种样式
├── utils // 公共工具，非公共工具，在各自 view 下维护 
├── views // 各种 layout，页面
├── App.vue //***项目顶层组件***
├── main.js //***项目入口文件***
└── permission.js //认证入口

```



# 基本结构

```html
<body>
    <script src="vue.min.js"></script>
    <div id="app">
        <!-- 插值表达式-->
        {{message}}
    </div>
    <script>
        new Vue({
            el:'#app',
            data: {
                message:'hello vue'
            }
        })
    </script>
</body>
```



# 基础语法

### 1、{{}} - 相当于innerText

### 2、v-bind:attr 绑定属性值，单向。

v-bind后的变量控制值

例如，v-bind:value - 绑定value值

简写：    :value

```html
<script language="JavaScript">
    window.onload=function () {
        var vue = new Vue({
            // key : value
            // key加不加""都可以
            "el":"#div0",
            "data":{
                msg:"hello world",
                msg2:'color:green;',
                uname:"张三丰"
            }
        });
    }
</script>

<div id="div0">
    <span>{{title}}</span>
    <input type="text" v-bind:value="msg">
    <input type="text" :value="msg">
    <div :style="msg2">单向绑定</div> 
</div>
```



### 3、v-model 双向绑定

不仅变量控制值，值也会改变变量

v-model:value   , 简写  **v-model**

```html
<script>
    window.onload=function () {
        var vue = new Vue({
            // key : value
            // key加不加""都可以
            "el":"#div0",
            "data":{
                msg:"hello world",
                uname:"张三丰"
            }
        });
    }
</script>

<div id="div0">
    <span>{{msg}}</span><br>
    <!-- <input type="text" v-bind:value="uname"> -->
    <!--
        v-model指的是双向绑定，
        也就是说之前的v-bind是通过msg这个变量的值来控制input输入框
        现在 v-model 不仅msg来控制input输入框，
		反过来，input输入框的内容也会改变msg的值
     -->
    <!-- <input type="text" v-model:value="msg"/> -->
    <!-- v-model:value 中 :value可以省略，直接写成v-model -->
    <!-- trim可以去除首尾空格 -->
    <input type="text" v-model.trim="msg">
</div>

```



### 4、 v-if , v-else , v-show

v-if和v-else之间不能有其他的节点

v-show是通过样式表display来控制节点是否显示

```html
<div id="app">
    <input type="checkbox" v-model="ok"/>
    <br/>
    <!-- true表示选中，false未选中 -->
    <div v-if="ok">选中了</div>
    <div v-else>没有选中</div>
    <!-- 如果成立，则显示一个blueviolet的方形 -->
    <div v-show="num%2==0" 
         style="width:200px;height: 200px;background-color: blueviolet;">
        &nbsp
    </div>
</div>
<script src="vue.min.js"></script>
<script>
    new Vue({
        el: '#app',
        data: {
            ok:false,
            num:2
        }
    })
</script>

```



### 5、 v-for 迭代

`v-for={fruit in fruitList}:key="fruit"`

加上 key 会提高效率

```html
<script>
    window.onload=function () {
        var vue = new Vue({
            // key : value
            // key加不加""都可以
            el:"#div0",
            data:{
                fruitList:[
                    {fname:"苹果",price:5,fcount:100,remark:"好吃"},
                    {fname:"桃子",price:3,fcount:100,remark:"好吃"},
                    {fname:"菠萝",price:4,fcount:100,remark:"好吃"},
                    {fname:"香蕉",price:2,fcount:100,remark:"好吃"}
                ]
            }
        });
    }
</script>

<div id="div0">
    <table>
        <tr>
            <th>名称</th>
            <th>单价</th>
            <th>库存</th>
            <th>备注</th>
        </tr>
        <tr v-for="fruit in fruitList":key="fruit" align="center">
            <td>{{fruit.fname}}</td>
            <td>{{fruit.price}}</td>
            <td>{{fruit.fcount}}</td>
            <td>{{fruit.remark}}</td>
        </tr>
    </table>
</div>
```



### 6、 v-on 绑定事件

v-on:click="方法名" ->简写-> @click="方法名"

trim:去除首尾空格 , split() , join()

```html
<script language="JavaScript">
    window.onload=function () {
        var vue = new Vue({
            // key : value
            // key加不加""都可以
            el:"#div0",
            data:{
                msg:"hello world"
            },
            methods:{
                myReverse:function () {
                    this.msg = this.msg.split("").reverse().join("");
                }
            }
        });
    }
</script>

<div id="div0">
    <span>{{msg}}</span><br>
    <!-- v-on:click 表示绑定点击事件 -->
    <!-- v-on可以省略，变成 @click -->
    <!--<input type="button" value="反转" v-on:click="myReverse"/>-->
    <input type="button" value="反转" @click="myReverse">
</div>

```



### 7、watch侦听属性

```html
<script type="text/javascript">
    window.onload=function(){
        var vue = new Vue({
            el:"#div0",
            data:{
                num1:1,
                num2:2,
                num3:3
            },
            watch:{
                // 侦听属性num1和num2
                // 当num1的值有改动时，那么需要调用后面定义的方法 , 
                // newValue指的是num1的新值
                num1:function(newValue){
                    this.num3 = parseInt(newValue) + parseInt(this.num2);
                },
                num2:function(newValue){
                    this.num3 = parseInt(this.num1) + parseInt(newValue) ;
                }
            }
        });
    }
</script>

<div id="div0">
    <input type="text" v-model="num1" size="2"/>
    +
    <input type="text" v-model="num2" size="2"/>
    =
    <span>{{num3}}</span>
</div>

```



### 8、生命周期

```html
<script>
    window.onload=function(){
        var vue = new Vue({
            "el":"#div0",
            data:{
                msg:1
            },
            methods:{
                changeMsg:function(){
                    this.msg = this.msg + 1 ;
                }

            },
            /*vue对象创建之前*/
            beforeCreate:function(){
                console.log("beforeCreate:vue对象创建之前---------------");
                console.log("msg:"+this.msg);
            },
            /*vue对象创建之后(在页面渲染之前执行)*/
            created:function(){
                debugger
                // 调试 debugger
                console.log("created:vue对象创建之后---------------");
                console.log("msg:"+this.msg);
            },
            /*数据装载之前*/
            beforeMount:function(){
                console.log("beforeMount:数据装载之前---------------");
                console.log("span:"+document.getElementById("span").innerText);
            },
            /*数据装载之后(在页面渲染之后执行)*/
            mounted:function(){
                console.log("mounted:数据装载之后---------------");
                console.log("span:"+document.getElementById("span").innerText);
            },
            beforeUpdate:function(){
                console.log("beforeUpdate:数据更新之前---------------");
                console.log("msg:"+this.msg);
                console.log("span:"+document.getElementById("span").innerText);
            },
            updated:function(){
                console.log("Updated:数据更新之后---------------");
                console.log("msg:"+this.msg);
                console.log("span:"+document.getElementById("span").innerText);
            }
        });
    }
</script>

<div id="div0">
    <span id="span">{{msg}}</span><br/>
    <input type="button" value="改变msg的值" @click="changeMsg"/>
</div>

```



# 路由

1. 理解： 一个路由（route）就是一组映射关系（key - value），多个路由需要路由器（router）进行管理。
2. 前端路由：**key是路径**，**value是组件**。

### 1、基本使用

编写 router 配置项

```js
//引入VueRouter
import VueRouter from 'vue-router'
//引入Luyou 组件
import About from '../components/About'
import Home from '../components/Home'

//创建router实例对象，去管理一组一组的路由规则
const router = new VueRouter({
	routes:[
		{
			path:'/about',
			component:About
		},
		{
			path:'/home',
			component:Home
		}
	]
})

//暴露router
export default router
```

实现切换（active-class可配置高亮样式）

```vue
<router-link active-class="active" to="/about">About</router-link>
```

指定展示位置

```vue
<router-view></router-view>
```



### 2、几个注意点

1. 路由组件通常存放在```pages```文件夹，一般组件通常存放在```components```文件夹。
2. 通过切换，“隐藏”了的路由组件，默认是被销毁掉的，需要的时候再去挂载。
3. 每个组件都有自己的```$route```属性，里面存储着自己的路由信息。
4. 整个应用只有一个router，可以通过组件的```$router```属性获取到。



### 3、多级路由

1. 配置路由规则，使用children配置项：

	```js
	routes:[
		{
			path:'/about',
			component:About,
		},
		{
			path:'/home',
			component:Home,
			children:[ //通过children配置子级路由
				{
					path:'news', //此处一定不要写：/news
					component:News
				},
				{
					path:'message',//此处一定不要写：/message
					component:Message
				}
			]
		}
	]
	```

2. 跳转（要写完整路径）：

	```vue
	<router-link to="/home/news">News</router-link>
	```

3. 指定展示位置

	```vue
	<router-view></router-view>
	```




### 3、命名路由

简化路由的跳转

给路由命名：

```js
{
	path:'/demo',
	component:Demo,
	children:[
        {
            path:'test',
            component:Test,
            children:[
                {
                    name:'hello' //给路由命名
                    path:'welcome',
                    component:Hello,
                }
            ]
        }
	]
}
```

简化跳转

```vue
<!--简化前，需要写完整的路径 -->
<router-link to="/demo/test/welcome">跳转</router-link>

<!--简化后，直接通过名字跳转 -->
<router-link :to="{name:'hello'}">跳转</router-link>

<!--简化写法配合传递参数 -->
<router-link 
	:to="{
		name:'hello',
		query:{
		   id:666,
            title:'你好'
		}
	}"
>跳转</router-link>
```



### 5、路由的query参数

传递参数

```vue
<!-- 跳转并携带query参数，to的字符串写法 -->
<router-link :to="/home/message/detail?id=666&title=你好">跳转</router-link>
				
<!-- 跳转并携带query参数，to的对象写法 -->
<router-link 
	:to="{
		path:'/home/message/detail',
		query:{
		   id:666,
           title:'你好'
		}
	}"
>跳转</router-link>
```

接收参数：

```js
$route.query.id
$route.query.title
```



### 6、路由的params参数

配置路由，声明接收params参数

```js
{
	path:'/home',
	component:Home,
	children:[
		{
			path:'news',
			component:News
		},
		{
			component:Message,
			children:[
				{
					name:'xiangqing',
					path:'detail/:id/:title', 
                    //使用占位符声明接收params参数
					component:Detail
				}
			]
		}
	]
}
```

传递参数

```vue
<!-- 跳转并携带params参数，to的字符串写法 -->
<router-link :to="/home/message/detail/666/你好">跳转</router-link>
				
<!-- 跳转并携带params参数，to的对象写法 -->
<router-link 
	:to="{
		name:'xiangqing',
		params:{
		   id:666,
           title:'你好'
		}
	}"
>跳转</router-link>
```

> 特别注意：路由携带params参数时，若使用to的对象写法，则不能使用path配置项，必须使用name配置！

接收参数：

```js
$route.params.id
$route.params.title
```



