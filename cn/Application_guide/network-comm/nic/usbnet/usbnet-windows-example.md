# Windows 系统使用 USB 网卡

下面以 QuecPython EC200UEU_AB 开发板为例，在 Windows 环境下使用 USB 网卡功能进行演示。

> RNDIS 模式在 Windows 下使用无需驱动可直接加载使用，如需使用 ECM 模式在 Windows 系统下使用，我们也提供了在 Windows 下 ECM 驱动进行适配。

1. 开发环境准备  
   
   请参考[快速入门章节](../../../../Getting_started/README.md)（如已熟悉请跳过此步骤），搭建开发使用环境。 

2. 打开 QPYCOM 工具连接设备

    将模块上电，通过USB口连接电脑，使用QPYCOM工具连接 USB 的 python 交互口。
    ![](../../../media/network-comm/nic/network_usbnet_demo_qpycom.png)

3. 通过 QPYCOM 工具输入交互命令开启 USB 网卡

   具体USBNET功能接口请参考 Wiki 上对应的 [USBNET章节](../../../../api_reference/peripherals/misc.USBNET.md)。  

   交互口输入以下命令。  
    ```python
    # 首先从misc中导入USBNET包
    >>> from misc import USBNET
    >>> from misc import Power

    # 查询当前USBNET的工作模式，1 表示ECM模式, 3 表示 RNDIS模式。
    >>> USBNET.get_worktype()
    1

    # 设置为RNDIS模式
    >>> USBNET.set_worktype(USBNET.Type_RNDIS)
    0

    # 重启模组，用以生效之前配置的模式
    >>> Power.powerRestart()

    # 重启模组后，导入所需的包
    >>> from misc import USBNET
    >>> import dataCall

    >>> USBNET.get_worktype()
    3

    # 查询模组是否拨号成功
    >>> dataCall.getInfo(1,0)
    (1, 0, [1, 0, '10.62.213.130', '211.138.180.4', '211.138.180.5'])

    # 有拨号信息后打开USB网卡
    >>> USBNET.open()
    0
    ```

 4. 查看 USB 网卡连接

    可以看到增加了一个 RNDIS 类型的网络接口。

    ![](../../../media/network-comm/nic/network_usbnet_demo_3.png)

    打开 cmd，输入 ipconfig，可以看到基本网络信息都已获取，此时关闭其余网络接口，只保留 USB 网卡的网络接口，电脑也可以正常上网。

    ![](../../../media/network-comm/nic/network_usbnet_demo_4.png)

 5. 查看网络连接

此时可以通过电脑浏览器访问网页。比如浏览器输入 https://developer.quectel.com 即可正常访问。

![](../../../media/network-comm/nic/network_eth_gateway_web.png)
