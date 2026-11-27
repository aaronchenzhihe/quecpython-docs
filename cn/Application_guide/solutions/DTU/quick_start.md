# 快速上手

## 硬件准备

- Windows 电脑一台，建议 Win10 系统。
- 一套 [EC800GCN 华系列 DTU 开发板](../../../dev_board_guide/ec800g_hua_dtu.md)。
- 一张可用的 Nano Sim 卡。
- 一根胶棒天线。
- 一根 Mini USB 数据线。
- 一个 USB 转 TTL 模块。

## 环境搭建

- 下载并安装 EC800G 系列模组驱动：[QuecPython_USB_Driver_Win10_U_G](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)。
    - 双击运行，按提示完成安装。
    - 验证驱动：将开发板用 Mini USB 线连接到电脑，打开“设备管理器”，展开“端口（COM 和 LPT）”，你应该能看到类似 Quectel USB COM Port 的端口号（例如 COM3）。如果没有出现，请检查驱动是否安装成功，或尝试更换 USB 线/接口。
- 下载并安装 [VSCode](https://code.visualstudio.com/)。
- 下载并解压 [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 工具到电脑的合适位置。
- 下载并安装固件包[QPY_OCPU_BETA0002_EC800G_CNLD_FW](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_BETA0002_EC800G_CNLD_FW.zip)。（固件包烧录请参阅 [使用 QPYcom 烧录固件](../../dev-tools/QPYcom/qpycom-dw.md)）
- 下载[开发资源汇总](dev_resources.md)的实验源码。

## 硬件连接

按照下图进行硬件连接：

<img src="../../media/solutions/DTU/DTU_wires_connection.png" style="display: block; margin: 0 auto; zoom: 50%;" />
1. 连接胶棒天线。
2. 插入 Nano Sim 卡。
3. 使用 mini USB 数据线连接至电脑，在 QPYCom 选择串口处展开，可以看到 Quectel REPL Prot ，将他打开。
4. 使用 USB 转 TTL 模块连接至电脑。TX（DTU开发板） 接 RX（USB 转 TTL 模块）；RX（DTU开发板） 接 TX（USB 转 TTL 模块）；GND（DTU开发板） 接  GND（USB 转 TTL 模块）共地。（如果采用 RS485 接口，则两端的 A 线连接至 A 线，B 线连接至 B 线。）
5. 开发板采用 9~36 V 宽幅供电。

## 开发示例
完成硬件连接后，您可以根据项目需求选择以下协议进行设备开发：

- [基于 TCP 协议的设备开发](./tcp_demo.md)
- [基于 MQTT 协议的设备开发](./mqtt_demo.md)

每个文档都提供了详细的示例代码和说明，帮助您快速实现与云平台的通信。

