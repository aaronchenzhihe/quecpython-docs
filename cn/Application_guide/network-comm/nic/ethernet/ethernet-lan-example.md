
# 模组通过 4G 转以太网为外部设备提供网络

W5500作为网关使用配置时，通过网络转发，使得以太网卡局域网通过模组4G网络进行外网访问。比如 w5500 网卡连接电脑，电脑需要配置静态IP与 w5500 以太网卡同一网段，网关与 w5500 网卡地址一致，使得电脑能够正常通过 4G网卡连接网络。

## 硬件准备工作

使用外挂以太网功能，需要额外准备外挂的以太网卡、网线、路由器、杜邦线。

通过杜邦线将模块与以太网卡连接，模块与网卡接线可以根据实际需求配置，具体接口配置参数请参考[ethernet - 以太网相关功能](../../../../api_reference/peripherals/ethernet.md)章节，然后通过网线将网卡与路由器连接。

|  以太网芯片型号  | 以太网模块实物图  | 硬件连接实物图  | 芯片手册 | 代码示例 |
| ---- | ---- | ---- | ---- | ---- |
| W5500    | [W5500模块实物图](../../../media/network-comm/nic/network_eth_w5500.png)  | [W5500硬件连接实物图](../../../media/network-comm/nic/network_eth_w5500_connection.png) | [W5500芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/w5500) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| DM9051 | [DM9051模块实物图](../../../media/network-comm/nic/network_eth_dm9051.png) | [DM9051硬件连接实物图](../../../media/network-comm/nic/network_eth_dm9051_connection.png) | [DM9051芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/dm9051) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| CH395  | [CH395模块实物图](../../../media/network-comm/nic/network_eth_ch395.png) | [CH395硬件连接实物图](../../../media/network-comm/nic/network_eth_ch395_connection.png) | [CH395芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/ch395) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| YT8512     | [YT8512模块实物图](../../../media/network-comm/nic/network_eth_yt8512h.png) | [YT8512硬件连接实物图](../../../media/network-comm/nic/network_eth_yt8512h_connection.png) | [YT8512芯片手册](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/yt8512) | [示例代码参考下载](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |

> 1. 硬件连接实物图参考使用，请根据自己实际环境进行连接配置。  
> 2. SZ18201/J1101与YT8512连接及使用方式一致，均使用 RMII 接口，此处不在列举展示。

## 软件准备

**网卡初始化**

交互口输入以下命令初始化以太网设备，配置网卡以网关模式（LAN）使用。  
```python
# 网关模式需要使用到拨号信息，所以多导入一个dataCall包
>>> import ethernet
>>> import dataCall

# 初始化网卡，配置各个参数信息，此处填写的CS脚、RST脚、INT脚均为GPIO引脚编号（GPIO引脚编号与物理引脚的映射关系请参考API参考手册内《class Pin - 控制I/O引脚》章节）
>>> eth = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','192.168.1.1','','',-1,38,36,37,1)

# 启动以太网卡
>>> eth.set_up()
0

```

**路由配置**

路由配置功能主要是不同网卡下进行网络转发的方法，当前只是通过默认网卡接口进行配置，默认网卡作为转发对象，对于通过 LAN 口非本机数据，将通过默认网卡进行 NAT 转发。

交互口输入以下命令配置路由转发为 LAN 设备提供 4G 网络。  
```python
# 网关模式需要使用到拨号信息，所以多导入一个dataCall包
>>> import dataCall

# 获取当前 4G网卡 拨号IP信息
>>> info=dataCall.getInfo(1, 0)

# 查看是否有获得4G网卡的IP
>>> print(info)
(1, 0, [1, 0, '10.62.209.177', '211.138.180.4', '211.138.180.5'])

# 设置默认网卡，网关模式下设置4G网卡为默认网卡
>>> eth.set_default_network_card(info[2][2])
0

# 查询以太网卡IP信息
>>> eth.ipconfig()
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.1.100', '255.255.255.0 ', '192.168.1.1', '8.8.8.8', '114.114.114.114')]

```

## 对端配置

以PC为例，此时PC端是无法进行网络通信的，还需配置静态IP，与模组静态IP配置在同一个网段方可正常通信。  
右键`以太网`，选择属性，点击`Internet协议版本4(TCP/IPv4)`，取消自动获得IP地址和自动获得DNS服务器地址，改为如下图所示的静态IP地址，点击确定。

> 此处电脑也可以获取动态IP，部分模块未内置DHCP Server需要通过静态IP的方式进行网络配置。

![](../../../media/network-comm/nic/network_eth_demo_3.png)

## 对端通过模组的以太网上网

此时可以通过电脑浏览器访问网页。比如浏览器输入 https://developer.quectel.com 即可正常访问。

![](../../../media/network-comm/nic/network_eth_gateway_web.png)
