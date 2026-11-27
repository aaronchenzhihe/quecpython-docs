# Quick start

## Hardware preparation

- One Windows computer, recommended Win10 system.
- A set of [EC800GCN Hua series DTU development board](../../../dev_board_guide/ec800g_hua_dtu.md).
- A usable Nano Sim card.
- A rubber stick antenna.
- A Mini USB data cable.
- A USB to TTL module.

## Environment construction

- Download and install EC800G series module driver: [QuecPython_USB_Driver_Win10_U_G](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip).
    - Double-click to run and follow the prompts to complete the installation.
    - Verify the driver: Connect the development board to the computer using a Mini USB cable, open the "Device Manager", expand "Ports (COM and LPT)", and you should be able to see a port number similar to Quectel USB COM Port (for example, COM3). If it is not displayed, please check if the driver has been installed successfully, or try replacing the USB cable/connector.
- Download and install [VSCode](https://code.visualstudio.com/).
- Download and unzip [QPYCom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) Tool to the appropriate location on the computer.
- Download and install the firmware package [QPy_OCPU_SETA0002-EC800G_CNLD_SW](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_BETA0002_EC800G_CNLD_FW.zip). Please refer to [Using QPYcom to Burn Firmware](../../dev-tools/QPYcom/qpycom-dw.md ) for firmware package burning
- Download the experimental source code of [Development Resource Summary](dev_resources.md).

## Hardware connection

Connect the hardware according to the following diagram:

<img src="../../media/solutions/DTU/DTU_wires_connection.png" style="display: block; margin: 0 auto; zoom: 50%;" />

1. Connect the rubber rod antenna.
2. Insert the Nano Sim card.
3. Connect to the computer using a mini USB data cable. In QPYCom, select the serial port and expand it. You will be able to see Quectel REPL Prot. Then, open it.
4. Connect to the computer using a USB to TTL module. TX (DTU development board) connected to RX (USB to TTL module); RX (DTU development board) connected to TX (USB to TTL module); GND (DTU development board) is connected to GND (USB to TTL module) for common ground. If an RS485 interface is used, the A line at both ends is connected to the A line, and the B line is connected to the B
5. The development board adopts a 9-36V wide power supply.

## Equipment development
After completing the hardware connection, you can select the following protocols for device development based on your project requirements:

- [Device development based on TCP protocol](./tcp_demo.md)
- [Device development based on MQTT protocol](./mqtt_demo.md)

Each document provides detailed sample code and explanations to help you quickly establish communication with the cloud platform.