# Cellular Network API

This document introduces APIs that are required in various user scenarios and provides usage instructions and precautions for these APIs.

## Get/Set Module Work Mode

Device work modes refer to the functionality modes of mobile terminals, also known as CFUN states. QuecPython supports setting and getting the work mode of the module. Please refer to [Work Mode Configuration](../../../../api_reference/iotlib/net.md#Work-Mode-Configuration) in the wiki on the QuecPython official website for details.

### Get the Module Work Mode

```python
net.getModemFun()
```

Referring to the [CFUN](common-concepts.md#CFUN) section in the introduction to cellular network basics, the module has three work modes. Only when the module is in mode 1 (full functionality), can it communicate with the cellular network.

### Set the Module Work Mode

```python
net.setModemFun(fun [, rst])
```

The first parameter **fun** indicates the mode to be set. The second parameter **rst** is an optional parameter that indicates whether to reboot the module. Generally, the second parameter is not required, which means that the module will not be rebooted by default when the work mode is set.

### Application Scenarios

You can call the above functions to get and set the module's work mode in the following scenarios:

* When the module fails to automatically activate the NIC at startup, you need to get the current work mode of the module and check if the response is `1`. If the work mode is not `1`, you need to set the module work mode to `1` first.

* When you want to disable the RF function of the module when no network service is needed to reduce power consumption, you can set the module work mode to 0 or 4.

* When the current network communication quality of the module is poor, you can set the module work mode to 4 (airplane mode), and then set it back to 1 (full functionality mode). In this way, the module will re-search for networks and register.

* If you remove or switch the SIM card without powering down the module, and you do not want to reboot the module, you can set the module work mode to 0 first, and then set it to 1. In this way, the module will re-initialize the SIM card, then re-search for networks and register.



## Get/Set Network Interface Card (NIC) Parameters

NIC parameters include IP protocol type, APN, username, password, and APN authentication method. QuecPython supports setting and getting the parameters of the cellular NIC. Please refer to [APN Configuration and Retrieval](../../../../api_reference/iotlib/dataCall.md#APN-Configuration-and-Retrieval) in the wiki on the QuecPython official website for details.

### Get NIC Parameters

```python
dataCall.getPDPContext(profileID)
```

### Set Network Card Parameters

```python
dataCall.setPDPContext(profileID, ipType, apn, username, password, authType)
```

Once calling this API to set NIC parameters, the configuration takes effect after the module is rebooted. In addition, the parameters set by this API are saved automatically.

### Application Scenarios

You can call the above functions to get and set the parameters of the NIC in the following scenarios:

* Some special SIM cards require APN configuration before successful network registration. In this case, you need to confirm with the operator what APN, username, and password should be used for this card, and then call *dataCall.setPDPContext()* to configure the APN information for the cellular NIC, and finally reboot the module.

* When you need to activate multiple cellular NICs at the same time and use different NICs to connect to different networks, for example, the first NIC is used to access the public network, and the second NIC is used to access a private network. You need to call `dataCall.setPDPContext` to configure the corresponding APN information for the first and second NICs.

For how to configure APN, please refer to the following sections in the *Scenario Instructions* chapter, which provide detailed APN configuration examples:

* [Automatically Activate One NIC at Startup and Configure APN](application.md#Automatically-Activate-One-NIC-at-Startup-and-Configure-APN)

* [Automatically Activate Multiple NICs at Startup and Configure APN](application.md#Automatically-Activate-Multiple-NICs-at-Startup-and-Configure-APN)

* [Manually Activate One NIC](application.md#Manually-Activate-One-NIC)

* [Manually Activate Multiple NICs](application.md#Manually-Activate-Multiple-NICs)



## Activate/Deactivate NICs

QuecPython supports manual activation and deactivation of cellular NICs. Please refer to [Activation and Deactivation](../../../../api_reference/iotlib/dataCall.md#Activation-and-Deactivation) in the wiki on the QuecPython official website for details.

### Activate NIC

```python
dataCall.activate(profileID)
```

### Deactivate NIC

```python
dataCall.deactivate(profileID)
```

### Application Scenarios

By default, QuecPython modules automatically activate cellular NICs at startup, so in most cases, you do not need to manually activate or deactivate cellular NICs. However, in some special scenarios or for specific requirements, it is necessary to manually activate or deactivate cellular NICs by using the above functions. 

If you disable the automatic activation of cellular NICs at startup and set a program to activate and deactivate NICs based on needs at a certain time, you need to call `dataCall.activate` and `dataCall.deactivate` in the program accordingly. For example, some electricity meters do not need to establish a network connection most of the time. Only when the electricity meter data needs to be reported, the NICs will be activated. When the data reporting is completed, the NICs will be deactivated and the network connection will be disconnected. This not only saves SIM card traffic, but also reduces the power consumption of the device.

For how to manually activate NICs, please refer to the following sections in the *Scenario Instructions* chapter, which provide detailed APN configuration examples:

* [Manually Activate One NIC](application.md#Manually-Activate-One-NIC)
* [Manually Activate Multiple NICs](application.md#Manually-Activate-Multiple-NICs)



## Configure DNS

When the module activates the cellular NIC, if the activation is successful, the core network will automatically assign DNS server addresses to the module. In other words, under normal circumstances, you do not need to manually configure DNS server addresses. However, sometimes the DNS server addresses assigned by the core network may be unavailable, and in this case, you have to manually configure DNS server addresses. QuecPython supports manual DNS configuration. Please refer to [DNS Configuration](../../../../api_reference/iotlib/dataCall.md#DNS-Configuration) in the wiki on the QuecPython official website for details.

### Configure DNS

```python
dataCall.setDNSServer(profileID, simID, priDNS, secDNS)
```

### Application Scenarios

Sometimes the DNS server addresses assigned by the core network may be unavailable, and in this case, you have to manually configure DNS server addresses.



## Get NIC Status Information

QuecPython supports getting NIC status information, such as NIC activation status, IP address, and DNS server address. Please refer to [Cellular Network Channel Establishment Information](../../../../api_reference/iotlib/dataCall.md#Cellular-Network-Channel-Establishment-Information) in the wiki on the QuecPython official website for details.

### Get NIC Status Information

```python
dataCall.getInfo(profileID, ipType)
```

### Application Scenarios

It is necessary for you to get NIC status information. Regardless of the application scenario, as long as you need to perform network-related operations, you must first get the activation status of the NIC to confirm that the cellular NIC has been activated successfully. Specifically, you can call the above function to get the NIC status information in the following scenarios:

* Confirm whether the cellular NIC has been activated successfully by checking the `state` value in the returned tuple of `dataCall.getInfo`. 1 indicates successful activation.

* When you use the socket feature of QuecPython, you need to know the current IP address of the module.

* After activating multiple cellular NICs, it is necessary to establish multiple sockets. Each socket uses a different NIC, so it is necessary to obtain the IP address of each NIC and bind the IP to the corresponding socket.

* Get the current DNS address used by the cellular NIC.



## Automatically Activate NIC at Startup

By default, QuecPython modules automatically activate the first cellular NIC at startup, and the IP protocol type used is IPv4 with APN address, username and password empty. You can configure the automatic activation of any one or multiple cellular NICs at startup by calling *dataCall.setAutoActivate()*. Please refer to [Establish the Cellular Network Channel Automatically at Startup](../../../../api_reference/iotlib/dataCall.md#Establish-the-Cellular-Network-Channel-Automatically-at-Startup) in the wiki on the QuecPython official website for details.

### Set Whether to Automatically Activate NIC at Startup

```python
dataCall.setAutoActivate(profileID, enable)
```

The parameters set by this method are non-volatile and the configuration takes effect after the module is rebooted.

### Application Scenarios

You can call the above function to set whether to automatically activate an NIC at startup in the following scenarios:

* When you want to disable the automatic activation of the cellular NIC at startup, call `dataCall.setAutoActivate(profileID, 0)`.

* When you want the module to automatically activate multiple cellular NICs at startup, call `dataCall.setAutoActivate(profileID, 1)` to set the NICs that need to be activated. For example, if you want the module to automatically activate the first and second cellular NICs at startup, the configuration would be as follows.

  ```python
  dataCall.setAutoActivate(1, 1) # Enable automatic activation for the first cellular NIC
  dataCall.setAutoActivate(2, 1) # Enable automatic activation for the second cellular NIC
  ```

  

## Set Automatic Reconnection of NIC

By default, QuecPython modules will enable the automatic reconnection for the first cellular NIC at startup. You can configure the automatic reconnection for any one or multiple cellular NICs by calling *dataCall.setAutoConnect()*. Please refer to [Automatic Reconnection](../../../../api_reference/iotlib/dataCall.md#Automatic-Reconnection) in the wiki on the QuecPython official website for details.

It is recommended not to disable the automatic reconnection of the NIC unless there are special requirements.

### Set Whether to Automatically Reconnect to Network

```python
dataCall.setAutoConnect(profileID, enable)
```

The parameters set by this method are non-volatile and the configuration takes effect after the module is rebooted.

### Application Scenarios

You can call the above function to set whether the NIC automatically reconnects to the network in the following scenarios:

* When you do not need the NIC to automatically reconnect to the network for some special requirements, call `dataCall.setAutoConnect(profileID, 0)` to disable the feature.

* After calling `dataCall.setAutoActivate(profileID, 1)` to set the automatic activation of a certain or multiple cellular NICs at startup, you need to determine whether this or these NICs need to automatically reconnect to the network. If the automatic reconnection feature is required, call `dataCall.setAutoConnect(profileID, 1)`, otherwise, call `dataCall.setAutoConnect(profileID, 0)`.



## Network Event Listening

QuecPython provides APIs for users to listen to network state change events. The specific approach is to allow users to register callback functions. When the connection status to the network changes, the system will notify the current network connection status through the user's registered callback function. Please refer to [Register Callback Function](../../../../api_reference/iotlib/dataCall.md#Register-Callback-Function) in the wiki on the QuecPython official website for details.

### Register Callback Function

```
dataCall.setCallback(fun)
```

### Application Scenarios

In practical use, due to various reasons (such as network exception, environmental interference, poor signal, and SIM card arrears), the module may disconnect from the network and cannot continue network-related operations. Therefore, you need to monitor the module network status to ensure that when the network is disconnected and reconnected, the corresponding processing can be performed timely.

For how to use network event listening to handle network exceptions, please refer to the following section in the *Handle Network Exceptions* chapter:

* [Example of Handling Network Exception Events](exception-handling.md#Example-of-Handling-Network-Exception-Events)



## Get Signal Strength

QuecPython provides APIs to obtain parameters such as signal strength and signal quality. These parameters can help you determine the signal strength and signal quality of the current device's environment. The commonly used parameters include CSQ, RSSI, SINR, RSRP, and RSRQ. It is recommended to read the [Signal Quality](common-concepts.md#Signal-Quality) section in the *Basic Concepts of Cellular Network* chapter together. Please refer to [Get Signal Strength](../../../../api_reference/iotlib/net.md#Get-Signal-Strength) and [Get Detailed Signal Strength](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength) in the wiki on the QuecPython official website for details.

### Query Signal Strength and Quality

Call the following function to query CSQ value.

```
net.csqQueryPoll()
```

Call the following function to query RSSI, SINR, and RSRQ values.

```
net.getSignal([sinrEnable])
```

Note:

CSQ represents signal strength and is a parameter used to indicate RSSI strength. In other words, CSQ is essentially the same as RSSI. Generally, you can determine signal strength by CSQ value. Range: 0 – 31, with higher values indicating better signal quality.

SINR represents the ratio of the desired signal power to the sum of the power of all other interfering signals (noise and interference), reflecting the link quality of the current channel. Range: -10 dBm – 40 dBm, with higher values indicating better signal quality.

RSRP represents the path loss intensity of the current channel and is used for measuring cell signal coverage and cell selection/reselection. We can judge the signal coverage of the cell based on this parameter value. Range: -140 dBm – -44 dBm, with higher values indicating better signal quality.

RSRQ represents the signal-to-noise ratio and interference level of the current channel quality. RSRQ changes with network load and interference. The larger the network load and interference, the smaller the RSRQ value. Range: -20 dB – -3 dB, with higher values indicating better signal quality.

### Application Scenarios

You may need to query signal strength and quality in the following scenarios:

* When the module fails to register on the network, you need to call `net.csqQueryPoll()` to query the current CSQ value. If the CSQ value is less than 6, the module will have difficulty communicating on the network.

* When the module frequently disconnects during network communication, you need to call the above two functions to query CSQ, SINR, RSRP, and RSRQ values. It is recommended to continuously query these parameters for a period of time to obtain more data samples. First, check the signal strength with CSQ value. If the signal strength is OK, determine the signal quality with SINR and the interference in communication with RSRQ. You can also check the RSRP value to comprehensively evaluate the signal coverage of the current cell.



## Get Cell Information

QuecPython provides APIs to get information about the current serving cell and neighboring cells. Cell information includes cell type (serving cell or neighboring cell), CID, MCC/MNC, ARFCN and physical cell identity. Please refer to [Get Cell Information](../../../../api_reference/iotlib/net.md#Get-Cell-Information) in the wiki on the QuecPython official website for details.

### Get Cell Information

```python
net.getCellInfo()
```

When calling this function to get cell information, the module will perform a real-time scan, which generally takes a few seconds. In addition, the cells that the module can scan depend on the supported bands of the module. If the module does not support the band of a certain cell, the cell will not be scanned.

### Application Scenarios

You may need to get the cell information in the following scenarios:

* For base station positioning, information about surrounding cells is required. If using QuecPython's base station positioning feature, you do not need to get cell information because this operation is automatically performed by the base station positioning code. If you implementing base station positioning through a third-party platform, you need to call `net.getCellInfo()` to get cell information and upload them to the relevant server.

* If you want to know which cells are included in the current environment, call `net.getCellInfo()`.



## Get/Set Cellular Network Mode

QuecPython supports various module models, some of which only support LTE, some support GSM and LTE, and some support GSM, WCDMA, and LTE. You can get and set the network modes of the module by calling corresponding APIs. Please refer to [Network Mode and Roaming Configuration](../../../../api_reference/iotlib/net.md#Network-Mode-and-Roaming-Configuration) in the wiki on the QuecPython official website for details.

### Get Network Mode

```python
net.getConfig()
```

### Set Network Mode

```python
net.setConfig(mode [, roaming])
```

The first parameter of this function is the network mode you want to set. The second parameter is whether to enable roaming, which is an optional parameter that generally does not need to be set.

From the return value of `net.getConfig()` in [Network Mode and Roaming Configuration](../../../../api_reference/iotlib/net.md#Network-Mode-and-Roaming-Configuration), you can see that there are many configurable network modes, either single mode or a combination of multiple modes. The following points need to be clarified:

* Although many modes are listed in the return value of `net.getConfig()`, not all modules support these modes. For the specific network modes supported by each module model, please refer to the corresponding module specification.

* When configuring a combination of multiple network modes, such as GSM and LTE, the module will have a priority when searching for networks. The specific priority order depends on the network mode selected from the network mode table in the return value of `net.getConfig()`. For example, when setting a combination of GSM and LTE, if mode 8 is selected from the network mode table, the module will search for LTE cells first. If no LTE cells are found or if the attachment to an LTE cell fails, the module will then search for GSM cells to attach to.
* Generally, if the module supports multiple network modes, the default mode is a combination of the supported network modes.

### Application Scenarios

You may need to set the network modes of the module in the following scenarios:

- If the module supports multiple network modes, but when you call `net.getConfig()` to get the current network mode and find that the module is currently configured to only support a certain network mode, you can call `net.setConfig()` to reconfigure the module to support multiple network modes.

* If the module supports multiple network modes, the default mode is a combination of the supported network modes. However, if the SIM card used only supports one of the network modes, you can call `net.setConfig()` to set the network mode of the module to the one supported by the SIM card. This will speed up the module's network registration process by searching for cells directly on the network mode supported by the SIM card.

* If the module supports multiple network modes, the default mode is a combination of the supported network modes. However, if the surrounding cells of the module only support one network mode, such as GSM, you can call `net.setConfig()` to set the network mode of the module to the one supported by the surrounding cells.



## Cellular Network Technology

As mentioned earlier, some modules support multiple network modes. When the default network mode of a module is set to a combination of multiple network modes, it is important to determine which network the module will register on. By getting the current network technology of the module, you can determine which network mode is currently used. QuecPython provides relevant APIs to get the current network technology of the module. Please refer to [Get Network Configuration Mode](../../../../api_reference/iotlib/net.md#Get-Network-Configuration-Mode) in the wiki on the QuecPython official website for details.

### Get Network Technology

```python
net.getNetMode()
```

### Application Scenarios

The main application scenario is to determine the specific network that the module is currently connected to when the module supports multiple network modes and the default mode is set to a combination of multiple network modes.



## Get Operator Information

Operator information refers to the information of the operator to which the currently connected cell belongs, including the operator name, MCC, and MNC. QuecPython provides an API to get the operator information. Please refer to [Get Operator Information](../../../../api_reference/iotlib/net.md#Get-Operator-Information) in the wiki on the QuecPython official website for details.

### Get Operator Information

```python
net.operatorName()
```

Please note that the operator to which the currently connected cell belongs may not necessarily be the operator to which the SIM card used by the module belongs. For example, if the module is connected to a cell through roaming, the operator information will be the operator to which the cell belongs, but the SIM card may belong to another operator.

### Application Scenarios

As mentioned above, this function is used to get the operator information of the cell to which the module is currently connected, so if you want to get the information, call `net.operatorName()`.



## Get Device Network Registration Status

The cellular network registration status of a device is a very important parameter because a successful network registration is required for the activation of a cellular NIC. QuecPython provides an API to query the network registration status of the device. Please refer to [Get Network Registration Information](../../../../api_reference/iotlib/net.md#Get-Network-Registration-Information) in the wiki on the QuecPython official website for details.

### Get Device Registration Status

```python 
net.getState()
```

### Application Scenarios

Under normal circumstances, you do not need to get the registration status of the device as long as the module's cellular NIC can be activated successfully. The scenario you need to get the registration status of the device is as follows:

* If the activation of the module's cellular NIC fails, you have to check the SIM card status and network registration status. In this case, call `net.getState()` to get the registration status and confirm whether the network registration was successful.



## Set and Get Band

QuecPython modules generally support multiple bands, and by default, all supported bands are enabled. QuecPython provides APIs to get and set the bands currently supported by the module. Please refer to [Set and Get Band](../../../../api_reference/iotlib/net.md#Set-and-Get-Band) in the wiki on the QuecPython official website for details.

> Currently, only the BG95 series module and EG912N-ENAA module support the following APIs.

### Get Band

```python
net.getBand(netRat)
```

### Set Band

```python
net.setBand(netRat, gsmBand, bandTuple)
```

### Application Scenarios

You may need to get and set bands in the following scenarios:

* If you want to know which bands the module currently supports, you can call `net.getBand()` or refer to the module specification. Please note that module specifications describe the bands theoretically supported by the module, while the `net.getBand()` returns the bands actually supported.

* Some special SIM cards only support specific bands. You can call `net.setBand()` to lock the module band to the specific band supported by the SIM card.

* In the environment where the module is located, there are multiple bands, and the module supports all of them. However, some bands have poor signal quality while others have good signal quality. You can call `net.setBand()`  to lock the module band to the bands with good signal quality, so that the module can directly access these bands with good signal quality.

* If you want to lock the module band to a specific band for functional verification or testing, call `net.setBand()`.