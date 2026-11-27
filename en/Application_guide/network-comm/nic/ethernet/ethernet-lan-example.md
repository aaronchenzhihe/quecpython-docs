# Module Provides Network Access to External Devices via 4G-to-Ethernet

When the W5500 NIC is configured as a gateway, it enables network forwarding to allow Ethernet devices on the local network to access the Internet via the module's 4G connection. For example, if the W5500 NIC is connected to a computer, the computer needs to be configured with a static IP address in the same subnet as the W5500 NIC. If the gateway address is consistent with the address of the W5500 NIC, the computer will be successfully connected to the Internet through the 4G NIC.

## Hardware Preparation

To use the external Ethernet feature, you need to prepare an external Ethernet NIC, cable, router, and DuPont wires.

Connect the module to the Ethernet NIC with DuPont wires according to your actual needs (Refer to [ethernet - Ethernet Driver](../../../../api_reference/peripherals/ethernet.md) for specific interface configuration parameters), and then connect the Ethernet NIC to the router with a cable.

| Ethernet Chip Model | Ethernet Module Picture | Hardware Connection Picture | Chip Manual | Code Example |
| ---- | ---- | ---- | ---- | ---- |
| W5500    | [W5500 Module Picture](../../../media/network-comm/nic/network_eth_w5500.png) | [W5500 Hardware Connection Picture](../../../media/network-comm/nic/network_eth_w5500_connection.png) | [W5500 Chip Manual](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/w5500) | [Download Code Example](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| DM9051 | [DM9051 Module Picture](../../../media/network-comm/nic/network_eth_dm9051.png) | [DM9051 Hardware Connection Picture](../../../media/network-comm/nic/network_eth_dm9051_connection.png) | [DM9051 Chip Manual](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/dm9051) | [Download Code Example](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| CH395  | [CH395 Module Picture](../../../media/network-comm/nic/network_eth_ch395.png) | [CH395 Hardware Connection Picture](../../../media/network-comm/nic/network_eth_ch395_connection.png) | [CH395 Chip Manual](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/ch395) | [Download Code Example](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |
| YT8512     | [YT8512 Module Picture](../../../media/network-comm/nic/network_eth_yt8512h.png) | [YT8512 Hardware Connection Picture](../../../media/network-comm/nic/network_eth_yt8512h_connection.png) | [YT8512 Chip Manual](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/yt8512) | [Download Code Example](https://github.com/QuecPython/examples/tree/main/network-comm/nic/ethernet) |

> 1. The hardware connection pictures are just for reference and you need to configure the connection according to your actual environment.  
> 2. SZ18201/J1101 has the same connection and usage method as YT8512, both using the RMII interface, so they are not listed here.

## Software Preparation

**Ethernet NIC Initialization**

Enter the following command on the REPL page of *QPYcom* to initialize the Ethernet device and configure the Ethernet NIC working mode to LAN mode (LAN).  
```python
# Data call information is required in LAN mode, so an additional dataCall package should be imported.
>>> import ethernet
>>> import dataCall

# Initialize the Ethernet NIC and configure the parameters. The CS pin, RST pin, and INT pin entered here are GPIO pin numbers (Please refer to "Pin-Control I/O Pins" for the mapping between GPIO pin numbers and physical pins).
>>> eth = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','192.168.1.1','','',-1,38,36,37,1)

# Enable the Ethernet NIC.
>>> eth.set_up()
0

```

**Routing Configuration**

The routing configuration feature refers to network forwarding between different network NICs. Currently, configuration is only applicable to the default NIC and the default NIC is used as the forwarding target. For the non-local data transmitted through the LAN port, it will be NAT forwarded through the default NIC.

Enter the following command on the REPL page of *QPYcom* to configure the routing to provide a 4G network to the LAN device.  
```python
# Data call information is required in LAN mode, so an additional dataCall package should be imported.
>>> import dataCall

# Get the current IP information of the 4G NIC.
>>> info=dataCall.getInfo(1, 0)

# Check if the IP information of the 4G NIC is obtained.
>>> print(info)
(1, 0, [1, 0, '10.62.209.177', '211.138.180.4', '211.138.180.5'])

# Set the 4G NIC as the default NIC in LAN mode.
>>> eth.set_default_network_card(info[2][2])
0

# Query the IP information of the Ethernet NIC
>>> eth.ipconfig()
[('12-34-56-78-9A-BC', 'W5500'), (4, '192.168.1.100', '255.255.255.0 ', '192.168.1.1', '8.8.8.8', '114.114.114.114')]

```

## Opposite-End Configuration

Taking a PC as an example, a PC cannot communicate on the network before an IP address is configured. Only when the PC's static IP address is configured in the same network segment as the module's static IP address can the PC communicate on the network. 

Right-click "**`Ethernet`**", select "**Properties**", click "**Internet Protocol Version 4 (TCP/IPv4)**", de-select "Obtain an IP address automatically" and "Obtain DNS server address automatically", change the settings to the static IP address shown in the following figure and click "**OK**".

> In this case, the PC can also obtain a dynamic IP address. Some modules do not have a built-in DHCP server and need to be configured with a static IP for network configuration.

![](../../../media/network-comm/nic/network_eth_demo_3.png)

## Access Internet via Module Ethernet 

After the PC has connected to the network, you can access the Internet through a computer browser. For example, you can enter https://developer.quectel.com/en/ in the browser to access the official QuecPython website normally.

![](../../../media/network-comm/nic/network_eth_gateway_web.png)