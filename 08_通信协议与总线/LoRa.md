## 一、LoRa的css

1. Chirp = 频率随时间变化的信号

2. LoRa 用不同的 Chirp 表示不同数据

## 二、LoRa的三大参数

1.SF（扩频因子）：不同扩频因子将一段chirp的一段频率切分成不同点，然后逐点扫过去。

SF7：128 点

SF8：256 点

SF9：512 点

…

SF12：4096 点

点数越多 → 扫得越慢 → 时间越长 → 越稳、越远

2.BW（带宽）：一个chirp所跨越的频率

3.CR（编码率）：

## 三、LoRaWAN 网络的三类核心设备

1.节点（Node），采集传感器数据，按 LoRaWAN 协议打包并发送。

2.网关（Gateway），接收无线 LoRa 信号 → 转成 IP 包 → 发送到网络服务器，或者接收网络服务器下行数据 → 转成 LoRa 信号 → 发送给节点

3.网络服务器（Network Server）

- 解包 LoRaWAN 数据
- 去重（如果多个网关同时收到数据）
- 解密和验证 MIC（消息完整性）
- 控制节点速率（ADR）
- 下发下行数据到节点

## 四、节点类型

1. 上行（Uplink）

- Node 采集数据 → 按 LoRaWAN 协议打包 → 发送
- Gateway 接收 LoRa 信号 → 转成 IP 数据 → 发送到 Network Server
- Network Server 去重、解密、验证 → 转发到 Application Server
- Application Server 存储/可视化/处理

2. 下行（Downlink）

- Application Server 发送控制命令 → Network Server
- Network Server 编包 → 下发到 Gateway
- Gateway 通过 LoRa 信号发送 → Node
- Node 接收并执行

> **Class A 节点下行窗口只在上行之后打开**
> **Class B 节点定时同步下行**
> **Class C 节点始终可接收下行**

## 五、入网方式

1.OTAA

（1）Node 发送 **Join Request**：包含 DevEUI（设备唯一ID）、AppEUI（应用ID）、AppKey（长期密钥）

（2）Network Server 验证合法性 → 回复 **Join Accept**

（3）Node 生成 **会话密钥（NwkSKey + AppSKey）**

（4）入网成功，开始正常上报数据

2.ABP：节点在出厂/配置时就写入会话密钥，不需要无线协商，直接加入网络。

## 六、帧结构

| 部分           | 描述                   | 长度    | 作用                                                |
| -------------- | ---------------------- | ------- | --------------------------------------------------- |
| **MHDR**       | Message Header         | 1 byte  | 指示帧类型（Join / Data / Ack）                     |
| **MACPayload** | 数据 + 控制信息        | 可变    | 包含 FRMPayload（用户数据）和 FCtrl/FCnt/DevAddr 等 |
| **MIC**        | Message Integrity Code | 4 bytes | 校验帧是否被篡改，保证完整性                        |
