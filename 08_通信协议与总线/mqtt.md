## 一、发布订阅模型

MQTT 模式（Pub/Sub）

```
Publisher → Broker → Subscriber
```

核心思想：

- 发布者（Publisher）不关心谁接收
- 订阅者（Subscriber）不关心谁发送

## 二、topic

类似于网址

```
home/livingroom/temperature
```

## 三、QoS（服务质量）

1.QoS 0：最多一次，发送一次不用确认

2.QoS 1：至少发送一次，没有收到确认就一直发

3.QoS 2：恰好一次，双向确认，四次握手

## 四、保留消息和遗嘱消息

保留消息，broker记住最后一条消息并发送给新订阅的客户端

遗嘱消息，当客户端异常断开时，broker发送一条提示消息

## 五、心跳机制

每隔一段时间发送ping保证连接

## 六、报文结构

```text
MQTT 报文 = Fixed Header + Variable Header + Payload
```

## 七、三大报文作用

- CONNECT → 建立连接 + 配置
- PUBLISH → 发送数据
- SUBSCRIBE → 注册接收规则

