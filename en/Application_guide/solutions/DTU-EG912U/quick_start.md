# Quick start

## Hardware preparation

- A Windows computer, Win10 system is recommended
- One set [EG912U Core Board](../../../dev_board_guide/bg-eg-core-evb.md)
- One usable Nano Sim card
- An antenna
- A Mini USB cable
- A USB to TTL module

## Programming environment setup

- Download and install the EC912U series module driver：[QuecPython_USB_Driver_Win10_U_G](https://www.quectel.com.cn/wp-content/uploads/2026/03/Quectel_Windows_USB_DriverU_V1.0.19.zip)。
- Download and install [VSCode](https://code.visualstudio.com/)。
- Download and extract the [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) tool to a suitable location on your computer.
- Download and install the firmware package[QPY_OCPU_BETA0002_EG912U_GLAA_FW](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_BETA0002_EG912U_GLAA_FW.zip)。（For firmware package burning, please refer to [Using QPYcom for Firmware Burning](../../../../cn/Application_guide/dev-tools/QPYcom/qpycom-dw.md)）
- [Download experimental source code](https://github.com/QuecPython/solution-simple-DTU)。

## Hardware connection

Follow the following diagram for hardware connection:![](../../media/solutions/DTU-EG912U/EVB_link.png)

![](../../media/solutions/DTU-EG912U/EVB_link1.png)



1. The MAIN port connects to an antenna.
2. TX (EG912U development board) connected to RX (USB-to-TTL module); RX (EG912U development board) connected to TX (USB-to-TTL module); GND (EG912U development board) connects to GND (USB-to-TTL module) in common ground. The development board uses 3.3V power supply.
3. Insert the Nano Sim card.
4. Connect to computer using USB data cable, connect to computer using USB to TTL module.

## Device development

- [Device development based on TCP protocol](./tcp_demo.md)
- [Device development based on MQTT protocol](./mqtt_demo.md)

