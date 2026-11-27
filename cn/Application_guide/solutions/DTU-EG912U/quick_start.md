# 快速上手

## 硬件准备

- 一台windows电脑，推荐使用win10系统
- 一套 [EG912U Core Board](../../../dev_board_guide/bg-eg-core-evb.md) 开发板
- 一张 Nano sim 卡
- 一根 天线
- 一根 Mini USB 数据线
- 一根 USB 转 TTL 模块

## 环境搭建

- 下载并安装驱动：[QuecPython_USB_Driver_Win10_U_G](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)
- 下载并安装 [VSCode](https://code.visualstudio.com/)
- 下载 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 并解压
- 下载固件包 [QPY_OCPU_BETA0002_EG912U_GLAA_FW](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_BETA0002_EG912U_GLAA_FW.zip)。（[使用 QPYcom 烧录固件](../../dev-tools/QPYcom/qpycom-dw.md)）
- [下载源码](https://github.com/QuecPython/solution-simple-DTU)。

## 硬件连接

按照一下图示进行硬件连接：

![](../../media/solutions/DTU-EG912U/EVB_link.png)

![](../../media/solutions/DTU-EG912U/EVB_link1.png)



1.MAIN端口连接到天线。
2.TX（EG912U开发板）连接RX（USB转TTL模块）；RX（EG912U开发板）连接TX（USB转TTL模块）；GND（EG912U开发板）连接到公共接地中的GND（USB到TTL模块）。
3.插入Nano Sim卡。
4.使用USB数据线连接到计算机，使用USB转TTL模块连接到计算机。

## 设备开发

- [基于tcp协议设备开发](./tcp_demo.md)
- [基于mqtt协议设备开发](./mqtt_demo.md)

