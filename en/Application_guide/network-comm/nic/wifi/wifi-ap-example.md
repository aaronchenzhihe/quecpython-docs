# AP Mode

The Wi-Fi NIC connects to the hotspot to provide 4G network services to other devices.

Please refer to the [Getting Started](../../../../Getting_started/README.md) section for development environment setup. This section introduces the usage of ESP8266/ESP8285.

## Hardware Preparation

To use the external Wi-Fi  NIC functionality, you need to prepare an external Wi-Fi  NIC, a router, and DuPont wires.

Connect the module to the Wi-Fi NIC with DuPont wires. The wiring between the module and the NIC can be configured according to actual requirements. For specific interface configuration parameters, please refer to the [ESP8266](../../../../api_reference/wifilib/WLAN.ESP8266.md) section on Wiki.

[ESP8266 Hardware Connection Photo](../../../media/network-comm/nic/network_wifi_esp8266_connection.png)  
[ASR5803 Hardware Connection Photo](../../../media/network-comm/nic/network_wifi_asr5803_connection.png)  
[FCM360W Hardware Photo](../../../media/network-comm/nic/network_wifi_fcm360w.png)   
[FC41D Hardware Photo](../../../media/network-comm/nic/network_wifi_fc41d.png)  

## Software Preparation

**NIC Initialization**

NIC initialization including initializing the working environment of the Wi-Fi NIC in AP mode and ensuring the intercommunication of the Wi-Fi NIC. It also involves configuring the hotspot name and password for the Wi-Fi NIC. The following example demonstrates the initialization process for the ESP8266 Wi-Fi NIC.

```python

# The ESP8266 Wi-Fi NIC is included in the WLAN package. Import ESP8266 from the WLAN package, and then import the required UART module from the machine package
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# Initialize the NIC. Select UART2 as the serial port parameter and AP mode as the mode parameter
>>> wifi = ESP8266(UART.UART2, ESP8266.AP)

# Set the Wi-Fi NIC to start in AP mode and configure the name and password of the AP hotspot
>>> wifi.ap('test_8266','12345678')
0

# Get the current NIC status. A return value of 4 indicates that the Wi-Fi NIC is enabled in AP hotspot mode
>>> wifi.status()
4

```

**Router Configuration**

The routing configuration functionality involves methods for network forwarding under different NIC interfaces. Currently, configuration is done only through the default NIC interface, with the default NIC serving as the forwarding target. For non-local data transmitted via the Wi-Fi NIC, NAT forwarding will be performed through the default NIC. 
Enter the following command in the interactive interface to configure the router forwarding and provide 4G network to Wi-Fi terminal devices.  

```python

>>> import dataCall

# Get the dial-up information
>>> Info = dataCall.getInfo(1,0)

# Check if the 4G IP is obtained
>>> Info
(1, 0, [1, 0, '10.62.209.177', '211.138.180.4', '211.138.180.5'])

# Set the default NIC. Set 4G as the default NIC in AP mode
>>> wifi.set_default_NIC(Info[2][2])
0

# Add router information and set the NIC forwarding rules
>>> wifi.router_add('192.168.4.0', '255.255.255.0')
0

```

## Connect to the Wi-Fi Hotspot on a Mobile Phone

Open the WLAN interface on your mobile phone, find the Wi-Fi named `test_8266`, enter the password, and connect to it.

![](../../../media/network-comm/nic/network_wifi_ap_connecting.png)

## Check the Network Connection

You can see that the mobile phone has successfully connected to the Wi-Fi hotspot. At this point, close all other network connections on the phone and only keep this Wi-Fi connection. You can open a browser and access websites normally.
>Note: The ESP8266 is connected to the module via a serial port, and the communication speed is relatively low. It is recommended to use the phone for browsing news, and smooth video communication is not allowed. 

![](../../../media/network-comm/nic/network_wifi_ap_connected.png)