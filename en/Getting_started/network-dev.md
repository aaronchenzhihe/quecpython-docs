# Cellular infrastructure development

> This chapter mainly introduces the dial-up internet process of QuecPython module, how to register for network dial-up after receiving the module, and how to handle network exceptions

##  QuecPython dial-up internet process

### **Equipment hardware connection**

Firstly, follow the hardware preparation steps above to install the antenna that comes with the development board (some development board models with onboard antennas do not require installation), and insert the SIM card into the SIM card slot on the development board. Then connect the module to the power supply and turn it on. After turning on, the module will automatically activate the network card.

### **Get dialing information**

The QuecPython module automatically activates the first cellular wireless network card upon startup. Only when the cellular wireless network card is successfully activated can network services such as socket, HTTP, MQTT, etc. be performed.

So before taking any business action, it is necessary to obtain dialing information, including dialing status, IP address, DNS server address, etc., in order to determine whether the network card has been successfully activated

```python
import dataCall
dataCall.getInfo(profileID, ipType)
# profileID - PDP context ID, integer value, range 1-3
# ipType - IP protocol type, integer value, 0: IPV4  1: IPV6  2: IPV4&IPV6
```

The return value format is

`(profileID, ipType, [state, reconnect, addr, priDNS, secDNS])`

The operation of querying the status information of the network card is mandatory for users. Regardless of the application scenario, as long as users need to perform network business operations, they must first check the activation status of the network card and confirm that the cellular wireless network card has been successfully activated. Specifically, as long as users have the following requirements, they need to use `dataCall.getInfo` to query:

- Confirm whether the activation of the cellular wireless network card is successful by using the state returned by the `dataCall.getInfo` method, where a value of 1 indicates successful activation.
- When users use QuecPython's socket function, they need to know the current IP address of the module.
- After activating multiple cellular wireless network cards, multiple sockets need to be established, and different sockets use different network cards. At this time, it is necessary to obtain the IP address information of each network card and bind it with the corresponding socket.
- Obtain the current DNS address used by the cellular wireless network card.

If the dial-up information shows that the network card activation has failed, manual activation of the network card is required

### **Manually activate the network card**

After the above query processing, if the network card activation fails, the network card needs to be manually activated. Use the following API to manually activate the network card

```python
dataCall.activate(profileID)
# profileID - PDP context ID, integer value, range 1~3.
```

The QuecPython module automatically activates the cellular wireless network card when it is turned on by default. Under normal circumstances, there is no need to manually set the activation or deactivation operation. However, in some special scenarios or user specific needs, the above methods need to be used to manually activate or deactivate the cellular wireless network card, such as:

The user has disabled the automatic activation of the cellular wireless network card upon startup, and the user application will activate and deactivate the network card at a certain time as needed. In this case, users need to call the `dataCall.activate` and `dataCall.deactivate` methods as needed in their application.

### **Get SIM card status**

If the manual activation of the module cellular wireless network card fails, it is necessary to sequentially check the SIM card and network registration status. To query the SIM status, the following API needs to be used to obtain it

```python
import sim
sim.getStatus()
```

If the return value of `sim.getStatus()` is 0, it means that the device did not detect the SIM card, and it is necessary to determine whether the SIM card is inserted. If the user has already inserted the SIM card and restarted the device, but the query status value is still 0, the possible reasons are as follows:

- The SIM card is not properly inserted, such as being inserted incorrectly or not securely fastened. You can reinsert the SIM card and restart the device, then check if the SIM card status is 1 again after booting up.
- The SIM card itself is damaged. You can insert a functional SIM card, restart the device, and then check if the SIM card status is 1 again.
- The SIM card slot is damaged. If this issue is confirmed, a new card slot needs to be replaced.
- There is a hardware circuit issue with the SIM card, such as poor contact, which prevents the device from recognizing the SIM card properly. Hardware engineers need to check the circuit to confirm the issue.

Users can investigate the above situations in sequence to confirm the problem.

If the return value is 1, it means that the SIM card has been detected and the network registration status needs to be checked

Reference for SIM card abnormalities in other states [SIM card abnormality handling](../Application_guide/network-comm/nic/cellular/exception-handling.md#SIM-Card-Exception)

### **Get device network registration status**

The cellular wireless network registration status of the device is a very important parameter. The prerequisite for successfully activating the cellular wireless network card is that the device must first successfully subscribe to the network. QuecPython provides relevant APIs for querying the device's registration status. This API allows you to query the network status of the current device

```python
import net
net.getState()
```

The API return value is

`([voice_state, voice_lac, voice_cid, voice_rat, voice_reject_cause, voice_psc], [data_state, data_lac, data_cid, data_rat, data_reject_cause, data_psc])`

Among them, data_state represents the network registration status, which can have various situations. When the return value is 1 or 5, it can be considered as a successful network registration. For specific return values, please refer to [Get Network Registration Information](../api_reference/iotlib/net.md#Get-Network-Registration-Information), When the network registration status is not 1 or 5, it can be considered as a network abnormality. There may be multiple reasons for network abnormalities, such as SIM card arrears, SIM card only supporting specific network standards or frequency bands, poor RF performance, and no APN configuration. For detailed handling steps of the above abnormal situations, please refer to [Module Network Registration Failure](../Application_guide/network-comm/nic/cellular/exception-handling.md#Module-Network-Registration-Failure)

Under normal circumstances, as long as the module network card can be successfully activated, users do not need to check the network registration status. The main scenarios that require querying the device's network registration status are as follows:

- The activation of the module cellular wireless network card failed, and it is necessary to investigate the SIM card and network registration status in sequence. At this point, you can use `net.getState` to query the network registration status and confirm whether the device has successfully registered. If the registration network status is abnormal, you need to continue checking according to the above steps.

### **Network status detection**

QuecPython provides an API interface for detecting network status, which can be used to check if the network status is ready.

```none
checkNet.waitNetworkReady(timeout)
```

Wait for the module network to be ready. This method will sequentially detect the SIM card status, module network registration status, and PDP Context activation status; Within the set timeout period, if successful activation of PDP Context is detected, it will immediately return; otherwise, it will only exit after timeout. You can directly use this API interface to detect the network instead of the three steps of detecting SIM card status, module network registration status, and PDP Context activation status mentioned earlier. If any abnormalities are detected in the corresponding steps, the abnormal parts can be detected and processed

**Parameter description:**

- `timeout` - Timeout, integer value, range 1~3600 seconds, default '60' seconds.

**Return value description:**

Return a tuple in the following format:`(stage, state)`

| parameter  |   type | meaning                                                         |
| ----- | ---- | ------------------------------------------------------------ |
|    stage | integer | Indicate what state is currently being detected: 1 - Detecting SIM card status; 2 - Detecting network registration status; 3 - Detecting PDP Context activation status. |
|    state | integer | According to the stage value, different states are represented as follows: When stage=1, state represents the status of the SIM card, range 0-21, detailed description of each state value, please refer to[`sim.getStatus()`](../api_reference/iotlib/sim.md#sim.getStatus)the return value description of the method; When stage = 2, state indicate network registration status, range 0-11, detailed description of each state value, please refer to[`net.getState()`](../api_reference/iotlib/net.md#net.getState)the return value description of the method; When stage = 3, state represents the PDP Context activation status, 0 indicates no successful activation, 1 indicates successful activation. |



### **Exception handling**

There are two types of network anomalies. One is when a network exception occurs during startup, and the other is when the startup has been successfully logged into the network, and then the network exception occurs during subsequent business processing

#### Network exception handling during device startup

The network abnormalities during module startup mainly include three situations, namely:

- SIM card abnormality
- Module network registration failed
- Cellular wireless network card automatic activation failed

The direct result of these three situations is that the module cannot connect to the network. Therefore, we refer to these situations as "network anomalies", and the inspection and handling procedures for the above three types of anomalies are described in the previous text.

#### Network exception handling during device operation

After the cellular wireless network card is successfully activated, the user's application still needs to pay attention to one thing - the connection status between the device and the network. This is because during the operation of the device, the connection between the module and the network may be disconnected due to some abnormal reasons, such as network abnormalities, environmental interference, poor signals, etc. If the user application does not pay attention to such network events, it is likely to cause abnormal execution of network related business in the user application, resulting in unforeseen problems.

QuecPython provides network event monitoring functionality, allowing user applications to listen for network state change events by registering callback functions. When the connection status between the device and the wireless network changes, the system will automatically push the corresponding event to the user's application through the callback function registered by the user.

The method for registering a network listening callback function is as follows:

```python
dataCall.setCallback(fun)
```

An example of a callback function is as follows:

```python
def netCallback(args):
    profileID = args[0]
    netState = args[1]
    if netState == 0:
        print('### network {} disconnected.'.format(profileID))
    elif netState == 1:
        print('### network {} connected.'.format(profileID))
```

The parameter of this callback function is a tuple containing 3 elements, and currently users only need to focus on the first two elements. The first two parameters are described as follows:

| parameter    |  type | explain                                                         |
| ------- | ---- | ------------------------------------------------------------ |
|   args[0] | integer | The cellular wireless network card number indicates which wireless network card's network connection status has changed. |
|   args[1] | integer | Network status, 0 indicates network connection disconnected, 1 indicates network connection successful.             |

It is recommended that users register this callback function to monitor the network connection status, ensuring that when the network connection status changes, the user application can handle it in a timely manner according to the network status changes. Usually, we can refer to the following methods to handle it:

- After receiving an event indicating a change in network connection status in the callback function, the message queue function sends the network event to other threads for processing. Of course, users can also use QuecPython's `sys_bus` feature to replace message queues.
- When other threads receive a network event, they determine if it is a network disconnection event and stop network related services such as sockets and MQTT. At the same time, the thread can also choose to start a timer, such as setting the timer time to 60 seconds first. If the network has not been restored after 60 seconds, perform a CFUN0/1 switch and see if the network can be restored.

> **What is CFUN0/1 switching?**
>
> Through the explanation of CFUN in the chapter "Basic Concepts of Cellular Networks", it can be known that CFUN refers to the functional mode of mobile terminals. CFUN0/1 switching refers to using the `net.setModemFun(0)` method to first switch the device to mode 0 (minimum function mode), and then using the `net.setModemFun(1)` method to switch the device to mode 1 (full function mode). When switching to mode 0, the entire RF network protocol stack of the device is shut down, and the SIM card module stops supplying power; When switching to Mode 1 again, the power supply to the SIM card will be restored and initialized again. At the same time, the software and hardware functions related to RF will be restarted, and the device will initiate the network registration process again.
>
> **Why do we need to switch between CFUN0/1?**
>
> QuecPython has an automatic reconnection function. If a network exception occurs and the device is disconnected from the network, shouldn't it automatically recover after the exception disappears? Why is there still a need to switch between CFUN0/1 in the processing method mentioned above?
>
> We need to clarify that QuecPython's automatic reconnection function refers to the module automatically reactivating the wireless network card after a network anomaly is restored, rather than re registering with the network. The network registration behavior of the device is automatically controlled by the RF network protocol stack of the system. In theory, after the network abnormal factors disappear, the RF network protocol stack will automatically initiate network registration again. However, it cannot be ruled out that due to some reasons, the device may not have re registered with the network in a timely manner. At this point, because the device's network registration has not been successful, the wireless network card cannot be reactivated. Therefore, we took the initiative to switch between CFUN0/1. In fact, it's like when we encounter poor network or no signal when using our mobile phones, we choose to turn off the mobile network first and then turn it back on. Of course, users can also choose to restart.

