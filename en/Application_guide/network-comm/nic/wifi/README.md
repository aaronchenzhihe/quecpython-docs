# Wi-Fi NIC

> This section introduces the compatibility of Wi-Fi NICs in QuecPython solution, and provides detailed information on their software and hardware integration methods, making it convenient for users to develop and customize the NIC.

Contents  
[Introduction to Wi-Fi NIC](#click_jump_6.4.1)  
&emsp;[Supported Wi-Fi NIC Hardware Interfaces in QuecPython](#click_jump_6.4.1.1)  
&emsp;&emsp;[UART](#click_jump_6.4.1.1.1)  
&emsp;&emsp;&emsp;&emsp;[Hardware Structure](#click_jump_6.4.1.1.1)  
&emsp;&emsp;&emsp;&emsp;[Initialization Process](#click_jump_6.4.1.1.1)  
&emsp;&emsp;[SDIO](#click_jump_6.4.1.1.2 )  
&emsp;&emsp;&emsp;&emsp;[Hardware Structure](#click_jump_6.4.1.1.2 )  
&emsp;&emsp;&emsp;&emsp;[Initialization Process](#click_jump_6.4.1.1.2)  
&emsp;&emsp;[Built-in Wi-Fi](#click_jump_6.4.1.1.3)  
&emsp;&emsp;&emsp;&emsp;[Hardware Structure](#click_jump_6.4.1.1.3)  
&emsp;&emsp;&emsp;&emsp;[Initialization Process](#click_jump_6.4.1.1.3)  
&emsp;[Operating Mode](#click_jump_6.4.1.2)  
&emsp;[Frequency Band](#click_jump_6.4.1.3)  
&emsp;[Data Flow](#click_jump_6.4.1.4)  
&emsp;[Module Compatibility](#click_jump_6.4.1.5)  
[Wi-Fi API Description](#click_jump_6.4.2)  
&emsp;[External ESP8266 Wi-Fi NIC API Description](#click_jump_6.4.2.1)  
&emsp;[General Wi-Fi NIC API Description](#click_jump_6.4.2.2)  
[Application Examples](#click_jump_6.4.3)  
&emsp;[Station Mode](#click_jump_6.4.3)  
&emsp;[AP Mode](#click_jump_6.4.3)  
[Common Exception Handling](#click_jump_6.4.4)  

<a id="click_jump_6.4.1"></a>

## Introduction to Wi-Fi NIC

A Wi-Fi NIC is a device that connects terminal devices to a Wireless Local Area Network (WLAN). It allows devices to communicate wirelessly through radio signals, enabling the wireless access to the Internet,  shared files, and printers. The working principle of a Wi-Fi NIC is to use radio waves as the transmission medium to achieve wireless data transfer through receiving and sending wireless signals. It mainly uses the 2.4 GHz or 5 GHz frequency bands and is based on the IEEE 802.11b/g/n/ac/ax protocols.

**How does QuecPython support Wi-Fi?**

Normal network communication is based on the TCP/IP model, and QuecPython supports the LwIP protocol stack (a lightweight open-source TCP/IP protocol stack). Therefore, you can choose the QuecPython + MAC + PHY method to mount Wi-Fi (integrating MAC and PHY chips through the UART/SDIO hardware interface, such as ESP8266/ASR5803) or by porting QuecPython to the corresponding chip platform, such as FCM360W, based on the size of the Wi-Fi platform resources.

![](../../../media/network-comm/nic/network_eth_spi_adapter.png)

![](../../../media/network-comm/nic/network_wifi_core.png)

<a id="click_jump_6.4.1.1"></a>

### Supported Wi-Fi NIC Hardware Interfaces in QuecPython

QuecPython supports multiple hardware connection methods for Wi-Fi NICs: cellular wireless communication modules connects to Wi-Fi NICs via UART or SDIO interfaces, or port QuecPython to Wi-Fi chips of Quectel.

<a id="click_jump_6.4.1.1.1"></a>

#### UART Interface

UART is an asynchronous serial communication interface protocol used for data transmission between devices. It enables bidirectional asynchronous transmission between devices through the transmission and reception lines. The UART interface has a relatively slow transmission speed, usually between a few kilobytes per second (B/s) and several tens of kilobytes per second (KB/s).

For Wi-Fi NICs connected to cellular communication modules via the UART interface, SLIP protocol is used for link communication between the NIC and the cellular communication module.

SLIP protocol refers to Serial Line Internet Protocol, which is a point-to-point link layer communication protocol that supports TCP/IP protocol over serial communication lines.

QuecPython currently supports Espressif's ESP8266 and ESP8285 Wi-Fi modules connected via the UART interface.

- Espressif's Wi-Fi modules do not support SLIP protocol. The QuecPython team has developed a customized firmware based on ESP8266_RTOS_SDK that integrates the SLIP protocol. Users need to flash the customized firmware provided by the QuecPython team.
- Due to the limitations of the serial port baud rate, this connection method is only suitable for low-speed scenarios.

**Hardware Connection**

The UART interface has a relatively simple hardware connection, making it easy to integrate into devices. 
The hardware connection for UART is as follows:  
![](../../../media/network-comm/nic/network_wifi_uart.png)

|  Pin   | Description |
| ---- | ------- |
| TX | Serial data transmission pin |
| RX | Serial data reception pin |
| GND | Ground pin |

**Initialization Process**

The UART connection has a simple initialization process, only requiring the availability of the serial port. QuecPython does not verify external devices during the initialization of the serial port and SLIP. As long as the end device integrates the SLIP protocol, it can be used without restrictions on the device type. You can also try integrating SLIP with other devices as needed.  

![](../../../media/network-comm/nic/network_wifi_uart_flow.png)

<a id="click_jump_6.4.1.1.2"></a>

#### SDIO Interface

SDIO is a serial communication interface protocol used for storage cards and other external devices. It is a standard developed by the SD Card Association, which allows devices to perform bidirectional data transfer with the host through the SDIO interface. The SDIO protocol defines the physical interface, electrical characteristics, and communication protocol, supporting high-speed data transfer and the connection of multifunctional devices such as Wi-Fi NICs and cameras. The SDIO interface supports high-speed transmission with speeds of up to several tens of megabytes per second (MB/s).

**Hardware Connection**

The hardware connection for SDIO is as follows:  
![](../../../media/network-comm/nic/network_wifi_sdio.png)

| Pin | Description |
| ---- | ------- |
| WLAN_SLP_CLK | WLAN sleep clock |
| WLAN_PWR_EN | WLAN power enable control |
| WLAN_SDIO_DATA3 | WLAN SDIO data bit 3 |
| WLAN_SDIO_DATA2   | WLAN SDIO data bit 2 |
| WLAN_SDIO_DATA1 | WLAN SDIO data bit 1 |
| WLAN_SDIO_DATA0 | WLAN SDIO data bit 0 |
| WLAN_SDIO_CLK | WLAN SDIO clock |
| WLAN_SDIO_CMD | WLAN SDIO command |
| WLAN_WAKE | WLAN wake-up module |
| WLAN_EN   | WLAN enable control |
| WLAN_RST   | WLAN reset control |

**Initialization Process**

![](../../../media/network-comm/nic/network_wifi_sdio_flow.png)

<a id="click_jump_6.4.1.1.3"></a>

#### Built-in Wi-Fi

The built-in Wi-Fi of QuecPython refers to the direct porting of QuecPython onto a Wi-Fi chip, adapting the interfaces for Wi-Fi, and controlling the Wi-Fi chip directly.

**Hardware Structure**

As shown in the figure below, QuecPython is ported to the Wi-Fi chip, and the hardware interface of the Wi-Fi platform is also adapted to fully utilize the Wi-Fi hardware resources.

![](../../../media/network-comm/nic/network_wifi_internal.png)

<a id="click_jump_6.4.1.2"></a>

### Introduction to Working Modes

Wi-Fi supports multiple working modes. Currently, QuecPython mainly supports the basic functions of AP and Station modes, which means that Wi-Fi can be used as a terminal to connect to other access points (such as routers), or Wi-Fi itself can be used as an access point for other devices to connect to the network.

>AP Mode (Access Point Mode):
- AP mode is also known as hotspot mode or infrastructure mode.
- In AP mode, the Wi-Fi device acts as an access point and is responsible for creating and managing the wireless network.
- Devices in AP mode are usually connected to a wired network and provide network connectivity to other devices via wireless signals, which can be client devices in STA mode or other Wi-Fi devices.
- AP mode allows multiple client devices to connect to the access point simultaneously, providing network connectivity and the ability to share resources.
- In AP mode, devices usually have fixed IP addresses and can set network security parameters such as passwords and encryption methods.

STA Mode (Station Mode):
- STA mode is also known as client mode or wireless workstation mode.
- In STA mode, the Wi-Fi device acts as a client to connect to an access point (AP) of a wireless network.
- Devices in STA mode can scan for available wireless networks and select an AP to connect to.
- Once the connection is established, devices in STA mode can communicate with the AP and access the Internet or other network resources through the wireless network.
- Devices in STA mode usually obtain dynamically assigned IP addresses and can automatically obtain network configurations through the DHCP protocol.

<a id="click_jump_6.4.1.3"></a>

### Introduction to Frequency Bands

The frequency bands used in Wi-Fi technology for wireless communication mainly include the 2.4 GHz and 5 GHz frequency bands. Different frequency bands are not compatible. For example, a Wi-Fi NIC that only supports 2.4 GHz cannot be connected to a 5 GHz hotspot network. The following provides a detailed introduction to these two frequency bands:

2.4 GHz Frequency Band:
- The 2.4 GHz frequency band is one of the most commonly used Wi-Fi frequency bands.
- In the 2.4 GHz frequency band, there are 13 available wireless channels, ranging from 2412 MHz to 2484 MHz, with a 5 MHz spacing between each channel.
- The advantage of the 2.4 GHz frequency band is that the signal can propagate over a longer distance and has strong penetration ability, providing better coverage over a longer distance.
- Due to the 2.4 GHz frequency band being used by many other devices (such as Bluetooth devices, microwave ovens, etc.), there may be more interference and congestion issues.
- In the 2.4 GHz frequency band, commonly used Wi-Fi standards include 802.11b, 802.11g, and 802.11n.

5 GHz Frequency Band:
- The 5 GHz frequency band is a new Wi-Fi frequency band, also known as the high-speed frequency band.
- In the 5 GHz frequency band, there are more available wireless channels, with the specific number of available channels varying by region, usually 24 or more.
- The advantage of the 5 GHz frequency band is that it has a larger spectrum bandwidth, which can provide higher data transmission rates and lower latency.
- The 5 GHz frequency band is usually less susceptible to interference because many other devices tend to work in the 2.4 GHz frequency band.
- In the 5 GHz frequency band, commonly used Wi-Fi standards include 802.11a, 802.11n, 802.11ac, and 802.11ax (Wi-Fi 6).

<a id="click_jump_6.4.1.4"></a>

### Data Flow

In QuecPython solution, configuring Wi-Fi for different working modes involves distinct workflows. In Station mode, the module is connected to external devices to access the external Internet, while the AP mode provides a 4G network to local area devices.

>Note: For built-in Wi-Fi NICs, the module only has a Wi-Fi NIC and cannot perform 4G network forwarding.

1. STATION

Configuring the Wi-Fi NIC as Station mode allows the module to connect to an external hotspot through the Wi-Fi NIC, providing network access for the module. The module can access the Internet through the external hotspot (such as a router).

![](../../../media/network-comm/nic/network_wifi_station.png)
![](../../../media/network-comm/nic/network_wifi_station_single_1.png)

2. AP

Configuring the Wi-Fi NIC as AP mode allows Wi-Fi to act as an access point, providing network access for other devices that connect to this access point. Through network forwarding, other modules can access the Internet through the 4G network of the module.

![](../../../media/network-comm/nic/network_wifi_ap.png)
![](../../../media/network-comm/nic/network_wifi_ap_single.png)


<a id="click_jump_6.4.1.5"></a>

### Module Compatibility

The following table shows the support for UART interface Wi-Fi and SDIO interface Wi-Fi adaptation on different modules.

| Module\Interface Type | UART Wi-Fi | SDIO Wi-Fi | Wi-Fi only module |
| :-: | :-: | :-: | :-: |
| `EC200A` | ✕ | ✓ | ✕ |
| `EC600N` | ✓ | ✕ | ✕ |
| `EC600U` | ✓ | ✕ | ✕ |
| `EC600M/EC800M` | ✓ | ✕ | ✕ |
| `FCM360W` | ✕ | ✕ | ✓ |
>Notes:
> - ✓: Supported
> - ✕: Not supported

<a id="click_jump_6.4.2"></a>

## Wi-Fi API Description

QuecPython supports various models of Wi-Fi, including external or built-in Wi-Fi. Due to historical reasons, currently,  ESP8266/ESP8285 does not support the mpy interface. We will unify all Wi-Fi device interfaces in the future. For detailed interface descriptions of each model, please refer to the [WLAN](../../../../api_reference/wifilib/WLAN.md) section on the Wiki.

<a id="click_jump_6.4.2.1"></a>

### External ESP8266 Wi-Fi NIC API Description

Since the ESP8266 is supported on the serial SLIP protocol, in order to facilitate docking and expansion, we only provide the `WLAN.py` script, which contains the built-in ESP8266 control class. This allows you to define and develop interfaces according to your own needs.

For detailed interface information, please refer to the [ESP8266](../../../../api_reference/wifilib/WLAN.ESP8266.md) section on the Wiki. The content of the ESP8266 control script can be found in WLAN.py.

The interface call relationship is shown in the following figure:  
![](../../../media/network-comm/nic/network_wifi_esp8266_api_process.png)

#### Initialize Wi-Fi NIC Working Mode

Initializing the Wi-Fi NIC to confirm the serial port channel connected to the external Wi-Fi NIC and the working mode of the Wi-Fi NIC.

```python
# Initialize the Wi-Fi NIC STA mode configuration
esp8266=ESP8266(UART.UART2,ESP8266.STA)
```

#### Enable AP Mode

Enable AP mode, and you can configure the username and password to establish the corresponding hotspot. Note that the configuration here needs to be consistent with the mode set in the Wi-Fi NIC initialization interface.

```python
# Initialize the Wi-Fi NIC AP working mode configuration
esp8266=ESP8266(UART.UART2,ESP8266.AP)
# Configure the username/password for hotspot establishment
esp8266.ap('user', 'password')
```

#### STATION Mode

Connect to a hotspot and connect to the corresponding Wi-Fi access point based on the username and password of the hotspot.

```python
# Initialize the Wi-Fi NIC STA working mode configuration
esp8266=ESP8266(UART.UART2,ESP8266.STA)
# Connect to a hotspot and connect to the corresponding Wi-Fi access point based on the username and password of the hotspot
esp8266.station('user', 'password')
```

#### WEB Configuration

Enable WEB configuration. Configure the connection to the Wi-Fi hotspot through the WEB In STATION mode.

```python
# Initialize the Wi-Fi NIC STA working mode configuration
esp8266=ESP8266(UART.UART2,ESP8266.STA)
# Configure the username and password required for enabling a WEB configuration hotspot
esp8266.web_config('user', 'password')
```

#### SMARTCONFIG Configuration

Based on the smart configuration solution provided by Espressif, it is mainly used in STATION mode to configure the connection to the Wi-Fi hotspot through SMARTCONFIG.

```python
# Initialize the Wi-Fi NIC STA working mode configuration
esp8266=ESP8266(UART.UART2,ESP8266.STA)
# Enable smartconfig. The mode can be configured as ESPTOUCH mode / AIRKISS mode / ESPTOUCH mode and AIRKISS mode / ESPTOUCH-V2 mode
esp8266.smartconfig(mode)
```

#### Router Configuration

Provides a method for the Wi-Fi NIC data to be forwarded through the 4G NIC. Two steps are required: configuring the default NIC to confirm which NIC is used for forwarding, and configuring the routing forwarding settings by specifying the forwarding IP rule. Only data within the hotspot's network segment will be forwarded. Generally, when the Wi-Fi NIC is working in AP mode, the 4G network provides network services to Wi-Fi terminals.

> 1. When the module performs socket communication, the module will perform routing lookup and match the corresponding NIC for sending. If no NIC is bound for sending, the network will be sent through the default NIC. If the default NIC does not exist, an error will be reported and exited.
> 2. In the case of multiple NICs, when the network forwarding performs routing lookup to determine which NIC to send from, for non-local data, if there is no default NIC, it will be directly discarded. If a default NIC is configured, it will be forwarded through the default NIC. For ESP8266, it is necessary to determine the forwarding rules.

```python
# Configure the default NIC to perform network forwarding through the default NIC
esp8266.set_default_NIC(ip)

# Add a routing rule to forward only the IP in the specified network segment
esp8266.router_add(ip, mask)
```

#### Network Configuration

The network configuration information is not for configuring the ESP8266 network, but for configuring the serial SLIP NIC to ensure communication reliability. It does not provide IP configuration, but provide DNS configuration methods.

```python
# Configure the DNS server for the NIC
esp8266.set_dns(pri_dns, sec_dns)

# Query the IP information of the NIC
esp8266.ipconfig()
```

#### Status Query

Use this interface to obtain the current status of the NIC. During normal use, it is necessary to judge the status of the NIC and then perform corresponding operations.

```python
# Get the current working status of the NIC
esp8266.status()
```

#### OTA Upgrade

The OTA upgrade is only for the Wi-Fi NIC. Trigger the Wi-Fi NIC to perform OTA upgrade.

During the upgrade process, only the current status can be queried, and other operations will not be executed. Notifications will be sent according to the callback function during the upgrade process.

```python
# The Wi-Fi NIC firmware upgrade is realized by passing the firmware download address in the esp8266 OTA upgrade interface. This feature needs to be used when the Wi-Fi NIC is configured as STATION mode
esp8266.ota(url)
```

<a id="click_jump_6.4.2.2"></a>

### General Wi-Fi NIC API Description

This set of interfaces is developed and adapted according to the MicroPython official interface. It is suitable for SDIO external Wi-Fi NICs and built-in Wi-Fi NICs.

The information is only for the general Wi-Fi interface usage instructions. The implementation may vary for different models. For detailed interface information, please refer to the [Wi-Fi Communication](../../../../api_reference/wifilib/README.md) section on the Wiki.

![](../../../media/network-comm/nic/network_wifi_api_process.png)

#### Initialize Wi-Fi NIC Working Mode

Initialize the Wi-Fi NIC and check the working mode of the Wi-Fi NIC.

**Load FCM360W/FC41D**

```python
nic = network.WLAN(network.STA_IF)
```

**Load ASR5803**

```python
nic = network.ASR5803W(network.STA_IF)
```

#### Enable  AP  Mode

Enable AP mode, and you can configure the username and password to establish the corresponding hotspot. Note that the configuration here needs to be consistent with the mode set in the Wi-Fi NIC initialization interface.

```python
# Initialize the Wi-Fi NIC in AP mode
nic = network.WLAN(network.AP_IF)
# Configure the username/password for hotspot establishment
nic.config('ssid'='user', 'key'='password')
# Activate the Wi-Fi NIC
nic.active(True)
```

#### STATION  Mode

Connect to a hotspot and connect to the corresponding Wi-Fi access point based on the username and password of the hotspot.

Method 1:

```python
# Initialize the Wi-Fi NIC in STATION mode
nic = network.WLAN(network.STA_IF)
# The username/password for connecting to the hotspot
nic.connect('user', 'password')
```

Method 2:
```python
# Initialize the Wi-Fi NIC in STATION mode
nic = network.WLAN(network.STA_IF)
# Configure the username/password for connecting to the hotspot
nic.config('ssid'='user', 'key'='password')
# Activate the Wi-Fi NIC
nic.active(True)
```

#### Network  Configuration

Currently, network configuration can only be done through the Python interface. Other methods are still being developed.

#### NIC Configuration

Network configuration method is provided for users to configure the network according to their needs. For example, if the NIC needs to use a static IP address in a network environment, this interface can be used for the configuration. Alternatively, if the default network segment conflicts with other segments in AP mode, it can be modified to another network segment for use.

```python
# Configure the NIC with a static IP address
nic.ifconfig((ip, subnet, gateway, primary_dns, secondary_dns))

# Get the IP information of the NIC
nic.ifconfig()
```

#### Parameter Configuration and Query

The NIC provides a series of methods for configuration and query, making it easy for users to debug or apply the Wi-Fi NIC.

```python
# Configure or query the NIC information, including ssid, bssid, channel, RSSI, security, hidden, etc. Please refer to the corresponding model in the Wiki for specific support
nic.config('param')
nic.config(param=value)

# Activate or deactivate the NIC
nic.active([param])

# Disconnect from the router in STA mode
nic.disconnect()

# Check if the NIC is connected to the router in STA mode, or if there are any clients connected in AP mode
nic.isconnected()
```

#### Status Query

Get the current status and some property values of the NIC through this interface. In normal use, it is necessary to check the status of the NIC.

```python
# Query the status of the NIC. When no parameter is passed, it only queries the current status. When a parameter is passes, it queries the specific parameter information
nic.status([param])
```

<a id="click_jump_6.4.3"></a>

## Application Examples

The following examples demonstrate the use of Wi-Fi functionality on the QuecPython EC600NCN_LE EVB (referred to as the "EVB") connected to ESP8266 via UART2:

[6.4.3.1 Station Mode](./wifi-station-example.md)

[6.4.3.2 AP Mode](./wifi-ap-example.md)

<a id="click_jump_6.4.4"></a>

## Frequently Asked Questions

**1. Why can't ASR5803 modules find any Wi-Fi signals after booting?**

> a. Check if the hardware GPIO settings are correct.
> b. Check if the MAC address is valid (compliant with IEEE regulations for Wi-Fi AP mode MAC settings).
> c. Check if the Wi-Fi signal is too far or of poor quality.
> d. Other factors: wireless interference, security settings, network configuration, etc.

**2. Unable to connect to Wi-Fi**.

> a. Check if the hotspot frequency band is the same as the Wi-Fi NIC, e.g., 2.4G/5G.
> b. Check if the hotspot information contains garbled characters, as Wi-Fi does not support special characters.

**3. Which Wi-Fi chips are supported?**

Currently, the ESP8266/ESP8285/FC41D are supported through UART, the ASR5803 is supported through SDIO, and the FCM360W  is supported through built-in Wi-Fi.

