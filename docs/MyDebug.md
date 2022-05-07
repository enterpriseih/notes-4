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



### 模板状态码的标准问题

utils/request.js 中

response 拦截器的状态码20000，改成200，这是说明，不是20000不能访问



### Springboot2.2之后不自动给mongodb创建索引

但是2.6.7又不会报错了，可能是底层自行加入了

```yaml
// 第一步
//在application.properties文件中禁用自动索引创建
spring.data.mongodb.auto-index-creation=false
//或application.yml文件
spring:
  data:
    mongodb:
      auto-index-creation: false

```

自行建立一个配置类

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.event.EventListener;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.convert.MongoConverter;
import org.springframework.data.mongodb.core.index.MongoPersistentEntityIndexResolver;
import org.springframework.data.mongodb.core.mapping.BasicMongoPersistentEntity;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.MongoMappingContext;


@Slf4j
@RequiredArgsConstructor
@Configuration
public class MongoConfiguration {

private final MongoTemplate mongoTemplate;

private final MongoConverter mongoConverter;

@EventListener(ApplicationReadyEvent.class)
public void initIndicesAfterStartup() {
		
		log.info("Mongo InitIndicesAfterStartup init");
		Long init = System.currentTimeMillis();

		MappingContext mappingContext = this.mongoConverter.getMappingContext();

		if (mappingContext instanceof MongoMappingContext) {
			MongoMappingContext mongoMappingContext = (MongoMappingContext) mappingContext;
			for (BasicMongoPersistentEntity<?> persistentEntity : mongoMappingContext.getPersistentEntities()) {
				Class clazz = persistentEntity.getType();
				if (clazz.isAnnotationPresent(Document.class)) {
					MongoPersistentEntityIndexResolver resolver = new MongoPersistentEntityIndexResolver(mongoMappingContext);

					IndexOperations indexOps = mongoTemplate.indexOps(clazz);
					resolver.resolveIndexFor(clazz)
						.forEach(indexOps::ensureIndex);
				}
			}
		}

		log.info("Mongo InitIndicesAfterStartup take: {}",
			(System.currentTimeMillis() - init));
    }

}

```





### SpringCloudAlibaba和SpringBoot版本配对问题

见[版本配对设置](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)

