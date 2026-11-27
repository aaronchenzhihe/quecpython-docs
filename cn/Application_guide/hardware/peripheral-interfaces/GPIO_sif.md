# SIF通讯协议-一线通

 采用国际标准SIF通讯协议，接口通用、方便。 主从模式采用单线单向传输，即只需要一根传输线。 

## 协议介绍

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif_bus.png) 

SIF 协议满足以下一些特点：

- 一线通 SIF 协议比较简单，实现起来成本较低，适用于一些数据量不大且通讯品质要求不那么高的场景；
- 属于单线主从单工通讯模式，类似 1-Wire 协议但又与之有很大不同，为单工通讯的，即设备要么仅作为发送端，要么仅作为接收方，比如上面框图中 BMS 为发送发（主机），控制器和充电器仅为接收方（从机）；
- 波特率要么像 UART 一样提前固定好，要么让接收端自动适应发送方的同步信号从而来解析出匹配到的波特率；

一次传输一帧数据，每帧数据由 同步信号（发送主报文的前导信号） + 数据信号（8bit * 12个数据位的有效数据内容，按一定占空比进行发送）+ 结束信号（一帧完整的数据发送结束的标志信号）  3个部分组成，传输结束后要求线路空闲状态为低电平，每次传输需一次性完整传输所有数据；

数据的电平遵守 TTL 规范。

主机（BMS）的输出口是为下拉的，一般 IO 内部自带下拉也可以外部增加下拉电阻，而接收方的从机默认是需要被上拉的，上拉电阻值一般要求为 5V上拉 2.2K，3.3V 上拉 1K。



SIF逻辑电平信号定义如下：

Tosc定义： 250us < Tosc<2ms。 推荐值：500us

同步信号：

![1692688031272](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif_sync.png)

Bit(0)表示：

![1692688275482](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif-bit0.png)

Bit(1)表示：

![1692688405075](../../media/hardware/peripheral-interfaces/GPIO/bit1.png)

关于信号定义还有如下一些注意点：

- 32Tosc 范围为 0.5ms，考虑到响应速度，不要超过1ms
- 同步信号：>992Tosc 的低电平 + 32Tosc 的高电平，空闲位时间>15ms（0.5ms*992/32=15ms）
- 数据位逻辑 1 符合 高电平时间 > 低电平时间 + 0.5ms；
- 数据位逻辑 0 符合 低电平时间 > 高电平时间 + 0.5ms；
- 高低电平的比例一般使用 0.5ms 和 1ms 的比例，及占空比为 1:2 （适宜取 75%）



举例说明：

发送两字节数据：11000010，11000010（二进制）

![sif_example](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif_example.png)



## 功能概述

在QuecPython系列模组中，目前 EC600M/EC800M 支持该SIF从设备功能。



```python
import sif
sif.init(gpio, cb)	
```

GPIO引脚编号与物理映射关系请参考[machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin)

在通信模组系列中可用，因此请确保先导入它。然后您可以使用以下方法使用sif从机功能：

```python
import sif
def sif_cb(data):
    print("SIF data:",data)

sif.init(2, sif_cb)	
```

此处“2”表示SIF协议使用的是GPIO2.



## 设计

目前QuecPython仅支持SIF从机功能。

### 硬件

该部分说明基于具体的设备。 仅具有参考价值。

![SIF1](../../media/hardware/peripheral-interfaces/GPIO/GPIO_SIF1.png)

### 软件

用户接口设计

```python
class SifProtocol(gpio)
```

- 功能：创建SifProtocol对象
- 返回：SifProtocol对象
- SifProtocol：SifProtocol类
- gpio：单总线的GPIO编号。 详细请参考[machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin)



软件设计：

- 导入模块
  - 在QuecPython中，你可以使用`sif`模块来使用一线通SIF协议。
- 定义SifProtocol类
  - 创建一个SifProtocol类，其中包含创建等方法。

```python
import sif
import ubinascii


class SifProtocol():
    """This class is the protocol of xingheng SIF"""

    def __init__(self, gpio):
        """
        Args:
            gpio (int): SIF communiction gpio port
        """
        sif.init(gpio, self.__recv_sif_data_callback)

    """
    功能：sif初始化注册的回调函数，底层将一线通数据解析成字节数据传递给回调函数。在__parse_sif_data()函数中针对不通的厂家协议解析，获取电池电压、电流、温度等数据。
    """
    def __recv_sif_data_callback(self, data):
        print("SIF data: ", ubinascii.hexlify(data, ' '))
        self.__parse_sif_data(data)
    """
    功能：将数据根据协议解析数据。以比较常用的无锡团标协议为例，协议应采用小端（ Intel）模式的网络字节序来传递字和双字，所有传输均先发送低字节。
    """    
    def __parse_sif_data(self, data):
        if data != b'':
            try:
                if len(data) == 20 and data[0] == 1:  # public message
                    check_sum = sum(data[:-1])
                    if check_sum & 0xff == data[19]:
                        print("SIF public data sum check success")
                        self.__manufacturer = data[2]
                        self.__bat_type = data[3]
                        self.__battery_cell_material = data[4]
                        self.__rated_volt = (data[5] + (data[6] << 8)) / 10
                        self.__rated_capacity = (data[7] + (data[8] << 8)) / 10
                        self.__remain_capacity = data[9] / 2
                        self.__bat_volt = (data[10] + (data[11] << 8)) / 10
                        self.__electric_current = (
                            data[12] + (data[13] << 8)) / 10 - 500
                        self.__max_temperatiure = data[14] - 40
                        self.__min_temperatiure = data[15] - 40
                        self.__mos_temperatiure = data[16] - 40
                        self.__battery_fault = data[17]
                        self.__battery_work_state = data[18]
            except Exception as e:
                print("SIF receive data fault:", e)

```

实际的物理数据与总线传输的数据关系如下：

$$
P = \left ( C \times R \right ) +  F
$$

式中：
P：实际的物理数据；
C：总线传输的数据范围；
R：精度；
F：偏移量。

公有报文数据内容

| 信号名称         | 序号 | 长度   | 单位 | 偏移量 | 精度 | 范围     |
| ---------------- | ---- | ------ | ---- | ------ | ---- | -------- |
| 报文 ID          | 0    | 8 bit  | -    | 0      | 0    | -        |
| 协议版本         | 1    | 8 bit  | -    | 0      | 0    | -        |
| 电池厂商代码     | 2    | 8 bit  | -    | -      | -    | -        |
| 电池型号         | 3    | 8 bit  | -    | 0      | 1    | 0~15     |
| 电芯材料         | 4    | 8 bit  | -    | 0      | 1    | 0~15     |
| 额定电压         | 5    | 16 bit | V    | 0      | 0.1  | 0~1000   |
| 额定容量         | 7    | 16 bit | Ah   | 0      | 0.1  | 0~500    |
| 剩余电量 SOC     | 9    | 8 bit  | %    | 0      | 0.5  | 0~100    |
| 电池当前工作电压 | 10   | 16 bit | V    | 0      | 0.1  | 0~1000   |
| 电池当前工作电流 | 12   | 16 bit | A    | -500   | 0.1  | -500~500 |

