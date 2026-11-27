# Ethernet NIC

This chapter introduces the compatibility and usage of Ethernet NICs in QuecPython.


<a id="click_jump_6.5.1"></a>

## Introduction

Ethernet is a computer local area network (LAN) technology. The IEEE (Institute of Electrical and Electronics Engineers) 802.3 standard specifies the technical specifications of Ethernet, including physical layer wiring, electronic signals, and media access control. Ethernet is currently the most widely used LAN technology. Ethernet can be found everywhere in daily life, such as Ethernet ports on computers and routers.

**How does QuecPython support Ethernet?**

Network communication is generally built on the TCP/IP protocol, and QuecPython supports the lightweight TCP/IP protocol stack - lwIP. According to the Ethernet interface characteristics supported by QuecPython modules, there are two interface types:

- QuecPython module + Ethernet chip (MAC + PHY, such as W5500/CH395).
- QuecPython module + PHY chip (such as YT8512/JL1101).

![](../../../media/network-comm/nic/network_eth_spi_adapter.png)

![](../../../media/network-comm/nic/network_eth_rmii_adapter.png)


<a id="click_jump_6.5.1.1"></a>

### Supported Hardware Interfaces

<a id="click_jump_6.5.1.1.1"></a>

#### SPI Interface

>SPI stands for Serial Peripheral Interface, which is a synchronous serial transmission specification introduced by Motorola and a serial peripheral chip extension interface for microcontrollers. SPI is a high-speed, full-duplex, synchronous communication bus that can send and receive data at the same time, usually achieving or even exceeding 10 Mbps.

For QuecPython modules that do not support the MAC layer, MAC + PHY chips can be supported by external SPI hardware interfaces. Currently, multiple chip models are supported, such as W5500/DM9051/CH395.

**Hardware Structure**   
The hardware connection diagram of the SPI interface is as follows:

![](../../../media/network-comm/nic/network_eth_spi.png)

|  Pin   | Description |
| ---- | ------- |
| `SCLK` | SPI clock pin |
| `MISO` | SPI master input slave output pin |
| `MOSI` | SPI master output slave input pin |
| `CS`   | SPI chip select pin, this pin can also be configured customly |
| `PIN_X` | Custom pin used for data reception interrupt trigger |
| `PIN_Y` | Custom pin used for chip reset control |

**Initialization Process**

The initialization process of the NIC is shown in the following figure:

![](../../../media/network-comm/nic/network_eth_spi_init_flow.png)

<a id="click_jump_6.5.1.1.2"></a>

#### RMII

>RMII (Reduced Media Independent Interface) is one of the standard Ethernet interfaces. It is a simplified MII interface and has fewer I/O transmissions than MII, with half the number of signal lines (2 data bits) compared to MII. It includes a data interface and a management interface between the MAC and PHY. Like MII, RMII supports bus interface speeds of 10 Mbps and 100 Mbps.

For QuecPython modules that support the MAC layer, PHY chips can also be supported by external RMII hardware interfaces. Currently, multiple chip models are supported, such as YT8512/SZ18201/JL1101.

**Hardware Structure**

The hardware connection diagram of the RMII interface is as follows:  
![](../../../media/network-comm/nic/network_eth_RMII.png)

|  Pin   | Description |
| ---- | ------- |
| `RMII_RX_1` | Receive data bit 1 |
| `RMII_CTL_RX` | Receive control |
| `RMII_CLK` | Clock line |
| `RMII_RX_0`   | Receive data bit 0 |
| `RMII_TX_0` | Transmit data bit 0 |
| `RMII_TX_1` | Transmit data bit 1 |
| `RMII_CTL_TX` | Transmit control |
| `RMII_INT` | Burst data interrupt |
| `RMII_MD_IO` | Management data line |
| `RMII_MD_CLK`   | Management data clock line |
| `RMII_RST_N` | Chip reset line |

**Initialization Process**

The NIC initialization process after startup is shown in the following figure:

![](../../../media/network-comm/nic/network_eth_rmii_init_flow.png)

<a id="click_jump_6.5.1.2"></a>

### Wide Area Network (WAN) and Local Area Network (LAN)

Wide Area Network (WAN) generally refers to the external network or public network. It is a remote network that connects LANs in different regions or metropolitan area networks. It usually spans a large physical range, covering a range from tens of kilometers to thousands of kilometers. It can connect multiple cities or countries, or span several continents to provide long-distance communication, forming an international remote network.

Local Area Network (LAN) generally refers to the internal network or private network. It is a regional network formed in a local area. Its characteristic is that the distribution area is limited and can be large or small, ranging from connecting one building with adjacent buildings to connecting offices.

Ethernet can be divided into WAN port and LAN port according to its working mode. The WAN port is the external network access port, which connects to the network externally, while the LAN port is the local area network output port, which connects to LAN devices to form a private LAN network, commonly seen in routers.

QuecPython supports configuring the working mode of the Ethernet port and switching to different modes according to the working environment.

<a id="click_jump_6.5.1.3"></a>

### Data Flow

Different working modes of the Ethernet NIC in QuecPython have different working processes. In WAN mode, the module connects to the external network device through Ethernet and can access the external network. In LAN mode, the module provides 4G network for LAN devices.

1. WAN Mode

When the working mode of the Ethernet NIC is configured to WAN mode, the NIC acts as the WAN port to provide the network for the module. The module can access the external network through Ethernet.

![](../../../media/network-comm/nic/network_eth_node.png)

2. LAN Mode

When the working mode of the Ethernet NIC is configured to LAN mode, the NIC acts as the LAN port to provide the network for LAN devices. Other modules can access the external network through the module's 4G network via network forwarding.![](../../../media/network-comm/nic/network_eth_gateway.png)

<a id="click_jump_6.5.1.4"></a>

### Supported Interface on Module Models

The table below shows the support for SPI-Ethernet and RMII-PHY interfaces on different module models. The supported Ethernet NIC models vary with different platforms. Please refer to [ethernet-Ethernet Driver](../../../../api_reference/peripherals/ethernet.md) in the "API Reference" on the QuecPython official website for details.

| Model\Interface Type | SPI-ETHERNET | RMII-PHY |
| :-: | :-: | :-: |
| `EC200A` | ✕ | ✓ |
| `EC600N` | ✓ | ✕ |
| `EC600U` | ✓ | ✕ |
| `EC600M/EC800M` | ✓ | ✕ |

> - ✓: Supported
> - ✕: Not supported


<a id="click_jump_6.5.2"></a>

## Ethernet API Description

QuecPython provides unified APIs for the device with external SPI-Ethernet or RMII-PHY interfaces.

The Ethernet interface calling process is shown in the following diagram:  
![](../../../media/network-comm/nic/network_eth_api_process.png)

### Select an Ethernet NIC

Ethernet NICs are all included in the "Ethernet" feature. The loading methods of different types of NICs and the loading parameters of each module are different. Please refer to [ethernet-Ethernet Driver](../../../../api_reference/peripherals/ethernet.md) in the "API Reference" on the QuecPython official website for details.

Loading methods of each Ethernet NIC:

**W5500**
To better configure W5500, there are many custom parameters for the W5500 interface, which makes it easy to connect.

```python
nic = ethernet.W5500(mac, ip='', subnet='', gateway='', spi_port=-1, spi_cs_pin=-1, extint_pin=-1, reset_pin=-1, work_mode=0)
```

**DM9051**

```python
nic = ethernet.DM9051(mac, ip='', subnet='', gateway='', spi_port=-1, spi_cs_pin=-1, work_mode=0)
```

**CH395**

```python
nic = ethernet.CH395(mac, ip='', subnet='', gateway='', spi_port=-1, spi_cs_pin=-1, extint_pin=-1, reset_pin=-1, work_mode=0)
```

**YT8512/SZ18201/JL1101**

```python
nic = ethernet.YT8512H(mac, ip='', subnet='', gateway='')
```

The loading rules for different models are the same, and only the parameters are different. The parameter configuration of each model is described below.

| Parameter | Description | Required by Which Model(s) | Remarks |
| --- | --- | --- | --- |
| `mac` | Byte stream. `MAC` address with a length of 6 bytes. | All models | The MAC address here is customized and needs to be matched and used according to the Ethernet MAC address rules. |
| `ip` | The IP address of the Ethernet NIC. If the value is an empty string '', the default value `192.168.1.100` will be used. | All models | Used to initialize and configure the NIC IP information |
| `subnet` | The subnet mask address of the Ethernet NIC. If the value is an empty string '', the default value `255.255.255.0` will be used. | All models | Used to initialize and configure the NIC IP information |
| `gateway` | The gateway address of the Ethernet NIC. If the value is an empty string '', the last bit of the parameter  **`ip`** will be replaced with `1` to form the gateway address. | All models | Used to initialize and configure the NIC IP information |
| `spi_port` | [SPI port](../../../../api_reference/peripherals/machine.SPI.md) number. The default value is `-1`, which means the last configured value is used. The default configuration in the program is `SPI1` | W5500/DM9051/CH395 | The SPI port number can be configured for all SPI-Ethernet interfaces. The SPI port for RMII-PHY interfaces is fixed and does not need to be configured. |
| `spi_cs_pin` | SPI CS [GPIO pin](../../../../api_reference/peripherals/machine.Pin.md). The default value is `-1`, which means the last configured value is used. Please refer to the corresponding model section in the [ethernet-Ethernet Driver](../../../../api_reference/peripherals/ethernet.md) chapter for the default configuration in the program. | W5500/DM9051/CH395 | The SPI CS GPIO pin can be configured for all SPI-Ethernet interfaces. The SPI CS GPIO pin for RMII-PHY interfaces is fixed and does not need to be configured. |
| `extint_pin` | External interrupt [GPIO pin](../../../../api_reference/peripherals/machine.Pin.md). The default value is `-1`, which means the last configured value is used. Please refer to the corresponding model section in the [ethernet-Ethernet Driver](../../../../api_reference/peripherals/ethernet.md) chapter for the default configuration in the program. | W5500/CH395 | The external interrupt [GPIO pin](../../../../api_reference/peripherals/machine.Pin.md) cannot be configured for DM9051.<br/>The external interrupt [GPIO pin](../../../../api_reference/peripherals/machine.Pin.md) is fixed for YT8512/SZ18201/JL1101 RMII-PHY interface and cannot be changed. |
| `reset_pin` | Reset [GPIO pin](../../../../api_reference/peripherals/machine.Pin.md). The default value is `-1`, which means the last configured value is used. Please refer to the corresponding model section in the [ethernet-Ethernet Driver](../../../../api_reference/peripherals/ethernet.md) chapter for the default configuration in the program. | W5500/CH395 | The reset [GPIO pin](../../../../api_reference/peripherals/machine.Pin.md) cannot be configured for DM9051.<br/><br/>The reset [GPIO pin](../../../../api_reference/peripherals/machine.Pin.md) is fixed for YT8512/SZ18201/JL1101 RMII-PHY interface and cannot be changed. |
| `work_mode` | Ethernet NIC working mode.  `0`: WAN mode. `1`: LAN mode. Default value: 0. In WAN mode, the module is connected to the network device as a terminal device for Internet access. In LAN mode, the module provides network access for external devices via `4G` as a gateway. | W5500/DM9051/CH395 | Ethernet NIC working mode cannot be configured for YT8512/SZ18201/JL1101. For these models, the default configuration for working mode is not limited. You can call the API to confirm the working mode. |

### Obtain Dynamic IP Address

In most network environments, IP information is obtained through DHCP. The DHCP API provided below makes the module automatically obtain IP information in WAN mode. For example, when an Ethernet NIC is connected to a router for network access, call the API below to obtain the router-assigned IP address.

```python
nic.dhcp()
```

### Set Static IP Address

An Ethernet NIC requires different IP address configurations to access the network in different network environments. Therefore, the API for configuring static IP addresses is provided below, then you can configure and use IP addresses according to the specific network environments. For example, this API is useful for fixing device IP addresses or when there is no DHCP server available for IP address allocation within a network.

```python
nic.set_addr(ip, subnet, gateway)
```

### Set DNS Address

In a network, DNS servers are required for domain name resolution, converting domain names into corresponding IP addresses for communication. The following API is provided to enable you to configure DNS server addresses according to your environments.

```python
nic.set_dns(primary_dns, secondary_dns)
```

### Enable NIC

Enable the NIC as needed. In this case, the data on the NIC is processed, facilitating the management of the NIC.

```python
nic.set_up()
```

### Disable NIC

Disable the NIC as needed. In this case, the data on the NIC will not be processed, facilitating the management of the NIC.

```python
nic.set_down()
```

### Get NIC Configuration

Query the current NIC configuration to operate and display the NIC. For example, when using a specific NIC for network communication, you must bind the NIC. In this case, you must get the NIC configuration.

```python
nic.ipconfig()
```

### Configure Default NIC

The default NIC is effective for module network communication and network forwarding.
1. For socket communication by the module, the module will search for the router and match the corresponding NIC for sending. If no NIC is bound for sending, the network will be sent through the default NIC. If the default NIC does not exist, an error will be reported and the operation will exit.
2. For network forwarding in a multi-NIC environment, when determining which NIC to use for forwarding non-local data, if there is no default NIC, the data will be discarded directly. If a default NIC is configured, the data will be forwarded through the default NIC.

```python
nic.set_default_NIC(ip)
```

<a id="click_jump_6.5.3"></a>

## Application Examples

The following takes the QuecPython EC600NCN_LE EVB (referred to as the EVB) connecting to W5500 through SPI1 to use Ethernet as an example to demonstrate the use of Ethernet in both WAN mode and LAN mode (LAN).

[Module Accesses the Network via Ethernet](./ethernet-wan-example.md)

[Module Provides Network Access to External Devices via 4G-to-Ethernet](./ethernet-lan-example.md)

<a id="click_jump_6.5.4"></a>

## Frequently Asked Questions

**1. What does the error code EIO 5 mean when initializing the SPI-ETHERNET device?**

> This error code indicates a problem with the SPI connection. Please check if the Ethernet device is supported and if the SPI wiring and interrupt pin connections are correct.

**2. What does the error code OS 3 mean when initializing the RMII-PHY device?**

> This error is caused by the MAC layer not being initialized during startup. By default, the MAC layer is not initialized at startup. When a PHY chip is used, this error will be reported during the first initialization. If the MAC layer is initialized after restart, the NIC can be enabled normally. If the NIC is de-registered, this error will be triggered again. Therefore, restart the device when encountering this error.

**3. Which Ethernet chips are supported?**

> Currently, SPI-Ethernet chips W5500/DM9051/CH395, and RMII-PHY chips YT8512/SZ18201/JL1101. For detailed information, please refer to [Supported Interface on Module Models](#click_jump_6.5.1.4).

**4. What is the maximum transmission speed of the Ethernet chips?**

> The maximum speed varies with the chip platform. The speed support of different chip platforms may be limited by the 4G network speed or the hardware interface transmission speed. Currently, the chips W5500/DM9051/CH395 support speeds of around 2 Mbps for both upstream and downstream, while the PHY chips YT8512/SZ18201/JL1101 support speeds of around 40 Mbps downstream and 10 Mbps upstream.

**5. What is the transmission speed supported by the Ethernet chips?**

> 10 Mbps and 100 Mbps.