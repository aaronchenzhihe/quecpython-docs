# Cellular NIC Initialization Process

The initialization process of a cellular NIC includes the following stages:

![](../../../media/network-comm/nic/cellular/cellular-nic-init.png)

## Hardware Initialization

In this stage, the first step is to detect the SIM card to determine whether it is inserted. If a SIM card is detected, SIM card-related initialization operations will be performed, which corresponds to the "`Initialize SIM Card`" step in the above flowchart. If no SIM card is detected, the subsequent steps will not be executed.

## Network Search

After the SIM card is initialized, the UE (User Equipment) will search for possible wireless signals and cells nearby. This process is called "Network Search" and corresponds to the "`Search Network`" step in the above flowchart.

## Network Registration

During the network search, when the UE finds a suitable cell, it initiates an attach procedure on that cell. This process is called "Network Registration" and corresponds to the "`Network Register`" step in the above flowchart. Only the network registration is successful can the cellular NIC be activated.

The network search and network registration processes of the UE are automatically performed by the protocol stack and cannot be intervened. However, there are two methods to check whether the network registration is successful. 

**Method 1: AT Commands**

You can send the following AT command to query the network registration status of the UE:

```
AT+CREG?
# If the network registration is successful, the return value will be as follows, 
# with the second parameter value being 1 or 5; if it is any other value, 
# it indicates that the network registration failed.
+CREG: 0,1
OK
```

Step 1: Connect the terminal device to the computer's USB port using a USB cable.

Step 2: Open *QPYcom* on the computer, select the AT port, and open the port with the configuration parameters as shown in the figure.

![](../../../media/network-comm/nic/cellular/open-at-port_en.png)

Step 3: Send the AT query command to check the network registration status.

![](../../../media/network-comm/nic/cellular/at-creg.png)

**Method 2: QuecPython API**

You can call `net.getState` to query the network registration status of the UE. Please refer to [Get Network Registration Information](../../../../api_reference/iotlib/net.md#Get-Network-Registration-Information) in the "API Reference" on the QuecPython official website for details.

Step 1: Connect the terminal device to the computer's USB port using a USB cable.

Step 2: Open *QPYcom* on the computer, select the USB NMEA port (REPL command port of QuecPython), and enter the interactive mode.

![](../../../media/network-comm/nic/cellular/open-REPL-port_en.png)

Step 3: In the command line, call *net.getState()* to query the network registration status as follows.

```python
import net
net.getState()
```

![](../../../media/network-comm/nic/cellular/get-network-register.png)

When the parameter indicated by the red box in the figure in the return value of `net.getState()` is `1` or `5`, it means that the network registration is successful. `1` indicates registration on the home network, and `5` indicates registration on a roaming network.

> If the module fails to register on a network, please refer to the following section in the "*Handle Network Exceptions*" chapter for troubleshooting:
>
> * [Module Network Registration Failure](exception-handling.md#Module-Network-Registration-Failure)

## NIC Activation

The cellular NIC will be activated after the UE successfully registers on the network. Only when the cellular NIC is activated successfully can the UE obtain the IP address and other information allocated by the core network and create a virtual NIC for the TCP/IP protocol stack. Only when the virtual NIC is created and activated successfully can network services such as sockets, HTTP, and MQTT be performed. This step corresponds to the "Activate NIC" step in the above flowchart.

You can call `dataCall.getInfo()` of the `dataCall` feature to check whether the NIC is activated successfully. Please refer to [Cellular Network Channel Establishment Information](../../../../api_reference/iotlib/dataCall.md#Cellular-Network-Channel-Establishment-Information) in the "API Reference" on the QuecPython official website for details.

Query method:

```python
dataCall.getInfo(profileID, ipType)
```

The first parameter `profileID` indicates which NIC, and the second parameter `ipType` indicates the IP protocol type. QuecPython automatically activates the first cellular NIC at startup and uses the IPv4 protocol type, so you can set `profileID` to 1 and `ipType` to 0 when querying the NIC activation status.

![](../../../media/network-comm/nic/cellular/get-datacall-infos.png)

When the return value of the `dataCall.getInfo()` method indicates that the NIC activation status is `1` and the IP is not `0.0.0.0`, it means that the NIC is activated successfully.

> If the module fails to activate the NIC, please refer to the following section in the "*Handle Network Exceptions*" chapter for troubleshooting:
>
> * [Handle Network Exceptions](exception-handling.md#Handle-Network-Exceptions)