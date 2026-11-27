# 基于 MQTT 协议的设备开发

本文档将指导您完成开发通过 MQTT 协议进行通信的设备的过程。它涵盖了以下必要步骤：修改配置文件以建立 MQTT 连接、导入并运行基于 QuecPython 模块的脚本，以及调试数据传输。请按照以下各部分的内容来实现基于 MQTT 的数据透明传输解决方案。

## 修改配置文件

在[开发资源汇总](dev_resources.md)中的实验源码处获取代码后，可根据需求修改**code**目录下的工程配置文件**dtu_config.json**。

基于 TCP 私有服务器数据透传做如下配置：

- 设置 `system_config.cloud` 配置项定义为 `"mqtt"` 即 MQTT 透传模式。

    ```python
    {
        "system_config": {
            "cloud": "mqtt"  # 默认配置 mqtt 透传模式
        }
    }
    ```

- 本实验采用 MQTT 透传模式，用户需根据实际情况设置 `mqtt_private_cloud_config` 配置项中的 MQTT 服务器域名（*server*）、端口（*port*）、客户端id（*client_id*）以及订阅和发布主题等参数，如下。当前使用了移远推出的[ConnectLab](https://connectlab.phicotek.com) 物联网应用协议测试平台来进行的。

    ```python
    {
        "mqtt_private_cloud_config": {
            "server": "101.37.104.185",
            "port": 40304,
            "client_id": "quec_1682300809",
            "user": "quectel",
            "password": "12345678",
            "clean_session": true,
            "qos": 0,
            "keepalive": 60,
            "subscribe": {"down": "/public/TEST/down"},  # 下行数据透传主题
            "publish": {"up":  "/public/TEST/up"}  # 上行数据透传主题
        }
    }
    ```

- `uart_config` 配置项是串口配置参数，默认是根据当前实验开发板做的配置，不可更改。如用户采用其他开发板则需要根据实际硬件进行配置。

    ```python
    {
        "uart_config": {
            "port": 2,  # 串口号，根据实际硬件配置选择，当前实验不可更改
            "baudrate": 115200,  # 波特率
            "bytesize": 8,  # 数据位
            "parity": 0,  # 奇偶校验
            "stopbits": 1,  # 停止位
            "flowctl": 0,  # 流控
            "rs485_config": {  # RS485 配置
                "gpio_num": 28,  # 485 控制脚，当前实验不可更改
                "direction": 0  # 引脚电平变化控制，1表示引脚电平变化为：串口发送数据之前由低拉高、发送数据之后再由高拉低，0表示引脚电平变化为：串口发送数据之前由高拉低、发送数据之后再由低拉高
            }
        }
    }
    ```

- `network_config` 用于配置APN，如果没有配置APN或者配置了错误的APN，运营商的基站会在注网过程中自动纠正并下发一个正确的APN给终端，海外用户需配置此项，此时终端会选择接受并使用该APN继续入网。

    ```python
    {                     
        "apn": "",  # 接入点名称，例如 "cmnet" 或 "ctnet"
        "username": "",  # APN用户名，一般不需要
        "password": "",  # APN密码
        "auth_type": 0  # 认证类型：0-无认证，1-PAP，2-CHAP                  
    }
    ```

完整配置文件模版如下：

```json
{
    "system_config": {
        "cloud": "mqtt"
    },
    "mqtt_private_cloud_config": {
        "server": "101.37.104.185",
        "port": 40304,
        "client_id": "quec_1682300809",
        "user": "quectel",
        "password": "12345678",
        "clean_session": true,
        "qos": 0,
        "keepalive": 60,
        "subscribe": {"down": "/public/TEST/down"},
        "publish": {"up":  "/public/TEST/up"}
    },
    "socket_private_cloud_config": {
        "domain": "112.31.84.164",
        "port": 8305,
        "timeout": 5,
        "keep_alive": 5
    },
    "uart_config": {
        "port": 2,
        "baudrate": 115200,
        "bytesize": 8,
        "parity": 0,
        "stopbits": 1,
        "flowctl": 0,
        "rs485_config": {
        	"gpio_num": 28,
            "direction": 0
        }
    },
    "network_config": {                     
        "apn": "",            
        "username": "",                      
        "password": "",                      
        "auth_type": 0                        
    }
}
```

参数说明：

- `system_config.config`: 指定当前使用的私有云类型。目前支持tcp和mqtt。
- `mqtt_private_cloud_config`: MQTT私有云配置。
- `socket_private_cloud_config`: tcp私有云配置。
- `uart_config`：串口参数配置。
- `network_config`:APN配置，如有需要可进行配置。

## 新建MQTT服务器
1. 进入[ConnectLab](https://connectlab.phicotek.com)在线测试平台，选择游客登录，选择MQTT服务器新建。
2. 将图中新建的MQTT服务地址、端口、用户名、密码填到上述dtu_config.json的mqtt_private_cloud_config中，同时记得修改system_config的参数为"mqtt"。
![](../../media/solutions/DTU/connectlab.png)


## 脚本导入并运行

下载安装 **QPYCom** 工具后使用该工具下载脚本至 QuecPython 模组。

> 💡 **Tips**
>
> QPYCom 安装和使用教程：https://developer.quectel.com/doc/quecpython/Application_guide/zh/dev-tools/QPYcom/index.html

1. 安装好 QPYcom 后我们展开选择串口，打开Quectel USB REPL Port
2. 来到下载界面，按照图中第1步创建一个自己的项目
3. 右击usr选择**一键导入**，选择git克隆下来的solution-HuaDTU下的code目录
4. 点击图中第4步，由下载固件切换回下载脚本，点击下载脚本
5. 图中第6步打开串口

![](../../media/solutions/DTU/import_file.png)

6. 下载完成后切换文件界面，选择_main.py运行

![](../../media/solutions/DTU/run_scripts.png)

## 业务调试

**查看 REPL 交互口日志**

程序运行后，在 REPL 交互页面可以看到日志输出如下图所示。 DTU服务中有2个线程处理数据，一个是用于检测读取串口数据并转发数据给云端，一个是检测云端下行数据透传给串口，如下图所示。 

![](../../media/solutions/DTU/repl_logs.png)

 **上行数据透传**

使用串口调试工具模拟mcu给模组发送上行数据。

- 在`Input String`输入框中输入`hello world!`字符串。
- 点击`Send Command`按钮通过串口发送数据。

![](../../media/solutions/DTU/qcom_test.png)

DTU接收串口数据，直接透传至mqtt云端。

![](../../media/solutions/DTU/debug_log.png)

 云端接收上行数据日志

![](../../media/solutions/DTU/connectlab2.png)

**下行数据透传**

云端发送下行数据。

- 设置云端下行数据主题（与DTU应用配置的订阅主题一致）。
- 输入下行数据。
- 发布。

![](../../media/solutions/DTU/connectlab3.png)

 DTU下行数据日志。 

![](../../media/solutions/DTU/debug_log_down_data.png)

通过QCOM观察串口调试工具模拟mcu接收模组下行数据。

![](../../media/solutions/DTU/qcom_down.png)

