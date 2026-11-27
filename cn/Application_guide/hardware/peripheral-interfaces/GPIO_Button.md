# Key - 按键

在一个嵌入式设备中，有一个物理按键用于用户交互。用户可以通过这个按键执行一些基本操作。当按键被按下时，系统会检测到这个行为；如果按键被持续按下超过特定的时间（如1秒），则视为长按。当按键被释放时，系统会识别并报告这个状态。 



## 设计思路

### 硬件

当涉及到嵌入式系统的按键输入设计时，需要考虑几个关键的硬件设计元素和挑战，以确保系统的可靠性和持久性。 

1. **上拉/下拉电阻**:
   - 很多微控制器都提供了内部的上拉或下拉电阻，这可以确保GPIO引脚在浮空状态时有一个确定的电平。
   - 当按键未被按下时，如果配置为内部上拉，GPIO引脚会被拉至VCC，从而保持高电平。
   - 当按键被按下时，它会将GPIO引脚连接到地，从而使其变为低电平。
   - 选择正确的上拉或下拉电阻值也很重要。一般来说，微控制器的内部上拉/下拉电阻值通常在10kΩ到100kΩ之间，适用于大多数应用。
2. **GPIO配置**:
   - 选择一个合适的GPIO引脚并将其配置为输入模式。
   - 连接按键到选定的GPIO和地或VCC。
   - 需要确保选择的GPIO引脚没有其他重要的功能或备选功能，例如UART、I2C等。
3. **电源和地连接**:
   - 确保按键的另一端正确连接到地或VCC，这取决于按键的设计方式和是否使用上拉或下拉电阻。
   - 要确保电源和地的连接稳固且可靠，因为按键可能会被频繁操作。
4. **抖动与去抖电路**:
   - 虽然可以软件消抖，但在某些情况下，使用硬件去抖电路会更优。
   - 硬件去抖通常涉及到一个小容量的电容和一个电阻，它们一起形成了一个RC低通滤波器，可以消除按键产生的高频抖动。
   - 根据RC时间常数选择合适的电阻和电容值，通常时间常数在1ms到10ms之间。
5. **防止静电放电(ESD)**:
   - 物理按键可能会是静电放电的途径，尤其是在干燥的环境中。
   - 在GPIO引脚和按键之间添加一个小电容（例如1nF）可以帮助吸收和分散ESD，从而减少其对微控制器的影响。
6. **按键选择与设计**:
   - 根据应用场景选择合适的按键类型（如微动开关、触摸按钮等）。
   - 如果可能的话，选择具有一定的防水和防尘功能的按键，以延长其使用寿命。

### 软件

通信模组中的按键处理涉及到中断、计时器和状态机。以下是详细描述的软件设计流程：

1. **中断驱动**:
   - 中断允许微控制器在按键状态发生变化时立即响应，而不必持续检查或轮询按键状态。
   - 当按键被按下或释放时，GPIO的状态会改变，从而触发中断。
   - 与轮询相比，中断驱动的方法更为高效，因为它只在必要时才使用CPU资源。
2. **软件消抖**:
   - 物理按键在被按下或释放时会产生一系列快速的状态变化，这被称为抖动。
   - 软件消抖的目的是确保只报告一次按键的状态变化，忽略抖动。
   - 一个常见的方法是使用一个计数器：当检测到连续的、一致的按键状态变化时，计数器增加。只有当计数器达到某个阈值时，才认为是有效的按键操作。
3. **长按检测**:
   - 某些应用场景需要检测用户是否长时间按住了按键。
   - 为此，我们可以使用一个计时器：当按键被按下时，启动计时器。如果在定时器超时之前按键没有被释放，就认为是长按。
   - 这种方法要求我们能够精确地计算按键被按下的时间。
4. **状态机**:
   - 使用状态机是处理按键事件的一个结构化方法。
   - 状态机可以有多个状态，如“UP”（未按下）、“DOWN”（已按下）和“LONG_PRESS”（长按）。
   - 当中断被触发或计时器超时时，状态机会根据当前状态和新的事件来决定下一个状态。
   - 状态机可以帮助我们有效地管理和报告各种按键状态和事件。
5. **事件回调**:
   - 为了使软件模块化和可重用，可以为每种按键事件定义一个回调函数。
   - 当检测到某个事件（如按下、释放或长按）时，相应的回调函数被调用。
   - 这允许其他部分的代码响应按键事件，而不必直接与按键处理代码交互。
6. **配置与参数化**:
   - 为了使代码更加灵活和可配置，可以提供参数或方法来设置某些值，如长按的超时时间或软件消抖的阈值。
   - 这样，不同的应用或场景可以使用相同的代码，但具有不同的参数设置。
7. **资源管理**:
   - 考虑如何最有效地使用微控制器的资源，如中断、计时器和内存。
   - 例如，如果有多个按键，可能需要共享一个计时器或使用更高级的中断管理策略。



## 实验

### 场景描述

在一个通信模组设备中，有一个物理按键用于用户交互。用户可以通过这个按键执行一些基本操作。当按键被按下时，系统会检测到这个行为；如果按键被持续按下超过特定的时间（如1秒），则视为长按。当按键被释放时，系统会识别并报告这个状态。 

### 软件设计思路

1. **中断驱动**：使用GPIO的中断功能来实时检测按键的状态变化，而不是轮询。这样可以更加高效地使用系统资源。
2. **软件消抖**：由于物理按键可能会产生抖动，导致在短时间内多次触发中断。为了处理这种情况，我们使用了一个简单的软件计数器消抖方法。
3. **长按检测**：当按键被按下时，启动一个定时器，超过特定时间后，如果按键仍被按下，则识别为长按。

### 硬件设计思路

1. **上拉电阻**：使用内部的上拉电阻来确保当按键未被按下时，GPIO引脚保持高电平。
2. **GPIO配置**：将特定的GPIO引脚配置为输入模式，并连接到物理按键上。
3. **电源和地**：确保按键的另一端正确连接到地或电源，取决于按键的设计方式（常开或常闭）。

### 脚本实现

用户接口定义

```python
class Button(gpio,timer_id)
```

- 功能：创建Button对象
- 返回：Button对象
- Button：Button类
- gpio：控制LED的GPIO编号。 详细请参考[machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin)。

```python
import machine
from machine import ExtInt
import time

class Button:
    def __init__(self, pin, timer_id):
        self.pin = machine.ExtInt(pin, ExtInt.IRQ_RISING_FALLING, ExtInt.PULL_PU, self.callback)
        self.pin.enable()
        
        self.counter = 0
        self.prev_state = self.pin.read_level()
        self.debounce_count = 5
        
        self.click_timestamp = None
        self.long_press_time = 1000  # 1 second
        self.state = "UP"
        self.reported_long_press = False
        
        self.timer = machine.Timer(timer_id)
    
    def callback(self, pin):
        # Software counter debounce
        if self.pin.read_level() == self.prev_state:
            self.counter += 1
            if self.counter < self.debounce_count:
                return
        self.counter = 0
        self.prev_state = self.pin.read_level()
        current_time = time.ticks_ms()

        # Button pressed
        if self.pin.read_level() == 0:
            self.click_timestamp = current_time
            self.reported_long_press = False
            print("Button PRESSED")
            
            # Set timer for long press detection
            self.timer.start(period=self.long_press_time, mode=machine.Timer.ONE_SHOT, callback=self.long_press_handler)
            
        # Button released
        else:
            self.timer.stop()  # Cancel long press timer
            
            if self.click_timestamp:
                press_duration = time.ticks_diff(current_time, self.click_timestamp)
                
                if not self.reported_long_press:
                    if press_duration > self.long_press_time:
                        self.reported_long_press = True
                    else:
                        print("Button RELEASED")
                        self.state = "UP"
                self.click_timestamp = None

    def long_press_handler(self, timer):
        if not self.reported_long_press:
            print("Button LONG PRESS")
            self.reported_long_press = True
            self.click_timestamp = None

# Usage
button = Button(4, timer_id=0)  # Button on GPIO 2, Timer 0

```

1. 使用GPIO中断来检测按键状态的变化。
2. 使用软件计数器进行消抖。
3. 使用一个定时器来处理长按的超时检测。