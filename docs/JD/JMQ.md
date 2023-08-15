# 概念

底层封装kafka





# 配置jmq2



```xml
<!--    配置transport，每个transport实例对应一个APP，
同一APP如果生产和消费多个主题，只需要配置一个transport实例即可-->
<jmq:transport id="jmq.transport" address="${jmq.address}" user="${jmq.user}" 
               password="${jmq.password}" app="${jmq.consumer.app}"
               epoll="false" sendTimeout="1000" soTimeout="1000"/>

<jmq:producer id="producer" retryTimes="2" transport="jmq.transport"/>

<bean id="jmq2SpringProducerExample" class="com.jd.jmq.client.examples.jmq2.producer.Jmq2SpringProducerExample">
    <property name="topic" value="${jmq.topic}"/>
    <property name="producer" ref="producer"/>
</bean>

<!-- 同一消费者可以同时绑定多个 JmqListener，
以实现不同的 topic 使用不同的 messageListener -->
<!--    配置Consumer，
messageListener bean需要实现com.jd.jmq.client.consumer.MessageListener接口，
在onMessage方法中接收消息。-->
<jmq:consumer id="consumer" transport="jmq.transport">
    <jmq:listener topic="${jmq.topic}" listener="messageListener"/>
</jmq:consumer>
<bean id="messageListener" 
      class="com.jd.jmq.client.examples.jmq2.consumer.Jmq2SpringConsumerExample" 
      init-method="init"/>


```

