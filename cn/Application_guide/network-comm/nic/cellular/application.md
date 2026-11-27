# 场景使用说明

本文将根据用户的不同场景需求，来详细说明如何配置，以及在对应场景下，如何使用网卡进行网络通信，并给出示例代码供用户参考。

用户可参考如下场景来选择查看对应场景的配置：

| 应用场景                                                     | 参考章节                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| SIM卡无需配置APN就可以上网，用户只需要判断模组联网成功后，直接使用socket/mqtt/http等直接访问公共网络。 | [开机自动激活一路网卡且没有配置APN](./application.md#开机自动激活一路网卡且没有配置APN) |
| SIM卡需要配置APN才可以上网，用户只需要判断模组联网成功后，直接使用socket/mqtt/http等直接访问公共网络或者某个专用网络。 | [开机自动激活一路网卡且配置APN](./application.md#开机自动激活一路网卡且配置APN) |
| SIM卡需要配置APN才可以上网，用户业务上需要模组开机后自动激活多路网卡，比如一路用来访问公共网络，另一路用来访问某个专用网络。 | [开机自动激活多路网卡且配置APN](./application.md#开机自动激活多路网卡且配置APN) |
| SIM卡需要配置APN才可以上网，用户不希望模组开机就自动激活网卡，而是希望在需要的时候，由用户主动激活某一路网卡，然后在网络业务完成后再对网卡进行去激活。 | [手动激活一路网卡](./application.md#手动激活一路网卡)        |
| SIM卡需要配置APN才可以上网，用户不希望模组开机就自动激活多路网卡，而是希望在需要的时候，由用户主动激活多路网卡，比如一路用来访问公共网络，另一路用来访问某个专用网络，然后在网络业务完成后再对网卡进行去激活。 | [手动激活多路网卡](./application.md#手动激活多路网卡)        |



## 开机自动激活一路网卡且没有配置APN

这种场景是指，用户没有为任何一路蜂窝无线网卡配置过APN，并且开机自动激活一路网卡。这里其实有2种情况：

情况1：用户没有配置过任何一路蜂窝无线网卡开机自动激活。这种情况下，QuecPython开机时默认激活第一路蜂窝无线网卡。即下图中的NICn表示第一路蜂窝无线网卡NIC1。

情况2：用户通过`dataCall.setAutoActivate`方法配置了某一路蜂窝无线网卡开机自动激活。这种情况下，QuecPython开机时会激活用户指定的那一路蜂窝无线网卡。

不管是上面哪种情况，模组开机仅自动激活了一路蜂窝无线网卡。此时使用socket、http、mqtt等模块进行网络业务时，无需指定网卡，系统会自动选择已激活的那一路蜂窝无线网卡进行网络通信。

![](../../../media/network-comm/nic/cellular/无APN激活一路网卡.png)

在这种场景下，用户应该按照下面的步骤来编写其应用代码：

步骤1：使用checkNet等待网络就绪。

步骤2：进行网络通信。

下面以socket为例，说明这种场景下，如何编写代码进行网络通信，示例如下：

```python
import checkNet
import usocket


def main():
    stage, state = checkNet.waitNetworkReady(20)
    if stage == 3 and state == 1:
        print('Network connected successfully.')
        # 创建一个socket对象
        sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
        # 解析域名
        try:
            sockaddr = usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
        except Exception:
            print('Domain name resolution failed.')
            sock.close()
            return
        # 建立连接
        sock.connect(sockaddr)
        # 向服务端发送消息
        ret = sock.send('GET /News HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
        print('send {} bytes'.format(ret))

        # 接收服务端消息
        data = sock.recv(256)
        print('recv {} bytes:'.format(len(data)))
        print(data.decode())

        # 关闭连接
        sock.close()
    else:
        print('Network connected failed, stage={}, state={}'.format(stage, state))


if __name__ == '__main__':
    main()
```

上述示例代码下载链接：[示例下载](https://github.com/QuecPython/examples/blob/main/network-comm/nic/cellular/example_socket_activate_default_nic_no_apn.py)

这种场景下，用户没有为任何一路蜂窝无线网卡配置过APN，模组开机后也能正常进行网络通信。但是会存在一个很大的问题：用户很可能会发现，按照上述示例编写的应用程序，在有的地方可以正常运行，有的地方不能正常运行，尤其是在中国以外的其他国家，大概率不能正常运行，具体表现就是`checkNet.waitNetworkReady`方法返回值不是`(3,1)`。原因是因为没有配置APN导致模组网络注册失败。

当用户没有为蜂窝无线网卡配置APN时，模组的蜂窝无线网卡能不能成功激活，取决于当前通信的基站是否有APN自动纠错功能。该功能是指当UE进行网络附着时，如果用户没有给无线网卡配置APN或者配置了错误的APN，基站会自动下发一个正确的APN给设备，并让UE附着成功。

上面的应用程序能正常运行的根本原因就在于此。目前中国境内，大部分运营商的基站是有APN自动纠错功能的，但是也有一些不支持。我们无法提前判断某一个基站是否支持这个功能，因此我们建议用户在使用模组时一定要配置APN。也就是下面我们要介绍的场景。



## 开机自动激活一路网卡且配置APN

这种场景是指，用户根据使用的SIM卡，为某一路蜂窝无线网卡配置了正确的APN，并且开机自动激活的也是用户配置了APN的这一路蜂窝无线网卡。

这种情况下，模组使用socket、http、mqtt等模块进行网络业务时，无需指定网卡，系统会自动选择已激活的那一路蜂窝无线网卡进行网络通信。并且模组根据配置的APN不同，也可以访问不同的网络，如下图所示。需要说明的是，下图中APN1和APN2，并不是表示模组可以为某一路蜂窝无线网卡同时配置两个APN，而是表示为某一路蜂窝无线网卡配置不同的APN时，可以访问不同的网络。

![](../../../media/network-comm/nic/cellular/有APN激活一路网卡.png)

在这种场景下，用户应该按照下面的步骤来编写其应用代码：

步骤1：检查开机自动激活的那一路网卡的当前APN配置，确认是否需要配置APN。如果当前APN已经是用户需要配置的，则无需重复配置；否则需要重新配置，并重启设备。

步骤2：使用checkNet等待网络就绪。

步骤3：进行网络通信。

下面以开机自动激活第一路蜂窝无线网卡为例，说明这种情况下，如何编写代码，使用socket进行网络通信，示例如下：

```python
import checkNet
import usocket
import dataCall
from misc import Power

# 用户需要配置的APN信息，根据实际情况修改
usrCfg = {'apn': '3gnet', 'username': '', 'password': ''}


def checkAPN():
    # 获取第一路网卡的APN信息，确认当前使用的是否是用户指定的APN
    pdpCtx = dataCall.getPDPContext(1)
    if pdpCtx != -1:
        if pdpCtx[1] != usrCfg['apn']:
            # 如果不是用户需要的APN，使用如下方式配置
            ret = dataCall.setPDPContext(1, 0, usrCfg['apn'], usrCfg['username'], usrCfg['password'], 0)
            if ret == 0:
                print('APN configuration successful. Ready to restart to make APN take effect.')
                print('Please re-execute this program after restarting.')
                # 重启后按照配置的信息进行拨号
                Power.powerRestart()
            else:
                print('APN configuration failed.')
                return False
        else:
            print('The APN is correct and no configuration is required')
            return True
    else:
        print('Failed to get PDP Context.')
        return False


def main():
    checkpass = checkAPN()
    if not checkpass:
        return

    stage, state = checkNet.waitNetworkReady(20)
    if stage == 3 and state == 1:
        print('Network connected successfully.')
        # 创建一个socket对象
        sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
        # 解析域名
        try:
            sockaddr = usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
        except Exception:
            print('Domain name resolution failed.')
            sock.close()
            return
        # 建立连接
        sock.connect(sockaddr)
        # 向服务端发送消息
        ret = sock.send('GET /News HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
        print('send {} bytes'.format(ret))

        # 接收服务端消息
        data = sock.recv(256)
        print('recv {} bytes:'.format(len(data)))
        print(data.decode())

        # 关闭连接
        sock.close()
    else:
        print('Network connected failed, stage={}, state={}'.format(stage, state))


if __name__ == '__main__':
    main()
```

上述示例代码下载链接：[示例下载](https://github.com/QuecPython/examples/blob/main/network-comm/nic/cellular/example_socket_activate_default_nic_set_apn.py)



## 开机自动激活多路网卡且配置APN

这种场景指的是，用户需要模组开机时自动激活多路蜂窝无线网卡，其应用程序可以通过不同的网卡来访问不同的网络，比如一路网卡用于访问公网，一路网卡用于访问专网。

这种场景中，用户需要配置开机自动激活多路蜂窝无线网卡，并且分别为这几路需要激活的网卡配置不同的APN。下图是开机自动激活第一路和第二路蜂窝无线网卡的示意图。我们以此图为例进行说明。

当用户配置开机自动激活第一路和第二路蜂窝无线网卡，并且为第一路蜂窝无线网卡配置的是可以访问公网的APN，为第二路蜂窝无线网卡配置的是只能访问某个专网的APN。在开机自动激活这两路蜂窝无线网卡后，情况如下：

* http/https/mqtt这些功能默认只能使用第一路蜂窝无线网卡进行网络通信，用户无法指定使用哪一路蜂窝无线网卡。

* socket功能可以通过bind接口来绑定IP和端口号，决定使用哪一路蜂窝无线网卡进行网络通信。如果用户没有指定，则默认使用第一路蜂窝无线网卡。

![](../../../media/network-comm/nic/cellular/有APN激活多路网卡.png)

> 在QuecPython中，当模组激活了多路蜂窝无线网卡，在使用socket功能进行网络通信时，如果用户没有指定使用哪一路蜂窝无线网卡，则系统会默认在激活的几路网卡中，选择使用网卡编号最小的那一路进行网络通信。比如模组激活了第一路和第二路网卡，则默认选择使用第一路；如果模组激活了第二路和第三路，则默认选择使用第二路。

这种场景下，用户的应用代码应按照下面的步骤来编写：

步骤1：配置哪几路蜂窝无线网卡开机自动激活。

步骤2：分别为需要开机自动激活的蜂窝无线网卡配置APN。

步骤3：重启设备，使配置生效。

步骤4：使用checkNet等待网络就绪。

步骤5：进行网络通信。



下面提供一个例程，该例程中配置了第一路和第二路蜂窝无线网卡开机自动激活，并分别为第一路蜂窝无线网卡和第二路蜂窝无线网卡配置了不同的APN。然后分别创建了两个socket对象，socket1不指定网卡，即默认选择第一路网卡进行网络通信；socket2指定使用第二路蜂窝无线网卡与某个专网进行网络通信。示例代码如下：

```python
import checkNet
import usocket
import dataCall
from misc import Power


# 用户需要配置的APN信息，根据实际情况修改
usrCfg1 = {'profileID': 1, 'apn': '3gnet', 'username': '', 'password': ''}
usrCfg2 = {'profileID': 2, 'apn': '3gwap', 'username': '', 'password': ''}


def checkAPN(usrCfg, reboot=False):
    if type(usrCfg) != dict:
        print("Error: Input is not a dictionary.")
        return False

    print('Check the APN configuration of the {} network card.'.format(usrCfg['profileID']))
    # 获取网卡的APN信息，确认当前使用的是否是用户指定的APN
    pdpCtx = dataCall.getPDPContext(usrCfg['profileID'])
    if pdpCtx != -1:
        if pdpCtx[1] != usrCfg['apn']:
            # 如果不是用户需要的APN，使用如下方式配置
            ret = dataCall.setPDPContext(usrCfg['profileID'], 0, usrCfg['apn'], usrCfg['username'], usrCfg['password'], 0)
            if ret == 0:
                print('APN configuration successful.')
                # 重启后按照配置的信息进行拨号
                if reboot:
                    print('Ready to restart to make APN take effect.')
                    print('Please re-execute this program after restarting.')
                    Power.powerRestart()
                else:
                    return True
            else:
                print('APN configuration failed.')
                return False
        else:
            print('The APN is correct and no configuration is required')
            return True
    else:
        print('Failed to get PDP Context.')
        return False


def main():
    # 使能第一路网卡开机自动激活功能
    dataCall.setAutoActivate(1, 1)
    # 使能第一路网卡自动重连功能
    dataCall.setAutoConnect(1, 1)
    # 使能第二路网卡开机自动激活功能
    dataCall.setAutoActivate(2, 1)
    # 使能第二路网卡自动重连功能
    dataCall.setAutoConnect(2, 1)

    # 检查第一路网卡的APN配置，暂时不重启
    checkpass = checkAPN(usrCfg1, reboot=False)
    if not checkpass:
        return
    # 检查第二路网卡的APN配置，配置后重启
    checkpass = checkAPN(usrCfg2, reboot=True)
    if not checkpass:
        return

    stage, state = checkNet.waitNetworkReady(20)
    if stage == 3 and state == 1:
        print('Network connected successfully.')
        # 分别获取第一路和第二路网卡的IP地址信息
        ret1 = dataCall.getInfo(usrCfg1['profileID'], 0)
        ret2 = dataCall.getInfo(usrCfg2['profileID'], 0)
        print('NIC{}：{}'.format(usrCfg1['profileID'], ret1))
        print('NIC{}：{}'.format(usrCfg2['profileID'], ret2))
        ip_nic1 = None
        ip_nic2 = None
        if ret1 == -1 or ret1[2][2] == '0.0.0.0':
            print("Error: Failed to get the IP of the NIC{}.".format(usrCfg1['profileID']))
            return
        else:
            ip_nic1 = ret1[2][2]
        if ret2 == -1 or ret2[2][2] == '0.0.0.0':
            print("Error: Failed to get the IP of the NIC{}.".format(usrCfg2['profileID']))
            return
        else:
            ip_nic2 = ret2[2][2]
        print('NIC{} IP：{}'.format(usrCfg1['profileID'], ip_nic1))
        print('NIC{} ip：{}'.format(usrCfg2['profileID'], ip_nic2))

        print('---------------sock1 test-----------------')
        # 创建socket对象
        sock1 = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
        # 解析域名
        try:
            sockaddr = usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
        except Exception:
            print('Domain name resolution failed.')
            sock1.close()
            return
        # 建立连接
        sock1.connect(sockaddr)
        # 向服务端发送消息
        ret = sock1.send('GET /News HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
        print('send {} bytes'.format(ret))
        # 接收服务端消息
        data = sock1.recv(256)
        print('recv {} bytes:'.format(len(data)))
        print(data.decode())
        # 关闭连接
        sock1.close()
        print('---------------sock2 test-----------------')
        sock2 = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
        sock2.bind((ip_nic2, 0))
        sock2.settimeout(10)
        # 配置服务器IP和端口，下面的IP和端口仅作示例参考
        server_addr = ('192.0.2.2', 8305)
        # 建立连接
        sock2.connect(server_addr)
        # 向服务器发送消息
        ret = sock2.send('test data.')
        print('send {} bytes'.format(ret))
        # 接收服务端消息
        try:
            data = sock2.recv(256)
            print('recv {} bytes:'.format(len(data)))
            print(data.decode())
        except Exception:
            print('No reply from server.')
        # 关闭连接
        sock2.close()
    else:
        print('Network connected failed, stage={}, state={}'.format(stage, state))


if __name__ == '__main__':
    main()
```

上述示例代码下载链接：[示例下载](https://github.com/QuecPython/examples/blob/main/network-comm/nic/cellular/example_socket_activate_multiple_nic_set_apn.py)



## 手动激活一路网卡

手动激活蜂窝无线网卡的场景一般比较少，这种场景主要用于满足用户的一些特殊的需求。比如用户不需要设备开机时自动激活蜂窝无线网卡，而是在需要的时候，由用户应用程序来主动进行网卡激活操作。需要说明的是，用户手动进行网卡激活和模组开机自动进行网卡激活，两者没有本质上的区别，主要区别在于激活的时间点。在激活成功后，使用网卡进行网络通信上，这两种激活方式是完全一样的。

我们先来说一下，手动激活一路网卡的场景。这种场景下，当用户手动激活蜂窝无线网卡后，使用网卡进行网络通信时，其过程示意如下图所示：

![](../../../media/network-comm/nic/cellular/有APN激活一路网卡.png)

可以看到，这种场景下，用户使用网卡进行网络通信的过程和“开机自动激活一路网卡且配置APN”的场景中是完全一样的。这种场景下，用户应按照如下步骤来编写代码：

步骤1：关闭开机自动激活网卡功能。该配置重启生效。

步骤2：为需要手动激活的网卡配置APN。该配置重启生效。

步骤3：重启设备，使配置生效。

步骤4：在需要的时候，手动进行蜂窝无线网卡的激活。

步骤5：确认是否激活成功。

步骤6：进行网络通信。

步骤7：根据需要决定在通信结束后，是否需要对网卡进行去激活。

下面以手动激活第一路蜂窝无线网卡为例，说明这种情况下，如何编写代码，以及使用socket进行网络通信，示例如下：

```python
import checkNet
import usocket
import dataCall
import utime
from misc import Power

# 用户需要配置的APN信息，根据实际情况修改
usrCfg1 = {'profileID': 1, 'apn': '3gnet', 'username': '', 'password': ''}

'''
环境配置：
1.关闭开机自动拨号功能
2.APN配置
'''
def cfgEnv(usrCfg):
    errcode = 0
    need_reboot1 = False
    need_reboot2 = False
    if "datacall_config.json" not in uos.listdir('/usr'):
        # 关闭第一路网卡开机自动激活功能
        dataCall.setAutoActivate(usrCfg['profileID'], 0)
        # 建议用户使能网卡自动重连功能
        dataCall.setAutoConnect(usrCfg['profileID'], 1)
        need_reboot1 = True

    print('Check the APN configuration.')
    # 获取网卡的APN信息，确认当前使用的是否是用户指定的APN
    pdpCtx = dataCall.getPDPContext(usrCfg['profileID'])
    if pdpCtx != -1:
        if pdpCtx[1] != usrCfg['apn']:
            # 如果不是用户需要的APN，使用如下方式配置
            ret = dataCall.setPDPContext(usrCfg['profileID'], 0, usrCfg['apn'], usrCfg['username'], usrCfg['password'], 0)
            if ret == 0:
                print('APN configuration successful.')
                need_reboot2 = True
            else:
                print('APN configuration failed.')
                errcode = -1
        else:
            print('The APN is correct and no configuration is required')
            errcode = 0
    else:
        print('Failed to get PDP Context.')
        errcode = -1
    # 重启使配置生效
    if need_reboot1 or need_reboot2:
        print('Ready to restart.')
        print('Please re-execute this program after restarting.')
        Power.powerRestart()
        utime.sleep(2)
    return errcode


def main():
    if cfgEnv(usrCfg1) == -1:
        return

    # 手动激活蜂窝无线网卡
    print('Prepare to activate the NIC{}.'.format(usrCfg1['profileID']))
    ret = dataCall.activate(usrCfg1['profileID'])
    if ret == -1:
        print('NIC activation failed.')
        return

    stage, state = checkNet.waitNetworkReady(10)
    if stage == 3 and state == 1:
        print('Network connected successfully.')
        # 获取第一路网卡的IP地址等信息
        ret = dataCall.getInfo(usrCfg1['profileID'], 0)
        print('NIC{}：{}'.format(usrCfg1['profileID'], ret))

        print('---------------sock test-----------------')
        # 创建socket对象
        sock = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
        # 解析域名
        try:
            sockaddr = usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
        except Exception:
            print('Domain name resolution failed.')
            sock.close()
            return
        # 建立连接
        sock.connect(sockaddr)
        # 向服务端发送消息
        ret = sock.send('GET /News HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
        print('send {} bytes'.format(ret))
        # 接收服务端消息
        data = sock.recv(256)
        print('recv {} bytes:'.format(len(data)))
        print(data.decode())
        # 关闭连接
        sock.close()

        # 用户根据需要决定是否需要在通信结束对网卡进行去激活
        # dataCall.deactivate(usrCfg1['profileID'])
    else:
        print('Network connected failed, stage={}, state={}'.format(stage, state))


if __name__ == '__main__':
    main()
```

上述示例代码下载链接：[示例下载](https://github.com/QuecPython/examples/blob/main/network-comm/nic/cellular/example_socket_manually_activate_one_nic_set_apn.py)



## 手动激活多路网卡

这种场景和前面描述的“开机自动激活多路网卡且配置APN”的场景，在配置和功能使用上没有太大的区别。唯一的区别就是需要用户先把开机自动激活网卡功能关闭，然后在用户需要的时候手动进行多路网卡的激活操作。

下图是用户手动激活第一路和第二路蜂窝无线网卡的示意图。我们以此图为例进行说明。

假如用户为第一路蜂窝无线网卡配置的是可以访问公网的APN，为第二路蜂窝无线网卡配置的是只能访问某个专网的APN。在用户手动激活这两路蜂窝无线网卡后，情况如下：

* http/https/mqtt这些功能默认只能使用第一路蜂窝无线网卡进行网络通信，用户无法指定使用哪一路蜂窝无线网卡。

* socket功能可以通过bind接口来绑定IP和端口号，决定使用哪一路蜂窝无线网卡进行网络通信。如果用户没有指定，则默认使用第一路蜂窝无线网卡。

![](../../../media/network-comm/nic/cellular/有APN激活多路网卡.png)

这种场景下，用户的应用代码应按照下面的步骤来编写：

步骤1：关闭开机自动激活网卡功能。该配置重启生效。

步骤2：为需要手动激活的网卡配置APN。该配置重启生效。

步骤3：重启设备，使配置生效。

步骤4：在需要的时候，手动进行多路蜂窝无线网卡的激活。

步骤5：确认是否激活成功。

步骤6：进行网络通信。

步骤7：根据需要决定在通信结束后，是否需要对网卡进行去激活。



下面提供一个例程，该例程中首先关闭了开机自动激活网卡功能，并分别为第一路蜂窝无线网卡和第二路蜂窝无线网卡配置了不同的APN。然后分别创建了两个socket对象，socket1不指定网卡，即默认选择第一路网卡进行网络通信；socket2指定使用第二路蜂窝无线网卡与某个专网进行网络通信。示例代码如下：

```python
import checkNet
import usocket
import dataCall
import utime
from misc import Power

# 用户需要配置的APN信息，根据实际情况修改
usrCfg1 = {'profileID': 1, 'apn': '3gnet', 'username': '', 'password': ''}
usrCfg2 = {'profileID': 2, 'apn': '3gwap', 'username': '', 'password': ''}

'''
关闭开机自动拨号功能
返回值表示是否需要重启设备
True - 需要重启
False - 不需要重启
'''
def disableAutoActivate():
    need_reboot = False
    if "datacall_config.json" not in uos.listdir('/usr'):
        # 关闭第一路网卡开机自动激活功能
        dataCall.setAutoActivate(1, 0)
        # 建议用户使能网卡自动重连功能
        dataCall.setAutoConnect(1, 1)
        need_reboot = True
    return need_reboot

'''
为蜂窝无线网卡配置APN参数
返回值表示是否需要重启设备
True - 需要重启
False - 不需要重启
'''
def cfgAPN(usrCfg):
    need_reboot = False
    print('Check the APN configuration of the NIC{}.'.format(usrCfg['profileID']))
    # 获取网卡的APN信息，确认当前使用的是否是用户指定的APN
    pdpCtx = dataCall.getPDPContext(usrCfg['profileID'])
    if pdpCtx != -1:
        if pdpCtx[1] != usrCfg['apn']:
            # 如果不是用户需要的APN，使用如下方式配置
            ret = dataCall.setPDPContext(usrCfg['profileID'], 0, usrCfg['apn'], usrCfg['username'], usrCfg['password'], 0)
            if ret == 0:
                print('APN configuration successful.')
                need_reboot = True
            else:
                raise ValueError("APN configuration failed.")
        else:
            need_reboot = False
            print('The APN is correct and no configuration is required')
    else:
        raise ValueError("Failed to get PDP Context.")
    return need_reboot


def main():
    need_reboot1 = disableAutoActivate()
    need_reboot2 = cfgAPN(usrCfg1)
    need_reboot3 = cfgAPN(usrCfg2)
    if need_reboot1 or need_reboot2 or need_reboot3:
        print('Ready to restart.')
        print('Please re-execute this program after restarting.')
        Power.powerRestart()
        utime.sleep(2)

    # 手动激活蜂窝无线网卡
    print('Prepare to activate the NIC{}.'.format(usrCfg1['profileID']))
    ret = dataCall.activate(usrCfg1['profileID'])
    if ret == -1:
        print('NIC activation failed.')
        return
    print('Prepare to activate the NIC{}.'.format(usrCfg2['profileID']))
    ret = dataCall.activate(usrCfg2['profileID'])
    if ret == -1:
        print('NIC activation failed.')
        return

    stage, state = checkNet.waitNetworkReady(10)
    if stage == 3 and state == 1:
        print('Network connected successfully.')
        # 分别获取第一路和第二路网卡的IP地址信息
        ret1 = dataCall.getInfo(usrCfg1['profileID'], 0)
        ret2 = dataCall.getInfo(usrCfg2['profileID'], 0)
        print('NIC{}：{}'.format(usrCfg1['profileID'], ret1))
        print('NIC{}：{}'.format(usrCfg2['profileID'], ret2))
        ip_nic1 = None
        ip_nic2 = None
        if ret1 == -1 or ret1[2][2] == '0.0.0.0':
            print("Error: Failed to get the IP of the NIC{}.".format(usrCfg1['profileID']))
            return
        else:
            ip_nic1 = ret1[2][2]
        if ret2 == -1 or ret2[2][2] == '0.0.0.0':
            print("Error: Failed to get the IP of the NIC{}.".format(usrCfg2['profileID']))
            return
        else:
            ip_nic2 = ret2[2][2]
        print('NIC{} IP：{}'.format(usrCfg1['profileID'], ip_nic1))
        print('NIC{} ip：{}'.format(usrCfg2['profileID'], ip_nic2))

        print('---------------sock1 test-----------------')
        # 创建socket对象
        sock1 = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
        # 解析域名
        try:
            sockaddr = usocket.getaddrinfo('python.quectel.com', 80)[0][-1]
        except Exception:
            print('Domain name resolution failed.')
            sock1.close()
            return
        # 建立连接
        sock1.connect(sockaddr)
        # 向服务端发送消息
        ret = sock1.send('GET /News HTTP/1.1\r\nHost: python.quectel.com\r\nAccept-Encoding: deflate\r\nConnection: keep-alive\r\n\r\n')
        print('send {} bytes'.format(ret))
        # 接收服务端消息
        data = sock1.recv(256)
        print('recv {} bytes:'.format(len(data)))
        print(data.decode())
        # 关闭连接
        sock1.close()
        print('---------------sock2 test-----------------')
        sock2 = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM, usocket.TCP_CUSTOMIZE_PORT)
        sock2.bind((ip_nic2, 0))
        sock2.settimeout(10)
        # 服务器IP和端口，下面的IP和端口仅作示例参考
        server_addr = ('192.0.2.2', 8305)
        # 建立连接
        sock2.connect(server_addr)
        # 向服务器发送消息
        ret = sock2.send('test data.')
        print('send {} bytes'.format(ret))
        # 接收服务端消息
        try:
            data = sock2.recv(256)
            print('recv {} bytes:'.format(len(data)))
            print(data.decode())
        except Exception:
            print('No reply from server.')
        # 关闭连接
        sock2.close()

        # 用户根据需要决定是否需要在通信结束对网卡进行去激活
        # dataCall.deactivate(usrCfg1['profileID'])
        # dataCall.deactivate(usrCfg2['profileID'])
    else:
        print('Network connected failed, stage={}, state={}'.format(stage, state))


if __name__ == '__main__':
    main()
```

上述示例代码下载链接：[示例下载](https://github.com/QuecPython/examples/blob/main/network-comm/nic/cellular/example_socket_manually_activate_multiple_nic_set_apn.py)



## 配置DNS服务器地址

模组在激活蜂窝无线网卡时，只要使用的是公共网络（Internet）的APN，核心网一般都会分配DNS服务器地址给模组。用户正常情况下是不需要去手动配置DNS服务器地址的。但是有时候，会出现核心网分配的DNS服务器地址无法使用的情况，此时用户可以尝试手动配置DNS服务器地址。

可以使用如下方法来配置用户指定的DNS服务器地址：

```
dataCall.setDNSServer(profileID, simID, priDNS, secDNS)
```

关于该方法的详细描述，可以参考QuecPython官方网站中wiki文档的[DNS配置](../../../../api_reference/iotlib/dataCall.md#DNS配置功能)部分。这里说明一下`profileID`和`simID`参数的确定。

* `profileID`

  在配置DNS服务器地址时，`profileID`参数选择什么值，取决于激活的是哪一路蜂窝无线网卡。一般是第一路，即`profileID`参数选择`1`。

* `simID`

  默认写0即可。

下面是配置第一路蜂窝无线网卡的DNS服务器地址的示例：

```python
import dataCall
dataCall.setDNSServer(1, 0, "8.8.8.8", "114.114.114.114")
```

