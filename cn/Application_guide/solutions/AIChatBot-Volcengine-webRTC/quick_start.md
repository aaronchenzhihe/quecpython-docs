# 快速上手

## 硬件准备

- Windows 电脑一台，建议 `Win10` 系统。
- 一套 [EC800MCNLE / EC800MCNGB QuecDuino 开发板](https://www.quecmall.com/goods-detail/2c90800c9488358b01956e1f9cfb0257) （含天线、Type-C 数据线等）。

- 一张可正常使用的 Nano SIM 卡。
- 一个 2-5W 功率的喇叭。

## 环境搭建

- 下载并安装 EC800M 系列模组驱动：[QuecPython_USB_Driver_Win10_ASR](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverA_Customer_V1.1.13.zip)。

- 下载并安装 [VSCode](https://code.visualstudio.com/)。

- 下载并解压 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 工具到电脑的合适位置。

- 下载固件包。  
    - [EC800MCNLE](https://github.com/QuecPython/AIChatBot-Volcengine-webRTC/releases/download/v1.0.1/EC800MCNLER06A08M08_OCPU_QPY_TEST0219.zip)  
    - [EC800MCNGB](https://github.com/QuecPython/AIChatBot-Volcengine-webRTC/releases/download/v1.0.1/EC800MCNGBR06A01M08_OCPU_QPY_TEST0303.zip)  

- 下载[实验源码](https://codeload.github.com/QuecPython/AIChatBot-Volcengine-webRTC/zip/refs/heads/ec800m-quecduino)。

  > 💡 **Tips**
  > - 固件内火山对话 token 临时测试使用，随时可能取消，如遇到无法正常加入房间对话，请联系移远fae人员获取支持。
  > - 商业应用请走商务流程。

## 硬件连接

按照下图进行硬件连接：

<img src="../../media/solutions/AIChatBot-Volcengine-webRTC/wire_connection.jpg" style="zoom:67%;" /> 


1. 将喇叭连接至图中标识有`SPK+`和`SPK-`的排针上。
2. 在图示位置插入可用的 Nano SIM 卡。
3. 将天线连接至标识有`LTE`字样的天线连接座上。
4. 使用 Type-C 数据线连接开发板和电脑。

## 设备开发

### 开机

完成硬件连接的工作后，长按开发板上标识为`PWK`的按键，直到网络灯闪烁，或电脑设备管理器的端口列表中出现包含 `Quectel USB` 字样的 COM 口，表示开机成功。

![comport.png](../../media/solutions/AIChatBot-Volcengine-webRTC/comport.png) 

### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)，烧录对应型号固件包至开发板。

### 脚本导入与运行

1. 参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下 `code` 文件夹中的所有文件导入到模组文件系统，如下图所示：

<img src="../../media/solutions/AIChatBot-Volcengine-webRTC/code_ai.png" style="zoom:80%;" />
<img src="../../media/solutions/AIChatBot-Volcengine-webRTC/code_ai_audio.png" style="zoom:80%;" />

2. 参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件 `ai_main.py`。
3. 参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行。

## 业务调试

### 程序启动

执行 `ai_main.py` 脚本后，程序开始运行。

### 进入房间

按下 KEY S3 键进入房间，收到 TIKTOK_RTC_EVENT_START 事件表示进入房间成功，开始进行语音对话。

### 退出房间

按下 KEY S2 键退出房间，收到 TIKTOK_RTC_EVENT_STOP 事件表示退出房间成功。


**参考运行日志：**
```python
import example
>>> example.exec('/usr/ai_main.py')
volume: 11
>>> lte network normal
ai task running

# 按KEY S3键进入智能体
rtc_queue key event 1
start rtc
TIKTOK_RTC_EVENT_START
TIKTOK_RTC_EVENT_TTS_TEXT 你
TIKTOK_RTC_EVENT_TTS_TEXT 你好
TIKTOK_RTC_EVENT_TTS_TEXT 你好有
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么可
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么可以
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么可以帮
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么可以帮到
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么可以帮到你
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么可以帮到你的
TIKTOK_RTC_EVENT_TTS_TEXT 你好有什么可以帮到你的吗

# 按KEY S2键退出智能体
rtc_queue key event 2
stop rtc
TIKTOK_RTC_EVENT_STOP

```

