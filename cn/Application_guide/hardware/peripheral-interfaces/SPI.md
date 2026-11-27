# SPI

## SPI 概述

SPI，是英语Serial Peripheral interface的缩写，顾名思义就是串行外围设备接口。是Motorola首先在其MC68HCXX系列处理器上定义的。SPI接口主要应用在 EEPROM，FLASH，实时时钟，AD转换器，还有数字信号处理器和数字信号解码器之间。SPI，是一种高速的，全双工，同步的通信总线。SPI使用了主设备和从设备的概念。在任何给定的SPI通信中，有一个主设备和一个或多个从设备。主设备负责启动和结束通信会话。蜂窝模组SPI默认做主设备，外部设备做从设备，若需要蜂窝模组做从设备请联系厂家。

通用的SPI接口一般包含4根通讯线：

- SCK：主要的作用是主设备往从设备传输时钟信号, 控制数据交换的时机以及速率；
- SS/CS：用于主设备片选从设备，使被选中的从设备能够被主设备所访问；
- SDO/MOSI：在主设备上也被称为 Tx-Channel，作为数据的出口，主要用于 SPI 设备发送数据；
- SDI/MISO：在主设备上也被称为 Rx-Channel，作为数据的入口，主要用于SPI 设备接收数据；

SPI通信是全双工的，这意味着数据可以在两个方向上同时进行传输。对于SPI，这是通过MOSI（主设备输出，从设备输入）和MISO（主设备输入，从设备输出）线路实现的。

SPI共4种工作模式，由时钟极性（CPOL）和相位（CPHA）的不同配置决定，具体如下表所示：

|SPI Mode|CPOL|CPHA|
|-------|----|-----|
|0[00]  | 0  |  0  |
|1[01]  | 0  |  1  |
|2[10]  | 1  |  0  |
|3[11]  | 1  |  1  |

## 不同平台下SPI支持情况
BC25PA平台只支持SPI工作模式0、3，其他平台支持所有工作模式。

## QuecPython SPI API说明

### 构造函数
```python
class machine.SPI(port, mode, clk)
```
参数描述：
- port - 通道选择，int类型。选择通道前应查看WIKI，找到使用的SPI引脚对应通道。
- mode - SPI的工作模式。选择工作模式时应查看外部SPI通信芯片数据手册，查看芯片支持哪些模式，如MCP2515只支持0、3两种模式，此时只能选择模式0或者模式3。
- clk  - 时钟频率。不同平台支持最高和最低时钟频率均有差异，如ASR平台最高支持52MHz时钟频率，最低812.kHz。选择时钟频率时，应在小于外部SPI通信芯片的最高支持时钟频率的条件下，尽量选择更高频率，提高通信速率。

### SPI读接口
```python
SPI.read(recv_data, datalen)
```
该方法用于读取数据。
参数描述：
- recv_data - 读取数据的数组，bytearray类型。数据读取成功之后，读取到的数据将写入到recv_data中。
- datalen - 读取数据的长度，int类型。使用时注意datalen不能大于recv_data的实际长度。

返回值描述：
成功返回整型值0，失败返回整型值-1。

### SPI写接口
```python
SPI.write(write_data, datalen)
```
该方法用于写入数据。
参数描述：

- write_data - 写入的数据，bytes类型。
- datalen - 写入的数据长度，int类型。使用时注意datalen不能大于data的实际长度。

### SPI读写接口
```python
SPI.write_read(recv_data, data, datalen)
```
该方法用于同时写入和读取数据。SPI通信本质上是主设备和从设备进行数据交换，在每个 Clock 周期内，SPI 设备都会发送并接收一个 bit 大小的数据(不管主设备还是从设备)，相当于该设备有一个 bit 大小的数据被交换了。所以SPI设备在读取数据的同时也在写入数据，同理写入数据的时候也会读取数据。上面的SPI读接口和SPI写接口只是使用时忽略另一方的数据。
参数描述：

- recv_data - 读取数据的数组，bytearray类型。
- data - 发送的数据，bytes类型。
- datalen - 写入的数据长度，int类型。使用时注意datalen不能大于data的实际长度。


## SPI总线应用示例

### 和MCU交互
和MCU交互时，蜂窝模组做Master（主）设备，MCU做Slave（从）设备。使用前需要MCU端提供协议，协议中应当清楚定义蜂窝模组读写数据含义。此时MCU端类似普通的SPI外设，需提供寄存器的读写含义。当蜂窝模组对寄存器写入值时，MCU可以获取蜂窝模组的指令；MCU端可以自主更新寄存器（模拟）值，等待蜂窝模组读取并获取MCU数据。
示例如下：
1.MCU端虚拟寄存器表

|寄存器号|读写属性|寄存器数据长度|含义|
|-------|--------|-----|-----|
|0      | W      | 1|MCU GPIO1电平状态。写入1时MCU拉高GPIO1|
|1      | R      | 1|MCU 是否完成初始化|

2.定义协议如下
蜂窝模组端发送：0x7F(包头)+8位控制段（0：读指令，1：写指令）+ 8位寄存器号 + 8位数据长度 + N字节数据（读指令时为空）

当MCU端收到蜂窝模组数据后，且判断数据合法后，MCU端回复：
0x7E(包头) + 8位寄存器号 + 8位数据长度 + N字节数据（写指令时为空）

3.示例数据：
蜂窝模组 -> MCU：0x7F 0x00 0x01 0x01（读取寄存器1数据）
MCU -> 蜂窝模组：0x7E 0x01 0x01 0x01 (MCU完成初始化)


### SPI LCD显示屏
使用蜂窝模组连接SPI LCD共有两种方式，一种是专门的LCD SPI接口,一种是通用SPI。
蜂窝模组和LCD显示屏连接线如下图所示：
<img src="../../media/hardware/peripheral-interfaces/SPI/LCD.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- DOUT引脚用于串行数据的传输。它是液晶显示屏与外部设备（如控制器或微控制器）之间的双向数据通信线路。
- SCL/CLK引脚用于串行数据传输时的时钟同步。它提供了数据传输的时钟信号，确保数据的同步和正确传输。
- RS/DC引脚用于指示发送给液晶显示屏的数据类型，即数据或命令。当RS/DC引脚为低电平时，表示发送的是命令；当RS引脚为高电平时，表示发送的是数据。
- RST引脚用于复位液晶显示屏，将其恢复到初始状态。当RST引脚接收到复位信号时，显示屏将重新初始化，并清除之前的状态和数据。
- CS引脚用于选择或激活液晶显示屏的芯片。当CS引脚为低电平时，表示选择该芯片进行通信或操作。

当使用LCD SPI引脚与LCD连接时，RS、RST和CS脚只能选择硬件手册中的指定引脚，如EC600U系列中CS脚为65、RST脚为64、RS脚为63。LCD SPI接口初始化LCD如下：
```python
lcd.lcd_init(lcd_init_data, lcd_width, lcd_hight, lcd_clk, data_line, line_num, lcd_type, lcd_invalid, lcd_display_on, lcd_display_off, lcd_set_brightness)


```
当使用通用SPI引脚与LCD连接时，RS、RST和CS脚可根据需求任意选择合适引脚，但是引脚号需要在初始化中指定。如下：
```python
lcd.lcd_init(lcd_init_data, lcd_width, lcd_hight, lcd_clk, data_line, line_num, lcd_type, lcd_invalid, lcd_display_on, lcd_display_off, lcd_set_brightness, lcd_interface, spi_port, spi_mode, cs_pin, dc_pin, rst_pin)
```
和LCD SPI引脚相比，初始化参数中增加了SPI模式、SPI口以及CS、DC、RST引脚。

具体示例可参考[WIKI-LCD](../../../api_reference/peripherals/machine.LCD.md)中的参考代码。


### MCP2515 CAN控制器
市面上大部分CAN控制器都是使用SPI总线通信，其中MCP2515是应用最为广泛的一款，所以这里详细介绍如何使用Quecpython SPI模块和MCP2515进行通信。
模组和MCP2515连接示例图如下：
<img src="../../media/hardware/peripheral-interfaces/SPI/MCP2515.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

以下为MCP2515驱动中使用到SPI接口的代码
```python
# 初始化SPI，使用SPI端口0、工作模式0、6.5MHz（EC800M模组）
self.spi = SPI(0, 0, 3)

# 读单个寄存器值。INSTRUCTION.INSTRUCTION_READ：0x03（读指令）
# 参数： reg：寄存器号
def readRegister(self, reg):
    write_buf = bytearray([INSTRUCTION.INSTRUCTION_READ, reg, 0x00])
    read_buf = bytearray(len(write_buf))
    ret = self.spi.write_read(read_buf, write_buf, len(write_buf))
    if(ret != 0): 
        print("readRegister ret = ",ret)
    return read_buf[2]

# 读连续多个寄存器值
# 参数： reg：起始寄存器号，n：连续读取寄存器数目
def readRegisters(self, reg, n):
    write_buf = bytearray(2+n)
    write_buf[0] = INSTRUCTION.INSTRUCTION_READ
    write_buf[1] = reg
    read_buf = bytearray(len(write_buf))
    ret = self.spi.write_read(read_buf, write_buf, len(write_buf))
    if(ret != 0): 
        print("readRegisters ret = ",ret)
    return read_buf[2:]

# 写单个寄存器值 INSTRUCTION.INSTRUCTION_WRITE：0x02（写指令）
# 参数： reg：寄存器号，value:写入寄存器值
def setRegister(self, reg, value):
    write_buf = bytearray([INSTRUCTION.INSTRUCTION_WRITE, reg, value])
    ret = self.spi.write(write_buf, len(write_buf))
    if(ret != 0): 
        print("setRegister ret = ",ret)

# 写连续多个寄存器值 INSTRUCTION.INSTRUCTION_WRITE：0x02（写指令）
# 参数： reg：寄存器号，values:写入寄存器值（bytearray）
def setRegisters(self, reg, values):
    write_buf = bytearray([INSTRUCTION.INSTRUCTION_WRITE, reg]) + values
    ret = self.spi.write(write_buf, len(write_buf))
    if(ret != 0): 
        print("setRegisters ret = ",ret)
```


## SPI总线常见问题和故障排查

1.**硬件连接问题**: SPI接口需要四条线 (MISO、MOSI、SCLK和CS) 进行连接，如果这些线的连接不正确，可能导致通信失败。

2.**电源和地问题**: 如果SPI设备的电源和地没有正确连接，也可能导致通信问题。

3.**SPI模式错误**: SPI有四种模式，由时钟极性和相位决定。如果主设备和从设备的SPI模式不匹配，可能会导致通信错误。

4.**时钟频率过高**:如果SPI的时钟频率设置得过高，可能会超过设备的能力，导致数据传输错误。

5.**数据位数不匹配**: SPI的数据通常是8位或16位。如果发送和接收的数据位数不匹配，可能会导致数据错误。

6.**片选线控制错误**: 在多从设备的系统中，如果片选线没有被正确控制，可能会导致通信混乱。

7.**电平不匹配问题**: 如果主设备和从设备的电平(例如，3.3V和5V)不匹配，可能会导致通信问题或设备损坏。

8.**设备兼容性问题**: 由于SPI没有官方标准，不同设备的实现可能会有差异。这可能导致设备之间的兼容性问题。

9.**线路噪声和干扰**: 如果SPI线路过长，或者环境中有高频噪声，可能会影响SPI的信号质量导致通信错误。



