# Button

In terms of embedded device, one physical button is needed to interact with user. As a result, the user can do basic operations correspondingly: when the button is pressed, this behavior will be detected by system; When the button is pressed for given time (i.e. more than 1s), it will be considered as long-press. However, when the button is released, the system will detect and report this status. 



## Design Logic 

### HW (hardware)

In condition that the button input design in embedded system is involved, please take following vital components into consideration so as to guarantee the reliability and persistence of system. 

1. **Pull up/pull down Resistor**
   - Most of micro-controllers are embedded with inner pull up/down resistor, which is capable to ensure one definite level when the GPIO pin is floating. 
   - Under the circumstance that the button is not pressed, if it is configured as inner pull up, the GPIO pin will be pulled to VCC for sake of maintaining in high level. 
   - When the button is pressed, the GPIO pin is connected to GND correspondingly so as to switch to low level. 
   - It is also vital to select appropriate value of pull up/down resistor. Normally, the inner pull up/down resistor of micro-controller will be in 10kΩ~100kΩ. 
2. **GPIO Configuration**:
   - Select one suitable GPIO pin and configure it as input. 
   - Connect button to designated GPIO, GND or VCC. 
   - It is a must to ensure the selected GPIO pin is not endowed with other vital function or backup function such as UART and I2C. 
3. **Power Supply & GND Connection **:
   - Please guarantee the other side of button is connected to GND or VCC, which is determined by its design and pull up/down resistor used. 
   - Make sure the power and GND connection are stable and reliable since the button may be operated frequently. 
4. **Bounce and Debounce**: 
   - Even though it is capable to debounce via SW. However, in certain situation, it would be much better via HW. 
   - In terms of debounce in HW, one small volume of capacitor and resistor will be used as RC filter for sake of eliminating high frequency bounce caused by button. 
   - Select adequate resistor and capacitor in accordance with RC time constant, which is ranged 1ms~10ms normally. 
5. **ESD**:
   - The physical button may be a path for ESD, especially in dry environment.
   - One small capacitor with 1nF between GPIO pin and button can do a favor in absorbing and dissipating ESD to decrease the influence to micro-controller. 
6. **Button Selection and Design**:
   - Select suitable button type according to application scenario such as micro switch and touch button. 
   - If possible, the button with water-resistant and dust-resistant is suggested to prolong its life span. 

### SW

The button in communication module involves interrupt, timer and FSM (Finite State Machine). See specific description on SW design. 

1. **Interrupt driver**:
   - The interrupt allows micro-controller respond immediately instead of checking or polling button status if it changes. 
   - When the button is pressed or released, the GPIO status will change as well, which triggers interrupt correspondingly. 
   - Compared with polling, the interrupt driver will be more effective since it will deploy CPU resource only when necessary. 
2. **SW debounce**:
   - When pressing or releasing physical button, series of quick state changes will occur, that is called bounce. 
   - The target is to report button state change for only one time. 
   - Normally, one counter will be used: when consecutive and aligned button state changes are detected, the number in counter will be increased. Only when the counter reaches to the threshold can it be considered as effective operation. 
3. **Long press detection**:
   - In some application scenario, it is demanded to detect whether the button is pressed for a long time by user. 
   - Therefore, one timer can be applied: it will start when the button is pressed. If the button is not released before the timer reaches OT, it will be considered as long-press. 
   - This method demands us to calculate when the button is pressed accurately. 
4. **FSM**:
   - It is one structured method to deal with event related to button. 
   - The FSM contains multiple states like "UP", "DOWN" and "LONG_PRESS"
   - when the interrupt is triggered or the OT in timer occurred, the FSM will determine the next status according to current state and latest event. 
   - The FSM enables us to manage and reports various button state and event effectively. 
5. **Event callback**:
   - In order to modularize and reuse SW, it is available to define one callback function for each button event. 
   - Once certain event is detected (I.e. UP, DOWN or LONG_PRESS), the corresponding callback function will be called. 
   - It allows other codes to respond button event without interacting with codes related to button. 
6. **Configuration and parametrization**:
   - In order to make codes more flexible and configurable, it is valid to set some value by relevant parameters and methods such as OT time of long_press or threshold of SW debounce. 
   - Therefore, same codes and different parameter settings can be applied in different applications or scenario. 
7. **Resource management**:
   - Please consider how to make full use of resource in micro-controller such as interrupt, timer and memory. 
   - If there exists more than one button, it is required to share one timer or deploy higher interrupt management tactic probably. 



## Test 

### Scenario Description 

In terms of embedded device, one physical button is needed to interact with user. As a result, the user can do basic operations via it: when the button is pressed, this behavior will be detected by system; When the button is pressed for more than 1s, it will be considered as long-press. However, when the button is unpressed, the system will detect and report this state. 

### SW design logic 

1. **Interrupt Driver**: The interrupt instead of polling in GPIO will be used to detect the state change of button in real time, which is capable to utilize system resource more efficiently. 
2. **SW debounce**: Since the physical button will generate bounce, which will trigger interrupt repeatedly in short time. In order to solve it, one simple counter will be used to debounce. 
3. **Long Press detection**: When the button is pressed, one timer will be initiated. Once surpassed specific time, if the button is still pressed, it can be identified as Long Press. 

### HW design logic 

1. **Pull up resistor**: The internal pull up resistor will ensure the GPIO pin in high level in condition that the button is not pressed. 
2. **GPIO Configuration**: Configure specific GPIO pin as input mode and connect it to physical button. 
3. **Power supply and GND** :Please guarantee the other side of button is connected to GND or power, which is determined by button design (i.e., Normally open or normally closed)

### Script implementation 

API definition 

```python
class Button(gpio,timer_id)
```

- Functionality: Create Button object 
- Return: Button object 
- Button: Button class
- gpio: GPIO number to control LED. For specific, please refer to [machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin).

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

1. The GPIO interrupt will detect the change of button state. 
2. Debounce via SW counter 
3. Detect the OT of long press via timer. 