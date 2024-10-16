

#### 简述kafka架构设计是什么样

语义概念

```
1 broker
Kafka 集群包含一个或多个服务器，服务器节点称为broker。

broker存储topic的数据。如果某topic有N个partition，集群有N个broker，那么每个broker存储该topic的一个partition。

如果某topic有N个partition，集群有(N+M)个broker，那么其中有N个broker存储该topic的一个partition，剩下的M个broker不存储该topic的partition数据。

如果某topic有N个partition，集群中broker数目少于N个，那么一个broker存储该topic的一个或多个partition。在实际生产环境中，尽量避免这种情况的发生，这种情况容易导致Kafka集群数据不均衡。

2 Topic
每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

类似于数据库的表名

3 Partition
topic中的数据分割为一个或多个partition。每个topic至少有一个partition。每个partition中的数据使用多个segment文件存储。partition中的数据是有序的，不同partition间的数据丢失了数据的顺序。如果topic有多个partition，消费数据时就不能保证数据的顺序。在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1。

4 Producer
生产者即数据的发布者，该角色将消息发布到Kafka的topic中。broker接收到生产者发送的消息后，broker将该消息追加到当前用于追加数据的segment文件中。生产者发送的消息，存储到一个partition中，生产者也可以指定数据存储的partition。

5 Consumer
消费者可以从broker中读取数据。消费者可以消费多个topic中的数据。

6 Consumer Group
每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制-给consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。

7 Leader
每个partition有多个副本，其中有且仅有一个作为Leader，Leader是当前负责数据的读写的partition。

8 Follower
Follower跟随Leader，所有写请求都通过Leader路由，数据变更会广播给所有Follower，Follower与Leader保持数据同步。如果Leader失效，则从Follower中选举出一个新的Leader。当Follower与Leader挂掉、卡住或者同步太慢，leader会把这个follower从“in sync replicas”（ISR）列表中删除，重新创建一个Follower。

9 Offset
kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka
```

KAFKA天生是分布式的，满足AKF的XYZ轴特点，扩展性，可靠性，高性能是没得说

而且，kafka具备自己的特色，比如动态ISR集合，是在强一致性，过半一致性之外的另一个实现手段






#### Kafka消息丢失的场景有哪些

生产者在生产过程中的消息丢失

broker在故障后的消息丢失

消费者在消费过程中的消息丢失

#### ACK机制

ack有3个可选值，分别是1，0，-1。

#### ack=0：生产者在生产过程中的消息丢失

简单来说就是，producer发送一次就不再发送了，不管是否发送成功。

#### ack=1：broker在故障后的消息丢失

简单来说就是，producer只要收到一个分区副本成功写入的通知就认为推送消息成功了。这里有一个地方需要注意，这个副本必须是leader副本。只有leader副本成功写入了，producer才会认为消息发送成功。

注意，ack的默认值就是1。这个默认值其实就是吞吐量与可靠性的一个折中方案。生产上我们可以根据实际情况进行调整，比如如果你要追求高吞吐量，那么就要放弃可靠性。

#### ack=-1：生产侧和存储侧不会丢失数据

简单来说就是，producer只有收到分区内所有副本的成功写入的通知才认为推送消息成功了。

#### Offset机制

kafka消费者的三种消费语义

at-most-once：最多一次，可能丢数据

at-least-once：最少一次，可能重复消费数据

exact-once message：精确一次





















---

# Kafka是pull？push？以及优劣势分析



Kafka最初考虑的问题是，customer应该从brokes拉取消息还是brokers将消息推送到consumer，也就是pull还push。

Kafka遵循了一种大部分消息系统共同的传统的设计：producer将消息推送到broker，consumer从broker拉取消息。

一些消息系统比如Scribe和Apache Flume采用了push模式，将消息推送到下游的consumer。

这样做有好处也有坏处：由broker决定消息推送的速率，对于不同消费速率的consumer就不太好处理了。

消息系统都致力于让consumer以最大的速率最快速的消费消息，但不幸的是，push模式下，当broker推送的速率远大于consumer消费的速率时，consumer恐怕就要崩溃了。

最终Kafka还是选取了传统的pull模式。

Pull模式的另外一个好处是consumer可以自主决定是否批量的从broker拉取数据。

Push模式必须在不知道下游consumer消费能力和消费策略的情况下决定是立即推送每条消息还是缓存之后批量推送。

如果为了避免consumer崩溃而采用较低的推送速率，将可能导致一次只推送较少的消息而造成浪费。

Pull模式下，consumer就可以根据自己的消费能力去决定这些策略。

Pull有个缺点是，如果broker没有可供消费的消息，将导致consumer不断在循环中轮询，直到新消息到达。

为了避免这点，Kafka有个参数可以让consumer阻塞知道新消息到达(当然也可以阻塞知道消息的数量达到某个特定的量这样就可以批量发



















---

# Kafka中zk的作用是什么

Zookeeper是分布式协调，注意它不是数据库

kafka中使用了zookeeper的分布式锁和分布式配置及统一命名的分布式协调解决方案

在kafka的broker集群中的controller的选择，是通过zk的临时节点争抢获得的

brokerID等如果自增的话也是通过zk的节点version实现的全局唯一

kafka中broker中的状态数据也是存储在zk中，不过这里要注意，zk不是数据库，所以存储的属于元数据

而，新旧版本变化中，就把曾经的offset从zk中迁移出了zk















---

# Kafka中高性能如何保障

首先，性能的最大瓶颈依然是IO，这个是不能逾越的鸿沟

虽然，broker在持久化数据的时候已经最大努力的使用了磁盘的顺序读写

更进一步的性能优化是零拷贝的使用，也就是从磁盘日志到消费者客户端的数据传递，因为kafka是mq，对于msg不具备加工处理，所以得以实现

然后就是大多数分布式系统一样，总要做tradeoff，在速度与可用性/可靠性中挣扎

ACK的0，1，-1级别就是在性能和可靠中权衡















---

# kafka的rebalance机制是什么

## 消费者分区分配策略

Range 范围分区(默认的)

RoundRobin 轮询分区

Sticky策略

## 触发 Rebalance 的时机

Rebalance 的触发条件有3个。

- 组成员个数发生变化。例如有新的 consumer 实例加入该消费组或者离开组。
- 订阅的 Topic 个数发生变化。
- 订阅 Topic 的分区数发生变化。

## Coordinator协调过程

消费者如何发现协调者

消费者如何确定分配策略

如果需要再均衡分配策略的影响





