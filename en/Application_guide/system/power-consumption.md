# Power Consumption

## Supported Power Modes of Modules
Cellular communication modules support multiple power modes, each with different power consumption. The common working modes include:

**ACTIVE**: The module is in the state of LTE data transmission, GSM call, or is running RTOS. The power consumption is influenced by specific business and network communication mode. The CPU power consumption and network RF power are different, so the actual power consumption varies greatly under different conditions.

**IDLE**: The module is in the idle state, with the hardware powered on and RTOS running, but no threads need to be executed. It will immediately resume operation when any business is started or a network service is incoming. The ECx00U series modules reduce the clock frequency and enter a light sleep state (turn off the high-speed clock, but the CPU does not sleep) in IDLE mode.

**Sleep**: The sleep mode is based on the premise that the module is in the idle state and autosleep is enabled. In order to reduce power consumption, after entering sleep mode, RTOS pauses, the clock frequency of the module slows down, some peripheral controllers (such as UART, SPI, etc.) are powered off, and only some interrupt controllers are retained.

**PSM**: PSM mode is a power-saving mode specified by 3GPP. In this mode, the module wakes up periodically and performs business operations, and spends the rest of the time in PSM. During PSM mode, the module's behaviors and power consumption are similar to those when the module is sht down.

**Shutdown**: The module is completely powered off. The BB chip and peripheral controllers are completely shut down, but the PMIC is still powered on. It can be awakened by the Powerkey or RTC.

**Supported power modes and power consumption of each modules:**

|                      | ECX00U   | ECX00G  | ECX00N        | ECX00M        | ECX00E  | BG95                | EC21          |
| :------------------- | -------- | ------- | ------------- | ------------- | ------- | ------------------- | ------------- |
| IDLE（LTE FDD@64ms） | 12.34 mA | 10.1 mA | 17.36 mA      | 16.72 mA      | 4 mA    | 18.9 mA(catm@128ms) | 16.9 mA       |
| SLEEP(LTE FDD@64ms)  | 2.05mA   | 1.99 mA | 1.64 mA       | 1.37 mA       | 0.64 mA | 1.89 mA(catm@128ms) | 2.74 mA       |
| SLEEP(CFUN0)         | 1.29mA   | 1.03 mA | 0.96 mA       | 0.8 mA        | 60 μA   | 0.575mA             | 1.00 mA       |
| PSM                  | 15 μA    | 15 μA   | Not supported | Not supported | 5 μA    | 5.10 μA             | Not supported |
| Shut down            | 33 μA    | 15 μA   | 20 μA         | 38 μA         | 3 μA    | 14.87 μA            | 7 μA          |

IDLE refers to the idle state of the module, and shutdown means cutting off the power supply. This article will not elaborate on these two power modes. The sleep mode we often use refers to the sleep and PSM modes are usually referred to in actual use.

## Sleep Mode
Sleep mode is the most commonly used low-power mode of cellular communication modules. When the module is idle, if autosleep is enabled, the module will enter sleep mode. At this time, the module will shut down some IP cores (such as peripheral controllers and interrupt controllers) and reduce the clock frequency to reduce power consumption.

> Autosleep is a flag that controls whether to enter sleep mode when the module is in idle state. Its features and principle will be explained in detail later.

### Sleep Principle of RTOS

The sleep mechanism of RTOS is usually used in conjunction with the sleep mode of the CPU. This mode will shut down the CPU, high-speed clock, and most peripherals (the context of peripherals will be saved before shutdown). At this time, SRAM remains powered on, preserving the running state of the CPU before sleep. Therefore, when RTOS and CPU wake up, they can resume from sleep mode immediately without significant delay.

**Sleep Detection Mechanism:**
During the initialization of RTOS, a task named IDLE with a priority of 0 will be established by default. This means that this task will only be scheduled when no other tasks are running in RTOS. The sleep-related operations are performed in the IDLE task. In this IDLE task, RTOS detects whether the module's peripherals are in use, whether there is data from the network, or other sleep conditions. If the sleep conditions are met and autosleep is enabled, the module will enter sleep mode.

**Sleep Mechanism:**
RTOS's sleep mode generally instructs the CPU to enter sleep mode. At this time, the high-speed clock will be turned off, the peripherals will be powered off after the context is saved, and the ARM kernel will stop running. However, NVIC (interrupt controller) and SRAM can still remain running, and any interrupt can wake up the kernel.

**Wake-up Mechanism:**
Any interrupt can wake up the kernel, and after waking up, the kernel will immediately resume operation. The high-speed clock will resume output, restore the context of the peripherals, and finally resume the execution of the application program(s). This sleep-wake-up mode not only reduces power consumption but also quickly resumes the execution of the application program.

**Sleep Flowchart Example:**

![Sleep](../media/system/power-consumption/RTOSSleep.png)

> Different RTOS may implement sleep logic differently, but the principle is similar.

**Typical Power Consumption Characteristics:**

![consumption_all](../media/system/power-consumption/consumption_all.png)

### Sleep of Cellular Communication Modules

To put QuecPython cellular communication modules into sleep mode, sleep mode needs to be enabled first. If sleep mode is not enabled, the module will remain in IDLE state when idle.

As mentioned in the previous chapter, whether the module enters sleep mode when idle or remains in IDLE state depends entirely on whether autosleep is enabled. In other words, there is no hard obstacle to enter sleep mode when the module is idle, but it is completely controlled by the user. So why not choose to enter sleep mode by default?

The reason is that some peripherals need to be continuously refreshed (such as LCD). However, when the module is in sleep mode, these peripherals cannot work continuously but go through a process of power-off and recovery, which seriously reduces the refresh efficiency and response speed of the peripherals. At the same time, when the module uses a low-speed clock in sleep mode, any device that relies on high-speed clock cannot work properly even if it is not powered off. Therefore, when the module is idle, it will default to remain in IDLE state to maintain the normal operation of all devices.

#### Autosleep Mechanism

Autosleep is essentially a flag that operates in the sleep detection mechanism of RTOS. When autosleep is not enabled, the detection mechanism instructs the module to remain in IDLE state. When autosleep is enabled, the detection mechanism considers that the module is allowed to be in sleep state, and enter sleep.

Please refer to [Set Automatic Sleep Mode](../../api_reference/syslib/pm.md#Set-Automatic-Sleep-Mode)

#### Wakelock Mechanism
In certain scenarios, we need the module to enter a sleep mode while also ensuring the normal operation of specific peripherals within certain code segments. In such cases, we introduce a sleep lock mechanism.

The wakelock is essentially a flag and allows for the creation of multiple wakelocks. The operating principle is as follows: as long as any wakelock is in locked state, the module will not enter sleep mode.

Please refer to [Create Wakelock](../../api_reference/syslib/pm.md#Create-Wakelock).

#### Factors Affecting Cellular Communication Module Sleep

##### Idle Current

The factors affecting idle current consumption are primarily related to the system's clock frequency and the power consumption of activated IP cores, which are also related to the PMIC. 

The factors influencing idle current stem from hardware sources, including the module itself and peripheral components:

- The power consumption factors of the module are: system clock frequency and the activation of IP cores.
- The power consumption factors of the peripherals are: That some peripherals are powered by the module's PMIC, in which case their power consumption is borne by the module.

Generally, when measuring the idle current, the network needs to be set to CFUN0 to disable the RF feature. The current measured at this time is the idle current of the module. Under normal circumstances, its characteristics are generally approximately linear, as shown in the following figure:

![BaseCurrent](../media/system/power-consumption/BaseCurrent.png)

##### DRX Period & Paging
Base station paging: When the network status changes/UE needs to be called/ETWS or CMAS system sends an alert, the base station will initiate paging to the module, notifying the module in IDLE state to start communication.

DRX period: When the module network does not have an RRC connection, it periodically listens to the base station paging to ensure timely response when the base station paging arrives. This listening period is called the DRX period. The DRX period is generally 32 ms, 64 ms, or 128 ms. The shorter the DRX period, the more times the base station signal is listened to within the same time, the higher the power consumption, but the faster the response to paging. Conversely, the longer the DRX period, the lower the power consumption, but the slower the response to paging.

Attached is the current consumption chart in sleep (CFUN1) mode, and the regular peaks in the chart are paging, and the period is called the DRX period:

![Paging](../media/system/power-consumption/Paging.png)

##### Signal Quality
When the signal quality is poor, the module needs a higher transmit power to complete the same network behavior. This will affect the module's power consumption in two scenarios:
1. When performing network services, the current consumption will increase. In addition to the transmit power described above, if the network service experiences reconnection or retransmission due to poor communication quality, the overall service time will be extended, resulting in further increase in current consumption.
2. When the module is in sleep mode, the peak current consumption of each paging will increase, resulting in an increase in overall sleep current.

##### Service Data
When the module is performing network communication, both the RF and CPU are working, resulting in higher current consumption. In actual business scenarios, continuous business data transmission is not always happening. The common business mode is heartbeat packets.
When using a long connection, if there is no communication for a long time, it may be considered offline by the other end, resulting in disconnection. Therefore, in business scenarios, the module sends a packet of data to the other end at a fixed period o keep the long connection alive.
The current consumption data under different heartbeat periods (LTE-FDD@64):

| ECx00U | ECx00G  | ECx00M  | ECx00E  | EC21    | BG95 |
| ------ | ------- | ------- | ------- | ------- | --------- |
| 5 min  | 2.16 mA | 2.13 mA | 1.32 mA | 4.41 mA | 5 mA      |
| 10 min | 1.84 mA | 1.76 mA | 1.03 mA | 3.39 mA | 4.43 mA   |


##### RRC Release Time
Radio Resource Control (RRC) is also called Radio Resource Management (RRM)  or Radio Resource Allocation (RRA). It is a set of independent deployment and management algorithms, including call admission control, handover, power control, channel allocation, packet scheduling, end-to-end QoS guarantee, etc. When the module needs to perform services, it needs to establish an RRC connection with the base station. Only when the RRC connection is released, the module can enter sleep mode.

However, as shown in the figure below, during the lifecycle of this RRC connection, only a few seconds at the beginning are used for actual data services. Between the completion of the service and the release of the RRC connection, there is a period of current consumption that is similar to IDLE but cannot enter sleep mode. The formation of this current consumption is related to the RRC operation strategy of the carrier:

![Paging](../media/system/power-consumption/RRC.png)

To avoid the ping-pong effect (referring to the situation where the network state switches back and forth, causing more system resource occupation and network performance degradation), the base station does not release the RRC connection immediately after the network service is completed. Instead, it waits for a period of time to ensure that the RRC connection does not need to be reestablished when performing service again in a short time. The duration of the RRC connection varies depending on the network environment and carrier strategy. When there is no network data and the RRC is not released, the module does not enter sleep mode. Therefore, the duration of the RRC connection significantly affects the module's power consumption.

### Sources of Waking up from Sleep Mode 
After the cellular module enters sleep mode, there are multiple modes that can wake up the module, including network data, phone calls, GPIO interrupts, timers, etc., and all these modes ultimately wake uo the system through interrupt(s).

####  SoC Wake-up Mechanism
The module can be woken up from sleep by any interrupt (provided it is valid, as some interrupt controllers may be turned off because of power consumption). After waking up, the device context is restored in the critical zone. At this time, the interrupt flag for this interrupt is stored in the interrupt controller, but it is not executed. After the peripheral resumes, the CPU exits the critical zone, and the ISR of the interrupt will immediately run.

#### Network Data & Phone Call Wake-up

Network data & phone calls: The base station notifies the module of a call request through paging, and then the CP (or DSP) wakes up the CPU through a callback and notifies the CPU that there is network data coming. The CPU starts to move the air interface data into the protocol stack buffer, and at this time, the upper-layer application (socket or phone call) can obtain the network data from the protocol stack.

![Network](../media/system/power-consumption/Networkwakeup.png)

#### GPIO Interrupt Wake-up

Prerequisites for GPIO availability: GPIO is kept powered during module sleep, and the interrupt controller connected to it is not turned off.
When the GPIO hardware is triggered, the interrupt controller connected to it responds immediately and wakes up the CPU. The CPU will resume the context of the peripheral and then exit the critical zone. At this time, the ISR detects that the GPIO interrupt has been triggered and immediately executes the GPIO interrupt function (generally, it sends a message to trigger the callback bound to the GPIO), and finally triggers the callback function bound to the GPIO.

![GPIO](../media/system/power-consumption/GPIO.png)

#### Timer Wake-up

When the timer times out, it triggers the interrupt bound to the system timer. The interrupt controller responds immediately and wakes up the CPU. The CPU resumes the context of the peripheral and then exits the critical zone. Finally, the callback bound to the timer is triggered by ISR.

![Timer](../media/system/power-consumption/Timer.png)

#### Wake-up Source Acquisition
In fact, from the above three sources, it can be seen that any wake-up behavior must go through the interrupt controller. It is the characteristic of sleep mode: waking up by any interrupt. Therefore, we can conclude that there is no need to deliberately acquire the wake-up source, as the wake-up behavior corresponds to the triggering of a specific interrupt.

#### Business Processing after Wake-up
As mentioned earlier, there is no need to deliberately acquire the wake-up source, as the wake-up behavior corresponds to the triggering of a specific interrupt. Therefore, the principle of business processing is simple: After the module exits the sleep mode, when the device context is resumed, it exits the critical zone and the ISR of the wake-up interrupt will immediately be executed. The desired business logic can be directly triggered here, usually by sending a message or semaphore in ISR to trigger the execution of the corresponding business.

#### Weak Signal Sleep Solution

When the module is not connected to the network, it will actively search for the network and attempt to connect. This mechanism ensures that the module can attach to the network as quickly as possible, but it also causes power consumption, that is, when the signal is weak or there is no network where the module is located, the module keeps searching for the network, and it cannot enter sleep mode at this time, and the RF power consumption is rather high.

1. OOS (Out of service) mechanism: When the module is powered on, service is interrupted or RF status switches from closed to open,  the module will attempt to camp on the current cell. If the network quality is good, the module will camp on the current cell; if the network quality is poor, the module may fail to camp on the cell. In addition, camping on the cell under poor RF conditions can cause downlink loss, resulting in the module re-camping. The OoS algorithm defines the time for the module to attempt to re-camp. If it fails, the module will enter sleep mode and after a certain period of time, it will attempt to camp on the network again. For a typical OoS mechanism, the sleep duration increases with the attempts of camping on the network increasing (The sleep duration increases and stops at the upper limit, and then repeats the process at the maximum duration). For example:

   Stage 1: Sleep for 30 seconds, attempt to camp on the cell, repeat 10 times.

   Stage 2: Sleep for 45 seconds, attempt to camp on the cell, repeat 10 times.

   Stage 3: Sleep for 60 seconds, attempt to camp on the cell, and repeat this cycle.

   The module's OoS (Out Of Service) mechanism typically attempts network reconnection multiple times before ceasing its efforts. During this period, the RF remains active continuously. Compared to normal network conditions, this mechanism may result in higher power consumption.

2. Automatically implement the process of backoff and reconnection in the service. When the network is not connected within a certain period of time, you can manually stop searching for the network, such as switching the module to enter flight mode, etc. When reconnection is needed, restore the module to full functionality mode. In fact, it is a backoff mode in the service that can control the time and processing logic by itself. The code example is as follows:

```python
import net
import checkNet %%
def check_net_status():
    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1:
        print('Network connection successful.')
    else
	    net.setModemFun(4)

def reconnect():
	net.setModemFun(1)
```



#### Typical Applications

**Network Routine: Network actively sends heartbeat packets & module wakes up by receiving paging from network**

```python
# Use MQTT to implement heartbeat packets and receive paging
from umqtt import MQTTClient
import utime
import net
import sim
import sms
import _thread
import gc
import pm

sub_path = '/a1A5W32fexl/test2/user/Text'
pub_path = '/a1A5W32fexl/test2/user/Text'
state = 0
a = 0
global c

def sub_cb(topic, msg):
    global state
    print("subscribe recv:")
    print(topic, msg)

def err_cb(err):
    print("thread err:")
    print(err)

def wait_msg():
    global c
    while True:
        try:
            c.wait_msg()
        except OSError as e:
            print("wait_msg thread err: %s"%str(e))
            break

global c
c = MQTTClient(
    client_id="a1A5W32fexl.test2|securemode=2,signmethod=hmacsha256,timestamp=1675836667378|",
    server="a1A5W32fexl.iot-as-mqtt.cn-shanghai.aliyuncs.com",
    port=1883,
    user="test2&a1A5W32fexl",
    password="a5882fb77108cbd93f1413a403b31ed06d0c0e97c0ebca4b0b2f8dffe286da77",
    ssl=False,
    keepalive=600) # Keepalive: MQTT keep-alive mechanism. Send a heartbeat packet every 10 minutes.
c.error_register_cb(err_cb)
c.set_callback(sub_cb)
print('set_callback')
pm.autosleep(1) # Enable autosleep
c.connect() # Connect to MQTT, essentially establishing a TCP long connection
print('connect')

print("MQTT is connecting")
c.subscribe(sub_path)
print("Connected to mq.tongxinmao.com, subscribed to %s" % sub_path)
c.publish(pub_path, b"hello")
print("Publish topic: %s, msg: hello" % pub_path)

_thread.start_new_thread(wait_msg, ()) # Listen to MQTT and wait for data to arrive. It is essentially the operations of waiting for paging. No additional operationa are needed if the module is in sleep mode, because the module will be automatically awakened when sending and receiving heartbeat packets.

```

**Hardware Routine 1: GPIO/keypad and other external hardware interrupt wake-up**

```python
import pm
pm.autosleep(1) # Enable autosleep

# Create ExtInt object
from machine import ExtInt
def fun(args):
        f = open("/usr/log.txt", "a+") # Open a file in appending mode to save debugging information when the interactive port cannot be connected during sleep mode.
        f.write('### interrupt  {} ###'.format(args)) # args[0]: GPIO number args[1]: rising edge or falling edge. Write to file
        f.close() # Close the file to save the written information
        
extint = ExtInt(ExtInt.GPIO1, ExtInt.IRQ_FALLING, ExtInt.PULL_PU, fun) # Bind the callback function to GPIO1

extint.enable() # Enable GPIO interrupt

# Trigger GPIO1 after entering sleep mode, then connect USB to check if the file has written the correct information.
# If the correct information is written to the file, it means that the GPIO interrupt effectively wakes up the module in sleep mode and executes its bound interrupt.
```

**Hardware Routine 2: UART wake-up and receive**

```python
"""
To run this routine, you need to connect the MAIN UART of the EVB to the PC via a serial cable, open the MAIN UART on the PC through a serial tool, and send data to the port to see the messages sent by the PC.
"""
import _thread
import utime
import pm
from machine import UART
​```'''
Connect the main port to the serial board and connect it to the PC.
 * Parameter 1: Port
        Note: Select the main port, because the MAIN UART of any modules supports the mechanism of waking up from sleep, but other serial ports have uncertainty
        UART2 - MAIN PORT
 * Parameter 2: Baud rate
 * Parameter 3: Data bits (5–8)
 * Parameter 4: Parity (0: NONE 1: EVEN 2: ODD)
 * Parameter 5: Stop bits (1~2)
 * Parameter 6: Flow control (0: FC_NONE 1: FC_HW)
'''


class Example_uart(object):
    def __init__(self, no=UART.UART2, bate=115200, data_bits=8, parity=0, stop_bits=1, flow_control=0):
        self.uart = UART(no, bate, data_bits, parity, stop_bits, flow_control)
        self.uart.set_callback(self.callback)


    def callback(self, para):
        if(0 == para[0]):
            self.uart.write("UART WAKRUP!")#Send specific data in the serial RX callback  
        

    def uartWrite(self, msg):
        self.uart.write(msg)

    def uartRead(self, len):
        msg = self.uart.read(len)
        utf8_msg = msg.decode()
        return utf8_msg


if __name__ == "__main__":
    uart_test = Example_uart()
    pm.autosleep(1)


# After entering sleep, send data to the main port. If "UART WAKRUP!" is returned, the serial port wakes up from sleep successfully and sends data out.
# Note: Some module may lose a packet of data.


```

**Hardware Example 3: Protect hardware timing and status with wakelock when the module wakes up from sleep**

```python
from machine import SPI
import utime
import pm

spi_obj = SPI(0, 0, 1)

if __name__ == '__main__':
    pm.autosleep(1)
    
    lpm_fd = pm.create_wakelock("test", len("test"))#Create a power lock to protect SPI read and write

	pm.wakelock_lock(lpm_fd) #Lock the wakelock and start SPI read and write

    r_data = bytearray(5)  # Create a buffer for receiving data
    data = b"world"  # Test data

    ret = spi_obj.write_read(r_data, data, 5)  # Write data and receive data
    spi_log.info(r_data)
    
	pm.wakelock_unlock(lpm_fd)#Release the wakelock and allow sleep when SPI is not read or written
	
```

#### FAQs
**1. Unable to enter sleep mode normally: Troubleshoot wake-up sources**

| Common Wake-up Sources | Remarks                                      |
| --------------------- | -------------------------------------------- |
| Wakelock        | Unable to sleep if any wakelock is not released. Please release all wakelocks. |
| UART                  | Unable to sleep when there is data transmitting through UART |
| USB                   | Unable to sleep when USB is inserted          |
| Thread                | Unable to enter IDLE when there is thread running |
| SPI                   | Unable to sleep when there is data transmitting through SPI |
| LVGL                  | LVGL keeps refreshing. Please make LVGL sleep first before module sleep |

**2. High idle current after entering sleep normally, check hardware and network environment, including:**

| Power Consumption Source | Remarks                                                      |
| ------------------------ | ------------------------------------------------------------ |
| PWM                      | PWM controller can output with low-speed clock during module sleep |
| GPIO                     | GPIO can maintain external output during sleep to check for leakage |
| VDD_EXT                  | Constant power supply. Need to check for leakage during sleep |

**3. High power consumption due to network environment**

| Factors | Remarks                                            |
| ------- | -------------------------------------------------- |
| RF Power Consumption | Check the peak power consumption and network quality during paging |
| RRC Cycle | Check the time from service completion to RRC link release |

## PSM
### What is PSM

PSM (Power Saving Mode) is a low power consumption mode that has lower power consumption than sleep mode. The hardware principle of PSM is to shut down the module and wake it up with an RTC alarm. Compared with shutdown and wake-up with an RTC alarm, PSM has the following two differences:

1. The wake-up time of the RTC alarm is sent by the network.

2. When entering PSM, although the module is shut down, the core network still retains its registration information. Therefore, waking up from PSM does not require re-attaching to the network, resulting in faster network connection and lower power consumption.

PSM is a mode where the UE stops sending and receiving signals and disables functions related to the AS (Access Stratum) after a certain idle time. It partial disables its functionalities to reduce power consumption in areas such as antennas, RF, and signaling processing.

The advantage of PSM is that it allows for long periods of sleep, but the disadvantage is that it cannot respond to terminal reception (Mobile Terminal, MT) services in a timely manner. It is mainly used in remote meter reading and other scenarios that do not require real-time performance.

### PSM Principle

PSM is a mode with lower power consumption than regular sleep mode, but its operation requires support from the network.

To enable PSM, a request needs to be sent to the base station (carrying timer information through ATTACH or TA_UPDATER). If the base station supports entering PSM, it will send the timer information in the corresponding REQUEST. It should be noted that the actual PSM parameters will use the values sent by the base station, which may not be the same as the values requested by us.

The module will configure the timer time based on the values sent by the base station. When the module enters IDLE and the ACT timer times out, the module will shut down the CPU and all RF functionalities. At this time, it is equivalent to a partial module shutdown, but leaving some wake-up sources on, usually including ACT, TAU timers, and PSM_INT. The power consumption generally drops to the microampere level. Since the base station has recorded that the UE is using PSM, it will not disconnect from the UE. If the next time the UE is powered on in the same cell, it does not need to dial up, but directly use the original IP and other information, thus further reducing power consumption. 

When the PSM TAU timer times out, the RTC alarm expires, or the PSM_INT wake-up source is triggered, the module will be awakened. Since the CPU (including SRAM) and PA are powered off, the module cannot restore the running state before sleep, but needs to go through the startup process again. After startup, network services can be performed, and when the network service is completed and RRC is released, the ACT timer starts counting, and when it times out, it enters PSM.

**Complete PSM Process**

![PSMProcess](../media/system/power-consumption/PSMMainProcess.png)

Finally, under the influence of two timers, the module undergoes sleep and wake transitions periodically. However, it is only during the wake phase that it can engage in network operations; during the sleep phase, it remains unresponsive to network paging.

**Current Consumption Characteristics are as follows (in this example, wake-up occurs every 1 minute):**

![Timer](../media/system/power-consumption/PSMCycle.png)

#### ACT and TAU Timers

ACT, also known as T3324, starts counting when the module completes network operations (i.e., upon RRC connection release). When the timer expires, the module immediately enters PSM.

TAU, also known as T3412, starts counting when the module completes network operations (i.e., upon RRC connection release). When the timer expires,it will wake the module up if the module is in PSM.

Their sequential relationship is as shown below:

![Timer](../media/system/power-consumption/3GPP_PSM_TIMER.png)

Upon receiving the PSM time from the base station, the module immediately sets up an RTC alarm and a timer. The RTC alarm serves as the wake-up source for PSM, and its timeout duration is equal to the value of T3412. In other words, after the period of T3412 time, the module will be awakened by the RTC alarm to enter the next cycle. The timer's timeout duration is set to T3324, and only when it expires is it possible for the module to enter PSM mode. This means that after the RTC alarm wakes the module up, it will remain operational for a duration of T3324.

From this, it is evident that for the module to enter PSM correctly, T3324 must be less than T3412. Otherwise, a situation may arise where the RTC alarm has already timed out, but the module still cannot enter PSM.

### Power Consumption and Module Support in PSM & RTC Mode

|          | EG915U/EG200U series | BG95 series |
| -------- | ------ | ---- |
| PSM      | 15 uA | 5.10 uA |
| RTC+Shutdown | 33 uA | 14.87 uA |

### PSM Wake-up Sources
#### PSM_INT

PSM_INT is the pin that wakes up PSM, which is generally led out from the PMIC. The function of PSM_INT is fixed and does not require additional configuration. After entering PSM, it can be triggered to wake up the module according to the specified method.

#### RTC Alarm

The RTC alarm can wake up the module from PSM. Its usage is the same as that of the shutdown RTC alarm when module shutdown .

Refer to: [RTC API Reference Manual](../../api_reference/peripherals/machine.RTC.md#Constructor)

#### Powerkey

The powerkey can also wake up the module from PSM, and its working principle is the same as that of triggering startup.

#### TAU Timer

As described above, the module will be awakened when the TAU timer times out.

> Note: The TAU Timer and RTC alarm of BG95 series module share the same PMIC wake-up source, so PSM and RTC cannot be used at the same time.

### Typical Applications of PSM

Although PSM has low power consumption, it has the following disadvantages:

1. When PSM starts, it needs to go through the restart logic, and the application recovery time is much longer than autosleep.

2. When a cell switch occurs, the original network information cannot be obtained, resulting in network disconnection and re-registration issues, which may increase power consumption instead.

3. The module does not respond to paging during PSM.

Therefore, the application scenarios of PSM generally meet the following three conditions:

1. The interval between timed tasks is long enough to ignore the startup time.

2. There is less mobility and frequent cell switching does not occur.

3. Real-time data requirements are not high.

**Example**

```python
import utime
import pm
from machine import RTC
from misc import Power
import checkNet

def Business_code_example(run_time):
    i = 0
    for i in range(run_time):	
        print("Business app running")
		#Business code here  
        utime.sleep(1)

    return    

def psm_try_set():
    if pm.get_psm_time()[0] == 1: # Check if PSM has been enabled during startup. If yes, there is no need to enable it again
        print("PSM has been enabled, set pass")
        return 0
    else:
        return pm.set_psm_time(0,1,0,1) #T3412=10min T3324=1min

def psm_failed_handle(delay_time):
    utime.sleep(delay_time) # After waiting for the specified time, if the module still does not enter PSM, continue running. Here it executes the PSM failure handling logic, i.e., shutdown + RTC shutdown alarm

    rtc = RTC()
    tm_rtc_tuple = rtc.datetime()
    tm_rtc_second = utime.mktime((tm_rtc_tuple[0], tm_rtc_tuple[1], tm_rtc_tuple[2], tm_rtc_tuple[4], tm_rtc_tuple[5], tm_rtc_tuple[6], 0, 0))

    alarm_second = tm_rtc_second + 600 # Set the RTC alarm to the current time + 10min
    alarm_tuple = utime.localtime(alarm_second)

    rtc.set_alarm([alarm_tuple[0], alarm_tuple[1], alarm_tuple[2], alarm_tuple[6], alarm_tuple[3], alarm_tuple[4], alarm_tuple[5], 0])
    rtc.enable_alarm(1)

    Power.powerDown()


if __name__ == '__main__':
    psm_failed_delay_time = 60 + 30 # After T3324 of PSM times out for 30 seconds, the error handling logic will be enabled
    lpm_fd = pm.create_wakelock("psm_lock", len("psm_lock")) # Apply for a wakelock
    
stage, state = checkNet.waitNetworkReady(30)
if stage == 3 and state == 1:
    print('Network connection successful.')
    psm_try_set() # Try to set PSM when network connection is successful. If PSM is already set, ignore this step.
else:
    print('Network connection failed.')
    psm_failed_delay_time = 1 # Use RTC instead of PSM when there is no network connection

pm.wakelock_lock(lpm_fd) # Lock the wakelock to prevent entering PSM in the scenario that sleep mode should be enabled in the business running process
Business_code_example(10) # Run business code
pm.wakelock_unlock(lpm_fd) # Release the wankelock after running the business code. The module will automatically enter PSM after being idle and T3324 timeout

psm_failed_handle(psm_failed_delay_time) # Run error handling. If the module can enter PSM normally, it will enter PSM during sleep and this logic will not be executed
```

[Click here to download the complete code from GitHub](https://github.com/QuecPython/examples/blob/main/system/power-consumption/psm_example.py)

#### How to Enter PSM

PSM can be used when the module is connected to the network and the operator supports PSM. The ACT and TAU periods can be set through API according to the business requirements:

Please refer to [PSM API Documentation](../../api_reference/syslib/pm.md#Set-PSM-Time)

#### PSM_INT Application

BG95 series module supports PSM_INT, which triggers on the rising edge. When entering PSM, the module's PMIC will configure this pin by default to ensure the trigger mode is correct for waking up the module in PSM.

#### Weak Signal PSM Solution

PSM may not work effectively under weak signal conditions because the connection with the base station may not be established properly, and it is uncertain whether the TAU and ATC timers for requesting and receiving data are functioning correctly. 

In this case, we can handle it actively in the business logic. One possible solution is to set a maximum time for attempting to connect to the network. If the network connection is still not successful after this time period, switch to using shutdown + RTC alarm to achieve the desired functionality.

#### Solution for Operators That Do Not Support PSM

If the operator does not support PSM, the base station will try to prevent the module from entering PSM. There are three common behaviors:

1. Reject the module's request for timers when requesting timers from the base station (this situation is rare, and most cases fall into the next two behaviors).
2. Issue mismatched ACT and TAU timers that cannot trigger PSM.
3. Do not issue timers.

In this case, the only option is to use RTC alarm to wake up the module from shutdown in order to implement similar business. Besides, this option can reduce current consumption during which no data interaction is needed. However, compared to PSM, this method of powering on the module after shutdown  requires an additional network registration and dialing process, resulting in higher RF power consumption each time the module wakes up.

### Common Troubleshooting

**1. Set to PSM and True is returned but the module cannot enter PSM**

The base station may not support PSM. To confirm this, check the information sent by the base station in the CP log. The items to check are:

**ATTACH_REQUEST**

**ATTACH_ACCEPT**

**TA_UPDATE_REQUEST**

**TA_UPDATE_ACCEPT**

In the example below, after the module is set successfully, the REQUEST always contains paired timer information. Pay attention to whether the ACCEPT contains the correct timer information sent by the base station.

![Timer](../media/system/power-consumption/PSMRequest.png)

In the example below, PSM cannot be entered because the network environment does not support PSM and the base station only sends only one ACT timer instead of a paired TAU timer:

![Timer](../media/system/power-consumption/PSMWrongACCEPT.png)

**2. The module successfully enters PSM, but the sleep and wakeup cycle is different from the set value**

Similarly, check the above CP log items. The base station may issue timers with periods different from the set value, and the module will use the values issued by the base station as the actual values.

## Common Application Scenarios
### Sleep Solution for Meters and Door Contact Sensors

PSM: Products such as meters and door contact sensors have long wake-up intervals and are generally not frequently moved. Real-time data is not required. PSM can be used in this scenario. If network connection is not available or the operator does not support PSM, a shutdown + RTC alarm solution can be used.



### Selecting Modules Based on Power Consumption Requirements

uA-level current consumption: Choose a module that supports PSM or supports the solution of shutdown +RTC alarm.



mA-level current consumption: All modules support autosleep. Evaluate suitable modules based on other requirements.



### Calculation of Standby Time

1. Determine the battery type and get to know its battery capacity.

2. Determine the module model and calculate the average standby current based on the specifications and actual business requirements. To obtain accurate standby current, the device should be operated according to the business process, and the average standby current should be measured with a power analyzer.

3. Calculate the standby time using the formula: Standby Time = Battery Capacity / Average Standby Current

For example, if the battery capacity is 800 mAh and powers the EG915U series module. The average standby current of the EG915U series module in autosleep mode with network connection (LTE-FDD@64) is approximately 0.64 mA.

Then the actual standby time T = 800 mAh / 0.64 mA = 1250 h

## Current Consumption Testing Guide

### Overview of Current Consumption Testing

There are multiple methods to measure the current consumed by a module. The preferred method is to use a dedicated power analyzer. Power analyzers can automatically switch the current measurement range based on the current magnitude, and they have high sampling frequency and accuracy. They can accurately measure current and record current consumption with a large dynamic range.

In addition to power analyzers, current meters can also be used to test module current consumption, but the range of current meters cannot dynamically change, making it difficult to record the wide dynamic range of module current consumption.

Please note that to accurately measure module current, it is necessary to minimize the influence of peripheral circuits. It is recommended to directly connect the module's power supply to the power analyzer.

### Preparations for Current Consumption Testing

![](../media/system/power-consumption/Current_test_hardware.png)

First, adjust the voltage of the power analyzer to the rated voltage of the device under test (Note: this step must be done first to prevent damage to the device under test due to excessive voltage input). Connect the output and GND of the power analyzer to the input pins of the device under test (for cellular communication modules, the power supply pins are generally called VBAT, and some modules have two independent VBAT, divided into VBAT_RF and VBAT_BB, both of which should be connected to the power analyzer).

For the device under test shown in the figure above, the power supply of the module comes from VBAT inside the red box, and the power supply of other components on the EVB comes from the EVB DC power supply or USB. Therefore, the current consumption of VBAT is completely the current consumption of the module.

### Checking the Current Consumption Test Environment

Before conducting the current consumption test, we should check the integrity of the current consumption test. Connect the device under test with a known power consumption that meets the standard to the current test environment and test the current consumption. 

Compare the obtained results with the standard current consumption. If the data matches the standard,  the test environment meets the requirements and the power consumption test can be conducted. Otherwise, factors that affect the current consumption detection results need to be eliminated.

### Measuring Shutdown Current Consumption

After the hardware connection of the device under test is completed, do not power on the device and disconnect the USB_VBus to directly measure the shutdown current. At this time, the module is not working, and all the output pins to the external are pulled low or left floating, and the average current consumption remains relatively stable, basically in the uA level.

![](../media/system/power-consumption/Current_test_powerdown.png)

### Measuring Module Sleep Current

After measuring the shutdown current value, press and hold the power key to power on the module. The default power consumption mode of the module when it is powered on is IDLE. Please call the interface related to sleep to make the module enter sleep when it is idle. The operation method can be found in the [Autosleep API](../../api_reference/syslib/pm.md#Set-Automatic-Sleep-Mode). After setting to sleep, disconnect the USB. The module cannot enter sleep when the USB is connected.

After entering sleep mode, the module should have a periodic and stable current consumption waveform, which is the DRX period mentioned above, as shown in the following figure:

![](../media/system/power-consumption/Current_test_Sleep.png)

Because the power consumption of a single DRX period has a certain randomness, the average current consumption of the module should be calculated by selecting several complete DRX periods and taking the average value. In the figure above, three complete periods are selected to calculate the average current consumption during sleep. Power analyzers generally provide the functionality of calculating the average current consumption within a specific time period, so selecting several DRX periods is sufficient.

### Measuring the Idle Current of the Module When It Is Powered On

After completing the current consumption test under network connection, reconnect the USB. Give command to the module to disable the RF, the interfaces can be found at [net - Work Mode Configuration](../../api_reference/iotlib/net.md#Work-Mode-Configuration). After the configuration is completed, disconnect the USB and observe the current consumption. At this time, the module is idle and the RF is disabled, and the current consumed is the lowest level that the module can achieve when it is in sleep mode. The current consumption at this time is generally referred to as the idle current:

![](../media/system/power-consumption/Current_test_CFUN0_sleep.png)

The idle current is approximately a straight line, and a segment of average current can be selected.

### Measuring PSM Current Consumption
Before measuring the PSM current consumption, please set to PSMand check if it takes effect. The application method and the method to check if PSM takes effect can refer to the previous section [PSM](#PSM).

After confirming that the module enters PSM normally, disconnect the USB and start the test. After waiting for the ACT timer to time out, we can check the idle current in PSM.

![](../media/system/power-consumption/PSM.png)

As shown in the figure, the idle current in PSM is much lower than that in the normal autosleep mode, and it has similar characteristics and size as the current consumption during module shutdown.

PSM will be periodically awakened by the TAU timer to complete network interaction, so there will be a current consumption peak that matches the TAU period:

![](../media/system/power-consumption/PSMCycle.png)

Similarly, the average current consumption in PSM needs to be calculated by selecting multiple periods and taking the average value.

### Common Sources of Current Consumption and Troubleshooting approaches

Both software and hardware design errors can cause the module's current consumption to not reach the expected level. Here are some common sources of wake-up or current consumption, and corresponding troubleshooting ideas.

**1. Code Execution**

Code execution in any thread will inevitably cause the module to be periodically awakened. At this time, the current consumption characteristics are that the idle current meets the requirements, but the module cannot remain in sleep all the time. If the code continues to run or the interval between two code executes is too short, the module will not be able to enter sleep.

```python
import utime

while 1:
    print("Can't sleep!") # Software current consumption source example 1: infinite loop, the module cannot enter sleep when there is an infinite loop in any thread
    utime.sleep(1) # When a long enough sleep is added, it can enter sleep during the interval between two runs, but it will be awakened when the sleep ends
```

Troubleshooting approach: Decompose the business logic and turn off each running code block one by one. When a certain logic is turned off and the current consumption returns to normal, it can be determined that there must be abnormal wake-up occurs here.

**2. Timers**

Timers, including osTimer, RTC, and hardware timers, will wake up the module when they time out. At this time, the current consumption characteristics are: the idle current meets the requirements, but it cannot remain in sleep all the time. If the timeout time between two timers is too short, the module will not be able to enter sleep.

```python
import osTimer

def test_cb(arg):
    print("Will wakeup!!")

# Create an os timer
timer = osTimer() # Software current consumption source example 2: timer, the module will be awakened periodically when the timer times out
# Start the loop timer, it will keep waking up until stopped
timer.start(10000,1,test_cb)

```

Troubleshooting approach: Decompose the business logic and turn off the timers one by one. When a specific timer is encountered, observe if the current consumption returns to the expected level. This can help identify whether there is an abnormal wake-up event associated with that particular timer."

**3. Peripheral Communication**
Peripheral communication includes SPI, UART, I2C, and more. These peripheral communications require a high-speed clock, which means the module cannot enter sleep mode during communication. In such cases, the current consumption characteristics are that the idle current meets expectations, but the module cannot stay in sleep mode continuously. Wake-up events are related to the operation of sending data from the peripheral device.

Troubleshooting approach:
1. Disable peripheral communication one by one to identify which communication channel triggers the abnormal power consumption in the module.
2. Examine the peer device to ensure that the timing and frequency of communication with the module match the requirements.
3. If the peer device sends data uncontrollably, and the module needs to enter sleep, consider disabling the corresponding peripheral before entering sleep mode to avoid receiving data from the peer device.

**4. Wakelocks**
The wakelocks directly lock the RTOS state, keeping it in the IDLE state even when the module is idle and sleep is enabled, preventing it from entering sleep mode. In this situation, the module cannot enter sleep mode, and the current consumption characteristics during module idle corresponds to the IDLE state.

Troubleshooting approach:
Entries in the module's logs typically indicate when wakelocks prevent the module from sleep. Analyze the logs to confirm that if there are any un-released wakelocks.

**5. Hardware Leakage Current**
In practical applications, there are often cases of leakage current in certain voltage domains. For the device, any voltage domain that experiences leakage (e.g., no proper grounding through resistors or abnormal power consumption within the voltage domain) can cause an overall increase in power consumption. In this situation, the current consumption characteristics are as follows: the idle current does not meet expectations, but significant changes in current consumption are observed when entering and exiting sleep mode (i.e., the module can enter sleep mode, but the idle current does not meet expectations).

Troubleshooting approach:
1. For voltage domains that can be controlled by module IO, take appropriate actions such as pulling high or low, with the principle of disconnecting the power supply for the voltage domain. Investigate voltage domains with significant current reduction after disconnection to check for ground leakage or abnormal component power consumption.
   
2. For voltage domains that cannot be controlled by the module, physically disconnect them on the hardware level. Investigate voltage domains with significant current reduction after disconnection to check for ground leakage or abnormal component power consumption.

> Common current consuming components:
>
> LCD: Both the backlight and driver chip consume power. If the LCD is continuously refreshed, it may prevent the module from entering sleep mode. During sleep, the LCD should stop refreshing and turned the screen off.
>
> Pull-up circuit: Without resistors or diodes to limit the current, the pull-up circuit can generate significant leakage current for low levels.
>
> Level conversion chip: Some level conversion chips have lower input impedance and higher leakage current.
>
> Audio amplifier circuit: It has higher leakage current and should be turned off when not in use.
>
> Touch chip: It consumes higher current. When entering sleep mode, the touch chip can be instructed to enter low power mode or directly turned off.
>
> PHY network card: It consumes higher current. When entering sleep mode, the network card should be controlled to enter low power mode (but the PHY network card cannot be turned off in order to maintain network connection).