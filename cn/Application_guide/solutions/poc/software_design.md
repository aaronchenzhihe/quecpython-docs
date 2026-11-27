# 软件设计讲解

## 软件框架

### 软件设计图

<img src="../../media/solutions/poc/software_design.png"/> 

- 按键模块
  - KeyManger：用于管理按键功能。
- 服务模块
  - DevInfoService：设备信息服务，用于查询设备 IMEI、ICCID、固件版本信息等；
  - MediaService：媒体服务，用于管理 tts 播报、mic 和 audio 音频处理；
  - NetService：网络服务，用于管理网络状态和心跳检测；
  - PocService：poc 服务，用于 poc 登录、群组获取、成员获取和对讲管理等。
- 界面模块
  - MenuBar：状态栏，用于显示信号、时间、电量以及对讲图标；
  - PromptBox：提示框，用于显示消息弹窗信息；
  - Screen：UI 屏幕，用于显示各种界面信息，用户可自定义；
  - PocUI：用于管理用户定义的 Screen。
- 事件管理模块：
  - EventMap：用于事件的发送与绑定。

### 业务系统启动流程

![](../../media/solutions/poc/start.png) 

## 代码讲解

### 核心业务模块(POC)

> 有关 poc 库的 API 函数可参考[POC-公网对讲机](../../../api_reference/industry/poc.md)说明文档。

- #### 账户登录

  通过注册 `poc.login()` 回调函数，监听登录状态，并根据登录结果执行相应的操作。
  具体功能如下：

  1. 登录回调：通过 `poc.login()` 注册回调函数，监听登录状态（`param` 参数表示登录成功或失败）。
  2. 登录成功处理：如果登录成功（`param == 1`），更新网络状态为正常，并根据平台类型（标准平台或其他平台）清理或存储安全数据。
  3. 登录失败处理：如果登录失败，启动定时器定期检查网络连接状态，并标记网络错误状态。
  4. 网络状态检查：通过定时器定期检查网络连接状态，确保设备能够重新尝试登录。
  5. 界面更新：登录成功后，通知欢迎界面（`WelcomeScreen`）更新登录状态，并查询群组信息。

  该功能模块确保设备能够正确登录 POC 平台，并在登录失败时自动尝试重新连接。

  ```python
  class PocService(AbstractLoad):
      ...
      
      # 登录回调
      poc.login(self.__poc_login_cb)
      
      def __poc_login_cb(self, param):
          EventMap.send("welcomescreen__check_cloud_status", param) # 登录成功首页显示已登录，且去查询组群信息
          # 已登录
          if param == 1:
              self.net_error = False
              if self.__platform_dict.get(self.__platform) == 'std':
                  self.__securedata_xin_clear()
              else:
                  self.__securedata_xin_store()
          # 未登录
          else:
              self.__cloud_check_timer.start(5*1000, 1, lambda arg: self.__check_cloud_connect())  
              self.net_error = True
  ```
  
- #### 进入群组

  通过注册 `poc.register_join_group_cb()` 回调函数，监听设备入组事件，并根据入组状态更新界面和播放提示音。
  具体功能如下：
  
  1. 注册入组回调：通过 `poc.register_join_group_cb()` 注册回调函数，监听设备是否成功进入群组。
  2. 获取群组信息：通过 `poc.group_getbyid()` 查询当前群组的详细信息。
  3. TTS 语音播报：根据入组状态和群组类型（临时群组或普通群组），生成相应的提示信息并通过 TTS 语音播报。
  4. 状态管理：更新当前群组名称、登录状态和发言权限状态，确保设备状态与群组信息同步。
  5. 临时群组处理：如果进入的是临时群组，启动定时器，在指定时间后自动退出临时群组。
  
  该功能模块确保用户在进入群组时能够及时收到语音提示，并正确处理群组切换和状态更新。
  
  ```python
  class PocService(AbstractLoad):
      ...
      
      # 注册入组回调
      poc.register_join_group_cb(self.__poc_join_group_cb)
      
      def __poc_join_group_cb(self, param):
          """
          入组回调, 二次入相同群组, 不需要播报提示,
          """
          PrintLog.log("PocService", "poc join group = {}".format(param))
          if not param[-1]:
              return
          group = poc.group_getbyid(0)
          if isinstance(group, list):
              now_group_name = group[1]
              if not group[2]:
                  self.__last_join_group = group
                  self.__call_time_status = False
                  self.__call_member_timer.stop()
              if not self.__group_name:
                  self.__group_name = now_group_name
              else:
                  if self.__group_name == now_group_name:
                      self.tts_play_enable = False
                  else:
                      self.tts_play_enable = True
                  self.__group_name = now_group_name
              if self.__login_status:
                  if group[2]:
                      tts_msg = "进入" + "临时群组" + self.__group_name
                  else:
                      tts_msg = "进入群组" + self.__group_name
              else:
                  tts_msg = self.__get_user_info() + "已登录" + "进入群组" + self.__group_name
                  self.tts_play_enable = True
              if not self.__login_status:
                  self.__login_status = True
              if self.tts_play_enable:
                  EventMap.send("mediaservice__tts_play", (tts_msg, 1))
                  if not self.__rocker_arm:
                      if not self.speak_close_first:
                          self.speak_close_first = True
                          EventMap.send("pocservice__close_speaker",None ,EventMap.MODE_ASYNC)
              if group[2]:
                  self.__call_time_status = True
                  self.__call_member_timer.start(self.__call_quit_time * 1000, 0, lambda arg: self.__call_member_exit())
  ```
  
- #### 获取群组、成员列表

  通过调用 POC 接口获取群组和成员数据，并将数据返回给界面层进行渲染。
  具体功能如下：
  
  1. 获取群组列表：通过 `poc.get_groupcount()` 获取当前账号加入的群组数量，并通过 `poc.get_grouplist()` 获取群组列表数据。
  2. 获取成员列表：通过 `poc.group_getbyid()` 获取当前群组信息，然后使用 `poc.get_membercount()` 和 `poc.get_memberlist()` 获取成员数量和成员列表数据。
  3. 异常处理：如果群组或成员数据无效（如返回值为 -1 或数量为 0），则返回空数据，界面层会显示相应的提示信息。
  
  该功能模块为群组管理和成员列表界面提供了数据支持，确保用户能够查看和操作群组及成员信息。
  
  ```python
  class PocService(AbstractLoad):
      ...
      
      EventMap.bind("group_get_list", self.__get_group_list)
      EventMap.bind("member_get_list", self.__get_member_list)
      
      def __get_group_list(self, event=None, msg=None):
          """获取群组列表"""
          group_count = poc.get_groupcount()	# 获取群组个数
          group_list = poc.get_grouplist(0, group_count)	# 根据群组个数获取群组列表
          return group_count, group_list
  
      def __get_member_list(self, event=None, msg=None):
          """获取成员列表"""
          group = poc.group_getbyid(0)	# 通过 gid 查询群组信息
          if -1 == group:
              return -1, None
          member_count = poc.get_membercount(group[0])
          if -1 == member_count or 0 == member_count:
              return -1, None
          member_list = poc.get_memberlist(group[0], 0, member_count)
          return member_count, member_list
  ```
  
- #### POC 对讲

  此部分是解决方案的核心功能模块，负责处理对讲状态的管理、网络状态检查以及界面提示的更新。
  具体功能如下：
  
  1. 开启对讲：当长按开发板 KEY1 键时，唤醒 LCD 屏幕并开启对讲功能。同时检查网络状态和当前群组状态，确保对讲功能正常使用。
  2. 关闭对讲：当松开 KEY1 键时，结束对讲并释放相关资源，更新界面状态。
  3. 网络状态检查：如果网络异常（如 SIM 卡问题），提示用户更换 SIM 卡。
  4. 群组状态检查：如果当前群组无效，提示用户选择有效群组。
  5. 界面提示更新：在对讲过程中，显示`讲话中...`提示框，并更新状态栏的对讲图标。
  6. 音频管理：在对讲开启和关闭时，启用或禁用降噪功能，并播放提示音。
  
  该功能模块确保了对讲功能的稳定性和用户体验的流畅性。
  
  ```python
  class PocService(AbstractLoad):
      ...
      
      EventMap.bind("pocservice__speaker_enable", self.__speaker_enable)
      
      def __speaker_enable(self, event, msg=None):
          # 开启Poc对讲
          PrintLog.log("PocService", "speaker enable: {}".format(msg))
          if msg:
              EventMap.send("poc_play_status", True)  # 唤醒LCD
              if self.__speaker_status:
                  EventMap.send("mediaservice__noise_reduction_enable", 1)
                  poc.speak(1)
                  if self.net_error:
                      if 3 != EventMap.send("welcomescreen__get_net_status"):
                          EventMap.send("mediaservice__tts_play", ("请更换卡", 1)) 
                          EventMap.send("load_msgbox", "请更换sim卡")  
                      return False
                  
                  # 检测当前群组
                  curr_group = poc.group_getbyid(0)
                  if -1 == curr_group:
                      EventMap.send("mediaservice__tts_play", (self.__group_name_default, 1)) 
                      EventMap.send("load_msgbox", self.__group_name_default)
                  else:
                      if not self.__rocker_arm:
                          EventMap.send("update_session_info", "您已被关闭发言")
                      else:
                          EventMap.send("load_msgbox", "讲话中...")	# 加载消息提示框
                          EventMap.send("menubar__update_poc_status", 1)	# 更新状态栏图标
                          
              else:
                  EventMap.send("mediaservice__audio_tone")
              return True
          # 关闭Poc对讲
          else:
              if self.__speaker_status:
                  EventMap.send("mediaservice__noise_reduction_enable", 0)
                  poc.speak(0)
                  utime.sleep_ms(100)
                  if not self.__rocker_arm:
                      pass
                  else:
                      EventMap.send("close_msgbox")
                      EventMap.send("menubar__update_poc_status", 0)
                      EventMap.send("poc_play_status", False)
  ```
  
- #### 对方呼叫回调

  通过注册 `poc.register_audio_cb()` 回调函数，监听对方的呼叫信息，并根据呼叫状态更新设备状态和界面提示。
  具体功能如下：
  1. 注册音频回调：通过 `poc.register_audio_cb()` 注册回调函数，监听对方的呼叫信息（`params` 参数包含语音状态、用户 ID、用户名和打断标志）。
  2. 呼叫状态处理：
     - 如果对方正在呼叫（`params[0] == self.BAND_CALL`），更新设备状态为“主动呼叫”。
     - 如果对方开始播放语音（`params[0] == self.BND_LISTEN_START`），更新设备状态为“呼叫结束”，并根据打断标志设置发言权限。
     - 如果对方停止播放语音（`params[0] == self.BND_LISTEN_STOP` 或 `params[0] == self.BND_SPEAK_STOP`），处理打断逻辑并更新设备状态。
  
  3. 界面更新：在对方呼叫时，显示消息提示框，唤醒 LCD 屏幕，并更新状态栏的对讲图标。
  4. 状态管理：根据呼叫状态更新设备的主叫状态、发言状态和会话信息，确保设备状态与呼叫信息同步。
  
  该功能模块确保设备能够正确处理对方的呼叫信息，并实时更新界面和状态。
  
  ```python
  class PocService(AbstractLoad):
      ...
      
      # 注册音频回调
      poc.register_audio_cb(self.__poc_audio_cb)
      
      def __poc_audio_cb(self, params):
          PrintLog.log("PocService", "poc audio: {}".format(params))
          if params[0] == self.BAND_CALL:
              self.main_call_end_state = self.CALL_STATE.IN_CALL
              self.__speaker_status = 3
              self.last_audio = params[0]
  
          elif params[0] == self.BND_LISTEN_START:
              self.last_audio = params[0]
              self.main_call_end_state = self.CALL_STATE.CALL_END
              if params[-1] == 0:
                  self.__speaker_status = 0   # 不允许打断
              else:
                  self.__speaker_status = 2
              self.__session_info = params[2]
              state_msg = self.__session_info    
              EventMap.send("load_msgbox", state_msg)	# 加载消息提示框
              EventMap.send("poc_play_status", True)	# 唤醒 LCD 屏幕
              EventMap.send("menubar__update_poc_status", 2)	# 更新状态栏图标
              EventMap.send("pocservice__call_member_status", 1)
  
          elif params[0] == self.BND_LISTEN_STOP or params[0] == self.BND_SPEAK_STOP:
              # 需要判断是否是高等级打断播放
              if params[0] == self.BND_LISTEN_STOP and self.main_call_end_state == self.CALL_STATE.IN_CALL:
                  return
              if params[0] == self.BND_LISTEN_STOP:
                  self.__speaker_status = params[-1]
              self.__error_ptt_handler(params)
          else:
              pass
  ```

### UI 界面

- #### 状态栏(MenuBar)

  MenuBar 类通过绑定、发送事件更新状态栏中的各个组件，确保用户能够实时查看设备的关键信息。
  具体功能如下：
  
  1. 更新信号强度：通过事件获取信号强度并显示对应的信号图标和网络类型（如4G）。
  2. 更新时间：通过事件获取当前时间并显示在状态栏中。
  3. 更新电量：通过事件获取当前电量并更新电量图标。
  4. 更新对讲状态：根据对讲状态（对讲、播放）显示对应的图标，并控制图标的可见性。
  
  状态栏的大小为 240 × 40，位于 LCD 屏幕的上方。
  
  ```python
  class MenuBar(AbstractLoad):
      NAME = "MenuBar"
      ...
      
      def __update_time(self, arg=None):
          time = EventMap.send("devinfoservice__get_time")
          if time:
              self.lab_time.set_text(time[1])
  
      def __update_battery(self, arg=None):
          battery = EventMap.send("screen_get_battery")
          if battery:
              self.img_battery.set_src(battery)
  
      def __update_signal(self, arg=None):
          sig = EventMap.send("screen_get_signal")
          if 0 < sig <= 31:
              self.img_signal.set_src('U:/img/signal_' + str(int(sig * 5 / 31)) + '.png')
              self.lab_signal.set_text("4G")
          else:
              self.img_signal.set_src("U:/img/signal_0.png")
              self.lab_signal.set_text("x")
              
      def __update_poc_status(self, event, msg):
          """
          0 停止  1 对讲  2 播放
          """
          PrintLog.log(MenuBar.NAME, "poc status: {}".format(msg))
          if 0 == msg:
              self.img_poc.add_flag(lv.obj.FLAG.HIDDEN)
          elif 1 == msg:
              self.img_poc.clear_flag(lv.obj.FLAG.HIDDEN)
              self.img_poc.set_src("U:/img/poc_speaking.png")
          elif 2 == msg:
              self.img_poc.clear_flag(lv.obj.FLAG.HIDDEN)
              self.img_poc.set_src("U:/img/poc_play.png")
  ```
  
- #### 消息提示框(PromptBox)

  PromptBox 类通过弹窗的形式展示消息内容，并支持动态更新和关闭弹窗。
  具体功能如下：
  
  1. 显示消息弹窗：根据传入的消息内容（`msg`）和元数据（`meta`），创建一个弹窗并居中显示在屏幕上。
  2. 动态更新消息：如果弹窗已存在，则在显示新消息前关闭旧弹窗，确保消息的实时性。
  3. 关闭弹窗：提供关闭弹窗的功能，释放资源并隐藏弹窗。
  
  弹窗的大小为 180 × 90，消息内容支持自动换行，并居中显示。
  
  ##### 定义
  
  ```python
  class PromptBox(AbstractLoad):
      NAME = "PromptBox"
      ...
  
      def __close(self, event=None, msg=None):
          if self.prompt_box is not None:
              self.prompt_box.delete()
              self.prompt_box = None
  
      def __show(self, event, msg):
          if self.prompt_box is not None:
              self.prompt_box.delete()
              self.prompt_box = None
  
          meta = msg.get("meta")
          show_msg = msg.get("msg")
  
          self.prompt_box = lv.msgbox(meta, "PromptBox", "", [], False)
          self.prompt_box.set_size(180, 90)
          self.prompt_box.align(lv.ALIGN.CENTER, 0, 0)
          self.prompt_label = lv.label(self.prompt_box)
          self.prompt_label.set_pos(0, 0)
          self.prompt_label.set_size(140, 50)
          self.prompt_label.add_style(FontStyle.consolas_12_txt000000_bg2195f6, lv.PART.MAIN | lv.STATE.DEFAULT)
          self.prompt_label.set_text(show_msg)
          self.prompt_label.set_long_mode(lv.label.LONG.WRAP)
          self.prompt_label.set_style_text_align(lv.TEXT_ALIGN.CENTER, 0)  
  ```
  
  ##### 使用
  
  ```python
  class PocUI(AbstractLoad):
      ...
      
      def load_msgbox(self, event, msg):
          """
          加载消息框, 注意msg的格式:
          {
              "type": "promptbox", # 默认提示框
              "title": "[promptbox]"
              "msg": "hello world",
              "mode": 0
          }
          """
          if isinstance(msg, dict):
              _type = msg.get("type", PromptBox.NAME) # 默认提示框
              _type = "{}__show".format(type.lower())
              _msg = {
                  "meta":self.curr_screen.meta,
                  "msg": msg.get("msg", "[promptbox]"),
                  "mode": msg.get("mode", 0)
              }
              EventMap.send(_type, _msg)
          else:
              _msg = {
                  "meta":self.curr_screen.meta,
                  "title": "[promptbox]",
                  "msg": msg,
                  "mode": 0
              }
              EventMap.send("promptbox__show", _msg)
  
      def close_msgbox(self, event, msg):
          """
          在这里对所有消息框发送关闭消息
          """
          EventMap.send("promptbox__close")
  ```
  
- #### UI 屏幕

  >  以 MemberScreen 为例介绍。

  MemberScreen 类继承自 `Screen`，用于在 LCD 屏幕上显示成员信息，大小为 240 × 200，与状态栏共同构成完整的 LCD 显示区域。
  具体功能如下：
  
  1. 加载成员列表：从事件中获取成员列表数据，并动态创建列表项显示在屏幕上。
  2. 管理选中状态：通过高亮背景色和滚动效果，显示当前选中的成员项。
  3. 按键操作：支持按键事件（如单击、长按）来切换选中项或返回主界面。
  4. 异常处理：如果成员列表为空或无成员，显示提示弹窗并返回主界面。
  5. 界面更新：在成员列表发生变化时，动态更新列表内容。
  
  该类的设计实现了成员列表的动态加载和交互管理，确保用户能够方便地查看和操作成员信息。
  
  ##### 加载并添加样式
  
  ```python
  class MemberScreen(Screen):
      NAME = "MemberScreen"
  
      def __init__(self):
          ...
  
          self.meta = lv.obj()    # lvgl meta object
          self.meta.add_style(CommonStyle.default, lv.PART.MAIN | lv.STATE.DEFAULT)
          # 列表------------------------------------------------------------------------------------------
          self.list_menu = lv.list(self.meta)
  
      def load_before(self):
          EventMap.bind("get_member_check_list", self.__get_member_check_list)
          EventMap.bind("send_select_member_list", self.__send_select_member_list)
          EventMap.bind("update_member_info", self.update_member_info)
  
      def load(self):
          self.__load_member_list()
          self.__member_screen_list_create()
          self.__load_group_cur()
          if self.member_list is None or self.member_list == -1 or not len(self.member_list):
              EventMap.send("load_msgbox", "此群组无成员")
              return False
          if self.cur >= 0:
              self.clear_state()
          self.cur = 0
          self.add_state()
  
      def add_state(self):    # 添加选中状态
          currBtn = self.list_menu.get_child(self.curr_idx)
          currBtn.set_style_bg_color(lv.color_make(0xe6, 0x94, 0x10), lv.PART.MAIN | lv.STATE.DEFAULT)
          currBtn.set_style_bg_grad_color(lv.color_make(0xe6, 0x94, 0x10), lv.PART.MAIN | lv.STATE.DEFAULT)
          self.btn_list[self.curr_idx][2].set_long_mode(lv.label.LONG.SCROLL_CIRCULAR)
          currBtn.scroll_to_view(lv.ANIM.OFF)
          
      def clear_state(self):  # 清除选中状态
          currBtn = self.list_menu.get_child(self.curr_idx)
          currBtn.set_style_bg_color(LVGLColor.BASE_COLOR_WHITE, lv.PART.MAIN | lv.STATE.DEFAULT)
          currBtn.set_style_bg_grad_color(LVGLColor.BASE_COLOR_WHITE, lv.PART.MAIN | lv.STATE.DEFAULT)
          self.btn_list[self.curr_idx][2].set_long_mode(lv.label.LONG.SCROLL_CIRCULAR)
          currBtn.scroll_to_view(lv.ANIM.OFF)
  
      # 设置 key2 按键功能
      def key2_once_click(self, event=None, msg=None):
          self.clear_state()
          self.curr_idx = self.next_idx(self.curr_idx, self.count)
          self.add_state()
  
      def key2_long_press(self, event=None, msg=None):
          EventMap.send("close_msgbox")
          EventMap.send("load_screen",{"screen": "MainScreen"})
          if self.curr_idx > 0:
              self.clear_state()
              self.curr_idx = 0
  ```
  
  ##### 创建成员列表并显示成员信息
  
  ```python
  class MemberScreen(Screen):
      ...
      
      def __member_screen_list_create(self):
          """成员界面列表重新创建"""
          # 把之前的list删掉
          if self.member_update_flag:
              self.list_menu.delete()
              # 再创建list
              self.list_menu = lv.list(self.meta)
              self.list_menu.set_pos(0, 40)
              self.list_menu.set_size(240, 200)
              self.list_menu.set_style_pad_left(0, 0)
              self.list_menu.set_style_pad_right(0, 0)
              self.list_menu.set_style_pad_top(0, 0)
              self.list_menu.set_style_pad_row(1, 0)
              self.list_menu.add_style(CommonStyle.container_bgffffff, lv.PART.MAIN | lv.STATE.DEFAULT)
              self.list_menu.add_style(MainScreenStyle.list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.DEFAULT)
              self.list_menu.add_style(MainScreenStyle.list_scrollbar, lv.PART.SCROLLBAR | lv.STATE.SCROLLED)
              if self.count:
                  self.add_member_msg(0, self.count)
                  self.member_update_flag = False
              else:
                  EventMap.send("load_msgbox", "无成员")
  
      def add_member_msg(self, index, end):
          self.member_btn_list = []
          for each in self.member_list[index:end]:
              btn = lv.btn(self.list_menu)
              btn.set_pos(20, 0)
              btn.set_size(240, 47)
              btn.add_style(MainScreenStyle.btn_group, lv.PART.MAIN | lv.STATE.DEFAULT)
              img = lv.img(btn)
              img.align(lv.ALIGN.LEFT_MID, 10, 0)
              img.set_size(32, 32)
              img.set_src('U:/img/number_{}.png'.format(each[4] + 1))
              lab = lv.label(btn)
              lab.align(lv.ALIGN.LEFT_MID, 50, 13)
              lab.set_size(210, 40)
              lab.set_text(each[1])
              self.btn_list.append((btn, img, lab))
          self.add_state()
  
      def __load_group_cur(self):
          ret = EventMap.send("get_group_name")
          if ret:
              print(ret)
              EventMap.send("load_msgbox", '当前群组: {}'.format(ret))
              self.msgbox_close_timer.start(self.msgbox_close_time * 1000, 0, lambda arg: EventMap.send("close_msgbox"))
  ```
  
- #### 加载 UI 屏幕

  PocUI 类通过事件驱动的方式加载指定的屏幕，并处理屏幕切换时的资源管理和状态更新。
  具体功能如下：
  
  1. 屏幕切换：根据传入的屏幕名称（`msg["screen"]`），从屏幕列表中找到对应的屏幕并加载。
  2. 资源管理：在加载新屏幕前，释放当前屏幕的资源并更新状态。
  3. 状态栏显示：如果加载的屏幕不是欢迎界面（`WelcomeScreen`），则显示状态栏。
  4. 屏幕生命周期管理：调用屏幕的 `load_before()`、`load()` 和 `load_after()` 方法，确保屏幕的初始化逻辑正确执行。
  5. 图像缓存优化：在屏幕加载后，刷新图像缓存并设置缓存大小，以优化性能。
  
  该类是 UI 屏幕加载的核心逻辑，确保界面切换的流畅性和资源的高效管理。

  ```python
  class PocUI(AbstractLoad):
      ...
      
      def load_screen(self, event, msg):
          """
          加载UI屏幕
          """
          for scr in self.screen_list:
              if scr.NAME != msg["screen"]:
                  continue
              if self.curr_screen:
                  if scr.NAME != self.curr_screen.NAME:
                      scr.set_last_screen(self.curr_screen.NAME)
                  self.curr_screen.deactivate()
              self.curr_screen = scr
              
              PrintLog.log("PocUI", "load screen:{}".format(scr.NAME))
  
              # 加载屏幕之前先加载屏幕栏
              if self.curr_screen.NAME != "WelcomeScreen":
                  EventMap.send("menubar__show", self.curr_screen.meta)
  
              scr.load_before()
              scr.load()
              scr.load_after()
              lv.img.cache_invalidate_src(None)
              lv.img.cache_set_size(8)
              lv.scr_load(self.curr_screen.meta) # load lvgl meta object
  ```

### 按键模块

- #### 功能描述

  通过按键实现屏幕的滚动、选择，以及开启对讲服务。

  - KEY1
    长按：开启对讲服务。
  - KEY2
    单击：选择框往下滚动；
    双击：进入所选择屏幕；
    长按：返回上一级界面。

- #### 实现原理

  ```python
  class KeyManger(object):
  
      def __init__(self):
          ...
          
          # 按键中断初始化
          self.key1 = ExtInt(ExtInt.GPIO13, ExtInt.IRQ_RISING_FALLING, ExtInt.PULL_PU, self.__key1_event_handler)
          self.key1.enable()   # key1
  
          self.key2 = ExtInt(ExtInt.GPIO12, ExtInt.IRQ_RISING_FALLING, ExtInt.PULL_PU, self.__key2_event_handler)
          self.key2.enable()   # key2
  
      def __key1_event_handler(self, event):
          if event[1] == 1:
              self.__key1_press_handle()
          else:
              self.__key1_up_handle()
  
      def __key1_press_handle(self):
          self.__key1_long_timer.start(500, 0, self.__key1_long_handle)
  
      def __key1_long_handle(self, arg):
          self.__key1_long_timer_flag = True  # key1键 长按标志
          EventMap.send("ppt_press")
  
      def __key1_up_handle(self):
          self.__key1_long_timer.stop()
  
          if self.__key1_long_timer_flag:
              self.__key1_long_timer_flag = False
              EventMap.send("ppt_release")
              return
  
      def __key2_event_handler(self, event):
          if event[1] == 1:
              self.__key2_press_handle()
          else:
              self.__key2_up_handle()
  
      def __key2_press_handle(self):
          self.__key2_long_timer.start(1500, 0, self.__key2_long_handle)
  
      def __key2_long_handle(self, arg):
          self.__key2_long_timer_flag = True  # key2键 长按标志
          EventMap.send("key2_long_press")
  
      def __key2_up_handle(self):
          """key2键 抬起"""
          self.__key2_long_timer.stop()
  
          if self.__key2_long_timer_flag:
              self.__key2_long_timer_flag = False
              return
          self.__key2_count += 1
  
          # 判断是否准备双击
          if not self.__key2_double_timer_flag:   
              self.__key2_double_timer_flag = True
              self.__key2_double_timer.start(300, 0, self.__key2_up_timer)
  
      def __key2_up_timer(self, args):
          if 2 <= self.__key2_count:
              EventMap.send("key2_double_click")
          else:
              EventMap.send("key2_once_click")
          self.__key2_count = 0
          self.__key2_double_timer_flag = False
  ```

### 事件管理

- EventMap 类通过维护一个事件映射表（`__event_map`），实现事件的发送和绑定，支持同步和异步两种消息发送模式。
  具体功能如下：
  
  1. 事件绑定：通过 `bind()` 方法将事件名称与回调函数关联，存储在 `__event_map` 中。
  2. 事件解绑：通过 `unbind()` 方法移除指定事件的回调函数。
  3. 事件发送：
     - 同步发送（`MODE_SYNC`）：在当前线程中直接执行回调函数，并返回执行结果。
     - 异步发送（`MODE_ASYNC`）：在新线程中执行回调函数，不阻塞当前线程。
  
  4. 错误处理：在执行回调函数时捕获异常，并记录错误日志（如果启用了日志功能）。
  5. 日志记录：支持记录事件的执行信息，便于调试和问题排查。
  
- 该功能模块是系统中各个模块之间通信的核心，确保事件能够高效、可靠地传递和处理。
  
  ```python
  class EventMap(object):
      """===example===
  
      import EventMap
  
      def time_out(event=None, msg=None):
          pass
          
      EventMap.bind("time_out", time_out)
  
      EventMap.send("time_out")
      """
      __event_map = dict()
      __event_log = None
  
      MODE_SYNC = 0
      MODE_ASYNC = 1
  
      @classmethod
      def bind(cls, event, callback):
          """
          :param event: event name
          :param callback: event callback
          """
          if None == event or "" == event:
              return
          cls.__event_map[event] = callback
      
      @classmethod
      def unbind(cls, event):
          """
          :param event: event name
          """
          if None == event or "" == event:
              return
          cls.__event_map.pop(event, None)
          
      @classmethod
      def send(cls, event, msg=None, mode=MODE_SYNC):
          """
          :param event: event name
          :param msg: event message
          :param mode: send mode, sync or async
          """
          if event not in cls.__event_map:
              return
  
          if cls.MODE_SYNC == mode:
              res = None
              try:
                  if event in cls.__event_map:
                      res = cls.__event_map[event](event, msg)
              except Exception as e:
                  if cls.__event_log:
                      cls.__event_log.info("ERROR executed (event) -> {} (params) -> {} (result) -> {}".format(event, msg, res))
                  usys.print_exception(e)
              if cls.__event_log:
                  cls.__event_log.info("SYNC executed (event) -> {} (params) -> {} (result) -> {}".format(event, msg, res))
              return res
              
          elif cls.MODE_ASYNC == mode:
              try:
                  _thread.start_new_thread(cls.__event_map[event], (event, msg))
              except Exception as e:
                  if cls.__event_log:
                      cls.__event_log.info("ERROR executed (event) -> {} (params) -> {} (result) -> {}".format(event, msg, res))
                  usys.print_exception(e)
              if cls.__event_log:
                  cls.__event_log.info("ASYNC executed (event) -> {} (params) -> {} (result) -> {}".format(event, msg, None))
  
  ```

### 主程序

- APP 类采用模块化设计，通过统一的管理接口实现各组件的动态加载和生命周期管理。
  具体功能如下：
  
  1. 核心组件管理：
     - 管理 UI 界面（`set_ui()`）、按键模块（`add_key()`）、状态栏（`add_bar()`）、消息框（`add_msgbox()`）和屏幕（`add_screen()`）
     - 通过类型检查确保添加的组件符合 AbstractLoad 抽象类规范
  
  2. 服务管理：
     - 通过 `add_service()` 添加后台服务（如网络服务、音频服务）
     - 调用服务的 `instance_after()` 进行初始化，维护服务列表（`__service_list`）
  
  3. 应用启动：
     - 在 `exec()` 方法中依次启动 UI 组件（`__ui.start()`）和所有服务
     - 调用 LVGL 的任务处理器（`lv.task_handler()`）维持 GUI 运行
  
- 该类的设计实现了应用程序的模块化架构，为大型嵌入式 GUI 应用提供了可扩展的框架基础。

  ```python
  class App(object):
      __service_list = []
      __ui = None
      __key = None
  
      @classmethod
      def set_ui(cls, ui):
          cls.__ui = ui
  
      @classmethod
      def add_key(cls, key):
          cls.__key = key
  
      @classmethod
      def add_bar(cls, bar:AbstractLoad):
          """
          这里只负责向UI添加屏幕栏, 屏幕栏由UI进行管理
          """
          try:
              if isinstance(bar, AbstractLoad):
                  cls.__ui.add_bar(bar)     
          except Exception as e:
              raise Exception("[App](abort) add_bar error: ", e)
          return cls
  
      @classmethod
      def add_msgbox(cls, msgbox:AbstractLoad):
          """
          这里只负责向UI添加消息框, 消息框由UI进行管理
          """
          try:
              if isinstance(msgbox, AbstractLoad):
                  cls.__ui.add_msgbox(msgbox)     
          except Exception as e:
              raise Exception("[App](abort) add_msgbox error: ", e)
          return cls
  
      @classmethod
      def add_screen(cls, screen:AbstractLoad):
          """
          这里只负责向UI添加屏幕, 屏幕由UI进行管理
          """
          if None == cls.__ui:
              raise Exception("UI is None.")
          try:
              if isinstance(screen, AbstractLoad):
                  cls.__ui.add_screen(screen)    
          except Exception as e:
              raise Exception("[App](abort) add_screen error: ", e)
          return cls
          
      @classmethod
      def add_service(cls, service:AbstractLoad):
          """
          添加服务
          """
          try:
              if isinstance(service, AbstractLoad):
                  service.instance_after()   # 初始化服务
                  cls.__service_list.append(service)
          except Exception as e:
              raise Exception("[App](abort) add_service error: ", e)
          return cls
  
      @classmethod
      def exec(cls):
          """
          启动App
          """
          if None == cls.__ui:
              raise Exception("[App](abort) exec interrupt, UI is null.")
          try:
              # start ui
              cls.__ui.start()
              
              import lvgl as lv
              lv.task_handler()
              
              # start services
              for service in App.__service_list:
                  service.load_before()
                  service.load()
                  service.load_after()
          except Exception as e:
              print("[App] exec error: ", e)
  ```

## UML 类图

### 服务模块

`services.py` 定义了多个 `service`，用于提供各种服务。这些 `servies` 继承 `AbstractLoad` 抽象加载基类，便于在加载过程中提供各类服务事项。

1. `BatteryManger`：提供电池电量管理
2. `DevInfoService`：提供设备信息服务
3. `MediaService`：提供音频服务
4. `NetService`：提供网络服务
5. `PocService`：提供 Poc 对讲服务

各 `service` 之间的关系如下图：

![](../../media/solutions/poc/services.png) 

> 如用户需添加 `service`，可参考已有 `service` 样式进行添加，并添加到 `poc_main.py` 中对应的位置即可。

### 界面模块

在 `ui.py` 中，定义了多个 UI 界面，如：

1. `PocUI`：主 UI，提供 `MenuBar`、`PromptBox` 和 `Screen` 的管理以及按键事件的响应处理
2. `MenuBar`：菜单栏（用于显示网络状态、时间、电量以及其他图标，一直显示在屏幕上方，大小为 240×20）
3. `PromptBox`：消息提示框（用于消息提示，显示在当前 UI 界面之上）
4. `Screen`：UI 屏幕，也可以理解为 UI 界面，用于展示给用户看的各种界面。如 `GroupScreen`、`WelcomeScreen` 和 `MemberScreen` 等

各 UI 界面之间的关系如下图：

![](../../media/solutions/poc/ui_uml.png) 

> 如用户需添加 `Screen`，可参考已有 `Screen` 样式进行添加，并添加到 `poc_main.py` 中对应的位置即可。
