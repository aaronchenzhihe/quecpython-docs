# 配网

> 本章节主要介绍 QuecPython 下如何进行网络配置连接。当前QuecPython支持多种类型的网卡，不同的网卡有不同的使用方式，以及多网卡情况下网卡的使用切换及路由配置，本章节将逐一介绍，方便对网络配置应用了解。


<a id="click_jump_6.7.1"></a>

## 配网介绍  

QuecPython 支持的网卡类型较多，如何配置网卡，使其能够正常地进行网络通信呢？本章节主要对于网卡配置进行简单介绍，方便快速使用网卡设备进行连接网络。

QuecPython 提供了两种网卡配置方式：  
1. 通过 python 接口进行配置。
2. 通过内置的 web 服务，使用网页进行网络配置。
> 只有蜂窝无线网络的模组，无法使用 web 服务。  

![](../../media/network-comm/nic/network_config_cmd.png)

![](../../media/network-comm/nic/network_config_web.png)

<a id="click_jump_6.7.1.1"></a>

### 蜂窝无线网卡

[蜂窝无线网卡](cellular/README.md)依赖于运营商，需要模组连接运营商的SIM卡，并且在运营商基站信号覆盖范围内，方可进行网络连接。支持 QuecPython 的蜂窝通信模组在开机后会自动进行蜂窝数据网络连接。 

![](../../media/network-comm/nic/network_4g_single.png)

**示例：** 

```python
>>> import dataCall
>>> dataCall.setPDPContext(1, 0, '', '', '', 0) # 设置网卡信息
0
>>> dataCall.activate(1) # 激活第一路网卡
0
>>> dataCall.getInfo(1, 0) # 第一路拨号信息查询
(1, 0, [1, 0, '10.11.129.252', '211.138.180.4', '211.138.180.5'])

# 现在模块可以通过 4G 网络正常进行网络连接。
```

<a id="click_jump_6.7.1.2"></a>

### Wi-Fi 无线网卡

[Wi-Fi 无线网卡](wifi/README.md)不同的工作模式下具有不同的应用场景，对于 [station](wifi/wifi-station-example.md) 模式下，需要连接Wi-Fi 热点（比如路由器），通过 Wi-Fi 热点给模块提供网络。对于 [ap](wifi/wifi-ap-example.md) 工作模式下，Wi-Fi 无线网卡作为热点，接受其它 Wi-Fi 站点设备连接，并为其提供网络服务。 

Wi-Fi 无线网卡的网络配置有多种方式，如直接输入热点名称和密码、一键配网、AP 配网、web 页面配置等，但最终的目的就是为了成功获取热点名称和密码。本文着重说明 Wi-Fi 网卡的加载，并通过直接访问热点的方式进行网络连接。其余的配网方式请参考 [Wi-Fi 网卡](wifi/README.md) 章节。

![](../../media/network-comm/nic/network_wifi_station.png)

![](../../media/network-comm/nic/network_wifi_ap.png)

**示例：** 

通用 Wi-Fi 网卡配网

```python
>>> import network
>>> nic = network.WLAN(network.STA_IF) # 加载 Wi-Fi 网卡驱动，对于外挂设备需要确认是否还需要其他参数配置。
0
>>> nic.connect('ssid','password') # 连接 Wi-Fi 热点。
0
>>> nic.status() # 查询连接状态，5表示已连接。
5
>>> nic.ifconfig() # 查询 ip 信息。
('192.168.1.4', '255.255.255.0', '192.168.1.1', '192.168.1.1')

# 现在模块可以通过 Wi-Fi 网络正常进行网络连接。
```

外挂 ESP8266 Wi-Fi 网卡配网

```python
>>> from usr.WLAN import ESP8266
>>> from machine import UART
>>> esp8266 = ESP8266(UART.UART2, ESP8266.STA) # 初始化 esp8266 网卡。
0
>>> esp8266.station('ssid','password') # 连接 Wi-Fi 热点。
0
>>> esp8266.set_dns('8.8.8.8','114.114.114.114') # 配置 dns server。
0
>>> esp8266.status() # 查询连接状态，1表示已连接。
1
>>> nic.ipconfig() # 查询 ip 信息。
('172.16.1.2', '255.255.255.0', '172.16.1.1', 1500, '8.8.8.8', '114.114.114.114')
>>> esp8266.set_default_NIC('172.16.1.2') #设置 Wi-Fi 网卡作为默认网卡。
0

# 现在模块可以通过 Wi-Fi 网络正常进行网络连接。
```

<a id="click_jump_6.7.1.3"></a>

### 以太网卡

在不同的应用场景下，[以太网卡](ethernet/README.md)有不同的工作模式：[终端模式](ethernet/ethernet-wan-example.md)（WAN）、[网关模式](ethernet/ethernet-lan-example.md)（LAN）。

**终端模式**  

即作为 WAN 口，通过以太网卡为模组提供外网访问能力。模型如下图所示:

![](../../media/network-comm/nic/network_eth_node.png)

该模式下，可通过 DHCP 协议动态获取 IP 地址，亦可设置静态 IP 地址。  
通过 DHCP 协议获取 IP 地址的示例代码如下：  

```python
>>> import ethernet
>>> W5500 = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc') # 加载以太网卡驱动，在实际使用中，需要确认硬件连接是否使用默认配置，如使用自定义配置，请将网卡初始化接口参数补全。
0
>>> W5500.dhcp() # 动态获取IP信息。保证网络环境中有DHCP服务器，比如接的路由器。
0
>>> W5500.ipconfig() # 查询IP信息。
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.31.203', '255.255.255. 0', '192.168.31.1', '192.168.31.1', '0.0.0.0')]
>>> W5500.set_default_NIC('192.168.31.203') #设置以太网卡作为默认网卡。
0

# 现在模块可以通过以太网正常进行网络连接。
```

设置静态 IP 地址的示例代码如下:
```python
>>> import ethernet
>>> W5500 = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','192.168.2.100', '255.255.255.0', '192.168.2.1') # 加载以太网卡驱动，在实际使用中，需要确认硬件连接是否使用默认配置，如使用自定义配置，请将网卡初始化接口参数补全。其中静态IP信息配置需要根据自己的网络环境调整。
0
>>> W5500.dhcp() # 动态获取IP信息。
0
>>> W5500.ipconfig() # 查询IP信息。
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.2.100', '255.255.255.0', '192.168.2.1', '8.8.8.8', '114.114.114.114')]
>>> W5500.set_default_NIC('192.168.2.100') #设置以太网卡作为默认网卡。
0

# 现在模块可以通过以太网正常进行网络连接。
```

**网关模式**

即作为 LAN 口，与另一台以太网设备连接，借助 4G 网络为外接以太网设备提供外网访问能力。模型如下图所示：  

![](../../media/network-comm/nic/network_eth_gateway.png)

在该模式下，模组会默认启用 DHCP 服务，为外接的以太网设备分配动态 IP 地址。示例代码如下：  
```python
>>> import dataCall
>>> import ethernet
>>> info=dataCall.getInfo(1, 0) # 获取当前 4g 拨号 ip 信息，确认4G网络正常。
>>> print(info)
(1, 0, [1, 0, '10.84.113.152', '211.138.180.2', '211.138.180.3'])
>>> W5500 = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','192.168.43.1','','',-1,-1,-1,-1, 1) # 加载以太网卡驱动，在实际使用中，需要确认硬件连接是否使用默认配置，如使用自定义配置，请将网卡初始化接口参数补全。。
0
>>> W5500.set_default_NIC('10.84.113.152') # 设置 4G 网卡为默认网卡。
0
>>> W5500.ipconfig() # 查询 ip 信息。
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.43.1', '255.255.255. 0', '192.168.43.1', '8.8.8.8', '114.114.114.114')]
>>> W5500.set_up() # 启动以太网卡。
0

# 现在以太网终端设备可以正常进行网络连接。
```

<a id="click_jump_6.7.1.4"></a>

### USB 网卡

当前 [USB 网卡](usbnet/README.md)主要应用是使用USB网口作为 LAN 口，通过 4G 网卡进行数据转发进行网络访问。USB 网卡协议支持 ECM/RNDIS 协议，请根据需要进行选择，在 PC 下 RNDIS 可以直接加载，在 Linux/Android/IOS 系统中 ECM 可以直接加载。

支持 USB 网卡的蜂窝通信模组通过 USB 口连接至支持 ECM 或 RNDIS 协议的主机设备，借助 4G 网络为其提供外网访问能力。模型如下图所示:  

![](../../media/network-comm/nic/network_usbnet.png)

QuecPython 模组通过 set_worktype(USBNET_Type) 方法设置 USB 网卡的协议类型后，调用 open()方法即可使能 USB 网卡功能。

USBNET_Type 参数取值说明:
- USBNET.Type_RNDIS : 指定 RNDIS 协议; [Windows](usbnet/usbnet-windows-example.md) 操作系统默认支持 RNDIS 协议。
- USBNET.Type_ECM: 指定 ECM 协议; [Linux](usbnet/usbnet-linux-example.md)、[Android](usbnet/usbnet-android-example.md)、[IOS](usbnet/usbnet-linux-example.md)、[MACOS](usbnet/usbnet-linux-example.md)等操作系统默认支持 ECM 协议。

示例代码如下:

```python
>>> from misc import USBNET
>>> USBNET.get_worktype() # 获取工作模式。
3
>>> USBNET.set_worktype(USBNET.Type_RNDIS) # 配置工作模式，需要注意更新配置重启生效。
0
>>> USBNET.open() # 打开 usbnet 功能。
0

# 现在 usb 终端设备可以通过 usb 网口正常进行网络连接。
```

<a id="click_jump_6.7.2"></a>

## 简单场景  

主要介绍如何在各种网卡状态下进行网络通信。以下场景数据通信将使用 [socket tcp](../net-protocols/tcp-udp.md) 通信为例进行说明。

<a id="click_jump_6.7.2.1"></a>

### 使用默认蜂窝无线网卡传输数据  

对于单独蜂窝无线网卡情况下，可以不绑定网卡进行通信。

示例代码如下：  
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

<a id="click_jump_6.7.2.2"></a>

### 使用多张蜂窝网卡并指定网卡传输数据

对于支持多 sim 卡或者多路拨号的情况下，具体使用哪一路或者哪一个sim卡进行通信，需要指定从哪一路进行传输。  
socket API 标准接口提供了数据通信绑定网卡操作，调用*socket.bind()*绑定 ip 及端口号，可以解决多路网卡数据传输指向问题。

![](../../media/network-comm/nic/network_config_4g_doule.png)

如上图所示，使用NIC1网卡通信展示代码如下：  
```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("10.11.129.251", 0))
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

如上图所示，使用NIC2网卡通信展示代码如下：  
```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("10.11.129.252", 0))
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

<a id="click_jump_6.7.2.3"></a>

### 使用以太网卡传输数据

以太网在不同的工作方式下，数传有一定差异。我们分别以网关模式（LAN）及终端模式（WAN）工作状态下进行介绍。

网关模式（LAN）  
以太网 LAN 工作状态下，使用 4G 网络为 LAN 局域网下设备提供网络服务，此时 4G 作为默认网卡。如果模块需要通过以太网进行通信，则必须通过调用*socket.bind()*来指定使用哪个网卡进行网络通信。

![](../../media/network-comm/nic/network_config_lan_socket.png)

示例代码展示如下:  
```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("192.168.1.1", 0))
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

终端模式（WAN）  
WAN工作状态下，以太网口被配置为专门与外界通信的，默认以太网卡作为默认网卡。我们可以在不绑定的情况下直接通信即可。

![](../../media/network-comm/nic/network_eth_node.png)

示例代码展示如下：
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

<a id="click_jump_6.7.2.4"></a>

### 使用 Wi-Fi 网卡传输数据

Wi-Fi 网卡在不同的工作方式下，数传有一定差异。我们分别以 Station 模式及 AP 模式工作状态下进行介绍。

Station 模式   
Station 模式工作状态下，Wi-Fi 网口被配置为专门与外界通信的，默认 Wi-Fi 网卡作为默认网卡。我们可以在不绑定的情况下直接通信即可。

![](../../media/network-comm/nic/network_wifi_station.png)
![](../../media/network-comm/nic/network_wifi_station_single.png)

示例代码展示如下：
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

AP 模式    
Wi-Fi 网卡在 AP  工作状态下，使用 4G 网络为 AP 局域网下设备提供网络服务，此时 4G 作为默认网卡。如果模块需要通过 Wi-Fi 网卡 进行通信，则必须通过调用*socket.bind()*来指定使用哪个网卡进行网络通信。

> 对于单网卡设备可以直接通信，无需绑定操作。

![](../../media/network-comm/nic/network_wifi_ap.png)
![](../../media/network-comm/nic/network_wifi_single.png)

示例代码展示如下:  
```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("192.168.1.1", 0))
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

<a id="click_jump_6.7.3"></a>

## 路由配置讲解

我们正在优化此项，后续提供统一标准工具。

<a id="click_jump_6.7.4"></a>

## 多网卡下网络配置

如下图所示，模块集成了多种网卡设备，不同的网卡配置不同的工作模式，提供了不同的网络服务。 其中可以通过 [4G 网卡](cellular/README.md)、[Wi-Fi 网卡](wifi/README.md) [AP 模式](wifi/wifi-ap-example.md)、[以太网](ethernet/README.md)[终端模式](ethernet/ethernet-wan-example.md)（ WAN ）访问外部网络，Wi-Fi 网卡 [STATION 模式](wifi/wifi-station-example.md)、以太网[网关模式](ethernet/ethernet-lan-example.md)（ LAN ）、[USB 网卡](usbnet/README.md)均提供了局域网络，那么该如何选择网卡使用，以及如何配置网络转发，按照自己想要的方式进行网络通信？

![](../../media/network-comm/nic/network_multi_nic_connection.png)

针对多网卡使用情况下，QuecPython 提供如下两种方式进行操作。

1. 指定网卡数据发送  

多网卡情况下，当进行网络通信时，无法确定走哪个网卡进行发出，但是有时网络服务在不同的局域网内，需要从指定网卡发出，此时可以通过标准的socket接口，通过 bind 网卡的 ip，来固定 tcp/udp 通信从哪个网口发出。如下图所示，当绑定`10.11.129.252`时走4G网卡交互，当绑定`192.168.1.100`时走以太网卡交互。

![](../../media/network-comm/nic/network_eth_gateway_bind.png)

2. 网卡转发配置  

使用提供的默认网卡配置接口*set_default_NIC(ip)*，当配置默认网卡后，将该网卡作为对外访问网络网卡，启用 NAT 功能，对其他网卡数据进行转发。如下图所示，使用`nic.set_default_NIC('10.11.129.252')`配置立即生效，然后会对来自以太网卡的非本机数据从 4G 网卡转发处理。

> 1. 默认网卡接口开出，主要由用户维护默认网卡，便于切换网络转发。  
> 2. 其中 4G 网卡第一路固定配置为默认网卡，第一路拨号成功后会触发重新配置成默认网卡。目前使用场景基本都是4G网卡作为默认网卡对外转发，防止用户使用的不确定性，导致网络不可用，为了网络可靠性增加此操作。

![](../../media/network-comm/nic/network_eth_gateway_default.png)


<a id="click_jump_6.7.4.1"></a>

### Wi-Fi（AP）-> LTE（单路）

在 Wi-Fi 网卡和蜂窝无线网卡（LTE）同时存在时，通过 Wi-Fi 网络使用 4G 网络连通外网，该如何配置，工作方式见下图：  
![](../../media/network-comm/nic/network_wifi_ap.png)

对于多种网卡，配置转发关系，当前 QuecPython 尚未提供复杂详细的路由表转发配置功能，我们只需要配置默认网卡即可。再多的网卡，也只有唯一的一个默认网卡，通过默认网卡配置，都会从默认网卡进行 NAT 转发。

示例代码如下展示：  
```python

# 此处展示只是配置转发方向，具体 Wi-Fi 网卡初始化配置请参考 Wi-Fi 网卡应用指导章节。  
# nic 指具体网卡的网卡对象，当前我们默认网卡配置接口在具体网卡功能内部，后续会作为基础接口放置在网络配置模块下。

nic.set_default_NIC('10.11.129.251')

# 此时 AP 网络下的终端设备可以通过 LTE 连接外部网络

```

<a id="click_jump_6.7.4.2"></a>

### Ethernet（LAN）-> LTE（单路）

在以太网卡和蜂窝无线网卡（LTE）同时存在时，通过以太网络使用 4G 网络连通外网，该如何配置，工作方式见下图：  
![](../../media/network-comm/nic/network_config_lan_socket.png)

对于多种网卡，配置转发关系，当前 QuecPython 尚未提供复杂详细的路由表转发配置功能，我们只需要配置默认网卡即可。再多的网卡，也只有唯一的一个默认网卡，通过默认网卡配置，都会从默认网卡进行 NAT 转发。

示例代码如下展示：  
```python

# 此处展示只是配置转发方向，具体以太网卡初始化配置请参考以太网卡应用指导章节 。 
# nic 指具体网卡的网卡对象，当前我们默认网卡配置接口在具体网卡功能内部，后续会作为基础接口放置在网络配置模块下。

nic.set_default_NIC('10.11.129.252')

# 此时 LAN 网络下的终端设备可以通过 LTE 连接外部网络。

```

<a id="click_jump_6.7.4.3"></a>

### Wi-Fi（AP）-> LTE（多路）

在 Wi-Fi 网卡和蜂窝无线网卡（LTE）同时存在时，并且 LTE 进行多路拨号，一路为 Wi-Fi 网络提供 LTE 网络服务，一路为模组内部通信，该如何配置？工作方式见下图：  
![](../../media/network-comm/nic/network_config_ap_double_4g.png)

对于多种网卡，配置转发关系，当前 QuecPython 尚未提供复杂详细的路由表转发配置功能，我们只需要配置默认网卡即可。再多的网卡，也只有唯一的一个默认网卡，通过默认网卡配置，都会从默认网卡进行 NAT 转发。

如上图所示，蜂窝无线网卡第一路拨号 ip 为 10.11.129.251 ，第二路拨号 ip 为 10.11.129.252 ，我们使用第一路作为模组内部通信，第二路为 Wi-Fi 网络提供 LTE 网络服务。


第一路作为内部通信网络服务，使用第一路进行 TCP 通信示例代码如下展示： 
```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("10.11.129.251", 0))
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

第二路为 Wi-Fi 网络提供 LTE 网络服务，配置示例代码如下展示：  
```python

# 此处展示只是配置转发方向，具体 Wi-Fi 网卡初始化配置请参考 Wi-Fi 网卡应用指导章节。  
# nic 指具体的网卡对象，当前我们默认网卡配置接口在具体网卡功能内部，后续会作为基础接口放置在网络配置模块下。

nic.set_default_NIC('10.11.129.252')

# 此时 AP 网络下的终端设备可以通过 LTE 连接外部网络。

```

<a id="click_jump_6.7.4.4"></a>

### Ethernet（WAN）-> LTE（多路）

在以太网卡和蜂窝无线网卡（LTE）同时存在时，并且 LTE 进行多路拨号，一路为以太网络提供 LTE 网络服务，一路为模组内部通信，，该如何配置，工作方式见下图：  
![](../../media/network-comm/nic/network_config_lan_double_4g.png)

对于多种网卡，配置转发关系，当前 QuecPython 尚未提供复杂详细的路由表转发配置功能，我们只需要配置默认网卡即可。再多的网卡，也只有唯一的一个默认网卡，通过默认网卡配置，都会从默认网卡进行 NAT 转发。

如上图所示，蜂窝无线网卡第一路拨号 ip 为 10.11.129.251 ，第二路拨号 ip 为 10.11.129.252 ，我们使用第一路作为模组内部通信，第二路为以太网提供 LTE 网络服务。

第一路作为内部通信网络服务，使用第一路进行 TCP 通信示例代码如下展示： 
```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("10.11.129.251", 0))
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

第二路为以太网络提供 LTE 网络服务，配置示例代码如下展示：  
```python

# 此处展示只是配置转发方向，具体以太网卡初始化配置请参考以太网卡应用指导章节。  
# nic 指具体网卡的网卡对象，当前我们默认网卡配置接口在具体网卡功能内部，后续会作为基础接口放置在网络配置模块下。

nic.set_default_NIC('10.11.129.252')

# 此时 WAN 网络下的终端设备可以通过 LTE 连接外部网络。

```

<a id="click_jump_6.7.5"></a>

## 复杂场景

本章节旨在展示多网卡下的典型应用场景，以帮助用户理解多网卡环境下的网卡应用，并实现快速上手。

**案例**  

4G + ethernet + Wi-Fi 共用情况下网络应用。根据使用需求，将不同网卡配置不同的工作模式，进入正常的工作状态后，通过改变默认网卡，来确定转发规则。 

> 对于以太、Wi-Fi、蜂窝网络的使用，我们没有固定使用规则，由用户进行选择，我们建议对于多网络状态，模块使用网络优先级 以太 > Wi-Fi > 蜂窝。

1. 使用 4G 作为默认网卡，Wi-Fi 与以太网络通过 4G 转发进行网络访问。

![](../../media/network-comm/nic/network_4g_eth_wifi.png)

代码如下图所示：  
```python
# 导入usocket模块
import dataCall
import network
import ethernet

if __name__ == '__main__':
    # 默认网卡配置
    defalut_nic = None

    # 获取 4G 信息
    lte = dataCall.getInfo(1, 0)
    print(lte)

    # Wi-Fi 初始化
    WiFi = network.ASR5803W(network.AP_IF)
    WiFi.config('ssid'='QuecPython', 'key'='quecpython')
    WiFi.active(True)
    print('Wi-Fi init success')

    # 以太网初始化。
    eth = ethernet.YT8512H(b'\x12\x34\x56\x78\x9a\xbc','192.168.1.1')
    eth.set_up()
    print('ethernet init success')

    # 设置 4G 作为默认网卡
    defalut_nic = lte[2][2]
    eth.set_default_NIC(defalut_nic)
```

2. 使用以太网作为默认网卡，Wi-Fi 网络通过以太网转发进行网络访问。

![](../../media/network-comm/nic/network_eth_wifi_4g.png)

代码如下图所示：  
```python
# 导入usocket模块
import dataCall
import network
import ethernet

if __name__ == '__main__':
    # 默认网卡配置
    defalut_nic = None

    # 获取 4G 信息
    lte = dataCall.getInfo(1, 0)
    print(lte)

    # Wi-Fi 初始化
    WiFi = network.ASR5803W(network.AP_IF)
    WiFi.config('ssid'='QuecPython', 'key'='quecpython')
    WiFi.active(True)
    print('Wi-Fi init success')

    # 初始化以太网卡，以太网卡在WAN模式下使用DHCP协议动态获取IP地址,连接网络
    eth = ethernet.YT8512H(b'\x12\x34\x56\x78\x9a\xbc')
    eth.dhcp()
    eth.set_up()
    print('ethernet init success')

    # 设置以太网卡作为默认网卡
    defalut_nic = eth.ipconfig()[1][1]
    eth.set_default_NIC(defalut_nic)
```

<a id="click_jump_6.7.6"></a>

## 常见问题

<a id="click_jump_6.7.6.1"></a>

### Socket 如何和网卡绑定？

请查看 [TCP-UDP 章节](../net-protocols/tcp-udp.md)。

<a id="click_jump_6.7.6.2"></a>

### 如何测试网络链路通达?

提供 uping 模块检查网络，通过该模块 ping 各个网络，可以判断链路状态。比如以太网卡可以获取到ip，但是无法进行网络连接，此时需要检查整个网络环境，可能是上层路由器 wan 口未连接外网，也可能是外部网络配置限制导致网络不可用，就需要此功能进行整个链路检查。

```python
>>> import uping
>>> uping.ping('python.quectel.com')
uping.ping('python.quectel.com')

PING python.quectel.com (47.107.246.213): 64 data bytes

72 bytes from 47.107.246.213: icmp_seq=1, ttl=52, time=50.546000 ms
72 bytes from 47.107.246.213: icmp_seq=2, ttl=52, time=38.424000 ms

72 bytes from 47.107.246.213: icmp_seq=3, ttl=52, time=52.606000 ms

72 bytes from 47.107.246.213: icmp_seq=4, ttl=52, time=38.940000 ms
4 packets transmitted, 4 packets received, 0 packet loss
round-trip min/avg/max = 38.424/45.129/52.606 ms

```

