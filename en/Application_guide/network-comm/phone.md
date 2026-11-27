# Voice Call

This document introduces LTE voice call solutions, common terms, QuecPython's support for voice calls on various platforms, and the application of voice calls. 

## LTE Voice Call

LTE (Long-Term Evolution) is a 4G wireless communication technology that provides high-speed data transmission and high-quality voice call capabilities. Different voice call solutions can be used on LTE networks, such as CSFB-based voice calls and VoLTE-based voice calls.

### CS-based Voice Call Solution

Circuit Switched Fall Back is a mechanism that sends terminals that do not support IMS services in areas covered by both LTE and 2G/3G networks back to 2G/3G  where it can successfully complete a CS Voice Call to maintain data service continuity.

In the CSFB solution, LTE terminals usually camp in the LTE network. When a voice call service is required, the network assists the terminal in falling back to the CS domain of the 2G/3G network. After the call ends, the terminal returns to the LTE network.

In addition to CS voice calls, the CSFB solution also supports other CS services, such as SMS.

#### Background

1. There is no dedicated circuit-switched domain for voice call services in the LTE network, so voice services cannot be realized when IMS (terminal/network) is not fully developed. However, voice services are an important business and source of revenue for operators, so voice call services must be supported in LTE networks.
2. In the early stages of LTE development, there were still a large number of 2G/3G networks. By using the existing voice call services in the CS domain to provide voice services, operators can solve the problem of voice call services in LTE networks with minimal changes and investments.

#### CSFB Implementation

##### Prerequisites

1. For the module:
   - The UE needs to perform the combined EPS/IMSI attach procedure during attachment.
   - Support NAS layer triggering extended service requests.
   - Support 2G or 3G.

2. For the network:
   - Support NAS layer triggering extended service requests.
   - Support SGs interface. (SGs is a functional interface between MME and MSC, and it is essential for CSFB and SGs SMS. The signaling message transmission on the SGs interface is based on a reliable IP communication protocol layer.)

##### CSFB Flow

![CSFB_Flow_chart](../media/network-comm/phone/CSFB_Flow_chart.png)

The CSFB-based voice call solution can meet the requirements of mobile communication operators to provide voice communication in LTE networks and ensure the reliability of calls.

### VoLTE-based Voice Call Solution

VoLTE (Voice over LTE) is a high-definition voice communication technology based on 4G networks. Compared to traditional CSFB technology, it provides better voice quality, faster call connection speed, and fewer call interruptions, thus better meeting users' needs for voice communication.

The VoLTE-based voice call solution is a way to implement voice communication on 4G networks. It uses IP networks for voice transmission without the need to convert to analog signals like traditional circuit-switched networks, enabling simultaneous high-definition voice calls and data transmission. With the VoLTE-based voice call solution, users can maintain high-quality voice calls and data transmission simultaneously, while also reserving more network bandwidth to make the entire network more stable and efficient.

Although implementing the VoLTE-based voice call solution requires more network construction and equipment upgrades, with the popularity of 4G networks and the arrival of 5G networks, more and more communication operators are adopting VoLTE technology to improve user call experience and network efficiency.

## Common Terms

This section explains common terms related to voice communication, including DTMF, IMS, and VoLTE.

### DTMF
#### What is DTMF

DTMF is an abbreviation for "Dual Tone Multi Frequency" and is a multi-frequency signal technology used in telephony systems. It uses tones composed of two-frequency signals to identify the numbers, letters, or other special characters entered by telephone users. DTMF is used in various telephony scenarios, including telephone keypad input and remote control. DTMF is commonly used to identify commands entered by telephone users, such as checking group calls, call forwarding, and call termination. It can also be used to call automated service systems, such as bank interactive systems, where users can achieve specific functions by entering keys using DTMF.

#### Basic Principles and Encoding of DTMF

Basic Principles

The basic principle of DTMF is to use a combination of two frequency signals to represent the numbers, letters, or special characters entered on the telephone keypad. Typically, a combination of low-frequency and high-frequency signals represent the numbers 0 to 9, as well as special characters such as asterisk (*) and pound (#). Each number and special character corresponds to a fixed frequency combination. For example, the frequency combination for the number "1" is 697 Hz and 1209 Hz.

When a key is pressed on the telephone keypad, the calling terminal sends out a signal composed of two frequencies. This signal is transmitted to the receiving terminal, which can determine the number or special character entered by the user by analyzing this signal. According to the DTMF encoding method, each number or special character has a unique frequency combination, making DTMF an effective way to identify numbers and special characters.

Encoding Method

The DTMF encoding method selects different frequencies in the low-frequency and high-frequency signals to encode the numbers, letters, or special characters into corresponding signal sequences. DTMF signals are encoded using a set of eight frequencies of sine waves, with four frequencies used to represent the low-frequency part and another four frequencies used to represent the high-frequency part. The following is a frequency encoding table for DTMF:

| High/Low Frequency | 1209 Hz | 1336 Hz | 1477 Hz | 1633 Hz |
| ----------------- | ------ | ------ | ------ | ------ |
| 697 Hz            | 1      | 2      | 3      | A      |
| 770 Hz            | 4      | 5      | 6      | B      |
| 852 Hz            | 7      | 8      | 9      | C      |
| 941 Hz            | *      | 0      | #      | D      |

For example, when the number "8" is entered, signals with frequencies of 852 Hz and 1336 Hz are sent. By using different frequency combinations, different numbers, letters, or special characters can be encoded. Through this encoding method, the telephone system can accurately identify user commands and provide corresponding feedback.

#### The Role and Applications of DTMF in Telephony

The role of DTMF signals is to identify the numeric keys pressed by telephone users, thereby achieving functions such as dialing, call forwarding, and voicemail. For example, when users dial a landline number on a telephone, they press numeric keys, and the telephone sends out corresponding DTMF signals to dial the number and connect the call to the terminal device of the dialed number.

DTMF is usually used to identify commands entered by telephone users, such as checking group calls, call forwarding, and call termination. Specifically, the applications of DTMF in telephony include: 

1. Making Phone Calls: Users can use DTMF to dial the target phone number. Once DTMF signals are input to the telephone exchange, a connection is established with the target phone.

2. Automated Service System: DTMF can be used to access automated service systems, such as bank automated voice interactive systems. By pressing the DTMF keys during a call, users can select desired services.

3. Remote Control: DTMF can also be used to send control signals to remote devices, such as cranes and water pumps, through a telephone keypad. Specific DTMF commands can be sent to a command control center to control remote devices.

4. Conference Call: DTMF is also used in conference calls, such as initiating and ending conferences, and muting and unmuting participants.

### IMS

#### What is IMS

IMS (IP Multimedia Subsystem) is an IP-based multimedia service subsystem architecture that enables the integration of voice, video, data and multimedia content delivery over telecom networks.

IMS architecture combines traditional telephone networks with IP networks, utilizing SIP (Session Initiation Protocol) as the control signal to provide services across various networks. Furthermore, IMS can provide services on different terminals such as PCs, handheld devices, or VoIP terminals.

The key functionalities provided by IMS include:

1. Creating and maintaining multimedia sessions, such as voice calls, video conferences, and messaging.
2. Offering services, such as file sharing, instant messaging, social networks, and email.
3. Providing user data management, such as profiles, address books, and network identification.
4. Supporting QoS (Quality of Service) mechanisms to ensure service quality and performance of multimedia services.
5. Supporting security and authentication mechanisms to ensure the confidentiality and integrity of user data.

The emergence of IMS is very significant because it enables telecommunications to expand from traditional voice communication into areas like data and multimedia communication. It also allows operators to build entirely new business models and value-added services through the IMS platform, thus strengthening their business competitiveness.

#### Basic Principles of IMS

The basic principle of IMS is to transform traditional telephone networks by using IP networks and SIP protocol, thereby achieving integrated transmission and interaction of voice, video and data.

In IMS, all services and applications are implemented based on the SIP and HTTP protocols. When users require services provided by IMS, they need to initiate an IMS session on their terminal devices. The process is as follows:

1. Users start an application on IMS terminal devices (e.g. smartphones) and obtain services from the IMS message servers to register with the IMS servers.
2. IMS servers identify users and establish sessions with them.
3. Users can engage in multimedia communication using IMS terminal devices and access services provided by IMS, such as email, messaging, and file sharing.

The basic architecture of IMS includes the call control layer, multimedia control layer, multimedia processing layer, and multimedia transport layer.

- Call Control Layer: Coordinate interactions and communication among multiple entities within the IMS system, including user devices, session creation, and control signaling.
- Multimedia Control Layer: Generate and manage multimedia streams, controlling multimedia codecs and handling media transformations such as transcoding and mixing.
- Multimedia Processing Layer: Handles multimedia encoding, decoding, compression, decomposition, and parsing, ensuring the quality and fault tolerance of multimedia stream transmission.
- Multimedia Transport Layer: Transfer data packets, such as UDP or TCP.

IMS also supports QoS mechanisms to ensure the quality and performance of multimedia services. At the same time, it supports security and authentication mechanisms to ensure the confidentiality and integrity of user data.

#### Relationship Between IMS and Telephony

IMS can be seen as a solution that integrates traditional telephone networks with modern communication technologies. In addition to retaining traditional telephony functions, IMS also provides additional multimedia and data transmission capabilities.

Traditional telephony functions include voice calls and SMS, whereas IMS combines traditional telephone services with video call services, allowing users to make video calls on their mobile phones. Additionally, IMS can offer services such as voicemail and instant messaging.

Compared to traditional telephony, IMS offers more abundant, flexible, and diverse communication services and solutions, enabling users to achieve a complete transformation from voice communication to multimedia communication.

Of course, telephony functions remain a very important part of IMS. In addition to retaining traditional telephony functions, it adds more data and multimedia communication capabilities, providing users with more choices.

![IMS](../media/network-comm/phone/IMS.png)

### VoLTE

#### What is VoLTE

VoLTE (Voice over LTE) is a voice communication service based on LTE networks. It offers high-quality, high-performance voice call services while retaining IMS features like multimedia transmission, instant messaging, and network identification to achieve the integration of voice and data.

VoLTE technology is implemented based on the IMS architecture. It employs the SIP protocol for call control and uses high-quality techniques like AMR (Adaptive Multi-Rate) codecs to provide high-fidelity voice services. Compared to traditional communication technologies like GSM and CDMA, VoLTE provides more stable and high-quality voice call services.

Traditionally, voice calls were transmitted over 2G or 3G networks, while LTE networks were primarily used for data transmission. The emergence of VoLTE enables voice calls to be directly transmitted over LTE networks, without downgrading to 2G or 3G networks.

#### Basic Principles of VoLTE

The fundamental principle of VoLTE involves forwarding voice signals over IP networks, integrating traditional separate voice and data services. In VoLTE, voice data can be directly transmitted over IP networks without conversion and encoding/decoding processes. Additionally, VoLTE supports high-quality voice technologies like HD Audio, making calls clearer and more natural.

When users make phone calls, VoLTE uses the SIP protocol to communicate with the IMS network and utilizes Quality of Service (QoS) technology to ensure the priority and stability of voice services. VoLTE also supports features such as multi-party calls and call rejection to meet diverse user needs.

VoLTE is built on the IMS network, where IMS serves as an enhanced version of Circuit Switching (CS). Consequently, on top of the LTE foundation, IMS replaces CS, enabling phone users to make voice calls without interrupting data services. The addition of IMS essentially establishes a dedicated channel for voice services alongside the use of LTE data channels, resulting in the VoLTE solution.

#### Advantages of VoLTE

With the use of VoLTE technology, voice calls can be made over 4G LTE networks, achieving simultaneous voice and data transmission.

Compared to traditional voice communication technologies, VoLTE has the following advantages:

1. High-definition Voice: VoLTE employs high-quality encoding and decoding techniques to provide high-quality and high-definition voice call services.
2. Faster Connection Speed: VoLTE is based on LTE networks, with faster connection speed and shorter call setup times. It also supports seamless handover and high-quality calls.
3. Multimedia Communication: VoLTE supports multimedia and data services, including video calls, SMS, instant messaging, and file sharing.
4. Better Coverage: As VoLTE is built on LTE networks, it has significant advantages in terms of network coverage and indoor connectivity.

Overall, VoLTE technology enhances call quality and user experience while also improving network utilization.

#### Relationship Between VoLTE and IMS

VoLTE is an important application scenario of the IMS. IMS handles calls and establishes voice calls through SIP signals. VoLTE technology adopts the IMS architecture and uses the SIP protocol to implement call control and call session management.

There is a close relationship between VoLTE and IMS. VoLTE is based on IMS (IP Multimedia Subsystem) technology, requiring operators' support for VoLTE technology and 4G network compatibility. IMS, in turn, provides the multimedia voice and video services required by VoLTE.

IMS is an open, standardized, modular IP-based multimedia service subsystem and is one of the core technologies of 4G mobile networks. It enables real-time multimedia services such as voice, video, chat, games, and file transfers to be transmitted over IP networks and interconnectivity between various applications.

VoLTE, based on IMS technology, achieves high-quality voice calls through the voice and video services of IMS, and also facilitates high-speed data transmission. The multimedia control and processing capabilities provided by IMS optimize the quality of VoLTE calls and offer an efficient way for voice communication. Additionally, VoLTE's IMS-based technology takes full advantage of the high-speed data transmission capabilities of 4G networks, breaking the limitation that voice communication can only use 2G or 3G networks.

The primary function of VoLTE is to provide high-definition voice calls based on the IMS architecture, utilizing IMS capabilities such as session control, quality assurance, security, and multimedia processing.

In summary, VoLTE achieves high-definition voice call services based on the IMS architecture, supports multimedia and data services, and greatly enhances communication service quality and user experience.

## Voice Call Features on QuecPython Modules
voiceCall is the voice call feature of QuecPython, which refers to the ability to make phone calls. The table below shows the supported voice call features of QuecPython and the support status of these features on various platforms.

| Module Model<br />Feature   | EG912N-EN <br />EG915N-EU<br /> | EC200A-AU<br />EC200A-EU<br /> | EC200U-AU<br />EC200U-EU | EC600U-EU<br />EC600U-EC<br />EC600U-LA<br> | EG912U-GL<br>EG915U-EU<br>EG915U-LA<br> |
| --------------------------- | ------------------------------- | ------------------------------ | ------------------------ | ------------------------------------------- | --------------------------------------- |
| Set auto-answer time        | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Make phone calls            | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Answer phone calls          | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Hang up phone calls         | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Register callback functions | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Auto hang up incoming calls | ❌                               | ✔                              | ❌                        | ❌                                           | ❌                                       |
| Set DTMF                    | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Recognize DTMF              | ✔                               | ❌                              | ❌                        | ❌                                           | ❌                                       |
| Call forwarding             | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Switch voice channel        | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Adjust volume               | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |
| Auto record                 | ✔                               | ✔                              | ✔                        | ✔                                           | ✔                                       |

 ✔: Supported; ❌: Not supported

>* BC25/EC600G/EC800G/BG95 series modules do not support voice call features.
>* EC600M series module support voice call only when the firmware supports VOLTE.
>* Only customized versions of other series modules support voice call features.

Please note that the above information may change with module updates and new releases. It is recommended to carefully read [wiki-Cellular communication-voiceCall](../../api_reference/iotlib/voiceCall.md) and confirm with FAEs before using these features.

## Voice Call Application

The voice call features in QuecPython can achieve basic call functions, DTMF and call forwarding. The use of these functions is described below.

The following scenario demonstrates voice calls between QuecPython and a mobile phone over VoLTE using the voice call feature. In the following operations, the operations on the module are based on *QPYcom*. First, open *QPYcom* and select the interactive port of the module to enter the following interface:

![QPYcom_voiceCall_en](../media/network-comm/phone/QPYcom_voiceCall_en.png)

To view all events in the voice call process, such as incoming calls, call connections and hang-ups, the callback functions are registered in advance, as shown in *4.4 Callback Function Registration for Voice Calls*.

In the following examples, "xxxxxxxxxxx" represents a phone number. During actual testing, the actual phone number will be displayed.

### Basic Call Features

#### 	Call and Answer

The module makes a call to the mobile phone, and the mobile phone answers and then hangs up.

```python 
>>> import voiceCall
>>> voiceCall.callStart('xxxxxxxxxxx') # The module makes a call to the mobile phone
0
>>> 
args:(14, 1, 0, 3, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the call is ongoing, the callback function of ongoing call is called

args:(11, 1, 0, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the mobile phone answers the call, the callback function of call connection is called

args:(12, 1, 0, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the mobile phone hangs up the call, the callback function of call hangup is called
```

If you make a call to the module using a mobile phone, when you see the callback functions mentioned above called and the incoming phone number displayed, use the following interface to answer the call:

```python
>>> import voiceCall
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # When a call comes in, the callback function of incoming call is called

>>> voiceCall.callAnswer() # Answer the call

0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module answers the call, the callback function of call connection is called
```

#### 	Auto Answer

The mobile phone calls the module, and the module automatically answers and then hangs up.

Enable auto answer.

```python
>>> import voiceCall
>>> voiceCall.setAutoAnswer(3) # Enable auto answer after receiving a call for 3 seconds
0
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # When a call comes in, the callback function of incoming call is called

args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module auto answers the call, the callback function of auto answer is called
```

#### 	4.1.3 Hang Up Call

If you make a call to the module using a mobile phone, when you see the callback functions mentioned above called and the incoming phone number displayed, use the following interface to answer the call and then hang up.

```python
>>> import voiceCall
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # When a call comes in, the callback function of incoming call is called

>>> voiceCall.callAnswer() # Answer the call

0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module answers the call, the callback function of call connection is called

>>> voiceCall.callEnd() # Hang up the call

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module hangs up the call, the callback function of call hangup is called
0
>>> 
```

#### 	Auto Hang Up

If you make a call to the module using a mobile phone, when the module is set not to hang up automatically, the callback function of incoming call will be called normally. When the module is set to hang up automatically, no callback function will be called and there will be no response on the module. Meanwhile, you will hear that the call has been connected and then quickly hung up on your phone.

```python
>>> import voiceCall
# The mobile phone calls the module, and the module does not hang up automatically by default
>>> voiceCall.getAutoCancelStatus()
0

# Enable auto hangup. When the mobile phone calls the module, the module hangs up automatically by default
>>> voiceCall.setAutoCancel(1)
0
>>> voiceCall.getAutoCancelStatus()
1
```

### DTMF

The voice call feature in QuecPython can set and receive DTMF. The implementation of DTMF is described below.

#### 	Set DTMF Configuration

Call `voiceCall.startDtmf(dtmf, duration)` to set DTMF.

`dtmf`: DTMF string. Valid characters: `0-9, A, B, C, D, *` and `#`.

`duration`: Time duration. Unit: milliseconds.

This feature is valid only during a voice call.

```python
>>> import voiceCall
>>> voiceCall.startDtmf('8',100)  # Equivalent to pressing "8" on the phone keypad
0
```

#### 	Set DTMF Recognition

Call `voiceCall.dtmfDetEnable(enable)` to enable or disable DTMF recognition. The DTMF recognition is disabled by default. When the DTMF recognition is enabled, you can view the recognized DTMF string by registering the function callback of DTMF recognition.

```python
>>> import voiceCall
>>> voiceCall.dtmfDetEnable(0)  # Disable DTMF recognition

>>> voiceCall.dtmfDetEnable(1)  # Enable DTMF recognition
```

#### 	Call DTMF Recognition Back

Call `voiceCall.dtmfSetCb(dtmfFun)` to register the callback function of DTMF recognition, which is used to recognize the DTMF string entered by the other party.

The module calls the mobile phone. After the mobile phone answers, enters 1, 8, 9, and hangs up.

```python
>>> import voiceCall
>>> def cb(args):
... 	print('DTMF:{}'.format(args))

>>> voiceCall.dtmfSetCb(cb)
0
>>> voiceCall.dtmfDetEnable(1)
0
>>> voiceCall.callStart('xxxxxxxxxxx')  # The module makes a call to the mobile phone.
0
args:(14, 1, 0, 3, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the call is ongoing, the callback function of ongoing call is called

args:(11, 1, 0, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module answers the call, the callback function of call connection is called

>>>
DTMF:1   #When "1" is pressed on the mobile phone, the event of "pressing 1" is triggered and the callback function of DTMF recognition is called

DTMF:8   #When "8" is pressed on the mobile phone, the event of "pressing 8" is triggered and the callback function of DTMF recognition is called

DTMF:9   #When "9" is pressed on the mobile phone, the event of "pressing 9" is triggered and the callback function of DTMF recognition is called
    
args:(12, 1, 0, 6, 0, 0, 'xxxxxxxxxxx', 129, 0)  # When the module hangs up the call, the callback function of call hangup is called
```

### Other Features

#### Forward Call

Call `voiceCall.setFw(reason, fwmode, phonenum)` to control call forwarding. 

`reason`: The condition for call forwarding.

`fwmode`: Whether to forward the call.

`phonenum`: Target number.

When the mobile phone makes a call to the module, after the registered callback function of call forwarding  is called and the incoming number is displayed, the module will call `voiceCall.setFw()` to answer the call and then forward it to another number.

```python
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # When a call comes in, the callback function of incoming call is called

>>> voiceCall.callAnswer() # Answer the call

0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module answers the call, the callback function of call connection is called
    
>>> voiceCall.setFw(0, 1, 'YYYYYYYYYYY')  # Forward the call unconditionally to another number
```

#### Set Volume

Call`voiceCall.setVolume(volume)` to set the volume. 

`volume`: Volume level. Range: 0–11. The larger the value, the louder the volume.

When the mobile phone makes a call to the module, the module calls *voiceCall.callAnswer()* to answer the call, then *voiceCall.getVolume()* to query the volume, and *voiceCall.setVolume()* to set the volume, and finally hangs up.

```python
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # When a call comes in, the callback function of incoming call is called

>>> voiceCall.callAnswer() # Answer the call
0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module answers the call, the callback function of call connection is called

>>>  voiceCall.getVolume()  # Query the volume
8

>>> voiceCall.setVolume(6)  # Set the volume
0

>>>  voiceCall.getVolume()  # Query the volume
6
   
>>> voiceCall.callEnd() # Hang up

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module hangs up the call, the callback function of call hangup is called
0
>>> 
```

#### Set Call Sound Output Channel

You can call `voiceCall.setChannel(device)` to set the sound output channel during a call. The default channel is 0, which is the handset.

```python
>>> voiceCall.setChannel(2) # Switch to the speaker channel
0
>>> voiceCall.setChannel(1) # Switch to the headset channel
0
>>> voiceCall.setChannel(0) # Switch back to the handset channel
0
```

#### Record Call

Call  `voiceCall.setAutoRecord()` to enable call recording, including recording to a file and streaming recording.

Enable automatic call recording.  When the mobile phone makes a call to the module, the module calls *voiceCall.callAnswer()* to answer the call, then records the call automatically, and finally hangs up.

```python
>>> voiceCall.setAutoRecord(1,0,2,'U:/test.amr')  # Enable automatic call recording
0 
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # When a call comes in, the callback function of incoming call is called

>>> voiceCall.callAnswer() # Answer the call
0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module answers the call, the callback function of call connection is called

# Recording to file...
>>> voiceCall.callEnd() # Hang up

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module hangs up the call, the callback function of call hangup is called
0
>>> 
```

##### Record to a File

When the mobile phone makes a call to the module, the module automatically answers the call, starts recording to a file, stops recording, and finally hangs up.

```python
>>> voiceCall.setAutoAnswer(3) # Enable auto answer after receiving a call for 3 seconds
0
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # When a call comes in, the callback function of incoming call is called

args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module auto answers the call, the callback function of auto answer is called
    
>>> voiceCall.startRecord(0,2,'U:/test.amr')  # Start recording to a file
0
# Recording in progress...   
>>> voiceCall.stopRecord()  # Stop recording
0
>>> voiceCall.callEnd() # Hang up

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module hangs up the call, the callback function of call hangup is called
0
>>> 
```

##### Stream Recording

When the module makes a call to the mobile phone, the mobile phone automatically answers the call, starts recording the call (stream), then hangs up and plays back the recorded audio.

```python
>>> import voiceCall
>>> import audio

>>> f=open('usr/mia.amr','w')
# Receive recordings in stream form and write them to files
>>> def cb(para):
...     if(para[2] == 1):
...         read_buf = bytearray(para[1])
...         voiceCall.readRecordStream(read_buf,para[1])
...         f.write(read_buf,para[1])
...         del read_buf
...     elif(para[2] == 3):
...         f.close()
...         
>>> voiceCall.callStart('xxxxxxxxxxx')
0
args:(14, 1, 0, 3, 0, 0, 'xxxxxxxxxxx', 129, 0)  # When the call is ongoing, the callback function of ongoing call is called

args:(11, 1, 0, 0, 0, 0, 'xxxxxxxxxxx', 129, 0)  # When the mobile phone answers the call, the callback function of call connection is called
  
>>> voiceCall.startRecordStream(0,2,cb)   # Start recording to the call in stream form
0

# Recording...
>>> voiceCall.callEnd() # Hang up (Both hangups from MO or MT are valid)

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # When the module hangs up the call, the callback function of call hangup is called
0

# View the generated recording file and play back the recording
>>> uos.listdir('usr')
['system_config.json', 'mia.amr']
>>> aud=audio.Audio(0)
>>> aud.setVolume(11)
0
>>> aud.play(2,1,'U:/mia.amr')
0
```

### Callback Function Registration for Voice Calls

Call `voiceCall.setCallback(voicecallFun)` to register the callback function of voice call status. This function allows you to monitor different call statuses and provide feedback to the user through callback functions. It enables you to observe all events during a voice call, such as incoming calls, call connections, and hangups.

In the previous examples, this callback function was registered in advance.

Example:

```python
>>> import voiceCall
>>> def cb_fun(args):
...     print('args:{}'.format(args))
...      
>>> 
>>> voiceCall.setCallback(cb_fun)  # Register the callback function of voice call status
0
```

In actual use, to distinguish callback functions of different call statuses, the following callback function can be called:

```python
# -*- coding: UTF-8 -*-
import voiceCall

class QuecVoiceCall():
	def __init__(self):
        self.__enable_log = False
        self.voice_call_set_enable_log()
        self.voice_call = voiceCall
        self.voice_call.setCallback(self.__voice_Call_back)
        
	def voice_call_set_enable_log(self,flag=True):
        self.__enable_log = flag
        
    def __log(self,args):
        if self.__enable_log:
            print("QuecSMS_LOG: {}".format(args))
            
	def __voice_Call_back(args):
		 """Callback Function Registration for Voice Calls"""
		self._log("voicecall callback args {}".format(args))
		if args[0] == 1:
			self._log("voicecall init")
		elif args[0] == 2:
			# Incoming call alert, ringing
			self._log("Call in: {}, {}".format(args[1], args[2]))
		elif args[0] == 3:
			# Connected
			self._log("Call answer: {}, {}".format(args[1], args[2]))
		elif args[0] == 4:
			# Disconnected
			self._log("Call end: {}, {}".format(args[1], args[2]))
		elif args[0] == 5:
			# Unknown error
			self._log("Call error")
		elif args[0] == 6:
			# Call waiting
			self._log("Call wait")
			self._log(args)
		elif args[0] == 7:
			# Dialing
			self._log("Call wait")
		elif args[0] == 8:
			# Dialing failed
			self._log("Call out fail")
			self._log(args)
		elif args[0] == 9:
            # Waiting
            self._log("wait")
            self._log(args)
 		elif args[0] == 10:
            # Incoming call alert, ringing  (VoLTE call)
            self._log("volte call in")
            self._log(args)
        elif args[0] == 11:
            # Connected (VoLTE call)
            self._log("volte call answer")
            self._log(args)
        elif args[0] == 12:
            # Disconnected (VoLTE call)
            self._log("volte call end")
            self._log(args)
        elif args[0] == 13:
            # Call waiting (VoLTE call)
            self._log("volte call wait")
            self._log(args)
        elif args[0] == 14:
            # Dialing (VoLTE call)
            self._log("volte call out")
            self._log(args)
        elif args[0] == 15:
            # Dialing with no ring on the other party (VoLTE call)
            self._log("volte call out 2")
            self._log(args)
        elif args[0] == 16:
            # Waiting (VoLTE call)
            self._log("volte wait")
            self._log(args)
        else:
            self._log("voicecall error")
            
	def call_start(self, topic=None, phonenum=None):
        """Dial Voice Call"""
        print(" call start phone:",phonenum)
        if phonenum:
            try:
                state = self.voice_call.callStart(phonenum)
                self._log("call_start state {}".format(state))
                return state
            except Exception:
                self._log("call_start error")
                return False
				
	def call_answer(self, topic=None, data=None):
        """Answer Voice Call"""
        try:
            state = self.voice_call.callAnswer()
            self._log("call_answer state {}".format(state))
        except Exception:
            self._log("call_answer error")
			
    def call_end(self, topic=None, data=None):
        """End Voice Call"""
        try:
            state = self.voice_call.callEnd()
            self._log("call_end state {}".format(state))
        except Exception:
            self._log("call_end error")
         
if __name__ == '__main__':
    VoiceCall = QuecVoiceCall()
	# The module makes a call to the mobile phone
	VoiceCall.call_start('xxxxxxxxxxx')
	
	# Hang up
	VoiceCall.call_end()
```

The callback message `args` is different under CS calls and VoLTE calls, see below for details.

#### Callback Messages for CS Calls

In the callback message for CS calls, the number of parameters in the callback function is not fixed, but is determined by the first parameter (trigger event) `args[0]`, as shown in the table:

| Value | Number of Parameters | Trigger Event `args[0]` Value Explanation | Other Parameter Explanations                                 |
| ----- | -------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| 1     | 1                    | voice call is initialized successfully    |                                                              |
| 2     | 3                    | Incoming call notification, ringing       | `args[1]`: Caller ID<br>`args[2]`: Phone number              |
| 3     | 3                    | Call connected                            | `args[1]`: Caller ID<br>`args[2]`: Phone number              |
| 4     | 3                    | Call hung up                              | `args[1]`: Caller ID<br/>`args[2]`: Call hangup reason       |
| 5     | 1                    | Unknown error                             |                                                              |
| 6     | 5                    | Call waiting                              | `args[1]`: Caller ID<br/>`args[2]`: Phone number<br/>`args[3]`: Number type, 129 or 145 <br />129: Non-international number <br />145: International number<br/>`args[4]`: CLI status |
| 7     | 1                    | Call outgoing                             |                                                              |
| 8     | 4                    | Call outgoing failed                      | `args[1]`: Caller ID<br/>`args[2]`: Call failure reason<br/>`args[3]`: Whether in-band tones can be obtained from the network |
| 9     | 3                    | Waiting                                   | `args[1]`: Caller ID<br>`args[2]`: Phone number              |

#### Callback Messages for VoLTE Calls

In the callback messages for VoLTE calls, the number of callback function parameters is fixed at 8. The first parameter `args[0]` represents the trigger event, as shown in the table:

| Parameter | Number of Parameters | Trigger Event `args[0]` Value Explanation   | Other Parameter Explanations                                 |
| --------- | -------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| 10        | 8                    | Incoming call notification, ringing (VoLTE) | `args[1]`: Caller ID<br/>`args[2]`: Call direction (MO/MT)<br/>`args[3]`: Call status<br/>`args[4]`: Service type (usually 0, representing voice call)<br/>`args[5]`: Multi-party call flag<br />0: Not a multi-party call <br />1: Multi-party call<br/>`args[6]`: Phone number<br/>`args[7]`: Number type, 129 or 145 <br />129: Non-international number <br />145: International number<br/> |
| 11        | 8                    | Call connected (VoLTE)                      | `args[1] – args[7]`: Same explanations as above              |
| 12        | 8                    | Call hung up (VoLTE)                        | `args[1] – args[7]`: Same explanations as above              |
| 13        | 8                    | Call waiting (VoLTE)                        | `args[1] – args[7]`: Same explanations as above              |
| 14        | 8                    | Call outgoing (VoLTE)                       | `args[1] – args[7]`: Same explanations as above              |
| 15        | 8                    | Call outgoing, no answer (VoLTE)            | `args[1] – args[7]`: Same explanations as above              |
| 16        | 8                    | Waiting (VoLTE)                             | `args[1] – args[7]`: Same explanations as above              |

For args[0] with values of 10-16, args[1]-args[8] are described in [WiKl](../../api_reference/iotlib/voiceCall.md#voiceCall.setCallback):

## Frequently Asked Questions

What are the requirements for the SIM card when the module makes a call?

If the module only supports 4G networks, voice calls can only be made using the 4G network, and the firmware of the module must support VoLTE

If the module supports both 2G and 4G networks, and the SIM card inserted supports both 2G and 4G networks, but the firmware does not support VoLTE, voice calls will default to using the 2G network with CSFB solution for voice calls.

The following table shows the specific requirements for the SIM card, module, and firmware when making voice calls:

| CS/voLTE Call | Requirement for SIM Card | Requirement for Module | Requirement for Firmware |
| ------------- | ------------------- | ------------------ | ------------------------ |
| 2G CS Call    | Support 2G networks | Support 2G networks | N/A                      |
| 4G VoLTE Call | Support 4G networks | Support 4G networks | Support VoLTE |
