# Android 系统使用 USB 网卡

下面以 QuecPython EC200UEU_AB 开发板为例，在 Android 手机环境下使用 USB 网卡功能进行演示。

> 1. ECM 模式在 Linux/Android/IOS 系统中无需驱动，可以直接加载使用。    
> 2. ECM 模式下部分型号模块 MAC 地址手机无法识别，请根据 Wiki 上对应 [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) 章节，使用 MAC 配置接口更新可用 MAC 即可。

1. 开发环境准备  

   请参考[快速入门章节](../../../../Getting_started/README.md)（如已熟悉请跳过此步骤），搭建开发使用环境。  
   另需要手机一部，以及能够转换成手机接口的转换线。

2. 打开 QPYCOM 工具连接设备

    将模块上电，通过 USB 口连接电脑，使用 QPYCOM 工具连接 USB 的 python 交互口。
    ![](../../../media/network-comm/nic/network_usbnet_demo_qpycom.png)

3. 编写 USB 网卡开机启动脚本

    具体 USB 网卡功能接口请参考 Wiki 上对应的 [USBNET章节](../../../../api_reference/peripherals/misc.USBNET.md)。  
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

    通过 QPYCom 文件栏把编写的脚本拖入模组文件系统 usr 分区，开机启动会执行main.py文件。

    ![](../../../media/network-comm/nic/network_usbnet_demo_5.png)

5. USB 连接手机

    通过电脑配置好开机启动后，将 USB 口接到手机上的 Type-C 口，并按 PWRKEY 键给模组上电。请自行准备转接线。

    ![](../../../media/network-comm/nic/network_usbnet_demo_6.png)

6. 手机网络连接

    模块连接手机上电后，手机可在状态栏看到 <···> 样图标，表示手机已识别到 USB 网卡，此时打开浏览器可以正常访问网络。

    ![](../../../media/network-comm/nic/network_usbnet_demo_7.png)
