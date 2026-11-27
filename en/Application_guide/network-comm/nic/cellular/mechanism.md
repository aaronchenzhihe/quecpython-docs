# Unique Mechanisms of QuecPython Cellular NIC

This document introduces the unique mechanisms of QuecPython cellular NIC and describes the basic processes and principles of these mechanisms to help you better use the cellular NIC functionality of QuecPython.

## Overview

The processing mechanisms of QuecPython cellular NIC mainly include the following three parts:

* Automatic activation of the cellular NIC at startup (shorted as "automatic activation" below).

* Automatic reconnection of the cellular NIC (shorted as "automatic reconnection" below).

* Saving of cellular NIC configuration.

The following diagram shows the process of automatic activation of the QuecPython cellular NIC at startup. In general, it can be divided into three stages:

* Stage 1: Determine which cellular NIC(s) to enable automatic activation and automatic reconnection based on the existence of the *`datacall_config.json`* file.

* Stage 2: Determine which parameters to use for the activation of the cellular NIC based on whether you have configured APN and other NIC parameters.

* Stage 3: The previous two stages mainly involve parameter configuration, and this stage involves the actual activation of the NIC.

![](../../../media/network-comm/nic/cellular/QuecPython-cellular-nic-activation-mechanism.png)

The following will introduce the various processing mechanisms of the QuecPython cellular NIC based on the above flowchart for details.

## Automatic Activation

Automatic activation of the cellular NIC at startup refers to that after the module is powered on, the system automatically activates the cellular NIC when the module has registered on the network successfully.

Based on the flowchart above, you can see that the system will first determine which cellular NIC(s) to activate according to whether the *`datacall_config.json`* file exists. There are two processing mechanisms:

* If the *`datacall_config.json`* file does not exist, the system will activate the first cellular NIC.

* If the *`datacall_config.json`* file exists, the system will determine which cellular NIC(s) to activate at startup according to the configuration in the file.

After the system determines which cellular NIC(s) to activate, before activating the NIC, the system will further confirm whether you have configured APN and other parameters for the corresponding cellular NIC. There are two processing mechanisms:

* If you have not configured APN and other parameters, the default parameters will be used for the activation of the cellular NIC. The default parameter configuration is as follows:

| Parameter | Default Value | Description |
| --------- | ------------- | ----------- |
| ipType    | 0             | IPv4 protocol. |
| apn       | ""           | No APN.        |
| username  | ""            | No username. |
| password  | ""           | No password.   |
| authType  | 0             | No encryption. |

> If you use the default parameter configuration for the NIC activation, the activation may fail. You are recommended to configure the correct APN parameters according to the actual situation.

* If you have configured APN and other parameters, the configured APN parameters will be used for the activation of the cellular NIC.

## Automatic Reconnection

Automatic reconnection of the cellular NIC refers to the process where the module re-registers on the network and reactivates the cellular NIC after the exceptions are handled if the connection between the module and the network is closed due to some exceptions (such as network anomalies, environmental interference, and poor signal).

Based on the flowchart above, you can see that the system will first determine to enable the automatic reconnection of which cellular NIC(s) according to whether the *`datacall_config.json`* file exists. There are two processing mechanisms:

* If the *`datacall_config.json`* file does not exist, the system will enable the automatic reconnection of the first cellular NIC.

* If the *`datacall_config.json`* file exists, the system will enable the automatic reconnection of cellular NIC(s) according to the configuration in the file.

The basic working principle of the automatic reconnection mechanism of the cellular NIC is as follows:

![](../../../media/network-comm/nic/cellular/auto-connect.png)

The system will monitor the connection status between the device and the cellular network. When the network is disconnected and the factors causing the network exception disappear, the device will automatically initiate the attach procedure to re-register on the network. Once the registration is successful, the system will reactivate the NIC based on the parameters configured before.

Some common reasons that may cause the device's network disconnection are listed below:

* The device is in an environment with poor signal, which may easily cause the device's network to be disconnected, such as in a tunnel.

* The environment where the device is located has co-frequency interference or other interference, which affects the normal communication between the device and the base station, and may also cause the device's network disconnection.

* The cell that the device is connected to may become overloaded when there are too many devices connected to it, which may cause the device's network disconnection.

* An abnormality in the wireless network may cause the device's network disconnection.

## Saving of Cellular NIC Configuration

In QuecPython, the system saves the parameters related to the cellular NIC. The mechanism can be divided into two parts:

**Saving of Automatic Activation and Automatic Reconnection Configuration**

Based on the flowchart in the "*Overview*" section, you can see that the system will determine which cellular NIC(s) to enable automatic activation and automatic reconnection according to whether the *`datacall_config.json`* file exists. This is because the *`datacall_config.json`* file saves the configuration of whether each cellular NIC should be automatically activated and automatically reconnected. The content and format of the file are as follows:

```json
{
	"1": {"autoActivate": 0, "autoConnect": 0},
	"2": {"autoActivate": 0, "autoConnect": 0},
	"3": {"autoActivate": 0, "autoConnect": 0}
}
```

The configuration file contains 3 sets of configurations, which are used to control the automatic activation and automatic reconnection of the 3 cellular NICs. The parameter **`autoActivate`** controls whether to enable automatic activation, and the parameter **`autoConnect`** controls whether to enable automatic reconnection. 0: Disabled. 1: Enable.

You can modify the *`datacall_config.json`* file by calling the following two methods:

Configure automatic activation:

```python
dataCall.setAutoActivate(profileID, enable)
```

Configure automatic reconnection:

```python
dataCall.setAutoConnect(profileID, enable)
```

By default, the *`datacall_config.json`* file does not exist. Only when you call `dataCall.setAutoActivate` or `dataCall.setAutoConnect` to configure the corresponding feature will the system create the *`datacall_config.json`* file in the *`usr`* directory of the module and save the configuration in it. This file will not be lost if the device is powered off.

<br>

**Saving of Configured APN and Other Parameters**

The configured APN and other parameters for each cellular NIC will be saved in the system's NVM and will not be lost due to power off. That is, the information configured in the following method will be saved.

```python
dataCall.setPDPContext(profileID, ipType, apn, username, password, authType)
```



## checkNet Mechanism

`checkNet` is a feature provided by QuecPython. This feature is mainly used to check whether the network is ready. Please refer to [checkNet - Network Detection](../../../../api_reference/iotlib/checkNet.md) in the "API Reference" on the QuecPython official website for details.

**Scenarios for Using `checkNet`**

The process of UE network registration after power-on is relatively complicated, which requires the terminal device to interact with the base station and the core network to confirm information. This process is affected by many factors, such as the RF performance of the terminal device (hardware design and antenna), surrounding environment, cell signal coverage, and base station load. Therefore, it is common and normal for the cellular NIC to not be activated successfully when the device starts up and your application starts running.

If you perform network-related business operations in your applications from the beginning, the network business will likely fail due to the cellular NIC not being activated. Therefore, you are recommended to check whether the network is ready before performing network-related business operations such as socket, HTTP/HTTPs, and MQTT. You can call the following method to check whether the network is ready:

```python
checkNet.waitNetworkReady(timeout)
```

Advantages of this method:

- This method will check the SIM card status, device network registration status, and cellular NIC activation status. It can also set a timeout period, within which the result will be returned if the cellular NIC is detected and activated successfully; otherwise, the method will block and keep waiting until the timeout is reached before exiting.

- When this method times out and exits, a detailed error code will be returned to inform you that the cellular NIC is not activated and at which stage the activation fails. For example, the failure stage may be abnormal SIM card status, network registration failure, or NIC activation failure. The returned error code narrows the scope of the problem and helps you to quickly identify the problem.

**`checkNet` Mechanism Description**

The process of checking whether the network is ready by using `checkNet` is shown in the following figure. It is divided into three stages:

- Stage 1: First, check the SIM card status to confirm whether the SIM card is ready. If the SIM card is ready, proceed to the next stage; otherwise, wait for the SIM card to be ready until the timeout is reached before returning.

- Stage 2: Then, check the device's network registration status. If the device has successfully registered on the network, proceed to the next stage; otherwise, wait for the device to register on the network until the timeout is reached before returning.

- Stage 3: Finally, check the activation status of the cellular NIC. If the cellular NIC has been activated successfully, return the check result directly; otherwise, wait for the cellular NIC to be activated until the timeout is reached before returning.

![](../../../media/network-comm/nic/cellular/checkNet.png)

## Network Event Listening

Network event listening is one of the unique mechanisms of QuecPython cellular NICs. It aims to notify your application in an efficient and timely manner when the device's network connection status changes. You can make appropriate processing based on changes in the network status.

QuecPython provides APIs for your application to register callback functions. When the connection status between the module and the wireless network changes, the system will automatically push the corresponding event to your application through the registered callback function. Compared with frequently querying the device's network connection status actively in your application, QuecPython's network monitoring mechanism is more efficient and timely.

Call the following method to register the network status callback function.

```python
dataCall.setCallback(fun)
```

Please refer to [Register Callback Function](../../../../api_reference/iotlib/dataCall.md#Register-Callback-Function) in the "API Reference" on the QuecPython official website for details.

Example of the callback function:

```python
def network_event_callback(args):
    profileID = args[0]
    netState = args[1]
    if netState == 0:
        print('### network {} disconnected.'.format(profileID))
    elif netState == 1:
        print('### network {} connected.'.format(profileID))
```

Parameters of the callback function form a tuple consisting of 3 elements. Currently, you only need to pay attention to the first two elements. The explanations of the first two parameters are as follows:

| Parameter | Type    | Description                                                  |
| --------- | ------- | ------------------------------------------------------------ |
| args[0]   | Integer | Cellular NIC number, indicating which NIC's network connection status has changed. |
| args[1]   | Integer | Network status. 0: Network is disconnected. 1: The network is connected. |

In the following sample code, a callback function for network event listening is registered, and the network status is printed in the callback function. Switching the module to airplane mode first to simulate a network disconnection caused by network exceptions, and then switching back to full-functionality mode to simulate the process of network reconnection after the network exception is resolved.

```python
import net
import utime
import dataCall


def network_event_callback(args):
    profile_id = args[0]
    conn_state = args[1]
    ret = dataCall.getInfo(1, 0)
    if conn_state == 0:
        print('### NIC{} network disconnected.'.format(profile_id))
    elif conn_state == 1:
        print('### NIC{} network connected.'.format(profile_id))
    else:
        print('Unknown event.')
    print('Current NIC information:{}'.format(ret))


def main():
    # Register the callback function for network event listening
    dataCall.setCallback(network_event_callback)
    ret = dataCall.getInfo(1, 0)
    print('Current NIC information:{}'.format(ret))
    # Switch to airplane mode to disconnect the module from the network
    net.setModemFun(4)
    utime.sleep(5)
    # Switch to full-functionality mode to make the module re-register on the network and activate the cellular NIC
    net.setModemFun(1)


if __name__ == '__main__':
    main()
```

The above example aims to give you an intuitive understanding of QuecPython's network event listening feature. If you need to use this feature in actual projects, please refer to the [*Example of Handling Network Exception Events*](exception-handling.md#Example-of-Handling-Network-Exception-Events).



## Multiple NICs

"Multiple NICs" refers to the module virtually creating multiple cellular NICs in software to meet the needs of a device simultaneously connecting to multiple networks.

### Scenario

As mentioned before, APN is a required parameter when a terminal device connects to the network, and it determines the network access method for the terminal device. Different access points cover different ranges and access methods, so the APN determines how the terminal device accesses what kind of network. Can you configure multiple different APNs to allow the terminal device to simultaneously connect to different networks? The answer is yes.

Therefore, when you want your products to connect to public networks (Internet) while also access some dedicated networks, you can use multiple NICs. In this case, you need to configure different APNs for different NICs and then activate the NICs.

![](../../../media/network-comm/nic/cellular/multi-cellular-nics.png)

### Description

QuecPython modules on various platforms can create multiple virtual cellular NICs. For platform uniformity, three NICs are available for you to activate and use simultaneously and these NICs do not have any special purposes. At the same time, the VoLTE feature on each platform will occupy one NIC, but the specific NIC occupied by VoLTE varies on different platforms. But it will not occupy the three NICs open to users.

> For BG95 series modules, at most 2 NICs can be activated in NB-IoT networks.