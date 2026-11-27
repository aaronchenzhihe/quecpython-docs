# 软件设计讲解



## 软件框架



### 软件设计图

![image-20231124112722602](../../media/solutions/Wear/image-20231124112722602.png)	



### 业务系统启动流程

![image-20231124140245780](../../media/solutions/Wear/image-20231124140245780.png)	



## 具体实现



### 代码目录

穿戴解决方案的代码托管于 [github](https://github.com/QuecPython/solution-wearable.git)，代码的目录结构如下：

```PlainText
.
|-- EventMesh.py
|-- common.py
|-- constant.py
|-- lcd.py
|-- css.py
|-- ui.py
|-- mgr.py
|-- main_t.py
|-- img
|   |-- point.png
|   |-- gps.png
|   |-- bat4.png
|   |-- ...png
```

- `EventMesh.py` 是一个事件管理器模块，当屏幕控件上的事件被触发时，用户仅需要调用相关接口发送事件即可，[EventMesh](https://github.com/QuecPython/EventMesh) 会自动进行屏幕内容切换。
- `common.py` 主要提供了公共的抽象基类和公共的依赖类， 例如`class Abstract`他提供了生命周期的抽象等。
- `constant.py`主要提供常量的配置。
- `lcd.py`主要提供LCD的驱动初始化， TP的驱动初始化， 以及LVGL的初始化， 只有当此文件初始化成功屏幕才会有显示。
- `css.py`主要提供了css的功能， 静态的样式， 字体样式等提供给`ui`文件中的屏幕对象使用。
- `ui.py`
  - 提供了屏幕对象的基类 `class Screen()` ，提供所有屏幕对象都需要的公共方法，如创建屏幕、状态栏信息显示（运营商名称、信号强度、系统时间、电池电量等）。
  - `class **Screen` 均实现了一个继承于 `class Screen()` 的类，是对应屏幕需要显示的画面的代码实现。界面代码的主体内容是由 GUI Guider 工具自动生成的，而后进行少许修改，主要是对象初始化方法 `__init__()` 和控件事件处理回调函数（`__xxx_event_cb()`）的修改， 或控件的生命周期中修改
- `mgr.py`主要提供后台功能， 允许界面在不同的生命周期或事件触发对， 和各个管理器进行交互。
- `main_t.py` 是应用入口脚本文件。
  - 调用`lcd.py`初始化屏幕驱动及显示。
  - 初始化`APP`， 穿戴的APP。
  - 调用 `ui.py` 提供的类 `class UI()` 创建穿戴方案的 GUI 对象，并调用该 GUI 对象的屏幕对象添加器添加需要的屏幕对象， 添加时会执行页面的实例化方法。
  - 调用`APP`添加管理器， 并执行管理器的实例化方法。
  - 启动`APP`。
  - 跳转到主界面。



### 界面切换机制



#### EventMesh 事件管理框架

上文已经提及，界面切换采用 EventMesh 事件管理框架进行驱动。

- EventMesh 是一个基于订阅和发布机制的应用框架，用户可以订阅任何感兴趣的主题，该主题即表示一个事件。
- 在订阅主题的同时，还需要注册一个回调函数，用于处理产生的事件。
- 用户可在任何产生事件的地方发布该事件，并可以携带消息内容。此时订阅者注册的回调函数会被调用，事件得到处理。

EventMesh 的设计框架如下图所示：

![event-mesh.png](../../media/solutions/Wear/event-mesh.png)	



#### **界面切换实现原理**

<img src="../../media/solutions/Wear/image-20231124133650808.png" alt="image-20231124133650808.png" style="zoom:150%;" />	



页面的模版和跳转的生命周期示例

```python
# ui.py中

# 如果添加新的界面 class **Screen 必须继承与Screen， lv.obj对象必须提前实现好

class DemoScreen(Screen):
	# NAME必须， 这个是找到对应的界面跳转的唯一通道
	# 例如我们使用 EventMesh.publish("load_screen", {"screen": "demo_screen"})
	# 其中demo_screen就是去找NAME这个字段找到他
	NAME = "demo_screen"
    def __init__(self):
        super().__init__()
        # 这是核心跳转对象
        self.meta = app_list_screen
		# prop可以获取跳转信息例如， 可以获取跳转来的信息
		# EventMesh.publish("load_screen", {"screen": "demo_screen", "test": 1})
		# 则此界面的prop = {"screen"： "demo_screen", "test": 1}
        self.prop = None

	# 用户实现此接口， 会在ui.add_screen(DemoScreen())的时候调用， 只会在添加的时候执行一次
    def post_processor_after_instantiation(self, *args, **kwargs):
        """实例化后调用"""
        pass

	# 每当EventMesh.publish("load_screen", {"screen": "demo_screen"})跳转到此界面
	# 会依次执行
        # post_processor_before_initialization
		# initialization
		# post_processor_after_initialization
	# 用户可以在此实现他的功能， 因为每次跳转进来都会执行
    def post_processor_before_initialization(self, *args, **kwargs):
        """初始化之前调用"""
        pass

    def initialization(self, *args, **kwargs):
        """初始化load"""
        pass


    def post_processor_after_initialization(self, *args, **kwargs):
        """初始化之后调用"""
        pass
	# 会在离开此界面的时候执行， 比如说目前在此界面你通过下面
	# EventMesh.publish("load_screen", {"screen": "main_screen"})
	# 跳转到main界面的话， 会先执行此界面的离开方法， 在执行main界面的初始化方法
    def deactivate(self, *args, **kwargs):
        """初始化load"""
        pass



# 注意事项， 如果界面是瓦片界面tileview的界面的话需要额外添加对应关系才能使用
# EventMesh.publish("load_screen", {"screen": "main_screen"})进行跳转
# 详见下列UI的类中的关系

class UI(Abstract):
    def __init__(self):
        self.screens = []
        self.current = None
        # 需要在下面的map中添加瓦片的映射关系， display_screen对应的是0行0列的瓦片
        # 并且跳转是不进行动画的行为的
        # main_screen是1行， 0列的瓦片依次如下
        self.tileview_map = {
            "display_screen": [0, 0, lv.ANIM.OFF],
            "main_screen": [1, 0, lv.ANIM.OFF],
            "watch_face_screen": [2, 0, lv.ANIM.OFF],
            "app_list_1_screen": [3, 0, lv.ANIM.OFF],
            "app_list_2_screen": [4, 0, lv.ANIM.OFF],
            "heart_screen": [5, 0, lv.ANIM.OFF],
            "blood_screen": [6, 0, lv.ANIM.OFF],
            "temp_screen": [7, 0, lv.ANIM.OFF],
        }
		# 这和上面界面和瓦片的关系相反
		# 0行0列代表， display_screen
		# 依次如下
        self.tileview_position_map = {
            (0, 0): "display_screen",
            (1, 0): "main_screen",
            (2, 0): "watch_face_screen",
            (3, 0): "app_list_1_screen",
            (4, 0): "app_list_2_screen",
            (5, 0): "heart_screen",
            (6, 0): "blood_screen",
            (7, 0): "temp_screen",
        }


# 什么是瓦片， 比如说， 以下是静态界面tile1瓦片位置在1行0列位置， 支持右滑左滑的模式
tile1 = tileview.add_tile(1, 0, lv.DIR.RIGHT | lv.DIR.LEFT)
# main_screen放置在tile1的瓦片上， 则屏幕左滑会进入到(0, 0)的瓦片上右滑会进入(2, 0)的瓦片上
main_screen = lv_obj(
    parent=tile1,
    size=(240, 280),
    style=[
        (style_main_screen, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

```

从上述代码可以看到，每一个触摸事件的回调函数的实现方式都是类似的。

以应用列表控件的触摸为例，代码如下：

```python
class AppList1Screen(Screen):
    NAME = "app_list_1_screen"

    def __init__(self):
        super().__init__()
        self.meta = app_list_screen
        self.container = app_list_cont
        self.profile = [
            ["U:/media/app_heart.png", {"screen": "blood_screen"}],
            ["U:/media/app_phone.png", {"screen": "dail_screen"}],
            ["U:/media/app_chat.png", None],
            ["U:/media/app_time.png", None]
        ]
        self.btn_list = []
        self.bottom = None
        self.bottom_profile = [
            ["U:/media/wpoint.png"],
            ["U:/media/bpoint.png"],
            ["U:/media/bpoint.png"],
            ["U:/media/bpoint.png"]
        ]
        self.bottom_btn_list = []

	def btn_click(self, event, i):
        screen_info = self.profile[i][1]
        if screen_info:
            # 在找到对应的点击事件， 跳转页面
            EventMesh.publish("load_screen", screen_info)

    def post_processor_after_instantiation(self):
        # 会在add_screen的时候调用进行实例化
        for i, btn_profile in enumerate(self.profile):
            btn = lv_img(
                parent=self.container,
                size=(110, 110),
                src=btn_profile[0],
                flag=lv.obj.FLAG.CLICKABLE,
                style=[
                    (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
                    (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
                    (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
                    (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
                    (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
                ]
            )
            # 给每个按键添加了， 点击事件
            btn.add_event_cb(lambda event, cur=i: self.btn_click(event, cur), lv.EVENT.CLICKED, None)
            self.btn_list.append(btn)
        self.bottom = lv_obj(
            parent=app_list_screen,
            pos=(88, 254),
            size=(56, 8),
            flex_flow=lv.FLEX_FLOW.ROW,
            flex_align=(lv.FLEX_ALIGN.SPACE_AROUND, lv.FLEX_ALIGN.START, lv.FLEX_ALIGN.START),
            style=[
                (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
                (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
                (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
                (style_pad_default, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
                (style_app_list, lv.PART.MAIN | lv.STATE.DEFAULT),
            ]
        )
        for btn_profile in self.bottom_profile:
            btn = lv_img(
                parent=self.bottom,
                size=(8, 8),
                src=btn_profile[0],
                style=[
                    (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
                    (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
                    (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
                    (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
                    (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
                ]
            )
            self.bottom_btn_list.append(btn)
```

该段代码的核心语句是 `EventMesh.publish("load_screen", screen_info)` ，该语句发送一个名为 `"load_screen"` 的事件，意为加载新的屏幕界面；携带的消息为 `screen_info`， `screen_info`就是`{"screen": "blood_screen"}`要跳转的界面，即屏幕界面的名称，表示加载的是血氧显示界面。

#### **UI 启动**

`"load_screen"` 事件在 `Agri_ui.py` 脚本文件的 `class AgriUi(object)` 类的 `start()` 方法中被订阅。

代码如下：

```python
class UI(Abstract):

    def __init__(self):
		...

    def lv_load(self, event, msg):
        for screen in self.screens:
            if screen.NAME == msg["screen"]:
                scr = screen
                if self.current != scr:
                    if self.current:
                        self.current.deactivate()
                        scr.last_screen_info = {"screen": self.current.NAME}
                    scr.set_prop(msg)
                    scr.post_processor_before_initialization()
                    scr.initialization()
                    scr.post_processor_after_initialization()
                    self.current = scr
                    if msg["screen"] in self.tileview_map:
                        lv.scr_load(tileview_screen)
                        tileview.set_tile_id(*self.tileview_map[msg['screen']])
                    else:
                        lv.scr_load(self.current.meta)

    def post_processor_after_instantiation(self):
        EventMesh.subscribe("load_screen", self.lv_load)
        EventMesh.subscribe("load_tileview", self.lv_tileview)

    def add_screen(self, screen):
        screen.post_processor_after_instantiation()
        self.screens.append(screen)

    def start(self):
        self.post_processor_after_instantiation()
```

上述代码中的 `start()` 方法用于启动穿戴方案的图形化界面，其工作流程如下：

- 调用 `add_screen` 方法创建每个屏幕界面的元数据。

- 调用 `EventMesh.subscribe()` 方法订阅名为 `"load_screen"` 的事件，当该事件产生时，调用 `lv_load()` 方法处理该事件。

  > `lv_load()` 方法根据发布事件时携带的消息，即屏幕界面的名称，去匹配相应的界面对象，调用 lvgl 的 `lv.scr_load()` 方法加载出新的界面。
  >
  > 细心的读者会发现，每一个屏幕界面脚本文件中实现的类中，其类下面 中，都会有类似 `NAME = "main_screen"` 的语句。该语句便记录了屏幕界面的名称。

- `EventMesh.publish("load_screen", {"screen": "main_screen"})` 语句用来触发第一个界面显示，即主界面。

至此，穿戴方案的图形化界面启动完毕，后续界面的切换由用户触摸控制。



### 图形化界面设计

上文提到，QuecPython 使用 NXP 公司的 [GUI Guider](https://www.nxp.com.cn/design/design-center/software/development-software/gui-guider:GUI-GUIDER) 作为图形化界面设计工具，该工具不仅能够进行界面布局设计，还能自动生成 QuecPython 代码。[点此查看 GUI Guider 工具的使用教程](../../multi-media/lvgl/lvgl_ui_tool.md)。

下文以穿戴方案的应用为例，来介绍图形化界面的设计过程。



#### 布局和背景设置

创建一个手表方案，选择一个适合的布局摸板和背景设计。 该阶段将从空白布局开始设计，即选择空白摸板，设置分辨率为 240*280。

![](../../media/solutions/wear/image-20231124113050485.png)

#### 界面绘制

该部分主要绘制表盘主页， 和讲解页面布局以及如何快速使用代码实现。

1. 顶部栏

![](../../media/solutions/wear/image-20231124113553528.png)	

- **组件：** obj， img， label。
- **运营商和信号(盒子1)：** 是一个obj， 里面又一个img显示信号， label显示运营商， 布局中他是靠左对齐的， 兵器img和label之间有个4px的间隔。
  - obj尺寸大小是具体的长度 * 20px。
  - img尺寸是 20px * 20px。
- **导航和电池电量(盒子2)：** 是一个obj， 里面又一个导航的img和电池电量的img， 中间是有个8px的间隔， 尺寸在。
  - obj大小需要 48px * 20px。
  - img尺寸是 20px * 20px。
- **整个顶部布局为：**
  - flex布局， 默认主轴水平方向。
  - 尺寸为240px * 34px。
  - 布局容器为主轴左右对齐space-between。
  - 子属性全使用flex-end对齐方式， 代表靠纵轴方向底部对齐。
  - 布局padding为，离上0px， 右12px， 下4px， 左12px。

代码实现：

```python
# 所有的， style均来自于css.py


# 创建一个主页面
main_screen = lv_obj(
    size=(240, 280),
    style=[
        (style_main_screen, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 设置头部栏
main_top = lv_obj(
    parent=main_screen,
    # 大小是240px * 34px
    size=(240, 34),
    # flex_flow 为flex， row水平布局
    flex_flow=lv.FLEX_FLOW.ROW,
    # 主轴是space_between， 其他轴底部对齐
    flex_align=(lv.FLEX_ALIGN.SPACE_BETWEEN, lv.FLEX_ALIGN.END, lv.FLEX_ALIGN.END),
    style=[
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_header, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        # padding我们采用了style设置
        (style_bar_top_main, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 盒子1
main_top_cont_1 = lv_obj(
    # 盒子一是在头部栏里面的， 设置父节点是main_top
    parent=main_top,
    # 大小是88 * 20px
    size=(88, 20),
    flex_flow=lv.FLEX_FLOW.ROW,
    flex_align=(lv.FLEX_ALIGN.SPACE_BETWEEN, lv.FLEX_ALIGN.END, lv.FLEX_ALIGN.END),
    style=[
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        # 这里设置了main_top_cont_1中设置了gap间距为4
        (style_main_top_cont_1, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 信号图片父节点是盒子1
main_top_cont_1_img_signal = lv_img(
    parent=main_top_cont_1,
    size=(20, 20),
    src="U:/media/s4.png",
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 运营商父节点是盒子1
main_top_cont_1_label_operator = lv_label(
    parent=main_top_cont_1,
    size=(64, 19),
    text="中国移动",
    style=[
        (style_font_16, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_font_grey, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 盒子2
main_top_cont_2 = lv_obj(
    parent=main_top,
    size=(48, 20),
    flex_flow=lv.FLEX_FLOW.ROW,
    flex_align=(lv.FLEX_ALIGN.SPACE_BETWEEN, lv.FLEX_ALIGN.END, lv.FLEX_ALIGN.END),
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 导航图片父节点是盒子2
main_top_cont_2_img_gps = lv_img(
    parent=main_top_cont_2,
    size=(20, 20),
    src="U:/media/point.png",
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 电池电量图片父节点是盒子2
main_top_cont_2_img_bat = lv_img(
    parent=main_top_cont_2,
    size=(20, 20),
    src="U:/media/bat4.png",
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

```



2. 时间栏

![image-20231124131736253](../../media/solutions/wear/image-20231124131736253.png)	

- **时间栏**
  - 大小是116px * 200px。
  - 定位如下：
    - top： 64px
    - left： 16px
- **时针栏(盒子1)**
  - 尺寸为118px * 200px。
  - 有两张图片组成。
    - 每张图片尺寸是58 * 100px。
  - 布局为flex布局， 横轴方向。
- **分针栏(盒子2)**
  - 有两张图片组成。
  - 尺寸是58 * 100px。
  - 有两张图片组成。
    - 每张图片尺寸是58 * 100px。
  - 布局为flex布局， 横轴方向。
- **定位为**
  - flex布局， 纵轴方向分布。

```python
# ... main_screen

# 创建时间盒子
main_content_cont_1 = lv_obj(
    parent=main_screen,
    size=(116, 200),
    # 定位为左19， 右64， 基于父界面
    pos=(19, 64),
    # 纵轴方向flex布局
    flex_flow=lv.FLEX_FLOW.COLUMN,
    flex_align=(lv.FLEX_ALIGN.SPACE_BETWEEN, lv.FLEX_ALIGN.END, lv.FLEX_ALIGN.END),
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT)
    ]
)

# 创建盒子1
main_content_cont_1_hour = lv_obj(
    parent=main_content_cont_1,
    size=(116, 100),
    # 水平布局
    flex_flow=lv.FLEX_FLOW.ROW,
    flex_align=(lv.FLEX_ALIGN.SPACE_BETWEEN, lv.FLEX_ALIGN.END, lv.FLEX_ALIGN.END),
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 盒子1中时钟图片1
main_content_cont_1_hour_0 = lv_img(
    parent=main_content_cont_1_hour,
    size=(58, 100),
    src="U:/media/h0.png",
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 盒子1中时钟图片2
main_content_cont_1_hour_1 = lv_img(
    parent=main_content_cont_1_hour,
    size=(58, 100),
    src="U:/media/h8.png",
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 创建盒子2由于父节点盒子是flex纵节点布局， 可以分布到盒子适当位置
main_content_cont_1_m = lv_obj(
    parent=main_content_cont_1,
    size=(116, 100),
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_flex_raw_between, lv.PART.MAIN | lv.STATE.DEFAULT)
    ]
)

# 盒子2创建分钟图片0
main_content_cont_1_m_0 = lv_img(
    parent=main_content_cont_1_m,
    size=(58, 100),
    src="U:/media/m0.png",
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)

# 盒子2创建分钟图片1
main_content_cont_1_m_1 = lv_img(
    parent=main_content_cont_1_m,
    size=(58, 100),
    src="U:/media/m8.png",
    style=[
        (style_cont, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT),
        (style_list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED),
        (style_gap_default, lv.PART.MAIN | lv.STATE.DEFAULT),
        (style_pad_default, lv.PART.MAIN | lv.STATE.DEFAULT),
    ]
)
```



在 GUI Guider 中放置这些元素时，要确保每个组件都根据网格或灵活布局系统放置，以适应不同的屏幕尺寸和分辨率。界面设计应简洁，保持足够的空白避免拥挤，使用的颜色方案应使得重要信息突出但又不刺眼，交互元素应提供即时反馈，例如悬浮或点击时改变外观。此外，应在不同设备上测试设计，以确保其可用性和性能。

如果系统将在各种光照条件下使用，比如直射阳光下，应能调整对比度和亮度以增强可见性。最后，界面设计应直观，使新用户无需广泛培训即可导航，反映出良好的用户体验（UX）设计原则。

GUI工具生成的代码， 本身不具备很好的伸缩性和维护性，建议生成后的代码使用上述的方式封装改写， 简介明了可维护性更高， 更好形成可维护性。
