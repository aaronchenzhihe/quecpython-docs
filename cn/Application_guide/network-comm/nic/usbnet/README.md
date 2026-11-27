# USB 网卡

> 本章主要介绍 QuecPython 下 USB 网卡适配情况，以及如何应用。


## USB 网卡介绍

USB网卡（USB Network Adapter）是一种外部设备，用于在计算机或其他设备上提供以太网连接。它通过USB接口连接到计算机，并通过USB总线进行数据传输。USB网卡通常用于为计算机添加或扩展网络连接功能，特别是在没有内置以太网接口或需要额外网络连接的情况下。

**QuecPython 上是如何支持 USB 网卡的？**  

USB 协议定义了设备类的类别码信息，可以用来识别设备并且加载设备驱动。这种代码信息有包含Base Class（[基类]）、SubClass（[子类]）、Protocol（[协议]）一共占有3个字节。常见的支持USB网卡的协议ECM/RNDIS/MBIM（USB 协议下的子类）。
- ECM（Ethernet Networking Control Model，以太网网络控制模型）用于在设备和主机之间交换以太网帧数据。ECM设备的一般用例是LAN/WLAN的点对点以太网适配器。

- RNDIS（Remote network Driver Interface Specification，远程网络驱动接口规范） 协议是微软对于 ECM 的变种实现，主要用于简化 Windows 平台中 USB 网络设备的驱动开发。

- MBIM（Mobile Broadband Interface Model，移动宽带接口模型），专用于3G/4G/5G模块。它定义了移动宽带设备与 PC 端的接口标准。

QuecPython 通过支持 USB 协议下 ECM/RNDIS/MBIM 子类协议实现对 USB 网卡适配，工作场景基本都是通过 USB 口给其他设备供网。


### 硬件结构

![USBNET硬件结构图](../../../media/network-comm/nic/network_usbnet_usb.png)

|  引脚   | 说明 |
| ---- | ------- |
| `VCC` | 提供电源供电给连接的USB设备 |
| `D-` |  数据线负极 |
| `D+` |  数据线正极 |
| `GND`   | 提供电路的接地 |


### 初始化流程

网卡初始化加载流程如下图所示：

![](../../../media/network-comm/nic/network_usbnet_usb_flow.png)


### 数据流

![](../../../media/network-comm/nic/network_usbnet.png)


### 工作模式

USB网卡是类以太网卡的，其工作方式与以太网卡相似，可以根据工作方式分为LAN/WAN（请参考[以太网章节介绍](../ethernet/README.md)）。

QuecPython在使用中以LAN的方式使用，通过 4G 给 USB 网卡供网。


### 各平台支持情况

下表是 USB 网卡在各平台适配支持情况。

<table>
    <tr>
        <th colspan = '2'>网卡类型</th>
        <th colspan = '1'>支持型号</th>
    </tr>
    <tr>
        <td rowspan = "3">USB网卡</td>
        <td>RNDIS</td>
        <td>除 BGxx 系列模块和 Wi-Fi 模块外，其余模块均支持</td>
    </tr>
    <tr>
        <td>ECM</td>
        <td>除 BGxx 系列模块和 Wi-Fi 模块外，其余模块均支持</td>
    </tr>
        <tr>
        <td>MBIM</td>
        <td>BGxx 系列模块支持</td>
    </tr>
</table>


## USBNET API 说明

USBNET 功能较为固定，所以操作上较简单，无网络相关配置。

![](../../../media/network-comm/nic/network_usbnet_api_process.png)

### 打开 USB 网卡

打开 USB 网卡，默认网卡接口是存在的，操作此接口后，USB 网卡可以正常连接网络使用。

```python
USBNET.open()
```

### 配置 USB 网卡工作模式

USB 网卡支持 ECM/RNDIS 协议，当前可以选择两种 Type_ECM/Type_RNDIS，需要根据自己使用的环境进行配置使用。

```python
USBNET.set_worktype(USBNET.Type_RNDIS)
```

### 查询 USB 网卡工作模式

查询 USB 网卡工作模式，主要在复杂情况下，判断网卡工作在什么模式下进行什么操作，也在调试时判断工作模式是否和预期一样。

```python
USBNET.get_worktype()
```

### 查询 USB 网卡工作状态

查询 USB 网卡工作状态，方便处理各种异常情况以及调试。

```python
USBNET.get_status()
```

### 关闭 USB 网卡

与打开 USB 网卡逆向操作，操作后会让网卡不可用，方便根据需要对网卡进行控制。

```python
USBNET.close()
```


## 应用示例

[Windows 系统使用 USB 网卡](./usbnet-windows-example.md)  

[Android 系统使用 USB 网卡](./usbnet-android-example.md)  

[Linux 系统使用 USB 网卡](./usbnet-linux-example.md)


## 常见异常处理

**1. 为什么同一台 USB 网卡设备连接手机后有的手机正常有的手机没有网络？**

> a. 检查网卡工作模式是否设置为 ECM 模式，RNDIS 模式不是所有的手机都支持。  
> b. 查看手机是否需要开启 OTG 功能，对于 OPPO 及其旗下品牌手机（如一加、IQOO等）需要在系统设置中打开 OTG 模式。

**2. 使用展锐 8910 的 USB 网卡功能，首次以默认 ECM 模式 open 后，电脑连接不上网络？**

> 展锐 8910 的 USB 网卡功能需要NAT功能支持，默认开机不启动NAT，需要启动NAT后才能正常使用 USB 网卡功能。请参考 Wiki 上对应 [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) 章节。展锐8910具体模块型号，可参考[模组型号与平台的对应关系](../../../background/hardware-platform.md#展锐平台)。

**3. 展锐8850使用 USB 网卡设置 RNDIS 后打开正常，关闭后再打开返回 -1。**

> 需要进行 NAT 模式的设置，具体用法请参考 Wiki 上对应 [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) 章节。展锐8850具体模块型号，可参考[模组型号与平台的对应关系](../../../background/hardware-platform.md#展锐平台)。

**4. 为什么我的手机状态栏没有`<···>`样图标，但网络可以使用？**

> 这是正常情况，由于手机厂商差异，某些型号的手机（如一加 9RT、红米 Note 8等）是没有`<···>`图标的。

**5. 电脑使用正常，但是手机却无法识别USB网卡？**

> a. 由于模块 MAC 地址导致手机无法正常识别，可以尝试使用 MAC 地址设置接口，更新 MAC 地址。  
> b. 由于手机厂商差异，无法识别 USB 网卡。

**6. 电脑上使用展锐模组的 USB 网卡功能，从 ECM 模式切换至 RNDIS 模式后，为什么电脑网卡接口页面依然显示ECM模式网卡？**

> 如下图所示，查询到网卡模式为 RNDIS，但网络适配器显示的为 ECM 网卡。
> 
> ![](../../../media/network-comm/nic/network_usbnet_Q&A_1.png)
> 
> 展锐平台由于使用驱动的特殊性，需要在切换网卡的工作模式后重新加载 RNDIS 网卡驱动，具体操作如下：
> 
> a. 进入`设备管理器界面`，选择`网络适配器`，选择 `Quectel ECM Adapter`，右键选择`更新驱动`
> ![](../../../media/network-comm/nic/network_usbnet_Q&A_2.png)
> 
> b. 选择`浏览我的计算机以查找驱动程序软件`
> ![](../../../media/network-comm/nic/network_usbnet_Q&A_3.png)
> 
> c. 选择`让我从计算机上的可用驱动列表中选取`
> ![](../../../media/network-comm/nic/network_usbnet_Q&A_4.png)
> 
> d. 双击`Remote NDIS based Internet Sharing Device`，切换到 RNDIS 驱动
> ![](../../../media/network-comm/nic/network_usbnet_Q&A_5.png)
> 
> e. 成功切换后如图所示，网卡驱动变更并被识别到
> ![](../../../media/network-comm/nic/network_usbnet_Q&A_7.png)