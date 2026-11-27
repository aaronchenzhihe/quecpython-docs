# 电话

本文档主要介绍了LTE电话方案、常见概念、QuecPython各平台电话功能支持情况以及voiceCall功能应用。针对语音通话功能进行一些基础概念的介绍，并就QuecPython的语音通话功能如何使用进行说明。

## LTE电话方案

LTE（Long-Term Evolution）是一种4G无线通信技术，它提供了高速数据传输和高质量语音通话功能。在LTE网络上，可以采用几种不同的电话方案。如基于CS的电话方案和基于VoLTE的电话方案。

  ### 基于CS的电话方案

CSFB（Circuit Switched Fallback）是指CS语音回落，用于在LTE和2、3G网络共同覆盖的区域对于不支持IMS业务的终端回落到2、3G网络使用CS进行语音业务，同时保持数据业务连续性的功能。

即LTE终端平常会驻留在LTE网络，当需要完成语音业务时，网络辅助终端回落到2G/3G网络的CS域，通话结束后再重新返回LTE网络。

CSFB方案除了需要提供CS语音支持外，还支持其他的CS业务，如SMS等。

#### 背景

1. LTE网络下没有专门用于承载语音呼叫业务的CS电路交换域，并且在IMS功能尚未完善时（终端/网络侧），不能实现语音业务。但语音业务又是运营商不可少的一项重要业务和收入来源，因此在LTE网络也要支持语音呼叫业务。
2. 在LTE发展的初期，此时还存在大量的2G/3G网络， 采用原有的CS域语音方案来提供语音服务，可以让运营商以最小的改动，最小的投资解决LTE网络语音业务的问题。

#### CSFB实现

##### 前提条件

1. ###### 模组侧

   - UE在ATTACH时需要进行联合附着(Combined EPS/IMSI attach Procedure)。
   - 支持NAS层扩展业务请求功能(Extended Service Request)。
   - 模组需要支持2G或者3G。

2. ###### 网络侧

   - 支持NAS层扩展业务请求功能(Extended Service Request)。
   - 需支持SGs接口。
   - SGs是介于MME和MSC之间的一个功能接口，是实现CSFB和SGs短信必不可少的，SGs接口信令消息传输是基于可靠IP通信协议层的。

##### CSFB流程

![CSFB_Flow_chart](../media/network-comm/phone/CSFB_Flow_chart.png)

基于CSFB的电话方案可以满足移动通信运营商需要在LTE网络中提供语音通信的要求，并且可以保证通话的可靠性。

  ### 基于VoLTE的电话方案

VoLTE (Voice over LTE) 是一种基于4G网络的高清语音通信技术，相对于传统的CSFB技术，它能提供更好的语音质量、更快的通话连接速度和更少的通话中断，从而更好地满足用户对语音通信的需求。

基于VoLTE的电话方案是在4G网络上实现语音通信的方式，它使用IP网络来进行语音传输，而不需要像传统电路交换网络那样转换为模拟信号，从而实现了高清语音通话和数据传输并行。使用基于VoLTE的电话方案，用户可以同时保持高质量的语音通话和数据传输，同时还可以预留更多的网络带宽，使得整个网络更加稳定和高效。

尽管实现基于VoLTE的电话方案需要投入更多的网络建设和设备改造，但随着4G网络的普及，以及5G网络的到来，越来越多的通信运营商开始采用VoLTE技术，以提升用户的通话体验和网络效率。

## 常见概念说明

本章节主要介绍语音通信相关的常见概念，包括DTMF、IMS和VoLTE。

  ### DTMF
#### 什么是DTMF

DTMF（Dual Tone Multi Frequency）是“双音多频”的缩写，是一种使用在电话系统中的多频信号技术。它使用由两个频率信号组合而成的音调来识别电话用户输入的数字、字母或其他特殊字符等。DTMF在电话中的应用场景包括电话键盘输入、远程控制等。它通常用于识别电话用户输入的命令，例如查集团电话、转接电话、停止呼叫等。同时，也可以用于调用自动服务系统，例如银行自动交互系统，用户可以通过DTMF输入按键完成特定功能的目的。

#### DTMF的基本原理和编码方式

基本原理

其基本原理是使用两个频率信号的组合来代表电话键盘上输入的数字、字母或特殊字符等。通常，使用低频信号和高频信号的组合可以表示数字0到9，以及星号(*)和井号(#)等特殊字符。不同的数字和特殊字符对应的频率是固定的，例如数字“1”对应的频率组合是697 Hz和1209 Hz。

在电话按键上按下某个键时，呼叫终端会发出由两个频率构成的信号，这个信号被传输到另一端，另一端的接收终端可以通过解析这个信号来确定用户输入的数字或特殊字符。根据DTMF编码方式，每个数字或特殊字符有唯一的频率组合，这使得DTMF成为一种用于数字和特殊字符识别的有效方式。

编码方式

DTMF编码方式则是通过在低频和高频信号中选取不同的频率，将数字、字母或特殊字符等信号编码成相应的信号序列。DTMF信号是用一组8个频率的正弦波来编码的，4个频率用于表示低频部分，另外4个用于表示高频部分。下面是DTMF的频率编码表：

| 高/低频率 | 1209Hz | 1336Hz | 1477Hz | 1633Hz |
| --------- | ------ | ------ | ------ | ------ |
| 697Hz     | 1      | 2      | 3      | A      |
| 770Hz     | 4      | 5      | 6      | B      |
| 852Hz     | 7      | 8      | 9      | C      |
| 941Hz     | *      | 0      | #      | D      |

比如，输入数字“8”时，发送了频率为852Hz和1336Hz的两个信号。使用不同的频率组合，可以编码出不同的数字、字母或特殊字符等信号。通过这种编码方式，电话系统可以准确地识别用户输入的命令并给出相应的反馈。

#### DTMF在电话中的作用和应用

DTMF信号的作用是识别电话用户按下的数字键，从而实现拨号、呼叫转移、消息留言等功能。例如，当用户在电话上拨打座机号码时，他们按下数字键，电话会发送相应的DTMF信号来拨号，并将电话连接到所拨打号码的终端设备上。

DTMF通常用于识别电话用户输入的命令，例如查集团电话、转接电话、停止呼叫等。具体来讲，DTMF在电话中的应用包括：

1. 拨打电话：用户可以使用DTMF拨打目标电话号码。DTMF信号输入电话交换机后，会与目标电话建立连接。
2. 自动服务系统：DTMF可以用于调用自动服务系统，例如银行自动语音交互系统，在电话中通过输入DTMF按键，用户可以选择想要的服务。
3. 远程控制：DTMF也可以通过电话拨号盘向远程设备发送控制信号，例如控制塔吊、抽水泵等。通过特定的DTMF命令，可以向命令控制中心发送远程控制信号，以控制远程设备。
4. 电话会议：DTMF在电话会议中也有应用，例如控制电话会议的开启和关闭、静音和取消静音等操作。

  ### IMS
#### 什么是IMS

IMS (IP Multimedia Subsystem) 是一种基于 IP 技术的多媒体业务子系统架构，它可以使电信网络实现集成语音、视频、数据和多媒体内容的传输和交互。

IMS 架构将传统的电话网络与 IP 网络相结合，使用 SIP（Session Initiation Protocol）作为控制信令，使服务能够在各种网络中提供。而且，IMS 可以在不同的终端上提供业务，例如 PC、手持设备或 VoIP 终端。

IMS 提供的主要功能包括：

1.创建和维护多媒体会话：例如语音通话、视频会议和消息传递。

2.提供服务：例如文件共享、即时消息、社交网络和电子邮件。

3.提供用户数据管理：例如个人档案、地址簿和网络识别。

4.支持 QoS（Quality of Service）机制，以保证多媒体业务的服务质量和性能。

5.支持安全和身份验证机制，以确保用户数据的保密性和完整性。

IMS 的出现可以使电信业务从传统的语音通信向数据、多媒体通信等方向扩展，具有非常重要的意义，使得运营商可以通过 IMS 平台构建全新的业务模式和增值服务，加强了其商业竞争力。

#### IMS的基本原理

IMS 的基本原理是通过使用 IP 网络和 SIP 协议，将传统的电话网络进行改造，从而实现集成语音、视频和数据的传输和交互。

在 IMS 中，所有的服务和应用程序都是基于 SIP 和 HTTP 协议实现的。当用户需要使用 IMS 提供的服务时，他们需要在终端设备上启动一个 IMS 会话，其过程如下：

1.用户使用 IMS 终端设备（例如智能手机）启动应用程序，并从 IMS 报文服务器中获取服务并向 IMS 服务器注册。

2.IMS 服务器识别用户并与其建立会话。

3.用户可以使用 IMS 终端设备进行多媒体通信，也可以访问 IMS 提供的服务，例如邮件、消息和文件共享。

IMS 的基本架构包括：呼叫控制层、多媒体控制层、多媒体处理层和多媒体传输层。

- 呼叫控制层：主要负责协调 IMS 系统中多个实体之间的交互和通信，例如用户设备、创建会话和控制信令等。
- 多媒体控制层：负责多媒体流的生成和管理，以及多媒体编解码器的控制和媒体处理的变化，例如转码、混音等。
- 多媒体处理层：用于处理多媒体的编解码、压缩、分解和解析等，确保多媒体流传输的质量和容错性。
- 多媒体传输层：负责数据包的传输，例如 UDP 或 TCP。

IMS 还支持 QoS（Quality of Service）机制，以保证多媒体业务的服务质量和性能。同时，它也支持安全和身份验证机制，以确保用户数据的保密性和完整性。

#### IMS和电话功能的关系

IMS 可以看作是整合了传统电话网络和现代通信技术的一种解决方案。IMS 在保留传统电话通信功能的同时，还能够提供更多的多媒体和数据传输功能。

传统的电话通信功能包括语音通话和SMS等，而IMS可以将传统电话服务和视频通话服务结合起来，允许用户在手机上进行视频通话。此外，IMS 还可以提供Voicemail (语音留言) 和 Instant Messaging (即时消息)等服务。

与传统的电话相比，IMS提供了更加丰富、灵活和多样化的通信服务和解决方案，用户能够通过 IMS 实现从语音通信到多媒体通信的完整转变。

当然，在 IMS 中，电话功能依然是非常重要的一部分，IMS在保留传统电话通信功能之外，增加了更多的数据和多媒体通信功能，能够为用户提供更多的选择。

![IMS](../media/network-comm/phone/IMS.png)

  ### VoLTE
#### 什么是VoLTE

VoLTE (Voice over LTE) 是一种基于 LTE 网络的语音通信服务，它可以提供高质量、高性能的语音通话服务，同时，保留多媒体传输、即时消息、网络识别等 IMS 功能，实现了语音和数据的融合。

VoLTE 技术是基于 IMS 架构实现的，它使用 SIP 协议进行呼叫控制，并使用高品质的 AMR (Adaptive Multi-Rate) 编解码器等技术，提供高保真度的语音服务。与传统的 GSM、CDMA 等通信技术相比，VoLTE 可以提供更加稳定、高质量的语音通话服务。

传统上，语音通话使用2G或3G网络进行传输，而LTE网络主要用于数据传输。VoLTE的出现使得语音通话可以直接在LTE网络上进行，而不需要通过降级到2G或3G网络来实现。

#### VoLTE的基本原理

VoLTE 的基本原理是将语音信号通过 IP 网络转发，将传统的分离的语音服务和数据服务进行整合。在 VoLTE 中，语音数据可以直接在 IP 网络上传输，而不需要再进行转换和编解码。同时，VoLTE 还支持 HD Audio 等高质量语音技术，使通话更加清晰、自然。

当用户拨打电话时，VoLTE 使用 SIP 协议来与 IMS 网络进行通信，并使用 QoS 技术来保证语音服务的优先级和稳定性。VoLTE 还支持多方通话和拒绝来电等功能，以满足用户的多样化需求。

VoLTE是基于IMS网络的，简单来说，IMS是增强版的CS，于是在LTE的基础上，IMS取代了CS，使得手机用户可以打语音电话，且数据业务不会中断。增加IMS，相当于在使用LTE数据通道的基础上，给语音业务专门开辟了一条业务通道，这种IMS+LTE的语音解决方案，就是VoLTE。

![voLTE_Flow_chart](../media/network-comm/phone/voLTE_Flow_chart.png)

#### VoLTE的优势说明

使用VoLTE技术后，语音通话将也可以通过4G LTE网络进行，实现语音和数据同时传输。

VoLTE 相对于传统的语音通信技术，有以下优势：

1.高清晰度语音：VoLTE 可以使用高品质编解码技术，提供高质量和高清晰度的语音通话服务。

2.更快的连接速度：VoLTE 基于 LTE 网络进行通信，连接速度更快，呼叫建立时间更短，同时支持无缝切换和高质量通话。

3.多媒体通信：VoLTE 可以支持多媒体和数据业务，例如视频通话、短信、即时消息和文件共享等。

4.更好的覆盖范围：由于 VoLTE 是基于 LTE 网络实现的，因此在网络覆盖范围和室内覆盖方面具有较大的优势。

总的来说，VoLTE技术可以提高通话质量和用户体验，同时也有提高网络利用率的作用。

#### VoLTE和IMS的关系

VoLTE 是 IMS 的一个重要应用场景。IMS 通过 SIP 控制信令来处理呼叫和建立语音通话，VoLTE 技术采用了 IMS 架构，通过 SIP 协议实现呼叫控制和呼叫会话的管理。

VoLTE和IMS之间有着密切的关系。VoLTE基于IMS（IP Multimedia Subsystem）技术，使用VoLTE技术的电话必须具备VoLTE技术的运营商支持和4G网络支持，而IMS则提供了VoLTE所需的多媒体语音和视频服务。

IMS是一个开放的、标准化的、IP模块化的多媒体业务子系统，是4G移动网络的核心技术之一。它允许各种媒体信号（包括语音、数据、视频等）的实时多媒体业务在IP网络上进行传输，例如语音、视频、聊天、游戏、文件传输等，允许各个应用之间进行互联互通。

VoLTE基于IMS技术，通过IMS的语音和视频服务，实现高质量的语音通话，同时也可以实现高速数据传输。IMS提供的多媒体控制和处理能力有助于优化VoLTE通话的质量，并提供了一种高效的方式进行语音通信。同时，VoLTE基于IMS的技术还能够充分利用4G网络的高速数据传输能力，打破了以往语音通信只能单独使用2G或3G网络的局限。

VoLTE 最主要的功能就是实现基于 IMS 架构的高清晰度语音通话，它利用了 IMS 能力包括会话控制、质量保证、安全性、多媒体处理等。

总之，VoLTE 通过 IMS 架构实现高清晰度的语音通话服务，并能够支持多媒体和数据业务，大幅提升了通信服务质量和用户体验。

## QuecPython各平台电话功能支持情况
  voiceCall是QuecPython的语音通话功能模块，即我们通常说的打电话功能。下表是QuecPython的voiceCall支持的功能，以及各个平台的模组对这些功能的支持情况。

| 平台模组<br />功能 | EC200N系列：<br />CN_AA<br />CN_AC<br />CN_LA | EC600N系列：<br />CN_LC<br />CN_LD<br />CN_LF | EG912N系列：<br />EN_AA<br />EG915N系列：<br />EU_AG<br>EA_AC | EC200A系列：<br />AU_HA<br />CN_DA<br />CN_HA  <br />CN_LA<br />EU_HA | EC200U系列：<br />AU_AA<br />AU_AB<br>CN_AA<br />EU_AA<br />EU_AB | EC600U系列：<br />EU_AB<br />EC_AC<br>LA_AC | EG912U系列：<br>GL_AA<br>GL_AC<br>EG915U系列：<br>EU_AB<br>LA_AB |
| ------------------ | --------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- | ------------------------------------------------------------ |
| 设置自动应答时间   | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 拨打电话           | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 接听电话           | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 挂断电话           | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 回调注册功能       | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 来电自动挂断       | ❌                                             | ❌                                             | ❌                                                            | ✔                                                            | ❌                                                            | ❌                                           | ❌                                                            |
| 设置DTMF           | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 识别DTMF           | ❌                                             | ✔                                             | ✔                                                            | ❌                                                            | ❌                                                            | ❌                                           | ❌                                                            |
| 呼叫转移           | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 切换语音通道       | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 音量大小配置       | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |
| 自动录音           | ✔                                             | ✔                                             | ✔                                                            | ✔                                                            | ✔                                                            | ✔                                           | ✔                                                            |

当前voiceCall功能的支持以及各个平台的模组对这些功能的支持情况表： ✔ 支持；❌不支持

>* BC25/EC600G/EC800G/BG95系列模组不支持voiceCall功能。
>* EC600M/EC800M/EG810M系列模组需要支持VOLTE功能固件才支持voiceCall功能。
>* 其他系列模组需要定制版本才能支持voiceCall功能。

请注意，以上信息可能会随着模组的更新和新版本的发布而变化。建议在使用这些功能之前，仔细查看[官网wiki文档](../../api_reference/iotlib/voiceCall.md)并与FAE进行确认。

## voiceCall功能应用

QuecPython的语音通话功能模块voiceCall，能够实现基本通话，DTMF，呼叫转移等，下面具体介绍这些功能的使用。

下面基于voiceCall的voLTE通话演示手机和模组进行语音通话的场景。以下操作中，模组端的操作都基于QPYcom工具，先打开QPYcom，并选择模组的python交互端口，进入如下界面：

![QPYcom_voiceCall_zh](../media/network-comm/phone/QPYcom_voiceCall_zh.png)

为了方便看到整个语音通话过程中的所有事件，比如电话呼入、接通、挂断等事件。以下功能演示均提前注册回调函数，见4.4通话回调注册。

下面所有示例中的“xxxxxxxxxxx”表示的是手机号码，实际测试时，显示的是真实的手机号码。

  ### 基本电话功能

#### 	呼叫、接听

模组给手机打电话，手机端接听，然后挂断电话。

```python 
>>> import voiceCall
>>> voiceCall.callStart('xxxxxxxxxxx') # 模组主动拨打电话给手机
0
>>> 
args:(14, 1, 0, 3, 0, 0, 'xxxxxxxxxxx', 129, 0) # 回调通知当前正在呼出中

args:(11, 1, 0, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 手机端接听，回调通知电话接通事件

args:(12, 1, 0, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # 手机端挂断，回调通知电话挂断事件
```

用手机拨打电话给模组，当看到上述注册的回调触发并显示呼入电话的信息时，使用如下接口来接听电话：

```python
>>> import voiceCall
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # 电话呼入时，触发回调

>>> voiceCall.callAnswer() # 接听电话

0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 接通后，触发回调
```

#### 	自动接听

手机给模组打电话，模组端自动接听，然后主动挂断电话。

设置自动应答

```python
>>> import voiceCall
>>> voiceCall.setAutoAnswer(3) # 设置3s后自动接听电话
0
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # 电话呼入时，触发回调

args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 自动接听后，触发回调

```

#### 	挂断电话

用手机拨打电话给模组，当看到上述注册的回调触发并显示呼入电话的信息时，使用如下接口来接听电话，随后挂断电话：

```python
>>> import voiceCall
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # 电话呼入时，触发回调

>>> voiceCall.callAnswer() # 接听电话

0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 接通后，触发回调

>>> voiceCall.callEnd() # 挂断电话

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # 通话挂断后，触发回调
0
>>> 
```

#### 	自动挂断

用手机拨打电话给模组，当模块设置不会自动挂断时，会正常收到回调；当设置模块为自动挂断时，模块不会触发回调，也没有任何反应，反应在手机上，会听到电话已接通，随后快速挂掉。

```python
>>> import voiceCall
#手机呼叫模块，默认不会自动挂断
>>> voiceCall.getAutoCancelStatus()
0

#设置自动挂断功能，手机呼叫模块，默认自动挂断
>>> voiceCall.setAutoCancel(1)
0
>>> voiceCall.getAutoCancelStatus()
1
```

  ### DTMF功能

QuecPython的语音通话功能模块voiceCall，能够实现设置和接收DTMF功能，下面是介绍DTMF的具体实现。

#### 	设置DTMF

使用`voiceCall.startDtmf(dtmf, duration)`能够设置DTMF，其中：

`dtmf`：表示DTMF字符串，有效字符数有：`0-9、A、B、C、D、*、#`

`duration`：表示 持续时间，单位：毫秒。

仅在语音通话过程中使用生效。

```python
>>> import voiceCall
>>> voiceCall.startDtmf('8',100)  #等效于手机在按键输入‘8’
0
```

#### 	DTMF识别功能配置

使用`voiceCall.dtmfDetEnable(enable)`可开启或关闭DTMF识别功能，默认不开启DTMF识别。当开启DTMF识别功能后，可通过注册DTMF识别功能回调函数查看识别到的DTMF字符串。

```python
>>> import voiceCall
>>> voiceCall.dtmfDetEnable(0)  #关闭DTMF识别功能

>>> voiceCall.dtmfDetEnable(1)  #开启DTMF识别功能
```

#### 	DTMF识别功能回调使用

使用`voiceCall.dtmfSetCb(dtmfFun)`可注册DTMF识别功能的回调函数，用户识别对端输入的DTMF字符串。

模组给手机打电话，手机端接听后，手机分别输入1，8，9 ，然后挂断电话。

```python
>>> import voiceCall
>>> def cb(args):
... 	print('DTMF:{}'.format(args))

>>> voiceCall.dtmfSetCb(cb)
0
>>> voiceCall.dtmfDetEnable(1)
0
>>> voiceCall.callStart('xxxxxxxxxxx')  # 模组主动拨打电话给手机，
0
args:(14, 1, 0, 3, 0, 0, 'xxxxxxxxxxx', 129, 0) # 回调通知当前正在呼出中， 触发通话状态回调，

args:(11, 1, 0, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 手机端接听，回调通知电话接通事件， 触发通话状态回调，

>>>
DTMF:1   #手机端按下1，回调函数中会收到按下的字符“1”， 触发DTMF识别功能回调，

DTMF:8   #手机端按下8， 触发DTMF识别功能回调，

DTMF:9   #手机端按下9， 触发DTMF识别功能回调。
    
args:(12, 1, 0, 6, 0, 0, 'xxxxxxxxxxx', 129, 0)  # 手机端挂断，回调通知电话挂断事件，触发通话状态回调。
```

### 其他电话功能
#### 呼叫转移

使用`voiceCall.setFw(reason, fwmode, phonenum)`可以控制呼叫转移业务，当收到来电时根据呼叫转移的条件`reason`,控制是否转移`fwmode`到目标号码`phonenum`。

用手机拨打电话给模组，当看到上述注册的回调触发并显示呼入电话的信息时，使用如下接口来接听电话，随后呼叫转移至另一个号码：

```python
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # 电话呼入时，触发回调

>>> voiceCall.callAnswer() # 接听电话

0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 接通后，触发回调
    
>>> voiceCall.setFw(0, 1, 'YYYYYYYYYYY')  # 无条件呼叫转移至另一个号码
```

#### 音量设置

使用`voiceCall.setVolume(volume)`可以设置音量等级`volume` ，范围`（0 ~ 11）`，数值越大，音量越大。

用手机拨打电话给模组，使用接口来接听电话，随后查询音量，设置音量，最后挂断：

```python
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # 电话呼入时，触发回调

>>> voiceCall.callAnswer() # 接听电话
0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 接通后，触发回调

>>>  voiceCall.getVolume()  # 查询音量
8

>>> voiceCall.setVolume(6)  # 设置音量
0

>>>  voiceCall.getVolume()  # 查询音量
6
   
>>> voiceCall.callEnd() # 挂断电话

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # 通话挂断后，触发回调
0
>>> 
```

#### 通话声音输出通道配置

使用`voiceCall.setChannel(device)`可以设置通话时的声音输出通道，默认是通道0，即听筒。

```python
>>> voiceCall.setChannel(2) #切换到喇叭通道
0
>>> voiceCall.setChannel(1) #切换到耳机通道
0
>>> voiceCall.setChannel(0) #切换回听筒通道
0
```

#### 通话录音

使用`voiceCall`录音接口，可以使用通话录音功能，包含录音到文件，和流式录音。

开启自动录音功能，用手机拨打电话给模组，使用接口来接听电话，自动录音，最后挂断：

```python
>>> voiceCall.setAutoRecord(1,0,2,'U:/test.amr')  # 开启自动录音功能
0 
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # 电话呼入时，触发回调

>>> voiceCall.callAnswer() # 接听电话
0
>>> 
args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 接通后，触发回调

# 自动录音到文件中...
>>> voiceCall.callEnd() # 挂断电话

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # 通话挂断后，触发回调
0
>>> 
```

##### （1）录音到文件

用手机拨打电话给模组，自动接听电话，开启录音到文件，停止录音，最后挂断：

```python
>>> voiceCall.setAutoAnswer(3) # 设置3s后自动接听电话
0
>>> 
args:(10, 1, 1, 4, 0, 0, 'xxxxxxxxxxx', 129, 0) # 电话呼入时，触发回调

args:(11, 1, 1, 0, 0, 0, 'xxxxxxxxxxx', 129, 0) # 自动接听后，触发回调
    
>>> voiceCall.startRecord(0,2,'U:/test.amr')  # 开启录音到文件
0
# 录音到文件中...   
>>> voiceCall.stopRecord()  # 停止通话录音
0
>>> voiceCall.callEnd() # 挂断电话

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # 通话挂断后，触发回调
0
>>> 
```

##### （2）流式录音

模组给手机打电话，手机端接听后，开始通话录音（流形式），然后挂断电话，播放之前的录音：

```python
>>> import voiceCall
>>> import audio

>>> f=open('usr/mia.amr','w')
# 用于接收流形式的录音，写入文件中
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
args:(14, 1, 0, 3, 0, 0, 'xxxxxxxxxxx', 129, 0)  # 回调通知当前正在呼出中，触发通话状态回调，

args:(11, 1, 0, 0, 0, 0, 'xxxxxxxxxxx', 129, 0)  # 手机端接听，回调通知电话接通事件，触发通话状态回调，
  
>>> voiceCall.startRecordStream(0,2,cb)   # 开始通话录音（流形式）
0

# 录音中...
>>> voiceCall.callEnd() # 挂断电话(MO/MT侧挂断都可以)

args:(12, 1, 1, 6, 0, 0, 'xxxxxxxxxxx', 129, 0) # 通话挂断后，触发回调
0

# 查看生成的录音文件，播放录音
>>> uos.listdir('usr')
['system_config.json', 'mia.amr']
>>> aud=audio.Audio(0)
>>> aud.setVolume(11)
0
>>> aud.play(2,1,'U:/mia.amr')
0
```

  ### 通话回调注册

使用`voiceCall.setCallback(voicecallFun)`可注册回调函数。监听不同的通话状态并通过回调反馈给用户，方便看到整个语音通话过程中的所有事件，比如电话呼入、接通、挂断等事件。

在前面的示例中均提前注册了此回调函数。

示例如下：

```python
>>> import voiceCall
>>> def cb_fun(args):
...     print('args:{}'.format(args))
...      
>>> 
>>> voiceCall.setCallback(cb_fun)  # 注册回调函数
0
```

实际使用为了区分不同电话状态的回调，可以使用下面示例：

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
		 """通话状态回调"""
		self._log("voicecall callback args {}".format(args))
		if args[0] == 1:
			self._log("voicecall init")
		elif args[0] == 2:
			# 来电通知
			self._log("Call in: {}, {}".format(args[1], args[2]))
		elif args[0] == 3:
			# 电话接通
			self._log("Call answer: {}, {}".format(args[1], args[2]))
		elif args[0] == 4:
			# 通话挂断
			self._log("Call end: {}, {}".format(args[1], args[2]))
		elif args[0] == 5:
			# 通话挂断
			self._log("Call error")
		elif args[0] == 6:
			# 呼叫等待
			self._log("Call wait")
			self._log(args)
		elif args[0] == 7:
			# 呼出中
			self._log("Call out")
		elif args[0] == 8:
			# 呼出中
			self._log("Call out fail")
			self._log(args)
		elif args[0] == 9:
            # 呼出中
            self._log("wait")
            self._log(args)
 		elif args[0] == 10:
            # 来电通知（volte通话）
            self._log("volte call in")
            self._log(args)
        elif args[0] == 11:
            # 通话接通（volte通话）
            self._log("volte call answer")
            self._log(args)
        elif args[0] == 12:
            # 通话挂断（volte通话）
            self._log("volte call end")
            self._log(args)
        elif args[0] == 13:
            # 呼叫等待（volte通话）
            self._log("volte call wait")
            self._log(args)
        elif args[0] == 14:
            # 呼出中（volte通话）
            self._log("volte call out")
            self._log(args)
        elif args[0] == 15:
            # 呼出中，对方未响铃（volte通话）
            self._log("volte call out 2")
            self._log(args)
        elif args[0] == 16:
            # 等待（volte通话）
            self._log("volte wait")
            self._log(args)
        else:
            self._log("voicecall error")
            
	def call_start(self, topic=None, phonenum=None):
        """拨打电话"""
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
        """接听电话"""
        try:
            state = self.voice_call.callAnswer()
            self._log("call_answer state {}".format(state))
        except Exception:
            self._log("call_answer error")
			
    def call_end(self, topic=None, data=None):
        """挂断电话"""
        try:
            state = self.voice_call.callEnd()
            self._log("call_end state {}".format(state))
        except Exception:
            self._log("call_end error")
         
if __name__ == '__main__':
    VoiceCall = QuecVoiceCall()
	# 主动拨打电话
	VoiceCall.call_start('xxxxxxxxxxx')
	
	# 接通后，挂断电话
	VoiceCall.call_end()
```

 在CS通话和VoLTE通话下回调消息`args`是不同的，具体看下面说明。

#### CS通话的回调消息

在CS通话的回调消息中回调函数的参数个数并不是固定的，而是根据第一个参数（触发事件）`args[0]`来决定，如下表：

| 值   | 参数个数 | 触发事件args[0]值说明 | 其他参数说明                                                 |
| ---- | -------- | --------------------- | ------------------------------------------------------------ |
| 1    | 1        | voicecall初始化成功   |                                                              |
| 2    | 3        | 来电通知，响铃        | `args[1]`：呼叫识别号码<br>`args[2]`：电话号码               |
| 3    | 3        | 通话接通              | `args[1]`：呼叫识别号<br>`args[2]`：电话号码                 |
| 4    | 3        | 通话挂断              | `args[1]`：呼叫识别号码<br/>`args[2]`：通话挂断原因          |
| 5    | 1        | 未知错误              |                                                              |
| 6    | 5        | 呼叫等待              | `args[1]`：呼叫识别号码<br/>`args[2]`：电话号码<br/>`args[3]`：号码类型[129/145],129:非国际号码,145:国际号码<br/>`args[4]`：CLI状态 |
| 7    | 1        | 呼出中                |                                                              |
| 8    | 4        | 呼出失败              | `args[1]`：呼叫识别号码<br/>`args[2]`：呼叫失败原因<br/>`args[3]`：指示是否可以从网络端获得in-band tones |
| 9    | 3        | 等待                  | `args[1]`：呼叫识别号码<br>`args[2]`：电话号码               |

#### VoLTE通话的回调消息

在VoLTE通话的回调消息中回调函数的参数个数是固定8个，第一个参数`args[0]`为触发事件，如下表：

| 参数 | 参数个数 | 触发事件args[0]值说明           | 其他参数说明                                                 |
| ---- | -------- | ------------------------------- | ------------------------------------------------------------ |
| 10   | 8        | 来电通知，响铃（volte通话）     | `args[1]`：呼叫识别号码<br/>`args[2]`：呼叫方向(MO/MT)<br/>`args[3]`：通话状态<br/>`args[4]`：业务类型(这里一般都是0，表示voice call，语音通话业务)<br/>`args[5]`：多方通话标志，0：非多方通话，1：多方通话<br/>`args[6]`：电话号码<br/>`args[7]`：号码类型[129/145],129:非国际号码,145:国际号码 |
| 11   | 8        | 通话接通（volte通话）           | `args[1] ~ args[7]`：具体说明同上                            |
| 12   | 8        | 通话挂断（volte通话）           | `args[1] ~ args[7]`：具体说明同上                            |
| 13   | 8        | 呼叫等待（volte通话）           | `args[1] ~ args[7]`：具体说明同上                            |
| 14   | 8        | 呼出中（volte通话）             | `args[1] ~ args[7]`：具体说明同上                            |
| 15   | 8        | 呼出中，对方未响铃（volte通话） | `args[1] ~ args[7]`：具体说明同上                            |
| 16   | 8        | 等待（volte通话）               | `args[1] ~ args[7]`：具体说明同上                            |

关于`args[0]`的值为10-16时，`args[1]`-`args[8]`的[详细说明见WiKI](../../api_reference/iotlib/voiceCall.md#voiceCall.setCallback)：

## 常见问题解答

（1）模组打电话，对SIM卡有什么要求？

如果模组仅支持4G网络，语音通话时，只能使用4G网络来进行语音通话，并且模组的固件必须支持VoLTE功能。

如果模组本身支持2G和4G，使用的SIM卡既支持2G也支持4G，但是使用的固件不支持VoLTE功能，则语音通话时，默认使用2G网络，会使用CSFB的电话方案。

下表是进行语言通话时需要对sim卡、模组、固件的具体要求：

| CS/voLTE通话  | SIM卡要求           | 模组要求       | 固件要求                |
| ------------- | ------------------- | -------------- | ----------------------- |
| 2G  CS通话    | SIM卡需要支持2G网络 | 模组需要支持2G | 无                      |
| 4G  voLTE通话 | SIM卡需要支持4G网络 | 模组需要支持4G | 需要支持VoLTE功能的固件 |

（2）中国境内，哪些运营商还支持2G/3G电话功能？

中国境内，自从2020年5月，工信部就正式发布了《关于深入推进移动物联网全面发展的通知》，其首次以公开发文形式正式提出2G/3G要迁移转网，明确中低速物联网应用要向NB-IoT和Cat.1网络迁移。

目前中国境内四大运营商对2G/3G的支持情况如下表：

| 运营商   | 目前2G/3G支持情况 | 备注                           |
| -------- | ----------------- | ------------------------------ |
| 中国移动 | 支持              | 正在逐步减少2G用户             |
| 中国联通 | 不支持            | 已经在2022年底完成2G网络的退网 |
| 中国电信 | 部分地区支持      | 计划在2024年全面停止2G网络     |
| 中国广电 | 不支持            | 不支持2G、3G网络               |

