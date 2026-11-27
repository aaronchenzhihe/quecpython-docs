# 电源管理

## 电源管理概述

随着SOC技术的发展，电源管理技术也在不断革新。现今的SOC一般采用PMU（Power Management Unit，电源管理单元）来进行供电，其设计目标一般包括以下几点：

**多档电源的需求：**如下图，SOC往往包含了不同功能和电压域的IP。随着SOC越来越复杂，包含的IP越来越多，单个SOC上实现了CPU, 射频模块，相机模块，DDR控制模块，外设等等功能。多种功能，多种IP也带来了多档电源的需求。同时为了满足低功耗的需求，SOC通常被分为多个电源域，不同的电源域可以独立的上下电，从而实现更灵活的功耗控制。

**典型SOC架构示意图：**

![soc_example](../media/system/power-manager/soc_example.png)

**节能和延长电池寿命：**可以根据不同的应用场景和负载要求，实现电源的高效管理和控制。例如，可以通过动态调整电源电压和电流等参数，来适应不同的负载要求，从而提高电池使用效率和寿命。

**保护系统安全和稳定性：**可以监测电池电量、电源温度、电压和电流等关键参数，以保证系统的安全和稳定性。例如，在电池电量过低或过高、电流过大或过小等情况下，电源管理IC芯片会自动控制和保护电源系统，防止系统因电源问题而损坏或出现故障。

**提供多种功能和接口：**除了基础的电源管理功能之外，PMU往往集成了一些其它功能，如ADC、CODEC、PWM、温度监测、LED驱动、RTC等。这些功能可以配合PMIC本身的电源管理，为不同的应用提供灵活和多样化的电源管理解决方案；亦可以通过通讯接口和CPU交互，丰富SOC的功能。

**提高系统效率和性能：**通过使用高效的DC_DC转换器和低功耗的睡眠模式等技术，可以减少系统的能量损耗和功耗，从而提高系统的效率和性能。


典型的SOC芯片供电系统和内部电源管理单元如下图所示：

![soc_power](../media/system/power-manager/soc_power.png)

如图可见，典型SOC的供电系统中，PMIC同时给功能不同的IP核进行供电，同时进行兼顾开机和充电的管理。PMIC除了电源的输入/输出之外，往往还集成了时钟电路、RTC电路、多种模拟IP（一般ADC和温度传感器比较多见，部分PMIC内集成了音频CODEC）。PMIC是如何利用自身的多种功能进行电源管理呢？以下章节我们由PMIC入手，介绍其组成以及电源管理的实现。


## PMU 简介

PMU就是电源管理单元，一种高集成的、针对便携式应用的电源管理方案，即将传统分立的若干类电源管理芯片，如低压差线性稳压器（LDO）、直流直流转换器（DC_DC），但现在它们都被集成到手机的电源管理单元（PMU）中，这样可实现更高的电源转换效率和更低功耗，及更少的组件数以适应缩小的板级空间，成本更低。


**SoC处理器的PMU一般需求以下特性：**

**1.多档电源：**需要外部电源芯片提供多档电源，或在某档电源上施加较高的电流。原因在于SoC处理器通常要求其核心、I/O设备和存储单元等独立供电，而这些单元对电压、电流、频率等的要求通常各不相同。

**2.状态控制：**对外部提供专用控制信号，通过PMU控制寄存器设定这些控制信号的状态，从而实现对SOC整体电源状态的控制。PMU的一些基本接口信号（如系统开机、系统复位、电源状态指示信号等）发生变化后，SOC处理器状态将随之改变。

PMU由内部DC_DC和PMU寄存器等构成，PMU为SOC的处理器、PLL、DDR、图像单元等供电。同时，PMU也利用电源管理总线（PMBus）控制外部供电系统。

PMU一些基本功能如下：

1.控制开关机，上电下电的时序
2.LDO&DCDC 开关控制
3.CPU 电压&主频调节
4.高温保护功能
5.模拟输入ADC
6.GPIO 功能
7.充电功能
8.RTC
9.看门狗
10.codec
11.低功耗管理

不同SOC的PMU可以实现的功能不同，上述功能中控制开关机、高温保护、GPIO、充电管理、RTC、低功耗管理功能和用户相关性较强，后续章节我们重点介绍，其中低功耗将由专门的章节（[功耗管理](power-consumption.md)）来介绍。

## 名词解释

在正式介绍PMIC的功能之前，我们先来了解一些电源管理中常用的功能或技术：

**1.电源管理芯片（PMIC）**
&emsp;&emsp; PMIC是power management IC的缩写，中文是电源管理集成电路，主要特点是高集成度，将传统的多路输出电源封装在一颗芯片内，使得多电源应用场景高效率更高，体积更小。

**2.电源管理单元（PMU）**
&emsp;&emsp; PMU就是电源管理单元，一种高集成的、针对便携式应用的电源管理方案。将传统分立的若干类电源管理芯片，如低压差线性稳压器（LDO）、直流直流转换器（DC_DC）等集成到电源管理单元（PMU）中。这样可实现更高的电源转换效率和更低功耗，及更少的组件数以适应缩小的板级空间，成本更低。
&emsp;&emsp; 在大部分情况下，PMU等同于PMIC，但是这里讨论的SOC外部的供电芯片是PMIC, SOC内部的电源管理单元被称为PMU。

**3.DC_DC**
&emsp;&emsp; DC_DC指直流转直流电路，广义上包括所有输入输出都是直流的电源，下文将要提到的LDO，也是一种DC_DC。这里我们所说的DC_DC实际上指的是DC_DC最典型的一种实现形式：开关恒流源。蜂窝基带的DC_DC一般都是BUCK电路，根据用途的不同会附带一些特殊功能，例如：

**DVC_BUCK_DC_DC电路：**给CPU供电：负载能力强，带DVC（动态调节电压）功能，允许动态调整电压，达到节能目的。
**APT_BUCK_DC_DC电路：**给射频以及PA供电：负载能力强，带APT（平均功率追踪）功能，减少射频耗电。

&emsp;&emsp; 从用途上可以很明显看出来，DC_DC的优势是负载大。除了负载，它还有输入范围宽、转换效率高的优点。相对于LDO，它的缺点是结构复杂，负载响应差，纹波较大，不适合给对噪声敏感的电路供电。

**4.LDO**
LDO是一种线性电源，只能做降压，负载能力较弱。但它结构简单，成本低，适合大规模铺设。而且噪声小，负载响应快，在对噪声敏感的模拟电路内用LDO是最好的选择。
蜂窝基带内一般用来给各个分立器件或外设供电（如RTC电路、GPIO、UART等），这些器件对负载的要求不高。利用LDO可以方便的实现多电压域的低噪声电路。

**5.RTC**
RTC（Real time clock）的含义是实时时钟，它为人们提供精确的实时时间，或者为电子系统提供精确的时间基准。实时时钟芯片大多采用精度较高的晶体振荡器作为时钟源（对蜂窝模组来说，RTC适配的晶振一般都是32768 Hz，这个频率能达到的计时精度相对最高）。有些RTC在CPU关机时，PMIC仍保持RTC电路的供电，故可以继续保持运行。

以上，我们介绍了电源管理中经常出现的功能和技术，现代PMIC经常用这些技术，完成电源管理的既定目标。其硬件连接、具体功能和实现方法将在下个章节进行描述。

## PMU 硬件框架

PMU的内部典型硬件框图如下

![PMIC_example](../media/system/power-manager/PMIC_example.png)

在典型的SOC供电系统中，PMIC/PMU的硬件连接一般如下图：

![PMIC_HD](../media/system/power-manager/PMIC_CPU_Hardware.png)

可见，在电源的输入输出之外，PMU最主要的对外连接引脚有以下几个：
**1.power_en：**一般即是通信模组的powerkey，在这个引脚中断被触发时，PMIC会主动改变SOC的电源状态，给CPU供电，触发整个SOC的开机行为。在开机状态下，这个引脚中断仍然可用，可以在应用层绑定对应行为。

**2.PMBus：**PMBus 是一种低成本的双线接口，是 SMBus 标准的扩展，该标准基于 I2C 协议构建。由于 PMBus 是 SMBus 协议的扩展，因此它共享其大部分物理层以及总线的运行方式。但是，PMBus 定义了电源控制和管理组件所需的一组特定命令和数据结构。
&emsp;&emsp; PMBus基于I2C协议，只需要两个必选引脚（SDA和SCL），因此具有成本低，扩展性强等优点。在蜂窝模组中，常选用PMBus来作为PMIC和其它器件的通信协议（也有部分，如EC21，使用SPMI协议）。从而使PMIC能控制、监测整个模组的电源状态。同时，CPU也可以通过PMBus来改写PMU的寄存器，从而实现应用层对电源状态的控制。
&emsp;&emsp;  除此之外，蜂窝模块的部分传感器电路（一般是ADC、温度传感器）就铺设在PMU上，CPU需要这些传感器的数据时，也会通过PMBus访问PMU对应的寄存器。

**3.PSM_INT：**PSM休眠的唤醒引脚，仅在模组进入PSM休眠时起作用。一般支持PSM休眠模式的蜂窝模块都会引出。

> PSM休眠的概念见[功耗管理](power-consumption.md)

藉由以上所述的硬件构型，PMU在蜂窝模组中能实现诸如电源管理、唤醒PSM休眠、电压检测和温度检测功能。以下章节我们将具体介绍PMU的具体功能、实现细节和使用方法。

## PMU主要功能介绍

### Powerkey和Reset

用来控制模组电源状态的引脚主要是Powerkey和Reset引脚，二者虽然功能都是控制模组电源状态，但硬件连接和触发原理都有一些差别，我们这里做对比介绍。

#### Powerkey的硬件连接和开关机原理
Powerkey，即电源键，用来控制模组开关机。该按键连接至 PMIC。当模组上电时，PMIC 处于在电状态，但是 PMIC 并未给 CPU 供电。当 powerkey 被按下时，PMIC 开始向 CPU 供电，程序开始运行。在 bootloader 阶段，powerkey 的按键检测程序对电源键进行消抖处理和长按判定。当 powerkey 被长按达到消抖阈值后，程序继续向后运行，完成开机过程。

在开机状态下，powerkey被长按会触发关机。此时CPU会收到powekey长按产生的中断并主动调用关机指令，操作PMIC下电，改变模组的电源状态。

**开机时序图:**

![PowerkeyOn](../media/system/power-manager/PowerkeyOn.png)

**关机时序图:**

![PowerkeyOff](../media/system/power-manager/Powerkeyoff.png)

#### Powerkey长按开机的实现
在bootloader中对Powerkey进行电平检测和消抖，由于bootloader中一般不能使用ISR，实际的消抖大部分是轮询powerkey状态，保持低电平的时间低于消抖阈值则操作PMIC关机。

#### Reset的硬件连接和重启原理
与powerkey连接在PMIC上不同，RESET一般是直接连接在CPU上的，其触发的动作并不涉及PMIC，只是CPU复位，中断向量表回到bootrom，然后从bootrom开始执行。
**RESET时序图：**

![PowerkeyOff](../media/system/power-manager/Reset.png)

#### 自定义Powerkey短按长按功能

自定义Powerkey的短按长按功能实际上就是按键检测，实现方法是在按下时的中断里启动一个定时器，在抬起的中断里关闭这个定时器。如果定时器尚未到期，就被powerkey抬起的中断关闭，则判定为短按。如果直到超时都没有抬起的中断触发，则判定为长按。定时器的超时时间就是界定短按和长按的阈值。

配置powerkey自定义功能用法：[class PowerKey - power key按键回调注册功能 ](../../api_reference/peripherals/misc.PowerKey.md)

配置powerkey自定义短按/长按示例代码：

```python
from misc import PowerKey
import osTimer

class Pwrkey(object):
    def __init__(self, time_threshold):
        self.pk = PowerKey()
        self.time_threshold = time_threshold
        self.timer = osTimer()
        self.is_long_press = 0
        self.pk.powerKeyEventRegister(self.pwk_callback)

    def long_press_cb(self):
        print('powerkey long press.')

    def short_press_cb(self):
        print('powerkey short press.')

    def pwk_timer_cb(self, arg):
        self.is_long_press = 1

    def pwk_callback(self, status):
        if status == 0:
            print('powerkey release.')
            self.timer.stop()
            if self.is_long_press:
                self.long_press_cb();
            else:
                self.short_press_cb();

        elif status == 1:
            print('powerkey press.')
            self.is_long_press = 0
            self.timer.start(self.time_threshold, 0, self.pwk_timer_cb)


if __name__ == "__main__":
    pwrkey = Pwrkey(1000)


```



#### 上电自启动功能实现

如需上电自动开机功能且不考虑关机（ECX00M在这种场景下无法关机），则可以把 Powerkey 直接下拉到地，下拉电阻建议不超过 1 kΩ。

**上电自启动典型设计电路：**

![autostart](../media/system/power-manager/autostart.png)


### PSM_INT
#### PSM_INT概述
PSM_INT是PMIC上的一个引脚，它的特征是在PSM模式下仍能响应，并触发模组唤醒。但需要注意的是，在关机模式下，这个引脚一般是失效的。也就是说，PSM_INT是一个仅在PSM模式下生效的唤醒源。

#### PSM_INT的原理
ECX00U&BG95：PMIC上的引脚，进入PSM模式时，PMIC默认使能其中断，触发时会将模组从PSM中唤醒。
ECX00E：与之类似的是六个wakeup引脚，需要在模组进入PSM前使能这几个wakeup引脚，并打开对应的中断控制器，触发时会将模组从PSM中唤醒。

#### 使用方法 - PSM_INT
ECX00U&BG95：进入PSM时默认开启使能，具有固定的电平状态和触发条件，进入PSM模式时直接触发即可。
ECX00E：需要配置使能哪几路wakeup，可设定自身电平状态和触发条件。

### 休眠GPIO电平控制

#### GPIO硬件概述
在大部分情况下，蜂窝模组的GPIO从CPU引出，但供电则来自PMIC的一路或多路LDO。此类GPIO能正常工作的前提条件有两个：
1.CPU使能GPIO（一般是写GPIO对应的寄存器，如果作输入，还要保证GPIO中断被打开）。
2.PMU保持GPIO的供电。
上述二者有任何一点不满足，都会导致GPIO失控。因此，我们需要特别关注休眠情况下GPIO电平的变化。

但需要注意的是，部分资源较为充裕的PMIC会直接引出GPIO。这类GPIO并不需要CPU的控制，而是由PMIC直接提供电源并控制其状态。这种GPIO是否保持正常功能完全取决于PMIC，只要PMIC能保持电源，就可以维持GPIO的状态，关于这两种GPIO在休眠时的行为，我们会在下一段仔细介绍。

#### 休眠时GPIO电平变化和原理
**sleep：**GPIO在sleep下有几种可能的行为，下面分别介绍

**进入休眠后不掉电：**对绝大部分蜂窝模组而言，sleep下CPU的GPIO控制器并不会掉电，此时，模组并不会去操作GPIO状态相关的寄存器，PMU也不会切断GPIO的供电，所以GPIO的电平一般保持不变。

**进入休眠后掉电：**特例是ECX00E系列模组，此平台在sleep下会停止GPIO的供电并关闭对应中断。AONGPIO可保持输出电平，其余GPIO全部下电，GPIO中断不可触发。原因是ECX00E系列模组为了降低休眠功耗，关闭了GPIO的供电LDO，所以这个型号的GPIO在休眠时会全部掉电。

**PSM&关机：**CPU掉电，和CPU连接的GPIO随之掉电，变成低电平。但PMIC上的GPIO仍在电，可以维持电平不变。

> 对Quecpython支持的蜂窝模组而言，PMIC有GPIO输出的仅EC21。

### VDD_EXT
#### VDD_EXT概述
一路直接连接到LDO或者常开IO的电源引脚，CPU上电时LDO开始供电，此引脚立刻拉高，并在开机后一直保持高电平。可用来标识CPU上电时间，可以作为常开的电源或上拉，部分连接在常开LDO上的VDD_EXT在CPU下电后仍能保持高电平。

#### VDD_EXT电平-时序图

![VDD_EXTTime](../media/system/power-manager/VDD_EXTTime.png)
如图，可见VDD_EXT在powerkey按下后迅速上电，其电平在开机后一直保持高电平。

#### 典型应用-VDD_EXT
**标识模组电源状态：**前面的章节我们描述过，powerkey被触发时，PMIC会给CPU上电并改变整个模组的电源状态，VDD_EXT会在此时同步被拉高。因此，VDD_EXT的电平状态可以作为整个模组电源状态的标志。

**作为模组开机时常开的电源或上拉：**由于VDD_EXT在开机时会一直保持高电平，且具有一定的输出能力，因此可以作为常开的上拉或者电源使用。

> 由于VDD_EXT一般由LDO供电，驱动能力并不高，作电源使用时，请务必确认是否满足用电器的耗流需求。

### RTC闹钟
#### RTC 闹钟概述
RTC（Real Time Clock）是PMIC上的一个单元，一般由一路关机时也可保持供电的LDO来驱动，其核心部件是数目不等的计数器，通过32K时钟源进行计数，并将实时时间存储在寄存器中，CPU可访问这个寄存器来获取时间。RTC alarm到期时可以产生中断，大部分型号可唤醒模组。

#### 闹钟原理和典型应用

闹钟原理

RTC单元上除了存储实时时间的寄存器，还有一组（部分型号的模组有多组）可读写的寄存器作为alarm寄存器，CPU可操作该寄存器，存入某个时间值作为alarm时间。

RTC单元时间刷新的时候，会和alarm中所设的时间进行比较。一旦实时时间与alarm值相等，就会触发RTC alarm超时中断。当CPU处于关机状态时，该中断能指令PMIC触发CPU开机。而CPU处于开机状态时，能够触发CPU的中断（类似硬件定时器）。

应用方法参考：[class RTC – 实时时钟 ](../../api_reference/peripherals/machine.RTC.md)

**关机闹钟示例代码：**

```python
import utime
from machine import RTC
from misc import Power

def Business_code_example(run_time):
    i = 0
    for i in range(run_time):
        print("Business app running")
        #Business code here
        utime.sleep(1)

    return

def rtc_alarm_set(alarm_time):
    rtc = RTC()
    tm_rtc_tuple = rtc.datetime()
    tm_rtc_second = utime.mktime((tm_rtc_tuple[0], tm_rtc_tuple[1], tm_rtc_tuple[2], tm_rtc_tuple[4], tm_rtc_tuple[5], tm_rtc_tuple[6], 0, 0))

    alarm_second = tm_rtc_second + alarm_time #RTC闹钟设为当前时间 + alarm_time, 即模组会在经过alarm_time s 后重启
    alarm_tuple = utime.localtime(alarm_second)

    rtc.set_alarm([alarm_tuple[0], alarm_tuple[1], alarm_tuple[2], alarm_tuple[6], alarm_tuple[3], alarm_tuple[4], alarm_tuple[5], 0])
    rtc.enable_alarm(1)

    utime.sleep(1)#部分模组RTC闹钟的设置是异步的，需要一定延迟，保证底层RTC信息能够被写入
    return

if __name__ == '__main__':
    alarm_time = 600 #RTC alarm 10min后触发
    Business_code_example(10)#业务代码运行

    rtc_alarm_set(alarm_time)#设置alarm
    Power.powerDown()#设置alarm后关机，RTC alarm超时会唤醒模组
```

[点此在github中下载完整代码](https://github.com/QuecPython/examples/blob/main/system/power-manager/RTC_example.py)



> 注意事项：
> ECX00E系列模组没有RTC电路，其RTC闹钟是硬件定时器实现的，可以在任何休眠等级下产生有效唤醒源，但关机时不可用。
> BG95的RTC alarm和PSM T3412复用一个alarm寄存器，因此不能共存。
> 定时精度，BG95平台定时精度需要专门校准调整
> 唤醒频率，ECX00E系列模组在特定sleep时会将RTC_alarm寄存器写入flash，会涉及到擦写flash的操作，不宜太频繁

### ADC
#### ADC概述
模拟数字转换器即A/D转换器，或简称ADC，通常是指一个将模拟信号转变为数字信号的电子元件。通常的模数转换器是将一个输入电压信号转换为一个输出的数字信号。

#### ADC原理
这种转换器的基本原理是把输入的模拟信号按规定的时间间隔采样，并与一系列标准的数字信号相比较，数字信号逐次收敛，直至两种信号相等为止，然后显示出代表此信号的二进制数。

蜂窝模组的PMIC一般有多路ADC，一般有一路固定对VBAT进行采样，监测VBAT的电压是否在合法范围内。还有数路开放对外，用户可自行连接需要采样的设备，并在应用层读取这些ADC的值。

#### 典型应用-ADC

**1.检测模组VBAT电压**
**使用方法：**[获取电池电压](../../api_reference/peripherals/misc.Power.md#获取电池电压)
这一路ADC固定连接在模组的VBAT上，我们可以在应用层监控模组VBAT的电压，在使用电池给模组供电时，此接口可以实现对电池的管理。
除了获取电压之外，当此路ADC探测到VBAT电压过高或过低时，会在PMIC上触发超压/低压的中断。此时为了保护模组硬件，PMIC会主动改变电源状态，指令整个模组下电关机。

**2.ADC的使用**
**使用方法：**[class ADC - 模数转换](../../api_reference/peripherals/misc.ADC.md)
将输出模拟量的设备连接到ADC上，可以读取到设备输出的电压幅值。此时我们在业务中利用这个设备的电压幅值来计算传感数据。

> 不同的PMIC，搭载ADC的有效检测范围也是不同的，需要根据硬件手册来确定。如果模拟设备的输出范围和ADC的检测范围不符，就需要在硬件上设计分压，将输出电压和ADC的有效范围做匹配。业务里再通过检测到的电压来反推模拟设备的真实输出值。

### 高温保护
#### 高温保护概述
PMIC的模拟电路中，往往附带了温度传感器。由于温度过高时，集成电路存在诸如性能下降、频率响应变差、背景噪声增加，乃至电路熔毁的风险；PMIC在侦测到过高的温度时，会主动改变电源状态，指令整个模组下电关机。

#### 高温保护原理
温度传感器是一个模拟器件，其输出电压随着温度变化而变化，PMIC通过ADC采集其输出电压，计算模组温度。PMIC会轮询温度传感器的值，当这个值超过模组的安全工作温度时，立即改变电压状态，指令模组下电关机，保护硬件。

#### 典型应用-高温保护

高温保护一般是PMIC的内置逻辑，从探测到实际的保护动作都由PMIC完成，安全工作温度的范围一般也是由厂家设定，应用层一般无需关注。

### 其它功能

**看门狗：**看门狗，又叫watchdog timer，是一个定时器电路，一般有一个输入，叫喂狗(kicking the dog/service the dog），一个输出可以控制CPU复位。CPU正常工作的时候，每隔一段时间输出一个信号到喂狗端，给 WDT清零，如果超过规定的时间不喂狗（一般在程序跑飞时），WDT定时超过，就会给出一个复位信号到CPU，使CPU复位，防止其死机。看门狗的作用就是防止程序发生死循环，或者说程序跑飞。

**Codec：**在硬件层面，音频编解码器指一个能编码模拟音频到数字音频和解码数字音频到模拟音频的独立设备。换种说法，它包含运行在同样时钟的模数转换器（ADC）和数模转换器（DAC）。部分PMU将音频Codec集成在模拟电路部分，让模组电路能够更精简。

**设备充电：**充电管理单元是PMIC中相对独立的单元，包括充电电路、电池传感器、充电状态指示灯驱动电路等。

充电管理是相对独立的功能，有自己的硬件单元和控制逻辑，接下来我们着重介绍。

## 设备充电

### 充电方案概述
通信模组的充电方案一般是借由PMIC的充电管理单元来实现的，部分模组（ECX00U系列模组）可以在模组开机的情况下，在软件中介入充电控制。

#### PMIC 充电管理单元原理
PMIC充电管理单元的实现形式一般是一个linear charger（线性充电器），只要检测到VBUS电压在合法范围，就会开始工作。

**典型充电管理单元的架构和硬件连接：**
![](../media/system/power-manager/Charger_example.png)

充电单元的传感器一般至少包括两个ADC，一路监控VBAT电压，另一路监控电池内阻（实际上通过内阻的变化来监控电池温度）。

#### 充电阶段控制
根据VBUS电压高低，PMIC会控制充电管理单元进入不同的充电管理阶段，典型划分如下(不同平台称谓不同，原理大致相似)：

涓流模式/激活模式：电池电压低于某个阈值时，电池管理系统会禁止向电池中充电，此时需要用小电流激活电池，使电池电压恢复到可充电状态。

预充电模式/小电流模式：当电池电压较低时，如果进行全功率充电，由于电池的化学特性，会给电池带来高温和过压等安全隐患。因此，当电压小于全电流充电阈值时，充电管理电源会用较小电流和电压对VBAT进行充电。

全电流模式/快速充电模式：此时电池已经到达能够安全进行全功率充电的电压，保持一个较大的恒定电流，快速对电池进行充电。

恒压充电：电池较接近满载电压，此时保持恒压，充电电流随VBAT和充电单元间压差减小逐渐下降，直到到达满电，充电停止。

复充：充电完成后，轮询VBAT电压，发现VBAT滑落到复充触发电压以下时，则进行恒压充电。

**典型的充电电流、电压曲线图如下：**

![ChargerCurrent](../media/system/power-manager/ChargerCurrent.png)

#### 充电的软件控制
允许通过软件调节充电行为，如停止/启动充电，控制恒流输出时的占空比，或者获取电池电压和温度等（仅ECX00U系列模组支持）。

### 关机充电方案
#### 关机充电方案概述
关机充电，即在关机状态下插入电源进行充电时，在不开机的情况下，实现对充电状态的监控。

#### 实现方案原理
充电单元本身含有控制充电阶段的FSM，可以在CPU不运行的情况下控制充电行为，所以我们要实现的即是在不开机的情况下，监控充电状态（如VBAT电压、电池温度等，可能还需要进行显示）。

此时，可在VBUS被电源拉高时通过硬件联动powerkey（如果PMIC支持charger插入触发开机，则无需触发powerkey，使能charger触发开机即可），CPU上电。

关机充电的逻辑停留在bootloader中实现，主要包括轮询电池状态和LCD显示等业务，但并不从bootloader中跳转进RTOS阶段，如果此时检测到powerkey再次被触发，就进入正常的开机流程。

![PoweroffCharger](../media/system/power-manager/PoweroffCharger.png)

### 平台支持情况
当前EC100Y和ECX00U内置充电管理，其它平台可外置充电管理芯片。



## 设备启动流程

### 启动流程概述
总体来说，启动流程可分为以下四个部分：
1.bootrom，上电立即运行的部分，一般固化在硬件的ROM内
2.bootloader，初始化必要外设并检查内置存储中的代码，满足启动条件后跳转到RTOS启动部分
3.RTOS，此处会完成RTOS的初始化，并且启动系统必要的task，此流程的最后会创建QuecPython虚拟机线程
4.QuecPython虚拟机启动，完成交互口、虚拟文件系统、网络环境的初始化后，在usr分区中寻找main.py并执行
总流程图如下：

![Mainstartup](../media/system/power-manager/Mainstartup.png)

### bootrom
bootrom是固化在模组硬件中的代码，复位向量代码即指向此处。当模组上电或复位时会立即执行此处的逻辑，一般包含强制下载检测、基本初始化等功能。运行结束前会检测并引导bootloader。

### bootloader

bootloader一般存储在内置存储中，作为第二级 boot，完成一些不确定的，相对复杂的，不方便在bootrom阶段完成的功能。
不同平台的bootloader功能不尽相同，以下介绍其典型流程。

#### 基本BSP外设初始化

bootloader中需要操作一些外设，如GPIO、UART等，此时会把这些外设进行初始化

#### powerkey消抖

bootloader中一般不能使用ISR，powerkey消抖即是在阈值时间内轮询powerkey的电平，如果其电平保持时间超过阈值时间，则继续进行开机流程。反之，则对此次powerkey触发进行消抖，操作PMIC关机。

#### 判断是否进行FOTA

大部分模组的FOTA依赖bootloader内进行跳转。一般上一次关机前会操作某个寄存器作为flag，标识需要进行fota。检测到FOTA的flag时，bootloader便不会跳转到RTOS系统，而是跳转到updater（一般是一块可以独立运行的代码，会将fota包内容写到指定的flash地址，完成固件的升级）。

#### 相关外设去初始化，跳转RTOS系统启动

将bootloader中使用过的外设去初始化，恢复到操作前的状态。校验RTOS代码区域的完整性，校验通过则跳转到RTOS入口函数，不通过一般会认为镜像损坏，跳进下载模式。

（ECX00N/M系列模组支持跳转到LOGO阶段，读取文件系统中的图片资源，并初始化LCD驱动，完成开机LOGO的显示，LOGO结束后跳转到RTOS）

### RTOS启动
RTOS是模组运行的主体程序，绝大部分通信模组此时会初始化两套RTOS，分别处理CP和AP任务。一般启动流程如下：

![RTOS](../media/system/power-manager/RTOS.png)

#### 时钟和外设进行初始化，为RTOS启动做好准备
为了获得准确的sys_tick，CPU的tick会被重新初始化（即tick恢复到0）。所有外设也会被置为上电初始状态。此时的BSP状态应和模组默认电平相对齐。

#### RTOS初始化

初始化任务堆栈、任务调度器，当任务调度器初始化完成，rtos就已经接管了底层，我们调用的函数开始全部运行在RTOS的任务中。

#### CP core启动搜网和SIM卡相关任务

CP侧开始启动搜网和SIM卡相关任务，原厂的CP core一般不开源，此处不再赘述

#### AP core启动必要的内部task

WDT、FS、协议栈等必要的内部task启动，完善APP运行的条件。

#### 启动QuecPython虚拟机

RTOS内部环境初始化完成，启动APP task。APP一般是一个优先级相对较低的task，用来处理第三方业务。相对于底层而言，我们的 QuecPython虚拟机就是一个APP，在RTOS初始化完成后启动。

## 开机APP自动运行

QuecPython虚拟机是python运行的基础，虚拟机在启动时会完成交互命令行、文件系统和网络环境的初始化，并在最后默认执行名为main.py的pyscript

大致流程如下：

![pythonVM](../media/system/power-manager/pythonVM.png)

### 虚拟机本身的初始化
包括栈、GC、python解释器等虚拟机组件初始化，初始化完成后已经能运行pyscript，此时进入\_boot.py进行下一步初始化.

### 虚拟文件系统初始化
要在python中访问文件系统，就要把模组上实体的文件系统挂载到python的VFS上。部分模组底层没有初始化我们需要的littleFS文件系统，此时就要将实体文件系统一并初始化。

当实体文件系统没有初始化成功、SPI通信失败等情况导致此步骤执行失败，模组会继续开机流程。但会直接跳过app_fota和备份还原，这两个功能对文件系统是强依赖，文件系统未能初始化成功时执行必定失败，没有意义。

### 备份还原
启用备份还原机制时，若usr中文件出现丢失或者更改，会从bak分区将备份文件恢复进usr。若未开启备份还原机制，或者usr内文件没有变更时，则直接跳过备份还原流程。

### app_fota
检测是否进行app_fota。app_fota在下载完成后会在文件系统中写入fota标志，如果在文件系统中读到这个标志，就会进行app_fota升级流程。将下载的内容更新到usr分区。

### 交互命令行初始化
从此环节开始，是一定会执行到的流程，repl命令行启动，如果system_config.json中开启了交互保护，则会打开交互保护功能。

### 自动拨号
网络环境初始化，根据system_config.json中的配置决定是否进行开机自动拨号(默认开启)。

### pyscript运行
以上步骤均结束后，检测usr文件系统内是否有名为main.py的脚本，有则立即运行此脚本。因此，main.py是APP任务的固定入口点。

## 开机LOGO显示

### 开机LOGO的使用场景
开机LOGO一般需求尽量快的显示，由于虚拟机初始化和APP载入需要一定时间，因此要在此虚拟机启动前先显示LOGO

### 方案1：bootloader（ECX00M/N系列模组方案）
在bootloader中引入LCD和文件系统部分的代码，在bootloader阶段完成LCD显示，适用于boot时间较长的场景。

ECX00M/N系列模组bootloader的最后会进入名为LOGO的代码块，这里我们适配了littleFS文件系统和SPI LCD驱动，可以直接访问usr分区的数据，并驱动LCD显示LOGO。由于usr分区在上层可以直接修改，客户可以方便的更换LOGO图像。

*此处预备插入BSP同事提供的logo示例代码链接*

### 方案2：RTOS（ECX00U系列模组展锐方案）
在RTOS初始化完成，但APP尚未启动时，先刷新屏幕，显示LOGO，然后再进行QuecPython虚拟机的初始化，适用于boot较快的场景。

在python还没启动时提前将lfs文件系统挂载到ECX00U系列模组底层的vfs上，就可以在虚拟机初始化之前访问文件系统并驱动LCD显示LOGO，同样的，LOGO图片报此马在usr分区，客户可以直接在python层进行修改。

### 自定义显示
以上两种方案，其显示内容都存储在我们的文件系统usr分区内，用户在python层修改usr分区内的logo内容即可。

## USB_BOOT

### USB_BOOT概述
   模组固定的一个IO引脚，bootrom阶段检测到此引脚处于某种特定的电平时，直接跳转到强制下载/紧急下载模式。开机后可当作普通IO使用。

### 强制下载的实现原理
   模组固件被破坏或未烧录时，USB_BOOT仍能够被正常触发。USB_BOOT的引脚号、触发电平和下载模式跳转全部固化在硬件的ROM中，且此ROM的代码入口就是CPU的复位向量，即使可擦写存储介质中的代码损坏，也不影响此处USB_BOOT的行为。

###  使用方法 - USB_BOOT
使用场景：首次烧录、救砖、不方便使用软件控制进入下载模式的场景
使用方法：将usb_boot短接到地（部分模组，如EC600U/EC600E是上拉到1.8V，具体触发电平可在硬件规格书中获取），然后上电。此时USB不再和正常启动时一样枚举多个口，上位机只能看到一个下载口

>    可能的下载口名称：
>    Quectel Download Port（ECX00N, ECX00M系列模组）
>    Quectel QDLoader 9008（BG95系列模组）
>    Quectel QDLoader Port（ECX00E系列模）
>    SPRD U2S Port（ECX00U ECX00G系列模组）

此时在上位机启动烧录软件，烧录软件操作如下：
**QPYCom：**载入固件后，根据型号选中模组对应下载口，然后点击“固件下载”进行烧录
**QFlash**：载入固件后，根据型号选中模组对应下载口，然后点击“Start”进行烧录

**USB_BOOT开机时序图：**

![USB_BOOTTime](../media/system/power-manager/USB_BOOTTime.png)

> 注意事项：不期望进行下载时，注意在开机流程结束前保护此引脚电平，防止模组检测到此引脚被触发从而进入下载模式
