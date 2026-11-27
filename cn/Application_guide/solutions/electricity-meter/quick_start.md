# 快速上手



## 硬件准备

- Windows 电脑一台，建议 `Win10` 系统。
- 一根TypeC数据线
- 一张SIM卡
- 一块[QuecPython_EG91X系列C1-P02开发板](https://developer.quectel.com/doc/quecpython/Dev_board_guide/zh/eg91x-evb.html)
- 一个CP2102模块
- 一根天线




## 环境搭建

- 下载并安装 EG912U 系列模组驱动：[Quectel_Windows_USB_DriverU](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)
- 下载并安装[VSCode](https://code.visualstudio.com/)
- 下载并解压 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 工具到电脑的合适位置。
- 下载[EG912U固件包](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_BETA0002_EG912U_GLAA_FW.zip)
- 下载[实验源码](https://codeload.github.com/QuecPython/solution-electricity-meter/zip/refs/tags/v1.0.0)



## 硬件连接

按照下图进行硬件连接：

<img src="../../media/solutions/electricity-meter/connection1.png" style="zoom: 23%;" />	<img src="../../media/solutions/electricity-meter/connection2.png" style="zoom: 24%;" />	

1. 插入SIM卡
2. 将天线连接至标识有 `LTE` 字样的天线连接座上。
3. 连接EG912U开发板的串口和CP2102模块。
4. 连接CP2102模块和电脑
5. 使用Type-C数据线连接EG912U开发板



## 设备开发



### 开机

完成硬件连接的工作后，长按开发板上标识为`PWk`的按键，直到网络灯`net`闪烁，或电脑设备管理器的端口列表中出现包含`Quectel USB` 字样的 COM 口，表示开机成功。(EG91X系列C1-P02开发板上电自动开机)

![](../../media/solutions/electricity-meter/comport.png)	



### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)烧录固件包 [QPY_OCPU_BETA0002_EG912U_GLAA_FW.zip](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_BETA0002_EG912U_GLAA_FW.zip) 至EG912U模组开发板。

​	

### 脚本导入与运行

1. 配置`dev.json`中使用的TCP服务器和UART的具体参数

<img src="../../media/solutions/electricity-meter/TCPandUART.png" style="zoom: 100%;" />	

2. 参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下 `code`文件夹中的所有文件导入到模组文件系统，如下图所示：

<img src="../../media/solutions/electricity-meter/downloadfile.png" style="zoom: 72%;" />	

3. 参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件`demo.py`

4. 参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行



## 业务调试

### 程序启动

执行`demo.py`之后，程序会启动各个服务模块，并且连接上配置的TCP服务器

<img src="../../media/solutions/electricity-meter/start.png" style="zoom:70%;" >	



### 电表上传数据

这里使用串口助手发送信息模拟电表上传数据，串口助手发送16进制的RFC1662报文，内容为“Hello, I am Meter!”

> 🚩 **Warning**
> 发送的信息需要是代码中RFC1662协议规定的报文格式！
>
> 例如: 需要发送“Hello I am Meter!”，将其组装成报文格式后16进制表示为：
>
> 7E FF 03 00 21 00 12 BF 48 65 6C 6C 6F 20 49 20 61 6D 20 4D 65 74 65 72 21 9C 1F 7E 
>
> 再通过串口助手以16进制的方式发送给模组

<img src="../../media/solutions/electricity-meter/uart_send.png"  >	

模组收到电表发来的RFC1662报文后会先进行解析，再发送给TCP服务器：

<img src="../../media/solutions/electricity-meter/module_rfc.png" style="zoom:65%;"   >	

TCP服务器收到电表发来的信息：

<img src="../../media/solutions/electricity-meter/eletricity_to_tcp.png"  >	



### TCP下行数据

TCP服务器向模组发送信息“Hello，I am TCP_Server!”，模组收到后会先对信息进行RFC1662协议组包，再发送给电表

<img src="../../media/solutions/electricity-meter/uart_recv.png"  >	

> 🚩 **Warning**
>
> 图中发现信息前后出现乱码的现象，其实这并不是乱码，是因为模组发过来的是一整个RFC1662协议数据包，
>
> 而串口助手并没有解包能力，所以这是正常现象
