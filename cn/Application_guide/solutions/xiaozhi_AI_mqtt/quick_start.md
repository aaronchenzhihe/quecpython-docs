# 快速上手

## 硬件准备

- Windows 电脑一台，建议 `Win10` 系统。
- 一套 [EC800MCNLE 开发板](https://www.quecmall.com/goods-detail/2c90800c9bdad914019ca242951203a8) （含天线、Type-C 数据线等）。

- 一张可正常使用的 SIM 卡。
- 一个 2-5W 功率的喇叭。

## 环境搭建

- 下载并安装 EC800M 系列模组驱动：[QuecPython_USB_Driver_Win10_ASR](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverA_Customer_V1.1.13.zip)。
- 下载并安装 [VSCode](https://code.visualstudio.com/)。
- 下载并解压 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 工具到电脑的合适位置。
- [下载固件包](../../media/solutions/xiaozhi_AI_mqtt/EC800MCNLER06A01M08_OCPU_QPY_TEST0810.zip)。  
- 下载[实验源码](https://codeload.github.com/QuecPython/AIChatbot-Xiaozhi-Mqtt/zip/refs/heads/main)。


## 硬件连接

按照下图进行硬件连接：

<img src="../../media/solutions/xiaozhi_AI_mqtt/connect.png" style="zoom:100%;" /> 


1. 在图示位置插入可用的 ` SIM` 卡。
2. 在图示的位置接入电池。
3. 使用 Type-C 数据线连接开发板和电脑。
4. 将天线连接至标识有`LTE`字样的天线连接座上。
5. 在图示的位置接入电池。将喇叭连接至图中标识有`SPK+`和`SPK-`的排针上。
6. 连接LCD屏(可选)。

## 设备开发

### 开机

完成硬件连接的工作后，电脑设备管理器的端口列表中出现包含 `Quectel USB` 字样的 COM 口，表示开机成功。

![comport.png](../../media/solutions/xiaozhi_AI_mqtt/comport.png) 

### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)，烧录对应型号固件包至开发板。

### 脚本导入与运行

1. 参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下 `src` 文件夹中的所有文件导入到模组文件系统，如下图所示：<img src="../../media/solutions/AIbox/src.png" style="zoom:80%;" />

2. 参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件 `_main.py`。

3. 参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行。

## 业务调试

### 程序启动

执行 `_main.py` 脚本后，程序开始运行。

### 激活小智

下图所示状态为待唤醒状态，开发板会出现红灯闪烁，需要语音“小智，小智”来唤醒小智AI进行语音对话。

<img src="../../media/solutions/xiaozhi_AI_mqtt/1.png" style="zoom:100%;" />

唤醒后会出现连接UDP服务器相关的数据或者是连接失败的提示，当出现连接失败时，请检查网络是否正常。

<img src="../../media/solutions/xiaozhi_AI_mqtt/2.png" style="zoom:100%;" />

当系统长时间未进行对话时，会收到goodbye消息，mqtt和udp都会主动关闭，程序自动进入待唤醒状态，等待下次使用唤醒词唤醒，再重新建立连接。

<img src="../../media/solutions/xiaozhi_AI_mqtt/3.png" style="zoom:100%;" />

