# USB Network Adapter

> This section introduces the compatibility of USB network adapter in QuecPython solution and how to use the USB network adapter.

Contents  
[Introduction to USB Network Adapter](#click_jump_6.6.1)  
&emsp;[Hardware Structure](#click_jump_6.6.1.1)   
&emsp;[Initialization Process](#click_jump_6.6.1.2)  
&emsp;[Data Flow](#click_jump_6.6.1.3)  
&emsp;[Operating Mode](#click_jump_6.6.1.4)  
&emsp;[Module Compatibility](#click_jump_6.6.1.5)   
[USBNET API Description](#click_jump_6.6.2)  
[Application Examples](#click_jump_6.6.3)  
&emsp;[Apply USB Network Adapter on Windows](#click_jump_6.6.3)  
&emsp;[Apply USB Network Adapter on Android](#click_jump_6.6.3)  
&emsp;[Apply USB Network Adapter on Linux](#click_jump_6.6.3)  
[Common Exception Handling](#click_jump_6.6.4)  

<a id="click_jump_6.6.1"></a>

## Introduction to USB Network Adapter

A USB network adapter is an external device used to provide Ethernet connectivity to a computer or other devices. It connects to the computer via a USB interface and transfers data through the USB bus. The USB network adapter is commonly used to add or extend network connectivity to a computer, especially in cases where there is no built-in Ethernet interface or additional network connections are needed.

**How does QuecPython support USB network adapters?**  

The USB protocol defines class code information for device classes for device identification and driver loading. These class codes consist of three bytes: Base Class, SubClass, and Protocol. The commonly supported protocols for USB network adapters are ECM/RNDIS/MBIM (subclasses under the USB protocol).
- ECM (Ethernet Networking Control Model) exchanges Ethernet frame data between the device and the host. The general use case for ECM devices is point-to-point Ethernet adapters for LAN/WLAN.

- RNDIS (Remote Network Driver Interface Specification) is a Microsoft variant implementation of ECM, mainly used to simplify driver development for USB network devices on the Windows platform.

- MBIM (Mobile Broadband Interface Model) is specifically designed for 3G/4G/5G modules. It defines the interface standard between mobile broadband devices and PCs.

QuecPython supports USB network adapters by implementing the ECM/RNDIS/MBIM subclass protocols under the USB protocol for adapter compatibility. The typical use case is to provide network connectivity to other devices through the USB port.

<a id="click_jump_6.6.1.1"></a>

### Hardware Structure

![USBNET Hardware Structure](../../../media/network-comm/nic/network_usbnet_usb.png)

|  Pin   | Description |
| ---- | ------- |
| `VCC` | Provides power to the connected USB device |
| `D-` |  Negative data line |
| `D+` |  Positive data line |
| `GND`   | Provides circuit ground |

<a id="click_jump_6.6.1.2"></a>

### Initialization Process

The initialization and loading process of the network adapter is shown in the following figure:

![](../../../media/network-comm/nic/network_usbnet_usb_flow.png)

<a id="click_jump_6.6.1.3"></a>

### Data Flow

![](../../../media/network-comm/nic/network_usbnet.png)

<a id="click_jump_6.6.1.4"></a>

### Operating Mode

USB network adapters are similar to Ethernet NIC in terms of their operating modes. They can be categorized into LAN/WAN modes (please refer to the [Ethernet NIC](../ethernet/README.md) section) based on their working modes.

QuecPython applies the LAN mode, providing network connectivity to the USB network adapter via 4G.

<a id="click_jump_6.6.1.5"></a>

### Module Compatibility

The table below shows the compatibility of USB network adapters on different modules.

<table>
    <tr>
        <th colspan = '2'>NIC Type</th>
        <th colspan = '1'>Supported Modules</th>
    </tr>
    <tr>
        <td rowspan = "3">USB Network Adapter</td>
        <td>RNDIS</td>
        <td>Supported by all modules except BGxx series modules and Wi-Fi modules</td>
    </tr>
    <tr>
        <td>ECM</td>
        <td>Supported by all modules except BGxx series modules and Wi-Fi modules</td>
    </tr>
        <tr>
        <td>MBIM</td>
        <td>Supported by BGxx series modules</td>
    </tr>
</table>


<a id="click_jump_6.6.2"></a>

## USBNET API Description

USBNET functionality is relatively fixed, so the operations are simple and do not require network-related configurations.

![](../../../media/network-comm/nic/network_usbnet_api_process.png)

### Open USB Network Adapter

The network adapter interface exists by default. After operating on this interface, the USB network adapter can be used to connect to the network.

```python
USBNET.open()
```

### Configure USB Network Adapter Operating Mode

USB network adapters support the ECM/RNDIS protocols. Currently, two types can be selected: Type_ECM and Type_RNDIS. The configuration should be based on the environment you are using.

```python
USBNET.set_worktype(USBNET.Type_RNDIS)
```

### Query USB Network Adapter Operating Mode

This interface determines the operating mode of the adapter and performs corresponding operations in complex situations. It can also be used for debugging to check if the actual mode matches the expected mode.

```python
USBNET.get_worktype()
```

### Query USB Network Adapter Operating Status

Querying the operating status of the USB network adapter is useful for handling various exceptions and for debugging.

```python
USBNET.get_status()
```

### Close USB Network Adapter

The reverse operation of opening the USB network adapter. After this operation, the network adapter will be unavailable, allowing you to control the adapter as needed.

```python
USBNET.close()
```

<a id="click_jump_6.6.3"></a>

## Application Examples

[Apply USB Network Adapter on Windows ](./usbnet-windows-example.md) 

[Apply USB Network Adapter on Android](./usbnet-android-example.md)

[Apply USB Network Adapter on Linux](./usbnet-linux-example.md)

<a id="click_jump_6.6.4"></a>

## Common Exception Handling

**1. Why does the same USB network adapter work on some phones but not on others after connecting?**

> a. Check if the network adapter is set to ECM mode. Not all phones support RNDIS mode.  
> b. Check if the phone needs to enable OTG function. For OPPO and its sub-brands (such as OnePlus, IQOO, etc.), OTG mode needs to be enabled in the system settings.

**2. When using the USB network adapter functionality on Unisoc 8910 modules, the computer cannot connect to the network after the first open in default ECM mode.**

> The USBNET of Unisoc 8910 modules requires NAT support. NAT is not enabled by default at boot and needs to be enabled before the USBNET can be used normally. Please refer to the corresponding [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) section on the Wiki. For specific Unisoc 8910 modules, see  [The Correspondence Between Platforms and Modules](../../../background/hardware-platform.md#Unisoc-Platform).

**3. For Unisoc 8850 modules, the module opens normally after the USB network adapter is set to RNDIS, but returns -1 when reopened.**

> NAT mode needs to be set. Please refer to the corresponding [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) section on the Wiki for specific usage. For specific Unisoc 8850 modules, see  [The Correspondence Between Platforms and Modules](../../../background/hardware-platform.md#Unisoc-Platform).

**4. Why doesn't my phone's status bar have the `<···>` icon, but the network can still be used?**

> This is normal. Due to differences in phone manufacturers, some phone models (such as OnePlus 9RT, Redmi Note 8, etc.) do not have the `<···>` icon.

**5. Why does the computer works fine, but the phone cannot recognize the USB network adapter?**

> a. The phone may not recognize the USB network card due to the module's MAC address. You can try using the MAC address setting interface to update the MAC address.  
> b. Due to differences in phone manufacturers, the USB network card may not be recognized.