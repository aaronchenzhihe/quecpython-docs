# LVGL-UI工具

## 简介

`Gui Guider`（恩智浦）是PC端GUI设计工具，可以通过拖放控件方式设计LVGL GUI页面，加速GUI设计。设计完成后可在PC上仿真运行，确认设计完毕后生成python脚本，再整合入模组中。完成开发。

[`Gui Guider`下载地址](https://www.nxp.com/products/processors-and-microcontrollers/s32-automotive-platform/s32k-auto-general-purpose-mcus/gui-guider:GUI-GUIDER)。

本文将介绍如何使用GUI Guider辅助进行Quecpython LVGL开发。

GUI Guider的主要特征

- 支持Windows 10和Ubuntu 20.04。
- 支持中文、英文。
- 兼容LVGL V8.3.5版本。
- 支持拖放的所见即所得(WYSIWYG)用户界面设计。
- 多种字体支持及第三方字体导入。
- 自动产生python语言源代码。
- 支持默认样式和自定义样式。
- 演示应用程序集成。
- 实时日志显示。
- 集成上位机仿真器。 

## 使用说明

正确安装好`Gui Guider`就可以开始设计GUI了，设计好模拟完毕后即可一键生成python代码，经过少量修改后就可以下载到模块中运行了。下面是具体的使用步骤。

### 创建工程

打开`Gui Guider`，在下面界面选择Create a new project。

![guiguider1](../../media/multi-media/lvgl/guiguider1.png)

进入下面界面后选择v8.3.5，点击next进入下一步。

![guiguider2](../../media/multi-media/lvgl/guiguider2.png)

在下面界面选择simulator模拟器。

![guiguider3](../../media/multi-media/lvgl/guiguider3.png)

在下面界面选择一个模板，您可根据需要选择，如下有播放器，标签，仪表盘等，选择完点击next进入下一步。

![guiguider4](../../media/multi-media/lvgl/guiguider4.png)

在下面界面的右侧填写项目名，选择保存路径，色深选择16位，根据项目LCD的实际分辨率选择分辨率，填写完成点击Create创建工程。

![guiguider5](../../media/multi-media/lvgl/guiguider5.png)

### GUI设计

创建完项目后进入UI设计界面，下面为几个主要的区域的介绍。

- 工具界面左侧Layers下的Screen为当前创建的UI界面列表，可以通过点击“+”按钮来创建新界面。
- 工具界面左侧Layers下的Widgets为当前编辑UI界面的组件列表。
- 工具界面中间的UI editor左侧为可以拖拽的组件库，点击下面的扩展可以展开所有的组件。
- 工具界面中间的UI editor主要部分为当前正在编辑的UI界面。
- 工具界面右侧的Attributes可以设置UI界面或组件的属性值。

![guiguider6](../../media/multi-media/lvgl/guiguider6.png)

请按照如下步骤完成UI的设计：

创建所有需要的界面，命名最好规范，方便后面的代码生成及维护；

设计每个界面，将所需的组件拖拽到界面上，调整位置和大小，设置属性；

![guiguider7](../../media/multi-media/lvgl/guiguider7.png)

如需给组件添加事件，比如点击切换界面，则先选中需要添加的组件，再在任意地方鼠标右击，点击Add Event按钮点开添加事件窗口，如下图所示。

![image-20230809115014573](../../media/multi-media/lvgl/guiguider10.png)

![image-20230809115139952](../../media/multi-media/lvgl/guiguider11.png)

UI设计完成后，点击右上角的三角形图标，选择Micropython模拟器点击启动，在模拟器上查看效果和验证事件。

![guiguider8](../../media/multi-media/lvgl/guiguider8.png)

![guiguider9](../../media/multi-media/lvgl/guiguider9.png)

### 代码生成

完成GUI的设计和模拟器模拟后，点击右上角的按钮生成代码。

![guiguider12](../../media/multi-media/lvgl/guiguider12.png)

点击中间视图的Code Viewer可以预览生成的代码，生成的python代码名为gui_guider.py如下。代码路径为工程下的generated目录下。

![image-20230809141858481](../../media/multi-media/lvgl/guiguider13.png)



## 适配Quecpython

GUI guider生成的脚本与实际应用的脚本还有些轻微差异。稍微修改后即可直接运行于模组中。主要实际操作如下：

1. 添加对LCD的初始化。
2. 修改gui guider生成的代码。
3. 下载脚本到模组运行。

### 添加LCD的初始化

在gui guider生成的py文件开始添加LCD显示屏初始化代码，参照[wiki](../../../api_reference/peripherals/machine.LCD.md)。

### 修改生成的代码

gui guider生成的py文件还需进行一些修改才能适配Quecpython LVGL。

### 修改对LVGL的初始化

将生成文件的LVGL初始化代码，参照[LVGL流程](https://gitee.com/qpy-doc-center/teedoc_with_qpydoc/tree/main/docs/Application_guide/zh/multi-media/lvgl/lvgl_flow.md)的LVGL初始化章节进行修改。

### 修改对图片的使用

生成的文件中，对图片的使用采用的的是其自定义的解码接口，而Quecpython LVGL自带图片解码器，故可直接使用路径调用。

把gui guider生成的py文件中所有的set_src接口的参数修改成图片的路径，示例如下。

```python
img_obj.set_src("U:/demo.png")
```

> **注意：**
>
> gui guider生成的代码中，每个组件都对应生成了一个style，这使得代码非常的臃肿，所以如需进阶开发，还得对其做进一步优化，比如减少不必要的style定义，比如去除不必要的try语句等。





