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

<!-- 同一消费者可以同时绑定多个 JmqListener，
以实现不同的 topic 使用不同的 messageListener -->
<!--    配置Consumer，
messageListener bean需要实现com.jd.jmq.client.consumer.MessageListener接口，
在onMessage方法中接收消息。-->
<jmq:consumer id="consumer" transport="jmq.transport" autoStart="true">
    <!-- mq1 -->
    <jmq:listener topic="topic1" listener="topic1Listener"/>
    <!-- mq2 -->
    <jmq:listener topic="topic2" listener="topic2Listener"/>
</jmq:consumer>

```

