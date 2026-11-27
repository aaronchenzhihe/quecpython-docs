#  SIM卡API说明

##  查询SIM卡信息

### 查询IMSI

用户可通过如下接口获取SIM卡的IMSI号码，以方便用户对IMSI的应用。

接口如下：

```python
sim.getImsi()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

imsi = sim.getImsi()
print("Get IMSI : {}".format(imsi))

```

### 查询ICCID

用户可通过如下接口查询SIM卡的ICCID，以方便用户对SIM卡的ICCID的应用。

接口如下：

```python
sim.getIccid()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

iccid = sim.getIccid()
print("Get IMSI : {}".format(iccid))

```



### 查询电话号码

用户可通过如下接口查询SIM卡电话号码，SIM电话号码需要先设置后查询。

接口如下：

```python
sim.getPhoneNumber()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

phone_num = sim.getPhoneNumber()
print("Get Phone Number is : {}".format(phone_num))

```

### 查询信息应用场景

应用程序启动后，等待注网成功，然后查询SIM卡信息，为后续处理业务做准备。

```python
# -*- coding: UTF-8 -*-
#示例
import sim
import checkNet
import sys

sim_status_dict={
    0:"SIM卡不存在/被移除",
    1:"SIM已经准备好",
    2:"SIM卡已锁定，等待CHV1密码",
    3:"SIM卡已被阻拦，需要CHV1密码解锁密码",
    4:"由于SIM/USIM个性化检查失败，SIM卡被锁定",
    5:"由于PCK错误导致SIM卡被阻拦，需要MEP密码解除阻拦",
    6:"需要隐藏电话簿条目的密钥",
    7:"需要解锁隐藏密钥的编码",
    8:"SIM卡已锁定，等待CHV2密码",
    9:"SIM卡被阻拦，需要CHV2解锁密码",
    10:"由于网络个性化检查失败，SIM卡被锁定",
    11:"由于NCK不正确，SIM卡被阻拦，需要MEP解锁密码",
    12:"由于子网络锁个性化检查失败，SIM卡被锁定",
    13:"由于错误的NSCK，SIM卡被阻拦，需要MEP解锁密码",
    14:"由于服务提供商个性化检查失败，SIM卡被锁定",
    15:"由于SPCK错误，SIM卡被阻拦，需要MEP解锁密码",
    16:"由于企业个性化检查失败，SIM卡被锁定",
    17:"由于CCK不正确，SIM卡被阻止，需要MEP解锁密码",
    18:"SIM正在初始化，等待完成",
    19:"CHV1/CHV2/PIN错误",
    20:"SIM卡无效",
    21:"未知状态"
}

if __name__ == '__main__':
	
    # 1、查询SIM卡状态
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("接口返回失败")
       	sys.exit()
    print("Get SIM  status : {}".format(sim_status_dict[sim_status]))
    if sim_status != 1:
        sys.exit()
    
    # 2、等待注网成功    
    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1:
        print('Network connection successful.')
    else:
        print('Network connection failed, stage={}, state={}'.format(stage, state))
        sys.exit()
    
    # 3、查询开机注网后的SIM卡信息
    
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



##  查询SIM卡状态

用户可通过如下接口获取当前SIM卡的状态，通过SIM卡状态判断SIM卡是否可以正常使用。

接口如下：

```
sim.getStatus()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim
import sys

sim_status_dict={
    0:"SIM卡不存在/被移除",
    1:"SIM已经准备好",
    2:"SIM卡已锁定，等待CHV1密码",
    3:"SIM卡已被阻拦，需要CHV1密码解锁密码",
    4:"由于SIM/USIM个性化检查失败，SIM卡被锁定",
    5:"由于PCK错误导致SIM卡被阻拦，需要MEP密码解除阻拦",
    6:"需要隐藏电话簿条目的密钥",
    7:"需要解锁隐藏密钥的编码",
    8:"SIM卡已锁定，等待CHV2密码",
    9:"SIM卡被阻拦，需要CHV2解锁密码",
    10:"由于网络个性化检查失败，SIM卡被锁定",
    11:"由于NCK不正确，SIM卡被阻拦，需要MEP解锁密码",
    12:"由于子网络锁个性化检查失败，SIM卡被锁定",
    13:"由于错误的NSCK，SIM卡被阻拦，需要MEP解锁密码",
    14:"由于服务提供商个性化检查失败，SIM卡被锁定",
    15:"由于SPCK错误，SIM卡被阻拦，需要MEP解锁密码",
    16:"由于企业个性化检查失败，SIM卡被锁定",
    17:"由于CCK不正确，SIM卡被阻止，需要MEP解锁密码",
    18:"SIM正在初始化，等待完成",
    19:"CHV1/CHV2/PIN错误",
    20:"SIM卡无效",
    21:"未知状态"
}

if __name__ == '__main__':
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("接口返回失败")
        sys.exit()
    if sim_status != 1:
        print("Get SIM status status : {}".format(sim_status_dict[sim_status]))
        sys.exit()
    print("Get sim_status is : {}".format(sim_status_dict[sim_status]))

```



##  PIN码验证功能

用户可通过开启PIN码验证功能来保证SIM卡的使用安全。PIN码是SIM卡的安全识别码，用于验证用户的身份和保护个人信息安全。开启PIN码验后，每次重启设备后（或CFUN0/1切换后）均需进行PIN码验证，在验证通过后SIM卡才能正常使用。PIN码验证连续3次输入错误后SIM卡将被锁定，需要用PUK码进行解码。

### 开启PIN验证

用户可通过如下接口开启SIM卡PIN码验证功能，用以保证SIM卡的使用安全。

接口如下：

```python
sim.enablePin(pin)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

result = sim.enablePin("1234")
print("Enable PIN : {}".format(result))

```



### 关闭PIN验证

用户可通过如下接口关闭PIN码验证功能，在保证SIM卡是在安全的环境下使用时，可以关闭PIN码验证功能来简化应用程序。

接口如下：

```python
sim.disablePin(pin)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

#根据实际PIN码输入，此处"1234"为示例PIN码
result = sim.disablePin("1234")
print("Disable PIN : {}".format(result))

```



### PIN验证

用户可通过如下接口来完成SIM卡的PIN码验证，开启PIN码验证后，SIM卡需完成PIN码验证后才能将状态将切换到正常状态，之后应用程序才能进行业务处理。

接口如下：

```python
sim.verifyPin(pin)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

#根据实际PIN码输入
result = sim.verifyPin("1234")
print("Verify PIN result is : {}".format(result))
```



### 修改PIN

用户可定期通过如下接口修改SIM卡的PIN码，提高SIM卡使用的安全性。

接口如下：

```python
sim.changePin(oldPin, newPin)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

sim.changePin("1234", "4321")
```

### PIN码应用场景

用户为了安全应用SIM卡，需要在开启PIN码验证功能的情况下进行业务处理。

```python
# -*- coding: UTF-8 -*-
#示例
import sim
import checkNet
import sys

sim_status_dict={
    0:"SIM卡不存在/被移除",
    1:"SIM已经准备好",
    2:"SIM卡已锁定，等待CHV1密码",
    3:"SIM卡已被阻拦，需要CHV1密码解锁密码",
    4:"由于SIM/USIM个性化检查失败，SIM卡被锁定",
    5:"由于PCK错误导致SIM卡被阻拦，需要MEP密码解除阻拦",
    6:"需要隐藏电话簿条目的密钥",
    7:"需要解锁隐藏密钥的编码",
    8:"SIM卡已锁定，等待CHV2密码",
    9:"SIM卡被阻拦，需要CHV2解锁密码",
    10:"由于网络个性化检查失败，SIM卡被锁定",
    11:"由于NCK不正确，SIM卡被阻拦，需要MEP解锁密码",
    12:"由于子网络锁个性化检查失败，SIM卡被锁定",
    13:"由于错误的NSCK，SIM卡被阻拦，需要MEP解锁密码",
    14:"由于服务提供商个性化检查失败，SIM卡被锁定",
    15:"由于SPCK错误，SIM卡被阻拦，需要MEP解锁密码",
    16:"由于企业个性化检查失败，SIM卡被锁定",
    17:"由于CCK不正确，SIM卡被阻止，需要MEP解锁密码",
    18:"SIM正在初始化，等待完成",
    19:"CHV1/CHV2/PIN错误",
    20:"SIM卡无效",
    21:"未知状态"
}

if __name__ == '__main__':
    
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("接口返回失败")
       	sys.exit()
    
    if sim_status == 2:
        # 开启了PIN码验证，需要验证PIN码
        PIN="1234"          #PIN码-请根据SIM卡真实的PIN码填写
        pin_result = sim.verifyPin(PIN)
        if pin_result != 0:
            print("Error PIN ! Please input right PIN!")
            sys.exit()
        
        # 验证成功后关闭PIN码验证，下次不再需要进行PIN码验证
        # result = sim.disablePin(PIN)
        #if result != 0:
        #    print("Error PIN ! Please input right PIN!")
        #    sys.exit()
    
    # 查询SIM状态
    sim_status = sim.getStatus()
    if sim_status not in sim_status_dict:
        print("接口返回失败")
       	sys.exit()
    if sim_status != 1:
        print("Get SIM status : {}".format(sim_status_dict[sim_status]))
        sys.exit()
    
    print("Get sim_status is : {}".format(sim_status_dict[sim_status]))
    
    #等待注网成功，进行业务处理
    stage,state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1:
        print('Network connection successful.')
    else:
        print('Network connection failed, stage={}, state={}'.format(stage, state))
        
```



##  SIM卡解锁

用户可通过如下接口处理连续3次输入PIN码错误导致锁PIN的情况。这个接口在用PUK码解锁的同时重新设置了PIN码。需要注意PUK码错误10次后SIM卡将被永久锁定，自动报废。

接口如下：

```
sim.unblockPin(puk, newPin)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

sim.unblockPin("12345678", "0000")

```

## 电话薄功能

用户在实际项目应用过程中，如需保存对端电话号码，需要用到电话薄功能。此接口可以提供保存电话号码信息的功能。此功能接口支持情况请参考QuecPython wiki[电话薄功能](../../../api_reference/iotlib/sim.md#电话簿功能)

### 读取电话薄

用户可通过如下接口读取电话薄。其中电话号码存储位置参考QuecPython官网wiki中`sim.readPhonebook()`[接口参数描述](../../../api_reference/iotlib/sim.md#sim.readPhonebook)。需注意start-end差值需要小于等于20。

接口如下：

```python
sim.readPhonebook(storage, start, end, username)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

# 读取本SIM卡电话薄中索引值1到4的电话信息
sim_phone_boo = sim.readPhonebook(9, 1, 4, "")
print("SIM卡中电话薄 : {}".format(miss_call))

# 读取未接来电中索引值1到4的电话信息
miss_call = sim.readPhonebook(4, 1, 4, "")
print("未接来电 : {}".format(miss_call))

```



### 设置电话薄

用户可通过如下接口来保存电话号码信息到电话簿中：

```
sim.writePhonebook(storage, index, username, number)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

# 保存Tom电话号码保存到SIM卡电话薄索引位置1的位置上
sim.writePhonebook(9, 1, 'Tom', '18144786859')

```

### 电话薄应用场景

在项目运行中，获取到电话号码，并保存电话号码到SIM卡电话薄中。

```python
# -*- coding: UTF-8 -*-
#示例
import sim

def save_phone_number_to_simbook(index,name,phonenumber,input_type = 0 ):
    # type = 0，如果设置保存电话号码位置已经存在电话号码，直接返回，不覆盖保存
    if input_type == 0:
        # 查询当前位置是否已有电话号码信息
        phone_info = sim.readPhonebook(9, index, index, name)
        if type(phone_info).__name__ == 'int':
            print("不存在")
        elif phone_info[1][0][1] == name and phone_info[1][0][2] == phonenumber:
            # 电话号码已存在
            print("已存在")
            return
        else:
            print("不存在")
    else:
        pass
    # 保存姓名，电话号码到SIM卡电话薄中index地址处        
    result = sim.writePhonebook(9, index, name, phonenumber)

def get_name_phone():
    return 'Tom','18144786858'

if __name__ == '__main__':
    name,phonenumber = get_name_phone()

    # 姓名电话号码已存在则不保存
    save_phone_number_to_simbook(1,name,phonenumber,0)

    #姓名电话号码已存在覆盖保存
    save_phone_number_to_simbook(1,name,phonenumber,1)

```



##  SIM卡热插拔

在项目中，用户如需监测SIM卡插拔情况，可以用如下接口注册回调函数。当SIM卡插入或者拔出时，系统将会调用如下接口注册的回调函数，回调函数会上送SIM卡插入或者拔出的事件给应用层。用户可以根据通知事件来判断SIM卡是插入还是拔出。QuecPython模组对此接口支持情况请参考QuecPython官网wiki[热插拔功能](../../../api_reference/iotlib/sim.md#热插拔功能)

### 注册热插拔回调

用户可通过如下接口注册SIM卡热插拔回调函数。

接口如下：

```python
sim.setCallback(usrFun)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

def sim_hot_call(event):
    if event == 1:
        # SIM 卡插入
        print("Get SIM Inster event : {}".format(event))
    elif event == 2:
        # SIM 卡拔出
        print("Get SIM Unplug event : {}".format(event))
    else:
        # 未知事件
        print("Get SIM Unkown event : {}".format(event))

sim.setCallback(sim_hot_call)

```



### 获取SIM卡热插拔配置

用户可通过如下接口查询SIM卡热插拔功能的当前配置信息，根据配置信息可以判断当前是否开启了热插拔功能。

接口如下：

```python
sim.getSimDet()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

sim_hot_conf = sim.getSimDet()
print("Get Current SIM config info : {}".format(sim_hot_conf))
if sim_hot_conf[0] == 1:
    # SIM卡热插拔功能启用
    print("Current SIM Hotplug is enable")
```



### 配置SIM卡热插拔

用户可通过如下接口启用或者关闭SIM卡热插拔功能。

接口如下：

```python
sim.setSimDet(switch, triggerLevel)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

# 先判断SIM卡插入时，电平是高电平还是低电平，在此以SIM插入时为高电平为例

# 1、注册热插拔回调函数
def sim_hot_call(event):
    print("Get SIM event : {}".format(event))
    
sim.setCallback(sim_hot_call)

sim_hot_conf = sim.getSimDet()
# 2、开启SIM卡热插拔功能并标记模组SIM卡插入时，电平为高电平
if sim_hot_conf[0] == 0:
    print("SIM hot-swapping is not currently enabled")
    print("Will enable this function")
    sim.setSimDet(1, 1)
else:
  	print("Already enbale this function")
    
```

### SIM卡热插拔应用场景

用户在项目中，检测SIM卡插拔事件，并设置标识，标记发生了SIM卡插入或者拔出事件。（用户可以根据需要决定是否通知应用程序中其他模块发生了SIM卡插拔事件）。

```python
# -*- coding: UTF-8 -*-
#示例
import sim
import net
insert_flag = False
def sim_hot_call(event):
    global insert_flag
    print("Get SIM event : {}".format(event))
    if event == 1:
        #检测到SIM卡插入，设置标记SIM卡被插入
        insert_flag = True
    else:
        insert_flag = False

if __name__ == '__main__':
    sim_hot_conf = sim.getSimDet()
    if sim_hot_conf[0] == 0:
    
        #为开启SIM卡热插拔功能,在位电平标记请根据实际SIM卡在位电平设置
        #此处以SIM卡在为电平为1为例
        print("enable this function")
        sim.setSimDet(1, 1)
    
    else:
        print("Already enbale this function")
    
    #设置回调函数检测SIM卡的插拔动作
    sim.setCallback(sim_hot_call)

```



## 双卡切换

用户在项目应用过程中，为了网络稳定，满足多种网络场景，需要两张不同SIM卡可以随时切换的功能，以应对各种网络异常情况。QuecPython模组对于此功能支持情况请参考QuecPython官网wiki中[SIM卡切卡功能](../../../api_reference/iotlib/sim.md#SIM卡切卡功能)。

### 注册切卡回调函数

用户可通过如下接口注册回调函数监测切卡事件。当发生了切卡事件，系统将通过注册的回调函数发出切卡事件通知应用层，应用层收到切卡事件通知后，将进行切卡导致的网络异常情况的处理。

接口如下：

```python
sim.setSwitchcardCallback(usrFun)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim
import net

sim_status_dict={
    0:"SIM卡不存在/被移除",
    1:"SIM已经准备好",
    2:"SIM卡已锁定，等待CHV1密码",
    3:"SIM卡已被阻拦，需要CHV1密码解锁密码",
    4:"由于SIM/USIM个性化检查失败，SIM卡被锁定",
    5:"由于PCK错误导致SIM卡被阻拦，需要MEP密码解除阻拦",
    6:"需要隐藏电话簿条目的密钥",
    7:"需要解锁隐藏密钥的编码",
    8:"SIM卡已锁定，等待CHV2密码",
    9:"SIM卡被阻拦，需要CHV2解锁密码",
    10:"由于网络个性化检查失败，SIM卡被锁定",
    11:"由于NCK不正确，SIM卡被阻拦，需要MEP解锁密码",
    12:"由于子网络锁个性化检查失败，SIM卡被锁定",
    13:"由于错误的NSCK，SIM卡被阻拦，需要MEP解锁密码",
    14:"由于服务提供商个性化检查失败，SIM卡被锁定",
    15:"由于SPCK错误，SIM卡被阻拦，需要MEP解锁密码",
    16:"由于企业个性化检查失败，SIM卡被锁定",
    17:"由于CCK不正确，SIM卡被阻止，需要MEP解锁密码",
    18:"SIM正在初始化，等待完成",
    19:"CHV1/CHV2/PIN错误",
    20:"SIM卡无效",
    21:"未知状态"
}

def switch_sim_call(result):
    print("Get switch SIM card event : {}".format(reslut))
    if result == 7:
        # SIM卡切卡成功，通知应用层，SIM卡切卡成功
        print("SIM switch success!")
    elif result == 9:
        print("SIM switch failed!")

if __name__ == '__main__':
    
    sim.setSwitchcardCallback(switch_sim_call)

```



### 查询当前 SIM卡ID

用户可通过如下接口查询当前启用的SIM卡ID。

接口如下 ：

```python
sim.getCurSimid()
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

simid = sim.getCurSimid()
print("Current SIM is : {}".format(simid))
```



### 切换SIM卡

用户可通过如下接口切换到指定SIM卡。如当前SIM卡出现不可恢复的异常网络情况，可用此接口将正在使用的SIM卡切换到另一张SIM卡，使应用程序可以继续处理业务。

接口如下：

```python
sim.switchCard(simId)
```

示例

```python
# -*- coding: UTF-8 -*-
#示例
import sim

simid = sim.getCurSimid()
print("Current SIM is : {}".format(simid))

if simid == 0:
    # 启用SIM2
    print("Current is SIM 1 and Will switch to SIM2")
    sim.switchCard(1)
    
```

### 双卡切换应用场景

以下场景根据客户需求和业务流程，可能需要切卡操作。

*  检测到当前卡信号弱满足不了业务需求就尝试切到另一张卡， 比如单车,对讲机。
*  一张卡接入公网，一张卡接入专网，公网卡主要做数据业务, 专网卡处理一些特殊任务， 比如电网客户。

* 用户应用程序在启动时检测到当前SIM卡状态无效，无法进行网络交互，切换到另一张SIM卡。

```python
# -*- coding: UTF-8 -*-
#示例
import sim
import net
import sys

sim_status_dict={
    0:"SIM卡不存在/被移除",
    1:"SIM已经准备好",
    2:"SIM卡已锁定，等待CHV1密码",
    3:"SIM卡已被阻拦，需要CHV1密码解锁密码",
    4:"由于SIM/USIM个性化检查失败，SIM卡被锁定",
    5:"由于PCK错误导致SIM卡被阻拦，需要MEP密码解除阻拦",
    6:"需要隐藏电话簿条目的密钥",
    7:"需要解锁隐藏密钥的编码",
    8:"SIM卡已锁定，等待CHV2密码",
    9:"SIM卡被阻拦，需要CHV2解锁密码",
    10:"由于网络个性化检查失败，SIM卡被锁定",
    11:"由于NCK不正确，SIM卡被阻拦，需要MEP解锁密码",
    12:"由于子网络锁个性化检查失败，SIM卡被锁定",
    13:"由于错误的NSCK，SIM卡被阻拦，需要MEP解锁密码",
    14:"由于服务提供商个性化检查失败，SIM卡被锁定",
    15:"由于SPCK错误，SIM卡被阻拦，需要MEP解锁密码",
    16:"由于企业个性化检查失败，SIM卡被锁定",
    17:"由于CCK不正确，SIM卡被阻止，需要MEP解锁密码",
    18:"SIM正在初始化，等待完成",
    19:"CHV1/CHV2/PIN错误",
    20:"SIM卡无效",
    21:"未知状态"
}

def switch_sim_call(result):
    print("Get switch SIM card event : {}".format(reslut))
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
        print("接口返回失败")
       	sys.exit()
    if sim_status != 20:
        # sim卡状态无效
        print("Get SIM status status : {}".format(sim_status_dict[sim_status]))
        
        #查询当前使用SIM卡
        sim_id = sim.getCurSimid()
        
        #切换到另一张卡上运行
        if sim_id == 0:
            sim.switchCard(1)
        else:
            sim.switchCard(0)

```

