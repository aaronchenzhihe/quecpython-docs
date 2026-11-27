# IIC

## IIC 概述

IIC(Inter-IntegratedCircuit)总线是一种由PHILIPS公司开发的两线式串行总线，用于连接微控制器以及其外围设备，IIC也被称为I2C，其实两者是完全相同的，只是名词不一样而已。
它是由数据线SDA和时钟线SCL构成的串行总线，可发送和接收数据。IIC 接口使用主机和从机的概念，通常一个主机会搭配一个或多个从机，主机负责启动和结束通信会话，QuecPython 支持的蜂窝通信模组目前仅做主机。
 
IIC拓扑结构如下：
<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_topology.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- 数据线SDA：用来传输数据;时钟线SCL：用来同步数据收发。
- 总线上每一个器件都有一个唯一的地址识别，所以我们只需要知道器件的地址，根据时序就可以实现微控制器与器件之间的通信。
- 数据线SDA和时钟线SCL都是双向线路，都通过一个电流源或上拉电阻连接到正的电压，所以当总线空闲的时候，这两条线路都是高电平。
- 总线上数据的传输速率在标准模式下可达100kbit/s，在快速模式下可达400kbit/s，在高速模式下可达3.4Mbit/s

## IIC 通信时序介绍
- **空闲电平**：SCL和SDA均为高电平
- **起始信号时序**：SCL为高电平（空闲）时，SDA由高电平到低电平的跳变
<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_start.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- **结束信号时序**：SCL为高电平（空闲）时，SDA由低电平到高电平的跳变
<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_stop.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- **应答信号时序**：接收端收到有效数据后向对方响应的信号，发送端每发送一个字节(8位)数据，在第9个时钟周期释放数据线去接收对方的应答。
当SDA是低电平为有效应答(ACK)，表示对方接收成功；当SDA是高电平为无效应答(NACK)，表示对方没有接收成功。

- **数据传输时序**：SCL低电平时，允许数据位变化。每8Bits数据传输完成后，从机将拉低SDA线，回传主机一位ACK信号；或拉高SDA线，回传主机一位NACK信号。
<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_time_sequence.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

## 不同平台下IIC支持情况
所有模组都支持硬件IIC，且都支持100k和400k两种频率；

## 软件模拟IIC
软件模拟IIC是一种使用普通的GPIO引脚，通过软件控制引脚的电平变化，来模拟IIC协议的通信方式。
当芯片自带的IIC个数无法满足需求或者所用引脚被其他功能占用时，可以使用软件模拟IIC接口来实现功能。

当前部分模组可支持模拟IIC，频率可在1~100k范围内设置，具体设置频率方法见[WIKI-I2C_simulation](../../../api_reference/peripherals/machine.I2C_simulation.md)。
支持模拟IIC模组如下：
- EC600M-CN系列
- EC800M-CN系列
- EC600U系列 
- EG915U系列
- EC200U系列
- EC200A系列
- EC600E-CN系列
- EC800E-CN系列
- BG95系列
- BC25系列
- EG912N-EN系列
- EG915N系列
- EC600N-CN系列
- EC800N-CN系列


## 功能概述

主要介绍如何使用 IIC 驱动程序的功能和数据类型在 QuecPython系列模组 与其他 IIC 设备之间建立通信。典型的编程工作流程分为以下几节：

1. [创建对象](#创建对象)
2. [发送数据](#发送数据) 
3. [接收数据](#接收数据)

详细API介绍请参考[machine.IIC](../../../api_reference/peripherals/machine.I2C.md)

### 创建对象

创建IIC对象时，需要设置IIC通道和工作模式。

```python
class machine.I2C(I2Cn, MODE)
```

参数介绍以及引脚对应关系请参考[machine.IIC](../../../api_reference/peripherals/machine.I2C.md)

创建对象时，需注意以下几点：

- 在一个项目中，一个通道只能创建一次对象，当使用一个通道的IIC与多个IIC外设芯片通信时只需在其中一个外设驱动中进行创建IIC对象。
- 标准模式下IIC通信波特率为100k，快速模式下IIC通信波特率为400k，注意设置波特率不能高于IIC外设芯片最高能支持波特率。


### 发送数据
```python
I2C.write(slaveaddress, addr, addr_len, data, datalen)
```
发送IIC数据时，需注意以下几点：
- slaveaddress为从机地址，是一个7bit的数值。在主机发送从机地址时会将slaveaddress值左移一位，当向从机写入数据时最低位补0，当读取从机数据时
最低位补1。例如某IIC外设芯片从机地址为0x23,当向外设芯片发送数据时主机应发送0x46,当读取外设芯片数据时主机应发送0x47。
- addr_len为寄存器地址长度，寄存器地址长度可从外设芯片数据手册中确认，目前经常用到的IIC外设芯片的寄存器地址长度一般是一个字节，如QMA7981三轴传感器芯片。
<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_reg_lenth.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

API介绍请参考[machine.UART.write](../../../api_reference/peripherals/machine.UART.md#uart.write)

### 接收数据
```python
I2C.read(slaveaddress, addr, addr_len, r_data, datalen, delay)
```
接收数据，需要注意以下几点：

**1. 缓冲区管理：** 正确管理接收缓冲区是非常重要的。如果在数据还没有读取完成就被新的数据覆盖，可能会导致数据丢失。因此，需要确保在数据被新的数据覆盖前能够及时读取和处理。  

**2. delay参数设置：** 由于部分IIC外设芯片收到主机写数据指令后需要延时才能正常返回数据，所以在模组（主机）发送完写指令后延时一段时间再读取IIC外设芯片数据，如图：
<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_read_delay.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

API及参数含义介绍请参考[machine.UART.read](../../../api_reference/peripherals/machine.UART.md#uart.read)


## 应用案例
### 三轴传感器QMA7981
QMA7981是一款单芯片三轴加速度计，集成了加速度传感器与信号调理ASIC，可以检测倾斜，运动，冲击和振动。
可应用于检测屏幕旋转，步数计数，睡眠质量，移动和可穿戴智能设备中的游戏和个人导航等。
QMA7981有以下特点：
- 结合定制设计的14位ADC ASIC，提供低噪声，高精度，低功耗和零点校准的优势。
- 内置自检功能，可以在生产过程中进行低成本的功能测试。
- 内置运动算法，支持步数计数，睡眠质量监测，游戏和个人导航等应用。
- 工作电压范围为1.71V至3.6V，工作温度范围为-40°C至+85°C。

#### QMA7981原理图
<img src="../../media/hardware/peripheral-interfaces/IIC/QMA7981_1.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

QMA7981使用时需注意如下几点：

 - QMA7981有一路IIC，模组可通过IIC获取QMA7981数据。
 - QMA7981最多支持两路外部中断，且可以设置高电平中断有效或者低电平中断有效。注意设置QMA7981中断电平配置需要和模组这边外部中断检测对应，如：设置QMA7981中断有效电平为高电平，则模组外部中断引脚需要设置为低电平，上升沿触发。
```python
self.extint = ExtInt(INT1, ExtInt.IRQ_RISING, ExtInt.PULL_PD, self.ext_cb)
data = self._read([self.INTPIN_CONF_ADDR], 1)
# set logic high as active level for INT1 pin
data[0] |= 0x01
print('INTPIN_CONF_ADDR: {}'.format(data))
self._write([self.INTPIN_CONF_ADDR], [data[0]])
```
- 当AD0接地时 IIC从机地址为0x12，当AD0接到VDDIO时从机地址为0x13。
- QMA7981有两种工作模式：待机模式（Standby Mode）和激活模式（Active Mode）。QMA7981上电或者重启之后默认进入待机模式，可以通过设置PM寄存器（0x11）值进入激活模式。
<img src="../../media/hardware/peripheral-interfaces/IIC/QMA7981_PM_REG.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- 当QMA7981水平静置条件下，由于地心引力的影响，Z轴会有一个g的加速度，X、Y轴为0。
QMA7981详细驱动可参考[QMA7981](https://github.com/QuecPython/examples/tree/main/hardware/peripheral-interfaces/IIC/qma7981.py)


### 温湿度传感器AHT10
AHT10 配有一个全新设计的 ASIC专用芯片、一个经过改进的MEMS半导体电容式湿度传感元件和一个标准的片上温度传感元件。主要引用场景有：
暖通空调 、除湿器、测试及检测设备、消费品、汽车 、自动控制、数据记录器、气象站、家电、湿度调节、医疗及其他相关温湿度检测控制。

AHT10工作流程如下：
- 给传感器上电，上电后传感器最多需要20ms时间（此时SCL为高电平)以达到空闲状态即做好准备接收由主机 (蜂窝模组)发送的命令。
- 主机通过IIC与AHT10通信，AHT10的从机地址为0x38。在主机发出初始化 命令之后（‘11100001’代表初始化，‘10101100’代表温湿度测量）， 主机必
须等待测量完成。基本的命令在下表中进行概述。

<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_cmd.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

下表为从机返回的状态位说明。

<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_state.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- 主机发送采集指令（0xAC）触发AHT10采集温湿度信息，延时75ms以上，等待AHT10完成采集之后再读取AHT10温湿度数据。
```python
def Trigger_measurement(self):
    # Trigger data conversion
    self.write_data([self.AHT10_START_MEASURMENT_CMD, 0x33, 0x00])
    time.sleep_ms(200)  # at last delay 75ms
    # check has success
    r_data = self.read_data(6)
    # check bit7
    if (r_data[0] >> 7) != 0x0:
        print("Conversion has error")
    else:
        self.aht10_transformation_temperature(r_data[1:6])
```

- 主机获取AHT10数据后，需要对原始数据进行转化，得到需要的温湿度信息。**相对湿度 RH** 根据 SDA 输出的相对湿
度信号 SRH 通过如下公式计算获得
<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_calculate_humidity.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">



**温度 T**可以通过将温度输出信号 ST 代入到下面的公式计算得到
<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_calculate_temp.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

```python
def aht10_transformation_temperature(self, data):
    r_data = data
    #　Convert raw data into temperature and humidity
    humidity = (r_data[0] << 12) | (
        r_data[1] << 4) | ((r_data[2] & 0xF0) >> 4)
    humidity = (humidity/(1 << 20)) * 100.0
    print("current humidity is {0}%".format(humidity))
    temperature = ((r_data[2] & 0xf) << 16) | (
        r_data[3] << 8) | r_data[4]
    temperature = (temperature * 200.0 / (1 << 20)) - 50
    print("current temperature is {0}°C".format(temperature))
```

AHT10详细驱动可参考[AHT10驱动](https://github.com/QuecPython/examples/tree/main/hardware/peripheral-interfaces/IIC/aht10_test.py)

## 常见问题和故障

IIC是一种非常简单和直接的通信协议，在实际应用中遇到的问题也比较少。


**1. 数据接收或者接收一直失败**

这可能是由于几种原因引起的，包括：波特率设置错误、硬件连接问题等。故障排查步骤可能包括：

- 检查模组IIC波特率是否超过外设芯片最高波特率限制。
- 检查硬件连接，确保SDA和SCL线正确连接，并且地线（GND）也要正确连接。


**2. 可以正常进行收发，但是有错误数据**

可能是以下几个原因：
- 主机SDA、SCL和从机SDA、SCL电平不一致。当主机电平为3.3V，从机为1.8V，此时从机的高电平是1.8V，但是主机一般认为电平高于1.65V是高电平，容易产生误判。
- IIC上拉电阻过大，上拉能力不够，导致上升沿和下降沿持续时间过长，主机判断数据错误。


