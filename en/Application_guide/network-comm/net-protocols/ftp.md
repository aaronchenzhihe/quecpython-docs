# FTP Application Note

This document provides a detailed introduction on how to use the FTP protocol for file transfer in QuecPython. It includes an overview of the protocol, environment setup, code examples, and result demonstrations.

## Overview

FTP (File Transfer Protocol) is a standard protocol used for transferring files over computer networks. It is based on the client-server model and achieve file uploads, downloads, and management by control connections and data connections.

In FTP communication, the client is responsible for initiating the connection and sending FTP commands, while the server is in charge of receiving these commands and performing the corresponding actions. FTP  transmit commands and responses through control connection,and performs actual file transfer with the data connection established by the control connection.

FTP operations include:

- Login Authentication: The client sends a username and password to the server for identity verification in order to gain access permissions.
- Directory Operations: The client can request to list files and directories on the server, switch the current working directory, create and delete directories etc.
- File Transfer: The client can upload files to the server or download files from the server to the local system.
- File Management: The client can perform operations on the server's files such as renaming, deleting, copying, etc.
- Passive and Active Modes: Data transfer between the client and server can be done in either active or passive mode, depending on the network environment and configuration.

## Function

The functions of FTP include, but are not limited to, the following:

- **File Transfer**: FTP allows users to transfer files between different computers. Users can download files from one computer (server) to their local computer (client), or they can upload files from their local machine to the server. This makes file sharing and transfer very convenient.
- **File Management**: FTP provides a series of commands and operations that enable users to manage files on a remote server. Users can list files and subdirectories in a directory, create new directories, delete directories and files, rename files, etc. Through these operations, users can organize, back up, and maintain remote files.

## Working Mode

FTP has two common working modes:

### Active Mode

In active mode, the client informs the server via the control connection about which port it will receive the data connection on and waits for the server's connection request. The server actively connects to the port specified by the client  to transfer file data for data connection.

![image-20230701162802597](../../media/network-comm/net-protocols/ftp/ftp_1.png)

#### Working Process of Active Mode

1. The client initiates a connection to the server's Port 21 from a random non-privileged Port N, which is a port greater than 1024.
2. The client starts listening on Port N+1.
3. The server actively connects from its Port 20 to the client's Port N+1.

#### Advantages of Active Mode

The server configuration is simple, which is conducive to server security management, and the server only needs to open Port 21.

#### Disadvantages of Active Mode

If the client has enabled a firewall, or if the client is in the intranet (behind a NAT gateway), then the server's attempt to connect to the client's port may fail.

### Passive Mode

In passive mode, the server informs the client via the control connection about which port it will receive the data connection on. The client actively connects to the port specified by the server to transfer file data for data connection.

![image-20230701162905210](../../media/network-comm/net-protocols/ftp/ftp_2.png)

#### Working Process of Passive Mode

1. The client connects to the server's Port 21 from a random non-privileged port.
2. The server allocates a non-privileged port as a passive port and sends it back to the client.
3. The client actively connects to the server's passive port from its non-privileged port + 1.

#### Advantages of Passive Mode

The server configuration management is slightly complicated, which is not conducive to security management. The server needs to open random high bit ports so that the client can connect. Therefore, most FTP service software allows manual configuration of the passive port range.

#### Disadvantages of Passive Mode

There are no specific network environment requirements for the client.

## Communication Model

![image-20230701163157920](../../media/network-comm/net-protocols/ftp/ftp_3.png)

The FTP (File Transfer Protocol) communication model is based on a client-server architecture, where the client and server communicate and transfer files over FTP. In the FTP communication model, the client initiates connection requests and sends various FTP commands, while the server receives commands and executes corresponding operations.

The communication model comprises two essential elements: the control connection and the data connection. The control connection is used to transmit commands and server responses. The client establishes an initial connection with the server through the control connection and sends FTP commands, while the server responds through the control connection. The data connection is used for actual file transfer. When file transfer is needed, a data connection is established between the client and server to transfer file content. This model enables reliable file transfer and management, allowing users to share and transfer files between different computers, as well as perform operations and maintenance on files located on remote servers.

- **Control Connection**: The control connection is a long-term connection between the client and the server used for transmitting commands, status information and control signals. The control connection uses Port 21 by default.
- **Data Connection**: The data connection is the connection used for actual file data transfer between the client and the server. The establishment of the data connection depends on the working mode (active or passive mode).

## FTP Application

QuecPython provides the `ftplib` module for establishing FTP client connections. This section is divided into two parts: one for the FTP server and the other for the FTP client.

This chapter demonstrates with a public FTP server and the QuecPython series module as an FTP client. In this demonstration, the public server is used by default and the module access the public server through cellular network (because the module can only access public servers if cellular network is used). Below is a flow chart for creating an FTP client connection with QuecPython:

![image-20231111152826884](../../media/network-comm/net-protocols/ftp/image-20231111152826884.png)

The above is the flow chart outlines the process of FTP client connecting to a server, as depicted below:

1. Instantiate the FTP class function, which returns an operable handle (referred to as an object in Python). This object includes all FTP API methods, such as initiating connections, logging into the server, uploading and downloading files etc.

2. Input the address and port information of the FTP server. Execute the `ftp.connect` method to initiate a connection request to the server.

3. Once connected to the server, log in to the FTP service with  `ftp.login` . This step requires the username and password.

4. After the connection and login are completed, upload a local file to the server. Uploading a local file needs to open and read the file content from the device file system first. The content is then transmitted to the server with `ftp.storlines` in conjunction with the "STOP" command.

5. To download a specific file from the server to the local client, the `ftp.retrlines` method, along with the "RETR" command, can be used on the local client. After requesting the download, receive the data and save it to the local file system.

### Communication Process of FTP Client

1. Establish Control Connection: The client initiates a TCP connection to establish a control connection with the FTP server. The client uses the IP address and FTP port number (usually Port 21) of the server to establish this connection.
2. Initiate Login Request: The client sends user credentials (username and password) as a login request. These credentials are used for authentication to gain access permissions to the server.

3. Receive Server Response: Upon receiving the login request, the server performs authentication and sends a corresponding response message to the client. The response message includes a response code indicating whether the login was successful or not.

4. Send FTP Commands: The client sends FTP commands to the server through the control connection to perform various operations. These commands include listing directory contents, changing the working directory, uploading files, downloading files, deleting files, and so on.

5. Receive Server Response: After receiving the FTP commands, the server executes the corresponding operations and sends response messages back to the client through the control connection. The response message includes a response code indicating the results of the command execution.

6. Establish Data Connection: When file transfer is needed, a data connection is established between the client and the server. The data connection can be in either active or passive mode, depending on the server's configuration.

7. Perform File Transfer: On the established data connection, the client can upload or download files. The client sends file to the server or requests file content, and the server transfers the data through the data connection.

8. Close Connection: Once the file transfer is complete or communication with the server is no longer required, the client can send a QUIT command to close the control connection and release related resources.

### FTP Client

To build an FTP client with `ftplib` on the module and establish a connection with the server for file upload and download, please make sure that FTP feature are used in QuecPython. Besides, you need to ensure that your device meet the following requirements:

1. Download QuecPython firmware: Download the appropriate QuecPython firmware version according to the module model.
2. Connect to network: Ensure your device can connect to the network.
3. QuecPython firmware includes ftplib module.

After downloading the firmware, you need to check whether the current firmware includes the `ftplib` module and check the network status. You can use [QPYcom](../../dev-tools/QPYcom/README.md) for debugging. The demonstrations in this document are performed with this tool. On the interactive page of QPYcom, you can confirm whether the module contains and uses APIs to check the network status by importing the feature. In Python syntax, you can import APIs with `import xxx` or `from xx import xxx`.

```python
# Inclueded if no exception occurs
from ftplib import FTP
```

You can import the `checkNet` API to query the network status of the device. The status values are available in the [wiki](../../../api_reference/iotlib/checkNet.md#checkNet.waitNetworkReady).

```python
import checkNet
stage, state = checkNet.waitNetworkReady(30)
print(stage, state) # 3 1
```

Once you have confirmed that the device's network is functioning properly, you can create an FTP object with the `ftplib` API. See as follows:

```python
# Import FTP class functions in ftplib module
from ftplib import FTP

# Create an FTP object
ftp_obj = FTP()
```

By following these steps, you can successfully create an FTP client object. Now you can utilize this object to perform various operations.

After creating the FTP client object, the next step is to actively request to connect with the server. You can achieve this with the `connect` method. This method requires two parameters: the server address and the port number.

```python
# FTP server address
host = ""
# Default port is Port 21
port = 21

# Initiate connection request with the FTP object
ftp_obj.connect(host, port)
```

Once the connection is successfully established, the next step is to perform the login operation. If the FTP server does not require specific credentials or if it is set to be empty, you can simply pass an empty string. This can be achieved with the `login` method.

```python
# FTP server username
user = "xx"
# FTP server password
pwd = "xx"

ftp_obj.login(user, pwd)
```

The running result is shown in the figure:

![image-20230711142733722](../../media/network-comm/net-protocols/ftp/image-20230711142733722.png)

Once the connection and login are completed, you can proceed with file transfer. When uploading a file, please note that the file you upload must actually exist. You should open the file following its file path. Here is an example of uploading a file.

```python
path = 'usr/' # The root directory of the module user partition is 'usr'
filename = 'ftp_file.txt'

# Determine whether the file exists
if filename in uos.listdir("usr/"):
    print("File exists")
    with open(path + filename, "rb") as fp:
    # cmd should be the STOR command. fp is a file object (opened in binary), each line of which will be read with its readline() method to provide the data to be stored.
    res = ftp_obj.storlines("STOR " + filename, fp)
    msg = "Upload %s to FTP Server %s."
    if res.startswith("226 Transfer complete"):
        print(msg % (filename, "success"))
        return True
    else:
        print(msg % (filename, "falied"))
        return False
else:
    print("File path does not exist")
```

Running result:

![image-20230711143921595](../../media/network-comm/net-protocols/ftp/image-20230711143921595.png)

View the uploaded file in the server:

![image-20230711143650551](../../media/network-comm/net-protocols/ftp/image-20230711143650551.png)

Here takes the example of downloading the "ftp_file.txt" file that was just uploaded to the server. When the client requests to download a file from the server, the FTP object finishes the downloading with the `retrlines` method. First, create an empty file locally and obtain a file handle with the `open` method. Once the client establishes the download connection with the server, the file data will be written into this empty file.

```python
path = 'usr/' #The root directory of the module user partition is 'usr'
filename = 'ftp_file.txt' # File name to be downloaded

# Open and create a file
save_fp = open(path + filename, "wb+")
# Query the current path of the server
server_path = ftp_obj.pwd()
# Establish the download connection. fp.write is an operation of writing files
res = ftp_obj.retrlines("RETR " + server_path + "/" + filename, fp.write)
# Finish downloading
msg = "Down %s to device %s."
if res.startswith('226 Transfer complete'):
    print(msg % (filename, "success"))
    return True
else:
    print(msg % (filename, "falied"))
    return False
```

Running Result：

![image-20230711145324194](../../media/network-comm/net-protocols/ftp/image-20230711145324194.png)



In this demonstration, the FTP client built with QuecPython is used to complete the interaction with the server and file transfer, and finally a complete sample code is provided for reference.

Code example:

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
        # Determine whether the file exists
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
            # Store files in text line mode. cmd should be the appropriate STOR command. fp is a file object (opened in binary), each line of which will be read with its readline() method to provide the data to be stored.
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

Before enabling the code on the module, you need to check the network status first. After confirming that the network is normal, initialize the `FtpManager` class function to start the client.

```python
import checkNet

host = ""
port = 21
user = "test"
passwd = "test"

# Create an FTP connection object
ftp_cli_obj = FtpManager()

stage, state = checkNet.waitNetworkReady(30)
if stage == 3 and state == 1: #Network status is normal
   # Connect the FTP server
	connect_info = ftp_cli_obj.connect(host, port)
	print(connect_info)
    # Log in to the FTP server
    login_info = ftp_cli_obj.login(user, passwd)
    print(login_info)
    # Upload the file and input the local file path
    upload_sta = ftp_cli_obj.upload_file("usr/", "ftp_file.txt")
    print(upload_sta)
else:
    print('Network connection failed, stage={}, state={}'.format(stage, state))
```

### FTP Client Commands

The commonly used FTP client commands are as follows.

- CONNECT: Establishes connection with the FTP server
- USER: Specifies the login username
- PASS: Specifies the login password
- LIST: Lists the files and subdirectories of the current directory on the server.
- CWD: Changes the current working directory
- PWD: Displays the path of the current working directory
- RETR: Downloads files from the server to local computer
- STOR: Uploads local files to the server
- DELE: Deletes files on the server
- MKD: Creates a directory on the server
- RMD: Deletes directories on the server
- RNFR: Specifies the file or directory to be renamed
- RNTO: Specifies the renamed file or directory
- PASV: Enters passive mode for data transfer
- TYPE: Specifies the type of data  transfer such as ASCII or binary
- SIZE: Gets the size of the files on the server
- SYST: Get the OS type of the server.
- NOOP: A no-op used to keep the control connection alive.
- QUIT: Disconnects from the server

These commands communicate and perform different operations between the FTP client and server, including login authentication, directory operations, file transfer, renaming, deletion, etc. The specific command syntax and usage may vary depending on the implementation of the FTP client.

### FTP Server

[Click to see](https://www.geeksforgeeks.org/how-to-setup-and-configure-an-ftp-server-in-linux-2/#) the reference document for setting up an FTP server in a Linux environment. If you do not have a public FTP service, please refer to this document or other resources to complete the setup.

An FTP server provides a reliable and standardized way for file transfer, widely used in scenarios such as file sharing, website maintenance, and backups. Communication between the server and the client involves the following key points:

1. Connection Establishment: The client establishes communication with the server via a TCP connection. The client sends a connection request to the server and provides necessary authentication credentials if required.
2. Command Transmission: The client sends FTP commands to the server through the control connection. These commands include tasks like geting file lists, uploading files, downloading files, and creating directories.
3. Data Transfer: Depending on the command type and transfer mode, the server establishes a data connection with the client for actual file transfers. Data connections can be either active mode (initiated by the server) or passive mode (initiated by the client).
4. Response and Status Codes: After receiving a command from the client, the server executes the corresponding operation and sends response messages and status codes to indicate the command's execution result or error information.

Configuration of the server (with public IP by default):
Inbound Port Mapping in Security Group: TCP/Port 21

```python
# FTP server address
host = ""
# Port number
port = 21
# Username
user = ""
# Password
passwd = ""
```

## FAQs

**Q: Failed to import the ftplib module**

A: First, make sure that you have correctly flashed the QuecPython firmware and that the firmware includes the ftplib module. If it's not included, you might need to change the firmware.

**Q: Failed to connect the device to the FTP server.**

A: Make sure that the device can successfully establish a network connection. After confirming network connectivity, check if the FTP connection parameters are correct.

**Q: What to do if there is not enough space for downloading a file?**

A: Before downloading a specific file, make sure that there is enough space available in the default path. Otherwise, the download might fail.

**Q: Error code "550" is returned while downloading a file.**

A: This error code indicates that the specified file name was not found on the server. Please check if the file exists on the server.