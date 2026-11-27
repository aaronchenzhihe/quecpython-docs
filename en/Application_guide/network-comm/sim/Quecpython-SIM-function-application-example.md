# SIM Card API Documentation

## Query SIM Card Information

### Query IMSI

Users can use the following interface to get the IMSI number of the SIM card for application purposes.

Interface:

```python
sim.getImsi()
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

imsi = sim.getImsi()
print("Get IMSI : {}".format(imsi))

```

### Query ICCID

Users can use the following interface to query the ICCID of the SIM card for application purposes.

Interface:

```python
sim.getIccid()
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

iccid = sim.getIccid()
print("Get ICCID : {}".format(iccid))

```

### Query Phone Number

Users can use the following interface to query the phone number of the SIM card. The SIM phone number needs to be set before querying.

Interface:

```python
sim.getPhoneNumber()
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

phone_num = sim.getPhoneNumber()
print("Get Phone Number is : {}".format(phone_num))

```

### Query Information Application Scenario

After the application starts, wait for the network registration to be successful, and then query the SIM card information to prepare for subsequent processing.

```python
# -*- coding: UTF-8 -*-
# Example
import sim
import checkNet
import sys

sim_status_dict={
    0:"SIM card does not exist/has been removed",
    1:"SIM card is ready",
    2:"SIM card is locked, waiting for CHV1 password",
    3:"SIM card is blocked, requires CHV1 password to unlock",
    4:"SIM card is locked due to SIM/USIM personalization check failure",
    5:"SIM card is blocked due to PCK error, requires MEP password to unblock",
    6:"Requires key to hide phone book entry",
    7:"Requires encoding to unlock hidden key",
    8:"SIM card is locked, waiting for CHV2 password",
    9:"SIM card is blocked, requires CHV2 unlock password",
    10:"SIM card is locked due to network personalization check failure",
    11:"SIM card is blocked due to incorrect NCK, requires MEP unlock password",
    12:"SIM card is locked due to sub-network lock personalization check failure",
    13:"SIM card is blocked due to incorrect NSCK, requires MEP unlock password",
    14:"SIM card is locked due to service provider personalization check failure",
    15:"SIM card is blocked due to incorrect SPCK, requires MEP unlock password",
    16:"SIM card is locked due to enterprise personalization check failure",
    17:"SIM card is blocked due to incorrect CCK, requires MEP unlock password",
    18:"SIM card is initializing, waiting for completion",
    19:"CHV1/CHV2/PIN error",
    20:"Invalid SIM card",
    21:"Unknown status"
}

if __name__ == '__main__':
	
    # 1. Query SIM card status
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("Interface returned failure")
        sys.exit()
    print("Get SIM status : {}".format(sim_status_dict[sim_status]))
    if sim_status != 1:
        sys.exit()
    
    # 2. Wait for network registration to be successful    
    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1:
        print('Network connection successful.')
    else:
        print('Network connection failed, stage={}, state={}'.format(stage, state))
        sys.exit()
    
    # 3. Query SIM card information after network registration
    
    imsi = sim.getImsi()
    if type(imsi).__name__ == 'int':
        print("Get IMSI failed !")
        sys.exit()
    print("Get IMSI is : {}".format(imsi))
    
    iccid = sim.getIccid()
    if type(iccid).__name__ == 'int':
        print("Get ICCID failed !")
        sys.exit()
    print("Get ICCID is : {}".format(iccid))
   	
    phone_number= sim.getPhoneNumber()
    if type(phone_number).__name__ == 'int':
        print("Get phone_number failed !")
        sys.exit()
    print("Get phone number is : {}".format(phone_number))
    
```

## Query SIM Card Status

Users can use the following interface to get the current status of the SIM card and determine if the SIM card can be used normally.

Interface:

```
sim.getStatus()
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim
import sys

sim_status_dict={
    0: "SIM card does not exist/has been removed",
    1: "SIM card is ready",
    2: "SIM card is locked, waiting for CHV1 password",
    3: "SIM card is blocked, requires CHV1 password to unlock",
    4: "SIM card is locked due to SIM/USIM personalization check failure",
    5: "SIM card is blocked due to PCK error, requires MEP password to unblock",
    6: "Requires key to hide phone book entry",
    7: "Requires encoding to unlock hidden key",
    8: "SIM card is locked, waiting for CHV2 password",
    9: "SIM card is blocked, requires CHV2 unlock password",
    10: "SIM card is locked due to network personalization check failure",
    11: "SIM card is blocked due to incorrect NCK, requires MEP unlock password",
    12: "SIM card is locked due to sub-network lock personalization check failure",
    13: "SIM card is blocked due to incorrect NSCK, requires MEP unlock password",
    14: "SIM card is locked due to service provider personalization check failure",
    15: "SIM card is blocked due to incorrect SPCK, requires MEP unlock password",
    16: "SIM card is locked due to enterprise personalization check failure",
    17: "SIM card is blocked due to incorrect CCK, requires MEP unlock password",
    18: "SIM is initializing, waiting to complete",
    19: "CHV1/CHV2/PIN error",
    20: "Invalid SIM card",
    21: "Unknown status"
}

if __name__ == '__main__':
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("Interface returned failure")
        sys.exit()
    if sim_status != 1:
        print("Get SIM status: {}".format(sim_status_dict[sim_status]))
        sys.exit()
    print("Get sim_status: {}".format(sim_status_dict[sim_status]))

```

## PIN Verification Function

Users can enable PIN verification function to ensure the security of SIM card usage. PIN is the security identification code of the SIM card, used to verify the user's identity and protect personal information security. After enabling PIN verification, the SIM card needs to be verified with the PIN code after each device restart (or CFUN0/1 switch) before it can be used normally. If the PIN code is entered incorrectly for three consecutive times, the SIM card will be locked and requires PUK code to unlock.

### Enable PIN Verification

Users can enable the PIN verification function of the SIM card through the following interface to ensure the security of SIM card usage.

The interface is as follows:

```python
sim.enablePin(pin)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

# Enter the actual PIN code, "1234" is an example PIN code here
result = sim.enablePin("1234")
print("Enable PIN: {}".format(result))

```

### Disable PIN Verification

Users can disable the PIN verification function through the following interface to simplify the application program when the SIM card is used in a secure environment.

The interface is as follows:

```python
sim.disablePin(pin)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

# Enter the actual PIN code, "1234" is an example PIN code here
result = sim.disablePin("1234")
print("Disable PIN: {}".format(result))

```

### PIN Verification

Users can complete the PIN verification of the SIM card through the following interface. After enabling PIN verification, the SIM card needs to complete the PIN verification before the status can be switched to normal, and then the application program can process the services.

The interface is as follows:

```python
sim.verifyPin(pin)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

# Enter the actual PIN code
result = sim.verifyPin("1234")
print("Verify PIN result is: {}".format(result))
```

### Change PIN

Users can periodically change the PIN code of the SIM card through the following interface to improve the security of SIM card usage.

The interface is as follows:

```python
sim.changePin(oldPin, newPin)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

sim.changePin("1234", "4321")
```

### PIN Code Application Scenario

In order to securely use the SIM card, users need to process the services with the PIN code verification function enabled.

```python
# -*- coding: UTF-8 -*-
# Example
import sim
import checkNet
import sys

sim_status_dict={
    0: "SIM card does not exist/has been removed",
    1: "SIM card is ready",
    2: "SIM card is locked, waiting for CHV1 password",
    3: "SIM card is blocked, requires CHV1 password to unlock",
    4: "SIM card is locked due to SIM/USIM personalization check failure",
    5: "SIM card is blocked due to PCK error, requires MEP password to unblock",
    6: "Requires key to hide phone book entry",
    7: "Requires encoding to unlock hidden key",
    8: "SIM card is locked, waiting for CHV2 password",
    9: "SIM card is blocked, requires CHV2 unlock password",
    10: "SIM card is locked due to network personalization check failure",
    11: "SIM card is blocked due to incorrect NCK, requires MEP unlock password",
    12: "SIM card is locked due to sub-network lock personalization check failure",
    13: "SIM card is blocked due to incorrect NSCK, requires MEP unlock password",
    14: "SIM card is locked due to service provider personalization check failure",
    15: "SIM card is blocked due to incorrect SPCK, requires MEP unlock password",
    16: "SIM card is locked due to enterprise personalization check failure",
    17: "SIM card is blocked due to incorrect CCK, requires MEP unlock password",
    18: "SIM is initializing, waiting to complete",
    19: "CHV1/CHV2/PIN error",
    20: "Invalid SIM card",
    21: "Unknown status"
}

if __name__ == '__main__':
    
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("Interface returned failure")
        sys.exit()
    
    if sim_status == 2:
        # PIN code verification is enabled, need to verify PIN code
        PIN="1234"          #PIN code - please fill in the real PIN code of the SIM card
        pin_result = sim.verifyPin(PIN)
        if pin_result != 0:
            print("Error PIN ! Please input right PIN!")
            sys.exit()
        
        # After successful verification, disable PIN code verification, no need to verify PIN code next time
        # result = sim.disablePin(PIN)
        #if result != 0:
        #    print("Error PIN ! Please input right PIN!")
        #    sys.exit()
    
    # Check SIM status
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("Interface returned failure")
        sys.exit()
    if sim_status != 1:
        print("Get SIM status : {}".format(sim_status_dict[sim_status]))
        sys.exit()
    
    print("Get sim_status : {}".format(sim_status_dict[sim_status]))
    
    # Wait for successful network registration and process services
    stage,state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1:
        print('Network connection successful.')
    else:
        print('Network connection failed, stage={}, state={}'.format(stage, state))
        
```

## SIM Card Unlock

Users can use the following interface to handle the situation where the PIN is locked due to three consecutive incorrect PIN entries. This interface unlocks the PIN using the PUK code and sets a new PIN at the same time. Please note that the SIM card will be permanently locked and automatically scrapped after 10 incorrect PUK code entries.

The interface is as follows:

```python
sim.unblockPin(puk, newPin)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

sim.unblockPin("12345678", "0000")

```

## Phone Book

In the actual project application process, if users need to save the phone number of the other party, they need to use the phonebook function. This interface provides the function of saving phone number information. Please refer to the QuecPython official website wiki [Phone Book](../../../api_reference/iotlib/sim.md#Phone-Book) for the support of this function interface.

### Read Phone Book

Users can use the following interface to read the phone book. The storage location of the phone number is described in the QuecPython official website wiki `sim.readPhonebook()` [Parameter](../../../api_reference/iotlib/sim.md#Phone-Book). Note that the difference between start and end needs to be less than or equal to 20.

The interface is as follows:

```python
sim.readPhonebook(storage, start, end, username)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

# Read the phone information with index 1 to 4 in the phonebook of this SIM card
sim_phonebook = sim.readPhonebook(9, 1, 4, "")
print("Phonebook in SIM card: {}".format(sim_phonebook))

# Read the phone information with index 1 to 4 in the missed call list
missed_call = sim.readPhonebook(4, 1, 4, "")
print("Missed call: {}".format(missed_call))

```

### Set Phone Book

Users can use the following interface to save phone number information to the phone book:

```python
sim.writePhonebook(storage, index, username, number)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

# Save Tom's phone number to index 1 in the phone book of the SIM card
sim.writePhonebook(9, 1, 'Tom', '18144786859')

```

### Phone Book Application Scenario

In the project operation, get the phone number and save it to the phone book of the SIM card.

```python
# -*- coding: UTF-8 -*-
# Example
import sim

def save_phone_number_to_simbook(index, name, phonenumber, input_type=0):
    # type = 0, if the phone number already exists in the location where you save the phone number, return directly without overwriting the save
    if input_type == 0:
        # check if there is already a phone number at the current position
        phone_info = sim.readPhonebook(9, index, index, name)
        if type(phone_info).__name__ == 'int':
            print("Does not exist")
        elif phone_info[1][0][1] == name and phone_info[1][0][2] == phonenumber:
            # the phone number already exists
            print("Already exists")
            return
        else:
            print("Does not exist")
    else:
        pass
    # save the name and phone number to the SIM card phone book at the index address
    result = sim.writePhonebook(9, index, name, phonenumber)

def get_name_phone():
    return 'Tom', '18144786858'

if __name__ == '__main__':
    name, phonenumber = get_name_phone()

    # if the name and phone number already exist, do not save
    save_phone_number_to_simbook(1, name, phonenumber, 0)

    # if the name and phone number already exist, overwrite the save
    save_phone_number_to_simbook(1, name, phonenumber, 1)
```

## SIM Card Hot Swap

In the project, if the user needs to monitor the SIM card insertion and removal, they can register a callback function through the following interface. When the SIM card is inserted or removed, the system will call the registered callback function and pass the SIM card insertion or removal event to the application layer. Users can determine whether the SIM card is inserted or removed based on the notification event. For the support of this interface by QuecPython modules, please refer to the QuecPython official website wiki [Hot Swap](../../../api_reference/iotlib/sim.md#Hot-Swap).

### Register Hot Swap Callback

Users can register a SIM card hot swap callback function through the following interface.

The interface is as follows:

```python
sim.setCallback(usrFun)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

def sim_hot_call(event):
    if event == 1:
        # SIM card inserted
        print("Get SIM Insert event: {}".format(event))
    elif event == 2:
        # SIM card removed
        print("Get SIM Unplug event: {}".format(event))
    else:
        # Unknown event
        print("Get SIM Unknown event: {}".format(event))

sim.setCallback(sim_hot_call)
```

### Get SIM Card Hot Swap Configuration

Users can query the current configuration information of the SIM card hot swap function through the following interface. Based on the configuration information, users can determine whether the hot swap function is currently enabled.

The interface is as follows:

```python
sim.getSimDet()
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

sim_hot_conf = sim.getSimDet()
print("Get Current SIM config info: {}".format(sim_hot_conf))
if sim_hot_conf[0] == 1:
    # SIM card hot swap function enabled
    print("Current SIM Hotplug is enabled")
```

### Configure SIM Card Hot Swap

Users can enable or disable the SIM card hot swap function through the following interface.

The interface is as follows:

```python
sim.setSimDet(switch, triggerLevel)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

# First determine whether the level is high or low when the SIM card is inserted, taking high level as an example here

# 1. Register the hot swap callback function
def sim_hot_call(event):
    print("Get SIM event: {}".format(event))
    
sim.setCallback(sim_hot_call)

sim_hot_conf = sim.getSimDet()
# 2. Enable the SIM card hot swap function and mark the module's SIM card insertion level as high level
if sim_hot_conf[0] == 0:
    print("SIM hot-swapping is not currently enabled")
    print("Will enable this function")
    sim.setSimDet(1, 1)
else:
  	print("Already enable this function")
```

### Application Scenarios of SIM Card Hot Swap

In the project, the user detects the SIM card insertion and removal events, and inserts a flag to indicate whether a SIM card insertion or removal event has occurred. (Users can decide whether to notify other modules in the application program that a SIM card insertion or removal event has occurred according to their needs).

```python
# -*- coding: UTF-8 -*-
# Example
import sim
import net
insert_flag = False
def sim_hot_call(event):
    global insert_flag
    print("Get SIM event: {}".format(event))
    if event == 1:
        # Detected SIM card insertion, set the flag to indicate SIM card is inserted
        insert_flag = True
    else:
        insert_flag = False

if __name__ == '__main__':
    sim_hot_conf = sim.getSimDet()
    if sim_hot_conf[0] == 0:
    
        # To enable the SIM card hot swap function, please set the level according to the actual SIM card level
        # Here is an example of setting the SIM card level to 1
        print("enable this function")
        sim.setSimDet(1, 1)
    
    else:
        print("Already enable this function")
    
    # Set the callback function to detect the insertion and removal of the SIM card
    sim.setCallback(sim_hot_call)

```

## Switch SIM Card 

In the process of project application, in order to ensure network stability and meet various network scenarios, it is necessary to have the ability to switch between two different SIM cards at any time to cope with various network abnormal situations. For the support of this function by QuecPython module, please refer to the [Switch SIM Card](../../../api_reference/iotlib/sim.md#Switch-SIM-Card) in the QuecPython official website wiki.

### Register Switch Card Callback Function

Users can register a callback function to monitor the switch card event through the following interface. When a switch card event occurs, the system will notify the application layer through the registered callback function. After receiving the switch card event notification, the application layer will handle the network abnormal situations caused by the switch card.

The interface is as follows:

```python
sim.setSwitchcardCallback(usrFun)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim
import net

sim_status_dict={
    0:"SIM card does not exist/has been removed",
    1:"SIM card is ready",
    2:"SIM card is locked, waiting for CHV1 password",
    3:"SIM card is blocked, requires CHV1 password to unlock",
    4:"SIM card is locked due to SIM/USIM personalization check failure",
    5:"SIM card is blocked due to PCK error, requires MEP password to unbar",
    6:"Requires key to hide phone book entry",
    7:"Requires encoding to unlock hidden key",
    8:"SIM card is locked, waiting for CHV2 password",
    9:"SIM card is blocked, requires CHV2 unlock password",
    10:"SIM card is locked due to network personalization check failure",
    11:"SIM card is blocked due to incorrect NCK, requires MEP unlock password",
    12:"SIM card is locked due to sub-network lock personalization check failure",
    13:"SIM card is blocked due to incorrect NSCK, requires MEP unlock password",
    14:"SIM card is locked due to service provider personalization check failure",
    15:"SIM card is blocked due to incorrect SPCK, requires MEP unlock password",
    16:"SIM card is locked due to enterprise personalization check failure",
    17:"SIM card is blocked due to incorrect CCK, requires MEP unlock password",
    18:"SIM card is initializing, waiting for completion",
    19:"CHV1/CHV2/PIN error",
    20:"Invalid SIM card",
    21:"Unknown status"
}

def switch_sim_call(result):
    print("Get switch SIM card event : {}".format(reslut))
    if result == 7:
        # SIM card switch succeeded, notify the application layer that the SIM card switch succeeded
        print("SIM switch success!")
    elif result == 9:
        print("SIM switch failed!")

if __name__ == '__main__':
    
    sim.setSwitchcardCallback(switch_sim_call)

```

### Query Current SIM Card ID

Users can query the ID of the currently enabled SIM card through the following interface.

The interface is as follows:

```python
sim.getCurSimid()
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

simid = sim.getCurSimid()
print("Current SIM is : {}".format(simid))
```

### Dual SIM Card Switching

Users can switch to a specified SIM card through the following interface. If the current SIM card encounters an unrecoverable abnormal network situation, this interface can be used to switch the currently used SIM card to another SIM card, so that the application can continue to process business.

The interface is as follows:

```python
sim.switchCard(simId)
```

Example:

```python
# -*- coding: UTF-8 -*-
# Example
import sim

simid = sim.getCurSimid()
print("Current SIM is : {}".format(simid))

if simid == 0:
    # Enable SIM2
    print("Current is SIM 1 and Will switch to SIM2")
    sim.switchCard(1)
    
```

### Dual SIM Card Switching Application Scenarios

The following scenarios may require dual SIM card switching operation based on customer requirements and business processes.

* If the current card signal is weak and cannot meet the business requirements, try switching to another card, such as bicycles, walkie-talkies.
* One card is connected to the public network and the other card is connected to the private network. The public network card is mainly used for data services, and the private network card is used for special tasks, such as power grid customers.

* When the user application detects that the current SIM card status is invalid and cannot perform network interaction, switch to another SIM card.

```python
# -*- coding: UTF-8 -*-
# Example
import sim
import net
import sys

sim_status_dict={
    0: "SIM card does not exist/has been removed",
    1: "SIM card is ready",
    2: "SIM card is locked, waiting for CHV1 password",
    3: "SIM card is blocked, requires CHV1 password to unlock",
    4: "SIM card is locked due to SIM/USIM personalization check failure",
    5: "SIM card is blocked due to PCK error, requires MEP password to unblock",
    6: "Requires key to hide phone book entry",
    7: "Requires encoding to unlock hidden key",
    8: "SIM card is locked, waiting for CHV2 password",
    9: "SIM card is blocked, requires CHV2 unlock password",
    10: "SIM card is locked due to network personalization check failure",
    11: "SIM card is blocked due to incorrect NCK, requires MEP unlock password",
    12: "SIM card is locked due to sub-network lock personalization check failure",
    13: "SIM card is blocked due to incorrect NSCK, requires MEP unlock password",
    14: "SIM card is locked due to service provider personalization check failure",
    15: "SIM card is blocked due to incorrect SPCK, requires MEP unlock password",
    16: "SIM card is locked due to enterprise personalization check failure",
    17: "SIM card is blocked due to incorrect CCK, requires MEP unlock password",
    18: "SIM card is initializing, waiting for completion",
    19: "CHV1/CHV2/PIN error",
    20: "Invalid SIM card",
    21: "Unknown status"
}

def switch_sim_call(result):
    print("Get switch SIM card event: {}".format(result))
    if result == 7:
        print("SIM switch success!")
        net.setModemFun(0)
        net.setModemFun(1)
    elif result == 9:
        print("SIM switch failed!")


if __name__ == '__main__':

    sim.setSwitchcardCallback(switch_sim_call)
    
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("Interface returned failure")
        sys.exit()
    if sim_status != 20:
        # Invalid SIM card status
        print("Get SIM status: {}".format(sim_status_dict[sim_status]))
        
        # Query current SIM card in use
        sim_id = sim.getCurSimid()
        
        # Switch to the other card to operate
        if sim_id == 0:
            sim.switchCard(1)
        else:
            sim.switchCard(0)
```