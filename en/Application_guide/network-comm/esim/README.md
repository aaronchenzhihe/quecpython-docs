# eSIM

This article provides a brief introduction to the eSIM function. For more information, users can refer to the [Official Website of GSMA](https://www.gsma.com/esim/) to access the detailed protocol documents.

When using the LPA function of the eSIM card, users need to confirm with the card provider whether the eSIM card supports LPA function.

## Background Knowledge of eSIM

### Abbreviations

| Abbreviation | English                                 |
| :----------- | :-------------------------------------- |
| eUICC        | embedded UICC                           |
| LPA          | Local Profile Assistant                 |
| Profile      | Profile                                 |
| SM-DP/SM-DP+ | Subscription Manager Data Preparation   |
| SM-SR        | Subscription Manager Secure Routing     |
| SM-DS        | Subscription Manager - Discovery Server |
| RSP          | Remote SIM Provisioning                 |
| MNO          | Mobile Network Operator                 |
| EUM          | eUICC Manufacturer                      |

### What is eSIM

eSIM (Embedded SIM) is a new technology compared to traditional SIM cards, where the information can be remotely programmed without the need for physical replacement to change the network operator. This SIM card technology packages the SIM card information into an independent file called a profile, which can be downloaded to the eSIM to become a complete and usable SIM card. eSIM cards can conveniently switch network operators by downloading configuration files (Profiles) over the network, which is becoming increasingly advantageous and in demand with the development of globalization.

### Advantages of eSIM Cards

eSIM cards are generally integrated directly into mobile devices, and according to GSMA development standards, mobile devices with integrated eSIM cards can download and store profiles from multiple network operators. Users can choose to activate the profile of one network operator as needed, allowing the mobile device to access the Internet using the selected profile.

Compared to traditional SIM cards, the advantages of eSIM cards are as follows:

* Avoid the need for traditional SIM card insertion and removal, making it safer and more stable to use.

* eSIM cards are smaller in size, making them more versatile in various applications compared to traditional SIM cards.

* Unlike traditional SIM cards that require different SIM cards for different network operators, eSIM cards only require downloading the data file provided by the network operator, avoiding unnecessary waste.

* In some scenarios, using eSIM cards gives products a competitive advantage as they can easily switch network operators to adapt to different application scenarios without the need for hardware operations.

### Overall eSIM Business Process

The overall eSIM business process is shown in the following diagram:

![](../../media/network-comm/esim/8-3.png)

<center>Figure 1-1</center>

Explanation:

* Users access the network through personal computers or mobile devices and purchase eSIM services on the enterprise eSIM management platform or directly on the network operator's eSIM management platform (if available).

* The network operator stores the corresponding eSIM data in the SM-DP+ server.

* The enterprise eSIM management platform or the network operator's eSIM management platform (if available) sends the required parameters for downloading the profile to the personal computer or mobile device.

* The profile data is downloaded from the SM-DP+ server to the user's device through the personal computer or mobile device.

* The downloaded profile is installed and stored in the eSIM card, and then the successfully installed profile is activated to complete the eSIM card's network access.

### eSIM System Architecture

There are two solution architectures: M2M solution and Consumer solution.

#### M2M Solution

The M2M solution is the first remote SIM card provisioning solution developed by GSMA. There are two main reasons for this:

* The M2M solution is simpler and does not require interaction with end users.
* The immediate commercial need was for technical solutions that supported B2B2C deployments alongside regulatory requirements for the launch of services such as eCall.

Remote SIM Provisioning for M2M utilises a server driven (push model) to provision and remotely manage operator Profiles. This solution is organized around three elements: SM-DP (Subscription Manager-Data Preparation), SM-SR (Subscription Manager-Secure Routing), and eUICC.

The diagram below is the high-level representation of the M2M main system elements. Beyond common SIM functions, such as SIM Toolkit and Bearer Independent Protocol (BIP) support, the M2M solution does not impose additional requirements on M2M devices to enable usage of eUICCs.

![](../../media/network-comm/esim/8-1.png)

<center>Figure 1-2</center>

#### Consumer Solution

The Consumer solution is developed based on the M2M solution, allowing users to control their devices and perform business operations through terminal devices, providing users with greater freedom. As a result, this solution is more complex and requires comprehensive considerations. The GSM remote SIM provisioning consumer solution follows a client-driven(pull model) and enables control over remote provisioning and local management of operator profiles by the end user of the device.

This solution is organized around four elements: SM-DP+ (Subscription Manager-Data Preparation+), SM-DS (Subscription Manager-Discovery Server), LPA (Local Profile Assistant), and eUICC.

The following diagram (Figure 1-3) shows the main system elements and architecture of the Consumer Solution.

![](../../media/network-comm/esim/8-2.png)

<center>Figure 1-3</center>

### eSIM Profile Download Process

The following diagram shows the interaction process for downloading profiles from SM-DP+.

![](../../media/network-comm/esim/8-4.png)

<center>Figure 1-4</center>

### eSIM Use Cases

* Wearable Devices

Due to the small size of wearable devices, using eSIM cards can effectively reduce the size of the devices and provide a better user experience.

- In-vehicle

Due to the large range of vehicle activities, it often shuttles between different service areas of operators and requires convenient switching of operators.

- Cross-border devices

In the scenario of cross-border device movement, operator network switching is required, which cannot be satisfied by ordinary SIM cards. Global roaming card fees are relatively expensive. With eSIM, it is possible to switch to local operators freely without modifying the hardware.



## eSIM Supported Models

The specific models currently supported by QuecPython for eSIM functionality are as follows:

| Model              |
| :----------------- |
| EC200AAU_HA_SANX   |
| EC200AAU_HA_SANXIR |
| EC200AEU_HA_SANX   |
| EC200AEU_HA_SANXIR |
| BG95M1             |
| BG95M8             |
| EC600UEC_AC_WS     |





## eSIM Tool

### Tool Introduction

The QEsimTool software is mainly used to handle the situation of an empty eSIM card. The QEsimTool software needs to download data from the operator's server and requires the personal computer to have normal access to the Internet.

The operating mechanism of the QEsimTool software: The user obtains the AccCode from the operator, and passes the AccCode as a parameter to QEsimTool. QEsimTool resolves the operator's server address through the AccCode and sends a request to the operator's server to download profile data. After QEsimTool completes the download of profile data, it will send the data to the device through AT commands, and then write the profile into the eSIM card.

How to obtain the QEsimTool software: Please go to the [Download](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/QEsimTool_V1.0.1.7.zip) and contact customer service to obtain it.

### Interface Introduction

The QEsimTool software comes with an application guide document. The following is a brief introduction.

The startup interface of the QEsimTool software is as follows:

![](../../media/network-comm/esim/1-1.png)

<center>Figure 1-5</center>

Port: AT command interaction serial port.

Baud rate: Serial port baud rate.

OPEN: Open the Port with the specified baud rate.

AccCode: AccCode needs to be provided by the operator and filled in QEsimTool as a parameter.

DownloadProfile: Click this button to resolve the AccCode and download the profile through the network.

ListProfile: View the current profile list.

ICCID: ICCID number of the eSIM card.

EnableProfile: Enable the profile corresponding to ICCID.

DisableProfile: Disable the profile corresponding to ICCID.

DeleteProfile: Delete the profile corresponding to ICCID.

### Operation Instruction

Users can follow the steps below:

* After the module device starts normally, open the QEsimTool software on the personal computer.

* Check whether the personal computer can access the Internet normally.

* Select the Port serial port as the AT port, set the baud rate, and click the open button in Figure 1-5.

* Fill in the AccCode provided by the operator.

After successfully connecting to the device, execute the ATI command. If the version information is correctly displayed as shown in the figure below, you can click the DownloadProfile button to download and install the profile.

![](../../media/network-comm/esim/1-2.png)
<center>Figure 1-6</center>

> If any exceptions occur during the use of the software, please provide QuecWinLpa.log to the engineer for analysis.

## eSIM API Introduction

The following interfaces are implemented in the LPA method and require users and card vendors to confirm whether the eSIM card supports LPA functionality.

### Get EID

Users can obtain the ICCID of the eSIM card through the following button.

The interface is as follows:

```python
esim.getEid()
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
from sim import esim

eid = esim.getEid()
print("Get Eid: {}".format(eid))

```

### Callback Registration

Users can register a callback function through the following interface to notify the application layer of the result of profile download and installation.

The interface is as follows:

```python
esim.setCallback(usrFun)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
from sim import esim

def usrFun(result):
    '''
    :param result: Result of OTA profile download.
    :type result: Integer, 0: success, 1: failure
    '''
    print("OTA result: {}".format(result))

esim.setCallback(usrFun)

```

### Query Current SIM Card Profile

Users can obtain the profile information in the current eSIM card through the following interface. Based on the queried profile information, users can choose to enable the corresponding operator network function.

The interface is as follows:

```python
esim.getProfileInfo(mode)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
from sim import esim

# List the profiles with category 2 in the configuration file
profile_list = esim.getProfileInfo(0)
print("Get class 2 profile: {}".format(profile_list))

# List all profiles
profile_list = esim.getProfileInfo(1)
print("Get all profiles: {}".format(profile_list))

```

### Download and Install Operator Profile

Users can download and install the operator's profile through the following interface. If the eSIM is an empty card, please use the QEsimTool tool to install the first profile. To ensure that this interface can download the profile information normally, the currently enabled profile of the eSIM card must be able to successfully register on the network and can interact with the operator's server address in the AccCode normally. After calling this interface, other operations cannot be performed until the download result notification arrives, otherwise it will affect the download and installation result.

The interface is as follows:

```python
esim.profileOTA(activationCode, confirmationCode)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
from sim import esim
def usrFun(result):
    '''
    :param result: Result of OTA profile download.
    :type result: Integer, 0: success, 1: failure
    '''
    print("OTA result: {}".format(result))

esim.setCallback(usrFun)

# activationCode Please use your own data. The following activationCode is for example only.
esim.profileOTA('LPA:XXX','')

```

### Control the Current SIM Card Profile

Users can enable, disable, or delete the profile corresponding to the specified ICCID through the following interface.

The interface is as follows:

```python
esim.profileHandle(profile_tag, iccid)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example, this example requires that the eSIM card must have a profile

from sim import esim

def handle_profile(ICCID,mode):
    esim.profileHandle(mode, ICCID)

# Get the ICCID of the current profile
profile_list=list(esim.getProfileInfo(0))

# Change the status of the profile
if profile_list[1][0][1] == 1:
    print("ICCID {} is enabled".format(profile_list[1][0]))
    # disable this ICCID
    esim.profileHandle(1, profile_list[1][0][0])
elif profile_list[1][0][1] == 0:
    print("ICCID {} is disabled".format(profile_list[1][0]))
    # enable this ICCID
    esim.profileHandle(0, profile_list[1][0][0])
else:
    raise ValueError("profile_list is invalid")

# Delete the profile
esim.profileHandle(2, profile_list[1][0][0])

```

### Get the Profile to be Deleted

Users can query the profile information that has been deleted locally but has not been deleted on the server through the following interface.

The interface is as follows:

```python
esim.getProfileDelNotification()
```

Example

```python
# -*- coding: UTF-8 -*-
# Example, this example requires that the eSIM card must have a profile

from sim import esim

del_error_profile=esim.getProfileDelNotification()
print("Profile information that has been deleted locally but not reported to the server:{}".format(del_error_profile))

```

### Report the Profile to be Deleted

Users can initiate a request to the server to delete the specified profile again through the following interface to ensure that the server completes the deletion operation of the profile information.

This function parameter needs to be obtained through `esim.getProfileDelNotification()`.

The interface is as follows:

```python
esim.reportProfileDelNotification(iccid)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example, this example requires that the eSIM card must have a profile

from sim import esim

del_error_profile=esim.getProfileDelNotification()
print("Profile information that has been deleted locally but not reported to the server: {}".format(del_error_profile))

# Initiate a request to the server to delete the profile corresponding to this ICCID
esim.reportProfileDelNotification(profile_list[1][0][0])

```

## eSIM Application Example

* Users purchase eSIM cards from operators.
* After using for a period of time, users want to switch to another operator.

To solve the above problems, the user application example is as follows:

```python
# -*- coding: UTF-8 -*-
# Example, this example requires that the eSIM card must have a profile

try:
    from sim import esim
except Exception as e:
    print("this version no this modles")
import utime
import checkNet
import _thread
import atcmd

class pyesim():
    def __init__(self):
        self.stage, self.state = checkNet.waitNetworkReady(60)
        if self.stage == 3 and self.state == 1:
            print("network is success!")
            self.debug_enbale = True
            self.ota_result = False
            pass
    def esim_log(self,args):
        if self.debug_enbale == True:
            print("esim {}".format(args))
            pass
    def get_eid(self):
        self.eid = esim.getEid()
        self.esim_log("esim get id {} ".format(self.eid))

    def get_profile_list(self,model = 0):
        self.profile_list=esim.getProfileInfo(model)
        self.esim_log("esim get_profile_list({}): {}".format(model,self.profile_list))

    def profile_handle(self,profile_tag,iccid):
        result = esim.profileHandle(profile_tag, iccid)
        self.esim_log("esim profile_handle({},{}) = {}".format(profile_tag,iccid,result))

    def profile_ota(self,activationCode,confirmationCode):
        ret = esim.profileOTA(activationCode, confirmationCode)
        self.esim_log("esim profileOTA('{}','{}') = {}".format(activationCode,confirmationCode,ret))

    def wait_ota_result(self):
        i_loop_nunmbers = 120
        while self.ota_result == True or i_loop_nunmbers > 0:
            i_loop_nunmbers = i_loop_nunmbers - 1
            utime.sleep(1)
            self.esim_log("wait ota ... {}".format(i_loop_nunmbers))

    def callback(self,args):
        if args == 0:
            self.esim_log("esim callback OTA {} success".format(args))
            self.ota_result = True
        else:
            self.esim_log("esim callback OTA {} failed".format(args))
            self.ota_result = False

    def set_callback(self):
        ret = esim.setCallback(self.callback)
        self.esim_log("esim set_callback  Result {}".format(ret))

    def get_profile_del_notification(self):
        self.profile_notification = esim.getProfileDelNotification()
        self.esim_log("esim get_profile_del_notification {}".format(self.profile_notification))

    def report_profile_notification(self,iccid):
        ret = esim.reportProfileDelNotification(iccid)
        self.esim_log("esim report_profile_notification {}".format(ret))

esim_t = pyesim()

def run():

    esim_t.get_eid()
    esim_t.set_callback()
    esim_t.esim_log("will OTA iccid ")

    esim_t.esim_log("will display current info")
    esim_t.esim_log("-------------------------------------------------")
    esim_t.get_profile_list(0)
    esim_t.esim_log("-------------------------------------------------")
    # Download and install the profile of the operator to be switched
    # The following activationCode is for example only.
     esim_t.profile_ota('LPA:XXX','')

    # Wait for the download result
    esim_t.wait_ota_result()

    # Delete the original operator's profile information
    uiccid = 'XXX'
    esim_t.esim_log("will delelte uiccid {}".format(uiccid))
    esim_t.profile_handle(2, uiccid)
    esim_t.esim_log("will display delelte iccid  info")
    esim_t.esim_log("-------------------------------------------------")
    esim_t.get_profile_list(0)
    esim_t.esim_log("-------------------------------------------------")


if __name__ == '__main__':
    _thread.start_new_thread(run,())

```




## Frequently Asked Questions

### Unable to download and install profile

* Check if the AccCode is correct, you can confirm with the operator.
* Check if the current device or PC can access the URL address parsed from the AccCode.
* Check if the profile is being downloaded and installed repeatedly.
* Check if the operator has any restrictions, such as only allowing one-time use of the same AccCode, which needs to be confirmed with the operator.
* Check if the network signal is too weak or unstable.

### Unable to access the internet after downloading and installing the profile

If you are unable to access the internet after successfully downloading and installing the profile, please perform a CFUN=0/1 switch operation. If you still cannot access the internet, please check the following:

* Check if the SIM card status is abnormal.
* Check if the downloaded and installed profile has been activated successfully.
* Check if the downloaded and installed profile is valid, you can confirm with the operator.
* Check if the device has not performed a dial-up operation (if the device is set to not activate automatically, check if the activation operation has not been performed).

### "eSIM card not inserted" error message

Please refer to the QuecPython official website wiki for cellular wireless card [Handle Network Exceptions](../nic/cellular/exception-handling.md#Handle-Network-Exceptions-at-Startup).

