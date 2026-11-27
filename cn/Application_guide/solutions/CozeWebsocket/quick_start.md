# 快速上手

## 硬件准备

- Windows 电脑一台，建议 `Win10` 系统。
- 一套 [EC800MCNLE 开发板](https://www.quecmall.com/goods-detail/2c90800c9bdad914019ca242951203a8) （含天线、Type-C 数据线等）。

- 一张可正常使用的 Nano SIM 卡。
- 一个 2-5W 功率的喇叭。

## 环境搭建

- 下载并安装 EC800M 系列模组驱动：[QuecPython_USB_Driver_Win10_ASR](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverA_Customer_V1.1.13.zip)。
- 下载并安装 [VSCode](https://code.visualstudio.com/)。
- 下载并解压 [QPYCom](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/QPYcom_V4.1.0.zip) 工具到电脑的合适位置。
- [下载固件包](../../media/solutions/xiaozhi_AI/EC800MCNLER06A01M08_AI_WS_OCPU_QPY_BETA0927.zip)。  
- 下载[实验源码](https://github.com/QuecPython/solution-xiaozhiAI/archive/refs/heads/master.zip)。


## 硬件连接

按照下图进行硬件连接：

<img src="../../media/solutions/xiaozhi_AI/connect.png" style="zoom:40%;" /> 


1. 将`WiFi`天线连接至图中标识有`WiFi`字样的天线连接座上。
2. 将天线连接至标识有`LTE`字样的天线连接座上。
3. 在图示位置插入可用的 `Nano SIM` 卡。
4. 将喇叭连接至图中标识有`SPK+`和`SPK-`的排针上。
5. 在图示的位置接入电池。
6. 使用 Type-C 数据线连接开发板和电脑。

## 设备开发

### 开机

完成硬件连接的工作后，电脑设备管理器的端口列表中出现包含 `Quectel USB` 字样的 COM 口，表示开机成功。

![comport.png](../../media/solutions/AIChatBot-Volcengine-webRTC/comport.png) 

### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)，烧录对应型号固件包至开发板。

### 脚本导入与运行

1. 参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下 `src` 文件夹中的所有文件导入到模组文件系统，如下图所示：

   <img src="../../media/solutions/CozeWebsocket/src.png" style="zoom:80%;" />

2. 参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件 `_main.py`。

3. 参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行。

## 业务调试

### 程序启动

执行 `_main.py` 脚本后，程序开始运行。

### 注册Coze

如果Coze没有被注册，那么无法进行正常交互，需要到[主页 - 扣子](https://www.coze.cn/home/)进行设备注册，注册完成后我们需要复制AI智能体

### 复制AI智能体

a.  进入模板

进入模板->聊天陪伴->选择智能体。 可以根据自己需要选择智能体。

<img src="../../media/solutions/CozeWebsocket/2026020201.png" style="zoom:100%;" />

b.  复制模板

选择复制模板

<img src="../../media/solutions/CozeWebsocket/2026020202.png" style="zoom:100%;" />

c.  发布模板

选择语音通话，发布模板。

选中API后， 点击发布即可。

<img src="../../media/solutions/CozeWebsocket/2026020203.png" style="zoom:100%;" />

<img src="../../media/solutions/CozeWebsocket/2026020204.png" style="zoom:100%;" />

### 配置服务器信息到模组

a.  配置个人令牌

创建个人令牌， 并配置到模组demo中。具体过程参考下图。

<img src="../../media/solutions/CozeWebsocket/2026020205.png" style="zoom:100%;" />

<img src="../../media/solutions/CozeWebsocket/2026020206.png" style="zoom:100%;" />

b.  配置智能体botid

进入项目开发界面，点击进入刚刚复制得智能体。访问得url中携带botid信息，讲bot后面得字符串配置到模组demo中，具体信息参考下图说明。

<img src="../../media/solutions/CozeWebsocket/2026020207.png" style="zoom:100%;" />

<img src="../../media/solutions/CozeWebsocket/2026020208.png" style="zoom:100%;" />

<img src="../../media/solutions/CozeWebsocket/2026020209.png" style="zoom:100%;" />

### 运行代码

运行日志如下，开场白结束后，即可进行实时对话。

<img src="../../media/solutions/CozeWebsocket/2026020210.png" style="zoom:100%;" />
