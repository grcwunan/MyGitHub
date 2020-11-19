#### 消息中间件的5大核心组成：

- 协议
- 持久化机制
- 消息分发机制
- 高可用设计
- 高可靠设计

**协议三要素**：

- 语法
- 语义
- 时序

常用协议：OpenWire、AMQP（Active MQ/RabbitMQ，事务支持）、MQTT、Kafka（无事务设计）、OpenMessage（Rocket MQ）

#### ActiveMQ

完全支持JMS规范：点对点、发布\订阅模式；

支持的传输方式：nio、http、tcp、ssl、vm、udp；

协议：openwire



