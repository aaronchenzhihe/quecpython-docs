## 显示器

许多应用需要把信息显示给用户，因此显示屏是很重要的一个显示设备。 QuecPython 系列模块为屏幕显示应用提供了丰富的外设支持，包含的接口类型有`SPI` 和 `MIPI`

一块屏幕通常有两个主要部分：`显示基板` 和 `驱动 IC`。显示基板（也称显示面板）决定了显示的尺寸、分辨率、色彩等参数；驱动 IC （也称屏幕控制器）是匹配显示基板并与基板封装在一起的，它通过一定的接口对外通信并控制显示基板进行显示。

### 类型

现代显示器的种类非常多样化，它们用于各种不同的设备和应用，从个人计算机到商业广告屏，从手机到电视，从游戏设备到嵌入式设备等。以下是一些常见的显示器类型：

1. **液晶显示器（Liquid Crystal Display，LCD）**：
   - 工作原理：LCD 显示器使用液晶分子控制光线的通过，从而显示图像。液晶本身不发光，所以需要背光源。
   - 特点：LCD 显示器体积小、功耗低、发热少，适合嵌入式系统使用。它可以根据需求进行各种尺寸和形状的定制。
   - 应用：LCD 显示器广泛应用于各种嵌入式设备，如手持设备、仪器仪表、家用电器等。
2. **有机发光二极管（Organic Light Emitting Diode，OLED）**：
   - 工作原理：OLED 显示器利用有机物质发光的特性，通过施加电压使得有机物质发出光线，从而显示图像。每一个像素都可以单独发光。
   - 特点：OLED 显示器具有色彩鲜明、对比度高、反应速度快的特点，且可以制造出柔性和透明的显示器。
   - 应用：OLED 显示器目前主要应用于高端移动设备，如智能手机和可穿戴设备。
3. **电子纸显示（E-Ink Display）**：
   - 工作原理：电子纸通过在两个透明电极间夹层中的微胶囊改变显示颜色。每个微胶囊内含有黑色和白色粒子，通过施加电压使得黑色和白色粒子在微胶囊中的位置发生变化，从而改变显示颜色。
   - 特点：电子纸的功耗极低，且具有非常好的阅读体验，尤其在光照条件下，清晰度极高。
   - 应用：电子纸主要应用于电子阅读器，也逐渐被应用到各种显示标签和长时间显示的设备上。
4. **微型 LED（Micro LED）**：
   - 工作原理：微型 LED 显示器由无数微小的 LED 像素组成，每个像素可以单独发光和调整亮度。
   - 特点：微型 LED 显示器色彩丰富、对比度极高、响应速度快，且功耗低、寿命长。
   - 应用：目前微型 LED 主要被应用于高端的大型显示屏和电视产品，但随着技术的进步，未来可能会逐渐应用于更多的嵌入式设备中。

以上就是一些常见的嵌入式系统中的显示器类型。嵌入式系统中的显示器需求可能会根据具体的应用和性能需求有所不同，因此选择适合的显示器类型是非常重要的。

### 驱动框图

其主控，驱动 IC, 接口关系如下：

![1687920840462](../../media/hardware/display/lcd_controller.png)

1. **主控（MCU/CPU）**：这是嵌入式系统的核心部分，负责处理数据，执行代码，以及管理和控制系统的各个部分。
2. **显示接口**：这是主控和显示芯片之间的通信连接。常见的显示接口包括：SPI、I2C、HDMI、VGA、LVDS、MIPI DSI 等等。主控通过这些接口向显示芯片发送数据和命令。
3. **显示芯片**：显示芯片接收来自主控的数据和命令，然后根据这些信息驱动显示面板进行相应的显示。
4. **显示基板（或显示面板）**：显示面板是嵌入式系统中实际显示图像的部分。它包含一系列像素，这些像素根据来自显示芯片的信号来改变其状态，从而显示出图像。

主控通过显示接口向显示芯片发送数据和命令，然后显示芯片根据这些信息驱动显示面板显示图像。这就是这些组件在一个嵌入式系统中的基本关系。

### 适配说明

大部分屏幕控制器可以支持多种显示接口。如 [ILI9486](https://www.waveshare.net/w/upload/7/78/ILI9486_Datasheet.pdf) 支持 MCU 接口（即 8080 接口）、SPI 接口、RGB 接口和 MIPI 接口。

QuecPython 目前支持接口如下：

`SPI 接口`

QuecPython 系列模块的 SPI 接口特点：

- SPI 时钟频率最高可达 50MHz

`MIPI接口`

QuecPython 系列模块的 MIPI 接口特点：

- MIPI 时钟频率最高可达 100MHz

- 最高支持 2Lane

<font color='red'>**QuecPython 系列模组支持情况如下：**</font>

| SoC        | MIPI 接口 | SPI 接口 |
| ---------- | --------- | -------- |
| EC200U_XXX | √         | √        |
| EC600U_XXX | √         | √        |
| EC600N_XXX | ×         | √        |
| EC800N_XXX | ×         | √        |
| EC600M_XXX | ×         | √        |
| EC800M_XXX | ×         | √        |
| EC800G_XXX | ×         | √        |
| EC810M_XXX | ×         | √        |
| EG912N_XXX | ×         | √        |
| EG912U_XXX | √         | √        |
| EG915N_XXX | ×         | √        |
| EG915U_XXX | √         | √        |

<font color='red'>**已适配的屏幕控制器型号如下表所示：**</font>

| 控制器型号 | 类型  | 接口 | 分辨率    | 驱动                                                                                                                                                                                                            | 数据手册                                                                                                                                                                                                         |
| ---------- | ----- | ---- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ILI9225    | Color | SPI  | 176\*220  | [ILI9225.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9225) | [ILI9225.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9225) |
| ILI9341    | Color | SPI  | 240\*320  | [ILI9341.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9341) | [ILI9341.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9341) |
| ILI9806    | Color | MIPI | 480\*800  | [ILI9806.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9806) | [ILI9806.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9806) |
| ST7735     | Color | SPI  | 128\*160  | [ST7735.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7735)    | [ST7735.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7735)    |
| ST7789     | Color | SPI  | 240\*320  | [ST7789.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7789)    | [ST7789.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7789)    |
| ST7701     | Color | MIPI | 480\*854  | [ST7701.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7701)    | [ST7701.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7701)    |
| JD9365     | Color | MIPI | 800\*1280 | [JD9365.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/JD9365)    | [JD9365.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/JD9365)    |
| SC7705     | Color | MIPI | 800\*1280 | [SC7705.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SC7705)    | [SC7705.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SC7705)    |
| SSD1306    | Mono  | SPI  | 128\*64   | [SSD1306.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SSD1306) | [SSD1306.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SSD1306) |
| UC1628     | Mono  | SPI  | 128\*64   | [UC1628.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/UC1628)    | [UC1628.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/UC1628)    |

<font color='red'>**说明**</font>：使用 lcd 方法时，只要参数传输正确，理论是支持任意 SPI或MIPI 屏幕的。

### 驱动结构

![1686981785042](../../media/hardware/display/controller_structure.png)

**QuecPython 为了更自由的控制屏幕，python 脚本可以通过 lcd.lcd_init 传入的参数驱动屏幕。**

<font color='purple'>**在设计上，每驱动一款屏幕差异点仅仅为脚本 lcd.lcd_init 或 lcd.mipi_init 中的参数不一样而已。**</font>

### 屏幕的分类

对屏幕进行分类的讨论将有助于我们对驱动的理解，这里将按照屏幕可显示的色彩来分类，而非 OLED、LCD 等屏幕的面板材料。 一般情况下，屏幕显示的色彩决定了 BPP (Bits Per Pixel)，而 BPP 的不同导致程序的处理方式有一些不同，下面将更直观地列举几种 GRAM 映射到像素点的方式：

![](../../media/hardware/display/screen_driver_RGB565.png)

                  BPP = 16 GRAM 结构

![](../../media/hardware/display/screen_driver_mono.png)

                   BPP = 1 GRAM 结构

![](../../media/hardware/display/screen_driver_gray.png)

```
			  BPP = 4 GRAM 结构
```



从以上图中可以看出，映射方式大概可以分为两类：

- **BBP >= 8**，通常是支持 RGB888、RGB666、RGB565 等编码的彩色屏幕。
- **BPP < 8**，通常是单色的屏幕，可能是黑白的，也可能是灰阶的。

BPP < 8 时，一个字节映射到了多个像素，因此无法直接地控制单个像素。而 QuecPython 为了更好的控制单色屏幕这种情况，lcd_write 接口依旧传入 RGB565 数据，底层将自动完成转换，这样 python 脚本能达到访问单个像素的目的。BPP >= 8 时，则可以轻松地访问单个像素。

<font color='red'>**注意**</font>：对于彩色屏幕，驱动仅支持 RGB565 颜色编码。

### 屏幕控制器驱动

该部分将根据不同的屏幕控制器分别实现显示等功能 。

#### 显示方向

这里设置的屏幕显示方向是完全由屏幕硬件实现的，这个功能在不同的屏幕控制器上会有差异。一共有 8 种可能的显示方向， 显示器可以旋转 0°、90°、180° 或 270°，也可以从顶部或底部查看，默认方向为 0° 和顶部查看。 这 8 (4 × 2) 个不同的显示方向也可以表示为 3 个二进制开关的组合：X-mirroring、Y-mirroring 和 X/Y swapping。

下表将列出全部 8 种组合显示的方向。如果显示方向不正常，请查看下表中的配置开关使其正常工作。

| <img src="../../media/hardware/display/original.png" style="zoom:50%;" />         | **L2R_U2D**                                      | <img src="../../media/hardware/display/mirror_y.png" style="zoom:50%;" />          | Y-mirroring<br />[L2R_D2U]                                        |
| ------------------------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| <img src="../../media/hardware/display/mirror_x.png" style="zoom:50%;" />         | **X_mirroring**<br />**[R2L_U2D]**               | <img src="../../media/hardware/display/mirror_xy.png" style="zoom:50%;" />         | **X-mirroring**<br />**Y-mirroring**<br />**[R2L_D2U]**           |
| <img src="../../media/hardware/display/swap_xy.png" style="zoom:50%;" />          | **X/Y swapping<br />[U2D_L2R]**                  | <img src="../../media/hardware/display/swap_xy_mirror_y.png" style="zoom:50%;" />  | **X/Y swapping<br />Y-mirroring<br />[D2U_L2R]**                  |
| <img src="../../media/hardware/display/swap_xy_mirror_x.png" style="zoom:50%;" /> | **X/Y swapping<br />X-mirroring<br />[U2D_R2L]** | <img src="../../media/hardware/display/swap_xy_mirror_xy.png" style="zoom:50%;" /> | **X/Y swapping<br />X-mirroring<br />Y-mirroring<br />[D2U_R2L]** |

对于不同屏幕控制器，屏幕显示方向的实现并不完全相同，通常分为以下的情况：

> - 对于彩色屏幕，支持 8 个方向的旋转。
> - 对于单色屏幕，如 SSD1306 等屏幕来说，只支持前 4 个方向，即不支持交换 XY 轴。

 <font color='red'>**注解：**</font>

显示方向还和使用的屏幕面板有关系，你可能会发现两种异常的情况：

- 显示方向设置为 L2R_U2D，屏幕却不是按照上表中对应的显示方向。这可能是因为屏幕面板上的走线对 X/Y 方向上进行了镜像，这时应该根据实际情况调整旋转以得到期望的显示方向。

- 旋转了屏幕后，显示内容不见了。这可能是因为屏幕面板的分辨率小于屏幕控制器的分辨率，导致旋转后显示区域没有完全落在屏幕面板上，这时应考虑设置正确的显示区域偏移。

### 常见屏幕参数解析

屏幕驱动 IC 包含多种寄存器，它们控制着各种功能和设定。以下是一些你可能需要关注的寄存器参数：

1. 模式寄存器：这些寄存器通常控制显示模式（如分辨率、颜色深度等）和电源管理模式。例如，一些屏幕驱动 IC 可能有专门的寄存器用于切换睡眠模式或正常模式。

2. 控制寄存器：控制寄存器用于配置屏幕驱动 IC 的各种功能，如屏幕旋转、翻转、反色等。

3. 定时寄存器：这些寄存器用于控制像素时钟、水平同步和垂直同步信号的定时。

4. Gamma 寄存器：Gamma 寄存器用于设置屏幕的 gamma 校正。这对于图像质量和颜色准确性非常重要。

5. RAM 寄存器：RAM 寄存器是用来存储帧缓冲区数据的。驱动 IC 会从这些寄存器读取数据，然后把数据发送到屏幕。

6. 状态寄存器：状态寄存器可以让你读取 IC 的当前状态，比如是否在正常工作、是否有错误发生等。

7. 亮度/对比度寄存器：一些驱动 IC 可能会有这些寄存器来调整屏幕的亮度或对比度。

8. 偏移寄存器：这些寄存器通常用于控制图像在屏幕上的位置。

<font color='red'>需要注意的是 </font>，每个屏幕驱动 IC 都有其自己的寄存器映射和功能。所以在开始使用新的驱动 IC 时，你应该阅读其数据手册来了解其寄存器的详细信息。

### API 说明

详细的接口说明请参考 [class LCD - LCD显示屏](../../../api_reference/peripherals/machine.LCD.md)

<font color='red'>**注意：**</font>该部分为 QuecPython LCD 驱动的相关 API，请驱动屏幕前，详细阅读

### 屏幕调试

#### SPI LCD

SPI LCD 显示屏是使用 SPI 接口进行通信的 LCD 显示屏。这种显示屏的优点在于其接口简单，线数少，使得硬件布线更加简洁。它们非常适合用于微控制器和其他资源有限的系统。

在 SPI LCD 中，数据和命令通过 MOSI 线发送到显示器，而 SCLK 线用于同步数据传输。CS 线用于在与多个设备通信时，选择特定的设备进行通信。

在一般使用中，控制器（如微控制器）会根据显示器的规格（例如分辨率和颜色深度）以及需要显示的内容，通过 SPI 接口发送适当的命令和数据。这可能包括设置显示参数、写入像素数据等。

本章节将基于 QuecPython 开发板驱动[st7789（240*320）](TODO 链接)屏幕，介绍 SPI LCD 的调试。

##### 前期准备工作

在实现 LCD 驱动之前，我们需要一些准备工作，该部分让我了解 QuecPython 驱动一款屏幕需要
哪些前期准备，我们需要知道 quecpython lcd 驱动 LCD 接口以及接口需要哪些参数。还需要对拿到
手的屏幕了解其 ID,驱动 IC 的类型，以及相关的命令。

<font color='red'>**注意：**</font> 原厂提供的初始化参数非常重要，后面 SPI LCD 初始化时，实际为将该参数替换成 lcd.lcd_init 所需要格式的参数。

了解[QuecPython LCD接口](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_init)，才能更好是利用它点亮屏幕 。

<font color='red'>**注意：**</font>目前 QuecPython SPI 驱动分为两种，LCM( Liquid Crystal Module )和通用 SPI( Serial Peripheral Interface )。两者的初始化接口有差异，具体详细可参考[QuecPython LCD接口](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_init)

<font color='red'>**了解 LCD.lcd_init 中的参数，对于 QuecPython 驱动 LCD 至关重要，请仔细阅读如下内容。**</font>

参数格式如下所示：

<font color='purple'>**type + len + value**</font>

| 参数  | 含义     | 说明                                                         |
| ----- | -------- | ------------------------------------------------------------ |
| type  | 类型     | 表示*value*的类型<br />0：命令<br />1：数据<br />2：延时     |
| len   | 数据个数 | 若 type 为命令时，len 表示后面接多少个数据<br />若 type 为数据时，len 表示数据的长度。注：数据可以为 1byte 也可能为 2byte<br />若 type 为延时，len 无实际意义。为 0 接口。 |
| value | 数据     | 若 type 为命令时，value 表示命令值<br />若 type 为数据时，value 表示数据值。<br />若 type 为延时时，value 表示延时的时长，单位 ms。 |

<font color='red'>注意</font>：由于lcd_init传入的是字节数组，需要进行 bytearray 转换。后续初始化参数，区域写屏参数，亮屏，息屏等参数均是按照该格式进行。

举例说明：

```python
init = (
0,1,0xXX,   #命令，后接一个data, cmd 值为 0xXX
1,2,0xXX,0xXX,  #数据，数据长度为2，data值为0x0100
2,0,120,    #延时 120ms
0,2,0xXX,   #命令，后接 2个data, cmd 值为0xXX
1,1,0xXX,   #数据，命令的第一个data,data值为0xXX
1,2,0xXX,0xFF #数据，命令的第二个data,数据的长度为 2，data值为0xXXFF
…
)
```

###### 初始化参数

该部分比较容易出错且非常重要，请<font color='red'>**重点关注**</font>

该部分参数主要由屏幕厂家提供的源码转换而来，而厂家提供大多数为 C 代码，下面我们将 C 代码转换为 lcd_init 所需要的参数。转换示例如下：

<font color='blue'>小技巧：</font>在VSCode中，使用正则表达式替换原始文本，以达到快速生成lcd_init方法所需的参数。

![1688125456671](../../media/hardware/display/lcd_init_para.png)

###### 区域写屏参数

不同的 LCD屏有不同的设置区域方式，我们将参数设置之后，用户就无需关心底层如何调用了。

一般屏幕设置有两种方式：(以 ili9225 和 st7789v 为例)

一：分两次写：高八位和低八位，此参数根据具体的屏幕去驱动而定(如 st7789v)

二：一次写一个两个字节（如 ili9225）

关于显示区域有几个重要的参数需要提交了解。

```python
XSTART_H = 0xf0     #代表X起始坐标 高八位 （以此值写入，底层会识别该位是x坐标的高位）
XSTART_L = 0xf1  #代表X起始坐标 低八位 （以此值写入，底层会识别该位是x坐标的低位）
YSTART_H = 0xf2  #代表Y起始坐标的高八位
YSTART_L = 0xf3  #代表Y起始坐标的低八位
XEND_H = 0xE0       #代表X结束坐标的高八位
XEND_L = 0xE1       #代表X结束坐标的低八位
YEND_H = 0xE2       #代表Y结束坐标的高八位
YEND_L = 0xE3       #代表Y结束坐标的低八位

XSTART = 0xD0       #代表X的起始坐标
XEND = 0xD1      #代表X的结束坐标
YSTART = 0xD2       #代表Y的起始坐标
YEND = 0xD3      #代表Y的结束坐标
```

<font color='red'>注意</font>：该参数为定位符，用于底层 lcd.lcd_write 接口替换坐标使用

```python
#举例如下：
#ILI9225 – 一次写两个字节
ili9225_invalid = (
0,1,0x36,
1,2,XEND,
0,1,0x37,
1,2,XSTART,
0,1,0x38,
1,2,YEND,
0,1,0x39,
1,2,YSTART,
0,1,0x20,
1,2,XSTART,
0,1,0x21,
1,2,YSTART,
0,1,0x22,
)

#st7789v - 分两次写，一次一个字节:
st7789_invalid = (
0,4,0x2a,
1,1,XSTART_H,
1,1,XSTART_L,
1,1,XEND_H,
1,1,XEND_L,
0,4,0x2b,
1,1,YSTART_H,
1,1,YSTART_L,
1,1,YEND_H,
1,1,YEND_L,
0,0,0x2c,
)
```

原厂提供 C 转 Python 示例如下：

![1688124724127](../../media/hardware/display/lcd_invalid_para.png)

###### 亮屏息屏参数

由于每款 LCD 的亮屏命令不一样，故需要用户提供该配置参数。此参数根据具体的屏幕去驱动而定。

以 ST7789V 为例：

![1688124980736](../../media/hardware/display/lcd_display_para.png)

根据图中内容可以确认该 LCD 的亮屏息屏参数 为 29h 和 28h。退出休眠和进入休眠对应的命令为 11h 和 10h。即可确定亮屏和息屏参数（也可由屏厂提供的示例确定）。

故亮屏息屏参数如下：

```python
lcd_displayON_data = (
0,0,0x11,       #写命令0x11, 后面不接data
2,0,120,         #延时120ms
0,0,0x29,       #写命令0x29, 后面不接data
)
lcd_displayON_data = bytearray(lcd_displayON_data)

lcd_displayOFF_data = (
0,0,0x28,       #写命令0x28, 后面不接data
2,0,120,        #延时120ms
0,0,0x10,       #写命令0x10, 后面不接data
)
lcd_displayOFF_data = bytearray(lcd_displayOFF_data)

```

##### 脚本编写

###### 创建对象

```
from machine import LCD
mipi_lcd = LCD()
```

###### SPI LCD 初始化接口介绍

请参考[LCD 接口 API](../../../api_reference/peripherals/machine.LCD.md#接口1：设备接模块LCD接口) 和 [SPI LCD 接口 API](../../../api_reference/peripherals/machine.LCD.md#接口2：设备接模块SPI接口)。

###### 编写初始化参数

以前期准备中的参数，按照接口介绍中顺序，传入 lcd.lcd_init 中即可。

以 ST7789V 为例：

```python
XSTART_H = 0xf0
XSTART_L = 0xf1
YSTART_H = 0xf2
YSTART_L = 0xf3
XEND_H = 0xE0
XEND_L = 0xE1
YEND_H = 0xE2
YEND_L = 0xE3
XSTART = 0xD0
XEND = 0xD1
YSTART = 0xD2
YEND = 0xD3

init_st7789_240X320=(
0, 0, 0x11,
2, 0, 120,
0, 0, 0x00,
0, 1, 0x36,
1, 1, 0x00,
0, 1, 0x3A,
1, 1, 0x05,
0, 1, 0x35,
1, 1, 0x00,
0, 1, 0xC7,
1, 1, 0x00,
0, 1, 0xCC,
1, 1, 0x09,
0, 5, 0xB2,
1, 1, 0x0C,
1, 1, 0x0C,
1, 1, 0x00,
1, 1, 0x33,
1, 1, 0x33,
0, 1, 0xB7,
1, 1, 0x35,
0, 1, 0xBB,
1, 1, 0x36,
0, 1, 0xC0,
1, 1, 0x2C,
0, 1, 0xC2,
1, 1, 0x01,
0, 1, 0xC3,
1, 1, 0x0D,
0, 1, 0xC4,
1, 1, 0x20,
0, 1, 0xC6,
1, 1, 0x0F,
0, 2, 0xD0,
1, 1, 0xA4,
1, 1, 0xA1,
0, 14, 0xE0,
1, 1, 0xD0,
1, 1, 0x17,
1, 1, 0x19,
1, 1, 0x04,
1, 1, 0x03,
1, 1, 0x04,
1, 1, 0x32,
1, 1, 0x41,
1, 1, 0x43,
1, 1, 0x09,
1, 1, 0x14,
1, 1, 0x12,
1, 1, 0x33,
1, 1, 0x2C,
0, 14, 0xE1,
1, 1, 0xD0,
1, 1, 0x18,
1, 1, 0x17,
1, 1, 0x04,
1, 1, 0x03,
1, 1, 0x04,
1, 1, 0x31,
1, 1, 0x46,
1, 1, 0x43,
1, 1, 0x09,
1, 1, 0x14,
1, 1, 0x13,
1, 1, 0x31,
1, 1, 0x2D,
0, 0, 0x29,
0, 1, 0x36,
1, 1, 0x00,
0, 0, 0x2c,
)
init_st7789_240X320_p = bytearray(init_st7789_240X320)
invalid_st7789_240X320 = (
0,4,0x2a,
1,1,XSTART_H,
1,1,XSTART_L,
1,1,XEND_H,
1,1,XEND_L,
0,4,0x2b,
1,1,YSTART_H,
1,1,YSTART_L,
1,1,YEND_H,
1,1,YEND_L,
0,0,0x2c,
)
invalid_st7789_240X320_p = bytearray(invalid_st7789_240X320)

displayOFF_st7789_240X320 = (
0,0,0x28,
2,0,120,
0,0,0x10,
)
displayOFF_st7789_240X320_p = bytearray(displayOFF_st7789_240X320)

displayON_st7789_240X320 = (
0,0,0x11,
2,0,20,
0,0,0x29,
)
displayON_st7789_240X320_p = bytearray(displayON_st7789_240X320)
from machine import LCD
spilcd = LCD()
spilcd.lcd_init(init_st7789_240X320_p, 240,320,52000,1,4,0,invalid_st7789_240X320_p,displayON_st7789_240X320_p,displayOFF_st7789_240X320_p,None)
```

效果如下：

![1688126689262](../../media/hardware/display/spilcd_show1.png)

###### 区域写屏

该接口 <font color='red'>十分重要</font>，UI 就是基于该接口对屏幕进行 UI 绘制。

[lcd.lcd_write 接口原型](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_write)

注意：

QuecPython 屏幕数据为大端模式。

**示例如下（ST7789V）：**

```python
buf = bytearray(10240) #此时buf值全为0， 颜色体现为黑色
spilcd.lcd_write(buf,110,150,130,170)
```

效果如下：

![1688127026026](../../media/hardware/display/spilcd_show2.png)

###### 清屏

[lcd.lcd_clear 接口原型](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_clear)

**示例如下（ST7789V）：**

```python
spilcd.lcd_clear(0xf800) #0xf800 is represented in red in RGB565
```

效果如下：

![1688127082390](../../media/hardware/display/spilcd_show3.png)

###### 图片显示

[lcd.lcd_show_jpg 原型](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_show_jpg)

<font color='red'>注意 ：</font>该接口并非所有模组都支持，具体支持情况请咨询移远通信技术支持。

**示例如下（ST7789V）：**

```python
spilcd.lcd_show_jpg("usr/background1.jpeg", 0, 0)
```

效果如下：

![1688127171526](../../media/hardware/display/spilcd_show4.png)

#### MIPI LCD

MIPI 协议实际上是一系列接口的协议，包含液晶、摄像头等等。液晶中所用的 MIPI 接口，其专业名称叫 DSI，全称 Display Serial Interface。顾名思义，该接口是指用于显示模块的一个串行接口，基于 MIPI 协议而产生，兼容 DPI(显示像素接口，Display Pixel Interface)、DBI(显示总线接口，Display Bus Interface)和 DCS(显示命令集，Display Command Set)。

本章节将基于 QuecPython 铀 235 开发板驱动 ST7701S（480\*854）屏幕，介绍 MIPI LCD 的调试。

由于 QuecPython 下 LCD 的相关**[API](../../../api_reference/peripherals/machine.LCD.md#lcd.mipi_init)**已经开发好，用户可以直接依照[API](../../../api_reference/peripherals/machine.LCD.md#lcd.mipi_init)格式，编写 MIPI 屏幕驱动。

<font color='red'>**注意：** 目前 QuecPython 支持 mipi 的平台仅支持 RGB565 格式. </font>

<font color='red'>**注意：**</font>实际使用中，LCD 多与 UI([LVGL ](https://lvgl.100ask.net/master/index.html))搭配使用。

##### 前期准备工作

确定使用的 MIPI 屏幕，咨询阅读其 datasheet 和驱动 IC 的技术手册。了解屏幕的特性，确定屏幕的初始化参数（分辨率， Lane 数等）。屏幕的<font color='red'>**初始化参数**</font>可以找屏幕原厂提供。

<font color='red'>**注意：**</font> 原厂提供的初始化参数非常重要，后面 MIPI LCD 初始化时，实际为将该参数替换成 lcd.mipi_init 所需要格式的参数。

![1686981785042](../../media/hardware/display/mipi_lcd_1.png)

从原厂给出的屏幕 datasheet 中可以看出其分辨率（480\*854）、驱动 IC（ST7701S）、接口接口（2-LINE MIPI）、触摸 IC（GT911）和触摸接口（I2C）。

##### 脚本编写

###### 创建 LCD 对象

```python
from machine import LCD
mipi_lcd = LCD()
```

###### MIPI 初始化接口介绍

[lcd.mipi_init 接口原型](../../../api_reference/peripherals/machine.LCD.md#lcd.mipi_init)

```python
lcd.mipi_init(initbuf, **kwargs)
```

| 参数        | 类型      | 说明                                                                                                              |
| ----------- | --------- | ----------------------------------------------------------------------------------------------------------------- |
| initbuf     | bytearray | 必传，传入 MIPI 的配置命令                                                                                        |
| width       | int       | 缺省值：480，屏幕的宽度，单位像素，示例：width=400                                                                |
| hight       | int       | 缺省值：854，屏幕的高度，单位像素，示例：hight=800                                                                |
| bpp         | int       | 缺省值：16，像素深度                                                                                              |
| DataLane    | int       | 缺省值：2，数据通道                                                                                               |
| MipiMode    | int       | 缺省值：0 模式： 0：DSI_VIDEO_MODE 1：DSI_CMD_MODE                                                                |
| PixelFormat | int       | 缺省值：0 像素格式： 0：RGB_PIX_FMT_RGB565 16：RGB_PIX_FMT_RGB888 32：RGB_PIX_FMT_XRGB888 48：RGB_PIX_FMT_RGBX888 |
| DsiFormat   | int       | 缺省值：0 DSI 格式： 0：DSI_FMT_RGB565 1：DSI_FMT_RGB666 2：DSI_FMT_RGB666L 3：DSI_FMT_RGB888                     |
| TransMode   | int       | 缺省值：3 转换模式： 0：DSI_CMD 1：DSI_PULSE 2：DSI_EVENT 3：DSI_BURST                                            |
| RgbOrder    | int       | 缺省值：8 RGB 顺序： 0：RGB 8：BGR                                                                                |
| BllpEnable  | bool      | 缺省值：true，blank low power 模式使能                                                                            |
| HSync       | int       | 缺省值：10，水平同步                                                                                              |
| HBP         | int       | 缺省值：10，水平后肩                                                                                              |
| HFP         | int       | 缺省值：10，水平前肩                                                                                              |
| VSync       | int       | 缺省值：4，垂直同步                                                                                               |
| VBP         | int       | 缺省值：10，垂直后肩                                                                                              |
| VFP         | int       | 缺省值：14，垂直前肩                                                                                              |
| FrameRate   | int       | 缺省值：60，帧率                                                                                                  |
| TESel       | bool      | 缺省值：false，TE 选择                                                                                            |
| RstPolarity | int       | 缺省值：1，reset 极性                                                                                             |

###### 编写初始化参数

该部分比较容易出错且非常重要，请<font color='red'>**重点关注**</font>

参数格式如下所示：

<font color='purple'>**cmd + delay_ms + data_len + data_list**</font>

| 参数      | 含义     | 说明                                       |
| --------- | -------- | ------------------------------------------ |
| cmd       | 命令     | 初始化时命令值                             |
| delay_ms  | 延时     | 延时多久后发送命令，单位 ms                |
| data_len  | 数据长度 | 命令后配置数据搭配的数据个数               |
| data_list | 数据     | 命令后配置的实际数据，个数由 data_len 指定 |

<font color='red'>注意</font>：由于lcd_init传入的是字节数组，需要进行 bytearray 转换。

**示例如下（st7701s）：**

该部分参数主要由屏幕厂家提供的源码转换而来

```python
init_480X854 = (
0x11,0,0,                               #命令为0x11, 延时0ms, 命令所接参数无
0xFF,120,5,0x77,0x01,0x00,0x00,0x10,    #命令为0xFF，延时120ms, 命令对应5个参数数据，分别为          #0x77,0x01,0x00,0x00,0x10
0xC0,0,2,0xE9,0x03,                     #命令为0xC0, 延时0ms, 命令对应2个参数数据，分别为           #0xE9,0x03
0xC1,0,2,0x11,0x02,                     # ... 下面依此类推
0xC2,0,2,0x31,0x08,
0xCC,0,1,0x10,
0xB0,0,16,0x00,0x0D,0x14,0x0D,0x10,0x05,0x02,0x08,0x08,0x1E,0x05,0x13,0x11,0xA3,0x29,0x18,
0xB1,0,16,0x00,0x0C,0x14,0x0C,0x10,0x05,0x03,0x08,0x07,0x20,0x05,0x13,0x11,0xA4,0x29,0x18,
0xFF,0,5,0x77,0x01,0x00,0x00,0x11,
0xB0,0,1,0x6C,
0xB1,0,1,0x43,
0xB2,0,1,0x07,
0xB3,0,1,0x80,
0xB5,0,1,0x47,
0xB7,0,1,0x85,
0xB8,0,1,0x20,
0xB9,0,1,0x10,
0xC1,0,1,0x78,
0xC2,0,1,0x78,
0xD0,0,1,0x88,
0xE0,100,3,0x00,0x00,0x02,
0xE1,0,11,0x08,0x00,0x0A,0x00,0x07,0x00,0x09,0x00,0x00,0x33,0x33,
0xE2,0,13,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
0xE3,0,4,0x00,0x00,0x33,0x33,
0xE4,0,2,0x44,0x44,
0xE5,0,16,0x0E,0x60,0xA0,0xA0,0x10,0x60,0xA0,0xA0,0x0A,0x60,0xA0,0xA0,0x0C,0x60,0xA0,0xA0,
0xE6,0,4,0x00,0x00,0x33,0x33,
0xE7,0,2,0x44,0x44,
0xE8,0,16,0x0D,0x60,0xA0,0xA0,0x0F,0x60,0xA0,0xA0,0x09,0x60,0xA0,0xA0,0x0B,0x60,0xA0,0xA0,
0xEB,0,7,0x02,0x01,0xE4,0xE4,0x44,0x00,0x40,
0xEC,0,2,0x02,0x01,
0xED,0,16,0xAB,0x89,0x76,0x54,0x01,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x10,0x45,0x67,0x98,0xBA,
0xFF,0,5,0x77,0x01,0x00,0x00,0x00,
0x3A,0,1,0x77,
0x36,0,1,0x00,
0x35,0,1,0x00,
0x29,0,0)
from machine import LCD
mipilcd = LCD()
mipilcd.mipi_init(initbuf=bytearray(init_480X854), TransMode=1)
```

效果如下：

![1688106357858](../../media/hardware/display/mipi_lcd_2.png)

<font color='red'>注意 ：</font>该处花屏属于正常现象，显示缓冲区中数据为随机数据，导致屏幕显示随机像素，表现即为花屏。此时清屏一下即可。

###### 区域写屏

该接口 <font color='red'>十分重要 </font>，UI 就是基于该接口对屏幕进行 UI 绘制。

[lcd.lcd_write 接口原型](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_write)

注意：

 QuecPython 屏幕数据为大端模式。

**示例如下（st7701s）：**

```python
test_buf = (
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,0x00,0x1f,
)
test_buf1 = bytearray(test_buf)
mipilcd.lcd_write(test_buf1,10,10,20,20)
```

![1688106994170](../../media/hardware/display/mipi_lcd_3.png)

###### 清屏

[lcd.lcd_clear 接口原型](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_clear)

**示例如下（st7701s）：**

```python
mipilcd.lcd_clear(0xf800) #0xf800 is represented in red in RGB565
```

效果如下：

![1688107261768](../../media/hardware/display/mipi_lcd_4.png)

###### 图片显示

[lcd.lcd_show_jpg 原型](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_show_jpg)

<font color='red'>注意 ：</font>该接口并非所有模组都支持，具体支持情况请参考 WIKI.

**示例如下（st7701s）：**

```python
mipilcd.lcd_show_jpg("usr/background.jpeg", 0, 0)
```

效果如下：

![1688123135485](../../media/hardware/display/mipi_lcd_5.png)

### 常见问题

#### 花屏

初始化显示屏但未写入任何数据，显示屏可能会出现所谓的“花屏”现象，主要原因可能包括：

1. **显示缓冲区的初始状态**：当你初始化显示屏但未写入任何数据时，显示缓冲区可能包含随机数据，这可能导致在屏幕上显示随机的像素，即出现花屏现象。这是因为大多数系统并不会在初始化时自动清除显示缓冲区。
2. **电源和信号问题**：如果显示屏的电源不稳定，或者数据信号存在干扰，也可能导致花屏。这可能需要检查电源和信号线路的质量。
3. **显示驱动程序或硬件问题**：如果显示驱动程序存在问题，或者显示硬件有故障，也可能导致花屏。例如，驱动程序可能未正确初始化显示硬件，或者硬件存在故障。

为了解决这个问题，可以尝试在初始化显示屏后立即清除显示缓冲区，即将其所有像素设置为同一颜色（通常是黑色或白色）。这应该可以消除花屏现象。如果问题仍然存在，可能需要检查你的电源和信号线路，或者检查显示驱动程序和硬件是否存在问题。

#### 刷新率低

LCD 刷新率低可以有多种原因，包括：

1. **硬件性能限制**：内存或数据总线带宽的限制。目前 QuecPython 各个模组对应的最大 SPI 频率为 50Mhz,且大多数屏幕无法适配这么大的频率，导致屏幕较大时，刷新帧率较小。
2. **驱动程序或操作系统问题**：驱动程序或操作系统可能没有正确配置或优化来充分利用硬件性能。例如，驱动程序可能没有正确配置显示控制器，或者操作系统可能没有正确调度图形任务。
3. **显示设置问题**：显示设置，如分辨率，可能过高，超出了硬件或驱动程序的性能能力。试着降低分辨率看是否可以提高刷新率。
4. **显示控制器或面板问题**：显示控制器或 LCD 面板本身也可能有性能限制，不能达到更高的刷新率。
5. **信号干扰或质量问题**：如果使用的是长数据线或信号质量差，可能会导致刷新率降低。尝试更换数据线或改善信号质量。
6. **应用程序问题**：运行的应用程序可能占用了大量的图形资源，导致刷新率降低。尝试关闭不需要的应用程序，或者降低应用程序的图形需求。

#### 屏幕无法点亮

LCD 屏幕无法点亮可能有很多原因，以下列出了一些常见的可能性：

1. **电源问题**：LCD 屏幕需要电源供应，如果电源供应有问题，例如供电电压过高或过低，电源线接触不良，电源适配器故障，都可能导致 LCD 屏幕无法点亮。
2. **接口问题**：LCD 屏幕通常通过某种接口（例如 MIPI SPI 等）连接到驱动电路或者主控板。如果这些接口连接不稳定或者接口线路有问题，可能会导致 LCD 屏幕无法正常工作。驱动 MIPI 时，因为MIPI频率较高，若连接线过长或者有折痕，也可能导致 LCD 无法工作。
3. **驱动问题**：如果驱动程序有问题，或者主控板没有正确配置 LCD 屏幕的参数，也可能导致 LCD 屏幕无法点亮。例如分辨率、色深、帧率等参数设置不正确，或者驱动程序本身有 bug。
4. **硬件故障**：LCD 屏幕自身也可能出现故障，如液晶面板、背光源、驱动 IC 或者其他电子元件损坏，都可能导致 LCD 屏幕无法点亮。
5. **信号问题**：例如信号线损坏，或者信号质量不好（如噪声过大，信号过弱等），也可能导致 LCD 屏幕无法正常显示。
