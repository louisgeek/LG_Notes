

99 年：IBM 推出 MQTT 协议，最开始是用于通过卫星连接石油管道遥测系统，MQTT 中的 TT (Telemetry Transport) 就是源于这样一个遥测系统，10 年：MQTT 协议免费发布，14 年：MQTT 协议正式成为 OASIS 标准



MQTT 是基于 TCP 的应用层协议





针对两个痛点设计：

硬件性能低下 

网络状况不稳定的场景



在低带宽高延迟不可靠的网络下进行数据相对可靠传输的应用层协议，用于物联网数据传输、IM聊天软件等



消息上限

MQTT 的传输单位是消息，每条消息字节上限在 MQTT Broker 代理服务器上进行设置





QoS

- “正好一次” 用于计费系统和 IM App 推送中，能确保用户收到且只收到一次；





 提供了遗嘱消息和保留消息的特性





发布者和订阅者通过 broker 进行消息传递，相互之间感知不到对方的存在。当一个客户端断线时，整个系统可以继续工作





publisher 和 subscriber 不一定需要同时运行





MQTT 是一种基于二进制的协议，所谓基于二进制，**是指 MQTT 协议操作的元素是二进制数据而不是文本数据**

MQTT是不支持分包等机制，并不适宜一些数据包特别大的应用场景。





采用命令 & 命令确认的格式，每个命令消息都有一个关联的命令确认消息；



| **MQTT 消息结构**           | **描述**               | **长度**    |
| --------------------------- | ---------------------- | ----------- |
| 固定报头（Fixed header）    | 存在于所有 MQTT 消息中 | 2 到 5 字节 |
| 可变报头（Variable header） | 存在于部分 MQTT 消息中 | 0 或 N 字节 |
| 载荷（Payloads）            | 存在于部分 MQTT 消息中 | 0 或 N 字节 |

## 主题

$SYS

$SYS/brokers/+/clients/+/connected

$SYS/brokers/+/clients/+/disconnected



### 主题通配符

+

| **主题**    | 例子           |      |
| ----------- | -------------- | ---- |
| group/+/123 | group/temp/123 |      |
|             | group/qq/123   |      |
|             | group/wx/123   |      |







\#

| **主题** | 例子          |      |
| -------- | ------------- | ---- |
| group/#  | group         |      |
|          | group/123     |      |
|          | group/vip/123 |      |





## QoS 发布服务质量等级



### QoS 0 最多发一次

- At most once

### OoS 1   最少发一次

- At least once

  

### QoS2  正好发一次

- Exactly once

QoS是Sender和Receiver之间的协议，而不是Publisher和Subscriber之间的协议。换句话说，Publisher发布了一条QoS1的消息，只能保证Broker能至少收到一次这个消息；而对于Subscriber能否至少收到一次这个消息，还要取决于Subscriber在Subscibe的时候和Broker协商的QoS等级

 



- EMQ X ([www.emqx.com/zh/download…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.emqx.com%2Fzh%2Fdownloads%3Fproduct%3Dbroker))
- ActiveMQ/Artemis([activemq.apache.org/](https://link.juejin.cn/?target=https%3A%2F%2Factivemq.apache.org%2F))



