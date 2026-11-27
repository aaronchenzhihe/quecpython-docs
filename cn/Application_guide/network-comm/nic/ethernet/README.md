# 以太网卡

本章主要介绍 QuecPython 下以太网卡适配情况，以及如何使用。


<a id="click_jump_6.5.1"></a>

## 以太网卡介绍

以太网是一种计算机局域网技术。IEEE（Institute of Electrical and Electronics Engineers，电气与电子工程师协会）组织的 802.3 标准制定了以太网的技术规范，它规定了包括物理层的连线、电子信号和介质访问控制的内容。以太网是目前应用最普遍的局域网技术。以太网在生活中随处可见，电脑上的以太网口，路由器上的以太网口等。  

**QuecPython 上是如何支持以太网的？**  

网络通信一般基于 TCP/IP 协议进行构建的，而 QuecPython 则支持轻量级的 TCP/IP 协议栈--lwIP。根据支持 QuecPython 的模组的以太网接口特性，可以分为两种方式:

- QuecPython 模组 + 以太网芯片 (MAC + PHY，如 W5500/CH395 等)。
- QuecPython 模组 + PHY 芯片 (如 YT8512/JL1101 等)。

![](../../../media/network-comm/nic/network_eth_spi_adapter.png)

![](../../../media/network-comm/nic/network_eth_rmii_adapter.png)


<a id="click_jump_6.5.1.1"></a>

### QuecPython 支持的以太网卡硬件接口

<a id="click_jump_6.5.1.1.1"></a>

#### SPI 接口

>[SPI](../../../hardware/peripheral-interfaces/SPI.md) 是串行外设接口（Serial Peripheral Interface）的缩写，是美国摩托罗拉公司最先推出的一种同步串行传输规范，也是一种单片机外设芯片串行扩展接口，是一种高速、全双工、同步通信总线，所以可以在同一时间发送和接收数据，通常能达到甚至超过 10M/bps。

对于不支持 MAC 层的 QuecPython 模组，可以通过 SPI 硬件接口外挂支持 MAC + PHY 的芯片。当前已支持多种型号，如 W5500/DM9051/CH395。

**硬件结构**   
SPI 接口硬件连接图如下所示：

![](../../../media/network-comm/nic/network_eth_spi.png)

|  引脚   | 说明 |
| ---- | ------- |
| `SCLK` | `SPI`时钟引脚 |
| `MISO` | `SPI`主机输入从机输出引脚 |
| `MOSI` | `SPI`主机输出从机输入引脚 |
| `CS`   | `SPI`片选引脚，此管脚也可以自定义配置 |
| `PIN_X` | 自定义引脚，用于数据接收中断触发 |
| `PIN_Y` | 自定义引脚，用于芯片`reset`控制 |

**初始化流程**

网卡初始化加载流程如下图所示：

![](../../../media/network-comm/nic/network_eth_spi_init_flow.png)

<a id="click_jump_6.5.1.1.2"></a>

#### RMII 接口

>RMII（Reduced Media Independant Interface）即简化媒体独立接口，是标准的以太网接口之一，比 MII 有更少的 I/O 传输。是简化的 MII 接口，在数据的收发上它比 MII 接口少了一倍的信号线（2数据位）。它包括一个数据接口，以及一个 MAC 和 PHY 之间的管理接口。和 MII 一样，RMII 支持 10Mbps 和 100Mbps 的总线接口速度。

对于支持 MAC 层的 QuecPython 模组，也可以通过 RMII 硬件接口外挂 PHY 芯片。当前已支持多种型号，如 YT8512/SZ18201/JL1101。

**硬件结构**

RMII 接口的硬件连接图如下所示:  
![](../../../media/network-comm/nic/network_eth_RMII.png)

|  引脚   | 说明 |
| ---- | ------- |
| `RMII_RX_1` | 接收数据位1 |
| `RMII_CTL_RX` |  数据接收控制 |
| `RMII_CLK` | 时钟线 |
| `RMII_RX_0`   | 接收数据位0 |
| `RMII_TX_0` | 发送数据位0 |
| `RMII_TX_1` | 发送数据位1 |
| `RMII_CTL_TX` | 发送数据控制 |
| `RMII_INT` |  突发数据中断线 |
| `RMII_MD_IO` | 管理数据线 |
| `RMII_MD_CLK`   | 管理数据时钟线 |
| `RMII_RST_N` | 芯片复位线 |

**初始化流程**

网卡启动初始化加载流程如下图所示： 

![](../../../media/network-comm/nic/network_eth_rmii_init_flow.png)

<a id="click_jump_6.5.1.2"></a>

### 广域网 (WAN) 与 局域网 (LAN)

广域网（Wide Area Network，缩写WAN）一般指外网、公网。是链接不同地区局域网或者城域网计算机通信的远程网络。通常跨接很大的物理范围，所覆盖的范围从几十公里到几千公里，它能连接多个城市或国家，或横跨几个洲并能提供远距离通信，形成国际性的远程网络。  

局域网（Local Area Network，缩写LAN）一般指内网、私网，是局部地区形成一个区域网络，其特点就是分布地区范围有限，可大可小，大到一栋建筑与相邻建筑之间的连接，小到可以是办公室之间的联系。  

以太网根据工作方式可以分为 WAN 口和 LAN 口，WAN是外网接接入入口，对外连接网络，LAN是局域网输出接口，连接局域网设备，形成私有局域网网络，常见的设备如路由器。 

QuecPython 支持对以太网口的工作方式进行配置，根据工作环境将切换到不同的工作方式。

<a id="click_jump_6.5.1.3"></a>

### 数据流

QuecPython 下以太网卡配置不同的工作方式，具有不同的工作流程，[终端模式](ethernet-wan-example.md)（WAN）主要是将模块通过以太网，连接外部设备接入外网，[网关模式](ethernet-lan-example.md)（LAN）主要为局域网设备提供 4G 网络。

1. 终端模式（WAN）

将以太网卡配置成终端模式，此时网卡作为WAN口，为模块提供网络，模块可以通过以太网对外网进行访问。

![](../../../media/network-comm/nic/network_eth_node.png)

2. 网关模式（LAN）

将网太网卡配置成网关模式，此时网卡作为LAN口，模块为LAN局域网内设备提供网络，通过网络转发，其他模块可以通过模块 4G 网络对外进行网络访问。

![](../../../media/network-comm/nic/network_eth_gateway.png)

<a id="click_jump_6.5.1.4"></a>

### 各平台接口支持情况

下表是各平台型号对于 SPI 接口以太网及 RMII 接口 PHY 适配支持情况。不同平台对于支持以太网卡型号目前不一致，请参考[ethernet - 以太网相关功能](../../../../api_reference/peripherals/ethernet.md)，进行以太网卡选型。

|  型号\接口类型   | SPI-ETHERNET | RMII-PHY |
| :-: | :-: | :-: |
| `EC200A` | ✕ | ✓ |
| `EC600N` | ✓ | ✕ |
| `EC600U` | ✓ | ✕ |
| `EC600M/EC800M` | ✓ | ✕ |

> - ✓： 支持
> - ✕： 不支持


<a id="click_jump_6.5.2"></a>

## Ethernet API 说明

QuecPython 对于外挂 RMII 接口 PHY 设备或者外挂 SPI 接口太网卡设备提供了统一的的API接口。

以太网接口调用流程图如下所示：  
![](../../../media/network-comm/nic/network_eth_api_process.png)

### 选择以太网卡

以太网均被包含在 ethernet 模块中，不同类型网卡加载有所区别，各模块加载参数也有所差异，请参考[ethernet - 以太网相关功能](../../../../api_reference/peripherals/ethernet.md) 查看各网卡初始化传参含义。各以太网卡加载方式如下所示：

**加载 W5500**
为了方便W5500适配，对于W5500接口设置较多自定义参数，方便灵活对接。

```python
nic = ethernet.W5500(mac, ip='', subnet='', gateway='', spi_port=-1, spi_cs_pin=-1, extint_pin=-1, reset_pin=-1, work_mode=0)
```

**加载 DM9051**

```python
nic = ethernet.DM9051(mac, ip='', subnet='', gateway='', spi_port=-1, spi_cs_pin=-1, work_mode=0)
```

**加载 CH395**
```python
nic = ethernet.CH395(mac, ip='', subnet='', gateway='', spi_port=-1, spi_cs_pin=-1, extint_pin=-1, reset_pin=-1, work_mode=0)
```

**加载 YT8512/SZ18201/JL1101**
```python
nic = ethernet.YT8512H(mac, ip='', subnet='', gateway='')
```

对于不同的型号加载规则都是相同的，只是参数有所差异，以下各型号对于各参数适配情况介绍。

|  参数  | 含义 | 各型号适配情况 | 备注 |
| --- | --- | --- | --- |
| `mac` | 字节流，6字节长度的 `mac` 地址。 | 均适配该参数 | 此处 mac 自定义，需要自行根据以太网 MAC 地址规则进行匹配使用。 |
| `ip` | 以太网卡的 `ip` 地址，若值为空字符串''，表示使用默认值`192.168.1.100`。 | 均适配该参数 | 用于初始化配置网卡 ip 相关信息 |
| `subnet` | 以太网卡的子网掩码地址，若值为空字符串''，表示使用默认值`255.255.255.0`。 | 均适配该参数 | 用于初始化配置网卡 ip 相关信息 |
| `gateway` | 以太网卡的网关地址，若值为空字符串''，表示将 `ip` 地址的最后一位替换成`1`作为网关。 | 均适配该参数 | 用于初始化配置网卡 ip 相关信息 |
| `spi_port` | [SPI端口](../../../../api_reference/peripherals/machine.SPI.md)，默认值为`-1`，表示使用上次配置的值，程序中默认配置为 `SPI1` 端口。 | W5500/DM9051/CH395均适配 |  SPI-ETHERNET 外挂设备均适配 SPI 可选接口。RMII 接口使用接口是固定的，无需适配。 |
| `spi_cs_pin` | SPI 片选[GPIO管脚](../../../../api_reference/peripherals/machine.Pin.md)，默认值为`-1`，表示使用上次配置的值，程序中默认配置参考 API参考手册内具体型号。 | W5500/DM9051/CH395已适配 | SPI-ETHERNET 外挂设备均适配 SPI CS 可选PIN脚。RMII 使用接口是固定的，无需适配。 |
| `extint_pin` | 外部中断[GPIO管脚](../../../../api_reference/peripherals/machine.Pin.md)，默认值为`-1`，表示上次配置的值，程序中默认配置参考 API参考手册内具体型号。 | W5500/CH395已适配 | DM9051尚未适配中断方式数据处理。<br/> YT8512/SZ18201/JL1101 RMII 接口内部固定中断脚，不可更改。|
| `reset_pin` | 重置[GPIO管脚](../../../../api_reference/peripherals/machine.Pin.md)，默认值为`-1`, 表示上次配置的值，程序中默认配置参考 API参考手册内具体型号。 | W5500/CH395 已适配 | DM9051 尚未适配。<br/> YT8512/SZ18201/JL1101 RMII 接口内部固定 RST 脚，不可更改。 |
| `work_mode` | 以太网工作模式配置，默认为终端模式，`0`/`1` 分别表示终端模式/网关模式。终端模式表示该模块作为终端设备连接供网设备上网。网关模式表示该模块作为网关，为外部设备提供网络访问，通过`4G`上网。 | W5500/DM9051/CH395 均适配 | YT8512/SZ18201/JL1101 尚未适配，不限制工作默认配置，自行调用接口确认工作模式。 |

### 获取动态 IP 地址

在大部分网络环境中都是 dhcp 的方式获取 ip 信息，当前提供 dhcp 接口，方便在终端模式（WAN）下，模块自动获取 ip 信息，如以太网卡外接到路由器进行网络连接，此时模块直接通过调用此接口即可获取到路由器分配的 ip 地址。

```python
nic.dhcp()
```

### 设置静态IP 地址

不同的网络环境，需要不同的 IP 地址配置加入网络，故提供网卡静态IP配置接口，方便用户根据不同的网络环境进行配置使用，使得网络连通。比如为了固定设备 ip 或者组网环境内没有dhcp server可以分配 ip 情况下可以使用该接口进行配置。

```python
ip = '192.168.0.4'                  # 假设ip地址为192.168.0.4，请根据实际情况更改
subnet = '255.255.255.0'            # 假设子网掩码为255.255.255.0，请根据实际情况更改
gateway = '192.168.0.1'             # 假设网关地址为192.168.0.1，请根据实际情况更改

nic.set_addr(ip, subnet, gateway)
```

### 设置 DNS 地址

在网络中，域名解析需要使用的 DNS 服务器，将对应的域名转换成 IP 地址进行通信，提供 DNS 服务器地址配置接口，可以让用户根据自己的环境灵活配置 DNS 服务器地址。

```python
primary_dns = '8.8.8.8'             # 假设DNS首选地址为8.8.8.8，请根据实际情况更改
secondary_dns = '114.114.114.114'   # 假设DNS备选地址为114.114.114.114，请根据实际情况更改

nic.set_dns(primary_dns, secondary_dns)
```

### 启动网卡

在网络环境中可以根据需要对网卡进行开启，此时网卡上的数据将进行处理，方便用户对网卡使用进行管理。

```python
nic.set_up()
```

### 停止网卡

在网络环境中可以根据需要对网卡进行禁用，此时网卡上的数据将不会进行处理，方便用户对网卡使用进行管理。

```python
nic.set_down()
```

### 获取网卡网络信息

查询当前网卡配置信息，便于对网卡进行操作及展示，比如我们使用指定网卡进行网络通信，需要绑定网卡操作，此时需要获取网卡的网络信息。

```python
nic.ipconfig()
```

### 配置默认网卡

默认网卡对于模块网络通信以及网络转发有效。  
1. 对于模块进行 socket 通信，模块会进行路由查找，匹配对应的网卡进行发送，如果不进行 bind 网卡发送，此时网络会通过默认网卡进行发送，如果默认网卡不存在则会报错退出。
2. 对于多网卡下，网络转发进行路由查找由哪个网卡发出时，对于非本机数据，如果没有默认网卡，将直接丢弃，如果配置默认网卡将通过默认网卡进行转发。

```python
# 假设默认网卡ip地址为10.10.1.1，请根据实际情况更改
ip = '10.10.1.1'

nic.set_default_NIC(ip)
```

<a id="click_jump_6.5.3"></a>

## 应用示例

下面以 QuecPython EC600NCN_LE 开发板（下称开发板）通过 SPI1 连接 W5500 使用ethernet功能，分别以终端模式（WAN）及网关模式（LAN）介绍以太网的应用。

[模组通过以太网卡访问网络](./ethernet-wan-example.md)

[模组通过 4G 转以太网为外部设备提供网络](./ethernet-lan-example.md)

<a id="click_jump_6.5.4"></a>

## 常见问题

**1. SPI-ETHERNET 设备初始化时报错EIO 5?**

> 此报错说明SPI接线有问题，在检查以太网设备时不存在导致，请确认以太网型号是支持型号以及SPI接线、中断脚连接是否正常。

**2. RMII-PHY 设备初始化时报错OS 3?**

> 此报错是由于开机启动未初始化 MAC 层导致，MAC 层默认开机不初始化，使用 PHY 芯片时，会在第一次初始化时报该错误，重启后会初始化 MAC 层再启动网卡则正常使用。此后如果注销网卡，则会再触发此问题。故遇到该问题直接重启设备就好。

**3. 支持哪些以太网芯片？**

> 目前我们支持 SPI 接口以太网芯片 W5500/DM9051/CH395，RMII 接口 PHY 芯片 YT8512/SZ18201/JL1101。详细适配情况请参考[各平台接口支持](#click_jump_6.5.1.4)情况章节。

**4. 以太网芯片实际速率能达到多少？**

> 不同芯片平台下均有差异，不同的芯片平台速率支持可能受限于 4G 网络速率，可能受限于硬件接口传输速率，要根据具体平台而定，就目前而言W5500/DM9051/CH395支持速率基本在2 Mbps左右上下行速率，PHY芯片YT8512/SZ18201/JL1101基本保持在下行40 Mbps，上行10 Mbps。

**5. 以太网芯片支持的传输速率是多少？**

> 目前均支持10 Mbps和100 Mbps 。