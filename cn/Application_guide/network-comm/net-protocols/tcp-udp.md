# TCP/UDP 协议应用指导

## 概述

### IP 地址与域名

IP 地址是网络中的主机地址，用于两台网络主机能够互相找到彼此，这也是网络通信能够成功进行的基础。IP 地址一般以点分十进制的字符串来表示，如`192.168.1.1`。

![](../../media/network-comm/net-protocols/tcp-udp/ip-address.png)

我们日常访问的网站，其所在的服务器主机都有唯一的 IP 地址，网络中的主机不计其数，靠记 IP 地址的方式来区分不同的主机显然比较困难，并且同一个网站可能有多个不同的 IP 地址，或者 IP 地址会因为某种原因而更换。

因此，用域名表示网站地址的方式便应运而生，如我们常见的`www.baidu.com`比 IP 地址更容易被记住。因为实际的网络通信报文中使用的仍然是 IP 地址，所以需要使用**域名解析**协议去获取域名背后所对应的 IP 地址。

> 下文的讲解均以 IPv4 协议为基础。

### OSI 七层模型

国际标准化组织（ISO）制定的一个用于计算机或通信系统的标准体系，一般被称为 OSI（Open System Interconnection）七层模型。它为网络通信协议的实现提供了一个标准，通信双方在相同的层使用相同的协议，即可进行通信；就同一台设备而言，下层协议为上层协议提供了调用接口，将上层协议打包为底层协议，最终发送到网络上进行传输。

这七层分别为：应用层、表示层、会话层、传输层、网络层、数据链路层、物理层。

![](../../media/network-comm/net-protocols/tcp-udp/osi-7-layers.png)

为了简化协议实现或者方便理解，五层模型或者四层模型的概念也诞生了。四层模型一般被提及的比较多，包括：应用层、传输层、网络层、网络接口层。

![](../../media/network-comm/net-protocols/tcp-udp/tcpip-4-layers.png)

上文中的 IP 地址则属于网络层。

> 网络层用于把该主机所有的网络数据转发到网卡，经由物理层电路发送到网络中去。

为了方便阐述，下文将按照四层模型来进行讲解。

### 传输层协议

IP 地址解决了网络中两台主机如何能够找到彼此，进而进行报文收发的问题。

试想下，一台主机上可能运行着多个应用程序，执行着不同的网络任务。这时，某台 IP 地址的主机收到了另一台主机的报文，这个报文数据要传递给哪个应用程序呢？

为了解决这个问题，人们基于网络层协议演化出了传输层协议，传输层协议为本地的网络应用分配不同的端口。收到网络层的报文后，根据不同的端口号，将数据递交给不同的应用。

为了应对不同的场景，传输层协议分为 UDP 和 TCP 协议。

#### UDP 协议

UDP 协议具有以下特点：

- 无连接
- 支持一对一、一对多和多对多通信
- 不保证可靠交付
- 全双工通信
- 面向报文

根据不同的需求，基于 UDP 衍生出了一些应用层协议，不同的应用会默认指定一个端口号。端口号亦可根据实际情况更换。

常见的基于 UDP 的应用协议及端口如下：

| 熟知端口号 | 协议       | 说明                |
| ---------- | ---------- | ------------------- |
| 0          | --         | 保留                |
| 7          | echo       | 报文回送服务器      |
| 53         | nameserver | 域名服务器          |
| 67         | bootps     | BOOT 或 DHCP 服务器 |
| 68         | bootpc     | BOOT 或 DHCP 客户端 |
| 69         | TFTP       | 简单文件传输协议    |
| 123        | NTP        | 网络时间协议        |
| 161        | SNMP       | 简单网络管理协议    |

#### TCP 协议

TCP 协议具有以下特点：

- 面向连接
- 每条连接只能有两个端点，即点对点
- 提供可靠的数据交付
- 全双工通信
- 面向字节流

根据不同的需求，基于 TCP 衍生出了一些应用层协议，不同的应用会默认指定一个端口号。端口号亦可根据实际情况更换。

常见的基于 TCP 的应用协议及端口如下：

| 熟知端口号 | 协议     | 说明                 |
| ---------- | -------- | -------------------- |
| 0          | --       | 保留                 |
| 7          | echo     | 报文回送服务器       |
| 20         | FTP-DATA | 文件传输协议（数据） |
| 21         | FTP      | 文件传输协议         |
| 23         | Telnet   | 终端连接             |
| 25         | SMTP     | 简单邮件传输协议     |
| 53         | DNS      | 域名服务器           |
| 80         | HTTP     | HTTP服务器           |
| 110        | POP3     | 邮局协议版本3        |
| 1080       | SOCKS    | 代理服务器协议       |

确定TCP连接的五元组：协议类型（TCP）、本地 IP、本地端口、远端 IP、远端端口。

## socket 编程

QuecPython 提供了`usocket`模块，用于网络通信的 socket 编程。关于`usocket`模块接口的用法，[点此查看](../../../api_reference/stdlib/usocket.md)。

本节分为 TCP 网络编程、UDP 网络编程和多网卡网络编程三部分。

### TCP 网络编程

在开始 TCP 网络编程之前，我们先通过下图，初步了解下 TCP 服务器与客户端的 socket 编程模型：

![](../../media/network-comm/net-protocols/tcp-udp/tcp-socket-model.png)

#### TCP 客户端网络编程

上图的右侧是最简的 TCP 客户端编程的接口调用流程：

1. 调用`socket()`接口创建 socket 对象。

2. 调用`connect()`接口连接服务器。

3. 调用`send()`接口向服务器发送数据。

4. 调用`recv()`接口接收服务器下发的数据。

5. 循环执行第 3 步和第 4 步，业务满足一定条件或连接断开，调用`close()`接口关闭 socket，释放资源。

几乎所有编程语言实现的 socket 接口，默认都是阻塞模式的，即所有涉及到网络报文收发的接口，如`connect()`、`send()`、`recv()`、`close()`等，默认都是阻塞式接口。

开始之前，我们先了解上图中 5 个与 TCP 客户端编程相关的接口的用法：

- `usocket.socket(af=AF_INET, type=SOCK_STREAM, proto=IPPROTO_TCP)`：创建 socket 对象
    - `af`：地址族，取值为`usocket.AF_INET`和`usocket.AF_INET6`，分别表示 IPv4 地址与 IPv6 地址。
    - `type`：socket 类型，取值为`usocket.SOCK_STREAM`、`usocket.SOCK_DGRAM`、`usocket.SOCK_RAW`，分别表示 流套接字、数据报套接字与原始套接字。
    - `proto`：协议类型，取值为`usocket.IPPROTO_TCP`、`usocket.IPPROTO_UDP`、`IPPROTO_TCP_SER`等，分别表示 TCP 协议、 UDP 协议、TCP 服务器等。
- `sock.connect(address)`：建立连接
    - `address`：包含 IP 地址字符串与端口号的元组或列表。
- `sock.send(data)`：发送数据，返回实际发送的数据长度。
    - `data`：bytes 类型的数据
- `sock.recv(size)`：接收数据，返回接收到的 bytes 类型的数据
    - `size`：读取数据的长度
- `sock.close()`：关闭 socket。

> `IPPROTO_TCP_SER`是 QuecPython 特定的，非标的，在 QuecPython 中进行 TCP server 编程时，`socket()`接口的第三个参数必须是`IPPROTO_TCP_SER`。

我们可以基于此来实现一个最简单的 TCP 客户端功能：连接百度服务器，手动组织 HTTP 报文并发送给服务器，循环接收服务器下发的 html 页面内容，直至接收完毕。

代码示例如下：

```python
import usocket

def tcp_client(address, port):
    # Create a socket object
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.IPPROTO_TCP)
    print('socket object created.')
    
    # Connect to the TCP server
    sock.connect((address, port))
    print('tcp link established: %s, %s' % (address, port))
    
    # Package user data
    data = 'GET / HTTP/1.1\r\n'
    data += 'Host: ' + address + ':' + str(port) + '\r\n'
    data += 'Connection: close\r\n'
    data += '\r\n'
    data = data.encode()
    
    # Send the data
    sock.send(data)
    print('<-- send data:')
    print(data)
    
    # Receive the data
    print('--> recv data:')
    while True:
        try:
            data = sock.recv(1024)
            print(data)
        except:
            # Connection ends until the data is fully received
            print('tcp disconnected.')
            sock.close()
            break
```

上述示例中，我们组织的 HTTP 请求报文是：

```http
GET / HTTP/1.1<cr><lf>
Host: <host_addr>:<port><cr><lf>
Connection: close<cr><lf>
<cr><lf>
```

该报文中：

- `<cr><lf>`：回车换行。
- `<host_addr>`：服务器的域名或 IP 地址。
- `<port>`：服务器的端口号。

> 需要注意的是，上述 HTTP 报文中添加了`Connection: close`头域，用来通知服务器这是一个短连接，当 HTTP 请求完成时，服务器会主动断开连接。
>
> 这么做的好处是：当服务器主动断开连接时，网络协议栈底层会抛出异常，用户可以通过捕获该异常，得知客户端也需要断开连接了。
>
> 这是为了快速实现 TCP 客户端测试代码的取巧做法。如果不增加`Connection: close`头域，对于 HTTP/1.1 协议来说，默认是长连接，服务器不会主动断开连接。此时就需要在客户端增加解析 HTTP 报文的代码，通过解析报文，判断数据被全部接收后，客户端才能断开连接。这会增加测试代码的复杂度，而且 HTTP 报文的解析也不在本文的阐述范围内。

请求报文组织完成后，通过`encode()`方法将字符串转换为 utf-8 编码的 bytes 类型，即上述代码中的`data = data.encode()`。而后调用`sock.send(data)`接口发送数据。

服务器下发的数据可能会很长，我们无法预知，因此需要在 while 循环中反复调用`sock.recv(1024)`接口读取数据，直到业务满足一定条件或连接异常，而后调用`sock.close()`关闭连接。

上文中说道，由于自行组织的 HTTP 报文中添加了`Connection: close`头域，服务器会主动断开连接，导致模组协议栈底层抛异常。所以在 while 循环中需要对`sock.recv(1024)`做`try`操作，当捕获到异常后，在`except`下断开连接，同时退出循环。

在启动客户端代码之前，需先检查网络状态，在确认网络正常的情况下，调用`tcp_client()`函数启动客户端。

```python
import checkNet

if __name__ == '__main__':
    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1: # Network connection is normal
        print('Network connection successful.')
        tcp_client('36.152.44.95', 80) # Start the client
    else:
        print('Network connection failed, stage={}, state={}'.format(stage, state))
```

上述代码中，我们直接使用了百度服务器的 IP 地址`36.152.44.95`，运行结果如下：

```log
Network connection successful.
socket object created.
tcp link established: 36.152.44.95, 80
<-- send data:
b'GET / HTTP/1.1\r\nHost: 36.152.44.95:80\r\nConnection: close\r\n\r\n'
--> recv data:
b'HTTP/1.1 200 OK\r\nAccept-Ranges: bytes\r\nCache-Control: no-cache\r\nContent-Length: 9508\r\nContent-Security-Policy: frame-ancestors......Baidu "</script></body></html>
tcp disconnected.
```

> [点此在 github 中下载上述完整代码。](https://github.com/QuecPython/examples/blob/main/network-comm/net-protocols/tcp-udp/example_tcp_client_without_dns.py)

而一般情况下，用户更容易记住域名`www.baidu.com`。这时需要我们调用域名解析的接口，将域名转换为 IP 地址，而不是直接记住这个 IP 地址。

域名解析的函数原型为`usocket.getaddrinfo(host, port)`，说明如下：

- 参数：

    - `host`：主机域名（IP 地址字符串亦可）。
    - `port`：端口号

- 返回值：`[(family, type, proto, canonname, sockaddr)]`

    - `family`：地址族。
    - `type` - socket 类型。

    - `proto` - 协议类型。

    - `canonname` - 主机域名。

    - `sockaddr` - 包含 IP 地址和端口号的列表。

了解了此接口的用法后，`tcp_client()`函数实现中，直接以`sockaddr`作为`sock.connect()`函数的参数即可。

优化后的`tcp_client()`函数实现如下（[点此在 github 中下载完整代码](https://github.com/QuecPython/examples/blob/main/network-comm/net-protocols/tcp-udp/example_tcp_client_with_dns.py)）：

```python
import usocket
import checkNet

def tcp_client(address, port):
    # Create a socket object
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.IPPROTO_TCP)
    print('socket object created.')
    
    # Domain name resolution
    sockaddr = usocket.getaddrinfo(address, port)[0][-1]
    print('DNS for %s: %s' % (address, sockaddr[0]))
    
    # Connect to the TCP server
    sock.connect(sockaddr)
    print('tcp link established.')
    
    # Package user data
    data = 'GET / HTTP/1.1\r\n'
    data += 'Host: ' + address + ':' + str(port) + '\r\n'
    data += 'Connection: close\r\n'
    data += '\r\n'
    data = data.encode()
    
    # Send the data
    sock.send(data)
    print('<-- send data:')
    print(data)
    
    # Receive the data
    print('--> recv data:')
    while True:
        try:
            data = sock.recv(1024)
            print(data)
        except:
            # Connection ends until the data is fully received
            print('tcp disconnected.')
            sock.close()
            break

if __name__ == '__main__':
    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1: # Network connection is normal
        print('Network connection successful.')
        tcp_client('www.baidu.com', 80) # Start the client
    else:
        print('Network connection failed, stage={}, state={}'.format(stage, state))
```

优化后的客户端代码，可以传递域名或者 IP 地址，更易使用。运行结果如下：

```python
Network connection successful.
socket object created.
DNS for www.baidu.com: 36.152.44.95
tcp link established.
<-- send data:
b'GET / HTTP/1.1\r\nHost: 36.152.44.95:80\r\nConnection: close\r\n\r\n'
--> recv data:
b'HTTP/1.1 200 OK\r\nAccept-Ranges: bytes\r\nCache-Control: no-cache\r\nContent-Length: 9508\r\nContent-Security-Policy: frame-ancestors......Baidu "</script></body></html>
tcp disconnected.
```

#### TCP 服务端网络编程

对于蜂窝通信模组来说，一般情况下基站分配的 IP 地址是局域网地址，外部网络设备无法直接访问模组，这种情况下，模组做 TCP 服务器的意义不是很大。

但是对于专网卡来说，基站为其分配专网地址，并允许专网内的网络设备能够访问模组，此时模组是可以做 TCP 服务器的，这在电表类的产品中是比较常见的。

而且在Wi-Fi和以太网的网络中，是允许局域网设备访问该主机的。

所以我们仍有必要讲解下如何在模组中实现 TCP 服务器功能。

TCP 服务器与客户端的 socket 编程模型示意图的左侧展示了服务器编程的接口调用流程：

1. 调用`socket()`接口创建 socket 对象。

2. 调用`bind()`接口绑定本地的地址和端口。

3. 调用`listen()`接口监听客户端连接请求。

4. 调用`accept()`接口接受客户端连接请求。
5. 调用`recv()`接口接收客户端上行的数据。
6. 调用`send()`接口向客户端发送数据。
7. 每一个客户端连接中，循环执行第 5 步和第 6 步，业务满足一定条件或连接断开，调用`close()`接口关闭 socket，释放资源。
8. 在接受客户端连接请求的线程中，循环执行第 4 步，以接受更多的客户端接入。

TCP 服务器编程调用的接口相比客户端，多了`bind()`、`listen()`、`accept()`三个接口，说明如下：

- `socket.bind(address)`：绑定本地 IP 地址与端口。
    - `address`：包含 IP 地址字符串与端口号的元组或列表。因为服务器一般需要固定的 IP 地址与端口，如果不进行绑定，客户端将无从得知服务器的 IP 地址和端口号。
- `listen(backlog)`：监听客户端连接请求。
    - `backlog`：除了正在处理的连接请求外，服务器允许后面排队等待处理的连接请求的个数。
- `accept()`：接受客户端连接请求。

了解了流程和接口用法，我们这里做一个实验：在模组中分别编写一个 TCP 服务器程序和一个 TCP 客户端程序，客户端周期性向服务器发送数据，而后等待服务器回送数据。

TCP 服务器代码如下：

```python
import usocket
import _thread

def _client_conn_proc(conn, ip_addr, port):
    while True:
        try:
            # Receive data sent by the client
            data = conn.recv(1024)
            print('[server] [client addr: %s, %s] recv data:' % (ip_addr, port), data)
            
            # Send data back to the client
            conn.send(data)
        except:
            # Exception occurred and connection closed
            print('[server] [client addr: %s, %s] disconnected' % (ip_addr, port))
            conn.close()
            break

def tcp_server(address, port):
    # Create a socket object
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.IPPROTO_TCP_SER)
    print('[server] socket object created.')
    
    # Bind the server IP address and port
    sock.bind((address, port))
    print('[server] bind address: %s, %s' % (address, port))
    
    # Listen for client connection requests
    sock.listen(10)
    print('[server] started, listening ...')
    
    while True:
        # Accept a client connection request
        cli_conn, cli_ip_addr, cli_port = sock.accept()
        print('[server] accept a client: %s, %s' % (cli_ip_addr, cli_port))
        
        # Create a new thread for each client connection for concurrent processing
        _thread.start_new_thread(_client_conn_proc, (cli_conn, cli_ip_addr, cli_port))
```

> 该段代码中，服务器每接入一个客户端，则新建一个线程，专门处理该客户端连接的相关事宜，做到并发处理。

TCP 客户端代码如下：

```python
import usocket
import utime

def tcp_client(address, port):
    # Create a socket object
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.IPPROTO_TCP)
    print('[client] socket object created.')
    
    # Connect to the TCP server
    print('[client] connecting: %s, %s' % (address, port))
    sock.connect((address, port))
    print('[client] connected.')
    
    data = b'1234567890'
    while True:
        try:
            # Send data to the server
            sock.send(data)
            print('[client] send data:', data)
            
            # Read the data sent back by the server
            data = sock.recv(1024)
            print('[client] recv data:', data)
            print('[client] -------------------------')
            
            # Delay for 1 second
            utime.sleep(1)
        except:
            # Connection ends until the data is fully received
            print('[client] disconnected.')
            sock.close()
            break
```

主流程代码如下：

```python
import checkNet
import _thread
import utime
import dataCall

if __name__ == '__main__':
    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1: # Network connection is normal
        print('[net] Network connection successful.')
        
        # Get the IP address of the module
        server_addr = dataCall.getInfo(1, 0)[2][2]
        server_port = 80
        
        # Start the server thread to listen for client connection requests
        _thread.start_new_thread(udp_server, (server_addr, server_port))
        
        # Delay for a while to ensure that the server starts successfully
        print('sleep 3s to ensure that the server starts successfully.')
        utime.sleep(3)
        
        # Start the client
        udp_client(server_addr, server_port)
    else:
        print('[net] Network connection failed, stage={}, state={}'.format(stage, state))
```

<a id="dataCall_getInfo"></a>该段代码中调用了`dataCall.getInfo()`接口获取模组的 IP 地址，该接口的返回值格式为`(profileID, ipType, [state, reconnect, addr, priDNS, secDNS])`。说明如下：

- `profileID`：PDP 上下文场景 ID。该术语比较晦涩，简单的理解就是：蜂窝通信模组可以和基站建立多路连接，每一路对应一个不同的 IP 地址。一般情况下，一路连接就足以满足需求。如果想要获取指定某一路的 IP 地址，该参数填入对应的 ID 即可。
- `ipType`：IP 协议类型。取值范围为 0-2，分别表示 IPv4协议、IPv6协议、IPv4与IPv6协议共存。初学者可以仅关心 IPv4 即可。
- `state`：与基站间的网络连接状态。0 表示未连接，1 表示连接。
- `reconnect`：重连标志。保留未用。
- `addr`：IP 地址。
- `priDNS`：主 DNS 服务器地址。
- `secDNS`：辅 DNS 服务器地址。

代码中的`server_addr = dataCall.getInfo(1, 0)[2][2]`表示获取第 1 路连接的 IPv4 协议的 IP 地址，将其作为本地服务器的 IP 地址。

> - 支持了 QuecPython 的蜂窝通信模组在上电后会自动进行第一路的蜂窝数据连接。
> - [点此查看](../../../api_reference/iotlib/dataCall.md)更多与蜂窝数据连接相关的接口用法。
> - [点此在 github 中下载上述完整代码。](https://github.com/QuecPython/examples/blob/main/network-comm/net-protocols/tcp-udp/example_tcp_server_and_client.py)

该实验代码的运行结果如下：

```log
[net] Network connection successful.
sleep 3s to ensure that the server starts successfully.
[server] socket object created.
[server] bind address: 10.104.189.115, 80
[server] started, listening ...
[client] socket object created.
[client] connecting: 10.104.189.115, 80
[client] connected.
[client] send data: b'1234567890'
[server] accept a client: 10.104.189.115, 55187
[server] [client addr: 10.104.189.115, 55187] recv data: b'1234567890'
[client] recv data: b'1234567890'
[client] -------------------------
[client] send data: b'1234567890'
[server] [client addr: 10.104.189.115, 55187] recv data: b'1234567890'
[client] recv data: b'1234567890'
[client] -------------------------
...
```

### UDP网络编程

在开始 UDP 网络编程之前，我们先通过下图，初步了解下 UDP 服务器与客户端的 socket 编程模型：

![](../../media/network-comm/net-protocols/tcp-udp/udp-socket-model.png)

从图中可以看出，UDP 服务器也需要调用`bind()`接口，绑定本地的 IP 地址和端口号，这是作为服务器所必须的接口调用。

同时，UDP 编程在接口调用上也有与 TCP 编程不同之处：

- `socket()`接口参数不同：
    - TCP 编程时，第二个参数`type`为`usocket.SOCK_STREAM`，而 UDP 编程时，第二个参数`type`为`usocket.SOCK_DGRAM`。
    - TCP 编程时，第三个参数`proto`为`usocket.IPPROTO_TCP`或`usocket.IPPROTO_TCP_SER`，而 UDP 编程时，第三个参数`proto`为`usocket.IPPROTO_UDP`。
- 由于 UDP 是无连接的，客户端无需调用`connect()`接口去连接服务器。
- 数据发送方只要直接调用`sendto()`接口将数据发送出去即可。
- 数据接收方调用`recvfrom()`接口接收数据。

>`sendto()`接口是否能真正将数据发送到目的地，视网络环境而定，如果无法找到目标 IP 地址对应的主机，则数据被丢弃。

接下来，我们做一个实验：在模组中分别编写一个 UDP 服务器程序和一个 UDP 客户端程序，客户端周期性向服务器发送数据，而后等待服务器回送数据。

有了前面 TCP 编程的经验，我们直接给出实验代码（[点此在 github 中下载完整代码](https://github.com/QuecPython/examples/blob/main/network-comm/net-protocols/tcp-udp/example_udp_server_and_client.py)）：

```python
import usocket
import _thread
import utime
import checkNet
import dataCall

def udp_server(address, port):
    # Create a socket object
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_DGRAM, usocket.IPPROTO_UDP)
    print('[server] socket object created.')
    
    # Bind server IP address and port
    sock.bind((address, port))
    print('[server] bind address: %s, %s' % (address, port))
    
    while True:
        # Read client data
        data, sockaddr = sock.recvfrom(1024)
        print('[server] [client addr: %s] recv data: %s' % (sockaddr, data))
        
        # Send data back to the client
        sock.sendto(data, sockaddr)

def udp_client(address, port):
    # Create a socket object
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_DGRAM, usocket.IPPROTO_UDP)
    print('[client] socket object created.')
    
    data = b'1234567890'
    while True:
        # Send data to the server
        sock.sendto(data, (address, port))
        print('[client] send data:', data)
        
        # Read data sent back from the server
        data, sockaddr = sock.recvfrom(1024)
        print('[client] [server addr: %s] recv data: %s' % (sockaddr, data))
        print('[client] -------------------------')
        
        # Delay for 1 second
        utime.sleep(1)
        
if __name__ == '__main__':
    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1: # Network connection is normal
        print('[net] Network connection successful.')
        
        # Get the IP address of the module
        server_addr = dataCall.getInfo(1, 0)[2][2]
        server_port = 80
        
        # Start the server thread
        _thread.start_new_thread(udp_server, (server_addr, server_port))
        
        # Delay for a while to ensure that the server starts successfully
        print('sleep 3s to ensure that the server starts successfully.')
        utime.sleep(3)
        
        # Start the client
        udp_client(server_addr, server_port)
    else:
        print('[net] Network connection failed, stage={}, state={}'.format(stage, state))
```

运行结果如下：

```log
[net] Network connection successful.
sleep 3s to ensure that the server starts successfully.
[server] socket object created.
[server] bind address: 10.110.90.159, 80
[client] socket object created.
[client] send data: b'1234567890'
[server] [client addr: ('10.104.189.115', 62104)] recv data: b'1234567890'
[client] [server addr: ('10.104.189.115', 80)] recv data: b'1234567890'
[client] -------------------------
[client] send data: b'1234567890'
[server] [client addr: ('10.104.189.115', 62104)] recv data: b'1234567890'
[client] [server addr: ('10.104.189.115', 80)] recv data: b'1234567890'
[client] -------------------------
...
```

### 多网卡网络编程

多网卡，顾名思义，一台设备中有多个网卡。在进行网络通信时，如何选取指定的网卡呢？有两种方法：

- 将指定网卡设置为默认网卡。指定了默认网卡了，前述 TCP 客户端编程的流程不需要变动。
- 获取指定网卡的 IP 地址，在创建完成 socket 对象后，并且在连接服务器之前调用`socket.bind()`接口。

> - `socket.bind()`仅用于将当前网络连接与指定的 IP 地址及端口号绑定，不区分客户端或服务器。
> - 端口号写 0 表示随机生成端口号，随机端口号在客户端编程中很常用。如果端口号非 0，即固定端口号，需要通过`socket.setsockopt(usocket.SOL_SOCKET, usocket.SO_REUSEADDR, 1)`设置端口重用，否则将会连接失败。

在实际应用场景中，有时会进行多路蜂窝数据连接，有时会 4G 网卡、外接的以太网卡或 Wi-Fi 网卡并存，此两种方法要如何实现呢？

#### 蜂窝数据网络

前文提到，模组上电后会自动进行第 1 路的蜂窝数据连接，并且将该路网卡设置为默认网卡。

如果调用`dataCall`相关接口进行了多路连接，则后面的连接可通过[`dataCall.getInfo()`](#dataCall_getInfo)接口获取模组蜂窝数据连接的 IP 地址。

> 目前暂未提供相关接口，将后来的蜂窝数据连接的网卡设置为默认网卡。

#### 以太网卡

##### 调用`socket.bind()`接口绑定 IP 地址

以太网卡提供`ipconfig()`方法，用以获取包含 IP 地址等信息的网卡属性，返回值格式为：`[(mac, hostname), (iptype, ip, subnet, gateway, primary_dns，secondary_dns)]`。

- `mac`：以太网mac地址。
- `hostname`：网卡名称。
- `iptype`：IP 协议类型。4 表示 IPv4，6 表示 IPv6。
- `ip`：IP 地址。
- `subnet`：子网掩码。
- `gateway`：网关地址。
- `primary_dns`：主 DNS 服务器地址。
- `secondary_dns`：辅 DNS 服务器地址。

获取以太网卡 IP 地址并且绑定的代码示例如下（[点此在 github 中下载完整代码](https://github.com/QuecPython/examples/blob/main/network-comm/net-protocols/tcp-udp/example_socket_bind_of_ethernet.py)）：

```python
import ethernet
import usocket

# Create an Ethernet NIC
eth = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','','','',-1,38,36,37, 0)
print('W5500 ethernet nic created.')

# Enable DHCP
eth.dhcp()
print('DHCP enabled.')

# Enable NIC
eth.set_up()
print('Ethernet nic enabled.')

# After the NIC is registered, check the network configuration information
ip_conf = eth.ipconfig()
print('get ip_conf:', ip_conf)

def tcp_client(address, port):
    # Create a socket object
    sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.IPPROTO_TCP)
    print('socket object created.')
	
    # Bind the Ethernet NIC IP address after creating the socket object and before connecting to the server
    local_address = ip_conf[1][1]
    sock.bind((local_address, 0))
    print('bind ethernet address: %s', local_address)
	
    # Resolve the domain name
    sockaddr=usocket.getaddrinfo(address, port)[0][-1]
    print('DNS for %s: %s' % (address, sockaddr[0]))
    
    # Connect to the TCP server
    sock.connect(sockaddr)
    print('tcp link established.')
    
    # More code in TCP client samples above
```

##### 设置为默认网卡

以太网卡提供`set_default_NIC(ip)`方法，用来将指定 IP 地址的网卡设置为默认网卡。

示例代码如下（[点此在 github 中下载完整代码](https://github.com/QuecPython/examples/blob/main/network-comm/net-protocols/tcp-udp/example_set_default_nic_of_ethernet.py)）：

```python
import ethernet

# Create an Ethernet NIC
eth = ethernet.W5500(b'\x12\x34\x56\x78\x9a\xbc','','','',-1,38,36,37, 0)
print('W5500 ethernet nic created.')

# Enable DHCP
eth.dhcp()
print('DHCP enabled.')

# After the NIC is registered, check the network configuration information
ip_conf = eth.ipconfig()
print('get ip_conf:', ip_conf)

# Set the Ethernet NIC as the default NIC
eth.set_default_NIC(ip_conf[1][1])
print('W5500 is set as default nic.')

# Enable the NIC
eth.set_up()
print('Ethernet nic enabled.')
```

> - [点此查看](../../../api_reference/peripherals/ethernet.md)以太网相关的接口用法。
> - 关于以太网卡更多的编程应用指导，请查看相关文档。


## 常见问题

**1. 为什么连接服务器会失败？**

- 服务器必须是公网地址（连接模组本地的 server 除外）。
- 使用 PC上 的 TCP/UDP 测试工具客户端、或者 mqtt.fx，连接服务器确认一下是否可以连接成功，排除服务器故障。
- 2G网络不要使用中国联通卡。
- 检查下模块信号、网络注册、网络附着、PDP激活状态。
- 检查下SIM卡是否欠费【4G模块有一种欠费表现：无法注册4G网络，可以注册2G网络】。

**2. TCP 有自动重连功能吗？**

底层没有自动重连，重连机制在应用层处理。

**3. DNS 解析失败怎么排查原因？**

检查 SIM 卡是否注网成功以及检查该地址的有效性再次进行尝试。

**4. 为什么 4G 模块专网卡连接服务器失败？**

- 检查 APN 参数是否设置正确。
- 如果有其他厂家的模块，对比测试下是否连接正常。
- 如果无法百分百保证服务器配置没问题，最好在服务器端用 wireshark 抓包，或者在服务器上安装一个第三方工具，开启一个服务器端口来对比测试。
- 用定向 IP 的物联网卡，需要把域名或 IP 加入白名单才能使用。

**5. 各系列模组最多可同时创建多少路 socket？**

- EC200A 系列模组最多可同时创建 64 路 socket。
- ECxxxG/ECxxxU 系列模组最多可同时创建 15 路 socket。
- BG95 系列模组最多可同时创建 20 路 socket。

**6. 4G 模块可以同时作为服务器和客户端使用吗？**

- 对于 IPv4 协议来说，一般不能作为服务器使用， 模块能获取的是运营商分配的内网 IP；对于专网卡，运营商分配的 IP 地址可以在专网内进行通信，因此可以作为服务器使用。
- 对于 IPv6 协议来说，其全局链路地址就是公网 IP 地址，因为可以作为服务器使用。
- 模组内的客户端可以连接模组内的服务器，某些应用架构有需要，可以这么做。

**7. 为什么我一包数据只有不到 50B，一天消耗的流量要远远大于实际传输值**

如果使用的是 TCP 协议，需要三次握手四次挥手才算完成了一次数据交互，原始数据不多但是由于 TCP 协议决定的一包数据必须要加包头包尾帧校验等，所以实际消耗的流量不止 50B，部分运营商有限制每一包数据必须 1KB 起发，不足 1KB 也会加各种校验凑足 1KB。

**8. 如果 KeepAlive 设置的时间长，会不会被基站断开？**

会，一般建议使用 2 分钟，不建议超过 4 分钟，基站策略会关闭长时间没有数据传输的连接，太长时间可能会导致连接被基站关闭。