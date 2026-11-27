# Linux 系统使用 USB 网卡

下面以 QuecPython EC200UEU_AB 开发板为例，在 Linux 环境下使用 USB 网卡功能进行演示。

> ECM 模式在 Linux/Android/IOS 系统中无需驱动，可以直接加载使用。  

1. 开发环境准备  

   请参考[快速入门章节](../../../../Getting_started/README.md)（如已熟悉请跳过此步骤），搭建开发使用环境。 
   此处 Linux 系统使用的是 Ubuntu 版本，在电脑下搭建 Linux 环境测试。

2. 打开 QPYCOM 工具连接设备

    将模块上电，通过USB口连接电脑，使用QPYCOM工具连接 USB 的 python 交互口。
    ![](../../../media/network-comm/nic/network_usbnet_demo_qpycom.png)

3. 编写 USB 网卡开机启动脚本

    具体 USB 网卡功能接口请参考 Wiki上对应的 [USBNET章节](../../../../api_reference/peripherals/misc.USBNET.md)。  
    新建一个 main.py 文件，输入以下内容：

    ```python
    # 导入所需包
    from misc import USBNET
    from misc import Power
    import sim
    import dataCall
    import utime
    
    def usbnet_start():
        #判断 USBNET 工作模式为 ECM，若不是，则设置为 ECM 并重启模组。
        saved_type = USBNET.get_worktype()
        if saved_type != USBNET.Type_ECM :
            USBNET.set_worktype(USBNET.Type_ECM)
            Power.powerRestart()
        utime.sleep(2)
        #判断 SIM 卡状态，没有插卡则退出。
        sim_info = sim.getStatus()
        if sim_info != 1:
            return
        #循环查询注网状态，得到注网信息则开启 USBNET。
        while True:
            lte = dataCall.getInfo(1,0)
            if type(lte) == tuple and lte[2][0] == 1:
                if lte[2][2] != '0.0.0.0':
                    break
            utime.sleep(1)
        #开启USBNET功能。
        USBNET.open()
    
    usbnet_start()
    ```

4. 导入开机启动文件  

    通过 QPYCom 文件栏把编写的脚本拖入模组文件系统 usr 分区，开机启动会执行 main.py 文件。

    ![](../../../media/network-comm/nic/network_usbnet_demo_5.png)

5. USB 连接Ubuntu

    将设备 USB 口接到 Ubuntu 系统下。

    ![](../../../media/network-comm/nic/network_usbnet_ubuntu_choose.png)

6. 确认网卡连接正常

    网卡加载后，通过 `ifconfig` 命令查看网卡状态，可以看到新增 usb0 网卡。如下图所示:

    ![](../../../media/network-comm/nic/network_usbnet_ubuntu_nic.png)

7. 确认网络连接

    此处通过 ping python.quectel.com 确认是否连接正常。如下图所示：

    ![](../../../media/network-comm/nic/network_usbnet_ubuntu_ping.png)
