## Displays

Many applications require displaying information to users, making displays an important output device. The QuecPython series modules provide rich peripheral support for screen display applications, including interfaces such as `SPI` and `MIPI`.

A display usually consists of two main parts: the `display panel` and the `driver IC`. The display panel (also known as the display substrate) determines the size, resolution, color, and other parameters of the display. The driver IC (also known as the screen controller) is matched with the display panel and packaged with the panel together. It communicates with the outside through certain interfaces and controls the display panel for display.

### Types

Modern displays come in a wide variety of types, used in various devices and applications, from personal computers to commercial advertising screens, from smart phones to televisions, from gaming devices to embedded devices, etc. Here are some common types of displays:

1. **Liquid Crystal Display (LCD)**:
   - Working principle: LCD displays use liquid crystal molecules to control the passage of light in order to display images. The liquid crystal itself does not emit light, so a backlight source is needed.
   - Features: LCD displays are small in size, low in power consumption, and generate less heat, making them suitable for embedded systems. They can be customized in various sizes and shapes according to specific requirements.
   - Applications: LCD displays are widely used in various embedded devices, such as handheld devices, instruments, home appliances, etc.
2. **Organic Light Emitting Diode (OLED)**:
   - Working principle: OLED displays use the property of organic materials emitting light, by applying voltage to make the organic materials emit light, thereby displaying images. Each pixel can emit light individually.
   - Features: OLED displays have vivid colors, high contrast, and fast response time, and can be manufactured as flexible and transparent displays.
   - Applications: OLED displays are currently mainly used in high-end mobile devices, such as smart phones and wearable devices.
3. **Electronic Paper Display (E-Ink Display)**:
   - Working principle: Electronic paper changes the display color by the microcapsules sandwiched between two transparent electrodes. Each microcapsule contains black and white particles, and applying voltage changes the position of the particles, thus changing the display color.
   - Features: Electronic paper has extremely low power consumption and provides a very good reading experience, especially in lighting conditions, with high clarity.
   - Applications: Electronic paper is mainly used in e-readers and gradually applied to various display tags and devices that require long-term display.
4. **Micro LED**:
   - Working principle: Micro LED displays are composed of countless tiny LED pixels, and each pixel can emit light and adjust brightness individually.
   - Features: Micro LED displays have rich colors, high contrast, fast response time, low power consumption, and long lifespan.
   - Applications: Currently, micro LED is mainly used in high-end large displays and televisions, but with technological advancements, it may gradually be applied to more embedded devices in the future.

These are some common types of displays in embedded systems. The display requirements in embedded systems may vary depending on specific applications and performance requirements, so choosing the appropriate type of display is crucial.

### Driver Block Diagram

The relationship between the MCU/CPU, driver IC, and interface is as follows:

![1687920840462](../../media/hardware/display/lcd_controller.png)

1. **Main Controller (MCU/CPU)**: This is the core part of the embedded system, responsible for processing data, executing code, and managing and controlling various parts of the system.
2. **Display Interface**: This is the communication link between the MCU/CPU and the display chip. Common display interfaces include SPI, I2C, HDMI, VGA, LVDS, MIPI DSI, etc. The MCU/CPU sends data and commands to the display chip through these interfaces.
3. **Display Chip**: The display chip receives data and commands from the MCU/CPU, and then drives the display panel to perform the corresponding display based on this information.
4. **Display Panel (or Display Substrate)**: The display panel is the part of the embedded system that actually displays the image. It contains a series of pixels that change their state according to signals from the display chip, thus displaying the image.

The MCU/CPU sends data and commands to the display chip through the display interface, and then the display chip drives the display panel to display the image according to the data and command information. This is the basic relationship between these components in an embedded system.

### Compatibility Notes

Most screen controllers can support multiple display interfaces. For example, the [ILI9486](https://www.waveshare.net/w/upload/7/78/ILI9486_Datasheet.pdf) supports MCU interface (i.e., 8080 interface), SPI, RGB interface, and MIPI.

QuecPython currently supports the following interfaces:

`SPI`

The SPI features of the QuecPython series modules are:

- The SPI clock frequency can reach up to 50 MHz.

`MIPI`

The MIPI features of the QuecPython series modules are:

- The MIPI clock frequency can reach up to 100 MHz.
- Supports up to 2 lanes.

<font color='red'>**The support status of MIPI and SPI on QuecPython series modules is as follows:**</font>

| SoC        | MIPI | SPI  |
| ---------- | ---- | ---- |
| EC200U_XXX | √    | √    |
| EG912N_XXX | ×    | √    |
| EG912U_XXX | √    | √    |
| EG915N_XXX | ×    | √    |
| EG915U_XXX | √    | √    |

<font color='red'>**The supported screen controller models are shown in the table below:**</font>

| Controller Model | Type  | Interface | Resolution | Driver                                                                                                                                                                                                           | Data Sheet                                                                                                                                                                                                        |
| ---------------- | ------ | --------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ILI9225          | Color  | SPI       | 176\*220   | [ILI9225.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9225) | [ILI9225.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9225) |
| ILI9341          | Color  | SPI       | 240\*320   | [ILI9341.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9341) | [ILI9341.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9341) |
| ILI9806          | Color  | MIPI      | 480\*800   | [ILI9806.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9806) | [ILI9806.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ili9806) |
| ST7735           | Color  | SPI       | 128\*160   | [ST7735.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7735)    | [ST7735.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7735)    |
| ST7789           | Color  | SPI       | 240\*320   | [ST7789.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7789)    | [ST7789.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7789)    |
| ST7701           | Color  | MIPI      | 480\*854   | [ST7701.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7701)    | [ST7701.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/ST7701)    |
| JD9365           | Color  | MIPI      | 800\*1280  | [JD9365.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/JD9365)    | [JD9365.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/JD9365)    |
| SC7705           | Color  | MIPI      | 800\*1280  | [SC7705.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SC7705)    | [SC7705.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SC7705)    |
| SSD1306          | Mono   | SPI       | 128\*64    | [SSD1306.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SSD1306) | [SSD1306.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/SSD1306) |
| UC1628 | Mono | SPI | 128\*64 | [UC1628.py](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/UC1628) | [UC1628.pdf](https://github.com/QuecPython/QuecPython_lib_bundles/tree/master/libraries/LCD/UC1628) |
<font color='red'>**Note**</font>: When you use the LCD method, as long as the parameters are transmitted correctly, it is theoretically possible to support any SPI or MIPI screen.

### Driver Structure

![1686981785042](../../media/hardware/display/controller_structure.png)

**In order to have more control over the screen, QuecPython allows Python scripts to drive the screen through the parameters passed to *lcd.lcd_init*.**

<font color='purple'>**In terms of design, the only difference in driving different screens is the parameters in the scripts *lcd.lcd_init* or *lcd.mipi_init*.**</font>

### Screen Classification

Classifying screens will help us understand the drivers better. Here, screens will be classified based on the colors they can display rather than the panel materials such as OLED or LCD. In general, the colors displayed on the screen determine the BPP (Bits Per Pixel), and different BPP values result in slightly different processing methods in the program. The following lists several ways in which GRAM is mapped to pixels:

![](../../media/hardware/display/screen_driver_RGB565.png)

                  Structure of BPP = 16 GRAM

![](../../media/hardware/display/screen_driver_mono.png)

                  Structure of BPP = 1 GRAM

![](../../media/hardware/display/screen_driver_gray.png)

```
			  Structure of BPP = 4 GRAM
```



From the above figures, it can be seen that the mapping methods can be roughly divided into two categories:

- **BBP >= 8**, usually supports color screens with RGB888, RGB666, RGB565, and other encodings.
- **BPP < 8**, usually monochrome screens, which can be black and white or grayscale.

When BPP < 8, one byte is mapped to multiple pixels, so it is not possible to directly control individual pixels. However, in order to better control monochrome screens, QuecPython still passes RGB565 data to the *lcd_write* interface, and the system automatically converts it. This allows Python scripts to access individual pixels. When BPP >= 8, individual pixels can be easily accessed.

<font color='red'>**Note**</font>: For color screens, the driver only supports RGB565 color encoding.

### Screen Controller Driver

This section will implement display and other functions based on different screen controllers.

#### Display Orientation

The screen display orientation is completely implemented by the screen hardware, and this function may vary on different screen controllers. There are a total of 8 possible display orientations, where the display can be rotated 0°, 90°, 180°, or 270°, and can be viewed from the top or bottom. The default orientation is 0° and top view. These 8 (4 × 2) different display orientations can also be represented as combinations of 3 binary switches: X-mirroring, Y-mirroring, and X/Y swapping.

The following table lists all 8 combinations of display orientations. If the display orientation is not correct, please refer to the configuration switches in the table to make it work correctly.

| <img src="../../media/hardware/display/original.png" style="zoom:50%;" />         | **L2R_U2D**                                      | <img src="../../media/hardware/display/mirror_y.png" style="zoom:50%;" />          | Y-mirroring<br />[L2R_D2U]                                        |
| ------------------------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| <img src="../../media/hardware/display/mirror_x.png" style="zoom:50%;" />         | **X_mirroring**<br />**[R2L_U2D]**               | <img src="../../media/hardware/display/mirror_xy.png" style="zoom:50%;" />         | **X-mirroring**<br />**Y-mirroring**<br />**[R2L_D2U]**           |
| <img src="../../media/hardware/display/swap_xy.png" style="zoom:50%;" />          | **X/Y swapping<br />[U2D_L2R]**                  | <img src="../../media/hardware/display/swap_xy_mirror_y.png" style="zoom:50%;" />  | **X/Y swapping<br />Y-mirroring<br />[D2U_L2R]**                  |
| <img src="../../media/hardware/display/swap_xy_mirror_x.png" style="zoom:50%;" /> | **X/Y swapping<br />X-mirroring<br />[U2D_R2L]** | <img src="../../media/hardware/display/swap_xy_mirror_xy.png" style="zoom:50%;" /> | **X/Y swapping<br />X-mirroring<br />Y-mirroring<br />[D2U_R2L]** |

For different screen controllers, the implementation of screen display orientation is not exactly the same. Generally, it can be divided into the following cases:

> - For color screens, 8 orientations are supported.
> - For monochrome screens, such as SSD1306, only the first 4 orientations are supported, and the XY axis cannot be swapped.

 <font color='red'>**Note:**</font>

The display orientation is also related to the screen panel used. You may encounter two abnormal situations:

- The display orientation is set to L2R_U2D, but the screen does not match the corresponding display orientation in the table. This may be because the wiring on the screen panel has mirrored the X/Y direction. In this case, adjust the rotation according to the actual situation to achieve the desired display orientation.
-  After rotating the screen, the displayed content disappears. This may be because the resolution of the screen panel is smaller than the resolution of the screen controller, causing the displayed area to not completely fall on the screen panel after rotation. In this case, you should consider setting the correct display area offset.

### Common Screen Parameter Analysis

The screen driver IC contains multiple registers that control various functions and settings. Here are some register parameters you may need to pay attention to:

1. Mode registers: These registers typically control the display mode (such as resolution, color depth, etc.) and power management mode. For example, some screen driver ICs may have dedicated registers for switching between sleep mode and normal mode.

2. Control registers: Control registers are used to configure various features of the screen driver IC, such as screen rotation, flipping, inversion, etc.

3. Timing registers: These registers are used to control the timing of pixel clock, horizontal and vertical sync signals.

4. Gamma registers: Gamma registers are used to set the gamma correction of the screen. This is important for image quality and color accuracy.

5. RAM registers: RAM registers are used to store frame buffer data. The driver IC reads data from these registers and sends the data to the screen.

6. Status registers: Status registers allow you to read the current status of the IC, such as whether it is working normally or if any errors have occurred.

7. Brightness/contrast registers: Some driver ICs may have these registers to adjust the brightness or contrast of the screen.

8. Offset registers: These registers are usually used to control the position of the image on the screen.

<font color='red'>Note:</font> Each screen driver IC has its own register mapping and functionality. So when starting to use a new driver IC, you should read its datasheet to understand the details of its registers.

### API Documentation

For detailed API, please refer to [LCD - LCD Driver](../../../api_reference/peripherals/machine.LCD.md).

<font color='red'>**Note:**</font> This section provides information about the QuecPython LCD driver's related APIs. Before driving the screen, please read it carefully.

### Screen Debugging

#### SPI LCD

SPI LCD displays are LCD screens that communicate through SPI. The advantage of this type of display is its simple interface and fewer wires, making the hardware wiring more concise. They are very suitable for microcontrollers and other resource-limited systems.

In SPI LCD, data and commands are sent to the display via the MOSI line, while the SCLK line is used to synchronize data transmission. The CS line is used to select a specific device for communication when communicating with multiple devices.

Normally, the controller (such as a microcontroller) sends appropriate commands and data through SPI based on the specifications of the display (such as resolution and color depth) and the content to be displayed. This may include setting display parameters, writing pixel data, etc.

This section will introduce the debugging of SPI LCD based on the QuecPython EVB driving the [st7789 (240*320)](../../../api_reference/peripherals/machine.LCD.md#Methods) screen.

##### Preparations

Before implementing the LCD driver, we need some preparations. This part introduces the preparations needed for driving a screen. Ii is necessary to know with the LCD interface of QuecPython LCD driver, the parameters required by the interface. For the screen, you should know the ID, the type of the driver IC, and relevant commands.

<font color='red'>**Note:**</font> The initialization parameters provided by the manufacture are very important because they are replaced with the format required by *lcd.lcd_init* when the SPI LCD is initialized.

Understanding the [QuecPython LCD interface](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_init) is essential for better utilizing it to light up the screen.

<font color='red'>**Note:**</font> Currently, QuecPython SPI drivers are divided into two types: LCM (Liquid Crystal Module) and general SPI (Serial Peripheral Interface). The initialization interfaces of the two types are different. For specific details, please refer to the [QuecPython LCD interface](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_init) .

<font color='red'>**Understanding the parameters in *LCD.lcd_init* is crucial for driving LCD with QuecPython. Please read the following content carefully.**</font>

The parameter format is as follows:

<font color='purple'>**type + len + value**</font>

| Parameter | Meaning | Description |
| --------- | ------- | ----------- |
| type | Type | Indicates the type of *value*:<br />0: Command<br />1: Data<br />2: Delay |
| len | Number of data | If *type* is 0 (command), *len* indicates how many data will follow.<br />If *type* is 1 (data), *len* indicates the length of the data. Note: The data can be 1 byte or 2 bytes.<br />If *type* is 2 (delay), *len* has no actual meaning and is set to 0. |
| value | Data | If *type* is 0 (command), *value* indicates the command value.<br />If *type* is 1 (data), *value* indicates the data value.<br />If *type*is 2 (delay), *value* indicates the duration of the delay in milliseconds. |

<font color='red'>Note:</font> Since the parameters of *lcd_init* passed are in bytearray, they need to be converted. Subsequent parameters for initialization, area write screen, screen on/off, etc., are all based on this format.

Here is an example:

```python
init = (
0,1,0xXX,   #Command. The following is one piece of data and the command value is 0xXX
1,2,0xXX,0xXX,  #Data. The data length is 2 and the data value is 0x0100
2,0,120,    #Delay 120 ms
0,2,0xXX,   #Command. The following is two pieces of data and the command value is 0xXX
1,1,0xXX,   #Data. The first data value of the command is 0xXX
1,2,0xXX,0xFF #Data. The length of the second data value is 2 and the value is 0xXXFF
…
)
```

###### Initialization Parameters

This section is crucial and prone to errors, so please <font color='red'>**pay close attention**</font>.

The parameters in this section are primarily derived from the source code provided by the screen manufacturer, which is mostly in C language. Below converts the C code into the parameters required for *lcd_init*. Here is an example of the conversion:

<font color='blue'>**Tip:**</font> In VSCode, use a regular expression to replace the original text to quickly generate the parameters required in *lcd_init*.

![1688125456671](../../media/hardware/display/lcd_init_para.png)

###### Parameters of Writing on Specific Screen Area

Different LCD screens have different ways of configuring the regions. After configuring these parameters, it is unnecessary to concern about the calling details of the low-level.

There are generally two ways to configure the screen regions (taking ili9225 and st7789v as examples):

1. Write in Two Steps: This method involves writing the high eight bits and low eight bits separately. The specific parameters depend on the screen driver (e.g., st7789v).

2. Write 2 bytes at a time (e.g., ili9225).

There are several important parameters related to the display region that need to be understood.

```python
XSTART_H = 0xf0     # Represents the high eight bits of the X starting coordinate (When written with this value, the low-level recognizes it as the high bit of the X coordinate).
XSTART_L = 0xf1     # Represents the low eight bits of the X starting coordinate (When written with this value, the low-level recognizes it as the low bit of the X coordinate).
YSTART_H = 0xf2     # Represents the high eight bits of the Y starting coordinate.
YSTART_L = 0xf3     # Represents the low eight bits of the Y starting coordinate.
XEND_H = 0xE0       # Represents the high eight bits of the X ending coordinate.
XEND_L = 0xE1       # Represents the low eight bits of the X ending coordinate.
YEND_H = 0xE2       # Represents the high eight bits of the Y ending coordinate.
YEND_L = 0xE3       # Represents the low eight bits of the Y ending coordinate.

XSTART = 0xD0       # Represents the starting X coordinate.
XEND = 0xD1         # Represents the ending X coordinate.
YSTART = 0xD2       # Represents the starting Y coordinate.
YEND = 0xD3         # Represents the ending Y coordinate.
```

<font color='red'>Note</font>: This parameter is a locator to replace the local coordinates when you perform partial screen refresh through *lcd.lcd_write* of the low level.

```python
#Example
#ILI9225 – Write 2 bytes at a time
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

#st7789v - Write in two steps. Write 1 byte at a time:
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

The example of C language to Python provided by the manufacture is as follows:

!![1688124724127](../../media/hardware/display/lcd_invalid_para.png)

###### Screen On/Off Parameters

Because the screen-on command varies from each LCD, users need to provide this parameter. This parameter depends on the specific screen being driven.

Take  ST7789V  as an example:

![1688124980736](../../media/hardware/display/lcd_display_para.png)

Based on the information in the diagram, the screen-on and screen-off parameters for this LCD are confirmed to be 29h and 28h respectively. The commands for exiting sleep mode and entering sleep mode are 11h and 10h respectively. This determines the parameters for turning the screen on and off (or can be determined from examples provided by the screen manufacturer).

Thus the parameters for turning the screen on and off are as follows. 

```python
lcd_displayON_data = (
0,0,0x11,       #The write command is 0x11 without any data followed
2,0,120,         #Delay 120 ms
0,0,0x29,       #The write command is 0x29 without any data followed
)
lcd_displayON_data = bytearray(lcd_displayON_data)

lcd_displayOFF_data = (
0,0,0x28,       #The write command is 0x28 without any data followed
2,0,120,        #Delay 120 ms
0,0,0x10,       #The write command is 0x10 without any data followed
)
lcd_displayOFF_data = bytearray(lcd_displayOFF_data)

```

##### Script Writing

###### Creating Objects

```
from machine import LCD
mipi_lcd = LCD()
```

###### Introduction to SPI LCD Initialization Interfaces

Please refer to [LCM Interface](../../../api_reference/peripherals/machine.LCD.md#Interface-1:-The-device-connects-to-LCM-interfaces-of-modules) and [SPI LCD Interface](../../../api_reference/peripherals/machine.LCD.md#Interface-2:-The-device-connects-to-module-SPI) for more information.

###### Writing Initialization Parameters

Pass the parameters from the earlier preparation to *lcd.lcd_init* following the order provided in the interface documentation.

Take ST7789V as an example：

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


The effect is as follows:

![1688126689262](../../media/hardware/display/spilcd_show1.png)

###### Area Write Screen

This interface is <font color='red'>very important</font> as it is used to draw UI on the screen.

[Prototype of *lcd.lcd_write*](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_write)

Note:

The screen data in QuecPython solution is in big-endian mode.

**Example (ST7789V):**

```python
buf = bytearray(10240) # Here the values of buf are all 0, and the color is black
spilcd.lcd_write(buf,110,150,130,170)
```

The screen is displayed as follows:

![1688127026026](../../media/hardware/display/spilcd_show2.png)

###### Clear Screen

[Prototype of *lcd.lcd_clear*](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_clear)

**Example (ST7789V):**

```python
spilcd.lcd_clear(0xf800) #0xf800 is represented in red in RGB565
```

The screen is displayed as follows:

![1688127082390](../../media/hardware/display/spilcd_show3.png)

###### Image Display

[Prototype of *lcd.lcd_show_jpg*](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_show_jpg)

<font color='red'>Note:</font> This interface is not supported by all modules. Please consult Quectel Technical Support for specific supporting status.

**Example (ST7789V):**

```python
spilcd.lcd_show_jpg("usr/background1.jpeg", 0, 0)
```

The screen is displayed as follows:

![1688127171526](../../media/hardware/display/spilcd_show4.png)

#### MIPI LCD

The MIPI protocol is actually a series of interface protocols, including LCD, camera, etc. The MIPI interface used in LCDs is called DSI, which stands for Display Serial Interface. As the name suggests, this interface is a serial interface used for display modules. It is based on the MIPI protocol and is compatible with DPI (Display Pixel Interface), DBI (Display Bus Interface), and DCS (Display Command Set).

This chapter will drive the ST7701S (480*854) screen on the QuecPython U-235 EVB and introduce the debugging of MIPI LCD.

Since the relevant **[API](../../../api_reference/peripherals/machine.LCD.md#lcd.mipi_init)** for LCD under QuecPython has been developed, users can directly write the MIPI screen driver according to the [API](../../../api_reference/peripherals/machine.LCD.md#lcd.mipi_init) format.

<font color='red'>**Note:** Currently QuecPython series modules that support MIPI only support the RGB565 format . </font>

<font color='red'>**Note:**</font> In actual use, LCD is often used in conjunction with UI([LVGL ](https://lvgl.100ask.net/master/index.html)).

##### Preparations

For the MIPI screen to be used, consult and read its datasheet and the technical manual of the driver IC to understand the characteristics of the screen and determine the initialization parameters of the screen (resolution, number of lanes, etc.). The <font color='red'>**initialization parameters**</font> of the screen can be obtained from the screen manufacturer.

<font color='red'>**Note:**</font> The initialization parameters provided by the manufacturer are very important. When initializing the MIPI LCD later, these parameters will be replaced with the parameters in the format that is required by *lcd.mipi_init*.

![1686981785042](../../media/hardware/display/mipi_lcd_1.png)

From the screen datasheet provided by the manufacturer, the screen resolution is 480*854, the driver IC is ST7701S, the interface is 2-LINE MIPI, the touch IC is GT911 and the touch interface is I2C.

##### Script Writing

###### Creating LCD Object

```python
from machine import LCD
mipi_lcd = LCD()
```

###### Introduction to MIPI Initialization Interface

[Prototype of *lcd.mipi_init*](../../../api_reference/peripherals/machine.LCD.md#lcd.mipi_init)

```python
lcd.mipi_init(initbuf, **kwargs)
```

| Parameter   | Type      | Description                                                  |
| ----------- | --------- | ------------------------------------------------------------ |
| initbuf     | bytearray | Required. Write command for passing MIPI.                    |
| width       | int       | Width of LCD screen. Default value: 480. Example: width = 400. |
| hight       | int       | Height of LCD screen. Default value: 854. Example: height = 800. |
| bpp         | int       | Bits per pixel. Default value: 16.                           |
| DataLane    | int       | Data channel. Default value: 2.                              |
| MipiMode    | int       | Mode:<br/>0: DSI_VIDEO_MODE<br/>1: DSI_CMD_MODE<br/>Default value: 0 |
| PixelFormat | int       | Pixel format:<br/>0: RGB_PIX_FMT_RGB565<br/>16: RGB_PIX_FMT_RGB888<br/>32: RGB_PIX_FMT_XRGB888<br/>48: RGB_PIX_FMT_RGBX888<br/>Default value: 0 |
| DsiFormat   | int       | DSI format:<br/>0: DSI_FMT_RGB565<br/>1: DSI_FMT_RGB666<br/>2: DSI_FMT_RGB666L<br/>3: DSI_FMT_RGB888<br/>Default value: 0 |
| TransMode   | int       | Transform mode:<br/>0: DSI_CMD<br/>1: DSI_PULSE<br/>2: DSI_EVENT<br/>3: DSI_BURST<br/>Default value: 3 |
| RgbOrder    | int       | RGB order:<br/>0: RGB<br/>8: BGR<br/>Default value: 8        |
| BllpEnable  | bool      | Enable blank low power. Default value: true.                 |
| HSync       | int       | Horizontal synchronization. Default value: 10.               |
| HBP         | int       | Horizontal back porch. Default value: 10.                    |
| HFP         | int       | Horizontal front porch. Default value: 10.                   |
| VSync       | int       | Vertical Synchronization. Default value: 4.                  |
| VBP         | int       | Vertical back porch. Default value: 10.                      |
| VFP         | int       | Vertical front porch. Default value: 14.                     |
| FrameRate   | int       | Frame rate. Default value: 60.                               |
| TESel       | bool      | TE selection. Default value: false.                          |
| RstPolarity | int       | Reset polarity. Default value: 1.                            |


###### Writing Initialization Parameters

This part is prone to errors and is very important. Please pay <font color='red'>**special attention**</font> to it.

The parameter format is as follows:

<font color='purple'>**cmd + delay_ms + data_len + data_list**</font>

| Parameter | Meaning | Description                                 |
| --------- | ------- | ------------------------------------------- |
| cmd       | Command | Command value during initialization          |
| delay_ms  | Delay   | Delay before sending command. Unit: ms. |
| data_len  | Data Length | Number of data items associated with the command |
| data_list | Data    | Actual data associated with the command, the number is specified by *data_len* |

<font color='red'>Note:</font>  Since the parameters of *lcd_init* passed are in bytearray, they need to be converted. 

**Example (st7701s):**

These parameters are mainly derived from the source code provided by the screen manufacturer.

```python
init_480X854 = (
0x11,0,0,                              # The command is 0x11 with 0 ms delay and no parameter is followed after the command
0xFF,120,5,0x77,0x01,0x00,0x00,0x10,   # The command is 0xFF with 120 ms delay and the command has 5 pieces of data, which are 0x77,0x01,0x00,0x00,0x10
0xC0,0,2,0xE9,0x03,                    # The command is 0xC0 with 0 ms delay and the command has 2 pieces of data, which are 0xE9,0x03
0xC1,0,2,0x11,0x02,                    # ... and so on
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
0xE2,0,13,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
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

The screen is displayed as follows:

![1688106357858](../../media/hardware/display/mipi_lcd_2.png)

<font color='red'>Note:</font> The screen flickering is a normal phenomenon. The data in the display buffer is random data, which causes the screen to display random pixels, resulting in flickering. Clearing the screen will solve this problem.

###### Area Screen Writing

This interface <font color='red'>is very important</font> as it is used to draw UI on the screen.

[Prototype of *lcd.lcd_write*](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_write)

Note:

QuecPython screen data is in big-endian mode.

**Example (st7701s):**

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

###### Clear LCD Screen

[Prototype of *lcd.lcd_clear*](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_clear)

**Example (st7701s):**

```python
mipilcd.lcd_clear(0xf800) #0xf800 is represented in red in RGB565
```

The screen is displayed as follows:

!![1688107261768](../../media/hardware/display/mipi_lcd_4.png)

###### Image Displaying

[Prototype of *lcd.lcd_show_jpg*](../../../api_reference/peripherals/machine.LCD.md#lcd.lcd_show_jpg)

<font color='red'>Note: </font>This interface is not supported by all modules. Please refer to the WIKI for specific supporting status.

**Example (st7701s):**

```python
mipilcd.lcd_show_jpg("usr/background.jpeg", 0, 0)
```

The screen is displayed as follows:

![1688123135485](../../media/hardware/display/mipi_lcd_5.png)

### FAQs

#### Screen Flickering

When initializing the display screen without writing any data, the phenomenon known as 'screen flickering' may occur. The main reasons for this may include:

1. **The initial state of the display buffer**: When you initialize the display but haven't written any data, the display buffer may contain random data, which can result in random pixels being displayed on the screen, also known as screen flickering. This is because most systems do not automatically clear the display buffer during initialization.

2. **Power and signal issues**: If the power supply to the display is unstable or if there is interference in the data signal, it can also cause screen flickering. This may require checking the quality of the power supply and signal lines.
3. **Display driver program or hardware issues**: If there are issues with the display driver program or if the display hardware malfunctions occurs, it can also cause screen flickering. For example, the driver program may not properly initialize the display hardware or there may be hardware malfunctions.

To resolve this issue, you can try clearing the display buffer immediately after initializing the display by setting all pixels to the same color (usually black or white). This should eliminate the screen flickering. If the problem still exists, you may need to check your power supply and signal lines or investigate if there are any issues with the display driver program and hardware.

#### Low Refresh Rate

There can be several reasons for a low refresh rate on an LCD, including:

1. **Hardware performance limitations**: Limitations in memory or data bus bandwidth. Currently, the maximum SPI frequency for QuecPython series modules is 50 MHz, and most screens cannot adapt to such a high frequency, resulting in a lower refresh rate for larger screens.
2. **Driver program or operating system issues**: The driver program or operating system may not be properly configured or optimized to fully utilize the hardware performance. For example, the driver program may not properly configure the display controller, or the operating system may not schedule graphics tasks correctly.
3. **Display settings issues**: Display settings, such as resolution, may be too high and exceed the capabilities of the hardware or driver program. Please try lowering the resolution to see if it improves the refresh rate.
4. **Display controller or panel issues**: The display controller or LCD panel itself may have performance limitations that prevent achieving a higher refresh rate.
5. **Signal interference or quality issues**: If using long data cables or if the signal quality is poor, it can lead to a lower refresh rate.  Please try replacing the data cables or improving the signal quality.
6. **Application issues**: The running application may be consuming a significant amount of graphics resources, resulting in a lower refresh rate.  Please try closing unnecessary applications or reducing the graphics demands of the application.

#### Screen Cannot Light Up

There can be various reasons why an LCD screen cannot light up. Here are some common possibilities:

1. **Power issues**: LCD screens require a power supply, and if there are issues with the power supply, such as excessive or insufficient voltage, poor contact with the power line, or a faulty power adapter, it can prevent the LCD screen from lighting up.
2. **Interface issues**: LCD screens are typically connected to the driver circuit or the main control board through some interface (e.g., MIPI, SPI). If these interfaces have unstable connections or if there are issues with the interface lines, the LCD screen may not work properly. When driving MIPI (due to hugh MIPI frequency), the LCD screen may not work properly if the connection line is too long or has folds.
3. **Driver issues**: If there are issues with the driver program or if the main control board is not properly configured for the LCD screen parameters, the LCD screen may not be able to be lit up, such as incorrect settings for resolution, color depth, refresh rate, or bugs in the driver program.
4. **Hardware failure**: The LCD screen itself may have hardware failures, such as damages on LCD panel, backlight, driver IC, or other electronic components. The LCD screen may fail to be lit up due to these issues.
5. **Signal issues**: Signal issues such as damaged signal lines or poor signal quality (e.g., excessive noise, weak signal) can also prevent the LCD screen from displaying properly.
