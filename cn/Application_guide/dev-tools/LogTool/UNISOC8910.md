# UNISOC8910平台Log工具使用说明

本文将介绍如何使用coolwatcher和ArmTracer这两种工具来进行AP和CP Log的抓取，以及dump文件的导出。

## 概述

### 工具简介

coolwatcher是用来抓取UNISOC8910平台模组AP Log的软件工具，同时支持dump文件导出。

ArmTracer是用来抓取UNISOC8910平台模组CP Log的软件工具，用于分析模组网络注册相关的问题。

### 软件安装

coolwatcher和ArmTracer工具均为免安装软件，只需要将工具压缩包解压到电脑某个目录下即可。需要注意，解压路径中，不要包含中文字符，否则可能导致工具无法正常运行。

### 运行环境

coolwatcher支持在Windows XP/7/10 系统上运行，计算机内存应在4G以上，处理器双核以上。

ArmTracer运行于 Windows XP 以上的系统版本，如果软件无法正常运行，可尝试以管理员身份运行或者以XP SP3兼容模式来运行。



## 硬件连接

抓取log之前，用户应做如下准备：

* 确认已经安装UNISOC8910平台模组的USB驱动程序。

* 使用USB数据线连接模组和计算机。

* 模组上电运行。

进入计算机的设备管理器界面，如果USB驱动程序加载正常，应该可以看到下图所示的几个端口：

![](../../media/dev-tools/LogTool/UNISOC8910-USB-port.png)

图中几个端口的作用如下：

* Quectel Modem：该端口主要用于ppp拨号，也可用于发送AT，但是一般不用该端口发送AT，而是使用专用AT端口。

* Quectel USB AP Log Port：该端口用于抓取AP Log。

* Quectel USB AT Port：该端口用于发送AT指令，是专用AT端口。

* Quectel USB CP Log Port：该端口用于抓取CP Log。

* Quectel USB Diag Port：该端口保留。

* Quectel USB MOS Port：该端口保留。

* Quectel USB REPL Port：对于QuecPython固件而言，该端口是python的REPL端口，用户可通过该端口执行一些python代码。需要注意的是，不同的平台，python的REPL端口名称可能是不同的。如果用户不清楚哪个端口是python的REPL端口，可以咨询我们公司的FAE进行确认。



## 使用指南

确认设备与PC硬件连接正常，并且所有USB端口枚举成功，接下来就可以进行软件工具的配置和使用。

### AP日志抓取

**步骤1：打开AP Log工具**

将cooltools软件工具压缩包解压到电脑某个目录下，需要注意，cooltools工具所在路径中，不要包含中文路径。解压完成后，进入到工具根目录下：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-root-dir.png)

双击coolwatcher_usb.exe文件打开工具，弹出如下配置窗口，窗口左侧为模块型号选择区，右侧为对应型号的配置项：

* 点击①处的8910，进入对应项目的配置界面。

* 确认②处型号为8910。

* 在③处的输入框中，填写“Quectel USB AP Log Port”所对应的端口号。点击“OK”按钮。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-1.png)



**步骤2：等待工具连接模组成功**

步骤1中，端口参数配置完成，点击“OK”后，coolwatcher工具会自动打开“Quectel USB AP Log Port”端口，并与模组进行通信，建立连接。当连接建立成功后，会出现下图所示的界面，左边会有几条绿色提示信息，并且最后会出现“[COM OPEN OK]”的提示信息。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-2.png)

上图就是我们打开coolwatcher工具进入的主界面。其中右边的Trace tool窗口就是AP Log的输出窗口，图中①处是控制AP Log的一些按钮，下面是几个常用按钮功能的介绍：

| 按钮                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-start.png) | 开始，点击后trace窗口开始输出AP Log。                        |
| ![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-stop.png) | 停止，点击后trace窗口停止输出AP Log。                        |
| ![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-clear.png) | 清空，点击后会清空trace窗口的所有AP Log。                    |
| ![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-set.png) | 设置，点击后会弹出设置窗口，可设置自动保存、保存的文件类型、保存路径、单个文件大小、pcap保存等。 |
| ![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-save.png) | 保存，点击后立即保存，实际可以通过“设置”按钮来配置自动保存，无需手动保存。 |

图种②处是AP Log过滤功能的关键字输入框，用户可以在该输入框中输入某个字符串作为关键字并按键盘的回车键确认，工具会把包含该关键字的所有AP Log条目都列举出来。

如果用户在步骤1中，端口参数配置完成，点击“OK”后，没有弹出上面图中右边的Trace tool窗口，而是进入下面的窗口，说明用户使用的软件版本不是最新的。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-old-ver-1.png)

此时按照下面步骤操作即可：

点击菜单栏的“Plugins”，在弹出的下拉窗口中，点击“Activate Tracer”，即可出现Trace tool窗口。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-old-ver-2.png)

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-old-ver-3.png)



**步骤3：参数设置**

点击Trace tool窗口工具栏里面的“设置”按钮，弹出如下设置窗口，用户需要关注和设置的即图中标注的部分：

* ①自动保存，默认是勾选的，如果没有勾选，建议用户勾选该功能。

* ②保存的Log文件类型，支持Bin和trc两种类型，默认是trc。Bin类型的文件需要coolwatcher才能查看，trc为普通文本文件。

* ③单个Log文件大小，当文件大小超过该值时，会自动保存到下一个新的文件中。

* ④Log文件保存路径，用户可根据需要来设置，默认是保存到coolwatcher工具根目录的logs目录下。

* ⑤是否保存pcap文件，默认不保存。建议用户勾选该选项，勾选后，coolwatcher工具会自动抓取并保存pcap文件。

按照上述说明设置完成后，点击“OK”按钮保存。

 ![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-config.png)



**步骤4：开始Log抓取**

点击Trace tool窗口的“开始”按钮，即可看到下方的窗口开始输出Log信息。如下图所示：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-start-trace.png)



**步骤5：Log文件打包**

当需要的AP Log已经抓取完成时，点击Trace tool窗口的“停止”按钮，进入用户设置的Log保存目录下，如果用户没有设置过，则默认保存到coolwatcher工具根目录下的logs文件夹。进入AP Log保存目录后，将logs目录下的所有文件都打包压缩，然后发送给我司FAE或者研发人员，用于分析问题。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-log-dir.png)

> 建议用户在每次抓取log之前，先将默认的log保存路径下的历史log文件删除。



### CP日志抓取

UNISOC8910平台模组CP Log的抓取，需要使用ArmTracer工具，下面将说明如何使用ArmTracer工具来抓取模组CP Log。

#### 抓取步骤

**步骤1：打开ArmTracer工具**

将ArmTracer工具的压缩包解压到电脑某个路径下，注意路径中不要包含中文字符。进入到ArmTracer工具根目录下，双击ArmTracer.exe文件打开该工具，进入如下所示主界面：

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-ui-1.png)

上图中①处是端口参数设置按钮，②处是开始按钮，③处是停止按钮，④处是log保存路径设置按钮。



**步骤2：端口参数配置**

点击工具栏的端口参数设置按钮，弹出如下设置窗口。由于使用的是USB端口，因此只需要选择端口号即可，其他参数一般保持默认。点击“Device Port”选项输入框的下拉按钮，选择“Quectel USB CP Log Port”端口对应的端口号，然后点击“OK”。

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-port-config.png)



**步骤3：设置log保存路径**

点击工具栏的log保存路径设置按钮，弹出如下设置窗口。“Saved Path”是CP log的保存路径，用户根据需要去设置。“Filename”是log文件的名称，ArmTracer工具会使用该名称和时间戳组合为最终的log文件名称。设置好log文件保存路径和文件名称后，点击“OK”。

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-log-save-setting.png)



**步骤4：开始CP log抓取**

点击工具栏的开始按钮，此时可能会自动弹出步骤3中的“log setting”窗口，直接点击“OK”。如果硬件连接正常，工具配置正确，应该可以看到工具窗口开始输出log信息，如下图所示：

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-ui-2.png)



**步骤5：Log文件打包保存**

当需要的Log抓取完成了，点击ArmTracer工具栏的停止按钮，log会自动保存到之前设置的目录下。将log保存目录下的最新生成的.tra文件都打包压缩为一个文件，发送给我司FAE即可。

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-log-file.png)



#### 配置工具自动抓取

抓取CP Log的目的是为了分析设备的无线网络相关的问题，比如设备网络注册失败问题。而设备的网络注册过程是在开机阶段就自动进行的，所以分析网络注册失败这一类问题，一般都需要抓设备开机阶段的CP Log。为了方便抓取设备开机阶段的CP log，我们可以配置ArmTracer工具自动检测CP log端口并抓取log。下面将说明如何配置该功能。

> 如果仅仅是分析模组网络注册失败问题，也可以不配置该功能，采用手动抓取的方式，抓取包含CFUN0/1切换过程的CP log。



**步骤1：打开配置文件**

进入到ArmTracer工具根目录下，找到ArmTracer.ini配置文件，如下图所示：

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-config-file.png)

以文本的方式打开该配置文件，找到[USBDevice]配置项，如下图所示：

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-USBDevice-1.png)



**步骤2：修改配置文件**

找到[USBDevice]配置项后，我们需要在`DeviceList`中增加“Quectel USB CP Log Port”端口，并将`DeviceSel`的值改为“Quectel USB CP Log Port”，修改后如下：

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-USBDevice-2.png)

修改完成后，保存退出。



**步骤3：选择自动检测CP端口**

如果修改配置文件之前，ArmTracer工具就已经处于打开状态，在修改配置文件之后，需要关闭ArmTracer工具并重新打开，配置才会生效。打开ArmTracer工具后，进入下图所示主界面：

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-ui-3.png)

点击图中箭头所指的图标，会弹出“Software Setting”窗口，可以看到此时“USB Device Description”对应的值已经是我们前面配置的“Quectel USB CP Log Port”，勾选“Automatic Detect CP Port”功能，点击“OK”。

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-software-setting.png)

如果配置正确，理论上在点击“OK”后，工具就会自动检测CP log端口并开始抓取log，可以看到CP log输出，如下所示：

![](../../media/dev-tools/LogTool/UNISOC8910-armtracer-log-output.png)

为了进一步验证配置是否正常，我们重启设备，观察ArmTracer工具是否可以自动检测CP log端口并自动开始输出log，如果能观察到ArmTracer工具log输出窗口不断有log输出，说明配置是生效的。就可以开始正式抓取CP log了。





### Dump文件导出

coolwatcher支持dump文件导出，当模组发生dump死机问题时，可以通过coolwatcher导出dump文件，然后将其打包压缩后发送给我司研发分析。下面介绍如何使用coolwatcher工具来导出dump文件。

**步骤1：配置dump模式**

模组在默认情况下，如果发生dump，会直接重启。这种情况下，无法导出dump文件。因此，我们需要在模组正常运行时，先发送AT指令来配置模组的dump模式，确保发生dump时不会自动重启，而是一直停在dump状态，等待用户导出dump文件。

下面是UNISOC8910平台模组配置dump模式，需要发送的AT指令:

```
AT+QDBGCFG="DUMPCFG",0
```

打开QPYcom工具，“COM Port”选择“Quectel USB AT Port”端口，点击“Open Port”，然后发送上面的AT指令：

![](../../media/dev-tools/LogTool/UNISOC8850-dump-config.png)



>上述AT发送后立即生效，但是掉电不保存，如果重启设备了，就需要重新发送一次。



**步骤2：复现dump问题**

按照步骤1发送相关AT指令后，就可以去复现相关的dump问题，当模组出现dump问题时，从coolwatcher的log输出窗口可以看到如下log信息：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump.png)



**步骤3：导出dump文件**

依次点击菜单栏的“Tools”——>“Blue Screen Dump”，如下图所示：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump-export-1.png)

点击之后会弹出dump导出设置窗口，如下图所示：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump-export-2.png)

在上图所示窗口中，用户需要进行的操作如下：

* “Mode”：选择“8910 (FreeRTOS)”选项。

* “XML file name”：选择图中所示的文件，一般默认就是该文件。

* “Elf file name”：选择模组当前使用的固件所对应的elf文件，一般可直接在固件包中找到，如下图所示。如果固件包中没有，可以联系我司FAE获取对应固件的elf文件。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-elf-file.png)

> 如果用户使用的coolwatcher工具版本较新，在导出dump文件时，是不需要选择elf文件的，即上面的“Bule Screen Dump”窗口中，“Elf file name”选项是处于无法操作的状态。如果“Elf file name”选项处于可选择的状态，用户按照上述说明选择对应elf文件即可。

* “Output Directory”：选择dump文件保存目录。需要注意选择的路径中不要包含任何中文字符以及空格等。

按照上述说明配置后，点击“start”按钮开始dump文件的导出并等待dump文件导出完成，如下图所示：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump-export-3.png)

进入到dump文件保存目录下，将所有的dump文件打包压缩后，发送给我司FAE或者研发人员进行问题定位和分析。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump-export-4.png)



### Pcap文件导出

coolwatcher工具支持自动保存pcap文件，但是该功能默认没有开启。开启方式在前面[AP日志抓取](./UNISOC8910.md#AP日志抓取)章节的步骤3中其实已经提到，即点击Trace tool窗口工具栏里面的“设置”按钮，弹出如下设置窗口，勾选图中⑤处的“Save Pcap”功能，点击“OK”。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-config.png)

勾选该功能后，工具就会自动抓取并保存pcap文件，保存的路径为上图④处设置的路径下的cap子目录，cap目录为工具自动创建，无需用户新建该文件夹。

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-pcap-file.png)

如需要我司FAE或者研发协助分析网络应用相关的问题，可以将AP Log文件和这些pcap文件一起打包压缩后，发送给我司FAE。



## 常见问题

**问题1：**dump导出失败了。

dump导出失败的原因有多种，一般常见的原因有下面这些。

情况1：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump-export-failed-1.png)

如果导出失败的提示信息类似上图这种，那么可能的原因有：

* 连接模组和电脑的USB数据线太长，用户可尝试换一根较短的USB数据线。

* 模组和电脑之间不是通过USB直连，而是经过了HUB集线器。这种情况很容易导致dump导出失败，建议用户使用USB数据线直连模组和电脑。

情况2：

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump-export-failed-2.png)

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-dump-export-failed-3.png)

如果导出失败的提示信息类似上述两种情况，那很可能是选择的elf文件和模组当前使用的固件不匹配导致。这种情况需要用户选择和模组当前使用的固件相匹配的elf文件。

如果用户按照上述几种情况排查后，仍然无法导出，可以联系我司FAE进行协助。



**问题2：**按照前文说明打开配置coolwatcher工具后，点击“开始”按钮后，trace窗口无AP Log输出。

AP Log默认是从USB端口输出，对于coolwatcher工具来说，即从"Quectel USB AP Log Port"端口输出。如果用户之前通过相关AT将AP Log输出切换到了物理串口的Debug串口，就会出现点击“开始”按钮后，trace窗口无AP Log输出的现象。此时，用户可以按照下面步骤进行操作：

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

由于上述AT指令需要重启才能生效，所以发送上述AT指令后，重启模组。然后回到coolwatcher工具界面，重新点击“开始”按钮，应该就可以看到有Log输出了。



**问题3：**log中看到如下这种信息，是系统某些功能运行异常吗？

![](../../media/dev-tools/LogTool/UNISOC8910-cooltools-invalid-tdb.png)

上图中这些信息并不是模组功能运行异常，而是因为使用的coolwatcher工具版本比较旧，工具无法识别解析一些log。用户可联系我司FAE申请最新版本的coolwatcher工具。

