# LVGL-示例

## 概述

前面章节介绍了LVGL在嵌入式GUI开发上的优势，以及介绍了LVGL的工作流程和常用组件。

本章节将教您如何零基础快速上手使用Quecpython LVGL开发一个GUI应用程序。



## 实验环境搭建

### 硬件准备

**开发板**：LTE OPEN-EVB_V1.1和EC600U-EU_AB TE-A

**LCD屏**：ST7789 240*320

### 固件下载

请前往[Quecpython官网下载中心](https://www.quectel.com.cn/download-zone)下载EC600U-EU_AB的公版固件。

![LVGL_demo1](../../media/multi-media/lvgl/LVGL_demo1.png)

### 硬件连接

将LCD屏接好开发板的LCD接口，连接开发板的USB口，长按PWRKEY开机，如下图所示。

![lvgl_demo2](../../media/multi-media/lvgl/lvgl_demo2.jpg)

### 固件烧录

使用QPYcom工具（[下载地址](https://www.quectel.com.cn/download-zone?cid=8)），进行固件烧录，

![lvgl_demo3](../../media/multi-media/lvgl/lvgl_demo3.jpg)

## 脚本编写

### LCD初始化

LCD的初始化部分参见wiki：[machine LCD](../../../api_reference/peripherals/machine.LCD.md)

本示例使用的是分辨率240*320的ST7789屏。以下为LCD初始化部分代码：

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

### LVGL初始化

```python
import lvgl as lv

lv.init()
```

### 注册显示驱动程序

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

### 开启lvgl线程

按如下代码设置LVGL的心跳以及开启LVGL线程。

```python
lv.tick_inc(5)
lv.task_handler()
```

至此，我们完成了初始化部分，可以进行GUI界面主体代码的编写了。

### 创建界面对象

按照前面章节的描述，通过如下语句创建一个没有父类的LVGL基本对象（即界面对象）。

```
screen = lv.obj()
```

### 创建图片对象

以刚刚创建的LVGL界面为父对象创建一个图片组件，并设置位置及图片源。

```python
# create a image object
img1 = lv.img(screen)
img1.set_pos(80, 80)
img1.set_src("U:/quectel.jpg")
```

### 创建标签对象

以刚刚创建的LVGL界面为父对象创建一个标签组件，设置文本及位置。

```python
# create a label object
label1 = lv.label(screen)
label1.set_text("Hello Quecpython")
label1.center()
label1.set_pos(0, 80)
```

### 加载界面

```python
lv.scr_load(screen)
```

## 效果验证

将脚本下载至模组，运行脚本，可以看到LCD屏显示界面如下图所示。

![lvgl_demo4](../../media/multi-media/lvgl/lvgl_demo4.jpg)