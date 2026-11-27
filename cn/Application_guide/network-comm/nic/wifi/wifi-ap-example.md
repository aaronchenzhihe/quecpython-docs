# AP 模式

Wi-Fi 网卡通过建立热点，为其他设备提供 4G 网络服务。

开发环境搭建请参考[快速入门章节](../../../../Getting_started/README.md)，本章节主要是以 ESP8266/ESP8285 为例，进行使用介绍。

## 硬件准备工作

使用外挂  Wi-Fi  网卡功能，需要额外准备外挂的 Wi-Fi 网卡、路由器、杜邦线。

通过杜邦线将模块与 Wi-Fi 网卡连接，模块与网卡接线可以根据实际需求配置，具体接口配置参数请参考 Wiki 下[ESP8266](../../../../api_reference/wifilib/WLAN.ESP8266.md)章节。

[ESP8266硬件连接实物图](../../../media/network-comm/nic/network_wifi_esp8266_connection.png)
[ASR5803硬件连接实物图](../../../media/network-comm/nic/network_wifi_asr5803_connection.png)
[FCM360W硬件实物图](../../../media/network-comm/nic/network_wifi_fcm360w.png)
[FC41D硬件实物图](../../../media/network-comm/nic/network_wifi_fc41d.png)

## 软件准备

**网卡初始化**

网卡初始化需要初始化 Wi-Fi 网卡 AP 模式下工作环境，保证 Wi-Fi  网卡的互通性。以及配置Wi-Fi 网卡热点名称及密码。以下示例介绍针对 ESP8266 Wi-Fi 网卡进行介绍。

```python

# ESP8266无线网卡包含在WLAN包中，从 WLAN 包导入ESP8266，然后继续从machine中导入需要的UART模块
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# 初始化网卡，串口参数选择UART2口，模式参数选择AP模式
>>> wifi = ESP8266(UART.UART2, ESP8266.AP)

# 设置无线网卡以AP模式启动，并配置AP热点的名称及密码
>>> wifi.ap('test_8266','12345678')
0

# 获取当前网卡状态，返回4说明无线网卡已启用AP热点模式
>>> wifi.status()
4

```

**路由配置**

路由配置功能主要是不同网卡下进行网络转发的方法，当前只是通过默认网卡接口进行配置，默认网卡作为转发对象，对于通过 Wi-Fi 网卡传输的非本机数据，将通过默认网卡进行 NAT 转发。
交互口输入以下命令配置路由转发，为 Wi-Fi 终端设备提供 4G 网络。

```python

>>> import dataCall

# 获取拨号信息
>>> Info = dataCall.getInfo(1,0)

# 查看是否有获得4G的IP
>>> Info
(1, 0, [1, 0, '10.62.209.177', '211.138.180.4', '211.138.180.5'])

# 设置默认网卡，AP模式下设置4G为默认网卡
>>> wifi.set_default_NIC(Info[2][2])
0

# 添加路由信息，设置网卡转发规则
>>> wifi.router_add('192.168.4.0', '255.255.255.0')
0

```

## 手机连接 Wi-Fi 热点

打开手机WLAN界面，找到名称为`test_8266`的 Wi-Fi 热点，输入密码，进行连接。

![](../../../media/network-comm/nic/network_wifi_ap_connecting.png)

## 查看网络连接

可以看到，手机已经成功连接到 Wi-Fi 热点，此时关闭手机其他网络连接，只保留此 Wi-Fi 连接，打开浏览器也可以正常访问网站。
> ESP8266是通过串口外挂到模块，通信速率较低，建议手机只能浏览新闻，无法提供流畅视频通信。

![](../../../media/network-comm/nic/network_wifi_ap_connected.png)
