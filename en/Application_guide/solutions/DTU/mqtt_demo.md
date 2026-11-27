# Device development based on MQTT protocol

This document will guide you through the process of developing devices that communicate via the MQTT protocol. It covers the following essential steps: modifying the configuration file to establish an MQTT connection, importing and running scripts based on the QuecPython module, and debugging data transmission. Please follow the contents of each section below to implement a data transparent transmission solution based on MQTT.

## Modify configuration file

After obtaining the code from the "Experimental Source Code" section in the [Development Resource Summary](dev_resources.md), you can modify the project configuration file **dtu_config.json** under the **code** directory as needed.

Based on TCP private server data transparent transmission, configure as follows:

- set `systemconfig.cloud` configuration item as `mqtt`, which means MQTT transparent mode. 

    ```python
    {
        "system_config": {
            "cloud": "mqtt"
        }
    }
    ```

- This experiment adopts the MQTT transparent transmission mode, and users need to set the MQTT server domain name, port, client ID, and subscription and publishing topics parameters in the `mqtt_pivate_cloud_config` configuration item according to the actual situation, as follows,This experiment was carried out using the [ConnectLab](https://connectlab.phicotek.com/connectlab/login?redirect=%2F&_gl=1%2a1tpg9cd%2a_gcl_au%2aMTc2MjIyMDM2Mi4xNzcyMTc0Mjg4LjE5MjAzODU1MDUuMTc3MjE4Mjc1NS4xNzcyMTgyNzU0) Internet of Things application protocol testing platform.

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
            "subscribe": {"down": "/public/TEST/down"},
            "publish": {"up":  "/public/TEST/up"}
        }
    }
    ```

- The configuration item `uart_config` is a serial port configuration parameter, which defaults to the configuration made based on the current experimental development board and cannot be changed. If users use other development boards, they need to configure according to the actual hardware.

    ```python
    {
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
        }
    }
    ```

- `network_config` is used to configure the APN. If the APN is not configured or an incorrect APN is configured, the base station of the operator will automatically correct the situation during the network connection process and send a correct APN to the terminal. Overseas users need to configure this item. At this time, the terminal will choose to accept and use this APN to continue the network connection. 
    ```python
    {
        "apn": "",  # Access Point Name, for example "cmnet" or "ctnet"
        "username": "",  # APN username, usually not required
        "password": "",  # APN password
        "auth_type": 0  # Authentication type: 0 - no authentication, 1 - PAP, 2 - CHAP 
    }
    ```

The complete configuration file template is as follows:

```json
{
    "system_config": {
        "cloud": "tcp"
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

Parameter description:

- ` systemconfig.config `: Specify the type of private cloud currently in use. Currently supports TCP and MQTT.

- `mqtt_private_cloud_config`:  MQTT private cloud configuration.
- `socket_private_cloud_config`:  TCP private cloud configuration.
- `uart_config`: Serial port parameter configuration.
- `network_config`: APN configuration. It can be configured if necessary.

## Create a New MQTT Server
1. Log in to the [ConnectLab](https://connectlab.phicotek.com) online testing platform, choose the guest login option, and select "Create MQTT Server".
2. Fill in the newly created MQTT service address, port, username, and password in the "dtu_config.json" file as shown in the picture, and also remember to modify the parameters in "system_config" to "mqtt". 

![](../../media/solutions/DTU/connectlab.png)


## Script import and run

After downloading and installing the QPYCom tool, use the tool to download the script to the QuecPython module.

> 💡 **Tips**
>
> QPYCom Installation and usage tutorial: https://developer.quectel.com/doc/quecpython/Application_guide/en/dev-tools/QPYcom/index.html

1. After installing QPYcom, we select the serial port and open the Quectel USB REPL Port.
2. Go to the download interface, follow the first step in the figure to create your own project.
3. Right-click on "usr" and select "One-click Import", then choose the "code" directory under solution-HuaDTU that was cloned via git.
4. Click on the fourth step in the figure, switch back from downloading the firmware to downloading the script, and click on the script.
5. The sixth step in the figure opens the serial port.

![](../../media/solutions/DTU/import_file.png)

6. After the download is complete, switch to the file interface and select _main.py for running.

![](../../media/solutions/DTU/run_scripts.png)

## Debugging

**View REPL interaction port logs**

After the program runs, the log output can be seen on the REPL interaction page as shown in the following figure. There are two threads in the DTU service that process data: one is used to detect and read serial port data and forward it to the cloud, and the other is used to detect the transmission of downstream data from the cloud to the serial port, as shown in the following figure.

![](../../media/solutions/DTU/repl_logs.png)

**Upstream data transparent transmission**

Simulate MCU sending upstream data to the module using serial debugging tools.

- Enter "hello world!" in the `Input String` input box character string.
- Click the `Send Command` button to send data through the serial port.

![](../../media/solutions/DTU/qcom_test.png)

DTU receives serial data and directly transmits it to MQTT cloud.

![](../../media/solutions/DTU/debug_log.png)

Cloud based reception of upstream data logs

![](../../media/solutions/DTU/connectlab2.png)


**Downstream data transparent transmission**

Send downstream data from the cloud.

- Set the theme for cloud downlink data (consistent with the subscription theme configured for DTU applications).
- Enter downstream data.
- Publish.

![](../../media/solutions/DTU/connectlab3.png)

DTU downlink data log.

![](../../media/solutions/DTU/debug_log_down_data.png)

Observe the serial port debugging tool through QCOM to simulate the downstream data of the MCU receiving module.

![](../../media/solutions/DTU/qcom_down.png)

