# 驱动相关问题

### 驱动下载地址

https://www.quectel.com.cn/download-zone

### 各平台驱动详情

- EC600N/EC600M/EC600K/EC800N/EC800M/EG810M/EC800K/EC200A等型号安装[QuecPython_USB_Driver_Win10_ASR_V1.1.13](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverA_Customer_V1.1.13.zip)
- EC200U/EC600U/EC600G/EC800G/EG912U/EG915U等型号安装[QuecPython_USB_Driver_Win10_U_G_V1.0.19](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)
- BG95/BG600L等型号安装[QuecPython_USB_Driver_Win10_BG_V2.7.14](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverQ_NDIS_V2.7.14.zip)
- Linux系统全系列模组共用驱动[QuecPython_USB_Driver_Linux_Source_Code](https://github.com/QuecPython/Quectel_Linux_USB_Serial_Option_Driver)

### 驱动安装流程

下载对应驱动后解压双击安装即可.

### 调试交互口名称

各平台统一交互口名称: **Quectel-USB-REPL-Port**. 历史交互口如下.

*说明: 统一交互口需要同步更新模块固件与驱动程序.*

### 历史交互口名称

| 模组平台                                                     | 历史交互口名称                                          |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| EC600N/EC600M/EC600K/EC800N/EC800M/<br />EG810M/EC800K/EC200A | **Quectel USB MI05 COM Port <br />USB串行设备**         |
| EC200A                                                       | **Quectel USB MI05 COM Port<br />Quectel USB LOG Port** |
| EC200U/EC600U/EC600G/EC800G/EG912U/<br />EG915U              | **Quectel USB NEMA Port**                               |
| BG95/BG600L                                                  | **Quectel USB NEMA Port**                               |
| EC600E/EC800E                                                | **Quectel USB 串行设备**                                |

