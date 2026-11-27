# 快速上手

## 硬件准备

- Windows 电脑一台，建议 `Win10` 系统。
- 两套 [EC600MCNLE QuecPython 标准开发板](../../../dev_board_guide/ec600x-evb.md) （每一套均含天线、Type-C 数据线等）。

  > 💡 **Tips**
  > 一套用于主叫设备演示，一套用于被叫设备演示。

- 两张可正常使用的 Nano SIM 卡。
- 两个型号为 ST7789、分辨率为 240*240 的 LCD 屏幕。
- 两个 2-5W 功率的喇叭。

## 环境搭建

- 下载并安装 EC600M 系列模组驱动：[QuecPython_USB_Driver_Win10_ASR](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverA_Customer_V1.1.13.zip)。

- 下载并安装 [VSCode](https://code.visualstudio.com/)。

- 下载并解压 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 工具到电脑的合适位置。

- 下载[固件包](https://github.com/QuecPython/solution-POC/releases/download/v2.0.1/EC600MCNLER06A01M08_POC_XBND_OCPU_QPY_BETA0117.zip)。

- 下载[实验源码](https://github.com/QuecPython/solution-POC/archive/refs/tags/v2.0.1.zip)。

- 申请伯纳德**芯平台**测试账号。
  
  根据下图所示步骤获取 IMEI 号，将其提供给移远销售人员，即可申请对讲机测试账号。
  
  ![](../../media/solutions/poc/get_imei.png) 
  
  > 💡 **Tips**
  > - 非商业测试，请联系移远销售人员（ li.bao@quectel.com ），申请伯纳德**芯平台**测试账号。
  > - 商业应用请走商务流程。

## 硬件连接

按照下图进行硬件连接：

<img src="../../media/solutions/poc/wire_connection.png" style="zoom: 80%;" > 

1. 将喇叭连接至图中标识有 `SPK+` 和 `SPK-` 的排针上。
2. 将 LCD 屏连接至标识有 `LCD` 字样的排针上。
3. 在图示位置插入可用的 Nano SIM 卡。
4. 将天线连接至标识有 `LTE` 字样的天线连接座上。
5. 使用 Type-C 数据线连接开发板和电脑。

## 设备开发

### 开机

完成硬件连接的工作后，长按开发板上标识为`PWK`的按键，直到网络灯闪烁，或电脑设备管理器的端口列表中出现包含 `Quectel USB` 字样的 COM 口，表示开机成功。

![comport.png](./../../media/solutions/poc/comport.png) 

### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)，烧录固件包 [EC600MCNLER06A01M08_POC_XBND_OCPU_QPY_BETA0117.zip](https://github.com/QuecPython/solution-POC/releases/download/v2.0.1/EC600MCNLER06A01M08_POC_XBND_OCPU_QPY_BETA0117.zip) 至开发板。

### 脚本导入与运行

1. 参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下 `code` 文件夹中的所有文件导入到模组文件系统，如下图所示：

<img src="../../media/solutions/poc/files.png" style="zoom:80%;" />

2. 参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件 `poc_main.py`。
3. 参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行。

## 业务调试

### 程序启动

执行 `poc_main.py` 脚本后，程序开始运行，进入到 `WelcomeScreen` 界面，同时获取  SIM 卡状态和当前账号，并通过 TTS 语音播报当前登录用户及其加入的群组信息。

<img src="../../media/solutions/poc/Logging.png" style="zoom: 16%;" /> 

> 🚩 **Warning**
> 未插入 SIM 卡时不会进入到主界面，插入 SIM 卡并重启设备后即可正常运行。
> <img src="../../media/solutions/poc/no_sim_card.png" style="zoom: 16%;" />

### 主界面

主界面包含多个选项列表（用户可参考 [软件设计讲解 - 界面模块](./software_design.md#界面模块) 自行添加、修改），每个选项对应一个新的界面。

- 单击 `key2` 键，选中框往下滚动。
- 双击 `key2` 键，进入所选中的界面。
- 长按 `key2` 键，返回上一级界面。

<img src="../../media/solutions/poc/MainScreen.png" style="zoom: 16%;" /> 

### 群组列表

进入群组管理界面可以查询到当前账号所加入的群组列表，并且消息提示框会提示当前所在哪个群组，如下图：

<img src="../../media/solutions/poc/group.png" style="zoom: 16%;" /> 

### 成员列表

进入成员列表界面可以查询到当前所在群组的所有成员，如下图：

<img src="../../media/solutions/poc/member.png" style="zoom:16%;" /> 

### 对讲

- 长按 `key1` 键，可以与同一群组内的成员进行对讲，菜单栏显示 `麦克风` 图标。
- 说话结束后，松开 `key1` 键，结束对讲功能。
- 在主动呼叫或被对方呼叫时，都会有消息弹窗提示。
- 对方讲话时，菜单栏显示 `听筒` 图标。

**主动呼叫图示：**
<img src="../../media/solutions/poc/speak.png" style="zoom: 16%;" /> 

**对方呼叫图示：**

<img src="../../media/solutions/poc/listen.png" style="zoom: 16%;" /> 
