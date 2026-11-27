# MQTT 协议应用指导

本文档详细介绍了如何在 Quecpython 中使用 MQTT 协议进行通信，并提供了环境搭建、代码示例和结果演示。

## MQTT协议概述

MQTT是一种轻量级的消息传输协议，旨在在物联网（IoT）应用中实现设备间的可靠通信。它使用发布-订阅模式，其中包括一个MQTT服务端（代理或服务器）和多个MQTT客户端之间的通信。MQTT协议具有以下特点：

- 轻量级：MQTT协议设计简单，协议头部开销小，适用于资源受限的设备和网络。
- 低带宽消耗：MQTT采用二进制编码，有效地利用网络带宽。
- 异步通信：客户端可以随时发布和订阅消息，无需等待对方的响应。
- 发布-订阅模式：消息发布者将消息发布到特定的主题，而订阅者则订阅感兴趣的主题。这种模式支持松耦合的通信和灵活的消息传递。

## MQTT应用场景

MQTT在物联网、传感器网络、远程监测和控制、实时数据传输、消息推送和通知、车联网以及能源管理等众多领域都有重要作用。其轻量级、低带宽消耗和可靠的消息传递机制使得MQTT成为许多应用中的首选通信协议，下面列举一些常用的场景简介：

![image-20230711185611691](../../media/network-comm/net-protocols/mqtt/image-20230711185611691.png)

1. 物联网（IoT）：MQTT是物联网应用中最常用的通信协议之一。它适用于连接大量设备的场景，提供可靠的消息传递和实时通信。MQTT的轻量级特性使其能够在资源受限的设备和网络环境下运行，同时支持发布-订阅模式和异步通信，使设备能够实时交换信息、监测和控制。
2. 传感器网络：MQTT可用于传感器网络中的数据收集和实时监控。传感器可以发布数据到特定的主题，而订阅者可以订阅感兴趣的主题以接收传感器数据。MQTT的低带宽消耗和高效的消息传递机制使其成为传感器网络中可靠的数据传输选择。
3. 远程监测和控制：MQTT使得远程监测和控制变得简单而可靠。通过MQTT，可以实时监测设备的状态、传感器数据，以及远程控制设备的操作。这在许多应用中都非常有用，如智能家居、智能城市、工业自动化等。
4. 实时数据传输：MQTT的异步通信机制使其非常适用于实时数据传输场景。它能够以极低的延迟传输数据，使实时监控和通信成为可能。例如，在金融交易系统中，MQTT可用于实时传输市场行情数据给交易系统和投资者。
5. 消息推送和通知：MQTT的发布-订阅模式使其成为消息推送和通知的理想选择。服务端可以发布消息到特定的主题，而订阅者将实时接收到这些消息。这在应用程序中的推送通知、聊天应用和实时数据更新等方面非常有用。
6. 车联网（Connected Car）：MQTT可用于车辆间的通信和车辆与云平台的通信。它可以支持车辆状态监测、车辆诊断和远程控制等功能，以提高车辆的安全性和效率。
7. 能源管理：MQTT可用于能源监测和管理系统中的数据传输。通过MQTT，能源消耗数据可以实时传输给监测和分析系统，以便进行能源消耗优化和监控。

## MQTT通信机制

在MQTT中，通信是基于客户端与服务端之间的TCP连接完成的。客户端通过发送CONNECT消息发起连接请求，并在连接建立后发送相应的操作消息，如SUBSCRIBE（订阅）、PUBLISH（发布）、UNSUBSCRIBE（取消订阅）等。

发布者在向特定主题发布消息时，将消息内容和指定的主题发送给MQTT服务端，然后服务端将消息传递给订阅了该主题的所有订阅者。

订阅者通过发送SUBSCRIBE消息来订阅感兴趣的主题，其中指定了主题和所需的QoS级别。服务端接收到订阅请求后，会记录订阅者的订阅信息，并在有相关消息发布时将其传递给订阅者。

MQTT还支持保留消息（Retained Message），即发布者可以发布保留消息，并设置保留标志。保留消息将由服务端保留，以便在有订阅者订阅相关主题时发送给订阅者。这样，新的订阅者可以接收到最新的保留消息。

此外，MQTT还提供持久化会话（Persistent Session）的机制，持久化会话允许客户端断开连接后重新连接时保留其订阅和发布的状态信息，确保不丢失重要的消息。

通过这些机制，MQTT实现了可靠的消息传递、解耦和异步的实时通信，适用于物联网、传感器网络和实时数据传输等场景。它提供了灵活的通信模型和机制，使设备和应用程序能够高效地进行消息交互。

![image-20230701163911290](../../media/network-comm/net-protocols/mqtt/mqtt_1.png)

- MQTT客户端：MQTT客户端是连接到MQTT服务端的设备或应用程序，每个客户端都具有唯一的客户端标识符（Client Identifier），用于在服务端中识别和区分不同的客户端。在QuecPython中我们通过`umqtt`实现MQTT客户端，通过传入初始化连接参数创建连接对象，[点击查看详情](../../../api_reference/networklib/umqtt.md#umqtt.MQTTClient)。

  ```python
  from umqtt import MQTTClient
  MQTTClient(client_id, server, port=0, user=None, password=None, keepalive=0, ssl=False, ssl_params={},reconn=True,version=4)
  ```

  **参数介绍：**

  - `client_id` - 客户端 ID，字符串类型，具有唯一性。
  - `server` - 服务端地址，字符串类型，可以是 IP 或者域名。
  - `port` - 服务器端口（可选），整数类型，默认为1883，请注意，MQTT over SSL/TLS的默认端口是8883。
  - `user` - （可选) 在服务器上注册的用户名，字符串类型。
  - `password` - （可选) 在服务器上注册的密码，字符串类型。
  - `keepalive` - （可选）客户端的keepalive超时值，整数类型，默认为0。
  - `ssl` - （可选）是否使能 SSL/TLS 支持，布尔值类型。
  - `ssl_params` - （可选）SSL/TLS 参数，字符串类型。
  - `reconn` - （可选）控制是否使用内部重连的标志，布尔值类型，默认开启为True。
  - `version` - （可选）选择使用mqtt版本，整数类型，version=3开启MQTTv3.1，默认version=4开启MQTTv3.1.1。

- MQTT服务端：MQTT服务端（也称为代理或服务器）负责接收和转发消息，管理订阅关系，并维护客户端的连接。它监听指定的端口，等待客户端的连接请求，并根据客户端的请求进行相应的处理，我们可以通过服务端主动下发主题消息，订阅该主题的客户端设备都会收到消息。

### 发布-订阅

- 发布者（Publisher）：发布者是MQTT中的消息发送方，发布者将消息发布到特定的主题（Topic），并通过MQTT服务端将消息传递给订阅该主题的所有订阅者，基于`umqtt`创建客户端对象后可使用[publish](../../../api_reference/networklib/umqtt.md#MQTTClient.publish)方法进行消息发布：

  ```python
  MQTTClient.publish(topic, msg, retain=False, qos=0)
  ```

  **参数描述：**

  - `topic` - mqtt 消息主题，字符串类型
  - `msg` - 需要发送的数据，字符串类型
  - `retain` - 布尔值类型，默认为False, 发布消息时把retain设置为true，即为保留消息。
    MQTT服务器会将最近收到的一条RETAIN标志位为True的消息保存在服务器端, 每当MQTT客户端连接到MQTT服务器并订阅了某个topic，如果该topic下有Retained消息，那么MQTT服务器会立即向客户端推送该条Retained消息。
    特别注意：MQTT服务器只会为每一个Topic保存最近收到的一条RETAIN标志位为True的消息！也就是说，如果MQTT服务器上已经为某个Topic保存了一条Retained消息，当客户端再次发布一条新的Retained消息，那么服务器上原来的那条消息会被覆盖！ |
  - `qos` - 整数类型， MQTT消息服务质量（默认0，可选择0或1）0：发送者只发送一次消息，不进行重试 1：发送者最少发送一次消息，确保消息到达Broker

- 订阅者（Subscriber）：订阅者是MQTT中的消息接收方，订阅者可以订阅感兴趣的主题，以接收与该主题相关的消息。一旦订阅者订阅了某个主题，它将接收到该主题下的所有发布消息,项目中一般以不同的事件来定义Topic，设备订阅事件主题后即可接收到该主题的推送消息。

  [点击查看示例API详情](../../../api_reference/networklib/umqtt.md#MQTTClient.subscribe)

  ```python
  MQTTClient.subscribe(topic,qos)
  ```

  **参数描述：**

  - `topic` - mqtt topic主题，字符串类型。
  - `qos` - MQTT消息服务质量（默认0，可选择0或1），整数类型0：发送者只发送一次消息，不进行重试 1：发送者最少发送一次消息，确保消息到达Broker。

- 主题（Topic）：主题是MQTT中消息发布和订阅的标识符，它可以是层次结构的，使用斜杠（/）分隔不同的层级，例如"Quectel/Python/temperature"。主题用于组织消息的传递，发布者将消息发布到特定的主题，而订阅者则订阅感兴趣的主题。

- 消息传递：MQTT服务端负责接收发布者发送的消息，并根据主题将消息传递给订阅了相应主题的订阅者。当发布者发布消息到某个主题时，所有订阅了该主题的订阅者将接收到该消息。

  ![image-20230712145141895](../../media/network-comm/net-protocols/mqtt/image-20230712145141895.png)

- 解耦和灵活性：发布-订阅模型使得发布者和订阅者之间解耦，它们不需要直接相互通信。发布者只需要将消息发布到特定的主题，而不需要知道谁将接收消息。同样，订阅者只需要订阅感兴趣的主题，而不需要知道消息来自哪个发布者。这种解耦和灵活性使得系统更加可扩展和灵活。

- 异步通信：在发布-订阅模型中，发布者和订阅者之间是异步通信的。发布者可以随时发布消息，而不需要等待订阅者的响应。订阅者可以接收到发布者发布的消息，并在需要时对消息进行处理。

### 服务质量（QoS）级别

MQTT定义了三个不同的QoS级别，用于控制消息传递的可靠性和保证。

- QoS 0（最多一次）：消息发布者发布消息，只发送一次，没有确认机制。此级别的消息可能会丢失或重复传递，适用于不强调可靠性的场景。
- QoS 1（最少一次）：消息发布者发布消息，确保至少传递一次，但可能导致重复传递。使用发布和确认机制实现可靠传递，适用于需要至少一次传递保证的场景。
- QoS 2（恰好一次）：消息发布者发布消息，确保恰好传递一次，通过两次握手和四次握手确认机制实现。此级别提供了最高的传递可靠性，适用于对传递准确性要求很高的场景。

QuecPython中设置QoS等级，[详情点击查看](../../../api_reference/networklib/umqtt.md#MQTTClient.subscribe)。

```python
"""
当一个客户端以QoS 0订阅了一个主题，然后向该主题发布了QoS为1或2的消息时，该客户端将无法收到自己所发布的消息.
这是因为QoS级别是针对消息传递的可靠性而定义的。在MQTT中，消息的QoS级别是在发布时指定的，而订阅时指定的QoS级别仅适用于接收到的消息。

对于QoS 0，发布者发送消息后不会收到任何确认或响应，也不会对消息进行重传。这意味着即使发布者自己订阅了相同的主题，也无法保证接收到自己所发布的消息。

如果发布者希望确保接收到自己所发布的消息，应将发布消息的QoS级别设置为1或2，并在订阅时使用相应的QoS级别。这样，客户端将在发布消息时获得确认，并在QoS级别为1或2的情况下进行重传，以确保消息的可靠传递。
"""
MQTTClient.subscribe("Quectel/Python/demo",1)
MQTTClient.publish("Quectel/Python/demo","Hello", qos=1)
```

### 遗嘱消息

MQTT允许客户端在连接时设置遗嘱消息（LWT），在建立连接的过程中，客户端可以设置遗嘱消息的相关参数，包括遗嘱消息的主题（Topic），遗嘱消息的内容和QoS级别。当服务端检测到客户端未在保活时间内上报心跳包，且未发送关闭连接请求，则认为客户端为异常断开，会根据客户端设置的遗嘱消息，将遗嘱消息发布到设置的遗嘱主题，这样其他订阅了该主题的订阅者就可以接收到该遗嘱消息，以得知该客户端的离线状态。

下面`set_last_will`为QuecPython设置遗嘱消息的API介绍，详情请[点击](../../../api_reference/networklib/umqtt.md#MQTTClient.set_last_will)：

```python
MQTTClient.set_last_will(topic,msg,retain=False,qos=0)
```

**参数描述：**

- `topic` - mqtt遗嘱主题，字符串类型。
- `msg` - 遗嘱的内容，字符串类型。
- `retain` - retain为True 表示服务器会一直保留消息，默认False，布尔值类型。
- `qos` - 整数类型，消息服务质量(0~1)。

### 持久化会话

MQTT支持持久化会话是一种机制，用于在客户端与服务端之间保持会话状态，即使客户端断开连接后重新连接，也能够保留订阅和发布的状态信息。MQTT服务端接收到客户端的连接请求后，会检查客户端标识符，如果客户端标识符是一个新的、未使用过的标识符，服务端将接受连接请求并为客户端分配一个新的会话，如果客户端标识符已经存在于服务端的会话列表中，服务端会继续使用现有的会话。也就是说客户端断开重新连接后，只要Client ID与上次连接一致，服务端将根据之前保存的会话状态，恢复该客户端的订阅信息，并将相关消息传递给客户端，这样客户端可以继续接收之前订阅的主题的消息。

### MQTT长连接

MQTT通过使用长连接（Long Connection）来保持客户端和服务端之间的持久通信。TCP保持活动机制和心跳保持机制用于保持连接的活跃状态，避免连接被中断，并检测连接的可用性，详细介绍如下：

- 心跳保持机制：MQTT协议定义了心跳保持机制，用于在长连接期间维持客户端和服务端之间的活动状态。客户端可以定期发送PINGREQ消息给服务端，服务端会回复PINGRESP消息作为响应。通过定期的心跳保持，客户端和服务端可以互相确认连接的存活状态。

- 心跳间隔（Keep Alive）：在建立MQTT连接时，客户端可以协商与服务端的心跳间隔（Keep Alive Interval）。心跳间隔是指客户端在该时间间隔内发送心跳请求，用于告知服务端连接的活跃性。服务端可以根据心跳间隔来检测客户端是否处于活动状态。

- 断开连接检测：如果服务端在一段时间内没有收到客户端的心跳请求或其他消息，它会认为客户端断开连接，并终止连接。同样地，客户端也可以检测到服务端的断开连接，并尝试重新连接。

MQTT初始化时配置Keep Alive不为0时默认开启保活机制，QuecPython会主动在心跳间隔时间内发送心跳包，使用的是`umqtt.ping()`方法，[点击查看详情](../../../api_reference/networklib/umqtt.md#MQTTClient.ping)：

```python
MQTTClient.ping()
```

## MQTT应用

QuecPython 提供了`umqtt`模块，用于MQTT协议的客户端连接。关于`umqtt`模块接口的用法，[点此查看](../../../api_reference/networklib/umqtt.md)。

本章节将搭建两个MQTT客户端进行演示，为方便演示效果，客户端A在PC端使用MQTT.fx工具完成搭建并连接到服务端，客户端B使用QuecPython的`umqtt`搭建。当两个客户端连接到同一服务端后，相互向对方推送主题消息，经服务端将消息转发至对方设备。在开始介绍MQTT应用演示之前，我们先通过下图初步了解下基于QuecPython完成MQTT客户端的应用流程：

![image-20230711162623192](../../media/network-comm/net-protocols/mqtt/image-20230711162623192.png)

上图以两个客户端连接到同一个服务端为示例描述应用流程：

1. 传入MQTT服务端连接参数给MQTT类函数进行实例化，返回一个可操作的句柄，Python里面称之为对象，该对象拥有mqtt所有的API方法，例如请求连接，发布订阅等。
2. 实现一个回调函数，通过`set_callback`方法将该回调函数注册到创建的mqtt对象中，当服务端转发主题消息到客户端时通过注册的回调函数进行通知。
3. 执行`connect`方法向服务端发起客户端连接请求。
4. 客户端连接建立成功后通过mqtt对象的`subscribe`方法完成主题订阅，可订阅多个事件主题。
5. 客户端主动推送主题消息，使用`publish`方法传入主题信息以及待发送消息至服务端，服务端会转发该主题消息至订阅者，即订阅该主题的所有客户端设备。
6. 客户端开启消息监听，MQTT协议是基于TCP协议进行连接的，默认都是阻塞形式，所以我们通过开启一个线程任务的方式监听服务端下行数据，并通过回调的方式通知使用者。

### MQTT客户端流程介绍

整个MQTT应用流程基于发布-订阅模型，允许设备和应用程序之间以异步、解耦的方式进行通信。通过订阅不同的事件主题，客户端可以接收对应主题的消息，并通过发布消息将数据发送给服务端和其他订阅者，如下描述：

1. 客户端连接建立：
   - 客户端通过TCP连接向MQTT服务端发起连接请求。
   - 客户端提供唯一的客户端标识符（Client Identifier），以便在服务端中标识和区分不同的客户端。
   - 可选地，客户端可以提供用户名和密码进行身份验证，以确保连接的安全性。
2. 订阅主题：
   - 客户端可以向MQTT服务端发送订阅请求，指定要订阅的主题和所需的QoS级别（服务质量级别）。
   - 订阅者可以订阅一个或多个感兴趣的主题，以接收与这些主题相关的消息。
   - 服务端维护订阅关系，并在有消息发布到订阅的主题时将消息传递给订阅者。
3. 发布消息：
   - 客户端可以向MQTT服务端发布消息，指定要发布的主题和消息内容。
   - 消息发布者将消息发送给服务端，服务端接收到消息后根据主题转发给已订阅该主题的所有客户端。
   - 服务端可以根据消息的QoS级别，将确认消息发送给发布者，以确保消息的可靠传递。
4. 消息传递：
   - 当服务端接收到发布的消息后，根据消息的主题，查找所有已订阅该主题的客户端。
   - 服务端将消息转发给这些订阅的客户端，以便它们可以接收到相应的消息。
   - 消息传递可以根据订阅者的QoS级别进行处理，以确保消息的可靠性传递。
5. 断开连接：
   - 客户端可以随时向服务端发送断开连接请求，终止与服务端的通信。
   - 断开连接后，服务端将清理客户端的相关信息，并停止向其发送消息。

### MQTT客户端A

本次演示我们在PC端使用MQTT.fx作为客户端A，填入所需的连接参数后进行服务端的连接。

参数填写如下图：

![image-20230711104602875](../../media/network-comm/net-protocols/mqtt/image-20230711104602875.png)

在成功连接上服务端后，客户端A订阅A主题，通过服务端向A主题发布消息，如下图：

![image-20230711104709552](../../media/network-comm/net-protocols/mqtt/image-20230711104709552.png)

### MQTT客户端B

模组端使用QuecPython的`umqtt`模块搭建MQTT客户端B来连接服务端，并订阅主题B，配合客户端A完成消息推送与接收，需要注意的是我们要在 Quecpython 中使用 MQTT 功能，您需要确保您的设备满足以下要求：

1. **烧录Quecpython 固件**：请根据您的模组型号，将 Quecpython 固件烧录到您的设备上。
2. **连接到网络**：确保您的设备已正确连接到网络。

完成固件烧录后需要检测当前固件是否包含`umqtt`模块以及找网状态，可使用[Qpycom工具](../../dev-tools/QPYcom/qpycom-repl.md)进行调试，本文演示均使用该工具。在工具交互页面通过导入模块的方式来确认是否包含和使用API检测网络情况，python语法通过`import xxx`或`from xx import xxx`的方式导入API。

```python
# 未抛出异常则包含
import umqtt
```

通过导入API`checkNet`来进行查询设备网络情况，状态值请查看[wiki](../../../api_reference/iotlib/checkNet.md#checkNet.waitNetworkReady)。

```python
import checkNet
stage, state = checkNet.waitNetworkReady(30)
print(stage, state) # 3 1
```

我们在确认设备网络正常的情况下使用`umqtt`API创建mqtt对象，如下：

```python
# 导入umqtt模块下的MQTTClient类函数
from umqtt import MQTTClient

# 客户端唯一标识ID
client_id = "QuecPython_cli_2023"
# MQTT服务端地址
server = "mq.tongxinmao.com"
# MQTT服务端端口
port = 18830
# 订阅主题
sub_topic_b = "/public/TEST/QuecPython2023_B"
# 创建mqtt连接对象并返回
mqtt_cli_obj = MQTTClient(client_id, server, port)
```

这样我们就成功创建了一个mqtt客户端对象，接下来我们可以使用该对象进行后续操作。

创建mqtt客户端对象后需要主动请求连接服务端，`connect`方法可以帮助我们完成这一步骤，使用`connect`方法需要server地址和端口两个参数，但我们通过mqtt对象来调用时不需要，因为我们在创建该对象时已经将这两个信息完成了传入，`connect`方法中可以直接获取到。

```python
# 成功返回0，失败则抛出异常
mqtt_cli_obj.connect()
```

成功与服务端建立连接后尝试订阅一个主题，主题可以是自定义的，不过项目中一般以事件来划分主题，这里我们演示一个自定义主题,Qos等级为0：

```python
mqtt_cli_obj.subscribe("/public/TEST/QuecPython2023_B", qos=0)
```

客户端B订阅主题成功后就要开始监听服务端的推送消息了，需要开启消息监听任务，该方法是通过`umqtt.wait_msg()`方法封装而来，因该方法是阻塞等待，所以通过线程的方式单独运行,我们可以在示例代码中找到`loop_forever`方法：

```python
mqtt_cli_obj.loop_forever()
```

当前状态下如果服务端收到该主题的推送消息会直接转发给客户端，下面我们使用客户端A给`"/public/TEST/QuecPython2023_B"`这个主题发送消息来模拟，结果如下：

PC（客户端A）向主题"/public/TEST/QuecPython2023_B"发送消息，客户端B订阅了该主题，经服务端转发后由回调通知到用户。

![image-20230711112722629](../../media/network-comm/net-protocols/mqtt/image-20230711112722629.png)

上述演示了客户端订阅主题后成功接收到服务端推送的主题消息，在QuecPython中主动发布主题消息需要使用`publish`方法，如下：

```python
mqtt_cli_obj.publish("/public/TEST/QuecPython2023_A", "Hello, PC", qos=0)
```

结果图示：

![image-20230711113002143](../../media/network-comm/net-protocols/mqtt/image-20230711113002143.png)

本次演示我们通过两个客户端完成了消息流转，以及MQTT客户端的应用流程，最后给出完整的示例代码供参考。

示例代码如下：

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
        连接mqtt Server
        '''
        return self.client.connect()

    def set_callback(self, sub_cb=None):
        '''
        设置mqtt回调消息函数
        '''
        if sub_cb is None:
            sub_cb = self.mqtt_cli_cb
        self.client.set_callback(sub_cb)

    def mqtt_cli_cb(self, topic, data):
        print("QuecPython receive Topic={},Msg={}".format(topic.decode(), data.decode()))

    def subscribe(self, topic, qos=0):
        '''
        订阅Topic
        '''
        return self.client.subscribe(topic, qos)

    def publish(self, topic, msg, qos=0):
        '''
        发布消息
        '''
        return self.client.publish(topic, msg, qos)

    def disconnect(self):
        '''
        关闭连接
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

在启动模组端代码之前，需先检查网络状态，在确认网络正常的情况下，初始化`MqttClientManage`类函数启动客户端。

```python
import checkNet

client_id = "QuecPython_cli_2023"
server = "mq.tongxinmao.com"
port = 18830
# 设备主题
sub_topic_b = "/public/TEST/QuecPython2023_B"
# 创建mqtt连接对象
mqtt_cli_obj = MqttClientManage(client_id, server, port)
# 注册消息回调
mqtt_cli_obj.set_callback()

stage, state = checkNet.waitNetworkReady(30)
if stage == 3 and state == 1: # 网络状态正常
    # 请求连接mqtt服务器
    connect_state = mqtt_cli_obj.connect()
    print("mqtt connect state: ", connect_state)
    # 订阅客户端A主题
    sub_state = mqtt_cli_obj.subscribe(sub_topic_b)
    print("mqtt subscribe state: ", sub_state)
    # 启动消息监听
    mqtt_cli_obj.loop_forever()
else:
    print('Network connection failed, stage={}, state={}'.format(stage, state))
```

将此代码上传至模组端后运行，成功连接至MQTT服务端，结果如下图：

![image-20230711112042927](../../media/network-comm/net-protocols/mqtt/image-20230711112042927.png)

### MQTT服务端

当服务端接收到发布的消息时，它将根据消息的主题，查找所有已订阅该主题的客户端。然后，服务端将消息转发给这些订阅的客户端，以便它们可以接收到相应的消息。消息的转发可以根据订阅者的QoS级别进行处理，以确保消息的可靠性传递。对于QoS级别为1和2的消息，服务端会发送确认消息给发布者，以确保消息已成功传递给订阅者。

此文档选择一个公共MQTT服务器进行演示，实际可使用自己搭建的MQTT服务以及其它支持MQTT协议的平台。

MQTT服务器连接信息准备：

```python
# 公共MQTT服务器地址
server = "mq.tongxinmao.com"
# 公共MQTT服务器端口
port = 18830

# A客户端订阅topic信息
sub_topic_a = "/public/TEST/QuecPython2023_A"
# B客户端订阅topic信息
sub_topic_b = "/public/TEST/QuecPython2023_B"
```

## 常见问题

**Q: 连接MQTT服务器失败怎么排查原因？**

A：首先排查设备网络状态，确认SIM卡或其他网络能正常访问远端；服务端地址以及是否需要证书认证，连接信息是否填写正确；服务端是否可以正常访问，可通过MQTT.fx工具尝试是否可以连接。

**Q:设备无法正常运行python代码，也无法交互。**

A：请检查是否正确烧录QuecPython固件

**Q：订阅topic失败**

A：检查该topic权限是否支持订阅

**Q：如何进行MQTTS加密连接？**

A：在创建MQTT连接对象时将参数ssl置为True即可，该参数默认为False

**Q:   为什么发布qos等级1的消息无法收到，topic已成功订阅**

A：订阅topic时传入qos，与当前发布的消息等级保持一致

**Q：网络异常后如何进行异常重连？**

A：umqtt模块内部会进行自动重连
