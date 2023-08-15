# 简介

JD的配置中心



# 使用

添加依赖

```xml
<dependency>
	<groupId>com.jd.laf.config</groupId>
	<artifactId>laf-config-client-jd-springboot-starter</artifactId>
	<version>${client.version}</version>
	<type>pom</type>
</dependency>
```

在properties中配置laf的url等内容

```properties
laf.config.manager.application=regular-purchase
laf.config.manager.parameters[0].name=autoListener
laf.config.manager.parameters[0].value=true
laf.config.manager.resources[0].name=regular_config
laf.config.manager.resources[0].uri
=ucc://regular-purchase:c483843d6e5b42e79704ebce598ace5f@ducc.jd.local/v1/namespace/regular_purchase/config/regular_config/profiles/production

```



创建配置类实例，以 jdbc 为例

```java
@Component
public class JdbcConfig {

	@Value("${jdbc.url}")
	private String url ;
	@Value("jdbc.userName")
	private String userName ;
	@Value("${jdbc.password}")
	private String password ;
	@Value("${jdbc.driverClass}")
	private String driverClass ;	
    
	//getter setter method 
}
```



<img src="./assets/image-20230710180912569.png" alt="image-20230710180912569" style="zoom:;" />

[注解说明](https://cf.jd.com/pages/viewpage.action?pageId=446932624#id-2Spring%E9%9B%86%E6%88%90%E4%BD%BF%E7%94%A8-4.2@LafValue%E6%B3%A8%E8%A7%A3%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E)

>   注：
>
>   @LafValue 放在方法上，则自动以该方法来绑定监听器赋值



配置样例

```java
public class TestConfig {
 
 	//示例1：通过Sping的@Value注解, 获取配置key1的value值，注： spring 支持的 @Value 支持占位符
 	@Value("${key1}") 
 	@LafUcc //示例2： 配置此注解，这个变量key1的值会动态变更, 默认为true开启
 	private String key1; 
	
 	// 示例3： 通过 DUCC 的 @LafValue 注解，
    // 获取配置 key1 的 value 值
 	// 注： ducc 的 @LafValue 不支持占位符
 	@LafValue("key1") //或 @LafValue(key="key1")
 	private String url1 ;
	
 	// 示例4： 通过 DUCC 的 @LafValue 注解，获取配置 key1 的 Property 对象
 	// 注： 示例3 和 示例4 注解配置没有区别，区别在于 变量类型， 
    // 示例4 是 Property 类型，DUCC SDK 会根据变量类型注入
 	@LafValue("key1")//或 @LafValue(key="key1")
    // Property是个键值对
    private Property property1 ;
	
 	// 示例5： 通过 DUCC 的 @LafValue 注解，获取配置源名称（ 对应 laf-config:resource 值）config1 的配置集合对象
 	@LafValue("config1") //或 @LafValue(name="config1") 
 	private Configuration config1 ;
	
 	// 示例6：在方法上使用 DUCC 的 @LafValue 注解， 
    // 监听一个指定的 key/value 数据变化。
    // 通常使用场景是：动态获取 DUCC 配置后需要做一些自定义处理，例如： 打印日志、格式转换
 	@LafValue("key3")//注解说明可查看上文 【@LafValue注解参数说明】
 	public void onPropertyChange(Property property){
 	....
 	}
	
 	// 实例7： 在方法上使用 DUCC 的 @LafValue 注解。 
    // 当配置集中任何一个 key/value 变化的时候，这里可以的监听可以获取到完整配置集。 
 	@LafValue(name="conf2")//注解说明可查看上文 【@LafValue注解参数说明】
 	public void onConfigurationChange(Configuration configuration){
 	....
 	}
 
}
```