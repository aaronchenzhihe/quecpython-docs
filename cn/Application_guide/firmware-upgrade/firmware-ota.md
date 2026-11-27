# 固件 OTA 升级

## 概述

OTA（Over The Air）为空中下载技术。OTA升级则是通过无线网络下载、升级设备所需要的软件更新，比如问题修复、版本更新等，不用通过有线连接来下载和升级。

## OTA 升级分类

模组 OTA 升级主要由无线网络下载升级包、升级包内容写入flash存储、升级包校验、重启升级等部分构成。按照技术实现可以分为全量升级、差分升级和最小系统升级。

### 全量升级

全量升级将使用目标版本的完整内容替换设备中当前版本的所有内容。

为了在软件流程上兼容后面将要提到的差分升级，全量升级也不是简单的将原始升级包放在云端共设备下载即可。它也需要按照一定的格式，将原始数据增加包头、包尾、校验数据等。我们将这个新的数据包称之为全量包。

### 差分升级

差分升级将设备中的当前版本和目标版本之间的差异提取出来，该差异文件下载到设备中后，使用差分还原算法，以打 patch 的方式将差异数据作用到当前版本中，进而还原出目标版本的数据。这个差异文件，我们称之为差分包。

该过程整体来说分为 4 个阶段：

- 差分包制作：差分数据的提取和压缩
- 差分包上传到云端
- 差分包下载到设备
- 差分包解压和数据还原

### 最小系统升级

最小系统升级结合了差分升级和全量升级2种方式，属于特殊定制的方案。主要针对 Flash 大小有限，没有足够空间存放升级包文件的设备。制作升级包时，会生成两个升级包文件，并需要进行多次自动重启。

> 三种升级方式的对比:
>
> - 全量升级包占用的存储空间比差分升级大，因而下载时消耗的流量比差分升级多。
>
> - 全量升级的速度比差分升级快。
>
> - 全量升级不考虑当前运行的版本，可以升级到任意其它版本；而差分升级只能升级到某个特定版本，而且还要维护各个不同版本之间的差分包和目标固件包，固件维护稍显麻烦。
>
> - 最小系统升级的特性介于全量升级和差分升级之间。

## QuecPython OTA方案

### 系统架构

移远通信的模组普遍由多个分区组成，分别用来存储程序镜像、出厂参数、文件系统等。

即使有一些分区数据发生了改变，由于全量升级是覆盖式升级，因此仍然可以使用全量升级的方式。数据最容易发生改变的就是文件系统。

对于差分升级，是通过在当前版本的数据上生成 patch，并将 patch 作用在设备中实现升级的，因此只能对数据不会发生改变的分区进行差分升级。对于程序镜像、出厂参数等，出厂后不使用特殊工具无法进行修改，这些分区可以使用差分升级。

QuecPython分区表构成细节可以参见[QuecPython存储设备介绍](../system/memory-device.md)。

### 方案设计

固件程序镜像占用的存储空间较大，而设备的存储空间有限，因此固件采用的是差分升级方案。而文件系统分区在设备启动后内容会发生变动无法作差分升级，因此用户应用脚本只能作全量升级。进而导致固件和应用脚本无法同时升级。所以，QuecPython OTA升级也分为固件升级和APP升级。实际使用过程中，为了实现远程控制设备自动进行OTA升级，通常都要结合云平台实现设备端加云端的一整套OTA方案，如下框图。

![](../media/firmware-upgrade/firmware-ota/OTA_cloud.jpg)

### 固件升级步骤

#### 设备端准备 - 固件升级

##### 固件准备

需要准备新、旧两个版本的固件，其中旧版本的固件要和烧录到模组内的固件保持一致，否则会导致升级失败。

##### 差分包准备

不同型号设备制作方法会有差异，具体参见[固件OTA升级包制作指引](./firmware-ota-delta-package-making.md)。

##### 代码准备

###### 1. 初始化云平台功能

OTA云平台的作用是：

​   a. 存放升级包文件。

​   b. 进行消息通知，如开始升级。

​   c. 升级状态管理，如升级成功或者失败。

借助于OTA云平台可实现网页控制OTA自动升级。使用前需要先初始化云平台相关的功能。API细节参考[QuecPython IoT 平台](../../api_reference/cloudlib/README.md)。

###### 2. 接收云平台升级消息

用户在云平台网页触发OTA升级，设备会收到消息通知，从消息中获取到升级包URL。在前述初始化云平台功能后，订阅OTA升级相关的主题即可接收到云平台发送的消息通知。

###### 3. 下载升级包

获取到升级包URL后，设备则调用固件升级相关API进行升级包下载、写入及校验。API细节参考[fota - 固件升级相关功能](../../api_reference/syslib/fota.md)。

###### 4. 重启

升级包校验成功后，需重启模块（自动重启或手动重启），系统在设备重启之后自动进行OTA升级。

###### 5. 运行新固件

升级完成后自动运行新固件，将当前版本号上报到云平台，云平台判断是否升级成功。

#### 云端准备 - 固件升级

##### 阿里云

如果使用阿里云平台的OTA功能，首先需要接入阿里云，设备如何接入阿里云可以参考[aLiYun - 阿里 IoT 平台](../../api_reference/cloudlib/aLiYun.md)。接入阿里云之后如何使用云平台OTA功能可以参见[阿里云物联网平台OTA升级操作相关文档](https://help.aliyun.com/document_detail/130990.html?spm=a2c4g.130990.0.0.28722110xYe5Td)。

这里以阿里云为例展示OTA平台使用操作步骤：

1. 创建OTA模块，以设备平台名称命名，如: `EC600N-CNLC`

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_fota_module.png)

2. 创建OTA升级包

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_fota_upgrade_package.png)

3. 选择批量升级, 创建升级计划

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_fota_plain.png)

4. 等待设备升级，查看升级结果

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_fota_upgrade_process.png)

##### 腾讯云

如果使用腾讯云平台的OTA功能，首先需要接入腾讯云，设备如何接入腾讯云可以参考[TenCentYun- 腾讯 IoT 平台](../../api_reference/cloudlib/TenCentYun.md)。接入腾讯云之后如何使用云平台OTA功能可以参见[腾讯云物联网开发平台固件升级协议](https://cloud.tencent.com/document/product/1081/39359)、[腾讯云物联网开发平台固件升级操作](https://cloud.tencent.com/document/product/1081/40296)。

### APP升级使用步骤

用来升级QuecPython设备客户应用脚本，包括python文件、json文件及音频文件等。采用的升级方案为全量升级，即下载目标文件之后，替换设备中原有的文件。

#### 设备端准备 - APP升级

##### 升级包准备

由于是全量升级，只需要将待升级的目标文件作为升级包。

##### 代码准备

###### 1. 初始化云平台功能

借助于OTA云平台可实现网页控制OTA自动升级。使用前需要先初始化云平台相关的功能。API细节参考 [QuecPython IoT 平台](../../api_reference/cloudlib/README.md)。

###### 2. 接收云平台升级消息

用户在云平台网页触发OTA升级，设备会收到消息通知，从消息中获取到升级包url。在前述初始化云平台功能后，订阅OTA升级相关的主题即可接收到云平台发送的消息通知。

###### 3. 下载升级包

获取到升级包URL后，设备根据URL下载待升级的目标文件到文件系统中。支持单文件下载和多文件批量下载方式。API细节参考[app_fota - 用户文件升级相关功能](../../api_reference/syslib/app_fota.md)。

###### 4. 设置升级标志

设置升级标志记录是否要进行升级。

###### 5. 重启

重启设备（自动重启或手动重启），系统在设备重启之后自动进行OTA升级。

###### 6. 运行新固件

升级完成后会清除升级标志，进入新的应用程序。将当前版本号上报到云平台，云平台判断是否升级成功。

#### 云端准备 - APP升级

##### 阿里云

参考固件升级配置方法。

以阿里云为例展示OTA平台使用操作步骤：

1. 创建OTA模块，以`PROJECT_NAME`命名，如: `QuecPython-Tracker`

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_sota_module.png)

2. 创建OTA升级包，将需要升级的项目文件后缀名修改为`.bin`。此处需要在**推送给设备的自定义信息**中编写升级文件名对应的设备全路径文件名, 如: `{"files":{"common.bin":"/usr/modules/common.py","settings.bin":"/usr/settings.py","test_tracker.bin":"/usr/test_tracker.py"}}`。

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_sota_upgrade_package.png)

3. 选择批量升级, 创建升级计划

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_sota_plain.png)

4. 等待设备升级，查看升级结果

![](../media/firmware-upgrade/firmware-ota/aliyun_ota_sota_upgrade_process.png)

##### 腾讯云

参考固件升级配置方法。

## 示例代码

### 阿里云 - 示例代码

```python
import uos
import fota
import app_fota
import utime
import modem
import ujson
from misc import Power
from aLiYun import aLiYun
import _thread

# 定义软件名称
PROJECT_NAME = "EG915U-XXXX"
# 定义软件版本
PROJECT_VERSION = "1.0.1"
# 获取固件型号
FIRMWARE_NAME = uos.uname()[0].split("=")[1]
# 获取固件版本号
FIRMWARE_VERSION = modem.getDevFwVersion()

# 初始化aLiYun功能
ProductKey = "XX"
ProductSecret = None
DeviceName = "XX"
DeviceSecret = "XX"
MqttServer = "XX"
cloud = aLiYun(ProductKey, ProductSecret, DeviceName, DeviceSecret, MqttServer)

# 初始化OTA相关Topic
# 设备模块版本信息上报topic
ota_topic_device_inform = "/ota/device/inform/%s/%s" % (ProductKey, DeviceName)
# 设备OTA升级计划下发topic
ota_topic_device_upgrade = "/ota/device/upgrade/%s/%s" % (ProductKey, DeviceName)
# 设备升级进度上报topic
ota_topic_device_progress = "/ota/device/progress/%s/%s" % (ProductKey, DeviceName)
# 设备OTA升级计划查询topic
ota_topic_firmware_get = "/sys/%s/%s/thing/ota/firmware/get" % (ProductKey, DeviceName)
# 设备OTA升级计划查询应答topic
ota_topic_firmware_get_reply = "/sys/%s/%s/thing/ota/firmware/get_reply" % (ProductKey, DeviceName)

# 设置MQTT连接
client_id = modem.getDevImei()
clean_session = True
cloud.setMqtt(client_id, clean_session)

ota_module = None
fota_data = ''
fota_topic = ''
fota_flag = False
chafen_url=''
ota_data=''
chafen_url1=''
chafen_url2=''

# 订阅Topic回调函数
def sub_cb(topic, data):
    global fota_data, fota_topic, fota_flag, ota_module,chafen_url,ota_data
    # 解码字节数据为字符串
    fota_data = data.decode('utf-8')
    fota_topic = topic.decode('utf-8')
    if fota_topic in (ota_topic_device_upgrade, ota_topic_firmware_get_reply):
        try:
            # 解析 JSON 数据
            fota_data = ujson.loads(fota_data)
            # 检查 "data" 键和 "module" 键是否存在
            if "data" in fota_data and "module" in fota_data["data"]:
                ota_module = fota_data["data"]["module"]
                #固件升级
                if ota_module == FIRMWARE_NAME:
                    chafen_url = fota_data["data"]["url"]
                    print("chafen_url =", chafen_url)
                    fota_flag=True
                #app升级
                if ota_module == PROJECT_NAME:
                    ota_version = fota_data["data"]["version"]
                    print("OTA 模块:", ota_module)
                    print("OTA 版本:", ota_version)
                    ota_data = [{"url": i["fileUrl"], "file_name": "/usr/" + i["fileName"].replace(".bin", ".py")} for i in fota_data.get("data", {}).get("files", [])]
                    print("OTA 文件信息:", ota_data)
                    fota_flag=True
        except ValueError:
            print("JSON 解析失败")
    print("fota_lock release")


def fota_start():
    global fota_data, fota_topic, fota_flag, flag,ota_module,ota_data,chafen_url,ota_data
    while True:
        if fota_flag:
            if fota_topic in (ota_topic_device_upgrade, ota_topic_firmware_get_reply):
                # OTA升级
                if ota_module:
                    if ota_module == FIRMWARE_NAME:
                        # FOTA升级
                        _fota = fota()
                        _fota.httpDownload(url1=chafen_url,callback=fota_callback)
                    elif ota_module == PROJECT_NAME:
                        # # SOTA升级
                        _app_fota = app_fota.new()
                        res = _app_fota.bulk_download(ota_data)
                        print("app_fota.bulk_download res:", res)
                        # SOTA上报升级结果
                        ota_process = 100 if not res else -1
                        if ota_process == 100:
                            _app_fota.set_update_flag()
                        process_data = {
                            "id": 5,
                            "params": {
                                "step": ota_process,
                                "desc": "desc",
                                "module": ota_module,
                            }
                        }
                        cloud.publish(ota_topic_device_progress, ujson.dumps(process_data), qos=1)
                        # 升级完成之后需要重启设备
                        Power.powerRestart()
        utime.sleep(1)




# FOTA升级进度回调函数
def fota_callback(args):
    print("Download status: %s, Download process: %s" % tuple(args))
    ota_process = None
    if args[0] in (0, 1, 2) and args[1] == 100:
        ota_process = 100
    else:
        ota_process = -1
    if ota_process is not None:
        # FOTA上报升级结果
        process_data = {
            "id": 5,
            "params": {
                "step": ota_process,
                "desc": "success",
                "module": ota_module,
            }
        }
        cloud.publish(ota_topic_device_progress, ujson.dumps(process_data), qos=1)
        # 升级成功后会自动重启, 此处无需手动重启
        # Power.powerRestart()

# 设置订阅Topic回调函数
cloud.setCallback(sub_cb)

# 订阅Topic
qos = 1
cloud.subscribe(ota_topic_device_upgrade, qos)
cloud.subscribe(ota_topic_firmware_get_reply, qos)

# 阿里云功能启动
cloud.start()

# 上报软件版本信息
sota_data = {
    "id": 1,
    "params": {
        "version": PROJECT_VERSION,
        "module": PROJECT_NAME,
    }
}
cloud.publish(ota_topic_device_inform, ujson.dumps(sota_data), qos=1)

# 上报固件版本信息
fota_data = {
    "id": 2,
    "params": {
        "version": FIRMWARE_VERSION,
        "module": FIRMWARE_NAME,
    }
}
cloud.publish(ota_topic_device_inform, ujson.dumps(fota_data), qos=1)

# 查询软件升级计划
sota_query = {
    "id": 3,
    "version": "1.0.2",
    "params": {
        "module": PROJECT_NAME,
    },
    "method": "thing.ota.firmware.get"
}
cloud.publish(ota_topic_firmware_get, ujson.dumps(sota_query), qos=1)

# 查询固件升级计划
sota_query = {
    "id": 4,
    "version": "1.0",
    "params": {
        "module": FIRMWARE_NAME,
    },
    "method": "thing.ota.firmware.get"
}
cloud.publish(ota_topic_firmware_get, ujson.dumps(sota_query), qos=1)

if __name__ == "__main__":
    _thread.start_new_thread(fota_start, ())
    while True:
        utime.sleep(3)

```

### 腾讯云 - 示例代码

程序架构同阿里云示例代码一致，只需修改云平台功能部分代码为腾讯云平台。

## 各型号支持情况

| 型号                             | 差分升级 | 最小系统升级 |
| -------------------------------- | -------- | ------------ |
| EC600NCNLC&EC600NCNLF&EGx00N系列 | 支持     | 支持         |
| EC600NCNLA&EC600NCNLE&EC800NCNLA | 不支持   | 支持         |
| ECx00M&EGx00M系列                | 不支持   | 支持         |
| EC200A系列                       | 支持     | 不支持       |
| ECx00U&EGx00U&ECx00G系列         | 支持     | 不支持       |
| ECx00E系列                       | 支持     | 不支持       |
| BG95&BG600L系列                  | 支持     | 不支持       |
