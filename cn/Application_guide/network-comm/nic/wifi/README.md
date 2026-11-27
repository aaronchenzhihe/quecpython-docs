# Wi-Fi 网卡

> 本章节主要介绍 QuecPython 下 Wi-Fi 网卡适配情况，并详细介绍其在软件及硬件方面对接方法，方便用户对网卡进行开发及自定义适配。


<a id="click_jump_6.4.1"></a>

## Wi-Fi 无线网卡介绍

Wi-Fi 无线网卡是一种用于连接终端设备到无线局域网（Wireless Local Area Network，WLAN）的设备，允许设备通过无线信号进行数据通信，使设备能够无线访问互联网、共享文件和打印机等。Wi-Fi 无线网卡的工作原理是以无线电波为传输媒介，通过接收和发送无线信号来实现无线数据传输。它主要使用2.4 GHz或5 GHz频段，并基于IEEE 802.11b/g/n/ac/ax协议。

**QuecPython 上是如何支持 Wi-Fi 的？**

正常网络通信是基于 TCP/IP 模型工作的，QuecPython 支持 LwIP 协议栈（轻量开源的 TCP/IP 协议栈），因此挂载 Wi-Fi 可以选择 QuecPython + MAC + PHY 的方式（通过串口/SDIO硬件接口外挂集成MAC和PHY的芯片，如 ESP8266/ASR5803），或者直接根据 Wi-Fi 平台资源大小，移植 QuecPython 到对应芯片平台上,如 FCM360W。

![](../../../media/network-comm/nic/network_eth_spi_adapter.png)

![](../../../media/network-comm/nic/network_wifi_core.png)

<a id="click_jump_6.4.1.1"></a>

### QuecPython 支持的 Wi-Fi 网卡硬件接口

QuecPython 支持的 Wi-Fi 网卡有多种硬件连接方式：蜂窝无线通信模组通过 [UART](../../../hardware/peripheral-interfaces/UART.md) 或 SDIO 接口外接 Wi-Fi 网卡，或者在移远的 Wi-Fi 芯片中移植 QuecPython。

<a id="click_jump_6.4.1.1.1"></a>

#### UART 接口

UART是一种异步的串行通信接口协议，用于在设备之间传输数据，通过发送和接收线路实现设备之间的双向异步传输。UART接口的传输速度相对较慢，通常在几千字节每秒（B/s）到几十千字节每秒（KB/s）之间。

通过 UART 接口与蜂窝通信模组连接的 Wi-Fi 网卡，其与蜂窝通信模组之间使用 SLIP 协议进行链路通信。

SLIP 协议是指串行线路网际协议（Serial Line Internet Protocol），是在串行通信线路上支持 TCP/IP 协议的一种点对点式的链路层通信协议。

QuecPython 通过 UART 接口外接的 Wi-Fi 网卡，目前支持乐鑫的 ESP8266 与 ESP8285。

- 乐鑫 Wi-Fi 模组不支持 SLIP 协议，QuecPython 团队基于 ESP8266_RTOS_SDK 开发集成了 SLIP 协议，用户需要烧录 QuecPython 团队定制后的固件。
- 受限于串口波特率，该种连接方式仅适用于低速率场景。

**硬件连接**

串口有较简单的硬件连接，方便设备集成。
UART 硬件连接如下所示：
![](../../../media/network-comm/nic/network_wifi_uart.png)

|  引脚   | 说明 |
| ---- | ------- |
| TX | 串口数据发送引脚 |
| RX | 串口数据接收引脚 |
| GND | 接地引脚 |

**初始化流程**

串口连接具有较简单的初始化使用流程，只需要保证串口可用即可。QuecPython 初始化串口及 SLIP 不会对外部设备进行校验，只要对端设备集成 SLIP 协议即可使用，不限制设备类型。你也可以根据需要尝试其他设备集成 SLIP 进行对接。

![](../../../media/network-comm/nic/network_wifi_uart_flow.png)

<a id="click_jump_6.4.1.1.2"></a>

#### SDIO 接口

SDIO 是一种用于存储卡和其他外部设备的串行通信接口协议。它是SD卡协会制定的标准，允许设备通过 SDIO 接口与主机进行双向数据传输。SDIO 协议定义了物理接口、电气特性和通信协议，支持高速数据传输和多功能设备的连接，如无线网卡、摄像头等。SDIO 接口可以支持高速传输，速度可达到几十兆字节每秒（MB/s）。

**硬件连接**

SDIO 硬件连接如下所示：
![](../../../media/network-comm/nic/network_wifi_sdio.png)

|  引脚   | 说明 |
| ---- | ------- |
| WLAN_SLP_CLK | WLAN 睡眠时钟 |
| WLAN_PWR_EN |  WLAN 电源使能控制 |
| WLAN_SDIO_DATA3 | WLAN SDIO数据位3 |
| WLAN_SDIO_DATA2   | WLAN SDIO数据位2 |
| WLAN_SDIO_DATA1 | WLAN SDIO数据位1 |
| WLAN_SDIO_DATA0 | WLAN SDIO数据位0 |
| WLAN_SDIO_CLK | WLAN SDIO 时钟 |
| WLAN_SDIO_CMD |  WLAN SDIO 命令 |
| WLAN_WAKE | WLAN 唤醒模块 |
| WLAN_EN   | WLAN 使能控制 |
| WLAN_RST   | WLAN 重启控制 |

**初始化流程**

![](../../../media/network-comm/nic/network_wifi_sdio_flow.png)

<a id="click_jump_6.4.1.1.3"></a>

#### 内置 Wi-Fi

QuecPython 内置 Wi-Fi，是指直接将 QuecPython 移植到 Wi-Fi 芯片上，适配 Wi-Fi 的接口，直接对 Wi-Fi 芯片进行控制。

**硬件结构**

如下图所示，QuecPython 移植到 Wi-Fi 芯片，同时会适配 Wi-Fi 平台的硬件接口，对 Wi-Fi 硬件资源充分利用。

![](../../../media/network-comm/nic/network_wifi_internal.png)

<a id="click_jump_6.4.1.2"></a>

### 工作模式介绍

Wi-Fi 支持多种工作模式，当前 QuecPython 主要支持基础功能 [AP](wifi-ap-example.md) 和 [Station](wifi-station-example.md) 模式，即 Wi-Fi 作为终端连接到其他接入点入网（如路由器），或者 Wi-Fi 本身作为接入点给其他设备入网。

>AP 模式（Access Point Mode）：
- AP 模式也被称为热点模式或基础设施模式。
- 在 AP 模式下，Wi-Fi 设备充当访问点，负责创建和管理无线网络。
- AP模式的设备通常连接到有线网络，并通过无线信号将网络连接提供给其他设备，这些设备可以是 STA 模式下的客户端设备或其他 Wi-Fi 设备。
- AP 模式允许多个客户端设备同时连接到它，提供网络连接和资源共享的能力。
- 在 AP 模式下，设备通常具有固定的IP地址，可以设置网络的安全性参数，如密码和加密方式。

>STA模式（Station Mode）：
- STA 模式也被称为客户端模式或无线工作站模式。
- 在 STA 模式下，Wi-Fi 设备充当客户端，用于连接到无线网络的访问点（AP）。
- STA 模式的设备可以通过扫描可用的无线网络并选择一个AP进行连接。
- 一旦连接建立，STA 模式的设备可以与 AP 进行通信，通过无线网络访问 Internet 或其他网络资源。
- STA 模式的设备通常获取动态分配的 IP 地址，可以使用 DHCP 协议自动获取网络配置。

<a id="click_jump_6.4.1.3"></a>

### 频段介绍

Wi-Fi 技术在无线通信中使用的频段主要包括 2.4 GHz 和 5 GHz 两个频段。不同的频段无法兼容，比如使用只支持 2.4G 的网卡去连接5G的热点网络，下面对这两个频段进行详细介绍：

2.4 GHz频段：
- 2.4 GHz频段是最常用的Wi-Fi频段之一。
- 在 2.4 GHz频段，可用的无线信道有 13 个，从频率 2412 MHz 到 2484 MHz，每个信道之间相隔 5 MHz。
- 2.4 GHz 频段的优点是信号传播距离较远，穿透能力较强，可以在较长的距离内提供较好的覆盖范围。
- 2.4 GHz 频段的缺点是由于被许多其他设备（如蓝牙设备、微波炉等）使用，可能存在较多的干扰和拥塞问题。
- 在 2.4 GHz 频段中，常用的 Wi-Fi 标准包括 802.11b、802.11g 和 802.11n。

5 GHz频段：
- 5 GHz 频段是较新的 Wi-Fi 频段，也被称为高速频段。
- 在 5 GHz 频段，可用的无线信道较多，具体可用信道数量因地区而异，通常有 24 个或更多。
- 5 GHz 频段的优点是拥有更大的频谱带宽，可以提供更高的数据传输速率和更低的延迟。
- 5 GHz 频段通常较少受到干扰，因为许多其他设备更倾向于在 2.4 GHz 频段中工作。
- 5 GHz 频段中，常用的 Wi-Fi 标准包括 802.11a、802.11n、802.11ac 和802.11ax（Wi-Fi 6）。

<a id="click_jump_6.4.1.4"></a>

### 数据流

QuecPython 下 Wi-Fi 配置不同的工作方式，具有不同的工作流程，Station 模式主要是将模块连接外部设备接入外网，AP 模式主要为局域网设备提供 4G 网络。
> 对于内置 Wi-Fi 网卡设备，模块只存在 Wi-Fi 网卡，无法进行 4G 网络转发。

1. STATION

将 Wi-Fi 网卡配置成 Station模式，此时通过 Wi-Fi 网卡连接外部热点，为模块提供网络，模块可以通过外部热点（如路由器）对外网进行访问。

![](../../../media/network-comm/nic/network_wifi_station.png)
![](../../../media/network-comm/nic/network_wifi_station_single_1.png)

2. AP

将 Wi-Fi 网卡配置成 AP 模式，此时 Wi-Fi 作为接入点，模块为其他接入该接入点设备提供网络，通过网络转发，其他模块可以通过模块 4G 网络对外进行网络访问。

![](../../../media/network-comm/nic/network_wifi_ap.png)
![](../../../media/network-comm/nic/network_wifi_ap_single.png)


<a id="click_jump_6.4.1.5"></a>

### 各平台接口支持情况

下表是各平台型号对于 UART 接口 Wi-Fi 及 SDIO 接口 Wi-Fi 适配支持情况。

|  型号\接口类型   | UART Wi-Fi | SDIO Wi-Fi | 内置 Wi-Fi |
| :-: | :-: | :-: | :-: |
| `EC200A` | ✕ | ✓ | ✕ |
| `EC600N` | ✓ | ✕ | ✕ |
| `EC600U` | ✓ | ✕ | ✕ |
| `EC600M/EC800M` | ✓ | ✕ | ✕ |
| `EC600E/EC800E` | ✓ | ✕ | ✕ |
| `FCM360W` | ✕ | ✕ | ✓ |

> - ✓： 支持
> - ✕： 不支持

<a id="click_jump_6.4.2"></a>

## Wi-Fi API 说明

QuecPython 支持多种型号 Wi-Fi，其中包括外挂或者内置 Wi-Fi，由于历史原因，当前 ESP8266/ESP8285 尚未支持mpy接口，我们后续将对所有 Wi-Fi 设备接口进行统一。详细各型号接口说明请参考 Wiki下 [WLAN](../../../../api_reference/wifilib/README.md) 章节。

<a id="click_jump_6.4.2.1"></a>

### 外置 ESP8266 Wi-Fi 网卡 API 说明

由于 ESP8266 是在串口 SLIP 协议上进行支持的，为了方便对接使用拓展，我们只是提供了 WLAN.py 脚本，内置 ESP8266 控制类，方便用户根据自己需求进行接口定义并开发。

详细接口信息请参考 WIKI 下[ESP8266](../../../../api_reference/wifilib/WLAN.ESP8266.md)章节介绍。

> [点此在 github 中下载 WLAN.py 代码。](https://github.com/QuecPython/examples/tree/main/network-comm/nic/wifi)

接口调用关系如下图所示：
![](../../../media/network-comm/nic/network_wifi_esp8266_api_process.png)

#### 初始化 Wi-Fi 网卡工作模式 - 外置-ESP8266

此处初始化 Wi-Fi 网卡，确认外挂 Wi-Fi 网卡连接的串口通道，以及 Wi-Fi 网卡的工作模式。

```python
# 初始化 Wi-Fi 网卡 STA 工作模式配置
esp8266=ESP8266(UART.UART2,ESP8266.STA)
```

#### 使能AP模式

开启AP模式，用户可以配置用户名/密码建立相应的热点。需要注意，此处配置需要跟网卡初始化接口模式保持一致。

```python
# 初始化 Wi-Fi 网卡 AP 工作模式配置
esp8266=ESP8266(UART.UART2,ESP8266.AP)
# 配置建立热点的用户名/密码
esp8266.ap('user', 'password')
```

#### STATION模式

连接热点，根据热点的用户名/密码，连接对应的 Wi-Fi 接入点。

```python
# 初始化 Wi-Fi 网卡 STA 工作模式配置
esp8266=ESP8266(UART.UART2,ESP8266.STA)
# 根据热点的用户名/密码，连接对应的 Wi-Fi 接入点
esp8266.station('user', 'password')
```

#### WEB 配网

开启 WEB 配网，主要在 STATION 模式下，通过 WEB 配置连接 Wi-Fi 热点。

```python
# 初始化 Wi-Fi 网卡 STA 工作模式配置
esp8266=ESP8266(UART.UART2,ESP8266.STA)
# 配置用于开启WEB配网热点的用户名/密码
esp8266.web_config('user', 'password')
```

#### SMARTCONFIG配网

根据乐鑫官方提供的智能配网方案。主要应用在 STATION 模式下，通过 SMARTCONFIG 配置连接到 Wi-Fi 热点。

```python
# 以使用AIRKISS & ESPTOUCH协议为例，请根据实际情况更改
protocol = 3

# 初始化 Wi-Fi 网卡 STA 工作模式配置
esp8266=ESP8266(UART.UART2,ESP8266.STA)
# 开启smartconfig，protocol可选择ESPTOUCH协议、AIRKISS协议、AIRKISS & ESPTOUCH协议和ESPTOUCH-V2协议中的一种进行配置
esp8266.smartconfig(protocol)
```

#### 路由配置

提供了 Wi-Fi 网卡数据通过 4G 网卡转发的方法，需要进行两步，通过配置默认网卡接口确认由哪个网卡转发，另外需要配置路由转发设置，通过指定转发 IP 规则，只有热点的网段内数据才进行转发。一般当 Wi-Fi 网卡工作在 AP 模式下时，由 4G 网络为 Wi-Fi 终端提供网络服务。

> 1. 对于模块进行 socket 通信，模块会进行路由查找，匹配对应的网卡进行发送，如果不进行 bind 网卡发送，此时网络会通过默认网卡进行发送，如果默认网卡不存在则会报错退出。
> 2. 对于多网卡下，网络转发进行路由查找由哪个网卡发出时，对于非本机数据，如果没有默认网卡，将直接丢弃，如果配置默认网卡将通过默认网卡进行转发。对于ESP8266需要判断转发规则。

```python
# 以子网掩码为255.255.255.0，ap模式网段为192.168.4.0 ~ 192.168.4.255为例

default_NIC_ip = '10.10.1.1'    # 假设默认网卡ip地址为10.10.1.1，请根据实际情况更改
segment_ip = '192.168.4.0'      # 假设ap模式网段范围是[192.168.4.0 ~ 192.168.4.255]，请根据实际情况更改
mask = '255.255.255.0'          # 假设子网掩码为255.255.255.0，请根据实际情况更改

# 配置默认网卡，通过默认网卡进行网络转发
esp8266.set_default_NIC(default_NIC_ip)

#添加路由规则，只有该网段IP才进行转发
esp8266.router_add(segment_ip, mask)
```

#### 网络配置

此处网络配置信息，并非是对 ESP8266 侧网络进行配置，此处只是针对串口 SLIP 网卡进行配置，需要保证通信可靠性，不提供 IP 配置，只提供了 DNS 配置方法。

```python
pri_dns = '8.8.8.8'             # 假设DNS首选地址为8.8.8.8，请根据实际情况更改
pri_dns = '114.114.114.114'     # 假设DNS备选地址为114.114.114.114，请根据实际情况更改

# 配置网卡DNS服务器
esp8266.set_dns(pri_dns, sec_dns)

# 网卡IP信息查询
esp8266.ipconfig()
```

#### 状态查询 - 外置-ESP8266

通过该接口获取网卡的当前状态，正常使用中，需要对网卡的状态进行对应的判断，然后进行操作。

```python
# 获取网卡当前工作状态
esp8266.status()
```

#### OTA 升级

此处 OTA 升级只是针对 Wi-Fi 网卡，触发网卡进行 OTA 升级。

> 升级过程中只可查询当前状态，其他操作均不可执行。升级过程中根据回调进行通知。

```python
# 以固件下载地址为http://www.example.com/fota.bin为例，请根据实际情况更改
url='http://www.example.com/fota.bin'

# esp8266 OTA升级接口，通过传入固件下载地址，实现 Wi-Fi 网卡固件升级，此功能需要 Wi-Fi 网卡配置成STATION模式使用
esp8266.ota(url)
```

<a id="click_jump_6.4.2.2"></a>

### 通用 Wi-Fi 网卡 API 说明

此套接口是完全按照 MicroPython 官方接口进行开发适配。适用于 SDIO 外挂 Wi-Fi 网卡以及内置 Wi-Fi 网卡。

当前信息只是对于通用 Wi-Fi 接口使用说明，部分型号适配根据实际情况实现有所差异，详细接口信息请参考 Wiki 下 [Wi-Fi通信](../../../../api_reference/wifilib/README.md) 章节介绍。

![](../../../media/network-comm/nic/network_wifi_api_process.png)

#### 初始化 Wi-Fi 网卡工作模式 - 通用

初始化 Wi-Fi 网卡，确认 Wi-Fi 网卡工作模式。

**加载 FCM360W/FC41D**

```python
nic = network.WLAN(network.STA_IF)
```

**加载ASR5803**

```python
nic = network.ASR5803W(network.STA_IF)
```

#### 使能 AP 模式

开启AP模式，用户可以配置用户名/密码建立相应的热点。需要注意，此处配置需要跟网卡初始化接口模式保持一致。

```python
# 初始化 Wi-Fi 网卡 AP 工作模式
nic = network.WLAN(network.AP_IF)
# 配置建立热点的用户名/密码
nic.config('ssid'='user', 'key'='password')
# 激活 Wi-Fi 网卡
nic.active(True)
```

#### STATION 模式

连接热点，根据热点的用户名密码，连接对应的 Wi-Fi 接入点。

方式1：
```python
# 初始化 Wi-Fi 网卡 STATION 工作模式
nic = network.WLAN(network.STA_IF)
# 连接热点的用户名/密码。
nic.connect('user', 'password')
```

方式2：
```python
# 初始化 Wi-Fi 网卡 STATION 工作模式
nic = network.WLAN(network.STA_IF)
# 配置连接热点的用户名/密码
nic.config('ssid'='user', 'key'='password')
# 激活 Wi-Fi 网卡
nic.active(True)
```

#### 配网

当前只能通过 Python 接口配置网路，其他方式正在完善中。

#### 网卡配置

提供了网络配置方法，方便用户根据需要进行网络配置。比如需要网卡在组网环境中使用静态IP，可以通过该接口进行配置，或者AP模式下，默认网段与其他网段有冲突，可以修改到其他网段进行使用。

```python
ip = '192.168.0.4'                  # 假设ip地址为192.168.0.4，请根据实际情况更改
subnet = '255.255.255.0'            # 假设子网掩码为255.255.255.0，请根据实际情况更改
gateway = '192.168.0.1'             # 假设网关地址为192.168.0.1，请根据实际情况更改
primary_dns = '8.8.8.8'             # 假设DNS首选地址为8.8.8.8，请根据实际情况更改
secondary_dns = '114.114.114.114'   # 假设DNS备选地址为114.114.114.114，请根据实际情况更改

# 网卡静态IP配置
nic.ifconfig((ip, subnet, gateway, primary_dns, secondary_dns))

# 网卡IP信息获取
nic.ifconfig()
```

#### 参数配置及查询

网卡提供了一系列配置及查询方法，方便用户调试或者应用 Wi-Fi 网卡。

```python
# 网卡配置与查询，可以配置ssid、bssid、channel、RSSI、security和hidden等信息。具体支持情况请查看Wiki下对应型号进行确认
nic.config('param')
nic.config(param=value)

# 网卡激活与去激活查询与配置接口
nic.active([param])

# 网卡STA模式下，与路由器断开连接
nic.disconnect()

# 网卡STA模式下，表示是否连接到路由器，网卡 AP 模式下，表示是否有客户端连接
nic.isconnected()
```

#### 状态查询 - 通用

通过该接口获取网卡的当前状态及一些属性值，正常使用中，需要对网卡的状态进行对应的判断。


```python
#网卡状态查询，当不进行传参时只是查询网卡当前状态，对于传参时查询具体的参数信息。
nic.status([param])
```


<a id="click_jump_6.4.3"></a>

## 应用示例

下面以 QuecPython EC600NCN_LE 开发板（下称开发板）通过 UART2 连接 ESP8266 使用 Wi-Fi 功能为例进行演示：

[6.4.3.1 Station 模式](./wifi-station-example.md)

[6.4.3.2 AP 模式](./wifi-ap-example.md)

<a id="click_jump_6.4.4"></a>

## 常见问题

**1. 为什么 ASR5803 启动后搜索不到 Wi-Fi 信号？**
> a. 硬件 GPIO 设置是否正确。
> b.MAC地址是否合法（符合 IEEE 中对于 Wi-Fi AP模式MAC 设置的规定）。
> c. Wi-Fi 距离过远或信号质量差。
> d. 其他因素：如无线干扰、安全设置、网络配置等。

**2. Wi-Fi 连接不上。**

> a. 热点频段与 Wi-Fi 网卡是否在一个频段，比如2.4G/5G。
> b. 热点信息是否存在乱码，Wi-Fi 不支持特殊字符。


**3. 支持哪些 Wi-Fi 芯片？**

当前通过串口外挂 Wi-Fi 芯片支持 ESP8266/ESP8285/FC41D，SDIO 外挂 Wi-Fi 芯片支持 ASR5803，内置 Wi-Fi 芯片支持 FCM360W。
