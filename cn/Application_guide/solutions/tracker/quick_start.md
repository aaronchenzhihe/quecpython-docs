# 快速上手

## 硬件准备

- Windows 电脑一台，建议 `Win10` 系统
- 一套 [EC200UEUAA QuecPython 标准开发板](https://developer.quectel.com/doc/quecpython/Dev_board_guide/zh/ec200x-evb.html)（含 LTE 天线、 Type-C 数据线等）
- 一根 [GNSS天线](https://e.tb.cn/h.TpAFyEz02BnCHRD?tk=fznae6ITVEX)
- 一个排针跳线帽
- 一张可正常使用的 Nano SIM 卡

## 环境搭建

- 下载并安装 EC200U 系列模组驱动： [QuecPython_USB_Driver_Win10_U_G](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverU_V1.0.19.zip)
- 下载并安装 [VSCode](https://code.visualstudio.com/)
- 下载并解压 [QPYcom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) 工具到电脑的合适位置
- 使用 QPYcom 烧录[固件包](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_V0005_EC200U_EUAA_FW.zip)
- 下载[实验源码](https://codeload.github.com/QuecPython/solution-tracker/zip/refs/heads/master)

## 硬件连接

按照下图进行硬件连接：

![](../../media/solutions/tracker/连线.png)

1. 将 LTE 天线连接至标识有 `LTE` 字样的天线连接座上
2. 将 GNSS 天线连接至标识有 `GNSS` 字样的天线连接座上
3. 在图示位置开发板背面插入可用的 Nano SIM 卡
4. 使用跳线帽将标识有 `GNSS_EN` 字样的两根排针短接，使能开发板内置 GNSS 功能
5. 使用 Type-C 数据线连接开发板和电脑

## 云服务平台

按照以下步骤创建、配置并连接阿里云物联网平台：

### 创建产品与设备

详情可参考阿里云说明文档：[创建产品与设备](https://help.aliyun.com/document_detail/73705.html)。

- 创建产品

![aliyun_create_product](../../media/solutions/tracker/aliyun_create_product.png)

- 创建设备

![aliyun_create_device](../../media/solutions/tracker/aliyun_create_device.png)

### 定义物模型

详情可参考阿里云说明文档：[为产品定义物模型](https://help.aliyun.com/document_detail/117636.html)。

物模型可以更直观的看到上传的数据值，本解决方案提供了物模型 demo，可直接导入工程项目中 `object_model_demo/ali_cloud_object_model.json` 文件自动生成，无需手动创建。

![aliyun_import_object_model](../../media/solutions/tracker/aliyun_import_object_model.png)

## 设备开发

本节重点阐述用户在业务上需要关注的配置或代码如何修改，以便用户能够快速上手。

在开始前，先用 `VSCode` 打开项目工程源码，找到 code 中的以 `settings_` 开头的配置文件进行配置。

> 本方案支持两个可选平台：阿里 IoT 和 Thingsboard。这里主要介绍使用阿里 IoT 的配置与连接。

### IoT 平台连接信息

`settings_server.py` 用于配置 IoT 平台连接信息

![](../../media/solutions/tracker/aliyun_mqtt_param.png)

```python
# 以下两个平台使用哪个就配置哪个，本解决方案以阿里 IoT 为例
class AliIotConfig:
    product_key = ""
    product_secret = None
    device_name = ""
    device_secret = ""
    server = "iot-as-mqtt.cn-shanghai.aliyuncs.com"		# 默认参数
    qos = 1

class ThingsBoardConfig:
    host = ""
    port = 1883
    username = ""
    qos = 0
    client_id = ""
```

### 定位模块配置信息

`settings_loc.py` 用于配置定位模块的配置信息(内置 GNSS/外挂 GNSS 串口，基站/Wifi 定位的认证信息)

>  本解决方案基于 EC200UEUAA QuecPython 标准开发板的内置 GNSS 功能实现。

```python
# GNSS 定位
gps_cfg = {
    "gps_mode": _gps_mode.internal,		# 内置 GNSS
    "UARTn": UART.UART2,
    "buadrate": 115200,
    "databits": 8,
    "parity": 0,
    "stopbits": 1,
    "flowctl": 0,
    "PowerPin": None,
    "StandbyPin": None,
    "BackupPin": None,
}

# 基站定位
cell_cfg = {
    "serverAddr": "www.xxxxx.com",
    "port": 80,
    "token": "xxxxxxxxxx",
    "timeout": 3,
    "profileIdx": profile_idx,
}

# Wifi 定位
wifi_cfg = {
    "token": "xxxxxxxxxx"
}
```

### 用户业务配置信息

`settings_user.py` 用于配置用户业务功能的相关配置参数(如报警开关、低电阈值等)

```python
class _server:
    none = 0x0
    AliIot = 0x1
    ThingsBoard = 0x2

class _drive_behavior_code:
    none = 0x0
    sharply_start = 0x1
    sharply_stop = 0x2
    sharply_turn_left = 0x3
    sharply_turn_right = 0x4

class _ota_upgrade_module:
    none = 0x0
    sys = 0x1
    app = 0x2

debug = 1
log_level = "DEBUG"
checknet_timeout = 60
server = _server.AliIot
phone_num = ""
low_power_alert_threshold = 20
low_power_shutdown_threshold = 5
over_speed_threshold = 50
sw_ota = 1
sw_ota_auto_upgrade = 1
sw_voice_listen = 0
sw_voice_record = 0
sw_fault_alert = 1
sw_low_power_alert = 1
sw_over_speed_alert = 1
sw_sim_abnormal_alert = 1
sw_disassemble_alert = 1
sw_drive_behavior_alert = 1
drive_behavior_code = _drive_behavior_code.none
loc_method = _loc_method.all
loc_gps_read_timeout = 300
work_mode = _work_mode.cycle
work_mode_timeline = 3600
work_cycle_period = 30
user_ota_action = -1
ota_status = {
    "sys_current_version": "",
    "sys_target_version": "--",
    "app_current_version": "",
    "app_target_version": "--",
    "upgrade_module": _ota_upgrade_module.none,
    "upgrade_status": _ota_upgrade_status.none,
}
```

### 开机

完成硬件连接的工作后，开发板会自动开机，查看电脑设备管理器的端口列表中是否出现包含 `Quectel USB` 字样的 COM 口。

![](../../media/solutions/tracker/comport.png)

### 烧录固件包

参考[此章节](../../../Getting_started/flash_firmware.md)，烧录固件包 [QPY_OCPU_BETA0001_EC200U_EUAA_FW](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_V0005_EC200U_EUAA_FW.zip) 至开发板。

### 脚本导入与运行

1. 参考[此章节](../../../Getting_started/first_python.md#传输脚本文件到模组)，将源码目录下 `code` 文件夹中的所有文件导入到模组文件系统，如下图所示：

   ![](../../media/solutions/tracker/code.png)

2. 参考[此章节](../../../Getting_started/first_python.md#执行脚本文件)，执行主程序文件 `_main.py`。
3. 参考[此章节](../../../Getting_started/first_python.md#停止程序运行)，停止程序运行。

## 业务调试

### 程序启动

执行 `_main.py` 脚本后，程序开始运行，在交互页面即可查看项目运行状态：

> 程序要运行一段时间后才能获取到 GNSS 定位信息，具体可参考相关[文档说明](../../../FAQ/hardware/gnss.md#什么是GNSS的冷启动、温启动和热启动)。

![tracker_running](../../media/solutions/tracker/tracker_running.png)

在阿里云物联网平台可查看设备状态信息。

![aliyun_tracker_running_saas_view](../../media/solutions/tracker/aliyun_tracker_running_saas_view.png)

### 云端控制

通过在线调试下发指令到设备端进行设备控制与数据交互。

![aliyun_online_send_message](../../media/solutions/tracker/aliyun_online_send_message.png)

### OTA升级

> **固件升级只支持差分升级，不支持整包升级**；
>
> **项目文件升级包，以修改项目代码文件后缀名为 `.bin` 的方式做成升级包，上传云端，可上传多个文件**。

#### 固件升级

1. 制作固件升级差分包(联系固件开发人员)；
2. 创建 OTA 模块，以设备平台名称命名，如： `EC200U-EUAA`。

![aliyun_ota_fota_module](../../media/solutions/tracker/aliyun_ota_fota_module.png)

3. 创建OTA升级包

   升级包名称以 `settings.py` 中 `PROJECT_NAME` 命名，如：`QuecPython-Tracker`。

![aliyun_ota_fota_upgrade_package](../../media/solutions/tracker/aliyun_ota_fota_upgrade_package.png)

4. 选择批量升级，创建升级计划

![aliyun_ota_fota_plain](../../media/solutions/tracker/aliyun_ota_fota_plain.png)

![](../../media/solutions/tracker/aliyun_ota_fota_plain_2.png)

5. 等待设备升级，查看升级结果
    + 当设备开启 OTA 升级和 OTA 自动升级，则等待设备升级完成，查看升级结果；

    + 当设备开启 OTA 升级，但未开启自动升级时，可通过在线调试模块下发 `user_ota_action=1` 的物模型设置指令，进行 OTA 升级。

      > 设备下载固件成功后会重启进入升级，需要等待一段时间。当观察到 QPYcom 重新出现 `Quectel USB` 字样的端口时，说明 OTA 升级完成。此时打开交互端口重新运行 `_main.py` 脚本，上报升级成功后的版本号，云端刷新即可看到 OTA 状态为 `升级成功`。

![](../../media/solutions/tracker/after_ota.png)

![aliyun_ota_fota_upgrade_process](../../media/solutions/tracker/aliyun_ota_fota_upgrade_process.png)

#### 项目升级

1. 创建 OTA 模块，以 `settings.py` 中 `PROJECT_NAME` 命名，如：`QuecPython-Tracker`。

![aliyun_ota_sota_module](../../media/solutions/tracker/aliyun_ota_sota_module.png)

2. 将需要升级的项目文件后缀名修改为 `.bin`
3. 创建 OTA 升级包
    + 此处需要在**推送给设备的自定义信息**中编写升级文件名对应的设备全路径文件名，如：`{"files":{"common.bin":"/usr/modules/common.py","settings.bin":"/usr/settings.py","test_tracker.bin":"/usr/test_tracker.py"}}`

![aliyun_ota_sota_upgrade_package](../../media/solutions/tracker/aliyun_ota_sota_upgrade_package.png)

4. 选择批量升级，创建升级计划

![aliyun_ota_sota_plain](../../media/solutions/tracker/aliyun_ota_sota_plain.png)

5. 等待设备升级，查看升级结果

    + 当设备开启 OTA 升级和 OTA 自动升级，则等待设备升级完成，查看升级结果；
    + 当设备开启 OTA 升级，但未开启自动升级时，可通过在线调试模块下发 `user_ota_action=1` 的物模型设置指令，进行 OTA 升级。

![aliyun_ota_sota_upgrade_process](../../media/solutions/tracker/aliyun_ota_sota_upgrade_process.png)