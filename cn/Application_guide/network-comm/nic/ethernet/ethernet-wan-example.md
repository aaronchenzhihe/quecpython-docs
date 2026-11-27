# 模组通过以太网卡访问网络

模块通过以太网卡上外网，需要将以太网卡配置到终端模式（WAN）。比如以太网卡连接路由器，通过DHCP协议获取 IP信息，从而通过以太网卡连接外部网络，使得模块能够正常连接网络。

开发环境搭建请参考[快速入门](../../../../Getting_started/README.md)，本章节主要是对以太网卡使用介绍。

## 硬件准备工作

使用外挂以太网功能，需要额外准备外挂的以太网卡、网线、路由器、杜邦线。

通过杜邦线将模块与以太网卡连接，模块与网卡接线可以根据实际需求配置，具体接口配置参数请参考[ethernet - 以太网相关功能](../../../../api_reference/peripherals/ethernet.md)章节，然后通过网线将网卡与路由器连接。

| 以太网芯片型号 | 以太网模块实物图  | 硬件连接实物图  | 芯片手册 | 代码示例 |
| ---- | ---- | ---- | ---- | ---- |
| W5500    | [W5500模块实物图](../../../media/network-comm/nic/network_eth_w5500.png)  | [W5500硬件连接实物图](../../../media/network-comm/nic/network_eth_w5500_connection.png) | [W5500芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/w5500) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| DM9051 | [DM9051模块实物图](../../../media/network-comm/nic/network_eth_dm9051.png) | [DM9051硬件连接实物图](../../../media/network-comm/nic/network_eth_dm9051_connection.png) | [DM9051芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/dm9051) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| CH395  | [CH395模块实物图](../../../media/network-comm/nic/network_eth_ch395.png) | [CH395硬件连接实物图](../../../media/network-comm/nic/network_eth_ch395_connection.png) | [CH395芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/ch395) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| YT8512     | [YT8512模块实物图](../../../media/network-comm/nic/network_eth_yt8512h.png) | [YT8512硬件连接实物图](../../../media/network-comm/nic/network_eth_yt8512h_connection.png) | [YT8512芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/yt8512) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |

> 1. 硬件连接实物图参考使用，请根据自己实际环境进行连接配置。  
> 2. SZ18201/J1101与YT8512连接及使用方式一致，均使用 RMII 接口，此处不在列举展示。

## 软件准备

**网卡初始化**

默认以太网卡不开启，需要手动进行打开，以下是加载以太网卡的过程，通过DHCP的方式获取IP信息与外部通信。  
以下示例介绍针对 W5500 以太网卡进行介绍，其他网卡在初始化时，调用该型号类即可，具体参数配置请参考 [ethernet - 以太网相关功能](../../../../api_reference/peripherals/ethernet.md)下具体型号类。

```python
# 首先导入ethernet包
>>> import ethernet

# 初始化网卡，配置各个参数信息，此处填写的CS脚、RST脚、INT脚均为GPIO引脚编号（GPIO引脚编号与物理引脚的映射关系请参考API参考手册内《class Pin - 控制I/O引脚》章节）
>>> eth = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','','','',-1,38,36,37, 0)

# 网卡注册成功后，查看默认的静态 IP 地址
>>> eth.ipconfig()
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.1.100', '255.255.255.0 ', '192.168.1.1', '8.8.8.8', '114.114.114.114')]

# 启动以太网卡
>>> eth.set_up()
0

# 此时可以通过以太网卡对访问外部网络。

```

**网络配置**

网络配置可以通过两种方式配置，一种是通过DHCP获取动态IP地址，另一种是配置静态 IP地址。

通过DHCP获取动态IP地址

```python
# 首先导入ethernet包
>>> import ethernet

# 初始化网卡，配置各个参数信息，此处填写的CS脚、RST脚、INT脚均为GPIO引脚编号（GPIO引脚编号与物理引脚的映射关系请参考API参考手册内《class Pin - 控制I/O引脚》章节）
>>> eth = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','','','',-1,38,36,37, 0)

# 网卡注册成功后，查看默认的静态IP地址
>>> eth.ipconfig()
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.1.100', '255.255.255.0 ', '192.168.1.1', '8.8.8.8', '114.114.114.114')]

# 开启DHCP
>>> eth.dhcp()
0

#  DHCP开启成功后，再次查看 IP 地址，可以发现IP地址已更新。
>>> eth.ipconfig()
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.31.203', '255.255.255. 0', '192.168.31.1', '192.168.31.1', '0.0.0.0')]

# 启动以太网卡
>>> eth.set_up()
0

# 此时可以通过以太网对访问外部网络。
```

配置静态 IP地址

```python
# 首先导入ethernet包
>>> import ethernet

# 初始化网卡，通过传参IP信息，启动时会使用IP传参而不适用默认IP信息。
>>> eth = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','192.168.2.10','255.255.255.0','192.168.2.1',-1,38,36,37, 0)

# 网卡注册成功后，查看默认的静态 IP 地址
>>> eth.ipconfig()
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.2.10', '255.255.255.0 ', '192.168.1.1', '8.8.8.8', '114.114.114.114')]

# 设置静态IP
>>> eth.set_addr('192.168.3.10','255.255.255.0','192.168.3.1')
0

# 设置dns
>>> eth.set_dns('114.114.114.114','8.8.8.8')
0

#  配置完成后，再次查看 IP 地址，可以发现IP地址已更新。
>>> eth.ipconfig()
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.3.10', '255.255.255.0', '192.168.3.1', '114.114.114.114', '8.8.8.8')]

# 启动以太网卡
>>> eth.set_up()
0

# 此时可以通过以太网对访问外部网络。

```

## TCP（[socket](../../net-protocols/tcp-udp.md)）应用示例

以下示例展示 TCP 通信时，通过以太网卡进行通信时的两种方法。

**TCP 客户端绑定以太网卡通信**

此例中模块通过以太网卡进行 TCP 通信，通过调用socket 模块内 bind 接口，指定以太网卡进行数据发送。

>下例中 bind 绑定网卡接口参数 192.168.1.100 指以太网卡的IP 地址。

```python
# 导入usocket模块
import usocket

if __name__ == '__main__':
    # 创建一个socket实例
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("192.168.1.100", 0))
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

**TCP 客户端非绑定以太网卡通信**

此例中模块通过以太网卡进行 TCP 通信，不使用 socket 模块内 bind 接口，通过配置默认网卡进行 socket 通信。需要确定配置以太网卡作为默认网卡。

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

使用以太网卡进行 MQTT/HTTP 应用协议通信，需要注意由于内置的 MQTT/HTTP 通信模块没有使用绑定网卡操作接口，需要配置以太网卡作为默认网卡即可。具体 MQTT/HTTP 使用，请参考QuecPython官网文档中心API参考手册下 [umqtt - MQTT客户端](../../../../api_reference/networklib/umqtt.md) 以及 [request - HTTP客户端](../../../../api_reference/networklib/request.md) 章节。
