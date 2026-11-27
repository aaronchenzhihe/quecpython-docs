# LVGL - Demo

## Overview

The previous chapters introduced the advantages of LVGL in embedded GUI development, as well as the workflow and common widgets of LVGL.

This chapter introduces how to quickly get started with Quecpython LVGL and develop a GUI application with no experience.

## Set Up Environment

### Hardware Preparation

**EVB**: LTE OPEN-EVB_V1.1 and EC600U-EU_AB TE-A

**LCD**: ST7789 240*320

### Download Firmware

Please go to [QuecPython-Download](https://developer.quectel.com/en/resource-download?cid=4) to download the official firmware for EC600U-EU_AB.

![LVGL_demo1](../../media/multi-media/lvgl/LVGL_demo1.png)

### Hardware Connection

Connect the LCD to the LCD interface of the EVB, and connect the EVB to the computer via a USB cable. Long press the "**PWRKEY**" to power on the EVB, as shown in the following figure.

![lvgl_demo2](../../media/multi-media/lvgl/lvgl_demo2.jpg)

### Flash Firmware

Flash the firmware by using QPYcom (Click [here](https://developer.quectel.com/en/resource-download?cid=260) to download QPYcom).

![lvgl_demo3](../../media/multi-media/lvgl/lvgl_demo3.jpg)

## Write Script

Click [here]() to download the application code.

### Initialize LCD

For the initialization of LCD, see [machine LCD](../../../api_reference/peripherals/machine.LCD.md).

Here takes an ST7789 LCD with a resolution of 240*320 as an example. The following is the code for LCD initialization:

```python
from machine import LCD

init_param=(
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
init_data = bytearray(init_param)

invalid_param = (
0,4,0x2a,
1,1,0xf0,
1,1,0xf1,
1,1,0xe0,
1,1,0xe1,
0,4,0x2b,
1,1,0xf2,
1,1,0xf3,
1,1,0xe2,
1,1,0xe3,
0,0,0x2c,
)
test_invalid = bytearray(invalid_param)
displayoff = (
0,0,0x28,
2,0,120,
0,0,0x10,
)
test_displayoff = bytearray(displayoff)
displayon = (
0,0,0x11,
2,0,20,
0,0,0x29,
)
test_displayon = bytearray(displayon)

lcd.lcd_init(init_data, 240, 320, 52000, 1, 4, 0, test_invalid, test_displayon, test_displayoff, None)
```

### Initialize LVGL

```python
import lvgl as lv

lv.init()
```

### Register Display Driver

```python
LCD_SIZE_W = 240
LCD_SIZE_H = 320

disp_buf1 = lv.disp_draw_buf_t()
buf1_1 = bytearray(LCD_SIZE_W * LCD_SIZE_H * 2)
disp_buf1.init(buf1_1, None, len(buf1_1))
disp_drv = lv.disp_drv_t()
disp_drv.init()
disp_drv.draw_buf = disp_buf1
disp_drv.flush_cb = lcd.lcd_write
disp_drv.hor_res = LCD_SIZE_W
disp_drv.ver_res = LCD_SIZE_H
disp_drv.register()
```

### Start LVGL Thread

Set the heartbeat of LVGL and start the LVGL thread.

```python
lv.tick_inc(5)
lv.task_handler()
```

At this point, we have completed the initialization and can start writing the main code for the GUI interface.

### Create Interface

Create a basic LVGL (i.e., interface) with no parent class.

```
screen = lv.obj()
```

### Create Image

Create an image widget with the LVGL interface created earlier as the parent, and set the position and image source.

```python
# create an image
img1 = lv.img(screen)
img1.set_pos(80, 80)
img1.set_src("U:/quectel.jpg")
```

### Create Label

Create a label widget with the LVGL interface created earlier as the parent, and set the text and position.
```python
# create an label
label1 = lv.label(screen)
label1.set_text("Hello Quecpython")
label1.center()
label1.set_pos(0, 80)
```


### Load Screen

```python
lv.scr_load(screen)
```

## Verification

Download the script to the module and run it. You will see the display interface of the LCD, as shown in the following figure.

![lvgl_demo4](../../media/multi-media/lvgl/lvgl_demo4.jpg)