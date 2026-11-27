#  OneWire Protocol

## 协议介绍

1-Wire协议是一种单线接口、半双工、双向、低速低功耗、远距离串行数据通信协议。尽管该协议被归类为1-Wire标准，但在1-Wire标准总线中至少需要两根线 — 一根用于数据和/或电源供应，另一根用于地线回路。根据电源模式，可能需要额外的线路。

1-Wire标准具有主设备和从设备的配置，其中只能有一个主设备，即计算机或微控制器，以及多个从设备。最多可以在1-Wire标准总线上连接100个从设备。但是，随着从设备加入总线，主设备对其轮询可能需要更多时间。

该协议不使用时钟信号。相反，从设备是由主设备的信号进行内部时钟同步的。主设备完全负责从设备的读写操作，因此它们不能自行启动数据传输。但是，它们可以在主设备复位时通过总线指示它们的存在。每个主设备由一个64位地址标识，存储在每个1-Wire从设备的只读存储器（ROM）中。

这是一种低速串行通信标准，典型的数据传输速率为15.4 kbps。总线可以超载，达到最大数据速率为125 kbps。与其他标准的串行数据通信协议（如UART、I2C和SPI）相比，1-Wire协议的数据速率较低，但1-Wire总线在生产和操作中非常经济。它提供了简单的硬件实现和极低的功耗。

尽管硬件简单，但在微控制器端的软件实现非常复杂。尽管功耗很低，但它可以在相对长的距离上传输数据。

1-Wire协议被温度传感器、实时时钟、计时器、EEPROM以及流行的iButton等设备使用。这些1-Wire从设备大多是（现在的）Maxim Integrated公司的产品。

### 标准总线

1线标准总线至少有两根线。一个是数据线，另一个是接地回路。 主设备和从设备都与数据线开漏（开集电极）连接。这就是为什么通常会使用4.7K电阻将数据线上拉。对于1-Wire从设备，有两种可能的供电模式：寄生和常规。

在寄生模式下，只需将数据线和地线连接到1-Wire从设备。如果使用常规电源模式，则必须将附加的正电源线追踪到连接到总线的每个1-Wire从设备 。

 因此，PCB上的1-Wire总线可能有两根或三根线。具有三根线的常规供电的1-Wire总线更加可靠。

### 寄生电源与常规电源

1-Wire从设备可以以寄生和常规模式供电。所有1-Wire从设备都有三个引脚：VDD、GND和数据。在寄生模式下，VDD和GND引脚连接到地，因此信号和电源都通过同一条线（即数据线）提供给从设备。

 从设备内部带有800 pF的电容器，当数据线为高电平时，电容器会被充电。当数据线为低电平时，存储的电荷使从设备保持活动状态。数据线通常由4.7K电阻上拉。

寄生供电需要严格的定时和精确的供电规格，以保持从设备的稳定工作。这就是为什么这种模式不太可靠。通常，使用附加的强制上拉电阻来确保电源供应。

寄生供电：

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_parasitic_power.png)

带额外硬上拉寄生供电：

  ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_parasitic_power_with_pullup.png)

在传统的供电模式下，1-Wire从设备是通过外部电源供电的。每个1-Wire从设备都会引出一根附加的电线。从外部电源向从设备供电，确保其在恶劣的高温条件下也能安全运行。

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_conventional_powering.png)

典型的1线制设备在电压范围为-1.71~1.89V、1.71~3.63V、2.97~6.63V和2.97~5.25V之间工作。所吸引的电流范围在1.06~5mA之间变化。无论设备是以寄生电源还是传统电源供电，上拉电阻器设置电流水平。

### 工作原理

这种接口通常不会在微控制器或微型计算机中使用。通常是通过软件使用位操作或通用异步接收器-发送器（UART）来实现的。

数据线上的通信由主机使用复位来启动。  它将数据线拉低 480 us，然后释放，上拉电阻将数据线拉高。 从设备连接到总线，通过将数据线拉低 60~240 us 来响应复位信号。 如果数据线被从设备拉低，主设备会确认它们在总线上存在。经过60~240微秒后，从设备释放数据线，这样主设备就可以开始写入操作

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_one_wire_rest.png)

复位后，主设备可以与从设备进行数据的读写交互。起初，它发送ROM命令，比如搜索ROM命令（0xF0），以访问从设备的ROM地址。在读取了所有连接的1-Wire从设备的ROM地址后，主设备可以通过发送匹配ROM命令（0x55）来访问其中之一。ROM命令后面跟着功能命令。

例如，如果将1-Wire温度传感器连接到总线上，微控制器可以发送功能命令来启动温度转换、读取温度等。ROM和功能命令的长度都是8位。

由于1-Wire标准不使用任何时钟信号，'0'和'1'位的通信是通过在特定的时间槽内设置数据线的逻辑电平来实现的。通常，时间槽的长度为60微秒。每个时间槽之间有1微秒的间隔，以便数据线再次被上拉电阻拉高。在每个60微秒的时间槽内，主设备和从设备之间传输1个位。如果总线超载，时间槽的长度最多可以缩短10倍。

当主设备需要通过数据线写入位时，它会将数据线拉低。

- 要写入'0'，主设备会在整个60微秒的时间槽内将数据线拉低，然后在时间槽之间的1微秒间隔内释放它。
- 要写入'1'，主设备会在更短的15微秒内将数据线拉低，持续整个时间槽，然后在时间槽之间的1微秒间隔内释放它。

从设备会在大约半个时间槽的中间（即在60微秒时间槽中的30微秒处）产生脉冲。它们具有基本的单稳态多谐振电路来检测脉冲的持续时间。ROM和功能命令的长度都是8位。传输的数据也是以8位为一组进行的。误差检测通过8位循环冗余校验来执行。

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_one_wire_write.png)

主设备在发送ROM搜索或功能命令后从从设备读取数据。读取操作由主设备控制。主设备逐位从从设备读取数据，而数据以8位为一组传输到主设备。每个位在60微秒的时间槽内读取（如果总线超载，则时间槽可以更短）。

主设备将数据线拉低1微秒然后释放。然后，在15微秒后从总线上采样数据。如果从设备在总线上写入'0'，它会在整个60微秒的时间槽内保持数据线拉低，然后在时间槽之间的1微秒间隔内释放数据线。如果从设备在总线上写入'1'，它会在15微秒内保持数据线拉低，然后释放数据线，以便上拉电阻将数据线拉高。

主设备在15微秒后对每个位进行采样。如果从设备发送的位是'0'，则在采样时数据线会被拉低。如果从设备发送的位是'1'，则在采样时数据线会被拉高。

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_one_wire_read.png)

主设备可以在1线制标准总线上与多达100个从设备进行通信。然而，连接到总线上的1线制从设备数量越多，主设备从它们那里获取数据所需的时间就越长。软件库通常使用位操作或UART来计时脉冲持续时间。在1线制协议中，最低有效位（LSB）总是首先发送。



## 功能概述

在QuecPython系列模组中，目前 EC200U/EC600U/EG912U/EG915U 支持该功能。具体单总线相关API使用介绍：[machine.OneWire](../../../api_reference/peripherals/machine.OneWire.md)

### 对象创建

```python
class machine.OneWire(GPIOn)
```

GPIO引脚编号与物理映射关系请参考[machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin)

在通信模组系列中可用，因此请确保先导入它。然后您可以使用以下方法创建一个one-wire：

```python
one_wire = machine.OneWire(2)
```

此处“2”表示单总线协议使用的是GPIO2.

### 复位和检测设备

```python
OneWire.reset()
```

复位总线并检测设备。返回0设备正确响应。 -1 表示无设备响应



复位操作用于开始与从设备的通信。主设备会发送一个复位脉冲，然后等待从设备的响应。复位操作包括以下步骤：

- 主设备拉低数据线（DL）（也称为1-Wire线）至少480微秒，然后释放。
- 主设备等待一段时间，以便从设备在接收到复位脉冲后响应。从设备应在15微秒至60微秒内将数据线拉低作为应答信号。
- 主设备检测从设备的应答信号，从而确认设备存在。如果没有应答信号，表示通信失败。

### 写

```python
OneWire.write(buf)
```

写入操作用于向从设备中写入数据。写入操作是通过逐位传输数据实现的。写入操作的基本步骤如下：

- 主设备发送一个写入命令，指示从设备准备接收数据。
- 从设备从最低有效位（LSB）开始，逐位接收主设备发送的数据。
- 主设备在每个位的传输之后会等待一段时间，然后继续发送下一个位的数据。

### 读

```python
OneWire.read(len)
```

读取操作用于从从设备中读取数据。在1-Wire协议中，读取操作是通过逐位传输数据实现的。读取操作的基本步骤如下：

- 主设备发送一个读取命令，指示从设备开始传输数据。
- 从设备从最低有效位（LSB）开始，逐位将数据从内部发送到数据线上。
- 主设备在每个位的传输之后会等待一段时间，然后读取数据线上的电平。



## 设计

基于QuecPython EC600U模组驱动温度传感器DS18B20 介绍如何machine.OneWire的使用。

场景描述：

设计一个能够读取DS18B20温度传感器的类。DS18B20是一款广泛使用的数字温度传感器，适用于许多嵌入式应用。我们将创建一个`DS18B20Sensor`类，它将提供初始化、读取温度等功能。

### 硬件

准备材料：DS18B20温度传感器，杜邦线三根。

<img src="../../media/hardware/peripheral-interfaces/GPIO/GPIO_DS18B20.png" alt="DS18B20" style="zoom:67%;" />

首先正确连接硬件，DS18B20有三个引脚，分别是VCC供电，DQ单线传输数据，GND接地。按照下图连接硬件，DQ连接的位置在 EC600U 模组上对应的GPIO号为GPIO15。

![EVB_DS18B20](../../media/hardware/peripheral-interfaces/GPIO/GPIO_EVB_DS18B20.png)

### 软件

用户接口设计

```python
class DS18B20Sensor(onewire_pin)
```

- 功能：创建DS18B20对象
- 返回：DS18B20对象
- DS18B20Sensor：DS18B20Sensor类
- onewire_pin：单总线的GPIO编号。 详细请参考[machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin)



```python
DS18B20Sensor.read_temperature()
```

- 功能：读取温度



软件设计：

- 导入模块
  - 在QuecPython中，你可以使用`machine.OneWire`模块来使用单总线协议。
- 定义DS18B20Sensor类
  - 创建一个DS18B20Sensor类，其中包含创建，读取温度等方法。

```python
import machine

class DS18B20Sensor:
    def __init__(self, onewire_pin):
        self.onewire = machine.OneWire(onewire_pin)
        # Assuming only one DS18B20 on the bus
        self.device_address = self.onewire.scan()[0]

    def reset(self):
        self.onewire.reset()

    def write(self, data):
        self.onewire.write(data)

    def read(self, len=1):
        return self.onewire.read(len)

    def read_temperature(self):
        # Skip ROM command and Convert temperature command
        write_data1 = bytearray([0xcc, 0x44])

        # Skip ROM command and Read scratchpad command
        write_data2 = bytearray([0xcc, 0x44])

        self.reset()
        self.write(write_data1)

        self.reset()
        self.write(write_data2)

        data = []
        for _ in range(9):
            data.append(self.read())

        raw_temp = (data[1] << 8) | data[0]
        temperature = (raw_temp / 16) if (raw_temp & 0x8000 ==
                                          0) else ((raw_temp - 65536) / 16)
        return temperature


if __name__ == "__main__":
    ds18b20 = DS18B20Sensor(15)
    print("temperature = ", ds18b20.read_temperature())
```

>  该类可以扩展为支持多个DS18B20传感器，只需稍作修改，让初始化函数识别多个设备地址，并在读取温度时选择要读取的特定传感器。

 这个类提供了基本的功能，但在实际应用中可能需要更多的错误处理和参数配置。此外，为了保证精确性，还可以添加校验和校准功能。
