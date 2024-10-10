


### RabbitMQ如何确保消息发送和消息接收

#### 消息发送确认

##### 1 ConfirmCallback方法

ConfirmCallback 是一个回调接口，消息发送到 Broker 后触发回调，确认消息是否到达 Broker 服务器，**也就是只确认是否正确到达 Exchange 中。**



##### 2 ReturnCallback方法

通过实现 ReturnCallback 接口，启动消息失败返回，此接口是在交换器路由不到队列时触发回调，该方法可以不使用，因为交换器和队列是在代码里绑定的，如果消息成功投递到 Broker 后几乎不存在绑定队列失败，除非你代码写错了。

#### 消息接收确认

RabbitMQ 消息确认机制（ACK）默认是自动确认的，自动确认会在消息发送给消费者后立即确认，但存在丢失消息的可能，如果消费端消费逻辑抛出异常，假如你用回滚了也只是保证了数据的一致性，但是消息还是丢了，也就是消费端没有处理成功这条消息，那么就相当于丢失了消息。

消息确认模式有：

AcknowledgeMode.NONE：自动确认。
AcknowledgeMode.AUTO：根据情况确认。
AcknowledgeMode.MANUAL：手动确认。
消费者收到消息后，手动调用 Basic.Ack 或 Basic.Nack 或 Basic.Reject 后，RabbitMQ 收到这些消息后，才认为本次投递完成。

Basic.Ack 命令：用于确认当前消息。
Basic.Nack 命令：用于否定当前消息（注意：这是AMQP 0-9-1的RabbitMQ扩展） 。
Basic.Reject 命令：用于拒绝当前消息。
Nack,Reject后都有能力要求是否requeue消息或者进入死信队列































---

# RabbitMQ事务消息原理是什么

## 事务V.S确认

确认是对一件事的确认

事务是对批量的确认

增删改查中，事务是对于增删改的保证

## 发送方事务

开启事务，发送多条数据，事务提交或回滚是原子的，要么都提交，要么都回滚

## 消费方事务

消费方是读取行为，那么事务体现在哪里呢

rabbitmq的消费行为会触发queue中msg的是否删除、是否重新放回队列等行为，类增删改

所以，消费方的ack是要手动提交的，且最终确定以事务的提交和回滚决定









# RabbitMQ的架构设计是什么样的

## 是AMQP的实现，相关概念语义

Broker:它提供一种传输服务,它的角色就是维护一条从生产者到消费者的路线，保证数据能按照指定的方式进行传输

Exchange：消息交换机,它指定消息按什么规则,路由到哪个队列。 

Queue:消息的载体,每个消息都会被投到一个或多个队列。 

Binding:绑定，它的作用就是把exchange和queue按照路由规则绑定起来. 

Routing Key:路由关键字,exchange根据这个关键字进行消息投递。 

vhost:虚拟主机,一个broker里可以有多个vhost，用作不同用户的权限分离。

Producer:消息生产者,就是投递消息的程序. 

Consumer:消息消费者,就是接受消息的程序. 

Channel:消息通道,在客户端的每个连接里,可建立多个channel.

### 核心概念

在mq领域中，producer将msg发送到queue，然后consumer通过消费queue完成P.C解耦

kafka是由producer决定msg发送到那个queue

rabbitmq是由Exchange决定msg应该怎么样发送到目标queue，这就是binding及对应的策略

### Exchange

Direct Exchange:直接匹配,通过Exchange名称+RountingKey来发送与接收消息. 
Fanout Exchange:广播订阅,向所有的消费者发布消息,但是只有消费者将队列绑定到该路由器才能收到消息,忽略Routing Key. 
Topic Exchange：主题匹配订阅,这里的主题指的是RoutingKey,RoutingKey可以采用通配符,如:*或#，RoutingKey命名采用.来分隔多个词,只有消息这将队列绑定到该路由器且指定RoutingKey符合匹配规则时才能收到消息; 
Headers Exchange:消息头订阅,消息发布前,为消息定义一个或多个键值对的消息头,然后消费者接收消息同时需要定义类似的键值对请求头:(如:x-mactch=all或者x_match=any)，只有请求头与消息头匹配,才能接收消息,忽略RoutingKey. 
默认的exchange:如果用空字符串去声明一个exchange，那么系统就会使用”amq.direct”这个exchange，我们创建一个queue时,默认的都会有一个和新建queue同名的routingKey绑定到这个默认的exchange上去

## 复杂与精简

在众多的MQ中间件中，首先学习Rabbitmq的时候，就理解他是一个单机的mq组件，为了系统的解耦，可以自己在业务层面做AKF

其在内卷能力做的非常出色，这得益于AMQP，也就是消息的传递形式、复杂度有exchange和queue的binding实现，这，对于P.C有很大的帮助























---

# RabbitMQ死信队列、延时队列分别是什么

## 死信队列

DLX（Dead Letter Exchange），**死信交换器**。

当队列中的消息被拒绝、或者过期会变成死信，死信可以被重新发布到另一个交换器，这个交换器就是DLX，与DLX绑定的队列称为死信队列。
造成死信的原因：

- 信息被拒绝
- 信息超时
- 超过了队列的最大长度

### 过期消息：

    在 rabbitmq 中存在2种方可设置消息的过期时间，第一种通过对队列进行设置，这种设置后，该队列中所有的消息都存在相同的过期时间，第二种通过对消息本身进行设置，那么每条消息的过期时间都不一样。如果同时使用这2种方法，那么以过期时间小的那个数值为准。当消息达到过期时间还没有被消费，那么那个消息就成为了一个 死信 消息。
    
    队列设置：在队列申明的时候使用 x-message-ttl 参数，单位为 毫秒
    
    单个消息设置：是设置消息属性的 expiration 参数的值，单位为 毫秒

## 延迟队列

延迟队列存储的是延迟消息

延迟消息指的是，当消息被发发布出去之后，并不立即投递给消费者，而是在指定时间之后投递。如：

在订单系统中，订单有30秒的付款时间，在订单超时之后在投递给消费者处理超时订单。

rabbitMq没有直接支持延迟队列，可以通过死信队列实现。

在死信队列中，可以为普通交换器绑定多个消息队列，假设绑定过期时间为5分钟，10分钟和30分钟，3个消息队列，然后为每个消息队列设置DLX，为每个DLX关联一个死信队列。

当消息过期之后，被转存到对应的死信队列中，然后投递给指定的消费者消费。



