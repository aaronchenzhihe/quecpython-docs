# MQTT Application Note

This document provides a detailed introduction on how to use the MQTT protocol for communication in QuecPython. It includes an overview of the protocol, environment setup, code examples, and result demonstrations.

## Overview

MQTT is a lightweight messaging protocol designed to facilitate reliable communication between devices in Internet of Things (IoT) applications. It operates on a publish-subscribe pattern, involving communication between an MQTT server (broker or server) and multiple MQTT clients. The MQTT protocol encompasses the following characteristics:

- Lightweight: The design of the MQTT protocol is simple, with minimal protocol header overhead, making it suitable for resource-constrained devices and networks.
- Low Bandwidth Consumption: MQTT employs binary encoding, effectively utilizing network bandwidth.
- Asynchronous Communication: The client can publish and subscribe to messages at any time without waiting for responses from the MQTT server.
- Publish-Subscribe Pattern: The message publisher releases messages to specific topics, while the subscriber subscribes to topics of interest. This pattern supports loose coupling communication and flexible message delivery.

## MQTT Application Scenarios

MQTT plays a significant role in various fields, including the Internet of Things (IoT), sensor networks, remote monitoring and control, real-time data transmission, message push and notifications, connected vehicles, and energy management. Its lightweight nature, low bandwidth consumption, and reliable message delivery mechanism make MQTT a preferred communication protocol in many applications. Here are some commonly used scenarios:

![image-20230711185611691](../../media/network-comm/net-protocols/mqtt/image-20230711185611691.png)

1. Internet of Things (IoT): MQTT is one of the most commonly used communication protocols in IoT applications. It is suitable for scenarios involving a large number of connected devices, offering reliable message delivery and real-time communication. MQTT's lightweight nature allows it to operate in resource-constrained devices and network environments, while supporting the publish-subscribe pattern and asynchronous communication, enabling devices to exchange information, monitor, and control in real time.
2. Sensor Networks: MQTT can be employed for data collection and real-time monitoring in sensor networks. Sensors can publish data to specific topics, and subscribers can subscribe to topics of interest to receive sensor data. MQTT's low bandwidth consumption and efficient message delivery mechanism make it a reliable choice for data transmission in sensor networks.
3. Remote Monitoring and Control: MQTT makes remote monitoring and control straightforward and reliable.  Real-time monitoring of device status, sensor data, and remote control can be performed through MQTT. This is highly beneficial in applications such as smart homes, smart cities, and industrial automation etc..
4. Real-time Data Transmission: MQTT's asynchronous communication mechanism makes it well-suited for real-time data transmission scenarios. It can transmit data with minimal latency, enabling real-time monitoring and communication. For instance, in financial trading systems, MQTT can be used to transmit real-time market data to trading systems and investors.
5. Message Push and Notifications: MQTT's publish-subscribe pattern makes it an ideal choice for message push and notifications. The server can publish messages to specific topics, and subscribers will receive these messages in real time. This is valuable for push notifications in applications, chat applications, and real-time data updates.
6. Connected Vehicles (Connected Car): MQTT can be employed for communication between vehicles and between vehicles and cloud platforms. It can support functionalities such as vehicle status monitoring, vehicle diagnostics, and remote control, enhancing vehicle safety and efficiency.
7. Energy Management: MQTT can be used for data transmission in energy monitoring and management systems. Through MQTT, real-time energy consumption data can be transmitted to monitoring and analysis systems for energy consumption optimization and monitoring purposes.

## MQTT Communication Mechanism

In MQTT, communication is accomplished through TCP connections between clients and the server. The client initiates a connection request by sending a CONNECT message and then sends various operational messages, such as SUBSCRIBE, PUBLISH, UNSUBSCRIBE, once the connection is established.

Publishers, when sending messages to a specific topic, transmit the message content along with the specific topic to the MQTT server. The server then delivers the message to all subscribers who have subscribed to that topic.

Subscribers use the SUBSCRIBE message to subscribe to topics of interest, specifying the topic and desired QoS level. Upon receiving a subscription request, the server records the subscriber's subscription information and forwards relevant messages to the subscriber when the messages are published.

MQTT also supports retained messages, where a publisher can send a retained message and set a retained flag. The retained message is stored by the server and sent to subscribers who subscribe to the relevant topic. This allows new subscribers to receive the latest retained message.

Additionally, MQTT provides the mechanism of persistent sessions. Persistent sessions allow clients to maintain their subscription and publication state information when reconnecting. This ensures that important messages are not lost.

Through these mechanisms, MQTT achieves reliable message delivery, decoupling, and asynchronous real-time communication. It is suitable for scenarios such as IoT, sensor networks, and real-time data transmission. MQTT offers a flexible communication model and mechanisms that enable efficient message interactions between devices and applications.

![image-20230701163911290](../../media/network-comm/net-protocols/mqtt/mqtt_1.png)

- MQTT Client: An MQTT client refers to a device or application that connects to an MQTT server. Each client is identified by a unique client identifier, which is used by the server to distinguish and differentiate between different clients. In QuecPython, we use `umqtt` to implement MQTT clients. A connection object is created by passing initialization parameters, and for more details, [click here](../../../api_reference/networklib/umqtt.md#umqtt.MQTTClient).

  ```python
  from umqtt import MQTTClient
  MQTTClient(client_id, server, port=0, user=None, password=None, keepalive=0, ssl=False, ssl_params={},reconn=True,version=4)
  ```

  **Parameter:**

  - `client_id` - String type. A unique client ID.
  - `server` - String type. Server address.  It can be an IP address or a domain name.
  - `port` -Integer type.  Server port (optional). Default port: 1883. Please note that  the default port of MQTT over SSL/TLS is 8883。
  - `user` - String type. The user name registered on the server (optional).
  - `password` - String type. Password registered on the server (optional).
  - `keepalive` - Integer type. Keep alive  timeout of the client (optional). Default value: 0.
  - `ssl` - Boolean value type. Enable or disable SSL/TLS support (optional).
  - `ssl_params` - String type. SSL/TLS  parameter (optional).
  - `reconn` -  Boolean value type. Control whether to use the internal re-connected flag. True means to enable (default).
  - `version` - Integer type. The MQTT version selected (optional). 3 represents MQTTv3.1; 4 represents MQTTv3.1.1 (default)。

- MQTT Server: The MQTT server, also known as a broker, is responsible for receiving and forwarding messages, managing subscription relationships, and maintaining connection with the clients. It listens on a specified port, awaiting client connection requests, and processes them accordingly. The server actively distributes topic messages to clients that have subscribed to those topics and the clients will receive the meddages.

### Publish-Subscribe

- Publisher: A publisher is the sender of messages in MQTT. Publishers publish messages to specific topics, and delivers these messages to all subscribers of that topic through the MQTT server. After creating a client object with `umqtt`, you can use the [publish](../../../api_reference/networklib/umqtt.md#MQTTClient.publish) method to publish messages.

  ```python
  MQTTClient.publish(topic, msg, retain=False, qos=0)
  ```

  **Parameter:**

  - `topic` - String type. MQTT message topic.
  - `msg` - String type. Data to be sent.
  - `retain` - Boolean value type.  When publishing a message, set retain to true, that is, the retaining message. Default value: False.
    The MQTT server retains the most recent message with the RETAIN flag set to True on the server. Whenever an MQTT client connects to the MQTT server and subscribes to a certain topic, if there is a retained message under that topic, the MQTT server immediately sends that retained message to the client.
    Important Note: The MQTT server will only retain the most recent message with the RETAIN flag set to True for each topic! This means that if the MQTT server has already retained a message for a specific topic and a client publishes a new message, the previously retained message on the server will be replaced!
  - `qos` - Integer type. Quality of service of the MQTT message. 0 - The sender sends the message only once with no retry.  1-The sender sends the messageat least once to make sure the message is sent to the broker. Default value: 0.

- Subscriber: A subscriber is the receiver of messages in MQTT. Subscribers can subscribe to topics of interest to receive messages related to those topics. Once a subscriber subscribes to a specific topic, it will receive all published messages under that topic. In projects, topics are often defined based on different events. When a device subscribes to an event topic, it can receive push messages related to that topic.

  [Click here to see API references](../../../api_reference/networklib/umqtt.md#MQTTClient.subscribe)

  ```python
  MQTTClient.subscribe(topic,qos)
  ```

  **Parameter:**

  - `topic` - String type. MQTT topic.
  - `qos` - Integer type. Quality of service of the MQTT message. 0 - The sender sends the message only once with no retry.  1-The sender sends the messageat least once to make sure the message is sent to the broker. Default value: 0.

- Topic: A topic is an identifier used for message publishing and subscription in MQTT. It can be hierarchical, using slashes (/) to separate different levels, such as "Quectel/Python/temperature". Topics are used to organize the delivery of messages. Publishers publish messages to specific topics, and subscribers subscribe to topics of interest.

- Message Delivery: The MQTT server is responsible for receiving messages sent by publishers and delivering them to subscribers who have subscribed to the corresponding topics. When a publisher publishes a message to a certain topic, all subscribers who have subscribed to that topic will receive the message.

  ![image-20230712145141895](../../media/network-comm/net-protocols/mqtt/image-20230712145141895.png)

- Decoupling and Flexibility: The publish-subscribe pattern decouples publishers and subscribers, eliminating the need for direct communication between them. Publishers only need to publish messages to specific topics without the need of knowing who will receive the messages. Similarly, subscribers only need to subscribe to topics of interest without the need of knowing the source of the messages. This decoupling and flexibility make the system more scalable and adaptable.

- Asynchronous Communication: In the publish-subscribe model, communication between publishers and subscribers is asynchronous. Publishers can publish messages at any time without waiting for a response from subscribers. Subscribers can receive messages published by publishers and process them as needed.

### QoS Level

MQTT defines three different levels of Quality of Service (QoS) for controlling the reliability and assurance of message delivery:

- QoS 0 (At most once): Message publishers send a message only once without any acknowledgment mechanism. Messages at this level can be lost or delivered multiple times. This level is suitable for scenarios where reliability is not critical.
- QoS 1 (At least once): Message publishers ensure that the message is delivered at least once, which might lead to duplicate deliveries. This level uses a publish-and-acknowledge mechanism to achieve reliable delivery and is suitable for scenarios that require at least once delivery.
- QoS 2 (Exactly once): Message publishers ensure that the message is delivered exactly once, achieved through a two-step handshake and four-step handshake confirmation mechanism. This level provides the highest level of delivery reliability and is suitable for scenarios where precise delivery is of utmost importance.

[Click here to see](../../../api_reference/networklib/umqtt.md#MQTTClient.subscribe) the method of setting the QoS level in QuecPython.

```python
"""
When a client subscribes to a topic with QoS 0 and then publishes a messagewith QoS 1 or 2 to the same topic, the client will not receive the message it published. This is because the QoS level is defined with respect to message delivery reliability. In MQTT, the QoS level of a message is specified at the time of publishing, and the QoS level specified during subscription only applies to the received messages.

For QoS 0, the publisher sends the message without receiving any acknowledgment or response, and no message retransmission occurs. This means that even if the publisher subscribes to the same topic, there's no guarantee that the publisher will receive its own published message.

If the publisher wants to ensure it receives its own published message, it should set the QoS level of the published message to 1 or 2, and use the corresponding QoS level during subscription. In this way, the client will receive an acknowledgment upon publishing the message and, in the case of QoS levels 1 or 2, message retransmission will occur to ensure reliable delivery.
"""
MQTTClient.subscribe("Quectel/Python/demo",1)
MQTTClient.publish("Quectel/Python/demo","Hello", qos=1)
```

### Last Will and Testament (LWT)

MQTT allows clients to set a Last Will and Testament (LWT) message when establishing a connection. During the connection setup process, a client can define the parameters for the LWT message, including the topic, message content, and QoS level. If the MQTT server detects that a client has not sent a keep-alive packet within the specified interval and the client has not requested to close the connection, the server considers the client to have disconnected abnormally. In such cases, the server publishes the LWT message to the specified topic based on the client's LWT settings. Subscribers of this topic can then receive the LWT message, indicating the offline status of the client.

The `set_last_will` API in QuecPython allows you to set the WT message.  [Click here](../../../api_reference/networklib/umqtt.md#MQTTClient.set_last_will) for more details.

```python
MQTTClient.set_last_will(topic,msg,retain=False,qos=0)
```

**Parameter:**

- `topic` - String type. MQTT LWT topic. 
- `msg` - String type.  LWT message content.
- `retain` -  Boolean value type.  When `retain`  is set to True, the broker will retain the message. Default value: False.
- `qos` - Integer type. QoS of the message. Range: 0, 1.

### Persistent Session

MQTT supports a mechanism called Persistent Session, which allows maintaining session state between a client and a server. This mechanism ensures that even if a client disconnects and reconnects, it can retain its subscription and publishing state. When the MQTT server receives a connection request from a client, it checks the client ID. If the client identifier is new and hasn't been used before, the server accepts the connection request and assigns a new session to the client. If the client ID already exists in the server's session list, the server continues to use the existing session. This means that when a client disconnects and reconnects with the same client ID, the server restores the client's subscription information based on the previous session state. It then delivers relevant messages to the client, allowing it to continue receiving messages from the previously subscribed topics.

### MQTT Long Connection

MQTT employs a long connection approach to maintain persistent communication between clients and servers. This is achieved through TCP keep-alive and heartbeat mechanisms to sustain connection activity, prevent interruptions, and monitor connection availability. The detailed mechanisms are as follows:

- Heartbeat Mechanism: MQTT protocol defines a heartbeat mechanism to sustain activity between the client and server during the duration of the long connection. The client can periodically send PINGREQ messages to the server, and the server responds with a PINGRESP message. This regular heartbeat enables mutual confirmation of the connection's liveness.

- Keep Alive Interval: During the establishment of an MQTT connection, the client can negotiate a keep alive interval with the server. The keep alive interval refers to the time interval within which the client sends heartbeat requests to inform the server of the connection's activity. The server can use this interval to check the client's active status.

- Disconnection Monitoring: If the server doesn't receive heartbeat requests or other messages from the client within a certain period, it considers that the client has disconnected and terminates the connection. Similarly, the client can detect the server's disconnection and attempt reconnection.

In MQTT initialization, configuring a non-zero value indicates that the the keep alive mechanism is enabled by default. QuecPython will actively send heartbeat packets within the defined heartbeat interval with`umqtt.ping()` . [Click here](../../../api_reference/networklib/umqtt.md#MQTTClient.ping) to fins more details.

```python
MQTTClient.ping()
```

## MQTT Application

QuecPython provides the `umqtt` feature for client connection of MQTT protocol. For the usage of `umqtt`  interfaces, [click here](../../../api_reference/networklib/umqtt.md) for details.

This section will set up two MQTT clients for demonstration. For better demonstration, Client A is set up on a PC with the MQTT.fx tool and connect to the server. Client B is implemented with QuecPython `umqtt`. Once both clients are connected to the same server, they will push topic messages to each other, and the messages are forwarded between devices through the server. Before the MQTT application demonstration, please get a brief understanding of the process for implementing an MQTT client application with QuecPython in the following diagram.

![image-20231111160509411](../../media/network-comm/net-protocols/mqtt/image-20231111160509411.png)

The diagram above illustrates two clients connecting to the same server. Here's a general outline of the application flow:

1. Instantiate the MQTT class function by providing the MQTT server connection parameters. This instantiation returns a handle (or object) that provides access to all MQTT API methods, such as requesting connection, publishing, and subscribing.
2. Implement a callback function. Register this callback function to the MQTT object with the `set_callback` method. This callback function will be used to notify the client when topic messages are forwarded by the server to the client.
3. Use the `connect` method to initiate a client connection request to the server.
4. Once the client connection is established, use the `subscribe` method of the MQTT object to subscribe to the specific topic. Multiple event topics can be subscribed to.
5. Push topic messages from the client.  Send the topic information and the message to the server with the `publish` method. The server will then forward this message to all client devices that subscribe this topic.
6. Enable listening  for incoming messages from the client. Since MQTT is based on TCP and uses a blocking connection by default, you can open a separate thread task to listen for downlink data from the server. Notifications can be sent through the callback function.

### MQTT Client Workflow Overview

The entire MQTT application workflow is based on the publish-subscribe pattern, allowing devices and applications to communicate asynchronously and decoupled. By subscribing to different event topics, clients can receive messages corresponding to those topics, and through publishing messages, they can send data to the server and other subscribers, as described below:

1. Client Connection Establishment:
   - The client initiates a connection request to the MQTT server via a TCP connection.
   - The client provides a unique client ID to identify and differentiate the clients within the server.
   - Optionally, the client can provide a username and password for authentication to ensure connection security.

2. Topic Subscription:
   - The client can send a subscription request to the MQTT server to specify the topics to be subscribed to and the QoS level (Quality of Service level).
   - Subscribers can subscribe to one or more topics of interest to receive messages related to these topics.
   - The server maintains subscription relationships and forwards messages to subscribers when messages are published to the subscribed topics.

3. Message Publishing:
   - The client can publish messages to the MQTT server to specify the topic and message content to be published.
   - The message publisher sends the message to the server, and the server then forwards the message to all clients that have subscribed to the corresponding topic.
   - Depending on the message's QoS level, the server can send acknowledgment messages to the publisher to ensure reliable message delivery.

4. Message Delivery:
   - When the server receives a published message, it looks for all clients that have subscribed to the corresponding topic.
   - The server forwards the message to these subscribed clients, allowing them to receive the relevant message.
   - Message delivery is processed based on the subscriber's QoS level to ensure reliable message delivery.

5. Disconnecting:
   - Clients can send a disconnect request to the server at any time to terminate communication with the server.
   - After disconnecting, the server cleans up the client's related information and stops sending messages to that client.

### MQTT Client A

This demonstration uses MQTT.fx on the PC as Client A, fills in the required connection parameters and then connects the server.

An example of the parameters filled in is shown below.

![image-20230711104602875](../../media/network-comm/net-protocols/mqtt/image-20230711104602875.png)

After successfully connecting the server, Client A subscribes to Topic A, and the message is released to Topic A through the server. See as follows.

![image-20230711104709552](../../media/network-comm/net-protocols/mqtt/image-20230711104709552.png)

### MQTT Client B

The module uses `umqtt`  of QuecPython to set up the MQTT Client B, connects to the server and subscribes to Topic B. This will work in coordination with Client A to complete the message push and receive process. Please note that for using MQTT functionality in QuecPython, you need to ensure the following:

1. **Downlaod QuecPython Firmware**: Depending on your module model, flash the QuecPython firmware into your device.
2. **Connect to the Network**: Ensure that your device is properly connected to the network.

After downloading the firmware, you need to check whether the current firmware includes the `umqtt` module and check the network status. You can use [QPYcom](../../dev-tools/QPYcom/README.md) for debugging. The demonstration in this document is performed with this tool. On the interactive page of QPYcom, you can confirm whether the module contains and uses APIs to check the network status by importing the feature. In Python syntax, you can import APIs with `import xxx` or `from xx import xxx`.

```python
# Inclueded if no exception occurs
import umqtt
```

You can import the `checkNet` API to query the network status of the device. The status values are available in the [wiki](../../../api_reference/iotlib/checkNet.md#checkNet.waitNetworkReady).

```python
import checkNet
stage, state = checkNet.waitNetworkReady(30)
print(stage, state) # 3 1
```

Once you have confirmed that the device's network is functioning properly, you can create an MQTT object with the `umqtt` API. See as follows:

```python
# Import MQTT client class functions in umqtt module
from umqtt import MQTTClient

# The unique ID of the client
client_id = "QuecPython_cli_2023"
# MQTT server address
server = "mq.tongxinmao.com"
# MQTT server port
port = 18830
# Subscribe a topic
sub_topic_b = "/public/TEST/QuecPython2023_B"
# Create an MQTT connection and return
mqtt_cli_obj = MQTTClient(client_id, server, port)
```

Here an MQTT client object is successfully created. Next, you can use this object for further operations.

After creating the MQTT client object, you need to initiate a connection to the MQTT server. The `connect` method can help you accomplish this step. When calling the `connect` method, you do not need to provide the server address and port because they are already passed when the object is created. Therefore, they can be obtained in `connect` directly.

```python
# Retrun 0 if the execution is successful. Exceptional message return if failed.
mqtt_cli_obj.connect()
```

After successfully establishing a connection with the server, Client B attempts to subscribe to a topic. The topic can be customized, but in projects, topics are generally defined based on events. Here is an example of   a custom topic with QoS level of 0.

```python
mqtt_cli_obj.subscribe("/public/TEST/QuecPython2023_B", qos=0)
```

Once Client B has successfully subscribed to a topic, it needs to start listening for messages pushed from the server. This is achieved by enabling message listening task, which is encapsulated in the `umqtt.wait_msg()` method. Since this method is blocking and waits for messages, it should be run in a separate thread. You can find the `loop_forever` method in the example code.

```python
mqtt_cli_obj.loop_forever()
```

In the current state, if the server receives the push messages of the topic, it will forward them to the client. Here is an example of sending messages to the "/public/TEST/QuecPython2023_B" topic with Client A.

PC (Client A) sends a message to the "/public/TEST/QuecPython2023_B" topic, and Client B subscribed this topic, then the message is notified to to user through the callback function after being forwarded by the server.

![image-20230711112722629](../../media/network-comm/net-protocols/mqtt/image-20230711112722629.png)

The steps of client subscribing a topic and receiving the topic message pushed from the server is shown above. The `publish` method should be called to actively publish the topic message fron QuecPython (see as follows).

```python
mqtt_cli_obj.publish("/public/TEST/QuecPython2023_A", "Hello, PC", qos=0)
```

Running Result：

![image-20230711113002143](../../media/network-comm/net-protocols/mqtt/image-20230711113002143.png)

 In this demonstration, the message transmitting is completed between the two clients and the application flow of the MQTT client is displayed. Finally, here is the complete example code for reference.

Code example:

```python
import _thread
from umqtt import MQTTClient

class MqttClientManage(object):
    """
    Quecpython client manage
    """

    def __init__(self, client_id, server, port, user=None, password=None, keepalive=60, ssl=False, ssl_params=None,reconn=True):
        self.client_id = client_id
        self.server = server
        self.port = port
        self.user = user
        self.password = password
        self.keepalive = keepalive
        self.ssl = ssl
        self.ssl_params = ssl_params
        self.reconn = reconn
        self.client = MQTTClient(self.client_id, self.server, self.port, self.user, self.password,
                                 keepalive=self.keepalive, ssl=self.ssl, ssl_params=self.ssl_params,
                                 reconn=reconn)
    def connect(self):
        '''
        Connect MQTT Server
        '''
        return self.client.connect()

    def set_callback(self, sub_cb=None):
        '''
        Set MQTT callback message function
        '''
        if sub_cb is None:
            sub_cb = self.mqtt_cli_cb
        self.client.set_callback(sub_cb)

    def mqtt_cli_cb(self, topic, data):
        print("QuecPython receive Topic={},Msg={}".format(topic.decode(), data.decode()))

    def subscribe(self, topic, qos=0):
        '''
        Subscribe the topic
        '''
        return self.client.subscribe(topic, qos)

    def publish(self, topic, msg, qos=0):
        '''
        Publish the message
        '''
        return self.client.publish(topic, msg, qos)
    
    def disconnect(self):
        '''
        Disconnect
        '''
        return self.client.disconnect()

    def __listen(self):
        while True:
            try:
                self.client.wait_msg()
            except Exception as err:
                print("mqtt client listen error: " + str(err))

    def loop_forever(self):
        _thread.start_new_thread(self.__listen, ())
```

Before enabling the code on the module, you need to check the network status first. After confirming that the network is normal, initialize the `MqttClientManage` class function to start the client.

```python
import checkNet

client_id = "QuecPython_cli_2023"
server = "mq.tongxinmao.com"
port = 18830
# Device topic
sub_topic_b = "/public/TEST/QuecPython2023_B"
# Create MQTT connection object
mqtt_cli_obj = MqttClientManage(client_id, server, port)
# Register the message callback function
mqtt_cli_obj.set_callback()

stage, state = checkNet.waitNetworkReady(30)
if stage == 3 and state == 1: # Network status id normal
    # Request to connect the MQTT server
    connect_state = mqtt_cli_obj.connect()
    print("mqtt connect state: ", connect_state)
    # Subscribe to a topic of Client A
    sub_state = mqtt_cli_obj.subscribe(sub_topic_b)
    print("mqtt subscribe state: ", sub_state)
    # Start listening for messages
    mqtt_cli_obj.loop_forever()
else:
    print('Network connection failed, stage={}, state={}'.format(stage, state))
```

Upload the code to the module and run it, and then the module will connect to the MQTT server successfully.

![image-20230711112042927](../../media/network-comm/net-protocols/mqtt/image-20230711112042927.png)

### MQTT Server

When the server receives a published message, it looks for all clients that have subscribed to the topic associated with that message. Then the server forwards the message to these subscribed clients so that they can receive the message. Message forwarding can be handled according to the subscriber's QoS level to ensure the reliability of message delivery. For messages with QoS level of 1 or 2, the server sends an acknowledgment message to the publisher to confirm that the message has been successfully delivered to the subscribers.

This document selects a public MQTT server for demonstration. Actually you can use the MQTT server that you set up or other platforms that support the MQTT protocol.

Preparation of MQTT server connection information:

```python
# Public MQTT server address
server = "mq.tongxinmao.com"
# Public MQTT server port
port = 18830

# Client A subscribes the topic
sub_topic_a = "/public/TEST/QuecPython2023_A"
# Client B subscribes the topic
sub_topic_b = "/public/TEST/QuecPython2023_B"
```

## FAQs

**Q: How to troubleshoot a failed MQTT server connection?**

A: First, check the device's network status to ensure that the SIM card or other network components can access the remote server. Verify the correctness of the server address and whether certificate authentication is required. Ensure that the connection information is filled in correctly. Check if the server is accessible; you can try connecting with the MQTT.fx tool.

**Q: The device is unable to run Python code and cannot interact.**

A: Please check if the QuecPython firmware is correctly flashed into the device.

**Q: Unable to subscribe to a topic.**

A: Check if the topic permissions allow for subscription.

**Q: How to establish an encrypted MQTTS connection?**

A: When creating an MQTT connection object, set the `ssl` parameter to True. This parameter is False by default.

**Q: Why can't I receive QoS 1 messages even though the topic has been successfully subscribed to?**

A: When subscribing to a topic, make sure to pass in the QoS level that matches the QoS level of the message you are trying to publish.

**Q: How to handle reconnection in case of network issues?**

A: The `umqtt` module handles automatic reconnection internally.

