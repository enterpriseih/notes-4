[参考](https://blog.csdn.net/cao1315020626/article/details/112590786?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165854447916781685326183%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165854447916781685326183&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-112590786-null-null.142^v33^new_blog_pos_by_title,185^v2^control&utm_term=kafka&spm=1018.2226.3001.4187)

# Kafka消费模式

### 点对点

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231109893.png" alt="一对一消费模式" style="zoom:75%;" />

消息生产者发布消息到Queue队列中，通知消费者从队列中拉取消息进行消费。**消息被消费之后则删除**，Queue支持多个消费者，但对于一条消息而言，只有一个消费者可以消费，即一条消息只能被一个消费者消费。

### 发布/订阅

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231109265.png" alt="一对多消费" style="zoom:75%;" />

利用Topic存储消息，消息生产者将消息发布到Topic中，同时有多个消费者订阅此topic，消费者可以从中消费消息，注意发布到Topic中的消息会被多个消费者消费，**消费者消费数据之后，数据不会被清除**，Kafka会默认保留一段时间，然后再删除。

# 基础架构

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231723315.png" alt="image-20220723172346266" style="zoom:50%;" />

- **Consumer Group**：消费者组，消费者组则是一组中存在多个消费者，消费者消费Broker中当前Topic的不同分区中的消息，消费者组之间互不影响，所有的消费者都属于某个消费者组，即消费者组是**逻辑上的一个订阅者**。**某一个分区中的消息只能被一个消费者组中的一个消费者所消费**
- **Broker**：经纪人，一台Kafka服务器就是一个Broker，一个集群由多个Broker组成，一个Broker可以容纳多个Topic。
- **Topic**：主题，可以理解为一个队列，生产者和消费者都是面向一个Topic
- **Partition**：分区，为了实现**扩展性**以及**备份**，一个非常大的Topic可以分布到多个Broker上，一个Topic可以分为多个Partition（分给多个broker服务器），每个Partition是一个有序的队列(分区有序，不能保证全局有序)
- **Replication**：副本，为保证集群中某个节点发生故障，节点上的Partition数据不丢失，Kafka可以正常的工作，Kafka提供了副本机制，一个Topic的每个分区有若干个副本，一个Leader和多个Follower
- **Leader**：每个分区多个副本的主角色，生产者发送数据的对象，以及消费者消费数据的对象都是Leader。
- **Follower**：每个分区多个副本的从角色，实时的从Leader中同步数据，保持和Leader数据的同步，Leader发生故障的时候，某个Follower会成为新的Leader。

> 一个Topic会产生多个分区Partition，分区中分为Leader和Follower，消息一般发送到Leader，Follower通过数据的同步与Leader保持同步，消费的话也是在Leader中发生消费，如果多个消费者，则分别消费Leader和各个Follower中的消息，当Leader发生故障的时候，某个Follower会成为主节点，此时会对齐消息的偏移量。

Zookeeper存储：运行的broker、leader、ISR（后续详情）

# 生产者-Kafka为什么吞吐量高

- kafka的生产者采用的是**异步发送消息机制**，当发送一条消息时，**消息井没有发送到Broker而是缓存起来**，然后直接向业务返回成功，当**缓存的消息达到一定数量时再批量发送给Broker**。
- 这种做法**减少了网络io**，从而提高了消息发送的吞吐量，但是如果消息生产者宕机，会导致消息丢失，业务出错，所以理论上katka利用此机制提高了性能却降低了可靠性。

### 提高吞吐量

- batch.size：批次大小，默认16k

- linger.ms：等待时间，修改为5-100ms

- compression.type：压缩snappy

- RecordAccumulator：缓冲区大小，修改为64m

# 生产者-发送流程

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241723861.png" alt="image-20220724172301478" style="zoom:50%;" />

# 生产者-分区策略

## 分区的原因

- **方便在集群中扩展**：每个partition通过调整以适应它所在的机器，而一个Topic又可以有多个partition组成，因此整个集群可以适应适合的数据；**负载均衡**
- **可以提高并发**：**以Partition为单位进行读写**。类似于多路。

## 分区的原则

- 指明partition（这里的指明是指第几个分区）的情况下，直接将指明的值作为partition的值
- 没有指明partition的情况下，但是存在值key，此时将key的hash值与topic的partition总数进行取余得到partition值
- 既没有**partition**值又没有**key**值的情况下，**Kafka**采用**Sticky Partition**(黏性分区器)，会随机选择一个分区，并尽可能一直 使用该分区，待该分区的**batch**已满或者已完成，**Kafka**再随机一个分区进行使用(和上一次的分区不同)。

# 生产者-ISR（in-sync replica）

topic的每个partition收到producer发送的数据后，都需要向producer发送ack，如果producer收到ack就会进行下一轮的发送，**否则重新发送数据**。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231505094.png" alt="消息发送示意图" style="zoom:75%;" />



## 副本数据同步策略

**半数follower同步完成即发送ack**

> 优点是延迟低
>
> 缺点是选举新的leader的时候，容忍n台节点的故障，需要2n+1个副本（因为需要半数同意，所以故障的时候，能够选举的前提是剩下的副本超过半数），容错率为1/2

**全部follower同步完成完成发送ack**

> 优点是容错率高，选举新的leader的时候，容忍n台节点的故障只需要n+1个副本即可，因为只需要剩下的一个人同意即可发送ack了
>
> 缺点是延迟高，因为需要全部副本同步完成才可

kafka选择的是第二种，因为在容错率上面更加有优势，同时对于分区的数据而言，每个分区都有大量的数据，第一种方案会造成大量数据的冗余。虽然第二种网络延迟较高，但是网络延迟对于Kafka的影响较小。

## ISR同步副本集

**问题**

采用了第二种方案进行同步ack之后，如果leader收到数据，所有的follower开始同步数据，但**有一个follower因为某种故障，迟迟不能够与leader进行同步**，那么leader就要一直等待下去，直到它同步完成，才可以发送ack，此时需要如何解决这个问题呢？

**解决**

leader中维护了一个动态的ISR（in-sync replica set），即**与leader保持同步的follower+leader集合**。

当ISR中的follower完成数据的同步之后，给leader发送ack，**如果follower长时间没有向leader同步数据，则该follower将从ISR中被踢出**，该时间阈值由replica.lag.time.max.ms参数设定，默认30s。当leader发生故障之后，会从ISR中选举出新的leader。



# 生产者-ACK机制

对于某些**不太重要的数据**，对数据的**可靠性要求不高**，能够容忍数据的少量丢失，所以没有必要等到ISR中所有的follower全部接受成功。

Kafka为用户提供了三种可靠性级别，用户根据可靠性和延迟的要求进行权衡选择不同的配置。

- `0`：producer不等待broker的ack，这一操作提供了**最低的延迟**，broker接收到还没有写入磁盘就已经返回，**当broker故障时有可能丢失数据**
- `1`：producer等待broker的ack，partition的leader落盘成功后返回ack，**如果在follower同步成功之前leader故障，那么将丢失数据。**
	- ack后生产者不会再发，因为它认为已经发过了

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231515736.png" alt="img" style="zoom:75%;" />

- `-1(all`：producer等待broker的ack，partition的leader和ISR的follower全部落盘成功才返回ack，但是**如果在follower同步完成后，broker发送ack之前，如果leader发生故障，会造成数据重复。**
	- 生产者没有收到ack，会继续重发，导致数据重复

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231515832.png" alt="img" style="zoom:75%;" />

# 生产者-数据去重

## 数据传递语义

**至少一次**(**AtLeastOnce**) = ACK级别设置为-1 + 分区副本大于等于2 + ISR里应答的最小副本数量大于等于2

- 可以保证数据不丢失，但不保证数据不重复

**最多一次**(**AtMostOnce**) = ACK级别设置为0

- 可以保证数据不重复，但不保证数据不丢失

**精确一次**(**Exactly Once**)：对于一些非常重要的信息，比如和钱相关的数据，要求数据既不能重复也不丢失。

=> 幂等性和事务

## 幂等性

幂等性就是指Producer不论向Broker发送多少次重复数据，**Broker端都只会持久化一条**，保证了不重复。

**精确一次**(**Exactly Once**) = 幂等性 + 至少一次( ack=-1 + 分区副本数>=2 + ISR最小副本数量>=2) 。

> 副本数 >= 2 是除了leader还要有个follower

使用：开启参数 **enable.idempotence** 默认为 true，false 关闭。

### 重复数据的判断标准

具有**<PID, Partition, SeqNumber>**相同主键的消息提交时，Broker只会持久化一条。

- PID是producer id，Kafka每次重启都会分配一个新的；
- Partition 表示分区号；
- Sequence Number是单调自增的。

> 幂等性只能保证的是在**单分区单会话**内不重复。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231631507.png" alt="image-20220723163133703" style="zoom:50%;" />

## 事务

> 开启事务，必须开启幂等性。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231635374.png" alt="image-20220723163544418" style="zoom:50%;" />



# 生产者-数据顺序

## 数据有序

发送的数据顺序和消费的顺序相同

单分区内有序（有条件）

## 数据乱序

乱序是因为，request1发送后发request2，然后发送request3，但是2没有ack，就先发了3，导致顺序是132，而不是123

1. 在1.x版本之前保证数据单分区有序，条件如下

	**max.in.flight.requests.per.connection**=1(不需要考虑是否开启幂等性)。

2. 在1.x之后，条件如下：

	- 未开启幂等性：**max.in.flight.requests.per.connection**需要设置为**1**
	- 开启幂等性：**max.in.flight.requests.per.connection**需要设置小于等于**5**

> 启用幂等后，kafka服务端会缓存producer发来的最近5个request的元数据， 故无论如何，都可以保证最近5个request的数据都是有序的（最多5个，6个都不行了）。
>
> 根据Sequence Number来判断，只要发现不是单调递增的数据，就暂停落盘，之前的可以落盘。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231649708.png" alt="image-20220723164939787" style="zoom:50%;" />



### 

# Broker-文件存储与清理

## 文件存储

Topic是逻辑上的改变，Partition是物理上的概念，每个Partition对应着一个log文件，该log文件中存储的就是producer生产的数据，`topic=N*partition；partition=log`

**Producer生产的数据会被不断的追加到该log文件的末端**，且每条数据都有自己的offset，consumer组中的每个consumer，都会实时记录自己消费到了哪个offset，以便出错恢复的时候，可以从上次的位置继续消费。

> 流程：Producer => Topic（Log with offset）=> Consumer。

主要通过相应的log和index等文件保存具体的消息文件。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231456601.png" alt="iShot_2022-07-23_14.51.02" style="zoom:50%;" />

生产者不断的向log文件追加消息文件，为了防止log文件过大导致定位效率低下，Kafka的log文件以1G为一个分界点，**当.log文件大小超过1G的时候，此时会创建一个新的.log文件**，同时为了快速定位大文件中消息位置，Kafka采取了**分片**和**索引**的机制来加速定位。

在kafka的存储log的地方，即文件的地方，会存在消费的偏移量以及具体的分区信息，分区信息的话主要包括.index和.log文件组成。

**index和log文件以当前segment的第一条消息的offset命名**。index和log位于一个文件夹下，该文件夹命名：topic名称+分区序号。

> 1. index为稀疏索引，大约每往log文件写入4kb数据，才会往index文件写入一条索引。参数`log.index.interval.bytes`默认4kb
> 2. **index文件中存放的是相对offset**，确保offset占用的空间不会过大。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231452051.png" alt="iShot_2022-07-23_14.50.17" style="zoom: 67%;" />

**此时如何快速定位数据**，**步骤**：

> .index文件存储的消息的offset+真实的起始偏移量。.log中存放的是真实的数据。

- 首先通过二分查找.index文件到查找到当前消息具体的偏移，如上图所示，查找为600，发现第二个文件为522，第三个文件为，则定位到第二个文件中。
- 找到小于等于目标offset的最大offset
- 定位到之后获取相对偏移量65+当前文件大小522=总的偏移量587。
- 获取到总的偏移量之后，直接定位到.log文件即可快速获得当前消息大小。

> 这里碰巧是6410，6415也在该位置，以为稀疏索引。

## 文件清理

默认7天，默认删除

- delete日志删除：将国旗数据删除

	- 基于时间：默认打开。以 segment 中所有记录中的**最大时间戳作为该文件时间戳**。

		比如一个segment中只过期最后的部分，但是该文件也删除，因为使用最大的时间戳

	- 基于大小：默认关闭。超过设置的所有日志总大小，删除最早的 segment。

- compact日志压缩：对于相同key的不同value值，只保留最后一个版本

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207232207746.png" alt="image-20220723220716680" style="zoom:50%;" />

> compact策略只适合特殊场景，比如消息的key是用户ID，value是用户的资料，通过这种压缩策略，整个消息集里就保存了所有用户最新的资料。



# Broker-ZK

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231722919.png" alt="image-20220723172224560" style="zoom:50%;" />

在zookeeper的服务端存储的Kafka相关信息: 

1. /kafka/brokers/ids 

	[0,1,2] 记录**有哪些服务器**

2. /kafka/brokers/topics/first/partitions/0/state

	 {"leader":1 ,"isr":[1,0,2] } 记录**谁是Leader**，有**哪些服务器可用**

3. /kafka/controller 

	{“brokerid”:0} **辅助选举Leader**；主要由controller负责选举

> 0.9之后，offset就不存在zk中了，因为每次产生offset都要存进去，影响性能。

# Broker-工作流程

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231728982.png" alt="image-20220723172814768" style="zoom:50%;" />

> AR：分区中所有副本统称；启动的时候会有个节点顺序
>
> ISR：leader和follower通讯正常的节点

1. broker启动后在zk中注册

2. controller是哪个broker先注册谁就说了算

3. 由controller监听broker节点变化

4. controller决定leader选举

	规则：在isr中存活为前提，按照AR中的顺序轮询

5. controller将节点信息上传到zk

6. 其他controller从zk同步信息，防止leader挂了

# Broker-副本

## 副本基本信息

1. Kafka 副本作用:提高数据可靠性。

2. Kafka 默认副本 1 个，生产环境一般配置为 2 个，保证数据可靠性;太多副本会 增加磁盘存储空间，增加网络上数据传输，降低效率。

3. Kafka 中副本分为：Leader 和 Follower。**Kafka 生产者只会把数据发往 Leader， 然后 Follower 找 Leader 进行同步数据。**

4. Kafka 分区中的所有副本统称为 AR(Assigned Repllicas)。

> AR = ISR + OSR
>
> OSR：表示 Follower 与 Leader 副本同步时，延迟过多的副本。即被从ISR中踢出的。



## 数据一致性问题

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207231517320.png" alt="img" style="zoom:75%;" />

- **LEO(Log End Offset)**：每个副本最后的一个offset
- **HW(High Watermark)**：**ISR队列中最小**的LEO。

**follower故障**：

- follower发生故障后会被临时踢出ISR，
- 等待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步，
- 等待该follower的LEO大于等于该partition的HW，即follower追上leader之后，就可以重新加入ISR了。

**leader故障**：

- leader发生故障之后，会从ISR中选出一个新的leader，
- 为了保证多个副本之间的数据的一致性，**其余的follower会先将各自的log文件高于HW的部分截掉**，然后从新的leader中同步数据。

> 这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复



# Broker-高效读取数据

- Kafka 本身是分布式集群，可以采用分区技术，并行度高
- 读数据采用稀疏索引，可以快速定位要消费的数据
- 顺序写磁盘：producer 生产数据，要写入到 log 文件中，写的过程是一直**追加**到文件末端
- 页缓存 + 零拷贝技术

**零拷贝**：**Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理**。Kafka Broker应用层不关心存储的数据，所以就不用走应用层，传输效率高。

**PageCache页缓存**：Kafka重度依赖底层操作系统提供的PageCache功能。当上层有写操作时，操作系统只是将数据写入 PageCache。当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。实际上PageCache是把尽可能多的空闲内存都当作了磁盘缓存来使用。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207232216743.png" alt="image-20220723221626036" style="zoom:50%;" />



# 消费者-消费方式

- **pull（拉）模式**

consumer从broker中主动拉取数据；kafka采用该方法。

pull可以由消费者自己控制，**根据自己的消息处理能力**来进行控制，但是消费者不能及时知道是否有消息，**可能会拉到的消息为空**。

- **push（推）模式**

Broker主动给消费者推送消息。

由broker决定消费速率，很难适应所有消费者的消费速率，**可能会造成网络堵塞**，消费者压力大等问题

# 消费者-工作流程

## 总体工作流程

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207232230504.png" alt="image-20220723223004825" style="zoom:50%;" />

## 消费者组原理

Consumer Group(CG)：消费者组，由多个consumer组成。形成一个消费者组的条件，是所有消费者的groupid相同。

- **消费者组内每个消费者负责消费不同分区的数据**，一个分区的数据只能由消费者组中的一个消费者消费。
- **消费者组之间互不影响**。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207232254396.png" alt="image-20220723225400882" style="zoom:50%;" />

# 消费者-分区的分配与再平衡

分区分配策略：Range、RoundRobin、Sticky、CooperativeSticky

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241726579.png" alt="image-20220724172641695" style="zoom:50%;" />

由哪个consumer来消费哪个partition的数据。

> Kafka 默认的分区分配策略就是 Range + CooperativeSticky。

## 一、Range

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241735909.png" alt="image-20220724173537768" style="zoom:50%;" />

Range针对的是每个topic。

> 容易产生数据倾斜！！！

> 0 号消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待，时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。

> 如果确认挂掉了，则挂掉的消费者消费的分区数据将会**整体**传给另一个消费者。



## 二、RoundRobin

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241747463.png" alt="image-20220724174745117" style="zoom:50%;" />

RoundRobin 针对集群中所有Topic而言。

RoundRobin **轮询**分区策略，是把所有的 partition 和所有的 consumer 都列出来，然后按照 hashcode 进行排序，最后通过轮询算法来分配 partition 给到各个消费者。

> 如果挂掉了，会在确认挂掉之后，按照RoundRobin规则，将挂掉的消费者消费的分区重新分配。

## 三、Sticky

**粘性分区定义**：可以理解为分配的结果带有“粘性的”。即在执行一次新的分配之前， **考虑上一次分配的结果**，**尽量少的调整分配的变动**，可以节省大量的开销。

- 首先会**尽量均衡**的放置分区到消费者上面，在出现同一消费者组内消费者出现问题的时候，会**尽量保持原有分配的分区**不变化。

> 如果挂掉了，会在确认挂掉之后，按照Sticky规则，将挂掉的消费者消费的分区重新分配。



# 消费者-offset偏移量

## offset的默认维护位置

- 0.9开始，consumer默认将offset保存在Kafka一个内置的topic中，该topic为__consumer_offsets

- 0.9之前， consumer默认将 offset保存在Zookeeper中

__consumer_offsets 主题里面采用 key 和 value 的方式存储数据。**key** 是 **group.id+topic+分区号**，value 就是当前 offset 的值。每隔一段时间，kafka 内部会对这个 topic 进行 **compact**，也就是每 group.id+topic+分区号就**保留最新数据**。

## 自动提交offset

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241812346.png" alt="image-20220724181218486" style="zoom:50%;" />

自动提交offset的相关参数：

- **enable.auto.commit**：是否开启自动提交offset功能，默认是true

- **auto.commit.interval.ms**：自动提交的时间间隔，默认5s

## 手动提交offset

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241814054.png" alt="image-20220724181441277" style="zoom:50%;" />

手动提交offset的方法有两种：分别是**commitSync**(同步提交)和**commitAsync**(异步提交)。

相同点：**都会将本次提交的一批数据最高的偏移量提交**；

不同点：同步提交阻塞当前线程，一直到提交成功，并且会**自动失败重试**(由不可控因素导致，也会出现提交失败)；而异步提交则**没有失败重试机制**，故有可能提交失败。

- commitSync(同步提交)：必须等待offset提交完毕，再去消费下一批数据。

- commitAsync(异步提交)：发送完提交offset请求后，就开始消费下一批数据了。

## 指定offset消费

auto.offset.reset = earliest | latest | none 默认是 latest。

当 Kafka 中没有初始偏移量(消费者组第一次消费)或服务器上不再存在当前偏移量时(例如该数据已被删除)

- earliest：自动将偏移量重置为最早的偏移量，--from-beginning。

- latest(默认值)： 自动将偏移量重置为最新偏移量。
- none：如果未找到消费者组的先前偏移量，则向消费者抛出异常。
- 任意指定 offset 位移开始消费

## 指定时间消费

在生产环境中，会遇到最近消费的几个小时数据异常，想重新按照时间消费。 例如要求按照时间消费前一天的数据。

**把时间转换成对应的offset就可以了**。

## 漏消费和重复消费

- 重复消费：已经消费了数据，但是 offset 没提交。自动提交引起的。 
- 漏消费：先提交 offset 后消费，有可能会造成数据的漏消费。手动提交引起的。

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241843495.png" alt="image-20220724184344430" style="zoom:50%;" />

如何解决：消费者事务



# 消费者-事务

如果想完成Consumer端的精准一次性消费，那么需要Kafka消费端将消费过程和提交offset 过程做原子绑定。

- 下游消费者必须支持事务，才能做到精确一次性消费。

# 消费者-数据积压之消费者提高吞吐量

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207241848242.png" alt="image-20220724184833581" style="zoom:50%;" />

1. Kafka消费能力不足

	增加topic分区数，并提升消费者组的消费者数量；消费者数 = 分区数。

2. 下游数据处理不及时

	提高每次拉取的数量

# 简问

## Kafka为什么吞吐量高

kafka的生产者采用的是**异步发送消息机制**，当发送一条消息时，**消息井没有发送到Broker而是缓存起来**，然后直接向业务返回成功，当**缓存的消息达到一定数量时再批量发送给Broker**。这种做法**减少了网络io**，从而提高了消息发送的吞吐量，但是如果消息生产者宕机，会导致消息丢失，业务出错，所以理论上katka利用此机制提高了性能却降低了可靠性。

> 迟迟未到设定的时间，也会发送。

## Kafka的Pull和Push分别有什么优缺点

1. **pull**表示消费者主动拉取，可以批量拉取，也可以单条拉取，所以pull可以由消费者自己控制，**根据自己的消息处理能力**来进行控制，但是消费者不能及时知道是否有消息，**可能会拉到的消息为空**
2. **push**表示Broker主动给消费者推送消息，所以肯定是**有消息时才会推送**，但是消费者不能按自己的能力来消费消息，推过来多少消息，消费者就得消费多少消息，所以**可能会造成网络堵塞**，消费者压力大等问题

## Kafka高效文件存储设计特点

1. Kafka 把 topic 中—个parition 大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完的文件，减少磁盘占用。
2. 通过索引信息可以快速定位 message 和确定response 的最大大小。
3. 通过 index 元数据全部映射到memory，可以避免 segment file 的 10磁盘操作。
4. 通过索引文件稀疏存储，可以大幅降低 index 文件元数据占用空间大小，

## Kafka与传统消息系统之间有三个关键区别

1. Kafka **持久化日志**，这些日志可以被重复读取和无限期保留

2. Kafka 是一个**分布式系统**：它以集群的方式运行，可以灵活伸缩，在内部通过复制数据提升容错能力和高可用性

3. Kafka 支持**实时的流式处理**

## Kafka的消费者如何消费数据

消费者每次消费数据的时候，消费者都会记录消费的物理**偏移量（offset）**的位置等到下次消费时，会接着上次位置继续消费

## Kafka消费者负载均衡策略

一个消费者组中的一个分片对应一个消费者成员，他能保证每个消费者成员都能访问，如果组中成员太多会有空闲的成员

## kafaka生产数据时数据的分组策略

生产者決定数据产生到集群的哪个 partition 中，每一条消息都是以（key， value）格式，key是由生产者发送数据传入，所以生产者（key）决定了数据产生到集群的哪个 partition

## Kafka中是怎么体现消息顺序性的

katka每个partition中的消息在写入时都是有序的，消费时，每个partition只能被每一个group中的一个消费者消费，保证了消费时也是有序的。整个topic不保证有序。如果为了保证topic整个有序，那么将partition调整为1。

## Kafka的分区partition



https://blog.csdn.net/ATYtian/article/details/125647322