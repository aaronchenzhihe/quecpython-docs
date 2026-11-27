# UNISOC8850平台Log工具使用说明

UNISOC8850平台AP和CP Log的抓取，都是使用ArmLogel工具，而dump文件的导出则需要使用dtools_win工具。本文将介绍如何使用ArmLogel工具来抓取AP和CP的Log，以及使用dtools_win工具导出dump文件。

## 概述

### 工具简介

ArmLogel是一个用来实时监控并抓取UNISOC8850平台模组的内部消息、空中消息、系统消息的测试工具。它可以将来自每个任务的消息分层显示，并提供了对消息结构和数据的解析功能。

### 软件安装

ArmLogel工具为免安装软件，只需要将工具压缩包解压到电脑某个目录下即可。需要注意，解压路径中，不要包含中文字符，否则可能导致工具无法正常运行。

### 运行环境

ArmLogel工具支持在Windows XP/7/10系统上运行，计算机内存应在8G以上，处理器至少双核。



## 硬件连接

抓取log之前，用户应做如下准备：

* 确认已经安装UNISOC8850平台模组的USB驱动程序。

* 使用USB数据线连接模组和计算机。

* 模组上电运行。

进入计算机的设备管理器界面，如果USB驱动程序加载正常，应该可以看到下图所示的几个端口：

![](../../media/dev-tools/LogTool/UNISOC8850-USB-port.png)

几个端口的作用如下：

* Quectel USB AT Port：该端口用于发送AT指令。
* Quectel USB CP Port：该端口用于抓取CP Log。
* Quectel USB Debug Port：在USB连接下，该端口保留，用户可忽略。

* Quectel USB DIAG Port：该端口用于抓取AP Log。

* Quectel USB REPL Port：对于QuecPython固件而言，该端口是python的REPL端口，用户可通过该端口执行一些python代码。需要注意的是，不同的平台，python的REPL端口名称可能是不同的。如果用户不清楚哪个端口是python的REPL端口，可以咨询我们公司的FAE进行确认。



## 使用指南

确认设备与计算机硬件连接正常，并且所有USB端口枚举成功，接下来就可以进行软件工具的配置和使用。

### AP和CP日志抓取

**步骤1：打开ArmLogel工具**

将ArmLogel工具的压缩包解压到电脑某个路径下，注意路径中不要包含中文字符。进入到ArmLogel工具根目录下，双击Logel.exe打开ArmLogel工具后，进入下图所示的界面，其中的“Traces”窗口就是AP Log的输出窗口。CP Log的输出窗口一般默认不直接打开，而是在用户配置好参数并连接设备成功后，会自动弹出。

![](../../media/dev-tools/LogTool/ArmLogel-1.png)



**步骤2：参数配置**

点击菜单栏的设置按钮，如下图所示：

![](../../media/dev-tools/LogTool/ArmLogel-2.png)

弹出如下所示的参数设置窗口，在该窗口中，用户需要设置Log的保存路径以及USB端口相关的参数。当用户设置好这些参数，并开始Log的抓取时，ArmLogel工具会自动保存AP和CP Log到用户设置的目录中。

![](../../media/dev-tools/LogTool/ArmLogel-3.png)

具体的参数设置说明如下：

* 在①处设置Log的保存路径。

* ②处的“Type”选择“Uart”。

* ③处的“Diag Port”选择“Quectel USB DIAG Port”，后面波特率输入框填写2000000即可。

* ④处的“Log Port”选择“Quectel USB CP Port”，后面波特率输入框填写8000000即可。

* 点击⑤处的“OK”按钮确认。

按照上述步骤配置之后，参数配置就完成了。



**步骤3：开始Log抓取**

点击菜单栏的“Capture Log”按钮开始抓取Log：

![](../../media/dev-tools/LogTool/ArmLogel-4.png)

当“Capture Log”按钮变成绿色，并且“Traces”窗口能看到Log输出，说明硬件连接正常，并且工具参数配置也正确。当工具开始输出Log时，还会弹出几个新的窗口，比如“CP All Message”、"CP Lay3 Message"和“Neighbour Cell Info”窗口，这些窗口输出的是CP Log相关的信息。

![](../../media/dev-tools/LogTool/ArmLogel-5.png)



**步骤4：Log文件打包保存**

当需要的Log抓取完成时，点击ArmLogel工具菜单栏的停止按钮，Log会自动保存到之前设置的目录下。

![](../../media/dev-tools/LogTool/ArmLogel-6.png)

进入之前设置的Log保存路径下，该目录下就是ArmLogel工具自动保存的所有Log文件，用户需要将这些文件打包压缩后，发送给我司FAE或者研发，用于分析问题。

![](../../media/dev-tools/LogTool/ArmLogel-log-files.png)



> 由于ArmLogel工具每次打开时，默认都会按照用户之前设置的路径来保存，所以会导致保存Log文件的目录下有大量历史Log文件。因此，建议用户在每次使用ArmLogel工具抓取Log之前，先将历史Log文件都删除，这样可以确保该目录下保存的文件是最新的。



### Dump文件导出

UNISOC8850平台模组抓取dump文件，需要使用dtools_win工具，且需要结合ArmLogel工具来使用。下面将说明如何使用ArmLogel工具合dtools_win工具来导出模组的dump文件。

**步骤1：配置dump模式**

模组在默认情况下，如果发生dump，会直接重启。这种情况下，无法导出dump文件。因此，我们需要在模组正常运行时，先发送AT指令来配置模组的dump模式，确保发生dump时不会自动重启，而是一直停在dump状态，等待用户导出dump文件。

下面是UNISOC8850平台模组配置dump模式，需要发送的AT指令:

```
AT+QDBGCFG="DUMPCFG",0
```

打开QPYcom工具，“COM Port”选择“Quectel USB AT Port”端口，点击“Open Port”，然后发送上面的AT指令：

![](../../media/dev-tools/LogTool/UNISOC8850-dump-config.png)



>上述AT发送后立即生效，但是掉电不保存，如果重启设备了，就需要重新发送一次。



**步骤2：复现dump问题**

按照步骤1发送相关AT指令后，就可以去复现相关的dump问题，当模组出现dump问题时，ArmLogel工具会弹出如下所示的Assert Information窗口：

![](../../media/dev-tools/LogTool/ArmLogel-dump-1.png)

并且Traces窗口中会列出模组当前所有Task的基本信息：

![](../../media/dev-tools/LogTool/ArmLogel-dump-2.png)



**步骤3：停止ArmLogel工具的Log抓取**

由于接下来需要使用dtools_win工具来导出dump文件。dtools_win工具导出dump时使用的也是模组的USB端口，如果不先停止ArmLogel工具，会导致模组的USB端口一直被ArmLogel工具占用，dtools_win工具则无法再使用模组的USB端口。

当模组确认发生dump，并出现步骤2中的现象时，用户需要点击ArmLogel工具菜单栏的停止按钮，停止成功后，该按钮会变成红色。

![](../../media/dev-tools/LogTool/ArmLogel-6.png)



**步骤4：打开dtools_win工具**

双击打开dtools_win工具，进入该工具的主界面，具体如下所示：

![](../../media/dev-tools/LogTool/dtools_win.png)



**步骤5：端口参数配置**

点击下图中①处的图标，弹出端口参数配置窗口，如下所示：

![](../../media/dev-tools/LogTool/dtools-port-config.png)

在弹出的端口参数配置窗口中，依次按照下面的步骤进行配置：

* 图中②处的Chip Type类型，选择8850。

* 选择图中③处的Diag Port，并且后面的端口号选择"Quectel USB DIAG Port"所对应的端口号。

* 最后点击图中④处的“Confirm”按钮。



**步骤6：dump文件导出**

在dtools_win工具的主界面上，先点击下图①处的“Blue Screen Dump”标签，进入对应的标签页：

![](../../media/dev-tools/LogTool/dtools-dump-export.png)

在“Blue Screen Dump”标签页中，依次按照如下步骤进行操作：

* 图中②处的Chip类型需要选择8850，如果步骤5中配置正确，此处应该默认就是8850。

* 图中③处是dump文件导出后的保存路径，用户可以点击输入框后面的按钮选择指定的目录来保存dump文件。

* 点击图中④处的按钮，开始dump文件的导出。下方⑤处会有dump文件导出的进度显示，当导出进度显示为100%时，表示dump文件导出已经完成。

我们可以点击“Open Dump Folder”按钮，直接打开dump文件保存目录，然后将dump文件打包压缩为一个文件后，发送给我司FAE或者研发进行dump问题的分析。



### Pcap文件导出

ArmLogel工具会自动抓取并保存pcap文件，并将pcap文件保存在用户设置的Log保存路径下，用户只需要按照前面[AP和CP日志抓取](./UNISOC8850.md#AP和CP日志抓取)章节的步骤设置即可，无需单独设置其他参数。

模组开机后，如果没有进行过任何网络数据业务，则Log保存路径下不会有pcap文件存在；只有当模组进行进行socket、mqtt以及http等业务时，ArmLogel工具才会抓取并保存pacp文件。

![](../../media/dev-tools/LogTool/ArmLogel-pcap-files.png)



## 常见问题

**问题1：**ArmLogel工具打开后，没有看到“Traces”窗口。

如果用户打开ArmLogel工具后，没有看到“Traces”窗口，可以手动打开。点击菜单栏的“View”选项，在弹出的下拉窗口中，依次点击“Common”——>"Traces"，即可显示“Traces”窗口。



**问题2：**点击菜单栏的“Capture Log”按钮后，该按钮已经变成绿色，但是“Traces”窗口看不到任何的AP Log输出。

AP Log默认是从USB端口输出，对于ArmLogel工具来说，即从"Quectel USB DIAG Port"端口输出。如果用户之前通过相关AT将AP Log输出切换到了物理串口的Debug串口，就会出现“Capture Log”按钮已经变成绿色，但是“Traces”窗口看不到任何的AP Log输出的现象。此时，用户可以按照下面步骤进行操作：

步骤1：确认当前AP Log输出端口。

发送如下AT，查询当前的AP Log输出端口：

```
at+qdbgcfg="tracecfg"
```

打开QPYcom工具，“COM Port”选择“Quectel USB AT Port”端口，点击“Open Port”，然后发送上面的AT指令：

![](../../media/dev-tools/LogTool/at-qdbgcfg-tracecfg-1.png)

如果AT指令查询结果如上图所示，最后两位数字是0和1，说明当前AP Log的输出端口是物理串口Debug口。

此时，用户可以发送如下AT命令，来将AP Log输出端口切换到USB端口：

```
at+qdbgcfg="tracecfg",0,2
```

![](../../media/dev-tools/LogTool/at-qdbgcfg-tracecfg-2.png)

由于上述AT指令需要重启才能生效，所以发送上述AT指令后，重启模组。然后回到ArmLogel工具界面，应该就可以看到有Log输出了。