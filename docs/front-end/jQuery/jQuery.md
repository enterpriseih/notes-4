# 使用

引入本地，或cdn

```html
<script src="./js/jquery-3.4.1.js"></script>
```



# jQuery对象

```html
<!-- 
    Dom对象
        通过js方式获取的元素对象（document）
    Jquery对象
        通过jquery方法获取的元素对象，返回的是jquery包装集
 -->
<script>
	/*  Dom对象 */
    var divDom = document.getElementById("mydiv");
    console.log(divDom);
    var divsDom = document.getElementsByTagName("div");
    console.log(divsDom);
    // js获取不存在的元素
    var spanDom = document.getElementsByTagName("span");
    console.log(spanDom);
    var spanDom2 = document.getElementById("myspan");
    console.log(spanDom2);

    console.log("==========");


    /* Jquery对象 */
    // 通过id选择获取元素对象  $("#id属性值")
    var divJquery = $("#mydiv");
    console.log(divJquery);
    // jquery获取不存在的元素
    var spanJquery = $("#myspan");
    console.log(spanJquery);

    console.log("========");
    /* Dom对象 转 Jquery对象 */
    // Dom对象转为jQuery对象，只需要利用$()方法进行包装即可
    var divDomToJquery = $(divDom);
    console.log(divDomToJquery);

    /* Jquery对象 转 Dom对象 */
    // 获取包装集对象中指定下标的元素，将jquery对象转换成dom对象
    var divJqueryToDom = divJquery[0];
    console.log(divJqueryToDom);
</script>
```





# 选择器

## 基础选择器

```html
<!-- 
    基础选择器
        id选择器			#id属性值			$("#id属性值")				
		选择id为指定值的元素对象（如果有多个同名id，则以第一个为准）
        
		类选择器			.class属性值			$(".class属性值	")			
		选择class为指定值的元素对象
        
		元素选择器		标签名/元素名		$("标签名/元素名")			
		选择所有指定标签的元素对象

        通用选择器		*					$("*")						
		选择页面中所有的元素对象
        
		组合选择器		选择器1,选择器2,..	$("选择器1,选择器2,..")		
		选择指定选择器选中的元素对象
 -->
 
 <script>
    // id选择器		#id属性值	
    var mydiv = $("#mydiv1");
    console.log(mydiv);

    // 类选择器			.class属性值	
    var clas = $(".blue");
    console.log(clas);

    // 元素选择器		标签名/元素名
    var spans = $("span");
    console.log(spans);

    // 通用选择器		*	
    var all = $("*");
    console.log(all);

    // 组合选择器		选择器1,选择器2,..
    var group = $("#mydiv1,div,.blue");
    console.log(group);


 </script>
```



## 层次选择器

```html
<!-- 
    后代选择器
        格式： 父元素 指定元素 （空格隔开）
        示例： $("父元素 指定元素")
        选择父元素的所有指定元素（包含第一代、第二代等）
    子代选择器
        格式： 父元素 > 指定元素 （大于号隔开）
        示例： $("父元素 > 指定元素")
        选择父元素的所有第一代指定元素
    相邻选择器
        格式： 元素 + 指定元素 （加号隔开）
        示例： $("元素 + 指定元素")
        选择元素的下一个指定元素
        （只会查找下一个元素，如果元素不存在，则获取不到）
    同辈选择器
        格式： 元素 ~ 指定元素 （波浪号隔开）
        示例： $("元素 ~ 指定元素")
        选择元素的下面的所有指定元素
 -->
```



# 操作元素的属性

```html
<!-- 
    
    属性的分类：
        固有属性：元素本身就有的属性（id、name、class、style）
        返回值是boolean的属性：checked、selected、disabled
        自定义属性：用户自己定义的属性

    attr()和prop()的区别：
        1. 如果是固有属性，attr()和prop()方法均可操作
        2. 如果是自定义属性，attr()可操作，prop()不可操作
        3. 如果是返回值是boolean类型的属性
            若设置了属性，attr()返回具体的值，prop()返回true；
            若未设置属性，attr()返回undefined，prop()返回false；

    1. 获取属性
        attr("属性名")
        prop("属性名")
    2. 设置属性
        attr("属性名","属性值")
        prop("属性名","属性值")
    3. 移除属性
        removeAttr("属性名");

    总结：
        如果属性的类型是boolean（checked、selected、disabled），
        则使用prop()方法；否则使用attr()方法。

 -->
<input type="checkbox" name="ch" checked="checked" id="aa" abc="aabbcc"/>	aa
<input type="checkbox" name="ch" id="bb" />	bb
```



```js
/* 获取属性 */
// 固有属性
var name = $("#aa").attr("name");
console.log(name);
var name2 = $("#aa").prop("name");
console.log(name2);
// 返回值是boolean的属性（元素设置了属性）
var ck1 = $("#aa").attr("checked"); // checked
var ck2 = $("#aa").prop("checked"); // true
console.log(ck1);
console.log(ck2);
// 返回值是boolean的属性（元素未设置属性）
var ck3 = $("#bb").attr("checked"); // undefined
var ck4 = $("#bb").prop("checked"); // false
console.log(ck3);
console.log(ck4);
// 自定义属性
var abc1 = $("#aa").attr("abc"); // aabbcc
var abc2 = $("#aa").prop("abc"); // undefined
console.log(abc1);
console.log(abc2);

/* 设置属性 */
// 固有属性
$("#aa").attr("value","1");
$("#bb").prop("value","2");

// 返回值是boolean的属性
$("#bb").attr("checked","checked");
$("#bb").prop("checked",false);

// 自定义属性
$("#aa").attr("uname","admin");
$("#aa").prop("uage",18);

/* 移除属性 */
$("#aa").removeAttr("checked")
```



# 操作元素的样式

```html
<!-- 
操作元素的样式
    attr("class") 			 	
	获取元素的样式名
    
	attr("class","样式名")	 	
	设置元素的样式 （设置样式，原本的样式会被覆盖）
    
	addClass("样式名")			
	添加样式 （在原来的样式基础上添加样式，
	原本的样式会保留，如果出现相同样式，则以样式中后定义的为准）
    
	css()						
	添加具体的样式（添加行内样式）
    css("具体样式名","样式值");	 设置单个样式
	css({"具体样式名":"样式值","具体样式名":"样式值"});	设置多个样式
    
	removeClass("样式名")		
	移除样式
-->
```

