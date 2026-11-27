
# QuecPython 支持网卡介绍

本章节主要针对当前 QuecPython 下支持的网卡（[蜂窝无线网卡](../cellular/README.md)、[Wi-Fi 网卡](../wifi/README.md)、[以太网卡](../ethernet/README.md)、[USB 网卡](../usbnet/README.md)）进行简单介绍。

## 支持网卡简介

### 蜂窝无线网卡

QuecPython 支持的模块，绝大多数都是蜂窝通信模块，因此 QuecPython 对于蜂窝无线网卡的支持有着天然的优势，其具有完善的功能接口和稳健的底层技术支撑。

### Wi-Fi 网卡
QuecPython 提供了 Wi-Fi 设备管理功能，通过硬件接口外挂 Wi-Fi 芯片，或者移植 QuecPython 到 Wi-Fi 芯片的方式进行控制。   
外置 Wi-Fi 网卡，针对模块芯片本身不自带 Wi-Fi 网卡，分别支持通过串口或 SDIO 硬件接口连接外部 Wi-Fi 网卡，如 ESP8266/FC41D/ASR5803。  
内置 Wi-Fi 网卡，直接在 Wi-Fi 芯片上移植 QuecPython，如 FCM360W/FC41D。

### 以太网卡
QuecPython 提供了以太网卡/PHY芯片管理功能，QuecPython 内置 lwIP 协议栈以及 EMAC 层，使得能够驱动以太网 / PHY 芯片。现已支持多种 SPI/RMII 接口以太网芯片。

### USB 网卡
QuecPython 提供了 USB 网卡管理功能，QuecPython 在 USB 接口集成 RNDIS/ECM/MBIM 协议，能够在不同系统下（如 [Windows](../usbnet/usbnet-windows-example.md)/[Linux](../usbnet/usbnet-linux-example.md)/[Android](../usbnet/usbnet-android-example.md)/[IOS](../usbnet/usbnet-linux-example.md)等）识别成网卡设备，直接进行网络连接。

## 网卡支持列表

<table>
    <tr>
        <th colspan = '2'>网卡类型</th>
        <th colspan = '1'>支持型号</th>
    </tr>
    <tr>
        <td  colspan = '2'>蜂窝无线网卡</td>
        <td>除 Wi-Fi 芯片平台外均支持</td>
    </tr>
    <tr>
        <td rowspan="4" >Wi-Fi网卡</td>
        <td>ESP8266</td>
        <td>EC600N系列、EC600M系列、EC600E/EC800E系列</td>
    </tr>
        <tr>
        <td>FC41D</td>
        <td>EC600N系列、EC600M系列、EC600E/EC800E系列</td>
    </tr>
    <tr>
        <td>ASR5803</td>
        <td>EC200A系列</td>
    </tr>
    <tr>
        <td>内置WLAN</td>
        <td>FCM360W、FC41D</td>
    </tr>
    <tr>
        <td rowspan="4" >以太网卡</td>
        <td>W5500</td>
        <td>EC600N系列、EC600M/EC800M系列</td>
    </tr>
    <tr>
        <td>DM9051</td>
        <td>EC600N系列</td>
    </tr>
    <tr>
        <td>CH395</td>
        <td>EC600U系列</td>
    </tr>
        <td>YT8512/SZ18201/JL1101</td>
        <td>EC200A系列</td>
    </tr>
    <tr>
        <td rowspan="3">USB网卡</td>
        <td>RNDIS</td>
        <td>除高通芯片平台及 Wi-Fi 芯片平台外均支持</td>
    </tr>
    <tr>
        <td>ECM</td>
        <td>除高通芯片平台及 Wi-Fi 芯片平台外均支持</td>
    </tr>
        <tr>
        <td>MBIM</td>
        <td>仅高通芯片平台支持</td>
    </tr>
</table>


