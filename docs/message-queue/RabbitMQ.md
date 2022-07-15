# AMQP协议

## 简介

**AMQP协议简介**：

- AMQP全称：`Advanced Message Queuing Protocol` 高级消息队列协议。
- AMQP定义：是具有现代特征的二进制协议。是一个提供统一消息服务的应用层标准高级消息队列协议，是**应用层协议的一个开放标准，为面向消息的中间件设计**。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207121518726.jpg" alt="AMQP协议模型" style="zoom:67%;" />



## 核心概念

**AMQP概念**：

- **Server**：又称作Broker，接收客户端的连接，实现AMQP实体服务。
- **Connection**：连接，应用程序与Broker的网络连接， TCP/IP 三次握手和四次挥手。
- **Channel**：网络信道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道。客户端可以建立多个Channel，每个Channel代表一个会话任务。
- **Message**：消息。服务器和应用程序之间传送的数据，由Properties和Body组成。
	- Properties可以对消息进行修饰，比如消息的优先级、延迟等高级特性；
	- Body就是消息体内容。
- **Virtual Host**：虚拟主机，用于进行逻辑隔离，最上层的消息路由。
	- 一个Virtual Host里面可以有若干个Exchange和Queue；
	- 同一个Virtual Host里面不能有相同名称的Exchange和Queue。
- **Exchange**：交换机，接收消息。根据Routing Key转发消息到绑定的队列。
- **Binding**：Exchange和Queue之间的虚拟连接，Binding中可以包含Routing Key。
- **Routing Key**：一个路由规则，虚拟机可以用它来确定如何路由一个特点消息。
- **Queue**：也成为了Message Queue，消息队列，保存消息并转发给消费者。

# MQ消息流转

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207121519197.jpg" alt="2.6-1-RabbitMQ消息流转图" style="zoom:67%;" />



# RabbitMQ架构

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207121518968.jpg" alt="2.5-1-RabbitMQ的整体架构图" style="zoom:67%;" />



**Broker**：rabbitmq的服务节点

**Queue**：队列，是RabbitMQ的内部对象，用于存储消息。RabbitMQ中消息只能存储在队列中。**生产者投递消息到队列，消费者从队列中获取消息井消费**。多个消费者可以订阅同一个队列，这时队列中的消息会被平均分摊(轮询)给多个消费者进行消费，而不是每个消费者都收到所有的消息进行消费。(注意：RabbitMQ不支持队列层面的广播消费，如果需要广播消费，可以采用一个交换器通过路由Key绑定多个队列，由多个消费者来订阅这些队列的方式。

**Exchange**：交换器。生产者将消息发送到Exchange，由交换器根据Routing Key将消息路由到一个或多个队列中。如果路由不到，或返回给生产者，或直接丢弃，或做其它处理。

**Routingkey**：路由Key。生产者将消息发送给交换器的时候，一般会指定一个Routingkey，**用来指定这个消息的路由规则**。这个路由Key需要与交换器类型和绑定键(BindingKey)联合使用才能最终生效。在交换器类型和绑定键固定的情况下，生产者可以在发送消息给交换器时**通过指定Routingkey来决定消息流向哪里**。

**Binding**：通过绑定将交换器和队列关联起来，在绑定的时候一般会指定一个绑定键，这样RabbitiMQ就可以指定如何正确的路由到队列了。

> 交换器和队列实际上是多对多关系。就像关系数据库中的两张表。他们通过Bindingkey做关联(多对多关系表)。在投递消息时，可以通过Exchange和Routingkey(对应Bindingkey)就可以找到相对应的队列。

**Channel**：信道是建立在 Connection 之上的虛拟连接。当应用程序与Rabbit Broker建立TCP连接的时候，客户端紧接着可以创建一个 AMQP 信道(Channel)，每个信道都会被指派一个唯一的D。RabbitMQ 处理的每条 AMQP 指令都是通过信道完成的。

# 交换机

Exchange属性：

- **Name**：Exchange名称。
- **Type**：Exchange的类型。`direct、topic、fanout、headers`。
- **Durability**：是否需要持久化，true为持久化。false代表重启服务器后该交换机会被删除。
- **Auto Delete**：当最后一个绑定到Exchange上的队列删除后，自动删除该Exchange。
- **Internal**：当前Exchange是否用于RabbitMQ内部使用，默认为false。**(很少使用)**
- **Arguments**：扩展参数，用于扩展AMQP协议自制定化使用。

## Direct Exchange

- 所有发送到Direct Exchange的消息被转发到Routing key中指定的Queue。
- 一句话：直连的方式，生产者发送消息的Routing Key和Direct Exchange的Routing Key必须完全匹配，才会路由到绑定的Queue。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207121642954.png" alt="direct"/>

## Topic Exchange

- 所有发送到Topic Exchange的消息被转发到所有关系Routing Key中指定Topic的Queue上。
- Exchange将Routing Key和某个Topic进行**模糊匹配**，此时队列需要绑定一个Topic。
- 一句话：Topic Exchange和Queue绑定Routing Key可以使用通配符，生产者发送消息的Routing Key只要和Topic Exchange的Routing Key匹配就能路由到Topic Exchange绑定的队列。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207121642805.png" alt="topic"/>

> 模糊匹配可以使用通配符
>
> - **符号 "#" 匹配0个或多个词。**
> - **符号 "*" 匹配一个词。**
> - **例如："log.#" 能够匹配到 "log.info.aa"。"log.*" 只能匹配到 "log.err"。**

## Fanout Exchange

- Fanout Exchange不处理Routing Key，只需要简单的将Queue绑定到Exchange上。
- 发送到Exchange的消息都会被转发到与该Exchange绑定的所有Queue上。
- Fanout Exchange转发消息是最快的。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207121642577.png" alt="发布订阅模式"  />

## Headers Exchange

**1: 不处理路由键**。而是根据发送的消息内容中的 `headers`属性进行匹配。

- 在绑定Queue与Exchange时指定一组键值对；
- 当消息发送到RabbitMQ时会取到该消息的headers与Exchange绑定时指定的键值对进行匹配；
- 如果完全匹配则消息会路由到该队列，否则不会路由到该队列。
- headers属性是一个键值对，可以是Hashtable，键值对的值可以是任何类型。而fanout，direct，topic 的路由键都需要要字符串形式的。

**2: 匹配规则x-match有下列两种类型**：

- x-match = all ：表示所有的键值对都匹配才能接受到消息；
- x-match = any ：表示只要有键值对匹配就能接受到消息。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207121641034.png" alt="headers" />

# 如何保证消息发送？消息接收？

## 一、消息发送：发送方确认机制

信道需要设置为 contirm 模式，则所有在信道上发布的消息都会分配一个唯一D。

1. 一旦消息**被投递到queue**（可持久化的消息需要写入磁盘），信道会发送一个确认**ack**给生产者（包含消息
	唯一ID）。如果 RabbitMQ 发生内部错误从而导致**消息丟失**，会发送一条 **nack**（未确认）消息给生产者。

2. 所有被发送的消息都将被 contirm（即ack） 或者被nack一次。
3. 但是没有对消息被 confirm 的快慢做任何保证，井且**同一条消息不会既被 confirm又被 nack**。

> 发送方确认模式是**异步的**，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者，生产者的回调方法会被触发。

ConfirmCallback接口：只确认是否正确到达 Exchange 中，成功到达则回调

ReturnCallback接口：消息失败返回时回调

## 二、消息接收：接收方确认机制

消费者在声明队列时，可以**指定noAck參数**，当noAck=false时， RabbitMQ会**等待消费者显式发回ack**信号后**才从内存**(或者磁盘，持久化消息)中**移去消息**。否则，消息被消费后会被立即删除。

消贵者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作)。**只有消费者确认了消息，RabbitMQ 才能安全地把消息从队列中删除**。

> RabbitMQ不会为末ack的消息设置超时时间，它判断此消息是否需要重新投递给消费者的唯一依据是消费该消息的消费者连接是否已经断开。这么设计的原因是RabbitMQ**允许消费者消费一条消息的时间可以很长**。保证数据的最终一致性。

如果消费者返回ack之前断开了链接，RabbitMQ 会重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）。



# 高级特性

## 一、生存时间TTL

- TTL：`Time To Live`，也就是生存时间。
- RabbitMQ支持消息的过期时间，在消息发送时可以指定。
- RabbitMQ支持队列的过期时间，从消息入队开始计算，只要超过了队列的超时时间配置，那么消息会自动清除。

过期时间TTL表示可以对消息设置过期的时间，在这个时间内都可以被消费者接收获取，过了这段时间消息将会被自动删除。

RabbitMQ 可以对消息和队列设置TTL，目前有两种方式可以设置：

- **通过队列属性设置**，队列中每条消息都有相同的过期时间。
- **对消息进行单独设置**，每条消息TTL可以不同。

> **如果上述两种方法同时使用，则消息的过期时间以两者TTL较小的那个数值为准。**
>
> 消息在队列的生存时间一旦超过设置的TTL值，就成为 dead message 被投递到死信队列，消费者将无法收道该消息！
>
> **注意：第一种方式可以将消息转移到死信队列中；第二种方式消息过期会直接被删除**。

## 二、死信队列DLX

**死信队列也是个消息队列，只是用来存放没有成功消费的消息，通常可以用来作为消息的重试。**

- 利用`DLX（Dead-Letter-Exchange）`，当消息在一个队列中变成Dead Message后，它会被重新publish到另一个Exchange，这个Exchange就是DLX。
- DLX也是一个正常的Exchange，和一般的Exchange没有区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性。
- 当这个队列中有Dead Message时，RabbitMQ就会自动的将这个消息重新发布到设置的Exchange上去，进而被路由到另一个队列。
- 可以监听这个死信队列中消息做相应的处理，这个特性可以弥补RabbitMQ3.0以前支持的immediate参数的功能。

> **消息变成Dead Message的情况**：
>
> - 消息被消费者拒绝（basicReject/basicNack）并且不能重回队列requeue=false。
> - 消息TTL过期。
> - 队列达到最大长度。



## 三、延迟队列

延迟队列就是用来存放需要在指定时间被处理的元素的队列，通常可以用来处理一些具有过期性操作的业务，比如十分钟内未支付则取消订单。

https://blog.csdn.net/IT_hot_pot/article/details/113616967