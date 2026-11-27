# LVGL-流程

## 概述

在当今嵌入式系统开发领域，图形化用户界面（GUI）的需求越发突显，Quecpython + LVGL将为您的GUI开发助力。

本章节从其与硬件设备的关联到界面的绘制和显示过程，为您直观呈现一个完整的LVGL工作流程。

## LVGL与显示设备关联

LVGL通过注册的LCD显示驱动对象，将绘制的图形数据刷新到LCD屏幕上。LCD作为显示设备，负责在屏幕上显示图形界面的内容。LCD显示驱动对象充当了LVGL与LCD之间的桥梁，它接收LVGL生成的图形数据，并将其有效地传输到LCD屏幕上进行显示。通过这种方式，LVGL库与LCD之间建立了紧密的联系，使开发者能够借助LVGL的功能和灵活性，轻松创建出美观、交互性强的嵌入式用户界面，并通过LCD屏幕进行实时显示和更新。

![lvgl_1](../../media/multi-media/lvgl/lvgl_1.png)

## LVGL工作流程

### LVGL初始化

#### 流程描述 - 初始化

LVGL初始化流程如下：

1. 正确初始化LCD显示屏，参照[class LCD - LCD显示屏](../../../api_reference/peripherals/machine.LCD.md)。
2. 初始化显示负载。
3. 初始化显示驱动，指定刷新接口为LCD对象的写屏接口
4. 如果有TP等输入设备，初始化TP对象。
5. 初始化输入设备驱动。
6. 开启LVGL线程。

下图为LVGL初始化流程图：

![lvgl_2](../../media/multi-media/lvgl/LVGL_initialization_flowchart.png)

#### 代码示例 - 初始化

以下为包含Touchpad(触摸板)的LVGL的初始化代码，该示例使用MIPI显示屏。

```python
import lvgl as lv
from machine import LCD, Pin
from tp import gt9xx

# 此处MIPI初始化参数请根据具体屏幕驱动型号填写。
init_para = (0x29,0,0)
# LCD初始化部分，详见代码下方注意事项。
mipilcd = LCD()
mipilcd.mipi_init(initbuf=bytearray(init_para), width=480, hight=854)

# LVGL初始化
lv.init()
# 创建一个显示缓冲区对象
disp_buf1 = lv.disp_draw_buf_t()
# 显示缓冲区对象大小为：width * height * 2
buf1_1 = bytes(480 * 854 * 2)
disp_buf1.init(buf1_1, None, len(buf1_1))

# 创建LVGL显示驱动对象
disp_drv = lv.disp_drv_t()
# 初始化LVGL显示驱动对象
disp_drv.init()
# 将显示缓冲区对象赋值给驱动对象的draw_buf属性
disp_drv.draw_buf = disp_buf1
# 将LCD对象的刷新回调函数lcd_write赋值给驱动对象的flush_cb属性
disp_drv.flush_cb = mipilcd.lcd_write
# 此处基于实际的屏幕宽度来设置水平分辨率
disp_drv.hor_res = 480
# 此处基于实际的屏幕高度来设置垂直分辨率
disp_drv.ver_res = 854
# 此处设置是否需要旋转
disp_drv.sw_rotate = 1
# 旋转角度
disp_drv.rotated = lv.DISP_ROT._270
# 注册LVGL显示驱动对象
disp_drv.register()

# Touchpad(触摸板)初始化
tp_gt911 = gt9xx(irq=40, reset=20)
tp_gt911.activate()
tp_gt911.init()
print("gt911 init...")

# 创建LVGL输入设备驱动对象
indev_drv = lv.indev_drv_t()
indev_drv.init()
indev_drv.type = lv.INDEV_TYPE.POINTER
# 将Touchpad对象的读取函数read赋值给LVGL输入设备驱动对象的read_cb属性
indev_drv.read_cb = tp_gt911.read
indev_drv.register()

# 启动LVGL 线程
lv.tick_inc(5)
lv.task_handler()
```

> **注意：**
>
> - LCD初始化部分参照[class LCD - LCD显示屏](../../../api_reference/peripherals/machine.LCD.md)。
>
> - LVGL需要LCD对象提供lcd_write接口来刷新屏幕。
> - LVGL需要TP对象提供read接口来进行触摸输入。

### LVGL界面绘制

#### 流程描述 - 界面绘制

LVGL绘制流程如下：

1. 创建LVGL组件对象。
2. 如有需要自定义样式，则创建LVGL样式对象。
3. 给LVGL组件对象添加样式对象。
4. 如有需要自定义事件，则创建LVGL事件回调函数。
5. 给LVGL组件绑定事件回调函数，并指定触发事件类型。
6. 加载界面，开始GUI显示逻辑。

下图为LVGL绘制流程图：

![LVGL drawing flowchart](../../media/multi-media/lvgl/LVGL_drawing_flowchart.png)

#### 代码示例 - 界面绘制

以下为一段示例代码，该示例创建了一个界面，并在界面上创建了一个可以点击的按钮。

```python
import lvgl as lv

#此处省略LVGL初始化流程代码

#----------script start----------

# 创建一个界面
screen = lv.obj()

# 创建button对象
btn1 = lv.btn(screen)
# button设置位置
btn1.center()
# button添加文字
label = lv.label(btn1)
label.set_text("click")

# 创建样式对象
style_btn = lv.style_t()
style_btn.init()
# 设置背景颜色
style_btn.set_bg_color(lv.palette_main(lv.PALETTE.YELLOW))
# 设置字体颜色
style_btn.set_text_color(lv.palette_darken(lv.PALETTE.YELLOW, 4))

# 给button对象添加样式
btn1.add_style(style_btn, 0)

# 定义事件回调函数
def event_handler(evt):
    code = evt.get_code()
    if code == lv.EVENT.CLICKED:
        print("Clicked event detected")

# 给button对象添加事件，在点击时触发
btn1.add_event_cb(event_handler, lv.EVENT.CLICKED, None)

# 加载界面
lv.scr_load(screen)
```

> **注意：**
>
> - LVGL对象创建需在LVGL线程开始之后，否则会出错。
> - LVGL绘制流程可以与显示交织进行，或者说可以在显示过程中动态绘制并实时显示。

### 效果展示

上述实例代码运行效果如下图所示：

![lvgl_btn_demo](../../media/multi-media/lvgl/lvgl_btn_demo.png)

## LVGL界面显示

LVGL以界面(obj对象)为基本单位进行显示，开发者根据业务逻辑调用界面加载接口进行界面的切换。

### 屏幕刷新机制

LVGL采用以下几种刷新机制并存的方式进行屏幕刷新，开发者了解即可。

1. 自动刷新(Automatic Refresh)

这是LVGL的默认刷新模式。在这个模式下，LVGL会自动检测对象的状态变化并刷新显示。开发者不需要手动刷新。

2. 部分刷新(Partial Refresh)

当屏幕的局部区域发生变化时，默认只刷新该区域而不是全屏刷新，该机制可以提高性能。