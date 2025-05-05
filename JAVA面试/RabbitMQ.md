# RabbitMQ

---

## RabbitMQ架构设计

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-17/4583225925541.png?Expires=4898470165&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=m2VFe5zs9pQJe68mFocOVTSE7Rc%3D)

1. 生产者（Producer）： 生产者是消息的发送方，负责产生并发送消息到 RabbitMQ。生产者通常将消息发送到交换机（Exchange）。
2. 交换机（Exchange）： 交换机是消息的分发中心，负责将接收到的消息路由到一个或多个队列。它定义了消息的传递规则，可以根据规则将消息发送到一个或多个队列。
   * 直连交换机（Direct Exchange）： 将消息路由到与消息中的路由键（Routing Key）完全匹配的队列。
   * 主题交换机（Topic Exchange）： 根据通配符匹配路由键，将消息路由到一个或多个队列。
   * 扇出交换机（Fanout Exchange）： 将消息广播到所有与交换机绑定的队列，忽略路由键。
   * 头部交换机（Headers Exchange）： 根据消息头中的属性进行匹配，将消息路由到与消息头匹配的队列。
3. 队列（Queue）： 队列是消息的存储区，用于存储生产者发送的消息。消息最终会被消费者从队列中取出并处理。每个队列都有一个名称，并且可以绑定到一个或多个交换机。
4. 消费者（Consumer）： 消费者是消息的接收方，负责从队列中获取消息并进行处理。消费者通过订阅队列来接收消息。
5. 绑定（Binding）： 绑定是交换机和队列之间的关联关系。生产者将消息发送到交换机，而队列通过绑定与交换机关联，从而接收到消息。
6. 虚拟主机（Virtual Host）： 虚拟主机是 RabbitMQ 的基本工作单元，每个虚拟主机拥有自己独立的用户、权限、交换机、队列等资源，完全隔离于其他虚拟主机。
7. 连接（Connection）： 连接是指生产者、消费者与 RabbitMQ 之间的网络连接。每个连接可以包含多个信道（Channel），每个信道是一个独立的会话通道，可以进行独立的消息传递。
8. 消息： 消息是生产者和消费者之间传递的数据单元。消息通常包含消息体和可选的属性，如路由键等。

---

## RabbitMQ交换机类型

---

### direct(直连交换机)
作为默认交换机。路由键与队列名完全匹配交换机，此种类型交换机，通过RoutingKey路由键将交换机和队列进行绑定， 消息被发送到exchange时，需要根据消息的RoutingKey，来进行匹配，只将消息发送到完全匹配到此RoutingKey的队列。
比如：如果一个队列绑定到交换机要求路由键为“key”，则只转发RoutingKey标记为“key”的消息，不会转发"key1"，也不会转发“key.1”等等。它是完全匹配、单播的模式

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-17/4741147111708.png?Expires=4898470323&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=NboPPT9rvMjEAN5q2dn1Df7H8I8%3D)

---

### fanout(扇型交换机，广播)
Fanout，扇出类型交换机，此种交换机，会将消息分发给所有绑定了此交换机的队列，此时RoutingKey参数无效。
fanout类型交换机下发送消息一条，无论RoutingKey是什么，直接广播消息，queue1,queue2,queue3,queue4都可以收到消息。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-17/4789407779916.png?Expires=4898470371&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=v5LjpH8%2FU19%2BQN5VIt8Xas1eeNA%3D)

---

### topic(主题交换机)
Topic，主题类型交换机，此种交换机与Direct类似，也是需要通过routingkey路由键进行匹配分发，区别在于Topic可以进行模糊匹配，Direct是完全匹配。
1. Topic中，将routingkey通过"."来分为多个部分
2. "*"：代表一个部分
3. "#"：代表0个或多个部分(如果绑定的路由键为 "#" 时，则接受所有消息，因为路由键所有都匹配)

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-17/4854914778125.png?Expires=4898470437&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=jfNGndcggc28aBV8is0YLntymGM%3D)

---

### headers(头部交换机)
headers 匹配 AMQP 消息的 header 而不是路由键，此外 headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了
消费方指定的headers中必须包含一个"x-match"的键。
键"x-match"的值有2个
1. x-match = all ：表示所有的键值对都匹配才能接受到消息
2. x-match = any ：表示只要有键值对匹配就能接受到消息

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-17/4918447282166.png?Expires=4898470500&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=zXbRrMNCtYL8PWDtjJaNLbG9%2B%2BM%3D)

---

## RabbitMQ如何保证消息不丢失
整个流程中可能会出现三种消息丢失场景：
* 生产者发送消息到 RabbitMQ 服务器的过程中出现消息丢失。 可能是网络波动未收到消息，又或者是服务器宕机。
* RabbitMQ 服务器消息持久化出现消息丢失。 消息发送到 RabbitMQ 之后，未能及时存储完成持久化，RabbitMQ 服务器出现宕机重启，消息出现丢失。
* 消费者拉取消息过程以及拿到消息后出现消息丢失。 消费者从 RabbitMQ 服务器获取到消息过程出现网络波动等问题可能出现消息丢失；消费者拿到消息后但是消费者未能正常消费，导致丢失，可能是消费者出现处理异常又或者是消费者宕机。
针对上述三种消息丢失场景，RabbitMQ 提供了相应的解决方案，confirm 消息确认机制（生产者），消息持久化机制（RabbitMQ 服务），ACK 事务机制（消费者）

---

### confirm 消息确认机制（生产者）

---

### 消息持久化机制（RabbitMQ 服务）

---

### ACK 事务机制（消费者）
