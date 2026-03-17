### 一、无线通信基础

1. 2.4GHz ISM频段，2.400 GHz – 2.4835 GHz，BLE使用这个频段的波来传递信息。
2. 信道，将 83.5MHz 频段划分为40个信道，每个信道2MHz，分为3个广播信道和37个数据信道。
3. 带宽，一个信号占用的波的频率范围，BLE的带宽为2MHz。
4. GFSK，用于将数字信号变为无线电波，具体是用高频波表示1，用低频波表示0。
5. 连接，在底层意味着时间的同步，（ A 和 B 约定好：“我们每隔 20ms（连接间隔）见一次面。下次见面在第 5 号频道，再下次在第 12 号频道。”），以及身份识别，连接上之后会约定一个接入地址，以后每包数据都会加上这个地址。

### 二、BLE协议栈

##### 控制器部分 (Controller)

1. 物理层 (PHY, Physical Layer)，负责将数字信号转为无线电波发射出去，或者从空气中接收电波。
2. 链路层 (LL, Link Layer)，负责管理广播，扫描以及连接等，并控制何时休眠何时唤醒，降低功耗，代码实现使用状态机加定时器。

##### 中间接口 (HCI)

1. 中间接口 (HCI,Host Controller Interface)，用于 Host 和 Controller 之间的通信，本质上是HCI Transport Layer（物理/链路承载方式）和HCI 包格式（统一前缀 + 类型）。

##### 主机部分 (Host)

1. L2CAP (Logical Link Control and Adaptation Protocol)，负责拆包组包，处理底层或者上层发送的数据，另外负责管理多个逻辑通道。
2. ATT (Attribute Protocol)，定义Attribute，以便数据的操作。所有的 Service、Characteristic 和 Descriptor 最终都会被拆解成一条条连续排列的 Attribute。每一条 Attribute 都有自己的“行号”（Handle/句柄）。
3. GATT (Generic Attribute Profile)，定义了数据的结构。它使用 Service（服务）和 Characteristic（特征）来组织数据。
4. GAP (Generic Access Profile)，定义了设备的角色（广播者、扫描者、外设、中心设备），负责管理广播，扫描以及连接，开发者直接操作gap层，gap层内部调用链路层实现。

##### 应用层 (App)

### 三、BLE通信流程

1. Peripheral发送广播
2. Central扫描
3. 双方建立连接，规定连接参数
4. 传输数据

### 四、BLE 广播（Advertising）

1. 广播包的数据结构

   ```
   BLE 广播包
   ┌────────────┬──────────────┬────────────────────┐
   │ Header     │ MAC Address  │ Advertising Data   │
   │ 2 bytes    │ 6 bytes      │ 0~31 bytes         │
   └────────────┴──────────────┴────────────────────┘
   
   其中Advertising Data
   Length | Type | Value
   ```

2. 广播的参数

   | 参数      | 作用         |
   | --------- | ------------ |
   | conn_mode | 是否允许连接 |
   | disc_mode | 是否可发现   |
   | interval  | 广播间隔     |

### 五、GATT

1. GATT数据结构，Service、Characteristic 和 Descriptor 组成树状结构。

2. GATT通信方式

   | 操作     | 方向            | 用途     |
   | -------- | --------------- | -------- |
   | Read     | Client → Server | 读取数据 |
   | Write    | Client → Server | 写入数据 |
   | Notify   | Server → Client | 主动推送 |
   | Indicate | Server → Client | 确认推送 |

### 六、BLE开发实践

### 七、BLE安全机制