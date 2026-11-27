# 功耗管理

## 模组支持的功耗模式
蜂窝通信模组支持多种工作模式，每种模式的功耗各不相同，常见的工作模式有以下几种：
   **ACTIVE**：模组进行LTE数传、GSM通话或RTOS在运行逻辑时的状态，功耗受到具体业务和网络通信制式的影响，CPU本身功耗和网络射频功率都有所不同，故实际功耗在不同工况下会有较大差异。

   **IDLE**：此时模组处于空闲状态，硬件正常在电，RTOS保持运行，但没有任何线程需要被执行。有业务启动或网络业务呼入时，会立即恢复运行。ECX00U系列模组会在IDLE模式下降低时钟频率，进入轻睡眠状态（关闭高速时钟，但CPU不休眠的状态）。

   **休眠**：休眠模式的前提是模组处于空闲状态且使能autosleep，进入休眠模式后，RTOS暂停运行，模组的时钟频率会变慢，部分外设控制器（UART、SPI等）下电，同时只保留部分中断控制器，达到减小功耗的目的。

   **PSM**：PSM模式是3GPP协议所规定的一种低功耗模式，这种模式下，模组只会周期性的唤醒并执行业务，其余时间都处于PSM休眠中。PSM休眠时，模组行为和功耗都近似于关机。

   **关机**：模组完全下电的状态，此时BB芯片和外设控制器完全关闭，但PMIC仍是在电的。一般可由Powerkey或者RTC闹钟唤醒。

**各平台工作模式支持情况和功耗：**

|  | ECX00U | ECX00G | ECX00N | ECX00M | ECX00E | BG95 | EC21 |
| :------- | ------ | ----- | ---- | ------ | ---- | -------- | -------- |
| IDLE（LTE FDD@64ms） | 12.34 mA | 10.1 mA | 17.36 mA | 16.72 mA | 4 mA | 18.9 mA(catm@128ms) | 16.9 mA |
| SLEEP(LTE FDD@64ms) | 2.05mA | 1.99 mA | 1.64 mA | 1.37 mA | 0.64 mA | 1.89 mA(catm@128ms) | 2.74 mA |
| SLEEP(CFUN0) | 1.29mA | 1.03 mA | 0.96 mA | 0.8 mA | 60 μA | 0.575mA | 1.00 mA |
| PSM | 15 μA | 15 μA | 不支持 | 不支持 | 5 μA | 5.10 μA | 不支持 |
| 关机 | 33 μA | 15 μA | 20 μA | 38 μA | 3 μA | 14.87 μA | 7 μA |

IDLE即是模组空闲的状态，关机则是切断电源，这两种功耗模式我们不多做赘述。实际使用中，我们经常用的休眠模式就是休眠和PSM模式。

## 休眠
休眠是蜂窝通信模组最常用的一种低功耗模式。模组处于空闲状态时，若autosleep被使能，则模组会进入休眠状态。此时，模组会关闭部分IP核（如外设控制器和中断控制器等）并且降低时钟频率，从而实现功耗的降低。

> autosleep是一个控制是否在空闲时进入休眠的标志，其作用和原理后续会详细介绍

### RTOS休眠原理

RTOS的休眠机制，基本都是配合CPU的休眠模式使用的。这个模式会关闭CPU、高速时钟和大部分外设（关闭前会保存外设的上下文）。此时SRAM保持在电，保留休眠前CPU的运行状态，因此在唤醒时，可以立即恢复运行，不会有很大的延迟。

**休眠检测机制：**
RTOS在上电初始化时，会默认建立一个名为IDLE的TASK，优先级为0，也就是说，这个task只在RTOS不运行其它task时才会被调度到。休眠相关处理就在IDLE TASK中被执行。RTOS在IDLE TASK中，会检测模组的外设是否正在使用，网络是否有数据等休眠条件等，满足休眠条件且autosleep使能时，控制模组进入休眠模式。

**休眠机制：**
RTOS的休眠一般也会指令CPU进入睡眠模式，此时，高速时钟将会关闭，外设会在保存上下文后断电，ARM的内核会停止运行，但NVIC（中断控制器）和SRAM仍可保持运行，任意中断都可将内核唤醒。

**唤醒机制：**
任意中断均可将内核唤醒，唤醒后内核立即恢复运行，高速时钟恢复输出，为断电的外设恢复上下文，最后恢复应用程序的运行。这种休眠-唤醒模式不仅实现了功耗的降低，还能较快恢复应用程序的运行。

**休眠流程图例：**

![Sleep](../media/system/power-consumption/RTOSSleep.png)

> 不同的RTOS可能实现休眠的逻辑不同，但原理基本类似

**典型耗流特征：**

![consumption_all](../media/system/power-consumption/consumption_all.png)

### 蜂窝通信模组休眠

Quecpython支持的蜂窝通信模组要进入休眠，需要先使能休眠模式。不使能休眠模式时，模组空闲时默认处于IDLE状态。

根据前一章节可知，模块空闲时保持IDLE状态还是进入休眠状态，完全取决于是否使能autosleep。换言之，模块空闲时进入休眠时没有任何硬性阻碍，而是完全由用户控制。那么，为什么不选择默认进入休眠呢？

原因是，某些外设需要一直刷新（如LCD）。然而当模组休眠时，这些外设便不能连续工作，而是不断进行掉电->恢复的过程，严重降低了外设的刷新效率和响应速度。同时，由于休眠时采用低速时钟，任何依赖高速时钟的设备即使不掉电也无法正常工作。故而，模组空闲时会默认保持在IDLE状态，维持所有设备的正常工作。

#### Autosleep机制

autosleep本质上是操作RTOS休眠检测机制中的一个flag，不使能autosleep时，模组的检测机制就会指令模组保持在IDLE状态。autosleep被使能时，检测机制才认为模组处于允许休眠的状态，从而进入休眠的逻辑。

使用方法参见：[自动休眠模式控制](../../api_reference/syslib/pm.md#自动休眠模式控制)

#### 休眠锁机制
在某些场景下，我们既要使模组能够进入休眠，又需要在特定代码段保护某些外设的正常工作，这时，我们就要引入休眠锁机制。

休眠锁本质上也是一个flag，允许创建多个。生效机制是：只要有任意一个休眠锁处于lock状态，模组就不会进入休眠。

使用方法参见：[创建wake_lock锁](../../api_reference/syslib/pm.md#创建wake_lock锁)

#### 影响蜂窝通信模组休眠的因素

#####  底电流
影响底电流的功耗的原因主要是系统的主频、打开的IP核的功耗，这块和PMIC 相关。
底电流的影响因素主要来源于硬件，包括模组本身和外设两部分：
模组本身的耗流因素主要包括：系统主频、打开的IP核。
外设耗电：部分外设使用模组PMIC进行供电，此时其耗流就会由模组负担。

一般来说，测量底电流时，需要将网络置为CFUN0，使射频停止工作，此时测试出的电流即为模组底电流，正常情况下其特征一般近似直线，如下图：

![BaseCurrent](../media/system/power-consumption/BaseCurrent.png)

##### DRX 周期&Paging
基站寻呼（paging）：当网络状态产生变化/需要对UE进行呼叫/ETWS或CMAS系统发送预警时，基站会对模组发起寻呼，通知处于IDLE状态的模组开始进行通信。

DRX周期：模组网络在没有RRC连接的情况下，会周期性的监听基站寻呼，保证基站寻呼到来时能及时响应。这个监听的周期就叫做DRX周期。DRX周期一般有32ms、64ms、128ms几种，DRX周期越短，同样时间内监听基站信号的次数就越多，功耗就越高，但响应寻呼就更及时。反之，DRX周期越长，功耗越低，但响应寻呼的速度越慢。

附上Sleep（CFUN1）模式下耗流图，图中规律性的凸起即为paging，其周期即DRX周期：

![Paging](../media/system/power-consumption/Paging.png)

##### 信号质量
信号质量差的时候，在完成相同的网络行为时，模组需要更大的发射功率。这会在两个场景下影响模组功耗：
1.进行网络业务时，耗流会上升，除上文描述的发射功率外，如果网络业务出现了因通信质量差导致的重连或重传，整体的业务时间会被拉长，导致整体耗流进一步上升。
2.模组休眠时，每次寻呼的峰值耗流会上升，导致整体休眠电流上升。

##### 业务数据
模组在进行网络通信时，射频和CPU都会工作，从而产生较大的耗流。一般实际业务中，不会一直进行业务数据传输，常见的业务模式是心跳包。
由于在使用长连接时，如果长期不进行通信，可能会被对端认为已经离线，从而断开连接。所以业务中一般会以固定周期向对端发送一包数据，用来保活长连接。
各心跳周期下的耗流数据(LTE-FDD@64)：

| ECX00U | ECX00G  | ECX00M  | ECX00E  | EC21    | BG95 |
| ------ | ------- | ------- | ------- | ------- | --------- |
| 5min   | 2.16 mA | 2.13 mA | 1.32 mA | 4.41 mA | 5 mA      |
| 10min  | 1.84 mA | 1.76 mA | 1.03 mA | 3.39 mA | 4.43 mA   |


##### RRC 释放时间
无线资源控制（Radio Resource Control，RRC），又称为无线资源管理（RRM）或者无线资源分配（RRA），是包括呼叫准入控制、切换、功率控制、信道分配、分组调度、端到端QoS保障等各自独立的调配和管理算法。模组要进行业务时，需要和基站建立RRC连接，只有当RRC连接被释放时，模组才能进入休眠。

但如下图所示，这个RRC连接的生命周期中，只有最开始数秒在进行真正的数据业务。从业务完成到RRC连接释放之间，有一段耗流近似IDLE，却无法休眠的时间，这段耗流的形成和RRC的运营商策略有关：

![Paging](../media/system/power-consumption/RRC.png)

为了避免乒乓效应（指网络状态反复切换的情况，会导致更多的系统资源占用和网络性能下降），基站并不会在网络业务完成后立即释放RRC连接，而是会等待一段时间，保证短时间内再次进行业务时无需重新建立RRC连接。RRC连接的时长会因网络环境和运营商策略而不同产生差别，在没有网络数据而RRC未释放的条件下，模组并不会进入休眠。因此RRC连接的时长会显著影响模组功耗。

### 休眠模式唤醒源
蜂窝通信模组进入休眠模式后，有多种模式可以唤醒模组，包括网络数据、电话、GPIO 中断、Timer等，各种方式实际上都是通过中断来唤醒系统。

#### Soc 唤醒机制
模组的休眠状态可以由任意中断唤醒（前提是有效的，部分中断控制器处于功耗考虑也会被关闭），唤醒后会在临界区中恢复设备上下文。此时中断控制器中存有此中断的标志，但是不会运行。等待外设恢复完毕，CPU会退出临界区，此时中断的ISR会立即运行。

#### 网络数据&电话唤醒

网络数据&电话：基站通过paging通知模组有呼叫请求，之后cp（或DSP）通过回调唤醒CPU，并通知CPU有网络数据到来，CPU开始将空口数据搬运进协议栈的buffer，此时上层应用（socket或电话）可从协议栈中获取到网络数据。

![Network](../media/system/power-consumption/Networkwakeup.png)

#### GPIO 中断唤醒

GPIO可用的前提：GPIO在模组休眠时保持供电，且其连接的中断的控制器也未被关闭。
GPIO硬件被触发时，其连接的中断控制器会立即响应并唤醒CPU，CPU会将外设的上下文恢复，然后退出临界区。此时ISR检测到GPIO的中断已经触发，会立即执行GPIO的中断函数（一般是发消息触发GPIO绑定的回调），最后触发该GPIO绑定的回调函数。

![GPIO](../media/system/power-consumption/GPIO.png)
#### timer 唤醒

Timer超时会触发系统定时器绑定的中断，中断控制器会立即响应并唤醒CPU，CPU会将外设的上下文恢复，然后退出临界区。最后通过ISR触发timer绑定的回调。

![Timer](../media/system/power-consumption/Timer.png)

#### 唤醒源获取
实际上，从上面三条可以看到，凡是涉及到唤醒行为，都要通过中断控制器进行，这正是休眠模式的特征：由任意中断唤醒。由此，我们可以得出结论：不需要刻意去获取唤醒源，唤醒行为必然对应着特定中断的触发。

#### 唤醒后业务处理
上一条已经做出结论：不需要刻意去获取唤醒源，唤醒行为必然对应着特定中断的触发。那么业务处理的原则也就很简单了：模组出休眠逻辑中，设备的上下文恢复后退出临界区，唤醒中断的ISR就会立即去执行，我们需要的业务逻辑直接由此触发即可，一般都是采用在ISR发送消息或者信号量，来触发对应业务的执行。

#### 弱信号休眠方案

模组在网络未连接时，会主动搜索网络并尝试连接。这种机制保证了模组能以最快速度附着到网络，但也会带来一个功耗问题：当模组所在地信号弱或者无网络时，模组会一直进行搜网，此时无法进入休眠，且射频功耗也较高。

对此问题的解决方案有以下两个:

1.OOS（Out of service）机制：在模组开机/服务中断/射频从关闭状态切换到打开时会尝试驻留小区。如果网络质量较好，模组驻留到当前小区；如果网络质量不好，模组可能驻留失败。 除此之外，驻留小区在射频条件不佳的情况下，会造成下行失步，从而模组执行重新驻留。 OoS 算法定义了模组尝试重新驻留的时间，如果未能驻留成功，则模组会进入睡眠状态，睡眠一定时间后，再次尝试进行网络驻留。通常的OSS机制随着尝试网络驻留的次数增加，休眠时间也会相应增长（增加到上限就不会再增加了，按最大的睡眠时间往复尝试）。例如：

第一阶段：睡眠30s,尝试驻留到小区，重复10次；

第二阶段：睡眠45s,尝试驻留到小区，重复10次；

第三阶段：睡眠60s,尝试驻留到小区，从此依次往复此过程。

模组的的OOS机制一般会尝试多次后才停止，且该时间段内射频一直处于工作状态，相较于正常网络环境，该机制可能会产生更多的的耗流。

2.在业务中自行执行退避和重连的流程。在一定时间内网络未连接时，可手动停止搜网，如指令模组进入飞行模式等，需要重连时，恢复全功能模式即可，实际上就是业务中可自行控制时间和处理逻辑的退避模式，代码示例如下：

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



#### 典型应用
**网络例程：网络主动发送心跳包&模块接收寻呼唤醒**

```python
#利用MQTT来实现心跳包和接受寻呼
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
    keepalive=600) #keepalive：MQTT的保活机制，此处为每10min发送一次心跳包
c.error_register_cb(err_cb)
c.set_callback(sub_cb)
print('set_callback')
pm.autosleep(1)#打开休眠模式
c.connect()#连接MQTT，本质上建立TCP长连接
print('connect')

print("MQTT is connecting")
c.subscribe(sub_path)
print("Connected to mq.tongxinmao.com, subscribed to %s" % sub_path)
c.publish(pub_path, b"hello")
print("Publish topic: %s, msg: hello" % pub_path)

_thread.start_new_thread(wait_msg, ())#监听MQTT,等待数据到来，本质上就是等待寻呼，低功耗无需做特殊处理，心跳包发送、接收时会自动唤醒模组
            
```

**硬件例程1：GPIO/keypad 等外部硬件中断唤醒**

```python
import pm
pm.autosleep(1)#开启休眠

# 创建ExtInt对象
from machine import ExtInt
def fun(args):
        f = open("/usr/log.txt", "a+") #以追加模式打开一个文件，低功耗时无法连接交互口，在文件中保存调试信息
        f.write('### interrupt  {} ###'.format(args)) # args[0]:gpio号 args[1]:上升沿或下降沿，写入文件
        f.close()#关闭文件，保存写入信息
        
extint = ExtInt(ExtInt.GPIO1, ExtInt.IRQ_FALLING, ExtInt.PULL_PU, fun)#给GPIO1绑定回调fuction

extint.enable()#使能GPIO中断

#进入低功耗后测试触发GPIO1，然后连接USB，查看文件是否写入了正确信息。
#若文件中写入了正确的信息，说明低功耗模式下GPIO中断有效唤醒了模组，并且执行了其绑定的中断。
```

**硬件例程2：UART唤醒和接收**

```python
"""
运行本例程，需要通过串口线连接开发板的 MAIN 口和PC，在PC上通过串口工具
打开 MAIN 口，并向该端口发送数据，即可看到 PC 发送过来的消息。
"""
import _thread
import utime
import pm
from machine import UART

'''
将主串口接到串口小板上，连接到PC
 * 参数1：端口
        注：选择主串口，所有平台的主串口都支持低功耗唤醒机制，其它串口具有不确定性
        UART2 – MAIN PORT
 * 参数2：波特率
 * 参数3：data bits  （5~8）
 * 参数4：Parity  （0：NONE  1：EVEN  2：ODD）
 * 参数5：stop bits （1~2）
 * 参数6：flow control （0: FC_NONE  1：FC_HW）
'''


class Example_uart(object):
    def __init__(self, no=UART.UART2, bate=115200, data_bits=8, parity=0, stop_bits=1, flow_control=0):
        self.uart = UART(no, bate, data_bits, parity, stop_bits, flow_control)
        self.uart.set_callback(self.callback)


    def callback(self, para):
        if(0 == para[0]):
            self.uart.write("UART WAKRUP!")#在串口RX回调中发送特定数据  
        

    def uartWrite(self, msg):
        self.uart.write(msg)

    def uartRead(self, len):
        msg = self.uart.read(len)
        utf8_msg = msg.decode()
        return utf8_msg


if __name__ == "__main__":
    uart_test = Example_uart()
    pm.autosleep(1)


# 进入低功耗后向主串口发送数据，如果返回了"UART WAKRUP!"，则串口唤醒低功耗成功，并执行了发送
# 注：部分平台会丢一包数据


```

**硬件例程3：利用功耗锁在休眠唤醒时保护硬件时序和状态**

```python
from machine import SPI
import utime
import pm

spi_obj = SPI(0, 0, 1)

if __name__ == '__main__':
    pm.autosleep(1)
    
    lpm_fd = pm.create_wakelock("test", len("test"))#创建功耗锁，保护SPI读写

	pm.wakelock_lock(lpm_fd) #功耗锁上锁，开始spi读写

    r_data = bytearray(5)  # 创建接收数据的buff
    data = b"world"  # 测试数据

    ret = spi_obj.write_read(r_data, data, 5)  # 写入数据并接收
    spi_log.info(r_data)
    
	pm.wakelock_unlock(lpm_fd)#释放功耗锁，允许在SPI不进行读写时进入低功耗
	
```



#### 常见问题
**1.无法正常进入休眠:排查唤醒源**

| 常见唤醒源 | 备注                                      |
| ---------- | ----------------------------------------- |
| 功耗锁     | 任一把锁未释放都无法休眠，需要全部释放    |
| UART       | UART有数据时不可休眠                      |
| USB        | USB插入不可休眠                           |
| Thread     | Thread运行时不会进入IDLE，无法休眠        |
| SPI        | SPI有数据时无法休眠                       |
| LVGL       | LVGL会不断刷新，休眠时应使LVGL先进入sleep |

**2.正常进入休眠后底电流高，检测硬件和网络环境，需要检查的部分包括：**

| 耗流源  | 备注                                         |
| ------- | -------------------------------------------- |
| PWM     | PWM控制器可在休眠时使用低速时钟进行输出      |
| GPIO    | GPIO休眠时能够保持对外输出，需要检查是否漏电 |
| VDD_EXT | 常开电源，休眠时需要检查漏电                 |

**3.网络环境导致功耗高**

| 影响因素 | 备注                            |
| -------- | ------------------------------- |
| 射频功耗 | 检查paging时耗流峰值和网络质量  |
| RRC周期  | 检查业务完成到RRC链接释放的时间 |



## PSM 模式
### 什么是PSM 模式

PSM模式是一种比休眠模式功耗更低的低功耗模式，其硬件原理就是模组关机+RTC闹钟唤醒。与关机+RTC闹钟唤醒相比，PSM模式有以下两点不同：

1.RTC闹钟的唤醒时间由网络下发。

2.进入PSM模式时，模组虽然关机，但核心网仍然保留其注网信息。因此PSM唤醒时无需重新进行网络附着，联网速度更快，且功耗更低。



PSM模式是在UE空闲一定时间后关闭信号的收发，以及AS层相关功能，这相当于部分关闭，降低了天线、射频、信令处理等功耗。

PSM模式的优点是可以长时间睡眠，但缺点是无法及时应对终端接收（移动终端、MT）服务。主要应用在远程抄表和一些对实时性要求不高的场景中。

### PSM 模式原理

PSM是一种比常规休眠功耗功耗更低的模式，但其运行需要网络侧支持。

启用PSM时，需要先向基站发送请求（通过ATTACH或TA_UPDATER携带定时器信息）。基站若支持进入PSM，则会在对应的REQUEST中下发定时器信息，需要注意的是，实际PSM的参数会使用基站下发的值，并不一定和我们请求的值相同。

模组会以基站下发的值来配置定时器时间，当模组进入IDLE且ACT定时器超时，模组会关闭CPU和一切射频，此时相当于部分关机，只是保留了比关机更多的唤醒源，一般包括ACT、TAU定时器和PSM_INT。功耗一般下降到微安级别。此时由于基站已经记录UE使用PSM，不会断开此UE的连接，下次如果在同一小区内重新开机，就无需拨号，直接用原有的IP等信息，进一步减少了功耗。

当PSM TAU定时器超时、RTC_alarm到期或PSM_INT等唤醒源被触发时，模组会被唤醒。由于硬件上CPU（包括SRAM）和PA均被下电，此时并不能恢复休眠前运行状态，而是要重新走开机流程。开机后可进行网络业务，网络业务完成且RRC被释放后ACT定时器编开始计时，超时便进入PSM中。

**整体流程如下：**

![PSMProcess](../media/system/power-consumption/PSMMainProcess.png)

最终，在两个定时器的作用下，模组会进行周期性的休眠，唤醒切换，但只有在唤醒的时候，才能进行网络业务，休眠时是无法响应网络寻呼的。

**耗流特征如下(本示例每1min唤醒一次)：**

![Timer](../media/system/power-consumption/PSMCycle.png)

#### ACT和TAU定时器

ACT，又称T3324，当模组完成网络业务（即RRC连接释放时）开始计时，超时后立即进入PSM。

TAU，又称T3412，当模组完成网络业务（即RRC连接释放时）开始计时，超时后，若模组在PSM模式，会将模组唤醒。

它们的时序关系如下图：

![Timer](../media/system/power-consumption/3GPP_PSM_TIMER.png)

模组收到基站下发的PSM时间后，会立即启动一个RTC闹钟和一个Timer。RTC闹钟是PSM模式的唤醒源，超时时间就是T3412的值，也就是说，经过T3412时间后，模组会被RTC唤醒，进入下一个周期。Timer的超时时间是T3324，只有其超时后模组才可能进入PSM模式，即RTC闹钟将模组唤醒后，模组会保持运行的时间为T3324。

由此可见，要想模组能正常进入PSM，T3324必须小于T3412，否则便会出现RTC闹钟已经超时，模组仍无法进入PSM模式的情况。

### PSM &RTC 模式下的功耗以及各平台支持情况

|          | ECX00U | ECX00G | ECX00M | ECX00E | BG95 |
| -------- | ------ | ---- | ------------ | ---- | ---- |
| PSM      | 15 uA | 15 uA | 不支持 | 5uA | 5.10 uA |
| RTC+关机 | 33 uA | 15 uA | 38 uA | 5uA(非关机，hibernate休眠模式)    | 14.87 uA |

### PSM 唤醒源
#### PSM_INT

PSM_INT是唤醒PSM的引脚，此引脚一般都引出自PMIC。PSM_INT的功能是固化的，不需要额外的配置，进入PSM休眠后按照指定方式触发即可唤醒模组。

但对于ECx00E系列模组，没有专门的PSM_INT，而是由wakeup引脚实现此功能，wakeup引脚需要在休眠前进行配置。

#### RTC 闹钟

RTC闹钟能将模块从PSM模式下唤醒。其使用方法与模块关机时RTC闹钟的使用方法相同。

参见：[RTC相关API说明](../../api_reference/peripherals/machine.RTC.md)

#### Powerkey

Powerkey也能唤醒PSM模式，作用原理和触发开机一致。

#### TAU Timer

上文描述过，TAU定时器超时会唤醒模组。

> 注意：BG95 的TAU Timer 和RTC alarm复用了同一个PMIC唤醒源，因此PSM和RTC不能同时使用

### PSM 典型应用

PSM功耗虽低，但有以下缺点：

1.PSM启动时需要走重启逻辑，应用恢复时间远远长于autosleep；

2.产生小区切换时，无法拿到原有的网络信息，导致出现掉网和重新注网问题，反而会使耗流上升；

3.PSM休眠时不响应寻呼

所以PSM的应用场景一般满足以下三个条件：

1.定时任务间隔较长，可忽略启动时间

2.较少进行移动，不会频繁切换小区

3.对数据实时性要求不高

**示例代码：**

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
    if pm.get_psm_time()[0] == 1:#开机时获取psm是否设置，如果已经使能，则无需再次进行设置
        print("PSM has been enable, set pass")
        return 0
    else:
        return pm.set_psm_time(0,1,0,1)#T3412=10min T3324=1min

def psm_failed_handle(delay_time):
    utime.sleep(delay_time)#等待指定时长后，若模组仍未进入PSM模式，才会往下运行。此处执行PSM失败的处理逻辑，即代之以关机+RTC关机闹钟

    rtc = RTC()
    tm_rtc_tuple = rtc.datetime()
    tm_rtc_second = utime.mktime((tm_rtc_tuple[0], tm_rtc_tuple[1], tm_rtc_tuple[2], tm_rtc_tuple[4], tm_rtc_tuple[5], tm_rtc_tuple[6], 0, 0))

    alarm_second = tm_rtc_second + 600#RTC闹钟设为当前时间 + 10min
    alarm_tuple = utime.localtime(alarm_second)

    rtc.set_alarm([alarm_tuple[0], alarm_tuple[1], alarm_tuple[2], alarm_tuple[6], alarm_tuple[3], alarm_tuple[4], alarm_tuple[5], 0])
    rtc.enable_alarm(1)

    Power.powerDown()


if __name__ == '__main__':
    psm_failed_delay_time = 60 + 30 #PSM的 T3324超时30S后，启用错误处理逻辑
    lpm_fd = pm.create_wakelock("psm_lock", len("psm_lock")) #申请功耗锁

    stage, state = checkNet.waitNetworkReady(30)
    if stage == 3 and state == 1:
        print('Network connection successful.')
        psm_try_set() #网络连接成功时尝试设置PSM，如果PSM设置已经生效，则不用再次设置
    else:
        print('Network connection failed.')
        psm_failed_delay_time = 1 #PSM依赖网络，无网络时应立即使用RTC代替之

    pm.wakelock_lock(lpm_fd)#锁定功耗锁，防止业务运行过程中出现sleep时错误进入PSM模式
    Business_code_example(10)#业务代码运行  
    pm.wakelock_unlock(lpm_fd)#业务运行完成后，释放功耗锁。模组空闲状态且T3324超时后，自动进入PSM

    psm_failed_handle(psm_failed_delay_time)#运行错误处理，若模组能正常进入PSM，在sleep中就进入PSM了，该处实际的处理逻辑并不会运行
```

[点此在github中下载完整代码](https://github.com/QuecPython/examples/blob/main/system/power-consumption/psm_example.py)

#### 如何进入PSM

需要在联网，且确认运营商支持PSM的前提下使用。根据业务需求决定ACT和TAU的周期，通过API设置即可：

参见：[PSM相关API说明](../../api_reference/syslib/pm.md#设置PSM模式的控制时间)

#### PSM_INT 应用

EC600U和BG95支持此引脚，上升沿触发，进入PSM时模组PMIC会默认配置此引脚，确保触发方式正确即可在PSM模式下唤醒模组。

> ECX00E系列模组使用wakeup引脚，需要额外进行配置

#### 弱信号PSM 方案

弱信号下PSM实际并不能保证生效，因为和基站的东西实际上未必能够正常建立，更罔论请求与下发TAU和ATC定时器是否正常。
此时我们可以主动在业务中的解决，同行方案是：设定一个最大尝试联网的时间，若超过这个时间且网络仍未能正常连接，转而使用关机+RTC alarm来实现方案。

#### 不支持PSM 的运营商的方案

如果运营商不支持PSM，基站会设法阻碍模组进入PSM。常见的行为有三种：

1. 在模组向基站请求定时器时，直接REJECT请求(此情况很少，绝大部分为后两种)。
2. 下发不成对的ACT和TAU定时器，不成对的定时器无法使模组进入PSM。
3. 不下发定时器

此时，只能通过RTC闹钟定时唤醒关机来实现类似的业务需求，这种方式也能达到在无需数据交互时减少耗流。但从关机状态重新开机相比PSM多了一个注网和拨号的动作，在每次唤醒注网时都会产生更多射频功耗。

> 此外，ECX00E系列模组在不支持PSM时需要通过pm.forcehib()方法强制进入hibernate休眠，因为此平台关机时RTC无法维持运行。

### 常见问题排查

**1.PSM设置返回True，但无法进入**

基站可能不支持PSM，确认这一点的方法是从CPlog中查看基站下发的信息，需要查看的条目为：

**ATTACH_REQUEST**

**ATTACH_ACCEPT**

**TA_UPDATE_REQUEST**

**TA_UPDATE_ACCEPT**

如图，模组设置成功后，REQUEST一定会带有成对的定时器信息，我们需要关注ACCEPT中基站是否正确下发了定时器信息。

![Timer](../media/system/power-consumption/PSMRequest.png)

如图，网络环境不支持PSM，基站未下发成对的定时器，只下发了一个ACT定时器，这时候是不能进入PSM的：

![Timer](../media/system/power-consumption/PSMWrongACCEPT.png)

**2.PSM成功进入，但休眠-唤醒周期与设置的不同**

同样的，查看以上CPlog条目，基站下发的定时器周期可能不等于设置的值，且模组最终以基站下发的值为准。

## 常见应用场景
### 增强型休眠模式方案

增强型休眠（RRC快速释放）：快速释放RRC能够保证在任何一个随机时间点，快速释放RRC，从而进入休眠。

业务上，将周期性任务的时间周期尽量统一，或均为某个最小周期的整数倍。在单次唤醒中处理尽量多的周期性任务（心跳包、呼吸灯、定位等）。若周期性任务时间不统一，唤醒次数会增加，同一周期内休眠的时间占比会下降。如果唤醒太频繁，甚至无法进入休眠。

> 增强型休眠模式启用后，在用户发起数据交互时，会给基站产生额外的信令负担，所以可能存在终端被网络侧惩罚不能上线的情况，此功能默认处于关闭状态，客户在使用前需要自行对此风险进行评估。
>
> 目前仅中国国内支持。



### tracker 低功耗方案

RRC快速释放：tracker是一个定位器，位置是不固定的，定期进行一次定位。可使用RRC快速释放减少RRC连接保持的时长，从而降低整体耗流。



### 表计、门磁低功耗方案

PSM，此类产品的唤醒间隔较长，且一般不会经常移动，对数据的实时性要求不高，可采用PSM。若需要在无网络或运营商不支持的情况下使用，可采用关机+RTC闹钟定时唤醒的方案。



### 根据功耗需求选型

uA级耗流：必须选择支持PSM或关机RTC唤醒的型号



mA级耗流：全平台支持autosleep，根据其它需求评估适合型号



### 待机时间推算方法

1.首先确定电池选型，得到电池容量。

2.确定模组选型，根据规格书和实际业务计算平均待机电流。如果需要准确的待机电流，应保持整机按照业务流程运行，然后使用功耗仪测试出平均待机电流。

3.从待机时间的计算公式：待机时间 = 电池容量/ 平均待机电流

例如：已知电池容量800mah，给ECX00E系列模组供电。ECX00E系列模组在autosleep模式下保持网络连接（LTE-FDD@64）时平均待机电流约0.64mA。

则实际待机时间T=800mah/0.64mA=1250h

## 耗流测试指导

### 耗流测试概述

有多种方法可以测量模组消耗的电流。首选方法通常是使用专用功率分析仪。功率分析仪通常能够根据电流大小自动切换电流测量范围，采样频率和精度也较高。可以精确测量电流，并记录具有大动态范围的电流消耗。

除了功率分析仪外，也可以用电流表来测试模组耗流，但电流表的范围并不能动态变化，故难以记录动态范围较大的模组耗流。

需要注意的是，若要测试准确的模组电流，需要尽量排除外围电路的影响，最好将模组电源直连到功耗仪上。

### 耗流测试前的准备

![](../media/system/power-consumption/Current_test_hardware.png)

首先，将功耗分析仪的电压调整到待测设备的额定电压（注意，这一步必须先进行，防止给待测设备输入过高电压导致损坏），将功耗分析仪的输出和GND连接到待测设备的输入引脚上（对蜂窝通信模组而言，供电引脚一般称为VBAT，部分模组有两路独立的VBAT， 分为VBAT_RF和VBAT_BB，都要连接到功耗分析仪上）。

对于上图的耗流待测设备而言，模组供电来自于红框内的VBAT，底板上其它器件的供电来自于底板DC电源或USB，因此，VBAT的耗流就完全是模组功耗。

### 检测耗流测试环境

在进行耗流测试前，我们应检测耗流测试的健全性。用已知功耗符合标准的待测设备，接入当前测试环境并测试耗流。
将此时获取的结果和标准耗流进行对比，若数据与标准相符合，则本测试环境是符合要求的，可以进行功耗测试。反之，需要排除影响耗流检测结果的因素。

### 测量关机耗流

待测设备硬件连接完毕后，不进行开机，断开USB_VBus，即可直接测量关机电流。此时，模组不工作，对外的输出引脚也全部拉低或悬空，整体平均耗流保持相对稳定，基本都在uA级。

![](../media/system/power-consumption/Current_test_powerdown.png)

### 测量模组休眠电流

完成关机电流测量后，长按powerkey开机，模组开机默认的功耗模式是IDLE，我们需要调用休眠相关的接口，让模组在空闲时进入休眠，操作方法参见[autosleep相关API说明](../../api_reference/syslib/pm.md#自动休眠模式控制)。设置完休眠后，注意断开USB，USB连接时模组无法进入休眠。

进入休眠模式后，模组应有的耗流波形会有周期稳定的凸起，这就是上文所述的DRX周期，如下图：

![](../media/system/power-consumption/Current_test_Sleep.png)

因为单个DRX周期的功耗具有一定随机性，此时模组平均耗流的计算应选取数个完整的DRX周期，取平均值。如上图，即是选取了三个完整周期，算出此时休眠的平均耗流。功率分析仪一般提供了特定时间段内平均耗流的计算功能，选取数个DRX周期即可。

### 测量开机时模组的底电流

完成联网情况下耗流测试后，重新连接USB，指令模组将射频关闭，接口见此处[net - 网络工作模式配置 ](../../api_reference/iotlib/net.md#工作模式配置)。配置完成后，断开USB，观察耗流，此时模组空闲且射频被关闭，所耗的电流是模组休眠时可达的最低水平，这时候的电流一般可称之为底电流：

![](../media/system/power-consumption/Current_test_CFUN0_sleep.png)

底电流近似一条稳定直线，选取一段平均电流即可。

### 测量PSM电流
测量PSM耗流之前，我们要设置PSM模式并检测是否生效，应用方法和检测PSM是否生效的方法可参照前文[PSM模式章节](#PSM 模式)。

确认能正常进入PSM休眠后，断开USB，我们开始测试。等待ACT定时器超时后，我们可查看PSM休眠下的底电流。

![](../media/system/power-consumption/PSM.png)

如图可见，PSM的底电流远低于普通的autosleep模式，和关机耗流的大小、特征较为相似。

PSM会定期被TAU定时器唤醒，完成网络交互，所以会有一个周期和TAU相符的耗流尖峰：

![](../media/system/power-consumption/PSMCycle.png)

同样，PSM平均耗流需要取多个周期计算平均值

### 常见的耗流源以及排查思路

软件和硬件的设计失误均会导致模组耗流无法达到预期水准，以下介绍一些常见的唤醒或耗流源，以及对应的排查思路。

**1.代码运行**

在任何线程中，代码运行必然会导致模组定期唤醒，此时的耗流特征为：底电流符合要求，但不能一直保持休眠。如果代码持续运行或两次运行的间隔时间过短，模组就无法进入休眠。

```python
import utime

while 1:
	print("Can't sleep!")#软件耗流源例1：死循环，任何线程出现死循环时无法进入休眠
	utime.sleep(1)#任务运行的间隔足够长时,在两次运行的间隔可以进入休眠，但任务再次运行时，模组将被唤醒
```

排查思路：分解业务逻辑，逐个关闭运行的代码块，当出现某一块逻辑关闭后，耗流恢复预期。则可定位此处逻辑是否有异常唤醒。

**2.定时器**
定时器，包括osTimer、RTC和硬件定时器，超时都会唤醒模组，此时的耗流特征为：底电流符合要求，但不能一直保持休眠。如果两次超时时间过短，模组会无法进入休眠。

```python
import osTimer

def test_cb(arg):
    print("Will wakeup!!")
# 创建os定时器
timer = osTimer()#软件耗流源例2：定时器，定时器超时会唤醒模组
# 启动循环定时器，未停止就会一直定期唤醒
timer.start(10000,1,test_cb)

```

排查思路：分解业务逻辑，逐个关闭定时器，当出现某一个定时器，耗流恢复预期。则可定位此处定时器是否有异常唤醒。

**3.外设通信**
外设通信，包括SPI、串口和I2C等。这些外设通信需要高速时钟的保障，因此通信时无法休眠。此时的耗流特征为：底电流符合预期，但不能一直保持休眠，唤醒行为和外设是否来数据有关。

排查思路：
1.关闭外设通信，寻找是哪一路通信引发的模组功耗异常。
2.排查对端，与模组通信的时间和频率是否符合需求。
3.如果对端发送不可控制，而模组又需求进入休眠，可在休眠前关闭响应的外设，不接收对端的数据。

**4.休眠锁**
休眠锁会直接锁定RTOS状态，使之在模组空闲且使能过休眠的情况下，仍保持在IDLE，而非进入sleep。此时无法进入休眠，模组空闲时耗流特征和模组IDLE耗流对应。

排查思路：
模组日志中一般可见休眠锁阻止休眠的条目，可通过日志分析是否有休眠锁未释放。

**5.硬件漏电**
在实际应用中，常出现某电压域漏电的情况。对整机而言，任何电压域产生漏电（例如：不经电阻接地、电压域内元件异常耗电等），就会引起整体耗流上升。此时的耗流特征为：底电流不符合预期，但开启和关闭休眠能观察到明显的耗流变化（即模组能正常进入休眠，但底电流不符合预期）。

排查思路：
1.对于模组IO能够控制的电压域，根据实际情况做拉高或拉低处理，原则是将电压域的电源切断。对于切断后耗流下降显著的电压域进行排查，检查是否有对地漏电或元器件异常耗流。

2.对于模组无法控制的电压域，可在硬件上做断开处理，对于切断后耗流下降显著的电压域进行排查，检查是否有对地漏电或元器件异常耗流。

> 常见的耗流器件：
> LCD：背光和驱动芯片均有耗流，如果LCD持续刷新，还会导致模组无法进入休眠，休眠时LCD应停止刷新并熄屏
> 上拉电路：没有电阻或二极管进行扼流时，上拉电路对低电平会产生较大漏电
> 电平转换芯片：部分电平转换芯片输入阻抗较低，漏电流较高。
> audio 功放电路：漏电流较高，不用时应关闭
> 触摸芯片：耗流较高，进入休眠时可指令触摸芯片进入低功耗或直接关闭
> PHY网卡：耗流较高，进入休眠时应控制网卡进入低功耗（但为了保持网络连接，此设备不能关闭）
