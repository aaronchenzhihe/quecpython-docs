# SMS

## SMS Concept

SMS refers to the text or numeric messages sent or received by users through mobile phones or other terminal devices. It is short for Short Message Service. The maximum length of a SMS is usually 140 English or numeric characters, or 70 Chinese characters.

SMS is mainly transmitted through cellular wireless networks. In general, SMS can be sent without an internet connection. It is transmitted through base stations to the SMS center (SMSC). The SMSC forwards the received SMS to the base station near the receiving device. Finally, the base station sends the message to the receiving device. From the above, it can be seen that SMS mainly relies on cellular wireless networks and does not depend on the internet.

The data flow framework of SMS is shown in the following diagram.

![](../media/network-comm/sms/1-1.png)

<center>Figure 1-1</center>

Explanation of data flow:

* The sender edits the SMS and sends the message content to the base station through a wireless signal.

* The base station receives the message content and forwards it to the operator's SMS service center through a series of network elements.

* The operator's SMS service center forwards the data to the base station near the receiving device through a series of network elements.

* The base station near the receiving device sends the SMS content to the receiving device.

## Background Knowledge

### Evolution of SMS Implementation Solutions

In the 2G era, a more complex CS/PS domain architecture was used. SMS, as a product of the last century, has always been based on the CS domain architecture of 2G. In the 2G/3G era, the SMS network elements deployed in the network are SMSC, which are mainly used for SMS storage and forwarding, and there are many of them in each operator.

In the LTE era, the LTE core network no longer has a dedicated SMS unit. The MME unit of the LTE core network and the MSC server unit of 2G are connected through SGs to achieve SMS functionality.

The implementation of SMS functionality can be divided into three categories:

* CS SMS: SMS is sent and received through the circuit domain of the 2G/3G network. When the module is registered under the 4G network, it can fall back to the 2G/3G network to support SMS functionality through CSFB.

* SMS Over SGs, also known as NAS SMS: It requires SIM card support for joint registration of 2G/3G/4G. In this type of SMS, the SMS messages from the UE to the base station are transmitted through the NAS messages of LTE. After reaching the core network, the core network connects to the MSC network element of the 2G/3G network through the SGs interface, and the MSC forwards the message to the target device.

* SMS Over IMS， IMS (VOLTE) SMS，SMS messages are transmitted through the LTE network and require module support for VOLTE and successful IMS registration.

#### 2G/3G Network Architecture

As shown in the figure 2-1 below, in the 2G/3G era, the UE device sends and receives data through a SMS receiver and sender, and the data is ultimately sent to the SMSC to complete the SMS communication.

![](../media/network-comm/sms/2-1.png)

<center>Figure 2-1</center>

#### 4G SMS Network Architecture

The current 4G SMS network framework is shown in the figure 2-2 below. The UE sends the SMS data to the eNodeB, and the eNodeB sends the SMS data to the SMSC, which then sends the data to the receiving end. The difference between the two is that the 2G/3G and 4G network elements and network structures are different. The 4G network elements and network structures are more diverse in this aspect, and SMS sending and receiving functions are implemented through the newly added IP-SM-GW network element.

![](../media/network-comm/sms/2-2.png)

<center>Figure 2-2</center>

### Introduction to SMS

Currently, SMS messages can be classified into: TEXT mode, block mode, and PDU mode.

* TEXT mode is the commonly used text mode.
* PDU mode is the protocol data unit mode.
* Block mode is rarely used now. QuecPython currently does not support this mode.

### SMS Encoding Methods

SMS encoding methods include: GSM encoding, IRA 8-bit encoding, and UCS2 encoding.

* GSM encoding uses 7 bits to represent a character in an SMS content.

* UCS2 uses 16 bits to represent a character in an SMS content.
* IRA uses 8 bits to represent a character in an SMS content.

Since a single SMS supports a maximum of 140 bytes at the underlying layer, with each byte being 8 bits, the formula for calculating the maximum character data that can be sent in a single SMS is as follows:

```
	Maximum number of characters in SMS content = 140 * 8 / E (E - number of bits representing a character data)
```

From the above, it can be seen that:

```
	GSM encoding, E = 7, maximum number of characters in SMS content = 140 * 8 /  7 = 160
	IRA encoding, E = 8, maximum number of characters in SMS content = 140 * 8 /  8 = 140
	UCS2 encoding, E = 16, maximum number of characters in SMS content = 140 * 8 /  16 = 70
```

Since QuecPython does not support the IRA encoding method, it currently only supports GSM and UCS2 encoding methods. GSM encoding SMS and UCS2 encoding SMS will be introduced below.

#### GSM SMS Encoding

GSM encoding, also known as GSM 7-bit encoding, uses 7 bits to represent a character. The characters entered on the keyboard are standard ASCII codes, and ASCII codes are 8-bit data. When actually sending data, only the character values corresponding to GSM encoding and ASCII encoding are the same, the data can be correctly received. Otherwise, it will cause inconsistent data between the sender and receiver. The value corresponding to the GSM code is not a simple discard of  the high bits of the ASCII code, and needs to be identified by yourself. The GSM 7-bit encoding table refers to 3GPP TS 03.38.

#### UCS2 SMS Encoding

UCS2 encoding is used to send Unicode characters, and a maximum of 70 characters can be sent in a single SMS. UCS2 encoding can be used to send Chinese, English, and characters from various countries. There are standard encoding rules for character values and UCS2 encoding corresponding values, which can be found in reference materials.

### Classification of SMS

SMS can be divided into TEXT SMS, PDU SMS, and block SMS. TEXT SMS is simple and easy to implement, but it may not meet the requirements in some scenarios. PDU SMS implementation is more complex and requires transcoding, but it can basically meet the requirements of the majority of scenarios. Block SMS is rarely used, and QuecPython currently does not support this mode, so it will not be further introduced.

#### TEXT SMS

When sending a TEXT SMS using the QuecPython interface, there is no need to split and combine the TEXT SMS content for sending. You only need to write the content directly. If the SMS content exceeds 140 bytes, the QuecPython module will automatically convert it into multiple SMS messages and send them (When the supported SMS content is marked as larger than 140 bytes in Table 3-1).

When sending SMS messages, the entered data are all ASCII codes, and ASCII codes are 8-bit character data. GSM encoding is 7-bit character data. In TEXT mode, GSM encoding data needs to ensure that the 7-bit data values are exactly the same as the values in the ASCII code table in order to send a  message data through the TEXT, otherwise it will cause inconsistency between the received and transmitted data.

### Classification and Structure of PDU Strings

For the message function, PDU types are divided into the following categories:

- SMS-DELIVER

  Sent from the message center to the mobile terminal device

- SMS-DELIVER-REPORT

  Sent from the mobile terminal device to the message center

- SMS-SUBMIT

  Sent from the mobile terminal device to the message center

- SMS-SUBMITE-REPORT

  Sent from the message center to the mobile terminal device

* SMS-STATUS-REPORT

  Sent from the message center to the mobile terminal device

* SMS-COMMAND

  Sent from the mobile terminal device to the message center

* Other Types

  Reserved for future use

For the Mobile Terminal Device, the focus is on SMS-SUBMIT and SMS-DELIVER. Their structures are as follows:

![](../media/network-comm/sms/2-6.png)

<center>Figure 2-3</center>

The common parts of the SMS-SUBMIT and SMS-DELIVER PDU string formats are introduced in 3GPP TS 03.40 Chapter 9. The important DCS (Data Coding Scheme) involves SMS compression, SMS types (four types in total, specifying the strategies for handling each type of SMS by the ME), SMS encoding types (GSM/UCS2), etc. For more information, please refer to Chapter 4 of GSM 03.38.

The following is a brief introduction to the SMS-SUBMIT and SMS-DELIVER types. For detailed content, please refer to the 3GPP TS 03.40 protocol document.

#### SUBMIT-PDU

The following figure shows the format of the PDU data for sending, and the data can be assembled in the following format. The meanings of the field values can be found in the 3GPP TS 03.40 protocol document. For the issue of user data packets in long SMS UD, please refer to Long SMS (UD) in Figure 2-7. This is already handled at the QuecPython underlying level and does not need to be concerned about.

![](../media/network-comm/sms/2-3.png)

<center>Figure 2-4</center>

##### PDU TYPE

For SMS-SUBMIT, the structure diagram of the PDU type is as follows:

![](../media/network-comm/sms/2-11.png)

<center>Figure 2-5</center>

 * TP-MTI, 2 bits
    * Indicates the SMS PDU type, such as SMS-SUBMIT, SMS-DELIVER, etc.
 * TP-RD, 1 bit
    * Indicates whether the SMS center should accept or reject the SMS-SUBMIT data when TP-MR and TP-DA are the same.
* TP-VPF, 2 bits
  * Validity period format
* TP-RP, 1 bit
  * Indicates whether the reply path is set
* TP-UDHI, 1 bit
  * Indicates whether the UD contains a header. For UD, a long SMS contains a header, while a single SMS only contains user data.
* TP-SRR, 1 bit
    * Indicates whether a status report is required

##### VP

VP represents the validity period of the SMS in the SMS center.

![](../media/network-comm/sms/2-4.png)

<center>Figure 2-6</center>

#### DELIVER-PDU

The following figure shows the format of the received PDU data, and the data can be parsed in the following format. The meanings of the field values can be found in the 3GPP TS 03.40 protocol document.

![](../media/network-comm/sms/2-5.png)

<center>Figure 2-7</center>

##### PDU TYPE

For SMS-DELIVER, the structure diagram of the PDU type is as follows:

![](../media/network-comm/sms/2-10.png)

<center>Figure 2-8</center>

* TP-MTI, 2 bits
  * Same as above
* TP-MMS, 1 bit
  * Indicates whether more SMS messages are waiting
* TP-RP, 1 bit
  * Same as above
* TP-UDHI, 1 bit
  * Same as above
* TP-SRI, 1 bit
  * Indicates whether the SMS status should be replied.

##### SCTS

* SMS center timestamp, the content format can be found in the 3GPP TS 03.40 protocol document.

## SMS Network Protocol Framework

The SMS network protocol framework is shown in the following figure:

![](../media/network-comm/sms/2-7.png)

<center>Figure 2-9</center>

| SM-AL               | Short Message Application Layer                              |
| -------------------- | ------------------------------------------------------------ |
| SM-TL               | Short Message Transfer Layer                                 |
| SM-RL               | Short Message Relay Layer                                    |
| SM-RP  SMR  CM-sub | Short Message Relay Protocol Short Message Relay (entity) Connection Management sublayer |
| SM-CP  SMC  MM-sub | Short Message Control Protocol Short Message Control (entity) Mobility Management sublayer |
| RR-sub              | Radio Resource Management sublayer                           |
| LLC-sub              | Logical Link Control sublayer                                |
| GRR-sub              | GPRS Radio Resource sublayer                                 |

### SMS Sending Process

After the user finishes editing the SMS, the SMS data is actively sent by the mobile terminal device, and the network layer replies with CP-ACK, followed by RP-ACK, and finally the mobile terminal device replies with CP-ACK.

As shown in the following figure:

![](../media/network-comm/sms/2-9.png)

<center>Figure 2-10</center>

### SMS Receiving Process

The mobile device first receives the SMS from the network side, and then the mobile device terminal replies with CP-ACK to the network. When the SMS parsing is completed, the mobile device terminal sends RP-ACK again, and the network layer replies with CP-ACK.

As shown in the following figure:

![](../media/network-comm/sms/2-8.png)

<center>Figure 2-11</center>

## Introduction to SMS Support Platforms

The storage location of SMS is generally divided into SM, ME, and MT storage.

* SM storage refers to storing the SMS on the SIM card after receiving it, without occupying device storage space.

* ME storage refers to storing the SMS in the device storage space after receiving it, without occupying SIM card space.

* MT storage refers to storing the SMS in both the SIM card and the device space. If one of the SIM card or device storage spaces is full, the subsequent SMS will be stored in the other storage space that is not full.

Currently, the QuecPython module only supports SM or ME storage, either one. For SMS quantity statistics and feature support, the default storage situation is shown in the table below.

<center>Table 3-1</center>
<table style="text-align: center">
        <tr>
            <td  style="background-color: red" rowspan="2">Model</td>
            <td  style="background-color: red" rowspan="2">Specific Model</td>
            <td  style="background-color: red" rowspan="2">Default Storage Location</td>
            <td  style="background-color: red" colspan="2">Maximum Storage Number</td>
            <td  style="background-color: red" colspan="2">Maximum SMS Content Length(Bytes)</td>
            <td  style="background-color: red">Remarks</td>
        </tr>
    	<tr>
            <td style="background-color: red">SM</td>
            <td style="background-color: red">ME</td>
            <td style="background-color: red">GSM</td>
            <td style="background-color: red">UCS2</td>
            <td style="background-color: red"></td>
        </tr>
        <tr >
            <td rowspan="2">EG915N<br>EG912N</td>
            <td >EG915N</td>
            <td >SM</td>
            <td >50</td>
            <td >180</td>
            <td >960</td>
            <td >420</td>
            <td ></td>
        </tr>
    	<tr>
            <td >EG912NEN_AA</td>
            <td >SM</td>
            <td >50</td>
            <td >180</td>
            <td >960</td>
            <td >420</td>
            <td ></td>
        </tr>
   		<tr >
            <td rowspan="1">EC200A</td>
            <td >EC200A</td>
            <td >ME</td>
            <td >50</td>
            <td >180</td>
            <td >960</td>
            <td >420</td>
            <td></td>
        </tr>
    <tr>
            <td rowspan="6">BG95<br>BG600</td>
	        <td >BG600LM3</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M1</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M2</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M3</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M8</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td >BG95M9</td>
            <td >ME</td>
            <td >50</td>
            <td >23</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
	        <td rowspan="4">EC200U<br>EG912U<br>EG915U</td>
        	<td >EC200U</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
        	<td >EG912U</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
    <tr>
        	<td >EG915U</td>
            <td >SM</td>
            <td >50</td>
            <td >100</td>
            <td >640</td>
            <td >280</td>
            <td></td>
    </tr>
</table>




## Application of SMS


SMS has great advantages in broadcasting and public information dissemination, and it is an important component of large-scale projects. The following focuses on the application of SMS. For esim cards, there is generally no SMS center code for downloading profiles. You need to first check if there is a SMS center code. If it does not exist, please consult the operator to obtain the correct SMS center code for setting.

### Register Callback Function

This method is used to register the SMS callback function. When the user's device receives an SMS, it will notify the current SMS storage location and the index value of the SMS storage location through the registered callback function. In the example below, if a long SMS is received and divided into two SMS messages, the callback function will be triggered twice.

The registration callback interface is as follows:

```python
sms.setCallback(usrFun)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

def sms_callback(args):
     print("Get SIM ID :{}".format(args[0]))					
     print("Get Message index :{}".format(args[1]))
     print("Get Message Storage :{}".format(args[2]))

sms.setCallback(sms_callback)

```

### SMS Center Code

The SMS center code is the number provided by the operator for SMS forwarding. The device sends the SMS to the SMS service center first, and the SMS service center forwards the received SMS to the base station where the destination device is located. The base station then sends the received SMS to the destination device. After setting the SMS center code, it is stored in the SIM card. It only needs to be set once for each SIM card, and the setting takes effect immediately. If you need to change it, you can set it again.

#### Get SMS Center Code

The interface for getting the SMS center code is as follows:

```python
sms.getCenterAddr()
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

center_numer=sms.getCenterAddr()
print("Current center number is :{}".format(center_numer))
```

#### Set SMS Center Code

When setting the SMS center code, make sure that the set SMS center code is correct, otherwise it may cause SMS sending failure. For SMS center codes in different countries or regions, please contact the local operator for confirmation.

The interface for setting the SMS center code is as follows:

```python
sms.setCenterAddr(addr)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

# Get the current SMS center code
center_numer=sms.getCenterAddr()
print("Current center number is :{}".format(center_numer))

# Set the SMS center code
center_number="8613010112512"
sms.setCenterAddr(center_number)

```


### SMS Storage Location

The SMS storage location is physically divided into SIM card and ME. Currently, only SIM card storage or ME storage is supported, and only one of them can be selected, they cannot be selected at the same time.

#### Get Storage Location

The SMS storage location is logically divided into read and delete storage area, send and write storage area, and receive storage area. The receive storage area, read and delete storage area should be consistent in order to read and delete the received SMS.

The interface for getting the SMS storage location is as follows:

```python
sms.getSaveLoc()
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

save_info= sms.getSaveLoc()
print("Message read and delete location : {}".format(save_info[0]))
print("Message write and send location  : {}".format(save_info[1]))
print("Message recv and save location   : {}".format(save_info[2]))
```

#### Set Storage Location

To receive, delete, and read SMS, you need to operate from the correct SMS storage location. You need to set the SMS storage location correctly in order to use the functions properly.

The interface for setting the SMS storage location is as follows:

```python
sms.setSaveLoc(mem1, mem2, mem3)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

# Get the current SMS storage location

save_info=sms.getSaveLoc()

# Set the storage location for reading and deleting SMS to SM
save_info[0][0]="SM"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

# Set the storage location for receiving and reading SMS to ME
save_info[2][0]="ME"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

```


### Send SMS

The QuecPython SMS sending interface is convenient to use. You only need to input the recipient's phone number, SMS content, and select the encoding method, and then call the interface directly. This feature is divided into two interfaces: sending TEXT messages and sending PDU messages. The interface style and parameters are the same. If the SMS content exceeds 140 bytes, the QuecPython underlying system will automatically convert it into a long SMS and send it in multiple messages (When the supported SMS content is marked as larger than 140 bytes in Table 3-1). The receiving end needs to read it in PDU format and parse the sequence number of each sub-message of this long SMS in order to merge them into one message and display it normally. Otherwise, this long SMS will be split into multiple messages on the receiving end.

#### Send TEXT Message

When sending a TEXT message, if the SMS content exceeds 140 bytes, it will be automatically split into multiple messages and sent as a long SMS (when the maximum number of supported SMS is greater than 1 in Table 3-1).

The interface is as follows:

```python
sms.sendTextMsg(phoneNumber, msg, codeMode)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

sms.sendTextMsg('18158626517', 'first test Message', 'GSM')

# TEXT message example 2, sending data through a variable
data="second test message"
sms.sendTextMsg('18158626517', data , 'GSM')

```

#### Send PDU Message

When sending a PDU message, if the SMS content length exceeds 140 bytes, it will be automatically split into multiple messages and sent as a long SMS (When the supported SMS content is marked as larger than 140 bytes in Table 3-1).

The interface is as follows:

```python
sms.sendPduMsg(phoneNumber, msg, codeMode)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

# PDU SMS Example 1, directly fill in the data to be sent
sms.sendPduMsg('18158626517', 'send pdu msg by GSM mode.', 'GSM')

# PDU SMS Example 2, send data through variables
data="second test pdu message"
sms.sendPduMsg('18158626517', data, 'GSM')
```

### Get SMS Count

In actual use, it is necessary to query the current number of SMS to determine whether the current SMS storage space is full. This interface query needs to pay attention to the current set SMS storage space, and the returned SMS count is the SMS count of the current SMS storage space.

The interface for obtaining the current number of received SMS is as follows:

```python
sms.getMsgNums()
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms


# Get current SMS storage information
save_info=sms.getSaveLoc()

# Get the number of SMS in the SM storage area
sms.setSaveLoc("SM","SM","SM")
print("Current save info : {}".format(save_info))

# Get the number of SMS in the current SMS storage location
message_count=sms.getMsgNums()
print("Current message count :{}".format(message_count))

# Get the number of SMS in the ME storage area
sms.setSaveLoc("SM","SM","ME")
message_count=sms.getMsgNums()
print("Current message count :{}".format(message_count))
```

### Read SMS

When the user receives a SMS notification, the information can be read through the following two interfaces. It is divided into reading SMS in TEXT mode and PDU mode. When receiving long SMS, reading SMS in TEXT mode cannot distinguish the order, so PDU mode should be used to read SMS, and the received PDU data format should be parsed and merged according to the SMS reception PDU data format. There is a demonstration in the complete application example code.

#### Read TEXT SMS

Read SMS content in TEXT mode:

```python
sms.searchTextMsg(index)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

message_count = sms.getMsgNums()
if message_count > 0:
    message_info = sms.searchTextMsg(0)
    print("Get Messge index 0 info :{}".format(message_info))
```

#### Read PDU SMS

Read SMS content in PDU mode:

```python
sms.searchPduMsg(index)
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

message_count = sms.getMsgNums()
if message_count > 0:
    pdu_message = sms.searchPduMsg(0)
    # Read PDU SMS, need to decode
    pdu_len = sms.getPduLength(pdu_message)
    message_info = sms.decodePdu(pdu_message, pdu_len)
    print("Get Messge index 0 info :{}".format(message_info))
```

### Delete SMS

When the SMS storage space is full, it may cause the device to not receive SMS. Users can delete some SMS and release the SMS storage space through the following interface.

#### Delete specified position SMS

The interface for deleting SMS is as follows:

```pyhton
sms.deleteMsg(index [, delmode])
```

Example

```python
# -*- coding: UTF-8 -*-
# Example
import sms

#-------Get current SMS storage information to ensure that the deletion of SMS position will not go wrong
save_info=sms.getSaveLoc()
print("Will delete message from {} filed!".format(save_info[0][0]))

# Delete SMS with index 0
sms.deleteMsg(0)

#---------Specify to delete the specified index SMS data in the SMS storage area, taking ME storage area as an example-------
# Step 1, set the operation SMS storage area to ME
save_info[0][0]="ME"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

# Step 2, set the index value of the SMS to be operated, taking index value 0 as an example
index  = 0
sms.deleteMsg(index)

```

#### Delete all SMS

The interface is the same as above, only the following example is given.

```python
# -*- coding: UTF-8 -*-
# Example
import sms

#-------Get current SMS storage information to ensure that the deletion of SMS position will not go wrong
save_info=sms.getSaveLoc()
print("Will delete all message from {} filed!".format(save_info[0][0]))

# Delete all SMS
sms.deleteMsg(1,4)

#---------Specify to delete all SMS data in the SMS storage area, taking ME storage area as an example-------
# Step 1, set the operation SMS storage area to ME
save_info[0][0]="ME"
sms.setSaveLoc(save_info[0][0],save_info[1][0],save_info[2][0])

# Step 2, delete all received SMS in the ME storage area
index  = 0
sms.deleteMsg(1,4)
```

## Application Example

Long SMS merge example.

```python
# codeing=UTF-8
import net
import checkNet
import sms as SMS

# This example demonstrates merging two long SMS in the device into one SMS
# This example requires that the messages with index value 0 and index value 1 are the contents of the same long message.
    
class QuecSMS():

    def __init__(self):
        self.__enable_log = False
        self.sms_set_enable_log()
        self.sms = SMS
        self.sms.setCallback(self.__SMS_Call_back)
        self.message=""
        
    def sms_deal_phone_number(self,args):
        data=enumerate(args)
        data_str1=""
        data_str2=""
        data_str3=""
        for index,value in data:
            if index % 2 == 0:
                data_str1=data_str1+value
            else:
                data_str2=data_str2+value
        for i in range(0,len(data_str1)):
            data_str3=data_str3+data_str2[i]
            data_str3=data_str3+data_str1[i]
        return data_str3

    def sms_display(self):
        self.__log("Get Message Loction : {}".format(self.sms.getSaveLoc()))
        sms.setSaveLoc("ME", "ME", "ME")
        self.__log("Get ME Message Numbers : {}".format(self.sms.getMsgNums()))
        sms.setSaveLoc("SM", "SM", "SM")
        self.__log("Get SM  Message Numbers : {}".format(self.sms.getMsgNums()))

    def sms_set_enable_log(self,flag=True):
        self.__enable_log = flag

    def __log(self,args):
        if self.__enable_log:
            print("QuecSMS_LOG: {}".format(args))

        
    def __SMS_Call_back(self,args):
        self.__log("Get SIM Id          : {}".format(args[0]))
        self.__log("Get Message index   : {}".format(args[1]))
        self.__log("Get Message Storage : {}".format(args[2]))
        
    def sms_delete_user_data_head(self):
        
        self.message=self.message[:58]+self.message[58+2*6:]
        
    def sms_replace_data_index(self,index,data):
        self.message = self.message[:index] + data + self.message[index+len(data):]
    
    def sms_append_sub_message_data(self,data):
        self.message = self.message + data
        
    def sms_decode_pdu_message(self):
        return self.sms.decodePdu(self.message,self.sms.getPduLength(self.message))
    
    def sms_get_message_info(self,index):
        """
        ：param index    ：SMS index value
        ：tyoe  index    ：integer
        
        ：return         ：tuple type
        Tuple content：
        （message_ref，message_total_num，message_seq，sub_message_data，pdu_tye，sub_message_len）
        
        ：message_ref        ：SMS reference identifier, the same identifier indicates the same SMS
        ：message_total_num  ：The total number of long messages for this SMS
        ：message_seq        ：The sequence number of this message in the long message
        ：sub_message_data   ：The content of this message
        ：pdu_tye            ：PDU type, bit 6 indicates whether the user message header is included, required for long messages
        ：sub_message_len    ：The length of the content of this message
        
        """
        message0=self.sms.searchPduMsg(index)
        self.message = message0
        sca_num  = int(message0[0:2],16) - 1
        data_len = 2
        addr_type= message0[data_len:data_len+1*2]
        data_len = data_len+1*2

        sca      = message0[data_len:data_len+sca_num*2]
        # The high and low bits of the phone number need to be converted.
        sca = self.sms_deal_phone_number(sca)

        if sca_num % 2 == 1:
             print("Get SCA phone {}".format(sca[:-1]))
        else:
             print("Get SCA phone {}".format(sca))

        data_len = data_len+sca_num*2

        pdu_tye   = int(message0[data_len:data_len+1*2],16)
        data_len = data_len+1*2
        
        oa_num   = int(message0[data_len:data_len+1*2],16)
        data_len = data_len+1*2
        oa_addr_type   = message0[data_len:data_len+1*2]
        data_len = data_len+1*2

        if oa_num % 2 == 1:
             oa_num_t   = int((oa_num + 1)/2)
        else:
             oa_num_t   = int((oa_num )/2)

        oa       = message0[data_len:data_len+oa_num_t*2]
        # The high and low bits of the phone number need to be converted.
        oa       = self.sms_deal_phone_number(oa)

        if oa_num % 2 == 1:
             print("Get OA phone {}".format(oa[:-1]))
        else:
             print("get OA phone {}".format(oa))

        data_len = data_len+oa_num_t*2
        pid      = message0[data_len:data_len+1*2]
        
        data_len = data_len+1*2
        dcs      = message0[data_len:data_len+1*2]
        data_len = data_len+1*2
        scts     = message0[data_len:data_len+7*2]
        data_len = data_len+7*2
        sub_message_len = int(message0[data_len:data_len+1*2],16)
        data_len = data_len+1*2
       

        #Long SMS
        if ((pdu_tye & 0x40 ) >> 6) == 1:
                
                sub_message_head = message0[data_len:data_len+6*2]
                data_len = data_len+6*2
                
                # Long SMS user data
                sub_message_data = message0[data_len:data_len+sub_message_len*2]
                # Long SMS, process the header of the long SMS user message, get IEI to determine the type of SMS, refer to 3GPP TS 03.40 TP-user-data
                sub_head_iei         = int(sub_message_head[2:4],16)
                if sub_head_iei == 00:
                    # Long SMS
                    message_ref = int(sub_message_head[6:8],16)
                    message_total_num = int(sub_message_head[8:10],16)
                    message_seq = int(sub_message_head[10:12],16)
                    return message_ref,message_total_num,message_seq,pdu_tye,sub_message_len,sub_message_data
                else:
                    # Other types of SMS
                    return 0,0,1,pdu_tye,sub_message_len,sub_message_data
        else:
             # Short SMS
             sub_message_data = message0[data_len:data_len+sub_message_len*2]
             return 0,1,1,pdu_tye,sub_message_len,sub_message_data

if __name__ == '__main__':
    sms0 = QuecSMS()
    sms1 = QuecSMS()

    # Read the content of message index 0
    message0=list(sms0.sms_get_message_info(0))

    # Read the content of message index 1
    message1=list(sms1.sms_get_message_info(1))
    print("Get Messge 0 {}".format(message0))
    print("Get Messge 1 {}".format(message1))
    if message0[0] != message1[0]:
        print("Message 0 and Message 1 are not the content data of the same long SMS!")
        exit
    # Decode the PDU string of message0
    message0_pdu=list(sms0.sms_decode_pdu_message())
    # Decode the PDU string of message1
    message1_pdu=list(sms1.sms_decode_pdu_message())

    # Merge messages
    # Determine which message, message 0 or message 1, has a lower message sequence number
    if message0[2] < message1[2]:
        message_merge=message0_pdu[1]+message1_pdu[1]
    else:
        message_merge=message1_pdu[1]+message0_pdu[1]
    print("Get Merger Message is {}".format(message_merge))

```

## Common Applications of SMS

### Functional Applications

* Application Scenario 1

When applying for a new SIM card, it is uncertain whether the message center code is already stored in the SIM card. It is necessary to first determine if the message center code exists. If it does not exist, please consult the corresponding operator to obtain the message center code for setting. Otherwise, the SMS function will not work properly.

* Application Scenario 2

When changing the operator of the SIM card, it is necessary to confirm the message center code with the original operator and the new operator for resetting, otherwise the SMS function will not work properly.

* Application Scenario 3

For eSIM cards, the message center code may not be preset when downloading the profile. In this case, it is necessary to confirm the message center code with the operator.

* Application Scenario 4

For eSIM cards, changing the profile will clear the message center code, and it needs to be reset.

* Application Scenario 5

When both the SM and ME message storage spaces are full, the SMS cannot be received. Only by clearing the messages and freeing up the message storage space can the SMS be received normally.

* Application Scenario 6

If a message is received but not read, and the message storage space for reading and deleting is changed, the message cannot be read. Please change back to the message storage space when the message was received to read and delete the message.

* Application Scenario 7

If the message storage space is SM and the SM storage space is full, but you want to keep the messages and ensure that new messages can be received, you can change the message receiving storage space to ME. If the ME storage space is also full, only by deleting messages can new messages be received.

* Application Scenario 8

If the device cannot receive messages in an area without signal, please move the device to an area within the operator's signal coverage to enable normal SMS sending and receiving operations.

* Application Scenario 9

If the device has been out of the operator's service area for a long time and then returns to the service area, but the message center no longer forwards messages due to the expiration of the message storage time, resulting in message loss. This situation should be avoided as much as possible. Currently, the validity period for sending messages in QuecPython is the longest. As for the maximum validity period for message storage set by operators, it depends on the settings of each operator and can be confirmed with the operator.

### Business Application Scenarios

With the continuous progress of technology, the cost of sending SMS messages is getting lower, and its application in projects is becoming more and more extensive. Common application scenarios include:

* User registration and login

  Nowadays, more and more service applications use mobile phone numbers for user registration and login, such as various banking mobile applications, e-commerce mobile applications, and various types of entertainment mobile applications.

* Identity verification

  In practical applications, mobile phone numbers are often closely linked to personal real-name information, which makes it easy to verify identities in actual business operations. Various applications also use this feature for identity verification.

* Security verification

  In confirmation verification scenarios such as password recovery in mobile applications and payment confirmation, SMS messages are not easily stolen, so they play a very important role in secure payment services and shine in such business operations.

* Notification and risk warning

  In financial services such as credit cards and debit cards, when financial service providers detect risky transactions or fraudulent transactions in risk control, they can use SMS messages to conveniently remind users to handle risks and recover losses in a timely manner.

* Application promotion

  When a company establishes a new project or business, SMS messages can be used to promote the project to users.

* Password recovery

  Nowadays, both personal computer applications and mobile applications are very rich, and there are more and more application accounts. Users easily forget their passwords. In order to facilitate customers and simplify system security design, using SMS messages for password recovery is a very convenient method.

The above common scenarios are closely related to people's lives and are very common. In addition to these, public service applications often use SMS messages to notify citizens to guard against fraud and provide various public service notifications and warnings.

## Frequently Asked Questions

**Question 1: Does QuecPython support long SMS messages? If so, what is the maximum length of an SMS message?**

QuecPython has many types of modules, some of which support long SMS messages, while others only support single SMS messages. For specific information, please refer to Table 3-1.

**Question 2: How does QuecPython handle the sending of long SMS messages? Are they sent in multiple packets or as a single message?**

QuecPython will split long SMS messages and send them one by one.

**Question 3: How does QuecPython handle received long SMS messages? Are they stored as separate messages or automatically merged into one message?**

QuecPython will store received long SMS messages separately. In this case, you need to read the messages in PDU format and extract the message reference number, total number of messages, and current message index from the user data header information in the PDU encoding for further processing.

**Question 4: The sender shows that the SMS message has been sent successfully, but the module does not receive the message. What could be the reason?**

There could be several possible reasons for this situation. Users can troubleshoot according to the following situations:

* Determine whether the module has not received the message or the message is not displayed due to decoding failure.

* If the message has been sent successfully, it means that the module has successfully sent the message data through the base station. If there is an exception during the propagation process to the receiving end, you need to troubleshoot the issue with the operator, base station, and message service center.

* For the situation where the message is not displayed due to decoding failure, check if the decoding settings are incorrect.

* Check if the message center service code is set incorrectly, which leads to the failure of message reception.

  In this case, consult the local operator to confirm the correct message center service code and set it if it is incorrect.

* Check if the storage space on the receiving end is full.

  In this case, check if the current message storage space is full.

* Check if the receiving end has mistakenly set the sender to the blacklist.

  In this case, check if the device has mistakenly set the sender to the blacklist.

* Check if the SIM card on the receiving end has been deactivated or has outstanding fees.

  In this case, communicate with the operator to check the current status of the device.

* Check if the receiving end has been out of the operator's service area for a long time, exceeding the time limit for the message service center to cache messages.

  In this case, move to the operator's service area.

* Check if the receiving end has enabled the SMS service.

  In this case, consult the operator to see if the SMS service has been enabled.

* Check if the message receiving storage location and the message reading storage location on the receiving end are different, resulting in the inability to read messages from the reading location.

  In this case, set the message receiving storage location and the message reading storage location to the same storage location.

**Question 5: After inserting the SIM card into the module, it can connect to the network normally, but why does the SMS sending fail?**

There could be several reasons for the failure to send SMS messages. Some common situations include:

* Check if the SMS service has been enabled by the operator. If the SMS service has not been enabled, please enable it.
* Check if the SMS service center code is correct. If the SMS center code is not set or set incorrectly, please consult the operator for the correct SMS center code for setting.

- Check if  the operator's SMS service is normal, you can confirm with the operator.
- Check if  the network signal is stable, log analysis is needed for judgment, please capture the log and provide it to the QuecPython team for analysis.
- Check if  the network status is in the process of changing from active to idle, or from idle to active when sending a SMS, log analysis is needed for judgment, please capture the log and provide it to the QuecPython team for analysis.
