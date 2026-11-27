# 快速上手

## 硬件准备

- Windows电脑一台，建议 `Win10` 系统。

- 一套[BG95-M3 Quecpython 标准开发板](../../../dev_board_guide/bg95-evb.md)

- 一张能够正常使用的SIM卡


## 环境搭建

- 下载并安装 BG95 系列模组驱动：[QuecPython_USB_Driver_Win10_ASR](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverQ_NDIS_V2.7.14.zip)。


- 下载并安装 [VSCode](https://code.visualstudio.com/)。
- 下载并解压 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 工具到电脑的合适位置。
- 下载[固件包](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_V0003_BG95M3_FW.zip )。
- 下载[实验源码](https://github.com/QuecPython/tracker-box/archive/refs/heads/main.zip) 

## 设置您的智能追踪器

按照以下步骤为智能追踪器供电：

1. 开机前，将 nano SIM 卡（需开通数据服务）插入 SIM 卡槽。若开机后插入 SIM 卡，需重启设备。
2. 通过以下任一方式开机：


- 方式 1：按下设备侧面的电源按钮。

  <img src="../../media/solutions/tracker_box(BG95M3)/power_button.PNG" style="zoom: 20%;" />

  

- 方式 2：使用随附的 USB-C 线缆将智能追踪器连接至笔记本电脑或充电宝，设备将自动开机。<img src="../../media/solutions/tracker_box(BG95M3)/usb-c_sim.jpg" style="zoom: 20%;" />

3.开机后，智能追踪器将连接至蜂窝网络并链接到 Acceleronix 资产管理 SaaS 平台。

## 连接至资产管理 SaaS 平台 

Acceleronix 开发的资产管理 SaaS 平台为特定行业提供全面解决方案。它涵盖从 TSL 模型功能定义到 SaaS 平台管理及移动应用设备控制的全业务流程。

该资产管理 SaaS 平台支持用户对设备进行操作，可基于 TSL 模型对硬件数据进行动态分析。它集成了水质、温度、湿度、光线、二氧化碳等多种传感器，以适应不同场景。功能包括实时数据查看、历史运行曲线、数据聚合，以及基于位置设备的设备地图与轨迹回放。

## 设备开发

### 开机

完成硬件连接工作后，当PWR，SCK1亮起或电脑设备管理器的端口列表出现包含Quectel USB字样的COM口，表示开机成功

<img src="../../media/solutions/tracker_box(BG95M3)/USB.png" style="zoom: 100%;" />

### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)，烧录[固件包](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_V0003_BG95M3_FW.zip )至开发板。

### 脚本导入与运行

1.参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下的code文件夹中的所有文件按原目录结构导入到模组文件系统中，如下图所示

<img src="../../media/solutions/tracker_box(BG95M3)/Qpycom.png" style="zoom: 100%;" />

2.参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件_main.py

3.参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行。

## 业务调试

### 程序启动

执行_main.py脚本后，程序开始运行，会打印拨号信息，包括拨号状态、IP地址、DNS服务器地址，设备号等

<img src="../../media/solutions/tracker_box(BG95M3)/start.png" style="zoom: 100%;" />

### 数据检测

开始运行后会打印启动信息

<img src="../../media/solutions/tracker_box(BG95M3)/jiance.png" style="zoom: 100%;" />

### 查看设备位置（LBS）

1. 在设备列表中点击 **详情**，打开设备详情页。
2. 进入 **位置** 标签页查看 LBS 位置数据。<img src="../../media/solutions/tracker_box(BG95M3)/saas_location.png" style="zoom: 100%;" />

> **注意**：位置每 30 分钟更新一次。如需手动刷新，请点击应用控制面板中的刷新按钮，然后重新加载页面即可查看更新后的位置。

### 查看和管理传感器数据

1. 当设备在线时，在 Wonderfree 应用中点击设备进入控制面板。
2. 查看智能追踪器的实时传感器数据和 LBS 位置数据。
3. 如需更新传感器数据或 LBS 位置，点击控制面板右上角的刷新按钮

<img src="../../media/solutions/tracker_box(BG95M3)/wonderfree4.png" style="zoom: 25%;" />

<img src="../../media/solutions/tracker_box(BG95M3)/wonderfree5.png" style="zoom: 25%;" />