# Quick start

## Hardware preparation

- A Windows computer, recommended for the Win10 system.
- a set [EC800MCNLE / EC800MCNGB QuecDuino EVB](https://www.quecmall.com/goods-detail/2c90800c94028da20194824724370127) (Including antenna, Type-C data cable, etc).

- A Nano SIM card.
- A speaker with a power of 2-5W.

## Environment construction

- Download and install EC800M series module driver: [QuecPython_USB_Driver_Win10_ASR](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverA_Customer_V1.1.13.zip).

- Download and install  [VSCode](https://code.visualstudio.com/).

- Download and unzip [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip)。

- Download Firmware  
    - [EC800MCNLE](https://github.com/QuecPython/AIChatBot-Volcengine-webRTC/releases/download/v1.0.1/EC800MCNLER06A08M08_OCPU_QPY_TEST0219.zip)  
    - [EC800MCNGB](https://github.com/QuecPython/AIChatBot-Volcengine-webRTC/releases/download/v1.0.1/EC800MCNGBR06A01M08_OCPU_QPY_TEST0303.zip)  

- Download [Source code](https://codeload.github.com/QuecPython/AIChatBot-Volcengine-webRTC/zip/refs/heads/ec800m-quecduino)。

  > 💡 **Tips**
  > - The volcano dialogue token in the firmware is temporarily being tested and may be cancelled at any time. If you encounter difficulties in joining the room conversation, please contact the remote Fae personnel for support.
  > - Please follow the business process for commercial applications.

## Hardware connection

Connect the hardware according to the following diagram:

<img src="../../media/solutions/AIChatBot-Volcengine-webRTC/wire_connection.jpg" style="zoom:67%;" /> 

1. Connect the horn to the pins labeled 'SPK+' and 'SPK -' in the diagram.
2. Insert an available Nano SIM card into the position shown in the diagram.
3. Connect the antenna to the antenna connector marked with the word 'LTE'.
4. Connect the development board and computer using a Type-C data cable.

## Equipment development

### Power on

After completing the hardware connection work, long press the button labeled 'PWK' on the development board until the network light flashes or a COM port containing the words' Quectel USB 'appears in the port list of the computer device manager, indicating successful boot.

![comport.png](../../media/solutions/AIChatBot-Volcengine-webRTC/comport.png) 

### Burn firmware package

Refer to [this chapter](../../dev-tools/QPYcom/qpycom-dw.md#Download Firmware Script) Burn the corresponding firmware package to the development board.

### Script import and execution

1. Refer to [this chapter](../../../Getting_started/first_python.md#Writing the First Script File)  Import all files from the 'code' folder in the source code directory into the module file system, as shown in the following figure:

<img src="../../media/solutions/AIChatBot-Volcengine-webRTC/code_ai.png" style="zoom:80%;" />
<img src="../../media/solutions/AIChatBot-Volcengine-webRTC/code_ai_audio.png" style="zoom:80%;" />

2. Refer to [this chapter](../../../Getting_started/first_python.md#Executing the Script File) Execute the script file and execute the main program file `main.py`. (During normal startup, the module will automatically execute the `main.py` file)
3. Refer to [this chapter](../../../Getting_started/first_python.md#Stop-a-Running-Program), stop program running.

## Debugging

### Program startup

After executing the 'ai_main. py' script, the program starts running.

### Get into the room

Press the KEY S3 key to enter the room. Upon receiving the TIKTOK_STC_SVENT-START event, it indicates successful entry into the room and begins a voice conversation.

### Exit the room

Press the KEY S2 key to exit the room, and receiving the TIKTOK_STC_SVENT-STOP event indicates that the exit from the room is successful.

**Reference operation log:**

```python
import example
>>> example.exec('/usr/ai_main.py')
volume: 11
>>> lte network normal
ai task running

# push down KEY S3 enter
rtc_queue key event 1
start rtc
TIKTOK_RTC_EVENT_START
TIKTOK_RTC_EVENT_TTS_TEXT you
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

# push down KEY S2 to exit
rtc_queue key event 2
stop rtc
TIKTOK_RTC_EVENT_STOP
```

