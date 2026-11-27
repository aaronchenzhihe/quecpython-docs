# Device development based on the MQTT protocol

## Modify the configuration file

Path of the engineering configuration file：`code/dtu_config.json`。 

Based on the data transmission of TCP private server, the following configuration is made:

- By default, the configuration item `system_config.cloud` is defined as `"tcp"` which means TCP transparent mode. The system will automatically read the configuration item `socket_private_cloud_config`.

  ```python
  {
      "system_config": {
          "cloud": "mqtt"  # The mqtt transparent transmission mode is configured by default
      }
  }
  ```

- This experiment adopts MQTT transparent transmission mode，Set the MQTT server domain name (*server*), port (*port*), client id (*client_id*), and subscription and publication topics in the mqtt_private_cloud_config configuration item as follows:

  ```python
  {
      "mqtt_private_cloud_config": {
          "server": "mq.tongxinmao.com",
          "port": 18830,
          "client_id": "txm_1682300809",
          "user": "",
          "password": "",
          "clean_session": true,
          "qos": 0,
          "keepalive": 60,
          "subscribe": {"down": "/public/TEST/down"},  # Downlink data transparent topic
          "publish": {"up":  "/public/TEST/up"}  # Uplink data transparent transmission topic
      }
  }
  ```

- The `uart_config`configuration item is the serial port configuration parameter. By default, it is configured according to the current experimental development board and cannot be changed. If other development boards are used, configure them based on the actual hardware.

```python
{
    "uart_config": {
        "port": 2,  # Serial port number. Set this parameter based on the actual hardware configuration
        "baudrate": 115200,  # Baud rate
        "bytesize": 8,  # data bits
        "parity": 0,  # Parity check
        "stopbits": 1,  # stop bit
        "flowctl": 0,  # fluid control

        }
    }
}
```

The complete profile template is as follows:

```json
{
    "system_config": {
        "cloud": "tcp"
    },
    "mqtt_private_cloud_config": {
        "server": "mq.tongxinmao.com",
        "port": 18830,
        "client_id": "txm_1682300809",
        "user": "",
        "password": "",
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
    }
}
```

参数说明：

- `system_config.config`: Specifies the type of private cloud currently in use. tcp and mqtt are currently supported.
- `mqtt_private_cloud_config`:MQTT private cloud configuration.
- `socket_private_cloud_config`:tcp private cloud configuration.
- `uart_config`：Set serial port parameters.

## script is imported and run

Download and install the **QPYCom** tool and use it to download scripts to the QuecPython module.

> 💡 **Tips**
>
> [QPYCom installation and use tutorial](../../dev-tools/QPYcom/README.md)

## Service debugging

**View REPL interface logs**

After the program is run, you can see the log output in the REPL interaction page as shown in the following figure. There are two threads in the DTU service to process data, one is used to detect and read serial port data and forward the data to the cloud, and the other is to detect the cloud downlink data to the serial port, as shown in the following figure.

![](../../media/solutions/DTU-EG912U/repl_logs.png)

**Uplink transparent transmission**

The serial debugging tool is used to simulate the mcu to send uplink data to the module.

- In the `Input String` input field, type `hello world! `Character string.
- Click `Send Command` button to send data through the serial port.

![](../../media/solutions/DTU-EG912U/qcom_test.png)

DTU receives serial data and transparently transmits it directly to the mqtt cloud.

![](../../media/solutions/DTU-EG912U/debug_log.png)

 The cloud receives uplink data logs

ps：The communication cat cloud platform is only used for testing, and the cloud platform can be replaced according to actual requirements

![1](../../media/solutions/DTU-EG912U/cloud_up.png)

**Downlink transparent data transmission**

The cloud sends downlink data.

- Set the cloud downlink data theme (the same as the subscription theme configured for the DTU application).
- Enter downlink data.
- publish

![](../../media/solutions/DTU-EG912U/cloud_down.png)

Using QCOM to observe serial debugging tool to simulate mcu receiving module downlink data.

![](../../media/solutions/DTU-EG912U/qcom_down.png)

