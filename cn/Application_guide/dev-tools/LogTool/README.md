# 平台Log工具使用说明

本文主要介绍QuecPython各个平台log抓取工具的使用，包括ASR、UNISOC8850、UNISOC8910和EIGEN平台。

## 模组log的作用

用户在使用模组的过程中，经常会遇到各种问题，为了确认问题的原因和模组代码运行的情况，我们通常需要抓取模组运行的日志。通过这些日志，可以帮助研发人员分析和定位问题，以及判断一些功能的运行是否符合预期。



## 模组Log分类

模组的Log一般分为AP log、CP log、pcap log和dump文件。

* AP log：一般是SDK以及APP输出的日志文件。AP log主要用于确认系统软件运行情况，可用于定位一些软件BUG。

* CP log：一般是Modem输出的无线协议栈相关的日志，比如我们经常说的空口log。CP log主要用于定位确认和无线网络相关的问题，比如模组网络注册问题、网络异常断开问题等。如果是确认设备的网络注册问题，一般需要抓取设备开机阶段的CP log，或者抓取包含CFUN0/1切换过程的CP log。

  > CFUN0/1切换是指通过`net.setModemFun(0)`方法先将设备切换到模式0（最小功能模式），然后再通过`net.setModemFun(1)`方法将设备切换到模式1（全功能模式），目的是为了让设备重新发起网络注册流程。如果用户没有抓到设备开机时的CP log，也可以抓取包含CFUN0/1切换过程的CP log提供给我司研发人员分析问题。

* pcap log：pcap日志文件可用于网络流量分析和调试。使用pcap日志文件可以了解设备或应用在网络层面的工作情况，以找出性能问题或故障。通过分析pcap日志，可以查看网络通信的细节，如连接建立的过程、传输的报文格式、请求和响应的参数等。比如分析TCP/IP、MQTT、HTTP/HTTPS等问题时，可能会经常用到pcap日志。用户一般可使用Wireshark软件工具来打开pcap文件。
* dump文件：当操作系统或应用程序遇到某些不可恢复的错误时，为了记录当时的状态和其他相关信息，它会将内存中的某些部分保存为文件，这些文件称为dump文件。比如程序崩溃、非法内存访问、内存耗尽、栈溢出等异常问题都可能导致模组程序崩溃，从而异常重启。如果用户遇到模组异常重启的情况，很可能是发生dump问题了，此时需要抓取并导出dump文件，提供给我司研发人员进行问题定位和分析。



## 各平台Log工具

下面是各个平台抓取AP/CP log、pcap log以及dump文件所使用的工具：

| 平台       | 模组型号                                                     | AP Log                                                       | CP Log                                                       | pcap Log                                    | Dump                                        | 图标                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| ASR        | EC100Y/EC200N/EC600S/<br>EC600N/EG915N/EC600M/<br>EC800M/EG810M/EC200A/<br/>UC200A | [CATStudio](./ASR.md#使用CATStudio抓取)<br>[QWinLog](./ASR.md#使用QWinLog抓取) | [CATStudio](./ASR.md#使用CATStudio抓取)<br/>[QWinLog](./ASR.md#使用QWinLog抓取) | [CATStudio](./ASR.md#Pcap文件导出)          | [CATStudio](./ASR.md#Dump文件导出)          | ![](../../media/dev-tools/LogTool/CatStudio.png)![](../../media/dev-tools/LogTool/QWinLog.png) |
| UNISOC8850 | EC600G/EC800G                                                | [ArmLogel](./UNISOC8850.md#AP和CP日志抓取)                   | [ArmLogel](./UNISOC8850.md#AP和CP日志抓取)                   | [ArmLogel](./UNISOC8850.md#Pcap文件导出)    | [DTools](./UNISOC8850.md#Dump文件导出)      | ![](../../media/dev-tools/LogTool/ArmLogel.png)![](../../media/dev-tools/LogTool/DTools.png) |
| UNISOC8910 | EC200U/EC600U/EG912U/<br/>EG915U                             | [coolwatcher](./UNISOC8910.md#AP日志抓取)                    | [ArmTracer](./UNISOC8910.md#CP日志抓取)                      | [coolwatcher](./UNISOC8910.md#Pcap文件导出) | [coolwatcher](./UNISOC8910.md#Dump文件导出) | ![](../../media/dev-tools/LogTool/cooltools.png) ![](../../media/dev-tools/LogTool/ArmTracer.png) |
| EIGEN      | EC600E/EC800E                                                | [EPAT](./EIGEN.md#AP和CP日志抓取)                            | [EPAT](./EIGEN.md#AP和CP日志抓取)                            | [EPAT](./EIGEN.md#Pcap文件导出)             | [EPAT](./EIGEN.md#Dump文件导出)             | ![](../../media/dev-tools/LogTool/EPAT.png)                  |



## 目录

[ASR平台Log工具使用说明](./ASR.md)

[EIGEN平台Log工具使用说明](./EIGEN.md)

[UNISOC8850平台Log工具使用说明](./UNISOC8850.md)

[UNISOC8910平台Log工具使用说明](./UNISOC8910.md)