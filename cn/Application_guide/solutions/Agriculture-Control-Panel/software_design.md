# 软件设计讲解



## 软件框架

### 软件设计图

<img src="../../media/solutions/Agriculture-Control-Panel/software.png" alt="image-20250205105239936" style="zoom:80%;display: block; margin: 0 auto;" />	

### 业务系统启动流程

<img src="../../media/solutions/Agriculture-Control-Panel/start.png" alt="image-20250205112309962" style="zoom:100%;display: block; margin: 0 auto;" />	





## 代码讲解



### 事件管理

各个模块之间通过 `EventMeth 事件管理类进行协作。`EventMeth` 事件管理类提供事件的发布、订阅以及消息的同步、异步发送。其框架如图：

![](../../media/solutions/Agriculture-Control-Panel/event-mesh.png)	

`EventMeth` 事件管理类定义如下：

```python
class EventStore(object):
def __init__(self):
        self.map = dict()
        self.log = None

    def append(self, event, cb):
        self.map[event] = cb

    def fire_async(self, event, msg):
        if event in self.map:
            _thread.start_new_thread(self.map[event], (event, msg))
            if self.log:
                self.log.info("ASYNC executed (event) -> {} (params) -> {} (result) -> {}".format(event, msg, None))

    def fire_sync(self, event, msg):
        res = None
        try:
            if event in self.map:
                res = self.map[event](event, msg)
        except Exception as e:
            usys.print_exception(e)
        if self.log:
            self.log.info("SYNC executed (event) -> {} (params) -> {} (result) -> {}".format(event, msg, res))
        return res
    
event_store = EventStore()

def subscribe(event, cb):
    """
    subscribe event and cb
    """
    return event_store.append(event, cb)

def publish(event, msg=None):
    """
    publish event and msg
    """
    return publish_sync(event, msg)

def publish_async(event, msg=None):
    """
    异步发送
    """
    return event_store.fire_async(event, msg)

def publish_sync(event, msg=None):
    """
    同步发送
    """
    return event_store.fire_sync(event, msg)
```

1. EventMesh 是一个基于订阅(subscribe)和发布(publish)机制的应用框架，用户可以订阅任何感兴趣的主题，该主题即表示一个事件。
2. 在订阅主题的同时，还需要注册一个回调(event_cb)函数，用于处理产生的事件。
3. 用户可在任何产生事件的地方发布该事件，并可以携带消息内容。此时订阅者注册的回调函数会被调用，事件得到处理。



### 界面切换实现原理

在每一个带有控件触摸事件产生的屏幕界面脚本代码中，都会调用控件的 `add_event_cb()` 方法为对应的事件注册一个回调函数，切换界面实现相对简单，当lvgl线程检测到触摸事件产生时，会通过控件对应的回调函数向`EventMeth`发布需要进行的操作。

实例代码：

```python
class MainScreen(Screen):
	# Some code is omitted here
	def create(self):
		# Some code is omitted here
		self.weather_btn.add_event_cb(lambda e: self.__weather_btn_event_cb(e), lv.EVENT.PRESSED, None)
        self.alarm_btn.add_event_cb(lambda e: self.__alarm_btn_event_cb(e), lv.EVENT.PRESSED, None)
        self.monitor_btn.add_event_cb(lambda e: self.__monitor_btn_event_cb(e), lv.EVENT.PRESSED, None)
        self.monitor_btn_2.add_event_cb(lambda e: self.__monitor_btn_event_cb(e), lv.EVENT.PRESSED, None)
        self.dev_btn.add_event_cb(lambda e: self.__dev_btn_event_cb(e), lv.EVENT.PRESSED, None)
        self.setting_btn.add_event_cb(lambda e: self.__setting_btn_event_cb(e), lv.EVENT.PRESSED, None)
        
    def __weather_btn_event_cb(self,e):
        src = e.get_target()
        code = e.get_code()
        print("The click event occurs")
        EventMesh.publish("load_screen", "WeatherScreen")

    def __monitor_btn_event_cb(self,e):
        src = e.get_target()
        code = e.get_code()
        print("The click event occurs")
        EventMesh.publish("load_screen", "MonitorScreen")

    def __alarm_btn_event_cb(self,e):
        src = e.get_target()
        code = e.get_code()
        print("The click event occurs")
        EventMesh.publish("load_screen", "AlarmScreen")

    def __dev_btn_event_cb(self,e):
        src = e.get_target()
        code = e.get_code()
        print("The click event occurs")
        EventMesh.publish("load_screen", "Dev1Screen")

    def __setting_btn_event_cb(self,e):
        src = e.get_target()
        code = e.get_code()
        print("The click event occurs")
        EventMesh.publish("load_screen", "SettingScreen")
```

以上述实例代码中的切换天气界面为例：

```Python
self.weather_btn.add_event_cb(lambda e: self.__weather_btn_event_cb(e), lv.EVENT.PRESSED, None)
```

为天气控件绑定回调函数`__weather_btn_event_cb()`

```Python
    def __weather_btn_event_cb(self,e):
        src = e.get_target()
        code = e.get_code()
        print("The click event occurs")
        EventMesh.publish("load_screen", "WeatherScreen")
```

回调函数中执行`EventMesh.publish("load_screen", "WeatherScreen")`，发布`load_screen`事件给`EventMeth`，意思为加载界面，携带消息`WeatherScreen`，表示加载`WeatherScreen`界面。



### 界面模块

`screen.py`中定义了一个`Screen`类作为所有界面的基类，其定义如下：

```Python
class Screen(object):
    """Abstract class of screen"""

    def __init__(self):
        super().__init__()
        self.style_siyuan_14_red = None
        self.style_siyuan_16_blue = None
        self.style_opa_btn = None
        self.style_siyuan_26 = None
        # Some code is omitted here
    def create_style(self):
        """
        create style
        """
        # Interface style, background color: 3a5283
        style_screen = lv.style_t()
        style_screen.init()
        style_screen.set_bg_color(lv.color_make(0x3a, 0x52, 0x83))
        style_screen.set_bg_opa(255)
        self.style_screen = style_screen

        # The style of the left menu bar
        style_cont_menu = lv.style_t()
        style_cont_menu.init()
        style_cont_menu.set_radius(0)
        style_cont_menu.set_bg_color(lv.color_make(0x28, 0x39, 0x5c))
        style_cont_menu.set_bg_grad_color(lv.color_make(0x28, 0x39, 0x5c))
        style_cont_menu.set_bg_grad_dir(lv.GRAD_DIR.VER)
        style_cont_menu.set_bg_opa(255)
        style_cont_menu.set_border_color(lv.color_make(0x21, 0x95, 0xf6))
        style_cont_menu.set_border_width(0)
        style_cont_menu.set_border_opa(0)
        style_cont_menu.set_pad_left(0)
        style_cont_menu.set_pad_right(0)
        style_cont_menu.set_pad_top(0)
        style_cont_menu.set_pad_bottom(0)
        self.style_cont_menu = style_cont_menu
        # Some code is omitted here
```

`Screen`类中提供所有屏幕对象都需要的公共方法，如创建屏幕、状态栏信息显示（运营商名称、信号强度、系统时间、电池电量等）。QuecPython 使用 NXP 公司的 [GUI Guider](https://www.nxp.com.cn/design/design-center/software/development-software/gui-guider:GUI-GUIDER) 作为图形化界面设计工具，该部分样式代码由该工具生成。

所有以 `_screen.py` 结尾的脚本文件，均实现了一个继承于 `class Screen()` 的类，是对应屏幕需要显示的画面的代码实现。界面代码的主体内容是由 [GUI Guider](https://www.nxp.com.cn/design/design-center/software/development-software/gui-guider:GUI-GUIDER) 工具自动生成的，而后进行少许修改，主要是对象初始化方法 `__init__()` 和控件事件处理回调函数（`__xxx_event_cb()`）的修改。



### GUI启动

`"load_screen"` 事件在 `Agri_ui.py` 脚本文件的 `class AgriUi(object)` 类的 `start()` 方法中被订阅。

代码如下：

```python
class AgriUi(object):

    # Some code is omitted here

    def __create(self):
        for screen in self.screens:
            screen.create()

    def start(self):
        self.__create()
        EventMesh.subscribe("load_screen", self.__route)
        EventMesh.publish("load_screen", "WelcomeScreen")
        utime.sleep(2)
        EventMesh.publish("load_screen", "MainScreen")

    def __route(self, topic, meta):
        for screen in self.screens:
            if screen.name == meta:
                lv.scr_load(screen.screen)
```

上述代码中的 `start()` 方法用于启动智慧农业中控面板的图形化界面，其工作流程如下：

- 调用 `__create()` 方法创建每个屏幕界面的元数据。

- 调用 `EventMesh.subscribe()` 方法订阅名为 `"load_screen"` 的事件，当该事件产生时，调用 `__route()` 方法处理该事件。

  > `__route()` 方法根据发布事件时携带的消息，即屏幕界面的名称，去匹配相应的界面对象，调用 lvgl 的 `lv.scr_load()` 方法加载出新的界面。
  >
  > 细心的读者会发现，每一个屏幕界面脚本文件中实现的类中，其对象初始化方法 `__init__()` 中，都会有类似 `self.name = "MainScreen"` 的语句。该语句便记录了屏幕界面的名称。

- `EventMesh.publish("load_screen", "WelcomeScreen")` 语句用来触发第一个界面显示，即欢迎界面。而后等让欢迎界面的显示持续 2 秒钟，营造一种良好的视觉体验。

- 欢迎界面结束后，通过`EventMesh.publish("load_screen", "MainScreen")` 语句用来触发主界面的显示。

至此，智慧农业中控面板的图形化界面启动完毕，后续界面的切换由用户触摸控制。

总结，`Agri_ui.py` 执行智慧农业中控面板的所有初始化工作，包括 lvgl 初始化和 MIPI LCD 初始化；同时，提供了图形化界面的屏幕对象添加器和 GUI 启动器。屏幕对象添加器用于添加应用中所有可能需要显示的屏幕对象，GUI 启动器用于启动图形化界面功能。



## 图形化界面设计

上文提到，QuecPython 使用 NXP 公司的 [GUI Guider](https://www.nxp.com.cn/design/design-center/software/development-software/gui-guider:GUI-GUIDER) 作为图形化界面设计工具，该工具不仅能够进行界面布局设计，还能自动生成 QuecPython 代码。[点此查看 GUI Guider 工具的使用教程](../../multi-media/lvgl/lvgl_ui_tool.md)。

下文以智慧农业中控面板的首页为例，来介绍图形化界面的设计过程。

### 布局和背景设置

创建一个智慧农业工程，选择一个适合的布局摸板和背景设计。 该阶段将从空白布局开始设计，即选择空白摸板，设置分辨率为 854*480。

​	![](../../media/solutions/Agriculture-Control-Panel/UI_Creat_Project.png)



### 界面绘制

该部分将开始绘制主页中的各个模块。

1.**左侧导航栏**：

![](../../media/solutions/Agriculture-Control-Panel/UI_Shortcut_Bar.png)

- **组件**：imgbtn。
- **图标**：使用易于识别的图标作为快捷方式，访问主页、显示数据、设置、控制等功能。
- **布局**：图标垂直对齐，放置在界面左侧，便于使用。
- **交互性**：为图标添加选中效果，提升交互体验。

在GUI Guider软件中，使用 `imgbtn` 组件来创建智慧农业图形化界面中的左侧快捷键栏时，您会需要关注几个关键设计要素：

1. 图标选择：
   - 每个 `imgbtn` 应选择一个直观的图标，代表它的功能，如家庭形状的图标代表主页，齿轮形状代表设置。
   - 图标设计应简洁明了，以便用户一眼就能识别其功能。
2. 大小和间距：
   - 每个 `imgbtn` 的大小应足够大，用户可以轻松点击，特别是在触摸屏设备上。
   - 快捷键之间应有适当间距，以防止误触，并使界面看起来更加整洁。
3. 图标状态：
   - 设计不同状态的图标，如默认状态、悬浮状态、点击状态，这样可以提供视觉反馈，增强用户交互体验。
   - `imgbtn` 组件应支持状态切换，使得用户可以看到他们的交云动作导致了界面的响应。
4. 功能性：
   - 每个 `imgbtn` 都应配置正确的事件处理器，以确保当用户点击时能够触发正确的动作或导航到相应的界面。
   - 如果 `imgbtn` 代表的功能当前已激活或位于当前页面，它应以不同的视觉样式显示，以指示其活动状态。

2.**顶端信息显示栏**：

![](../../media/solutions/Agriculture-Control-Panel/UI_Information_Display_Bar.png)

- **组件**：label、image。
- **运营商信息**：在界面顶部左侧显示移动运营商或网络状态。
- **时间显示**：在界面顶部中间位置显示当前时间。
- **信号强度和电量指示**：在界面顶部右侧显示Wi-Fi或网络信号强度以及当前电量的图标。

在使用 GUI Guider 软件设计智慧农业中控面板的顶端信息显示栏时，结合 `label` 和 `image` 组件来创建一个信息丰富且视觉上易于识别的状态栏。以下是如何使用这些组件来设计顶端信息显示栏的详细步骤：

1. 运营商信息（Label 组件）：
   - 使用 `label` 组件来显示运营商信息。选择清晰易读的字体和大小，确保在不同的设备上都能轻松阅读。
   - 将 `label` 组件放置在顶端信息栏的左侧，并设置合适的对齐方式，通常是左对齐。
2. 时间显示（Label 组件）：
   - 另一个 `label` 组件用于显示时间。您可以设置一个定时器，以确保时间始终是更新的。
   - 时间 `label` 通常位于信息栏的右侧位置，并且要有足够的辨识度，可以通过增大字体和加粗来实现。
3. 信号强度（Image 组件）：
   - 信号强度可以通过一系列预定义的信号图标来表示。这些图标可以根据信号强度的不同显示不同的图像。
   - 使用 `image` 组件来放置信号强度图标，并将其放置在顶端信息栏的右侧
4. 电量指示（Image 组件）：
   - 电量指示同样使用 `image` 组件来显示，您可以准备一组电量图标来反映不同的电量状态。
   - 将电量图标放置在顶端信息栏最右侧，紧邻信号强度图标

3.**天气模块**：

![](../../media/solutions/Agriculture-Control-Panel/UI_Weather_Column.png)

- **组件**：label、container、image。
- **当前天气**：展示当前天气状况和温度。
- **未来天气预报**：在当前天气旁边或下方提供横向滚动或小网格，显示未来几天的天气预报。

在 GUI Guider 中设计左上侧的天气信息模块时，可以通过组合 `label`、`image` 和 `container` 组件来创建一个直观并且信息丰富的天气显示区域。以下是如何使用这些组件来设计天气信息模块的步骤：

1. 基础容器设置（Container 组件）：
   - 在界面左上角放置一个 `container` 组件，作为天气模块的基础。这个容器将包含所有天气相关的子组件。
   - 调整 `container` 的大小，确保它可以容纳所有子组件而不显得拥挤。
2. 位置显示（Label 组件）：
   - 使用一个 `label` 组件在容器的顶部显示当前的地理位置或农场名。
   - 这个 `label` 可以使用较大的字体和加粗效果，以便用户可以清晰地识别位置信息。
3. 当前温度和天气（Label 组件）：
   - 为当前温度和天气描述使用两个 `label` 组件。这些应该使用清晰、易读的字体，并且大小适当，以便一目了然。
   - 当前温度的 `label` 应该是最显眼的，可能需要使用更大的字号和突出的颜色
4. 天气图标（Image 组件）：
   - 使用 `image` 组件来显示当前天气状况的图标，如晴天、阴天、雨天等。
   - 天气图标应该与当前温度的 `label` 相邻，提供直观的天气视觉表示。
5. 天天气预报（Label 和 Image 组件）：
   - 在 `container` 的下方或旁边，使用多个 `label` 和 `image` 组件来显示未来5天的天气预报。
   - 每天的预报可以使用一个小型 `image` 组件显示天气图标，旁边跟随个 `label` 组件显示日期。
6. 自适应设计：
   - 设计时要确保天气信息区域能够适应不同的屏幕尺寸和分辨率，特别是在响应式界面上。
   - 子组件应该能够在 `container` 大小变化时，自动调整其大小和位置。

4.**温度湿度历史曲线图**：

![](../../media/solutions/Agriculture-Control-Panel/UI_History_Curve_Bar.png)

- **组件**：container，chart ，label。
- **图表样式**：使用线状图展示温度和湿度的历史数据趋势。
- **时间选择**：提供查看不同时间段数据的选项，如日、周、月。
- **交互性**：允许用户悬停或点击图表上的点来查看特定数据点。

在使用 GUI Guider 设计智慧农业图形化界面的左下侧温度历史曲线时，布局 `label`、`chart`和 `container` 组件。这些组件将共同构成一个展示温度变化趋势的图表区域，以下是具体的设计说明：

1. 容器布局（Container 组件）：
   - 在界面的左下侧放置一个 `container` 组件，用来作为温度历史曲线的背景和框架。这个容器将包含图表和相关的标签。
   - 设定 `container` 的宽度和高度，以适应曲线图的展示需求，同时与其他界面元素保持协调。
2. 曲线图表（Chart 组件）：
   - 在 `container` 内部添加一个 `chart` 组件，用于绘制温度的历史数据曲线。
   - 配置 `chart` 组件来展示时间序列的温度数据，如设置 x 轴为时间轴，y 轴为温度值，确保数据点清晰可见。
   - 定义曲线样式，如曲线颜色、粗细，以及是否显示数据点的标记。
3. 标题标签（Label 组件）：
   - 在曲线图的上方或容器的顶部，使用 `label` 组件来显示标题，例如“历史曲线”。
   - 选择合适的字体大小和风格，以保证标题清晰且与整体设计风格一致。
4. 响应式设计：
   - 对于响应式界面，确保 `container` 和其中的 `chart` 和 `label` 组件能够根据不同屏幕尺寸适当缩放。
   - 在不同分辨率和设备上测试图表的显示效果，确保数据始终清晰可见。

5.**常见环境参数显示**：

![](../../media/solutions/Agriculture-Control-Panel/UI_Parameter_Display_Bar.png)

- **组件**：container，label。
- **数据展示块**：以独立的“块”或“卡片”形式展示关键参数，如土壤 pH 值、土壤温度、氧气和二氧化碳浓度等当前读数。

在 GUI Guider 软件中设计智慧农业图形化界面的右上侧环境参数显示时，可以使用 `label` 和 `container` 组件来清晰地展示土壤 pH 值、氧气浓度、土壤湿度和二氧化碳浓度等关键信息。以下是详细的设计步骤：

1. 环境参数容器（Container 组件）：
   - 在界面的右上侧放置一个 `container` 组件，作为所有环境参数的包裹元素。这个容器将帮助组织并分隔不同的参数显示。
   - 设置容器的宽度和高度，以适应内部 `label` 的排列，同时确保整体布局的美观和协调。
2. 参数标签（Label 组件）：
   - 对于每个环境参数，使用单独的 `label` 组件来显示参数名称和数值。例如，一个 `label` 显示“土壤PH值：7.1”，另一个 `label` 显示“氧气浓度：50%”等。
   - 参数名称的 `label` 可以使用较小的字体，而数值的 `label` 可以使用更大或加粗的字体，以突出显示数值信息。
3. 参数更新机制：
   - 如果可能的话，设计一个数据更新机制，以定时刷新显示的环境参数数值。这可以通过绑定 `label` 组件到数据源，并设置定时器来更新数值。

6.**告警信息区**：

![](../../media/solutions/Agriculture-Control-Panel/UI_Alarm_Information_Bar.png)

- **组件**：container，label，table。
- **列表视图**：以列表或日志形式显示最近的告警和警告。
- **优先级指示**：使用颜色编码或图标来指示告警的优先级或严重性。
- **交互性**：允许用户点击告警以获取更多详情或确认并清除通知。

在使用 GUI Guider 软件设计智慧农业图形化界面的右下侧告警信息区时，可以利用 `label`、`container`和 `table` 组件来有效地展示告警标题和具体的告警信息。这个区域将为用户提供关键的安全和系统状态更新，以下是详细的设计流程：

1. 告警信息容器（Container 组件）：
   - 在界面的右下侧放置一个 `container` 组件，用作告警信息区的主要框架。这个容器将包含告警信息的标题和具体内容。
   - 设定容器的尺寸，确保足够的空间来展示告警信息，同时保持与界面其他元素的协调。
2. 告警信息标题（Label 组件）：
   - 使用一个 `label` 组件在容器的顶部显示标题，例如“告警信息”。
   - 选择醒目且易于阅读的字体和大小，确保用户能够清楚地识别这一区域的功能。
3. 告警信息表格（Table 组件）：
   - 在 `container` 下方使用 `table` 组件来列出具体的告警信息。这可以包括告警时间、类型、状态等列。
   - 调整 `table` 的列宽和行高，确保所有信息都是清晰可见的。
4. 告警信息样式：
   - 为 `table` 组件设置合适的背景色、边框色和字体颜色，以便与整体界面风格保持一致。
   - 考虑使用不同的颜色或图标来表示不同级别的告警，如红色表示紧急告警，黄色表示警告。
5. 告警信息更新机制：
   - 设计一个数据更新机制，允许 `table` 组件动态显示最新的告警信息。这可能需要将 `table` 组件绑定到后端数据源，并设定定时刷新。
6. 响应式设计：
   - 确保 `container` 和其中的 `table` 和 `label` 组件在不同屏幕尺寸和分辨率上能够适当缩放。
   - 在小屏幕上可能需要调整 `table` 的布局，以保证信息的可读性。

在 GUI Guider 中放置这些元素时，要确保每个组件都根据网格或灵活布局系统放置，以适应不同的屏幕尺寸和分辨率。界面设计应简洁，保持足够的空白避免拥挤，使用的颜色方案应使得重要信息突出但又不刺眼，交互元素应提供即时反馈，例如悬浮或点击时改变外观。此外，应在不同设备上测试设计，以确保其可用性和性能。

如果系统将在各种光照条件下使用，比如直射阳光下，应能调整对比度和亮度以增强可见性。最后，界面设计应直观，使新用户无需广泛培训即可导航，反映出良好的用户体验（UX）设计原则。
