# 快速上手

## 硬件准备

- Windows 电脑一台，建议 `Win10` 系统。
- 一套meter-box产品。
- 一张可正常使用的 SIM 卡。

## 环境搭建

- 下载并安装 EG912U 系列模组驱动：[QuecPython_USB_Driver_Win10_U_G](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)
- 下载并安装 [VSCode](https://code.visualstudio.com/)。
- 下载并解压 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V3.9.0.zip) 工具到电脑的合适位置。
- [下载固件包](../../media/solutions/meter-box/8915DM_cat1_open_EG912UGLAAR05A01M08_TEST0807.zip)。  
- 下载[实验源码](https://codeload.github.com/QuecPython/meter-box/zip/refs/heads/main)。


## 硬件连接

按照下图进行硬件连接：

 1.插入SIM卡，使用Type-C数据线连接电脑

​	<img src="../../media/solutions/meter-box/connect.png" style="zoom: 67%;" />

2.按下开机按钮开机

​	<img src="../../media/solutions/meter-box/switch.png" style="zoom: 80%;" />

## 设备开发

### 开机

完成硬件连接的工作后，电脑设备管理器的端口列表中出现包含 `Quectel USB` 字样的 COM 口，表示开机成功。

![comport.png](../../media/solutions/meter-box/comport.png) 

### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)，烧录对应型号固件包至开发板。

### 脚本导入与运行

1. 参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下 `src` 文件夹中的所有文件导入到模组文件系统，如下图所示：<img src="../../media/solutions/meter-box/download.png" style="zoom:80%;" />

2. 参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件 `_main.py`。

3. 参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行。

## 业务调试

### 程序启动

执行 `_main.py` 脚本后，程序开始运行。

![](../../media/solutions/meter-box/start.png)

### 云平台收到上行数据

[点击跳转](https://core.acceleronix.io/)移远云平台

![](../../media/solutions/meter-box/uplink_data.png)

### 手机APP界面展示

​	![](../../media/solutions/meter-box/app.png)
