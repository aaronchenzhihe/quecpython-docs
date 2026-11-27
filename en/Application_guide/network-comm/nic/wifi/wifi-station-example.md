# Station Mode

The Wi-Fi NIC connects to the hotspot to provide network access to the module.

Please refer to the [Getting Started](../../../../Getting_started/README.md) section for development environment setup. This section introduces the usage of ESP8266/ESP8285.

## Hardware Preparation

To use the external Wi-Fi NIC functionality, you need to prepare an external Wi-Fi NIC, a router, and DuPont wires.

Connect the module to the Wi-Fi NIC with DuPont wires. The wiring between the module and the NIC can be configured according to actual requirements. For specific interface configuration parameters, please refer to the [ESP8266](../../../../api_reference/wifilib/WLAN.ESP8266.md) section on Wiki.

[ESP8266 Hardware Connection Photo](../../../media/network-comm/nic/network_wifi_esp8266_connection.png)  
[ASR5803 Hardware Connection Photo](../../../media/network-comm/nic/network_wifi_asr5803_connection.png)  
[FCM360W Hardware Photo](../../../media/network-comm/nic/network_wifi_fcm360w.png)   
[FC41D Hardware Photo](../../../media/network-comm/nic/network_wifi_fc41d.png)   

## Software Preparation

**NIC Initialization**

The NIC initialization does not configure the NIC, but initializes the working environment of the Wi-Fi NIC to ensure the intercommunication of the Wi-Fi NIC. The following example demonstrates the initialization process for the ESP8266/ESP8285 Wi-Fi NIC.

```python

# The ESP8266 Wi-Fi NIC is included in the WLAN package. Import ESP8266 from the WLAN package, and then import the required UART module from the machine package
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# Initialize the NIC. Select UART2 as the serial port parameter and STA mode as the mode parameter
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

```

**Wi-Fi NIC Configuration**

ESP8266/ESP8285 provides multiple ways to configure the network, including command line configuration, WEB configuration, and SMARTCONFIG configuration.

Command Line Configuration

```python
# The ESP8266 Wi-Fi NIC is included in the WLAN package. Import ESP8266 from the WLAN package, and then import the required UART package from the machine package
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# Initialize the NIC. Select UART2 as the serial port parameter and STA mode as the mode parameter
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# Configure the name and password of the Wi-Fi to be connected when the Wi-Fi NIC is enabled in station mode
>>> wifi.station('wifiname','wifipassword')
0

```

WEB Configuration

```python
# The ESP8266 Wi-Fi NIC is included in the WLAN package. Import ESP8266 from the WLAN package, and then import the required UART package from the machine package
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# Initialize the NIC. Select UART2 as the serial port parameter, and STA mode as the mode parameter
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# Configure the name and password of the Wi-Fi to be connected when the Wi-Fi NIC is enabled in station mode
>>> wifi.web_config('admin','admin123456')
0

```

SMARTCONFIG Configuration

```python
# The ESP8266 Wi-Fi NIC is included in the WLAN package. Import ESP8266 from the WLAN package, and then import the required UART package from the machine package
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# Initialize the NIC. Select UART2 as the serial port parameter, and STA mode as the mode parameter
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# Configure the name and password of the Wi-Fi to be connected when the Wi-Fi NIC is enabled in station mode
>>> wifi.smartconfig('wifiname','wifipassword')
0

```

**Network Forwarding Configuration**

ESP8266/ESP8285 is a standalone device that includes independent network configuration methods. After the ESP8266/ESP8285 network configuration is completed, QuecPython needs to configure the network on the module to enable normal network forwarding.

```python
# The ESP8266 Wi-Fi NIC is included in the WLAN package. Import ESP8266 from the WLAN package, and then import the required UART package from the machine package
>>> from usr.WLAN import ESP8266
>>> from machine import UART

# Initialize the NIC. Select UART2 as serial port parameter, and STA mode as the mode parameter
>>> wifi = ESP8266(UART.UART2, ESP8266.STA)

# Configure the name and password of the Wi-Fi to be connected when the Wi-Fi NIC is enabled in station mode
>>> wifi.station('wifiname','wifipassword')
0

# Get the current status of the NIC. The return value 1 indicates the Wi-Fi NIC is connected to Wi-Fi, and 2 indicates it is not connected to Wi-Fi
>>> wifi.status()
1

# Configure the DNS server for the Wi-Fi NIC
>>> wifi.set_dns('8.8.8.8','114.114.114.114')
0

# Check the IP information of the Wi-Fi NIC again. Note that 172.16.1.2 is a virtual IP, not the IP of a specific Wi-Fi terminal
>>> wifi.ipconfig()
('172.16.1.2', '255.255.255.0', '172.16.1.1', 1500, '8.8.8.8', '114.114.114.114')

# Set the Wi-Fi NIC as the default NIC for network communication
>>> wifi.set_default_NIC('172.16.1.2')
0

# At this point, you can access the external network through the Wi-Fi NIC

```

## TCP ([socket](../../../../api_reference/stdlib/usocket.md)) Application Example

The following examples demonstrate two methods of TCP communication over Wi-Fi network.

**TCP Client Communication with Binding to a Wi-Fi NIC**

In this example, TCP communication is performed with a Wi-Fi NIC. The `bind` interface of the socket module is used to specify sending through the Wi-Fi interface.

>In the example below, the `bind` interface binds to 172.16.1.2, which is the IP address of the Wi-Fi NIC. The IP address of the NIC can be queried through the `ipconfig` interface of the NIC object. For details, refer to the [Wi-Fi](../../../../api_reference/wifilib/README.md) chapter in the Wiki.

```python
# Import the usocket module
import usocket

if __name__ == '__main__':
    # Create a socket instance
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
    sock.settimeout(5)
    sock.bind(("172.16.1.2", 0))
    # Parse the domain name
    sockaddr=usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
    print('start connect')
    # Establish a connection
    sock.connect(sockaddr)
    # Send a message to the server
    ret=sock.send('GET /NEWS HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
    print('send %d bytes' % ret)
    # Receive messages from the server
    data=sock.recv(1024)
    print('recv %s bytes:' % len(data))
    print(data.decode())

    # Close the connection
    sock.close()
```

**TCP Client Communication without Binding to a Wi-Fi NIC**

In this example, TCP communication is conducted over a Wi-Fi network without using the `bind` interface within the socket module. The communication is achieved by configuring the default NIC. It is necessary to ensure that the Wi-Fi NIC is configured as the default NIC.

```python
# Import the usocket module
import usocket

if __name__ == '__main__':
    # Create a socket instance
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
    sock.settimeout(5)
    # Parse the domain name
    sockaddr=usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
    print('start connect')
    # Establish a connection
    sock.connect(sockaddr)
    # Send a message to the server
    ret=sock.send('GET /NEWS HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
    print('send %d bytes' % ret)
    # Receive messages from the server
    data=sock.recv(1024)
    print('recv %s bytes:' % len(data))
    print(data.decode())

    # Close the connection
    sock.close()
```

## Application of MQTT/HTTP and Other Application Protocols

Note that the MQTT/HTTP application protocol is used over a Wi-Fi NIC. Because the built-in MQTT/HTTP communication module does not apply the binding NIC operation interface, you need to configure an Ethernet NIC as the default NIC. For specific MQTT/HTTP usage, refer to the [umqtt](../../../../api_reference/networklib/umqtt.md) and [reqeust](../../../../api_reference/networklib/request.md) sections in the Application Layer Protocol Wiki.