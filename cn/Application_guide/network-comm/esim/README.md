#  eSIM

本文主要对eSIM功能进行简要的介绍，如果用户希望了解更详细的信息，可通过GSMA[官方网站](https://www.gsma.com/esim/)获取详细的协议文档。

在使用eSIM卡的LPA功能时，需要用户和卡商确认办理的eSIM卡是否支持LPA功能。

## eSIM背景知识

### 缩略词

| 名词缩写     | 英文                                    | 中文                   |
| :----------- | :-------------------------------------- | :--------------------- |
| eUICC        | embedded UICC                           | 嵌入式通用集成电路卡   |
| LPA          | Local Profile Assistant                 | 本地配置文件管理助手   |
| Profile      | Profile                                 | 配置文件               |
| SM-DP/SM-DP+ | Subscription Manager Data Preparation   | 签约管理数据准备服务器 |
| SM-SR        | Subscription Manager Secure Routing     | 签约管理路由服务器     |
| SM-DS        | Subscription Manager - Discovery Server | 签约管理发现服务器     |
| RSP          | Remote SIM Provisioning                 | 远程SIM配置            |
| MNO          | Mobile Network Operator                 | 移动网络运营商         |
| EUM          | eUICC Manufacturer                      | eUICC制造商            |



### 什么是eSIM

eSIM（Embedded SIM），即嵌入式SIM卡，是一种相对于传统SIM卡的新技术，其信息可以远程编程，无需物理更换即可更换运营商。这种SIM卡技术，将SIM卡信息打包成一个独立的文件，称之为profile，将profile文件下载到eSIM中即可成为一张完整可用的SIM卡。eSIM卡可以通过网络下载配置文件(Profile)，比较方便的切换运营商，这个特性随着全球化的发展，优势也越来越突出，对于eSIM的需求也越来越大。

### eSIM卡优势

eSIM卡一般直接焊接集成到移动终端设备中，根据GSMA开发标准，集成eSIM卡的移动终端可以下载存储多家运营商profile。用户可以根据需要选择启用其中一家运营商的profile，使移动终端可以通过选用的profile进行上网。

相比较传统SIM卡优势如下：

* 避免传统SIM卡插拔操作，在使用过程中更安全更稳定。

* eSIM卡体积更小，相比较传统的SIM卡，其应用场景更加广泛。

* 相比较传统SIM卡，不同运营商需要不同的SIM卡，而对于eSIM而言，仅仅需要下载运营商提供的数据文件即可，避免不必要的浪费。

* 在一些场景中，采用eSIM方式，产品更有竞争优势，因为其可以方便的切换运营商去适应不同的应用场景，而不必操作硬件。

### eSIM整体业务

对于eSIM整体业务流程框图下如：

![](../../media/network-comm/esim/8-3.png)

<center>图1-1</center>

说明:

* 用户通过个人电脑或者移动终端设备，接入网络，在企业eSIM管理平台或者直接在运营商eSIM管理平台(如果运营商有此类服务)购买eSIM服务。

* 运营商将相应的eSIM数据存入到SM-DP+服务器。

* 企业eSIM管理平台或运营商eSIM管理平台（如果运营商有此类服务）将下载profile所需参数下发到个人电脑或者移动终端设备。

* 通过个人电脑或移动终端设备，将profile数据从SM-DP+服务器中下载到用户设备中。

* 将下载成功的profile安装存储到eSIM卡中，然后启用下载安装成功的profile，即可完成eSIM卡接入网络。



### eSIM体系架构

分为两种解决方案架构：M2M方案、面向用户方案。

#### M2M方案

M2M解决方案是GSMA开发的第一个远程SIM卡供应解决方案。主要原因有两点：

* M2M解决方案更简单，其不需要和终端用户交互。

* 当前的商业需求是，技术解决方案支持B2B2C部署，同时满足推出eCall5等服务的监管要求。

M2M的远程SIM配置使用服务器驱动（推送模式）来配置和远程管理运营商配置文件。该解决方案围绕3个元素组织：SM-DP（Subscription Manager-Data Preparation）、SM-SR（Subscription Manager-Secure Routing）和eUICC。

下图1-2是M2M主要系统元素以及架构。除了常见的SIM功能，如SIM Toolkit6和承载独立协议（BIP7）之外，M2M解决方案对于启用eUICC没有对M2M设备施加额外的要求。

![](../../media/network-comm/esim/8-1.png)

<center>图1-2</center>

#### 面向用户方案

面向用户解决方案是在M2M解决方案提供的基础上开发的，用户通过终端设备来控制终端进行业务操作，对用户来说有更大的自由度，相应的这种解决方案也更复杂，考虑也需要更加的全面。GSM远程SIM配置用户解决方案遵循用户端驱动（pull模型），并允许用户控制运营商配置文件的远程供应和本地管理。

该解决方案围绕4个元素组织：SM-DP+（Subscription Manager-Data Preparation+）、SM-DS（Subscription Manager-Discovery Server）、LPA（Local Profile Assistant）和eUICC。

下图1-3为面向用户解决方案主要系统元素以及架构。

![](../../media/network-comm/esim/8-2.png)

<center>图1-3</center>

### eSIM下载profile流程

下图展示了从SM-DP+下载profile的交互流程。

![](../../media/network-comm/esim/8-4.png)

<center>图1-4</center>



### eSIM使用场景

* 可穿戴设备

由于可穿戴设备体积较小，选用eSIM卡可以有效的减小可穿戴设备体积，带来更好的用户体验。

* 车载

由于车辆活动范围较大，往往在不同的运营商服务范围内穿梭，需要方便的切换运营商。

* 跨境设备

设备移动跨境场景需要进行运营商网络切换，普通 SIM 卡无法满足，全球漫游卡资费又比较昂贵，通过eSIM 可以自由切换到当地运营商，而无需改动硬件。



## eSIM支持型号

QuecPython现支持eSIM功能具体型号如下表：

| 型号               |
| :----------------- |
| EC200AAU_HA_SANX   |
| EC200AAU_HA_SANXIR |
| EC200AEU_HA_SANX   |
| EC200AEU_HA_SANXIR |
| BG95M1             |
| BG95M8             |
| EC600UEC_AC_WS     |



## eSIM工具

###  工具介绍

QEsimTool软件主要用于处理eSIM空卡情况，QEsimTool软件需要从运营商服务器下载数据，需要个人电脑可以正常访问因特网。

QEsimTool软件的运行机制：用户从运营商获取AccCode，将AccCode作为参数传给QEsimTool，QEsimTool通过AccCode解析出运营商服务器地址，并向运营商服务器发送下载profile数据请求。当QEsimTool下载profile数据完成后，将通过AT指令将数据发送到设备中，进而将profile写入eSIM卡中。

QEsimTool软件获取方式：[下载](https://developer.quectel.com/wp-content/uploads/2024/09/QEsimTool_V1.0.1.7.zip)，联系客服进行领取。

###  界面介绍

QEsimTool软件附有应用指导文档，以下仅做简单介绍。

QEsimTool软件启动界面如下：

![](../../media/network-comm/esim/1-1.png)

<center>图1-5</center>

Port					：AT命令交互串口。

Baud rate		  ：串口波特率。

OPEN				 ：以指定的波特率打开Port。

AccCode			：AccCode需要运营商提供，填入QEsimTool中，作为参数。

DownloadProfile	：点击此按钮将解析AccCode，通过网络下载profile。

ListProfile		  ：查看当前profile列表。

ICCID				  ：eSIM卡的ICCID号码。

EnableProfile	：使能和ICCID对应的profile。

DisableProfile   ：禁用和ICCID对应的profile。

DeleteProfile	 ：删除和ICCID对应的profile。



###  使用说明

用户可以按照如下步骤进行操作：

* 正常启动模组设备后，在个人电脑中打开QEsimTool软件。

* 检查个人电脑是否可以正常访问因特网。

* Port串口选择AT口，设置波特率并点击图1-5中的open按钮。

* 填写运营商提供的AccCode。

连接设备成功后，执行ATI指令，如下图所示正确显示出版本信息，则可以执行点击DownloadProfile按钮进行下载安装profile操作。

![](../../media/network-comm/esim/1-2.png)
<center>图1-6</center>

> 如在软件使用过程中遇见异常，请将QuecWinLpa.log提供给工程师进行分析。



## eSIM API介绍

以下接口均是以LPA方式实现，需要用户和卡商确认eSIM卡是否支持LPA功能。

###  获取EID

用户可以通过如下按钮获取eSIM卡的ICCID。

接口如下：

```python
esim.getEid()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
from sim import esim

eid=esim.getEid()
print("Get Eid :{}".format(eid))

```



###  回调注册

用户可以通过如下接口注册回调函数，用于通知应用层profile下载安装结果。

接口如下：

```python
esim.setCallback(usrFun)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
from sim import esim

def usrFun(result):
    '''
    ：param result	：OTA下载profile结果。
    ：type  result   ：整型，0：成功   1：失败
    '''
    print("OTA result:{}".format(result))

esim.setCallback(usrFun)

```



###  查询当前SIM卡profile

用户可通过如下接口获取当前eSIM卡中的profile信息，用户可以根据查询到的profile信息，选择启用相应的运营商网络功能。

接口如下：

```python
esim.getProfileInfo(mode)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
from sim import esim

# 列出配置文件中类别为2的配置文件
profile_list=esim.getProfileInfo(0)
print("Get class 2 profile :{}".format(profile_list))

#列出所有的配置文件
profile_list=esim.getProfileInfo(1)
print("Get all profile :{}".format(profile_list))

```

###  下载安装运营商profile

用户可通过如下接口下载安装运营商的profile。如eSIM为空卡，请使用QEsimTool工具进行安装首个profile。为确保此接口可以正常下载profile信息，需eSIM卡当前启用的profile能够正常注网，并且可以正常和AccCode中的运营商服务器地址进行网络数据交互。在此接口调用后，必须等下载结果通知到达后才能调用eSIM接口进行别的操作，否则将影响下载安装结果。

接口如下：

```python
esim.profileOTA(activationCode, confirmationCode)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
from sim import esim
def usrFun(result):
    '''
    ：param result	：OTA下载profile结果。
    ：type  result   ：整型，0：成功   1：失败
    '''
    print("OTA result:{}".format(result))

esim.setCallback(usrFun)

# activationCode请使用自己的数据，下述activationCode仅作示例参数。
esim.profileOTA('LPA:XXX','')

```



###  控制当前SIM卡profile

用户可通过如下接口对指定的ICCID对应的profile进行启用、禁用、删除操作。

接口如下：

```python
esim.profileHandle(profile_tag, iccid)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例，此示例要求eSIM卡中必须有profile

from sim import esim

def handle_profile(ICCID,mode):
    esim.profileHandle(mode, ICCID)

# 获取当前profile ICCID
profile_list=list(esim.getProfileInfo(0))

# 更改profile的状态
if profile_list[1][0][1] == 1:
    print("ICCID {} 已启用".format(profile_list[1][0]))
    # disable this ICCID
    esim.profileHandle(1, profile_list[1][0][0])
elif profile_list[1][0][1] == 0:
    print("ICCID {} 已禁用".format(profile_list[1][0]))
    # enable this ICCID
    esim.profileHandle(0, profile_list[1][0][0])
else:
    raise ValueError("profile_list is invalid")

# 删除profile
esim.profileHandle(2, profile_list[1][0][0])

```



###  获取待通知事件

用户可通过如下接口查询本地已删除，但并未在服务器完成删除操作的profile信息。

接口如下：

```python
esim.getProfileDelNotification()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例，此示例要求eSIM卡中必须有profile

from sim import esim

del_error_profile=esim.getProfileDelNotification()
print("本地已删除，未能正常上报服务器的profile信息: {}".format(del_error_profile))

```



###  上报通知事件

用户可通过如下接口再次向服务器发起删除指定profile的请求，以确保服务端完成对profile信息的删除操作。

此函数参数需通过`esim.getProfileDelNotification()`获取。

接口如下：

```python
esim.reportProfileDelNotification(iccid)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例，此示例要求eSIM卡中必须有profile

from sim import esim

del_error_profile=esim.getProfileDelNotification()
print("本地已删除，未能正常上报服务器的profile信息: {}".format(del_error_profile))

# 发起请求服务端删除此ICCID对应的profile
esim.reportProfileDelNotification(profile_list[1][0][0])

```

## eSIM应用示例

* 用户从运营商购买eSIM卡。
* 用户用一段时间后，想切换到别的运营商。

为解决上述问题，用户应用示例如下：

```python
# -*- coding: UTF-8 -*-
#示例，此示例要求eSIM卡中必须有profile

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
    # 先下载安装将要切换的运营商profile信息
    # 下述activationCode仅作示例参数
    esim_t.profile_ota('LPA:XXX','')

    #等待下载结果
    esim_t.wait_ota_result()

    # 删除原来的运营商profile信息
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



## 常见问题解答

### 无法下载安装profile

* AccCode是否错误，可向运营商进行确认。
* 当前的设备或PC机是否可以正常访问从AccCode中解析出的url地址。
* 是否重复下载安装profile。
* 是否运营商进行限制，同一个AccCode只能使用一次，需和运营商进行确定。
* 网络信号是否太弱，网络不稳定。

### 下载安装profile后无法上网

如遇到下载安装profile成功后无法上网，请做一次CFUN=0/1的切换操作。如依然无法上网，请按如下方面进行检查。

* SIM卡状态是否异常
* 下载安装的profile是否已正常激活。
* 下载安装的profile是否有效，可向运营商确认。
* 是否设备未执行拨号操作（如设置设备不自动激活，请检查是否未执行激活操作）

### 提示eSIM卡未插卡

请参考QuecPython官网wik蜂窝无线网卡[网络异常处理](../nic/cellular/exception-handling.md)。

