# CAN：bxCAN 与 FDCAN

## 1. 先确认 CAN 类型

不要把 CAN 当成所有 STM32 都相同的外设。先确认完整 MCU 型号、外设实例和当前 HAL 驱动：
- bxCAN：常见于部分较早 STM32 系列。
- FDCAN：用于支持该外设的部分较新 STM32 型号。

F103/F407/G474 不能仅凭系列名称假定使用同一种 CAN 外设。必须检查完整型号、Reference Manual、设备头文件和 HAL 驱动。

bxCAN 与 FDCAN 的句柄、初始化结构、过滤器、FIFO、Message RAM、通知和 HAL API 不可直接互换。

## 2. 先核对板级收发链路

```text
MCU CAN_TX / CAN_RX
       ↓
CAN 收发器（可能带隔离）
       ↓
CANH / CANL
       ↓
总线连接器与终端电阻
```

核对：
- MCU 引脚复用和电平。
- 收发器完整型号、供电和逻辑侧电平。
- 收发器 TXD/RXD 方向。
- Standby/Enable 控制脚。
- 隔离器和隔离侧电源（若有）。
- CANH/CANL 接线。
- 总线终端电阻和拓扑。
- 共模范围、ESD/浪涌保护和地参考设计。

MCU 的 CAN 外设不能直接连接 CANH/CANL，除非芯片本身明确集成物理层收发器；通常需要外部收发器。

## 3. 位时序和波特率

必须核对 CAN 外设时钟来源、预分频和时间段参数。

对 bxCAN，常见位时间关系为：

```math
f_{\mathrm{bit}} =
\frac{f_{\mathrm{CAN}}}
{\mathrm{Prescaler}\,(1+\mathrm{BS1}+\mathrm{BS2})}
```

还需检查 Sync Segment、SJW、采样点和传播延迟余量。具体参数的寄存器编码和 HAL 字段含义，以当前 HAL 与 Reference Manual 为准。

FDCAN 的时钟来源、Nominal/Data Bit Timing、Prescaler、TSEG1/TSEG2、SJW 和数据相位配置需按对应 FDCAN 实现核对。不要直接套用 bxCAN 的配置结构。

不能只看 CubeMX 中显示的波特率；应从实际外设时钟和时间段参数反算，并确认网络中所有节点的位速率和采样点兼容。

## 4. bxCAN 配置重点

按当前芯片和 HAL 版本核对：
- Normal/Loopback/Silent 等模式。
- Prescaler、BS1、BS2、SJW。
- 自动总线关闭管理、自动重传、FIFO 锁定等选项。
- 过滤器 Bank、Filter Mode、Scale、FIFO 分配。
- 标准帧/扩展帧过滤条件。
- FIFO 中断通知。
- 接收回调与 FIFO 读取 API。
- 发送邮箱和发送完成/错误状态。

不要从另一个 F1/F4 项目复制过滤器 Bank 分配和 API 后假定可用。

## 5. FDCAN 配置重点

按当前型号和 HAL 版本核对：
- FDCAN Kernel Clock。
- Nominal Bit Timing。
- Data Bit Timing（若使用 CAN FD）。
- Classic CAN / FD / BRS 模式。
- Standard/Extended Filter 数量与类型。
- Rx FIFO0/FIFO1、Rx Buffer、Tx FIFO/Queue 的元素数量和元素大小。
- Message RAM 分配及实例之间的资源占用。
- 过滤器路由和全局过滤设置。
- 中断线、通知和回调。
- Tx Event FIFO（若使用）。

FDCAN 的 Message RAM 不是可以忽略的普通缓冲区配置。元素数量和数据字段大小必须与代码访问方式一致。

## 6. 接收与发送流程

建议把底层外设初始化、通用收发/缓存和项目业务分开：
- `Core/`：CubeMX 生成初始化和 IRQ 分发。
- `App/`：通用帧收发、缓存、数据打包与解析。
- `Control/`：项目决定何时发送、报文 ID 的业务含义、接收后采取什么控制动作。

接收中断/回调中通常只完成必要的取帧和缓存，不在 ISR 中运行复杂控制算法、等待或阻塞发送。

## 7. CAN 报文协议

明确记录：
- 标准 ID 或扩展 ID。
- 数据帧/远程帧等类型（按控制器支持情况）。
- DLC 和真实 payload 长度。
- 字节序。
- 整数符号、缩放系数和单位。
- 浮点数据格式与字节序。
- 报文周期、超时定义和发送优先级。
- 节点地址、启动同步和故障状态约定。

不要只在代码里用裸数字表示 ID 和数据偏移。协议定义要能被其他节点独立理解。

## 8. 接收缓存和并发

如果 ISR 生产、主循环消费：
- 明确队列是单生产者/单消费者还是多生产者。
- 明确队列满时是丢新帧、覆盖旧帧还是记录溢出。
- 明确读写索引的更新顺序和原子性。
- `volatile` 不等于线程安全。
- 不为简单场景引入没有必要的 RTOS 队列或复杂消息总线。

## 9. 常见故障定位

| 现象 | 优先检查 |
| --- | --- |
| 无法进入 Bus On/Start | 外设初始化、时钟、模式、过滤器和 HAL 启动返回值 |
| 发送邮箱/队列一直忙 | 总线连接、收发器使能、波特率、ACK、总线状态 |
| 能发送但收不到 | RX 引脚、收发器、过滤器、FIFO、通知和 IRQ |
| 偶发丢帧 | FIFO 溢出、回调处理时间、队列容量和消费者处理速度 |
| 错误帧较多 | 位时序、采样点、终端电阻、拓扑、线缆和信号完整性 |
| 只有部分节点通信 | 各节点波特率、标准/扩展 ID、过滤器和收发器状态 |

错误计数器和 HAL 返回状态要从当前控制器和驱动读取，不根据其他系列的字段名猜测。
