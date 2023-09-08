## 5 客户端配置

相对于 RocketMQ 的 Broker 集群，生产者和消费者是客户端。本节主要描述了生产者和消费者的常见行为配置。

### 5.1 客户端寻址模式

```RocketMQ``` 可以让客户端找到 ```Name Server```，再由 ```Name Server``` 找到 ```Broker```。以下展示了多种配置方式，按优先级从高到低排列，优先级高的配置会覆盖优先级低的配置。

- 在代码中指定 ```Name Server``` 地址，多个地址之间用分号分隔：

```java
producer.setNamesrvAddr("192.168.0.1:9876;192.168.0.2:9876");  

consumer.setNamesrvAddr("192.168.0.1:9876;192.168.0.2:9876");
```
- 在 Java 设置参数中指定 ```Name Server``` 地址：

```text
-Drocketmq.namesrv.addr=192.168.0.1:9876;192.168.0.2:9876  
```
- 在环境变量中指定 ```Name Server``` 地址：

```text
export   NAMESRV_ADDR=192.168.0.1:9876;192.168.0.2:9876   
```
- HTTP 静态服务器寻址（默认）

客户端启动后，将访问一个 http 静态服务器地址，例如：<http://jmenv.tbsite.net:8080/rocketmq/nsaddr>，该 URL 返回以下内容：

```text
192.168.0.1:9876;192.168.0.2:9876   
```
默认情况下，客户端每 2 分钟访问一次 HTTP 服务器，并更新本地的 Name Server 地址。该 URL 在代码中是硬编码的，您可以通过更新 ```/etc/hosts``` 文件来更改目标服务器，例如在 ```/etc/hosts``` 中添加以下配置：

```text
10.232.22.67    jmenv.taobao.net   
```
推荐使用 HTTP 静态服务器寻址，因为它可以简化客户端部署，并且 Name Server 集群可以进行热升级。

### 5.2 客户端配置

```DefaultMQProducer```、```TransactionMQProducer```、```DefaultMQPushConsumer```、```DefaultMQPullConsumer``` 都扩展了 ```ClientConfig``` 类，```ClientConfig``` 是客户端通用配置类。客户端配置采用 getXXX、setXXX 的风格，可以通过 Spring 进行配置，也可以在代码中配置。例如 ```namesrvAddr``` 参数：```producer.setNamesrvAddr("192.168.0.1:9876")```，其他参数也是类似的。

#### 1 客户端通用配置

| 参数名                              | 默认值          | 描述                                                         |
| ----------------------------- | ------- | ------------------------------------------------------------ |
| namesrvAddr                   |         | Name Server 地址列表，多个地址之间用分号分隔            |
| clientIP                      | 本地 IP   | 客户端本地 IP 地址，有些机器可能无法识别客户端 IP 地址，需要在代码中强制设置 |
| instanceName                  | DEFAULT | 客户端实例的名称，由客户端创建的多个生产者和消费者实际上共享一个内部实例（此实例包含网络连接、线程资源等） |
| clientCallbackExecutorThreads | 4       | 通信层异步回调线程数                                       |
| pollNameServerInteval         | 30000   | 以毫秒为单位的轮询 Name Server 的间隔                         |
| heartbeatBrokerInterval       | 30000   | 以毫秒为单位的心跳间隔，发送到 Broker                      |
| persistConsumerOffsetInterval | 5000    | 以毫秒为单位的持久化消费者消费进度间隔                 |

#### 2 生产者配置

| 参数名                              | 默认值          | 描述                                                         |
| ----------------------------- | ------- | ------------------------------------------------------------ |
| producerGroup                | DEFAULT_PRODUCER | 生产者组的名称。如果多个生产者属于同一个应用程序并发送相同的消息，则它们应该被分组到同一组中 |
| createTopicKey               | TBW102  | 发送消息时，如果服务器上不存在主题，则会自动创建主题，并指定一个可用于配置消息发送的默认路由的键 |
| defaultTopicQueueNums        | 4       | 发送消息和自动创建不存在于服务器上的主题时的默认队列数      |
| sendMsgTimeout               | 10000   | 以毫秒为单位的发送消息超时时间                               |
| compressMsgBodyOverHowmuch   | 4096    | 消息体开始压缩的大小（消费者会自动解压消息）。单位为字节  |
| retryAnotherBrokerWhenNotStoreOK | FALSE | 如果发送消息并返回 sendResult 但 sendStatus!=SEND_OK，则是否重发 |
| retryTimesWhenSendFailed     | 2       | 如果发送消息失败，最大重试次数，此参数仅对同步发送模式有效  |
| maxMessageSize               | 4MB     | 客户端限制的消息大小，超过此大小可能会出错。服务器也会进行限制，因此需要与服务器一起使用 |
| transactionCheckListener     |         | 事务消息的回查监听器，如果要发送事务消息，则必须设置此项 |
| checkThreadPoolMinSize       | 1       | Broker 回查 Producer 事务状态时线程池中的最小线程数        |
| checkThreadPoolMaxSize       | 1       | Broker 回查 Producer 事务状态时线程池中的最大线程数        |
| checkRequestHoldMax          | 2000    | 当 Broker 回查 Producer 事务状态时，生产者本地缓冲请求队列的大小 |
| RPCHook                      | null    | 创建 Producer 时传入的参数，包括消息发送前的预处理和消息响应后的处理。用户可以在第一个接口中进行一些安全控制或其他操作。|

#### 3 推送消费者配置

| 参数名                              | 默认值             | 描述

                                                         |
| ----------------------------- | ------------ | ------------------------------------------------------------ |
| consumerGroup                | DEFAULT_CONSUMER | 消费者组的名称。如果多个消费者属于同一个应用程序并订阅相同的消息并具有相同的消费逻辑，则它们应该被分组到同一组中 |
| messageModel                 | CLUSTERING       | 消息支持两种模式：集群消费和广播消费                         |
| consumeFromWhere             | CONSUME_FROM_LAST_OFFSET | 消费者启动后，默认从上次位置消费，包括两种情况：一是上次消费位置尚未过期，从上次位置开始消费；另一种是上次位置已过期，在当前队列的第一条消息处开始消费 |
| consumeTimestamp             | 半小时前           | 仅当 consumeFromWhere=CONSUME_FROM_TIMESTAMP 时才起作用       |
| allocateMessageQueueStrategy | AllocateMessageQueueAveragely | 实现 Rebalance 算法的策略                        |
| subscription                 |                 | 订阅关系                                                       |
| messageListener              |                 | 消息监听器                                                     |
| offsetStore                  |                 | 消费进度存储                                                   |
| consumeThreadMin             | 10              | 消费线程池中的最小线程数                                        |
| consumeThreadMax             | 20              | 消费线程池中的最大线程数                                        |
|                              |                 |                                                              |
| consumeConcurrentlyMaxSpan   | 2000            | 单个队列并行消费的最大跨度                                      |
| pullThresholdForQueue        | 1000            | 从本地队列缓存中拉取消息的最大数量                                |
| pullInterval                 | 0               | 拉取消息的间隔时间，因为是长轮询，所以为 0，但用于流量控制，可以将其设置为大于 0 的值，以毫秒为单位 |
| consumeMessageBatchMaxSize   | 1               | 批量消费消息                                                   |
| pullBatchSize                | 32              | 批量拉取消息                                                   |

#### 4 拉取消费者配置

| 参数名                              | 默认值             | 描述                                                         |
| ----------------------------- | ------------ | ------------------------------------------------------------ |
| consumerGroup                | DEFAULT_CONSUMER | 消费者组的名称。如果多个消费者属于同一个应用程序并订阅相同的消息并具有相同的消费逻辑，则它们应该被分组到同一组中 |
| brokerSuspendMaxTimeMillis   | 20000            | 长轮询，消费者拉取消息请求在 Broker 中挂起的最长时间，以毫秒为单位 |
| consumerTimeoutMillisWhenSuspend | 30000          | 长轮询，如果消费者拉取消息请求在 Broker 中挂起超过此时间值，客户端将认为超时。单位为毫秒 |
| consumerPullTimeoutMillis    | 10000            | 非长轮询，以毫秒为单位的拉取消息超时时间                     |
| messageModel                 | BROADCASTING   | 消息支持两种模式：集群消费和广播消费                         |
| messageQueueListener         |                 | 监听队列变化                                                   |
| offsetStore                  |                 | 消费调度存储                                                   |
| registerTopics               |                 | 注册主题的集合                                                   |
| allocateMessageQueueStrategy | AllocateMessageQueueAveragely | 实现 Rebalance 算法的策略                        |

#### 5 消息数据结构

| 字段名         | 默认值    | 描述                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| 主题           | null   | 必填，消息所属的主题名称                                         |
| 消息体         | null   | 必填，消息体                                                    |
| 标签           | null   | 可选，消息标签，方便服务器过滤。当前仅支持每条消息一个标签      |
| 键             | null   | 可选，表示此消息的业务键，服务器创建基于键的哈希索引。设置后，您可以在控制台系统中通过 ```Topics```、```Keys``` 找到消息。由于哈希索引，尽量使键具有唯一性，例如订单号、商品ID等。 |
| 标志           | 0      | 可选，完全由应用程序决定，RocketMQ 不会干预                   |
| 延迟级别       | 0      | 可选，消息延迟级别，0 表示无延迟，大于 0 表示可消费             |
| 等待存储消息   | TRUE   | 可选，表示消息在服务器宕机之前不会得到答复                     |
