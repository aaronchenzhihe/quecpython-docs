# FTP 协议应用指导

本文档详细介绍了如何在 QuecPython 中使用 FTP 协议进行文件传输，并提供了协议概述、环境搭建、代码示例和结果演示。

## 协议概述

FTP（File Transfer Protocol）是一种用于在计算机网络中传输文件的标准协议，它基于客户端-服务器模型，通过控制连接和数据连接实现文件的上传、下载和管理操作。

在FTP通信中，客户端负责发起连接并发送FTP命令，而服务器则负责接收命令并执行相应的操作，FTP使用控制连接来传输命令和响应，通过控制连接建立的数据连接用于实际的文件传输。

FTP的操作包括：

- 登录认证：客户端向服务器发送用户名和密码进行身份认证，以获取访问权限。
- 目录操作：客户端可以请求列出服务器上的文件和目录列表、切换当前工作目录、创建和删除目录等。
- 文件传输：客户端可以上传文件到服务器或从服务器下载文件到本地系统。
- 文件管理：客户端可以对服务器上的文件进行重命名、删除、复制等操作。
- 被动和主动模式：客户端和服务器之间的数据传输可以采用主动模式或被动模式，具体取决于网络环境和配置。

## FTP 的作用

FTP的作用包括但不限于以下方面：

- 文件传输：FTP允许用户在不同的计算机之间传输文件，用户可以从一个计算机（服务器）下载文件到本地计算机（客户端），也可以将文件从本地上传到服务器，这使得文件共享和传输变得非常方便。

- 文件管理：FTP提供了一系列的命令和操作，使用户能够对远程服务器上的文件进行管理。用户可以列出目录中的文件和子目录，创建新目录，删除目录和文件，重命名文件等。通过这些操作，用户可以对远程文件进行组织、备份和维护。

## 工作模式

FTP 有两种常见的工作模式：

### 主动模式（Active Mode）

在主动模式下，客户端通过控制连接告知服务器它将在哪个端口接收数据连接，并等待服务器的连接请求。服务器在数据连接时主动连接到客户端指定的端口，进行文件数据的传输。

![image-20230701162802597](../../media/network-comm/net-protocols/ftp/ftp_1.png)

#### 主动模式工作过程

1. 客户端以随机非特权端口N，就是大于1024的端口，对server端21端口发起连接。
2. 客户端开始监听 N+1端口。
3. 服务端会主动以20端口连接到客户端的N+1端口。

#### 主动模式的优点

服务端配置简单，利于服务器安全管理，服务器只需要开放21端口。

#### 主动模式的缺点

如果客户端开启了防火墙，或客户端处于内网（NAT网关之后）， 那么服务器对客户端端口发起的连接可能会失败。

### 被动模式（Passive Mode）

在被动模式下，服务器通过控制连接告知客户端它将在哪个端口接受数据连接，客户端在数据连接时主动连接到服务器指定的端口，进行文件数据的传输。

![image-20230701162905210](../../media/network-comm/net-protocols/ftp/ftp_2.png)

#### 被动模式工作过程

1. 客户端以随机非特权端口连接服务端的21端口。
2. 服务端开启一个非特权端口为被动端口，并返回给客户端。
3. 客户端以非特权端口+1的端口主动连接服务端的被动端口。

#### 被动模式缺点

服务器配置管理稍显复杂，不利于安全，服务器需要开放随机高位端口以便客户端可以连接，因此大多数FTP服务软件都可以手动配置被动端口的范围。

#### 被动模式的优点

对客户端网络环境没有要求。

## 通信模型

![image-20230701163157920](../../media/network-comm/net-protocols/ftp/ftp_3.png)

FTP（File Transfer Protocol）通信模型基于客户端-服务器架构，其中客户端和服务器之间通过FTP协议进行通信和文件传输。在FTP通信模型中，客户端负责发起连接请求并发送各种FTP命令，而服务器则负责接收命令并执行相应的操作。

通信模型包括控制连接和数据连接两个关键要素，控制连接用于传输命令和服务器响应，客户端通过控制连接与服务器建立初始连接并发送FTP命令，服务器通过控制连接返回响应。数据连接用于实际的文件传输，当需要传输文件时，客户端和服务器之间会建立一个数据连接，用于在两者之间传输文件内容。通过这种模型，FTP实现了可靠的文件传输和管理，允许用户在不同计算机之间共享和传输文件，并对远程服务器上的文件进行操作和维护。

- **控制连接（Control Connection）**：控制连接是客户端和服务器之间的长期连接，用于传输命令、状态信息和控制信号。控制连接默认使用端口 21。

- **数据连接（Data Connection）**：数据连接是客户端和服务器之间进行实际文件数据传输的连接，数据连接的建立方式取决于工作模式（主动或被动模式）。

## FTP应用

QuecPython 提供了`ftplib`模块，用于FTP客户端连接使用，本节分为FTP服务端和客户端两个章节进行说明。

本章节将通过一个公网FTP服务端和使用QuecPython实现一个FTP客户端来进行演示，由于模组使用蜂窝数据网络只能访问公网服务器，本次演示默认有公网服务器，可以使用蜂窝数据网络直接访问，我们先通过下图初步了解下基于QuecPython完成FTP客户端的应用流程：

![image-20230713142751410](../../media/network-comm/net-protocols/ftp/image-20230713142751410.png)

上图是FTP客户端连接到服务端的应用流程，如下描述：

1. 实例化FTP类函数，返回一个可操作的句柄，Python里面称之为对象，该对象拥有FTP所有的API方法，例如请求连接，登录服务端，上传下载文件等。
2. 输入FTP服务端地址和端口信息，执行`ftp.connect`方法向服务端发起连接请求。
3. 与服务端连接建立成功后需要登录FTP服务,使用`ftp.login`方法完成登录请求，这里需要用到用户名以及密码。
4. 连接登录完成后，上传本地文件到服务端需要先从设备文件系统中打开并读取文件内容，然后通过`ftp.storlines`方法配合"STOP"命令完成上传请求。
5. 本地客户端可以通过`ftp.retrlines`方法配合"RETR"命令从服务端下载指定文件到本地，请求下载后接收数据并保存至文件系统中。

### FTP客户端通信流程

1. 建立控制连接： 客户端通过建立TCP连接与FTP服务器建立控制连接。客户端使用服务器的IP地址和FTP端口号（通常是端口号21）来建立连接。
2. 发起登录请求： 客户端发送用户凭证（用户名和密码）作为登录请求。这些凭证用于身份验证，以获取对服务器的访问权限。
3. 接收服务器响应： 服务器接收到登录请求后，会进行身份验证，并发送相应的响应消息给客户端。响应消息包含一个响应码，指示登录是否成功或失败。
4. 发送FTP命令： 客户端通过控制连接发送FTP命令给服务器，以执行各种操作。这些命令可以包括列出目录内容、改变工作目录、上传文件、下载文件、删除文件等。
5. 接收服务器响应： 服务器接收到FTP命令后，执行相应的操作，并通过控制连接返回响应消息给客户端。响应消息包含一个响应码，指示命令执行的结果。
6. 建立数据连接： 当需要进行文件传输时，客户端和服务器之间会建立一个数据连接。数据连接可以采用主动模式或被动模式，具体取决于服务器的配置。
7. 进行文件传输： 在建立好的数据连接上，客户端可以通过数据连接上传或下载文件。客户端向服务器发送文件内容或请求获取文件内容，服务器通过数据连接进行传输。
8. 关闭连接： 当文件传输完成或不再需要与服务器通信时，客户端可以发送QUIT命令来关闭控制连接，并释放相关资源。

### FTP客户端

模组端使用`ftplib`模块搭建FTP客户端，与服务端进行连接并完成文件上传和下载，需要注意的是要在 QuecPython 中使用 FTP 功能，您需要确保您的设备满足以下要求：

1. 烧录 QuecPython固件：请根据您的模组型号找到对应的 QuecPython固件版本进行烧录。
2. 连接到网络：确保您的设备能连接到网络。
3. QuecPython固件中包含ftplib模块。

完成固件烧录后需要检测当前固件是否包含`ftplib`模块以及找网状态，可使用[Qpycom工具](../../dev-tools/QPYcom/README.md)进行调试，本文演示均使用该工具。在工具交互页面通过导入模块的方式来确认是否包含和使用API检测网络情况，python语法通过`import xxx`或`from xx import xxx`的方式导入API。

```python
# 未抛出异常则包含
from ftplib import FTP
```

通过导入API`checkNet`来进行查询设备网络情况，状态值请查看[wiki](../../../api_reference/iotlib/checkNet.md#checkNet.waitNetworkReady)。

```python
import checkNet
stage, state = checkNet.waitNetworkReady(30)
print(stage, state) # 3 1
```

我们在确认设备网络正常的情况下使用`ftplib`API创建ftp对象，如下：

```python
# 导入ftplib模块下的FTP类函数
from ftplib import FTP

# 创建ftp对象
ftp_obj = FTP()
```

这样我们就成功创建了一个ftp客户端对象，接下来我们可以使用该对象进行后续操作。

创建ftp客户端对象后需要主动请求连接服务端，`connect`方法可以帮助我们完成这一步骤，使用`connect`方法需要server地址和端口两个参数:

```python
# FTP服务端地址
host = ""
# 默认端口为21
port = 21

# 使用ftp对象发起连接请求
ftp_obj.connect(host, port)
```

连接成功后进行登录操作，如果ftp服务端未设置或者为空时传入一个空字符串即可，使用`login`方法完成登录请求：

```python
# FTP服务端用户名
user = "xx"
# FTP服务端密码
pwd = "xx"

ftp_obj.login(user, pwd)
```

运行结果如图：

![image-20230711142733722](../../media/network-comm/net-protocols/ftp/image-20230711142733722.png)

连接与登录完成后我们就可以进行文件传输，在上传文件时需要注意待上传的文件必须是真实存在，需要根据文件路径来打开该文件，下述代码示例为文件上传：

```python
path = 'usr/' # 模组用户分区的根目录为'usr'
filename = 'ftp_file.txt'

# 判断文件是否存在
if filename in uos.listdir("usr/"):
    print("文件存在")
    with open(path + filename, "rb") as fp:
    # cmd应为STOR 命令。fp是一个文件对象 (以二进制模式打开)，将使用它的 readline()方法读取它的每一行，用于提供要存的数据。
    res = ftp_obj.storlines("STOR " + filename, fp)
    msg = "Upload %s to FTP Server %s."
    if res.startswith("226 Transfer complete"):
        print(msg % (filename, "success"))
        return True
    else:
        print(msg % (filename, "falied"))
        return False
else:
    print("文件路径不存在")
```

运行结果：

![image-20230711143921595](../../media/network-comm/net-protocols/ftp/image-20230711143921595.png)

服务端查看上传文件：

![image-20230711143650551](../../media/network-comm/net-protocols/ftp/image-20230711143650551.png)

客户端请求下载服务端文件，我们以刚刚上传至服务端的"ftp_file.txt"文件为例来进行下载演示，ftp对象通过`retrlines`方法完成下载，首先在本地创建一个空文件并用open方法返回文件句柄，客户端与服务端下载链接建立连接后会将文件数据写入该文件中，如下：

```python
path = 'usr/' # 模组用户分区的根目录为'usr'
filename = 'ftp_file.txt' # 待下载的文件名

# 打开并创建一个文件
save_fp = open(path + filename, "wb+")
# 查询服务端当前路径
server_path = ftp_obj.pwd()
# 建立下载连接，fp.write为写文件操作
res = ftp_obj.retrlines("RETR " + server_path + "/" + filename, fp.write)
# 完成下载
msg = "Down %s to device %s."
if res.startswith('226 Transfer complete'):
    print(msg % (filename, "success"))
    return True
else:
    print(msg % (filename, "falied"))
    return False
```

运行截图：

![image-20230711145324194](../../media/network-comm/net-protocols/ftp/image-20230711145324194.png)



本次演示我们通过QuecPython搭建的FTP客户端完成与服务端的交互和文件传输，最后给出完整的示例代码供参考。

示例代码如下：

```python
import uos
from ftplib import FTP


class FtpManager(object):
    """
    FTP Client
    """
    def __init__(self):
        self.ftp_obj = FTP()

    def connect(self, host, port):
        """
        ftp connect
        """
        connect_sta = self.ftp_obj.connect(host, port)
        return connect_sta

    def login(self, username, password):
        """
        login ftp server
        """
        login_sta = self.ftp_obj.login(username, password)
        return login_sta

    def check_file(self, filename):
        """
        check file is usr
        """
        # 判断文件是否存在
        if filename in uos.listdir("usr/"):
            return True
        return False

     def upload_file(self, path, filename):
        """
        file_path
        upload file to ftp server
        """
        if not self.check_file(filename):
            print("please check file")
            return False
        with open(path + filename, "rb") as fp:
            # 以文本行模式存储文件。cmd应为恰当的 STOR 命令。fp是一个文件对象 (以二进制模式打开)，将使用它的 readline()方法读取它的每一行，用于提供要存的数据。
            res = self.ftp_obj.storlines("STOR " + filename, fp)
            msg = "Upload %s to FTP Server %s."
            if res.startswith("226 Transfer complete"):
                print(msg % (filename, "success"))
                return True
            else:
                print(msg % (filename, "falied"))
                return False

    def download_file(self, path, filename):
        """
        file_path
        download file
        """
        with open(path + filename, "wb+") as fp:
            server_path = self.ftp_obj.pwd()
            res = self.ftp_obj.retrlines("RETR " + server_path + "/" + filename, fp.write)
            msg = "Down %s to device %s."
            if res.startswith('226 Transfer complete'):
                print(msg % (filename, "success"))
                return True
            else:
                print(msg % (filename, "falied"))
                return False
```

在启动模组端代码之前，需先检查网络状态，在确认网络正常的情况下，初始化`FtpManager`类函数启动客户端。

```python
import checkNet

host = ""
port = 21
user = "test"
passwd = "test"

# 创建FTP连接对象
ftp_cli_obj = FtpManager()

stage, state = checkNet.waitNetworkReady(30)
if stage == 3 and state == 1: # 网络状态正常
   # 连接FTP服务器
	connect_info = ftp_cli_obj.connect(host, port)
	print(connect_info)
    # 登录FTP服务
    login_info = ftp_cli_obj.login(user, passwd)
    print(login_info)
    # 上传文件，输入本地文件路径
    upload_sta = ftp_cli_obj.upload_file("usr/", "ftp_file.txt")
    print(upload_sta)
else:
    print('Network connection failed, stage={}, state={}'.format(stage, state))
```

### FTP客户端命令

以下是常用的FTP客户端命令：

- CONNECT：建立与FTP服务器的连接。
- USER：指定登录用户名。
- PASS：指定登录密码。
- LIST：列出服务器上当前目录的文件和子目录。
- CWD：改变当前工作目录。
- PWD：显示当前工作目录的路径。
- RETR：从服务器下载文件到本地计算机。
- STOR：将本地文件上传到服务器。
- DELE：删除服务器上的文件。
- MKD：在服务器上创建新的目录。
- RMD：删除服务器上的目录。
- RNFR：指定要重命名的文件或目录。
- RNTO：指定重命名后的文件或目录名。
- PASV：进入被动模式，用于数据传输。
- TYPE：指定数据传输的类型，如ASCII或二进制。
- SIZE：获取服务器上文件的大小信息。
- SYST：获取服务器的操作系统类型。
- NOOP：空操作，用于保持控制连接的活动状态。
- QUIT：断开与服务器的连接。

这些命令用于在FTP客户端与服务器之间进行通信和执行不同的操作，包括登录认证、目录操作、文件传输、重命名、删除等。具体的命令语法和使用方式可能因FTP客户端的实现而有所不同。

### FTP服务端

基于Linux环境搭建FTP服务参考文档，[点击查看文档](https://www.geeksforgeeks.org/how-to-setup-and-configure-an-ftp-server-in-linux-2/#)，如果没有公网FTP服务，请参考该文档或其他资料完成搭建。

FTP服务端提供了一种可靠和标准的方式来进行文件传输，广泛应用于文件共享、网站维护、备份等场景，与客户端的通信主要为以下几点：

1. 连接建立：客户端与服务端通过TCP连接建立通信。客户端向服务端发送连接请求，并提供必要的身份验证凭据（如果需要）。
2. 命令传输：客户端通过控制连接向服务端发送FTP命令，例如获取文件列表、上传文件、下载文件、创建目录等。
3. 数据传输：根据命令的类型和传输模式，服务端与客户端建立数据连接并进行文件的实际传输。数据连接可以是主动模式（由服务端发起）或被动模式（由客户端发起）。
4. 响应和状态码：服务端接收到客户端的命令后，执行相应的操作，并向客户端发送响应消息和状态码，以指示命令的执行结果或错误信息。

服务器端配置（以下配置默认为有公网ip的服务器）：
安全组入站端口映射：TCP/21端口

```python
# FTP服务器地址
host = ""
# 端口
port = 21
# 用户名
user = ""
# 密码
passwd = ""
```

## 常见问题

**Q: 导入ftplib模块失败**

A：首先确保正确烧录了QuecPython固件，且该固件包含ftplib模块，如不包含则需更换固件。

**Q:设备连接FTP服务器失败。**

A：请检查是设备能够正常找网，在确保找网成功后检查FTP连接参数是否正确。

**Q：下载文件时空间不够怎么办？**

A：下载指定文件前需确认默认空间足够，不然会导致下载失败。

**Q：下载文件提示文件550异常码**

A：该异常码为在服务器上未找到对应的文件名，请检查服务端是否存在该文件。