# Software design

## Software framework

### Software Design Diagram

<img src="../../media/solutions/poc/software_design.png"/> 

- Key module
  - KeyManger: Used to manage key functions.
- Service module
  - DevInfoService: Device Information Service, used to query device IMEI ICCID、 Firmware version information, etc;
  - MediaService: a media service used to manage TTS broadcasts, microphones, and audio processing;
  - NetService: a network service used to manage network status and heartbeat detection;
  - PocService: a POC service used for POC login, group acquisition, member acquisition, and intercom management.
- Interface module
  - MenuBar: Status bar, used to display signals, time, battery level, and intercom icons;
  - PromptBox: a prompt box used to display message pop-up information;
  - Screen: UI screen, used to display various interface information, customizable by users;
  - PocUI: Used to manage user-defined screens.
- Event management module:
  - EventMap: Used for sending and binding events.

### Business system startup process

![](../../media/solutions/poc/start.png) 

## Code explanation

### Core Business Module (POC)

> For API functions related to the POC library, please refer to [POC Public Network Radio](../../../../cn/api_reference/industry/poc.md) Explanation document.

- #### Login

  By registering the ` poc.login() ` callback function, monitor the login status and perform corresponding operations based on the login results.
  The specific functions are as follows:

  1. Login callback: Register a callback function through ` poc.login() ` to listen for login status (` param ` parameter indicates login success or failure).
  2. Successful login processing: If the login is successful (` param==1 `), update the network status to normal and clean or store security data according to the platform type (standard platform or other platform).
  3. Login failure handling: If login fails, start a timer to regularly check the network connection status and mark the network error status.
  4. Network status check: Regularly check the network connection status through a timer to ensure that the device can try logging in again.
  5. Interface update: After successful login, notify the Welcome Screen to update the login status and query group information.

  This functional module ensures that the device can correctly log in to the POC platform and automatically attempt to reconnect in case of login failure.

  ```python
  class PocService(AbstractLoad):
      ...
      
      # login callback
      poc.login(self.__poc_login_cb)
      
      def __poc_login_cb(self, param):
          EventMap.send("welcomescreen__check_cloud_status", param) # After a successful login, the home page will display "Logged in" and proceed to query group information.
          # login
          if param == 1:
              self.net_error = False
              if self.__platform_dict.get(self.__platform) == 'std':
                  self.__securedata_xin_clear()
              else:
                  self.__securedata_xin_store()
          else:
              self.__cloud_check_timer.start(5*1000, 1, lambda arg: self.__check_cloud_connect())  
              self.net_error = True
  ```
  
- #### Enter the group

  By registering the ` poc.register_join_group_cb() ` callback function, listen for device grouping events, and update the interface and play prompt sounds based on the grouping status.
  The specific functions are as follows:
  
  1. Registration callback: Register the callback function through ` poc.register_join_group_cb() ` to listen for whether the device successfully enters the group.
  2. Obtain group information: Use ` poc.group_getbyid() ` to query the detailed information of the current group.
  3. TTS voice broadcast: Based on the membership status and group type (temporary group or regular group), generate corresponding prompt information and broadcast it through TTS voice.
  4. Status management: Update the current group name, login status, and speaking permission status to ensure device status is synchronized with group information.
  5. Temporary group handling: If entering a temporary group, start a timer and automatically exit the temporary group after a specified time.
  
  This functional module ensures that users receive timely voice prompts when entering a group, and correctly handle group switching and status updates.
  
  ```python
  class PocService(AbstractLoad):
      ...
      
      poc.register_join_group_cb(self.__poc_join_group_cb)
      
      def __poc_join_group_cb(self, param):
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
                      tts_msg = "enter into" + "Temporary group" + self.__group_name
                  else:
                      tts_msg = "Join the group" + self.__group_name
              else:
                  tts_msg = self.__get_user_info() + "already logged in" + "Join the group" + self.__group_name
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
  
- #### Get group and member list

  Retrieve group and member data by calling the POC interface, and return the data to the interface layer for rendering.
  The specific functions are as follows:
  
  1. Obtain the group list: Use ` poc.get_groupcount() ` to obtain the number of groups that the current account has joined, and use ` poc.get_grouplist() ` to obtain the group list data.
  2. Get member list: Use ` poc.group_getbyid() ` to obtain the current group information, and then use ` poc.get_comembercount() ` and ` poc. get_comemberlist() ` to obtain the number of members and member list data.
  3. Exception handling: If the group or member data is invalid (such as a return value of -1 or a quantity of 0), empty data will be returned, and the interface layer will display the corresponding prompt message.
  
  This functional module provides data support for group management and member list interfaces, ensuring that users can view and manipulate group and member information.
  
  ```python
  class PocService(AbstractLoad):
      ...
      
      EventMap.bind("group_get_list", self.__get_group_list)
      EventMap.bind("member_get_list", self.__get_member_list)
      
      def __get_group_list(self, event=None, msg=None):
          group_count = poc.get_groupcount()
          group_list = poc.get_grouplist(0, group_count)
          return group_count, group_list
  
      def __get_member_list(self, event=None, msg=None):
          group = poc.group_getbyid(0)
          if -1 == group:
              return -1, None
          member_count = poc.get_membercount(group[0])
          if -1 == member_count or 0 == member_count:
              return -1, None
          member_list = poc.get_memberlist(group[0], 0, member_count)
          return member_count, member_list
  ```
  
- #### POC

  This section is the core functional module of the solution, responsible for managing intercom status, checking network status, and updating interface prompts.
  The specific functions are as follows:
  
  1. Enable intercom: When long press the KEY1 key on the development board, the LCD screen will wake up and the intercom function will be activated. Simultaneously check the network status and current group status to ensure that the intercom function is functioning properly.
  2. Turn off intercom: When the KEY1 key is released, the intercom ends and relevant resources are released, updating the interface status.
  3. Network status check: If the network is abnormal (such as SIM card problems), prompt the user to replace the SIM card.
  4. Group status check: If the current group is invalid, prompt the user to select a valid group.
  5. Interface prompt update: During the intercom process, 'Speaking...' is displayed` Prompt box and update the intercom icon in the status bar.
  6. Audio management: Enable or disable noise reduction function and play prompt sound when intercom is turned on and off.
  
  This functional module ensures the stability of the intercom function and the smoothness of the user experience.
  
  ```python
  class PocService(AbstractLoad):
      ...
      
      EventMap.bind("pocservice__speaker_enable", self.__speaker_enable)
      
      def __speaker_enable(self, event, msg=None):
          PrintLog.log("PocService", "speaker enable: {}".format(msg))
          if msg:
              EventMap.send("poc_play_status", True)  # Wake up LCD
              if self.__speaker_status:
                  EventMap.send("mediaservice__noise_reduction_enable", 1)
                  poc.speak(1)
                  if self.net_error:
                      if 3 != EventMap.send("welcomescreen__get_net_status"):
                          EventMap.send("mediaservice__tts_play", ("Please change the card.", 1)) 
                          EventMap.send("load_msgbox", "Please replace the SIM card.")  
                      return False
                  
                  curr_group = poc.group_getbyid(0)
                  if -1 == curr_group:
                      EventMap.send("mediaservice__tts_play", (self.__group_name_default, 1)) 
                      EventMap.send("load_msgbox", self.__group_name_default)
                  else:
                      if not self.__rocker_arm:
                          EventMap.send("update_session_info", "You have been blocked from posting. You have been blocked from posting.")
                      else:
                          EventMap.send("load_msgbox", "In the speech...")
                          EventMap.send("menubar__update_poc_status", 1)
                          
              else:
                  EventMap.send("mediaservice__audio_tone")
              return True
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
  
- #### Call callback from the other party

  By registering the ` poc.register_audio_cb() ` callback function, listen for the other party's call information, and update the device status and interface prompts based on the call status.
  The specific functions are as follows:
  
  1. Register audio callback: Register the callback function through ` poc.register_audio_cb() ` to listen for the other party's call information (` params ` parameters include voice status, user ID, username, and interrupt flag).
  2. Call status processing:
    - If the other party is calling (` params[0]==self.BAND_CAL `), update the device status to 'active call'.
    - If the other party starts playing voice (` params[0]==self.BND_LISTN_START `), update the device status to "call ended" and set the speaking permission based on the interrupt flag.
    - If the other party stops playing voice (` params[0]==self.BND_LISTN_STOP ` or ` params[0]==self.BND_SPEAK-STOP `), interrupt the logic and update the device status.
  3. Interface update: When the other party calls, display a message prompt box, wake up the LCD screen, and update the intercom icon in the status bar.
  4. Status management: Update the calling status, speaking status, and session information of the device based on the call status, ensuring that the device status is synchronized with the call information.
  
  This functional module ensures that the device can correctly process the call information of the other party and update the interface and status in real time.
  
  ```python
  class PocService(AbstractLoad):
      ...
      
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
                  self.__speaker_status = 0
              else:
                  self.__speaker_status = 2
              self.__session_info = params[2]
              state_msg = self.__session_info    
              EventMap.send("load_msgbox", state_msg)
              EventMap.send("poc_play_status", True)
              EventMap.send("menubar__update_poc_status", 2)
              EventMap.send("pocservice__call_member_status", 1)
  
          elif params[0] == self.BND_LISTEN_STOP or params[0] == self.BND_SPEAK_STOP:
              if params[0] == self.BND_LISTEN_STOP and self.main_call_end_state == self.CALL_STATE.IN_CALL:
                  return
              if params[0] == self.BND_LISTEN_STOP:
                  self.__speaker_status = params[-1]
              self.__error_ptt_handler(params)
          else:
              pass
  ```

### UI 

- #### MenuBar

  The MenuBar class updates various components in the status bar by binding and sending events, ensuring that users can view key device information in real-time.
  The specific functions are as follows:
  
  1. Update signal strength: Obtain signal strength through events and display corresponding signal icons and network types (such as 4G).
  2. Update time: Obtain the current time through the event and display it in the status bar.
  3. Update battery level: Obtain the current battery level through events and update the battery level icon.
  4. Update intercom status: Display the corresponding icon based on the intercom status (intercom, playback) and control the visibility of the icon.
  
  The size of the status bar is 240 × 40, located above the LCD screen.
  
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
  
- #### PromptBox

  The PromptBox class displays message content in the form of pop ups and supports dynamic updates and closing of pop ups.
  The specific functions are as follows:
  
  1. Display message pop-up: Create a pop-up based on the incoming message content (` msg `) and metadata (` meta `) and center it on the screen.
  2. Dynamically update messages: If a pop-up window already exists, close the old pop-up window before displaying a new message to ensure the real-time nature of the message.
  3. Close Pop up: Provides the function to close pop ups, release resources, and hide pop ups.
  
  The size of the pop-up window is 180 × 90, and the message content supports automatic line wrapping and is displayed in the center.
  
  ##### Define
  
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
  
  ##### Usage
  
  ```python
  class PocUI(AbstractLoad):
      ...
      
      def load_msgbox(self, event, msg):
          """
          {
              "type": "promptbox",
              "title": "[promptbox]"
              "msg": "hello world",
              "mode": 0
          }
          """
          if isinstance(msg, dict):
              _type = msg.get("type", PromptBox.NAME)
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
          EventMap.send("promptbox__close")
  ```
  
- #### UI Screen

  The Membership Screen class inherits from Screen and is used to display member information on an LCD screen. It has a size of 240 × 200 and, together with the status bar, forms the complete LCD display area.

  The specific functions are as follows:
  
  1. Load member list: Retrieve member list data from the event and dynamically create list items to display on the screen.
  2. Manage selection status: Display the currently selected member item by highlighting the background color and scrolling effect.
  3. Button operation: Supports button events (such as single click, long press) to switch selected items or return to the main interface.
  4. Exception handling: If the member list is empty or there are no members, display a prompt pop-up and return to the main interface.
  5. Interface update: Dynamically update the list content when the member list changes.
  
  The design of this class implements dynamic loading and interactive management of member lists, ensuring that users can easily view and manipulate member information.
  
  ##### Load and add styles
  
  ```python
  class MemberScreen(Screen):
      NAME = "MemberScreen"
  
      def __init__(self):
          ...
  
          self.meta = lv.obj()    # lvgl meta object
          self.meta.add_style(CommonStyle.default, lv.PART.MAIN | lv.STATE.DEFAULT)
          # list------------------------------------------------------------------------------------------
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
              EventMap.send("load_msgbox", "This group has no members.")
              return False
          if self.cur >= 0:
              self.clear_state()
          self.cur = 0
          self.add_state()
  
      def add_state(self):
          currBtn = self.list_menu.get_child(self.curr_idx)
          currBtn.set_style_bg_color(lv.color_make(0xe6, 0x94, 0x10), lv.PART.MAIN | lv.STATE.DEFAULT)
          currBtn.set_style_bg_grad_color(lv.color_make(0xe6, 0x94, 0x10), lv.PART.MAIN | lv.STATE.DEFAULT)
          self.btn_list[self.curr_idx][2].set_long_mode(lv.label.LONG.SCROLL_CIRCULAR)
          currBtn.scroll_to_view(lv.ANIM.OFF)
          
      def clear_state(self):
          currBtn = self.list_menu.get_child(self.curr_idx)
          currBtn.set_style_bg_color(LVGLColor.BASE_COLOR_WHITE, lv.PART.MAIN | lv.STATE.DEFAULT)
          currBtn.set_style_bg_grad_color(LVGLColor.BASE_COLOR_WHITE, lv.PART.MAIN | lv.STATE.DEFAULT)
          self.btn_list[self.curr_idx][2].set_long_mode(lv.label.LONG.SCROLL_CIRCULAR)
          currBtn.scroll_to_view(lv.ANIM.OFF)
  
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
  
  ##### Create a member list and display member information
  
  ```python
  class MemberScreen(Screen):
      ...
      
      def __member_screen_list_create(self):
          """Recreation of the member interface list"""
          # Delete the previous list
          if self.member_update_flag:
              self.list_menu.delete()
              # Create another list
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
                  EventMap.send("load_msgbox", "No members")
  
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
              EventMap.send("load_msgbox", 'The current group: {}'.format(ret))
              self.msgbox_close_timer.start(self.msgbox_close_time * 1000, 0, lambda arg: EventMap.send("close_msgbox"))
  ```
  
- #### load UI screen

  The PocUI class loads specified screens in an event driven manner and handles resource management and status updates during screen switching.
  The specific functions are as follows:
  
  1. Screen switching: Based on the input screen name (` msg ["screen"] `), find the corresponding screen from the screen list and load it.
  2. Resource management: Release the resources of the current screen and update the status before loading a new screen.
  3. Status bar display: If the loaded screen is not a Welcome Screen, the status bar will be displayed.
  4. Screen lifecycle management: Call the screen's ` load_before() `, ` load() `, and ` load_after() ` methods to ensure that the initialization logic of the screen is executed correctly.
  5. Image cache optimization: After screen loading, refresh the image cache and set the cache size to optimize performance.
  
  This class is the core logic of UI screen loading, ensuring smooth interface switching and efficient resource management.

  ```python
  class PocUI(AbstractLoad):
      ...
      
      def load_screen(self, event, msg):
          for scr in self.screen_list:
              if scr.NAME != msg["screen"]:
                  continue
              if self.curr_screen:
                  if scr.NAME != self.curr_screen.NAME:
                      scr.set_last_screen(self.curr_screen.NAME)
                  self.curr_screen.deactivate()
              self.curr_screen = scr
              
              PrintLog.log("PocUI", "load screen:{}".format(scr.NAME))
  
              if self.curr_screen.NAME != "WelcomeScreen":
                  EventMap.send("menubar__show", self.curr_screen.meta)
  
              scr.load_before()
              scr.load()
              scr.load_after()
              lv.img.cache_invalidate_src(None)
              lv.img.cache_set_size(8)
              lv.scr_load(self.curr_screen.meta) # load lvgl meta object
  ```

### key module

- #### Function Description

  Use buttons to scroll, select, and activate intercom services on the screen.

  - KEY1
  Long press: Activate intercom service.
  - KEY2
  Click: scroll down the selection box;
  Double click: Enter the selected screen;
  Long press: Return to the previous interface.

- #### Implementation principle

  ```python
  class KeyManger(object):
  
      def __init__(self):
          ...
          
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
          self.__key1_long_timer_flag = True
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
          self.__key2_long_timer_flag = True
          EventMap.send("key2_long_press")
  
      def __key2_up_handle(self):
          """Key 2: Raise"""
          self.__key2_long_timer.stop()
  
          if self.__key2_long_timer_flag:
              self.__key2_long_timer_flag = False
              return
          self.__key2_count += 1
          
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

### Event management

- The EventMap class maintains an event mapping table (` __event_map `) to enable the sending and binding of events, supporting both synchronous and asynchronous message sending modes.
  The specific functions are as follows:
  - Event binding: Use the ` bind() ` method to associate the event name with the callback function and store it in ` __event_map `.
  - Event unbinding: Remove the callback function of the specified event using the 'unbind()' method.
  - Event sending:
    - Synchronized sending (` MODE_SYNC `): Directly executes callback function in the current thread and returns the execution result.
    - Asynchronous send (` MODE_SYNC `): Execute callback function in a new thread without blocking the current thread.
  - Error handling: Capture exceptions when executing callback functions and record error logs (if logging is enabled).
  - Logging: Supports recording the execution information of events for debugging and troubleshooting purposes.
- This functional module is the core of communication between various modules in the system, ensuring that events can be efficiently and reliably transmitted and processed.

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

### Main program

- The APP class adopts a modular design and implements dynamic loading and lifecycle management of each component through a unified management interface.
  The specific functions are as follows:
  - Core component management:
    - Manage UI interface (`set_ui()`), button module (`add_key()`), status bar (`add_mar()`), message box (`add_msgbox()`), and screen (`add_screen()`)
    - Ensure that the added components comply with the AbstractLoad abstract class specification through type checking
  - Service management:
    - Add backend services (such as network services and audio services) through `add_service()`
    - Call the `instance_after()` of the service to initialize and maintain the service list (`__service_list`)
  - Application startup:
    - Start the UI component (`ui.start()`) and all services in sequence in the `exec()` method
    - Call LVGL's task handler (` lv.task_handler() `) to maintain GUI operation
- The design of this class implements a modular architecture for applications, providing an extensible framework foundation for large-scale embedded GUI applications.

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
        This is only responsible for adding screen bars to the UI, which are managed by the UI
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
        This is only responsible for adding message boxes to the UI, which are managed by the UI
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
        This is only responsible for adding screens to the UI, which is managed by the UI
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
        add service
        """
        try:
            if isinstance(service, AbstractLoad):
                service.instance_after()   # Initialization service
                cls.__service_list.append(service)
        except Exception as e:
            raise Exception("[App](abort) add_service error: ", e)
        return cls

    @classmethod
    def exec(cls):
        """
        startup App
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

## UML

### Service module

`services.py` defines multiple services to provide various services. These`services` inherit the `AbstractLoad` abstract loading base class, making it easier to provide various service items during the loading process.

1. `BatteryManger`: Provides battery level management
2. `DevInfoService`: Provides device information services
3. `MediaService`: Provides audio services
4. `NetService`: Provides network services
5. `PocService`: Provides PoC intercom service

The relationship between each 'service' is shown in the following figure:

![](../../media/solutions/poc/services.png) 

> If users need to add a `service`, they can refer to the existing `service` style and add it to the corresponding position in `poc_main.py`.

### Screen module

In `ui.py`, multiple UI interfaces are defined, such as:

1. `PocUI`: Main UI, providing `MenuBar` , `PromptBox` and `Screen`, and Response Handling of Key Events
2. `MenuBar`: Menu bar (used to display network status, time, battery level, and other icons, always displayed at the top of the screen, with a size of 240 * 20)
3. `PromptBox`: Message prompt box (used for message prompts, displayed on the current UI interface)
4. `Screen`: UI screen, also known as UI interface, is used to display various interfaces to users. Such as `GroupScreen`,  `WelcomeScreen` and `MemberScreen`, etc

The relationship between various UI interfaces is shown in the following figure:

![](../../media/solutions/poc/ui_uml.png) 

> If users need to add a `Screen`, they can refer to the existing `Screen` style and add it to the corresponding position in `poc_main.py`.
