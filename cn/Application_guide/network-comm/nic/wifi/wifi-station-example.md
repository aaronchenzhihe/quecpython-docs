# Station 模式

Wi-Fi 网卡通过连接热点，为模块提供网络。

开发环境搭建请参考[快速入门章节](../../../../Getting_started/README.md)，本章节主要是以 ESP8266/ESP8285 为例，进行使用介绍。

## 硬件准备工作

使用外挂 Wi-Fi 网卡功能，需要额外准备外挂的 Wi-Fi 网卡、路由器、杜邦线。

通过杜邦线将模块与 Wi-Fi 网卡连接，模块与网卡接线可以根据实际需求配置，具体接口配置参数请参考 WIKI 下 [ESP8266](../../../../api_reference/wifilib/WLAN.ESP8266.md) 章节。

[ESP8266硬件连接实物图](../../../media/network-comm/nic/network_wifi_esp8266_connection.png)  
[ASR5803硬件连接实物图](../../../media/network-comm/nic/network_wifi_asr5803_connection.png)  
[FCM360W硬件实物图](../../../media/network-comm/nic/network_wifi_fcm360w.png)   
[FC41D硬件实物图](../../../media/network-comm/nic/network_wifi_fc41d.png)  

## 软件准备

**网卡初始化**

网卡初始化不对网卡进行配置，只是初始化 Wi-Fi 网卡工作环境，保证 Wi-Fi 网卡的互通性。以下示例介绍针对 ESP8266/ESP8285 Wi-Fi 网卡进行介绍。

```python

# ESP8266无线网卡包含在WLAN包中，从WLAN包导入ESP8266，然后继续从machine中导入需要的UART模块
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# 初始化网卡，串口参数选择UART2口，模式参数选择STA模式
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

```

**Wi-Fi 网卡配网**

ESP8266/ESP8285 提供多种配网方式，分别通过命令行配网、WEB配网、SMARTCONFIG配网。

命令行配网

```python
# ESP8266无线网卡包含在WLAN包中，从WLAN包导入ESP8266，然后继续从machine中导入需要的UART包
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# 初始化网卡，串口参数选择UART2口，模式参数选择STA模式
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# 配置无线网卡以station模式启动时所要连接的Wi-Fi名称及密码
>>> wifi.station('wifiname','wifipassword')
0

```

WEB配网

```python
# ESP8266无线网卡包含在WLAN包中，从WLAN包导入ESP8266，然后继续从machine中导入需要的UART包
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# 初始化网卡，串口参数选择UART2口，模式参数选择STA模式
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# 配置无线网卡以station模式启动时所要连接的Wi-Fi名称及密码
>>> wifi.web_config('admin','admin123456')
0

```

SMARTCONFIG配网

```python
# ESP8266无线网卡包含在WLAN包中，从WLAN包导入ESP8266，然后继续从machine中导入需要的UART包
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# 初始化网卡，串口参数选择UART2口，模式参数选择STA模式
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# 配置无线网卡以smartconfig功能启动时使用的协议
# 以使用AIRKISS & ESPTOUCH协议为例
>>> wifi.smartconfig(3)
0

```

**网络转发配置**

ESP8266/ESP8285 是一个独立完整的设备，内部包括独立的配网方式，ESP8266/ESP8285 配网完成后，QuecPython 需要在模块侧进行网络配置才能进行正常的网络转发。

```python
# ESP8266无线网卡包含在WLAN包中，从WLAN包导入ESP8266，然后继续从machine中导入需要的UART包
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# 初始化网卡，串口参数选择UART2口，模式参数选择STA模式
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# 配置无线网卡以station模式启动时所要连接的Wi-Fi名称及密码
>>> wifi.station('wifiname','wifipassword')
0

# 获取当前网卡状态，返回1说明无线网卡已连接Wi-Fi，返回2说明未连接Wi-Fi
>>> wifi.status()
1

# 给无线网卡配置DNS服务器
>>> wifi.set_dns('8.8.8.8','114.114.114.114')
0

# 再次查看无线网卡的IP信息，此处需要注意172.16.1.2是虚拟IP，不是具体Wi-Fi终端的IP
>>> wifi.ipconfig()
('172.16.1.2', '255.255.255.0', '172.16.1.1', 1500, '8.8.8.8', '114.114.114.114')

# 设置无线网卡作为默认网卡，使用无线网卡进行网络通信
>>> wifi.set_default_NIC('172.16.1.2')
0

# 此时可以通过Wi-Fi网卡访问外部网络

```

## TCP（[socket](../../../../api_reference/stdlib/usocket.md)）应用示例

以下示例展示 TCP 通信时，通过 Wi-Fi 网络进行通信时的两种方法。

**TCP 客户端绑定 Wi-Fi 网卡通信**

此例中通过 Wi-Fi 网卡进行 TCP 通信，通过 socket 模块内 bind 接口，指定通过 Wi-Fi 网口进行发送。

> 下例中 bind 绑定网卡接口参数 172.16.1.2 指 Wi-Fi 网卡的 ip 地址。网卡 ip 地址通过网卡对象下 ipconfig 接口进行查询，详细见 [Wi-Fi](../../../../../en/api_reference/wifilib/README.md)  接口 WIKI。

```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("172.16.1.2", 0))
    # 解析域名
    sockaddr=usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
    print('start connect')
    # 建立连接
    sock.connect(sockaddr)
    # 向服务端发送消息
    ret=sock.send('GET /NEWS HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
    print('send %d bytes' % ret)
    #接收服务端消息
    data=sock.recv(1024)
    print('recv %s bytes:' % len(data))
    print(data.decode())

    # 关闭连接
    sock.close()
```

**TCP 客户端非绑定 Wi-Fi 网卡通信**

此例中通过 Wi-Fi 网络进行 TCP 通信，不使用 socket 模块内 bind 接口，通过配置默认网卡进行 socket 通信。需要确定配置 Wi-Fi 网卡作为默认网卡。

```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
    sock.settimeout(5)
    # 解析域名
    sockaddr=usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
    print('start connect')
    # 建立连接
    sock.connect(sockaddr)
    # 向服务端发送消息
    ret=sock.send('GET /NEWS HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
    print('send %d bytes' % ret)
    #接收服务端消息
    data=sock.recv(1024)
    print('recv %s bytes:' % len(data))
    print(data.decode())

    # 关闭连接
    sock.close()
```

## MQTT/HTTP 等应用协议应用

使用 Wi-Fi 网卡进行 MQTT/HTTP 应用协议通信，需要注意由于内置的 MQTT/HTTP 通信模块没有使用绑定网卡操作接口，需要配置以太网卡作为默认网卡即可。具体 MQTT/HTTP 使用请参考应用层协议 WIKI 下 [umqtt](../../../../api_reference/networklib/umqtt.md) 以及 [reqeust](../../../../api_reference/networklib/request.md) 章节。
