# BG95 开发板介绍

## 支持的模组列表

- [BG95-M1](https://www.quectel.com.cn/product/eg95)
- [BG95-M3](https://www.quectel.com.cn/product/eg95)
- [BG95-M8](https://www.quectel.com.cn/product/eg95)


## 功能列表

### 基本概述

QuecPython_BG95_EVB 开发板是一款专门针对QuecPython设计的、小巧便携的“口袋型”开发板。

主模组搭载BG95，搭载Type-C接口，开发者仅需一条USB Type-C 数据线即可轻松玩转开发板。

该开发板兼容树莓派zero，树莓派zero的扩展板可以在BG95开发板上直接使用。

### 功能说明

开发板的主要组件、接口布局见下图

![img](./media/bg95/BG95_layout.png)

## 资料下载

- [QuecPython_BG95_Zero开发板规格说明及使用指导手册](https://developer.quectel.com/wp-content/uploads/2024/09/BG95-Pi-Zero系列开发板产品规格及用户指导.pdf)
- [BG95_Board_原理图](https://developer.quectel.com/wp-content/uploads/2024/09/BG95-Pi-Zero系列开发板原理图.pdf)


## 开发板资源

### 开发板接口

**8Pin排针管脚分配表**

| **排针** | **编号** | **丝印**  | **引脚** | **功能** |
| -------- | -------- | --------- | -------- | -------- |
| JP6      | 1        | PWK_AUTO  | 15       | POWERKEY |
| JP6      | 2        | PWK_AUTO  | -        | GND    |
| JP6      | 3        | VDD_EXT   | -        | 1.8V     |
| JP6      | 4        | VDD_EXT   | 29       | VDD_EXT  |
| JP6      | 5        | MODULE_EN | -        | 3.6V     |
| JP6      | 6        | MODULE_EN | -        | 3.6V     |
| JP6      | 7        | GNSS_EN   | -        | 3.3V     |
| JP6      | 8        | GNSS_EN   | 51       | GNSS_EN  |

- **1 & 2**:自动开机
- **3 & 4**:VDD_EXT 连接时可对外供电，测量功耗时需断开
- **5 & 6**:接通 DCDC 对模组供电
- **7 & 8**:使能 GNSS 有源供电

**40Pin排针管脚分配表**

| **排针** | **编号** | **名称** | **功能复用**                         | **功能**                                      | **排针** | **编号** | **名称** | **功能复用**               		| **功能** 											|
| -------- | -------- | -------- | ------------------------------------ | --------------------------------------------- | -------- | -------- | -------- | -------------------------------- | ------------------------------------------------- |
| JP7      | 1        | 3V3      | -                          			| 3.3V输出 										| JP8      | 2        | 5V       | -                          		| 5V输出   											|
| JP7      | 3        | SDA1     | I2C1_SDA<br>GPIO14            		| I2C1串行数据<br>通用输入输出        			| JP8      | 4        | 5V       | -                          		| 5V输出   											|
| JP7      | 5        | SCL1     | I2C1_SCL<br>GPIO13            		| I2C1串行时钟<br>通用输入输出        			| JP8      | 6        | GND      | -                          		| 接地       										|
| JP7      | 7        | P35      | GPIOX<br>UART4_TXD            		| 通用输入输出<br>UART4发送         			| JP8      | 8        | TX2      | UART2_TXD                  		| UART2发送       									|
| JP7      | 9        | GND      | -                          			| 地       										| JP8      | 10       | RX2      | UART2_RXD                  		| UART2接收         								|
| JP7      | 11       | P06      | GPIO3<br>PCM_DIN              		| 通用输入输出<br>数字音频输入信号         		| JP8      | 12       | P37      | GPIOX<br>I2C_SDA              	| 通用输入输出<br>I2C串行数据						|
| JP7      | 13       | P05      | GPIO2<br>PCM_SYNC             		| 通用输入输出<br>数字音频同步信号         		| JP8      | 14       | GND      | -                          		| 接地     											|
| JP7      | 15       | P04      | GPIO1<br>PCM_CLK              		| 通用输入输出<br>数字音频时钟信号         		| JP8      | 16       | P86      | GPIO19                     		| 通用输入输出         								|
| JP7      | 17       | 3V3      | -                          			| 3.3V输出 										| JP8      | 18       | P88      | GPIO21                     		| 通用输入输出         								|
| JP7      | 19       | MO2      | SPI2_MOSI<br>GPIO8<br>UART0_TXD  	| SPI2主输出从输入<br>通用输入输出<br>UART0发送 | JP8      | 20       | GND      | -                          		| 接地     											|
| JP7      | 21       | MI2      | SPI2_MISO<br>GPIO7<br>UART0_RXD  	| SPI2主输入从输出<br>通用输入输出<br>UART0接收 | JP8      | 22       | P20      | GPIOX                      		| 通用输入输出         								|
| JP7      | 23       | CLK2     | SPI2-CLK<br>GPIO5<br>I2C0_CLK    	| SPI2时钟<br>通用输入输出<br>I2C0时钟         	| JP8      | 24       | CS2      | SPI2_CS<br>GPIO6<br>I2C0_SDA		| SPI2片选<br>通用输入输出<br>I2C0串行数据        	|
| JP7      | 25       | GND      | -                          			| 接地     										| JP8      | 26       | P38      | GPIOX                      		| 通用输入输出         								|
| JP7      | 27       | P25      | GPIO9<br>SPI0_CS              		| 通用输入输出<br>SPI0片选        				| JP8      | 28       | P26      | GPIO10<br>SPI0_CLK            	| 通用输入输出<br>SPI0时钟         					|
| JP7      | 29       | P34      | GPIOX<br>UART4_RXD            		| 通用输入输出<br>UART4接收         			| JP8      | 30       | GND      | -                          		| 接地     											|
| JP7      | 31       | P87      | GPIO20                     			| 通用输入输出        							| JP8      | 32       | PWM      | PWM0<br>GPIO17                	| PWM0输出<br>通用输入输出         					|
| JP7      | 33       | P85      | GPIO18<br>PWM1                		| 通用输入输出<br>PWM1输出         				| JP8      | 34       | GND      | -                          		| 接地     											|
| JP7      | 35       | P28      | GPIO12<br>SPI0_MISO<br>UART3_RXD 	| 通用输入输出<br>SPI0主输入从输出<br>UART3接收 | JP8      | 36       | P39      | GPIOX                      		| 通用输入输出        								|
| JP7      | 37       | P07      | GPIO4<br>PCM_DOUT             		| 通用输入输出<br>数字音频输出信号        		| JP8      | 38       | P27      | GPIO11<br>SPI0_MOSI<br>UART3_TXD | 通用输入输出<br>SPI0主输出从输入<br>UART3发送		|
| JP7      | 39       | GND      |                            			| 接地     										| JP8      | 40       | P36      | GPIOX<br>I2C_SCL              	| 通用输入输出<br>I2C串行时钟         				|


开发板主要管脚布局见下图

![img](./media/bg95/BG95_EVB.png)

> **小提示**
> 开发板的更多资料，请访问 <https://www.quectel.com.cn/download-zone>

### 开发板配置

外设资源管脚分配表明细如下：

| 序号 | 名称                      | 型号         | 是否支持 | 丝印  | 备注     |
| ---- | ------------------------- | ------------ | -------- | ----- | -------- |
| 1    | USB_TYPEC接口             | -            | 是       | -     | -        |
| 2    | SIM卡座                   | SMN-315-ARP7 | 是       | CARD2 | Nano-SIM |
| 3    | LED指示灯（详见下方说明） | -            | 是       | -     | -        |

**板载指示灯如下**

- **SIM_CHK**: 卡槽 SIM 卡存在时，SIM_CHK 指示灯亮起。
- **NET**:网络指示灯。
- **PSM**:睡眠指示灯。
- **PWM**:PWM 功能指示灯。
- **PWR**:电源指示灯。

> 上述指示灯位置参考上文开发板正面图下方丝印（模组所在面为正面）


## 快速入门

### 硬件准备

以下是开发板上手之后的硬件准备步骤

- <font color='red'>**Step1：硬件准备**</font>

    开始之前需要准备以下配件

    1. **开发板**
    2. **USB 数据线** (USB-A TO USB-TypeC)
    3. **PC** (Windows10)
    4. **SIM 卡**
    5. **4G 天线**

- <font color='red'>**Step2：天线&SIM卡安装**</font>

安装开发板配套的天线,安装位置为 MAIN 天线座位置,并将 SIM 卡插入开发板上的 SIM 卡座。

- <font color='red'>**Step3：开发板连接**</font>

使用 USB Type-C 数据线连接开发板的 Type-C 接口和电脑 USB 口即可完成供电。

- <font color='red'>**Step4：开发板开机**</font>

将PWK_AUTO的两个引脚使用跳帽短接之后上电后会自动开机，或者上电后长按PWK开机，建议从上电到按下PWK按键之间的时间间隔不少于 30 ms。

进行以上操作后等待主板上电源指示灯亮（主板上丝印为PWR灯，参考上文指示灯）

<font color='red'>**执行以上操作后PWR灯常亮即开机成功**</font>


### 驱动安装

- 打开[QuecPython官网驱动下载链接](https://www.quectel.com.cn/download-zone)

- 选择与模组型号匹配的驱动，该开发板搭载的模组有BG95，在下载页面搜索该型号的USB驱动[QuecPython_USB Driver Win10_BG](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverQ_NDIS_V2.7.14.zip)点击下载，如果开发板搭载的是其他型号同样按照以上操作即可。

  ![](media/readme/driver_eg912u.png)


- 下载后解压驱动压缩包，找到**"setup.exe"** 或者是**"setup.bat"**，双击运行即可，安装完之后打开设备管理器就可以看到设备管理器中端口的黄色感叹号消失了，说明安装成功，能够正常通信。

>  出现 <font color='red'>Mobile ECM Network Adapter </font>或 <font color='red'>CDC Ethernet Control Modle (ECM) </font>等设备未被识别属于正常现象，不影响固件烧录和后续开发，无需理会。

- 右键打开【**我的电脑**】——选择【**管理**】——选择【**设备管理器**】,然后在设备管理器中选择 【**端口**】，按照步骤打开页面后如图所示能刷新出**Quectel USB** 名称开头的串口则USB驱动安装成功。


### 工具安装

- QPYcom获取

使用QuecPython进行开发需要用到专用的开发调试工具——**QPYcom**，包括但不限于调试代码、分析日志、文件传输、烧录固件、合并固件等。

打开[QuecPython工具下载页面](https://www.quectel.com.cn/download-zone)，根据操作系统选择对应的工具进行下载。

*注意:该工具无需安装，解压即用。*

- VSCode插件

编辑Python代码一般会用到专用于Python或者兼容多种语言的IDE，可以有效提升开发效率，这里推荐VSCode

针对VScode，QuecPython推出专用插件实现代码提示、代码补全和串口调试等功能，安装方法见下图

在VSCode中点击侧边栏插件市场，在插件市场中搜索 “QuecPython”，根据搜索结果下载该插件即可

![](media/readme/VScode_1.png)

### 固件烧录

**固件获取**

模组在出厂时通常烧录有标准 AT 固件或 QuecOpen 固件，如需基于 QuecPython 对模块进行开发，需要手动为其重新烧录专门的 QuecPython 固件。

[官网固件下载地址](https://developer.quectel.com/resource-download?cid=15)

![](media/readme/Firmware.png)

面对官网种类众多的固件，如何选择合适的固件包，首先需要知道使用的模组的型号，模组型号可以通过模组的镭雕或者发送AT指令来获得。

在获取到模组型号之后根据模组的型号去官网下载该模组对应的固件即可,固件的名称一般为`QPY_OCPU_版本号_模组型号_FW`，没有特殊需求默认下载最新版本即可。

点击下载列的下载按钮即可下载，下载成功后会得到一个固件名称命名的压缩包文件(zip包)

从官网下载的固件包为压缩包格式，固件压缩包下载到本地后，需进行解压。解压后可获得两个文件，其中 .bin 格式的是 QuecPython 固件本体，.md 格式的是更新日志。

![](media/readme/firmware_zip.png)

> 请务必将压缩包内容解压至一个**不包含中文、空格和其他特殊字符**的路径下，否则下载工具可能无法正常识别到固件，同时下载工具路径也**不可包含中文、空格和其他特殊字符**。

**固件烧录**

- <font color='red'>**Step1：创建项目**</font>

首先确保模组连接正常并已开机，打开QPYcom工具进入下载页面，点击**"创建"**项目，新建要下载的固件项目，项目名称自定义，配置完成后持久化保存

- <font color='red'>**Step2：选择固件**</font>

选择要下载到模组的固件(根据要下载的模组型号选择对应的固件，若为新格式固件则所有模组型号都选择.bin文件即可）

如果是老版本固件，固件文件格式不一定以bin结尾，其中BG95系列选择官网下载固件包解压后mbn结尾的文件。

- <font color='red'>**Step3：设置下载模式**</font>

单击**“Download script”**右侧的下拉选择箭头，选择**"Download FW"**

- <font color='red'>**Step4：开始烧录固件**</font>

点击**"Download FW"**后，开始下载固件,下载过程会有进度条和进度百分比显示，等待下载完毕会有弹窗提示下载成功

> 其中BG95系列需要打开DM串口后再点击**"Download FW"**，直接点击会提示打开串口 

![](media/readme/firmware.gif)

### REPL调试

REPL全称为<font color='red'>**Read-Eval-Print-Loop (交互式解释器)**</font>，可以在REPL中进行QuecPython程序的调试

运行 **QPYcom** 工具后，选择正确的串口（波特率无需指定）并打开，即可开始 Python 命令行交互。

- <font color='red'>**Step1：进入交互页面**</font>

打开QPYcom工具，端口选择连接**Quectel USB REPL Port**，选择“交互”界面

- <font color='red'>**Step2：打开串口**</font>

点击“打开串口”按钮，在交互界面输入**print(‘hello world’)**，按回车后可以看到执行的结果信息

```python
>>> print('hello world')
hello world
```

![](media/readme/hello_world.png)

<font color='red'>*注意：工具交互页面输入时需要输入英文字符，中文字符将会被屏蔽*</font>

## 开发第一个脚本

### 编写第一个脚本文件

创建*helloworld.py*文件输出“hello world”，打印“hello world”，编写脚本如下所示：

```python
print("hello world")
```

通过QPYcom将上面编辑好的文件下载到模组中去并运行

![](media/readme/QPYcom_hello.jpg)

### PC与模组间的文件传输

**下载方法一:**

<img src="media/readme/QPYcom_drag.jpg" alt="image-2021081301" style="zoom:150%;" />

- <font color='red'>**Step1：打开串口**</font>

首先选择模组的交互口,点击"打开串口"按钮

- <font color='red'>**Step2：通过工具按钮下载**</font>

可以通过文件页面右侧上面的 "**+**","**-**" 按钮来上传和删除文件

- <font color='red'>**Step3：通过拖拽形式下载**</font>

也可以通过拖拽的方式将文件页面左侧显示的本地文件直接拖拽到右侧模组中去（也可以拖拽文件夹）

- <font color='red'>**Step4：下载进度和结果**</font>

下载过程中会在状态栏显示下载文件名和下载进度

**下载方法二:**

<img src="media/readme/QPYcom_sc_1.jpg" alt="image-2021081301" style="zoom:150%;" />

- <font color='red'>**Step1：创建项目**</font>

根据需求，创建用户项目（点击"创建"按钮）,步骤同上文烧录固件

- <font color='red'>**Step2：配置要下载的文件**</font>

选择需要下载到模块的用户脚本(在"用户脚本"区域通过右键菜单添加)

- <font color='red'>**Step3：设置下载模式**</font>

左击下拉选择箭头，选择"下载脚本"，即"Download Script"

- <font color='red'>**Step4：开始下载脚本**</font>

点击"下载脚本"开始下载脚本，下载过程中有进度条提示

### 运行脚本文件

- <font color='red'>**Step1：下载脚本**</font>

将要执行的脚本文件下载到模组中去,具体步骤参考上文

- <font color='red'>**Step2：通过repl执行脚本**</font>

打开串口之后，在QPYcom交互页面输入以下代码执行脚本文件

```python
import example
example.exec("/usr/helloworld.py") # filePath为要执行的脚本文件路径
```

- <font color='red'>**Step3：通过GUI工具执行脚本**</font>

或者通过QPYcom文件页面 **执行** 按钮执行脚本文件,在工具的文件页面选择要执行的脚本文件然后点击 "**▷**"按钮

执行结果如图

![](media/readme/QPYcom_repl_hello.png)

### 停止程序运行

如何停止正在运行的程序,根据运行的脚本文件类型有以下方法:

<table valign="center">
	<thead>
    <tr>
		<th>程序名是否</br>为main.py</th>
		<th>程序中是否</br>包含了死循环</th>
		<th>程序中是否</br>使用了多线程</th>
		<th valign="middle">停止步骤</th>
	</tr>
	 </thead>
    <tr>
		<td rowspan="4">✓</td>
		<td rowspan="2">✓</td>
		<td rowspan="1">✓</td>
		<td>（1）按 Ctrl + A 键进入 RAW 模式</br>（2）按 Ctrl + D 键重启 QuecPython 虚拟机</br>（3）按 Ctrl + B 键回到普通交互模式</br>（4）若以上方法无效，请重新烧录固件</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>（1）按 Ctrl + C 键打断程序运行</br>（2）若以上方法无效，请重新烧录固件</td>
	</tr>
	<tr>
		<td rowspan="2">✗</td>
		<td rowspan="1">✓</td>
		<td>（1）按 Ctrl + A 键进入 RAW 模式</br>（2）按 Ctrl + D 键重启 QuecPython 虚拟机</br>（3）按 Ctrl + B 键回到普通交互模式</br>（4）若以上方法无效，请耐心等待程序运行结束</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>（1）按 Ctrl + C 键打断程序运行</br>（2）若以上方法无效，请重新烧录固件</td>
	</tr>
	<tr>
		<td rowspan="4">✗</td>
		<td rowspan="2">✓</td>
		<td rowspan="1">✓</td>
		<td>（1）按 Ctrl + D 键重启 QuecPython 虚拟机</br>（2）若以上方法无效，请直接重启模块</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>（1）按 Ctrl + D 键重启 QuecPython 虚拟机</br>（2）若以上方法无效，请直接重启模块</td>
	</tr>
	<tr>
		<td rowspan="2">✗</td>
		<td rowspan="1">✓</td>
		<td>（1）按 Ctrl + C 键打断程序运行</br>（2）若以上方法无效，请重新烧录固件或直接重启模块</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>（1）按 Ctrl + C 键打断程序运行</br>（2）若以上方法无效，请重新烧录固件或直接重启模块</td>
	</tr>
</table>


## 低功耗测试指导

BG95模组支持多种工作模式，每种模式的功耗各不相同，常见的工作模式有以下几种：

**ACTIVE**：模组进行LTE数传、GSM通话或RTOS在运行逻辑时的状态，功耗受到具体业务和网络通信制式的影响，CPU本身功耗和网络射频功率都有所不同，故实际功耗在不同工况下会有较大差异。

**IDLE**：此时模组处于空闲状态，硬件正常在电，RTOS保持运行，但没有任何线程需要被执行。有业务启动或网络业务呼入时，会立即恢复运行。ECX00U系列模组会在IDLE模式下降低时钟频率，进入轻睡眠状态（关闭高速时钟，但CPU不休眠的状态）。

**休眠**：休眠模式的前提是模组处于空闲状态且使能autosleep，进入休眠模式后，RTOS暂停运行，模组的时钟频率会变慢，部分外设控制器（UART、SPI等）下电，同时只保留部分中断控制器，达到减小功耗的目的。

**PSM**：PSM模式是3GPP协议所规定的一种低功耗模式，这种模式下，模组只会周期性的唤醒并执行业务，其余时间都处于PSM休眠中。PSM休眠时，模组行为和功耗都近似于关机。

**关机**：模组完全下电的状态，此时BB芯片和外设控制器完全关闭，但PMIC仍是在电的。一般可由Powerkey或者RTC闹钟唤醒。

| 模组型号             | BG95                |
| :------------------- | :------------------ |
| IDLE（LTE FDD@64ms） | 18.9 mA(catm@128ms) |
| SLEEP(LTE FDD@64ms)  | 1.89 mA(catm@128ms) |
| SLEEP(CFUN0)         | 0.575mA             |
| PSM                  | 5.10 μA             |
| 关机                 | 14.87 μA            |

### 测试准备

- <font color='red'>**Step1：硬件准备**</font>

  开始之前需要准备以下配件

  1. **开发板**
  2. **USB 数据线** (USB-A TO USB-TypeC)
  3. **PC** (Windows10)
  4. **SIM 卡**
  5. **4G 天线**
  6. **Power Monitor Mini**（功耗仪）

- <font color='red'>**Step2：天线&SIM卡安装**</font>

安装开发板配套的天线,安装位置为 MAIN 天线座位置,并将 SIM 卡插入开发板上的 SIM 卡座。

- <font color='red'>**Step3：开发板连接**</font>

使用 USB Type-C 数据线连接开发板的 Type-C 接口和电脑 USB 口，功耗仪正极连接MODULE_EN的Vbat，具体位置可以参考[BG95用户指导](https://developer.quectel.com/wp-content/uploads/2024/09/BG95-Pi-Zero%E7%B3%BB%E5%88%97%E5%BC%80%E5%8F%91%E6%9D%BF%E4%BA%A7%E5%93%81%E8%A7%84%E6%A0%BC%E5%8F%8A%E7%94%A8%E6%88%B7%E6%8C%87%E5%AF%BC.pdf)，功耗仪负极连接模组GND即可。具体连接如图：

<img src="./media/bg95/PowerMonitor_connect.png"  />

> 注意！！！
>
> 测量功耗时需要拔出所有跳线帽，以免测量结果受其他外界因素影响

### 测量功耗

#### 关机耗流

使用 USB Type-C 数据线连接开发板的 Type-C 接口和电脑 USB 口，待测设备硬件连接完毕后，不进行开机，断开USB_VBus（位于开发板背面，USB开关拨至off即可），即可直接测量关机电流。此时，模组不工作，对外的输出引脚也全部拉低或悬空，整体平均耗流保持相对稳定，基本都在uA级。

<img src="./media/bg95/close_power.png" alt="image-20250709190947650"  />

#### 空闲耗流

完成关机电流测量后，长按powerkey开机，模组开机默认的功耗模式是IDLE（即空闲模式）。此时模组处于空闲状态，硬件正常在电，但没有任何线程需要被执行。有业务启动或网络业务呼入时，会立即恢复运行。

<img src="./media/bg95/free_power.png" alt="image-20250709193332755"  />

#### 休眠耗流

模组处于空闲状态时，若autosleep被使能，则模组会进入休眠状态。此时，模组会关闭部分IP核（如外设控制器和中断控制器等）并且降低时钟频率，从而实现功耗的降低。

完成空闲耗流测量后，打开USB_VBus，电脑端打开[QPYcom工具](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V3.8.0.zip)，导入`pm`模块，调用`autosleep`方法打开模组的自动休眠，在没有任务执行时，模组会自动进入休眠状态，调用方法后记得断开USB_VBus(USB开关拨至off)，否则模组不会进入休眠状态。[API参考](../api_reference/syslib/pm.md#pm.autosleep)。关于休眠耗流的详细介绍[参考](../Application_guide/system/power-consumption.md)。

<img src="./media/bg95/open_auto_sleep.png" alt="image-20250709194047484"  />

进入休眠模式后，模组应有的耗流波形会有周期稳定的凸起，如下图：	

<img src="./media/bg95/sleep_power.png" alt="image-20250709194047484"/>

#### PSM耗流

启用PSM时，需要先向基站发送请求（通过ATTACH或TA_UPDATER携带定时器信息）。基站若支持进入PSM，则会在对应的REQUEST中下发定时器信息，需要注意的是，实际PSM的参数会使用基站下发的值，并不一定和我们请求的值相同。模组会以基站下发的值来配置定时器时间，当模组进入IDLE且ACT定时器超时，模组会关闭CPU和一切射频，此时相当于部分关机，只是保留了比关机更多的唤醒源，一般包括ACT、TAU定时器和PSM_INT。功耗一般下降到微安级别。

打开USB_VBus，电脑端打开[QPYcom工具](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V3.8.0.zip)，启用PSM，开启后记得断开USB_VBus(USB开关拨至off)。关于PSM详细介绍[参考](../Application_guide/system/power-consumption.md#PSM-模式)。启用PSM模式的[API参考](../api_reference/syslib/pm.md#pm.set_psm_time)。

<img src="./media/bg95/set_psmmode.png" alt="image-20250709194047484"  />

等待ACT定时器超时后，我们可查看PSM休眠下的底电流。如图可见，PSM的底电流远低于普通的autosleep模式，和关机耗流的大小、特征较为相似。

<img src="./media/bg95/psm_power1.png" alt="image-20250709194047484"  />

> 注意：
>
> 进入PSM模式需要确认运营商是否支持，还需配置支持的网络制式 ! ! !
>
> 更多耗流测试指导请[参考这里](../Application_guide/system/power-consumption.md)。
