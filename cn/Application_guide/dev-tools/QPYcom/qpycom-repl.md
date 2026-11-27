# QuecPython REPL 命令交互

## REPL调试

 *REPL*意为读取-求值-打印-循环(Read Evaluate Print Loop),是交互式提示的名称，可以通过在串口工具上访问交互式终端来进行REPL交互

在调试模组和设备的过程中，需要和模组进行通讯来执行指令进行调试，这个时候就需要用的REPL调试

通过REPL调试串口，可以输入任意的Python代码，并查看即时结果，这种编码方式有助于了解与实验 API 和库，并以交互方式开发要包含在项目中的工作代码，有助于提示开发和调试的效率。

下图为Python的REPL调试窗口：

![](../../media/dev-tools/qpycom/repl_py.png)

QuecPython的 REPL调试需要通过串口来交互，连接QuecPython模组的交互口即可开始REPL交互。

## 开启REPL调试

只需将已烧录QuecPython固件的模组连接电脑，通过串口工具连接交互口即可开始REPL串口交互调试

不同的串口工具连接串口的步骤不一致，以QPYcom为例，

![](../../media/dev-tools/qpycom/repl_qpycom.png)

1. 打开QPYcom工具，在交互页面选择模组的交互口并点击 `OPEN` 按钮
2. 工具的`REPL`页面，页面上会有有 `>>>`字符，在页面输入回车，会换行并出现新的`>>>`字符
3. 在页面输入Python代码即可进行调试，输入完指令后在按下回车后指令会输入到模组中执行并返回执行的结果

## 关闭REPL调试

当项目进入量产阶段时，出于设备的稳定运行和代码安全方面的考虑需要关闭设备的REPL调试功能，限制从串口访问设备的操作或者关闭交互串口的通讯，如果出厂后有返厂测试的需求只有通过特定的方式才能重新打开REPL调试

有以下两种方案

- 方案一：**使用UART接口**

通过`UART`接口将  **USB CDC PORT** 初始化成串口

```python
>>> # 创建uart对象
>>> from machine import UART
>>> uart1 = UART(UART.UART3, 115200, 8, 0, 1, 0)
```

完成初始化后，此时通过  **USB CDC PORT**  输入的数据将不再被REPL解析，通过该方式可以达到关闭交互口的效果，此时通过交互口输入的数据将作为串口数据被解析，使用教程参考[UART API使用手册](../../../api_reference/peripherals/machine.UART.md)

如果有关闭再重新打开的需求，使用`close`接口关闭上文创建的uart对象即可恢复REPL调试串口，以上方法通常用在产测流程中，需要在串口数据解析中增加对特定数据格式的数据进行解析，已达到指定方式可以重新打开REPL调试的串口的需求

- 方案二：**使用system接口**

使用`system`接口开启交互保护，设置开启交互保护后串口输入的所有外部指令以及代码都无法执行

```python
system.replSetEnable(flag，**kw_args)
```

开启后可以通过启动时设置的密码来关闭交互保护

详细使用教程参考 [system API使用手册](../../../api_reference/syslib/system.md)

## REPL原理

REPL交互的本质是串口交互，那我们的串口工具是怎么和模组进行通讯的呢

答案就是 pySerial库,这个包的使用非常简单,先安装pySerial。

想要用python操作串口，首先需要下载相关模块:

- pyserial (<https://pypi.org/project/pyserial/>)
- pywin32 (<https://pypi.org/project/pywin32/>)

### 通过pyserial操作串口

安装好库之后就可以通过python打开串口了

```python
import serial
ser = serial.Serial("com1",baudrate = 9600, bytesize = 8,parity = 'N',stopbits = 1,timeout=0.5) 
# winsows系统使用com1口连接
```

设备名对于不同的设备和操作系统是不一样的,对于模组而言这个com口的序号是计算机按照顺序分配的，所以我们一般通过串口设备的PID和VID来找到该设备的交互口

比如,在Windows系统上,你可以使用0,1等表示的一个设备来打开通信端”COM0”和”COM1”。 一旦端口打开,那就可以使用read() , readline() 和 write() 函数读写数据了

以下是通过pyserial操作串口示例

```python
ser.open()  # 打开端口
s = ser.read(10) # 从串口读10个字节
ser.write("hello")  # 向端口写数据
data = ser.read(20) # 从串口中读20个字符
data = ser.readline()  # 读一行，以\n结束，要是没有\n就一直读，阻塞。
ser.baudrate = 9600  # 设置波特率
ser.isOpen() # 看看这个串口是否已经被打开
ser.close()  # 关闭端口
```

### QuecPython串口操作原理

通过上面的pyserial库的介绍我们知道串口操作的原理，QuecPython也是通过这套机制来实现串口交互、远程代码执行以及下载文件等功能

当我们需要在模组内执行代码时，通过串口将想要执行的代码传输到模组中即可执行该代码，并可以通过串口读数据获取到执行结果，以下是写入示例

```python
ser.write(b"import uos\r\n")
ser.write(b"print(uos.listdir('/usr'))\r\n")   # 遍历usr目录下的文件
```

在执行完上述代码后，我们想要获取返回的结果，就要通过pyserial的read接口了，以下是读取示例

```python
num = ser.inWaiting()  # 获取串口缓存区的数据长度
data = ser.read(num） # 读取执行命令的返回结果
```

串口的执行模式为输入后立即执行并返回，所以当我们输入代码后会直接执行并返回结果，当我想要执行一段代码的时候，需要将完整的代码输入再执行并返回结果，这个时候就需要用到RAW模式

```python
ser.write(b"/x01")  # 进入raw模式
ser.write(b"import uos\r\n")
ser.write(b"print(uos.listdir('/usr'))\r\n")   # 遍历usr目录下的文件
ser.write(b"/x02")  # 退出raw模式
```

在raw模式内输入的代码不会立即执行，会在退出raw模式时一并返回执行结果

### 串口传输文件

传输文件就需要用到raw模式，在python中操作文件一般通过如下代码

```python
f = open(filename, "r")
f.write(data)
f.close()
```

那当我们需要将本地的文件通过串口传输到模组中时，只需要先在本地操作该文件，将要传输的文件内容读取出来

然后通过以上的命令将这些读出来的内容再输入到串口中即可，由于串口会执行这些命令，所以执行完毕之后则会将文件内容写入到模组之中

在整个传输过程中，其实传输的内容是python命令，通过执行命令的方式来传输文件，这些命令中包含了文件的内容，所以实际的传输效率会相对较低（实际传输的内容要大于文件本身），如果对于文件传输有速度要求可以参考QuecPython实现的其他传输协议[Ymodem](https://github.com/QuecPython/QuecPython_Ymodem)

除了传输文件以外，包括查看文件内容、更改文件名、删除文件、新建文件夹都可以通过传输命令的方式来实现





