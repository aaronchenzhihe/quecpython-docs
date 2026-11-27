# 系统时间

## 概述

系统时间表示在计算机系统中的时间与日期。通常用系统时钟（system clock）从某个时间起点的嘀嗒数（ticks）。

在大部分系统中，时间是不可或缺的一部分，QuecPython 设备有几个时间，支持多种时间信息及时间同步，如硬件时间时钟（RTC）模块、时间（utime）模块、定时器（timer）模块及时间同步协议 NITZ 模块、NTP 模块。

QuecPython 设备时间功能应用如下图所示：

![](../media/system/sys-time/time-introduction.png)

### RTC

实时时钟（Real-Time Clock，简称RTC）是一种集成电路。系统可以通过 RTC 建立和保持系统时间，帮助人们获得精确的实时时间，为电子系统提供精确的时间基准。

RTC 可以提供独立于操作系统的时间计时服务，即使设备关机（不断电）也能保持时间的准确性。

在 QuecPython 中内置 RTC 实时时钟，可以使用 `machine` 模块访问 RTC 实时时钟。对于 RTC 的所有用法参见 [class RTC – 实时时钟](../../api_reference/peripherals/machine.RTC.md)章节，下面主要介绍常用操作与注意事项。

> QuecPython 支持多种型号模组，不同模组 RTC 时间精度有所差异，对于时间精度要求较高的，可以根据各芯片硬件手册进行选择。
> QuecPython 模组 RTC 时间配置以及 ALARM 配置会擦写 FLASH，不宜频繁调用。

**RTC 时间获取及配置**

通过下面接口可以获取 RTC 时间以及根据需要配置 RTC 时间满足使用需求。

```python
from machine import RTC

# 初始化 RTC 实时时钟
rtc = RTC()

# 设置 RTC 时间
rtc.datetime([2026, 3, 12, 1, 12, 12, 12, 0])

# 获取 RTC 时间
rtc_time = rtc.datetime()

# 打印 RTC 时间
print(rtc_time)

```

**RTC ALARM**

闹钟是被设计成会在特定的时间向人发出讯号的时钟，用来提醒其它事务。

在 QuecPython 中，闹钟被设计用于设置 RTC 到期时间，时间到期就会调用注册的回调函数。该方式也可以用于低功耗唤醒，在低功耗状态下，通过 RTC 进行唤醒工作。
> EC600U 系列 / EC200U系列 / EC600M系列 / EC800M系列 支持自动开机，即设置 ALARM 功能之后将模块关机，ALARM 时间到了之后可以自动开机。其他平台不支持该特性。

```python

from machine import RTC

# 初始化 RTC 实时时钟
rtc = RTC()

# 定义唤醒回调函数接口，唤醒后执行该函数。
def callback(args):
   print('RTC alarm')

# 注册唤醒回调函数。
rtc.register_callback(callback)

# 设置闹钟，在某个时间唤醒。
rtc.set_alarm([2026, 7, 9, 5, 12, 30, 0, 0])

# 使能该闹钟。
rtc.enable_alarm(1)
```

### utime

时间是一种尺度，在物理定义是标量，借着时间，事件发生之先后可以按过去-现在-未来之序列得以确定（时间点/时刻），也可以衡量事件持续的期间以及事件之间和间隔长短（时间段）。

QuecPython 下 utime 模块提供获取当前时间和日期（UTC 时间）、测量时间间隔和延迟的函数。utime 是软件层面的时间。对于 utime 的所有用法参见标准库 [utime - 时间相关功能](../../api_reference/stdlib/utime.md) 章节。

#### System Tick

系统节拍通常是指 CPU 时钟，是操作系统的心脏，维持着整个系统运行的稳定性。操作系统要实现时间上的管理，必须依赖于系统节拍。

系统节拍是从开机开始一个不断递增的计数器，其时间精度取决于硬件平台底层的时钟。

##### 原理

系统节拍一般由晶振产生，以精确和固定的时间间隔，触发电信号，通过电信号翻转产生一个 tick，设备处理一次数据。比如晶振 12MHZ = 12 × 10 的 6 次方，即每秒发出 12000000 个脉冲信号，那么发出一个脉冲的时间就是时钟周期，也就是 1/12 微秒。通常也叫做系统时钟周期，是计算机中最基本的、最小的时间单位。

![](../media/system/sys-time/time-clock-signal.png)

##### 应用

System tick 是系统中最小的时间刻度，因此可以基于此项接口实现高精度时间管理。
> 注：由于 python 的运行机制，在 python 层无法进行高精度的实现。

**系统定时器**

基于 tick 封装 python 实现系统定时器。

```python
from machine import Timer

# timer 到时回调函数。
def timer_callback(t):
    print('timeout occured !')

# 创建定时器对象。
t = Timer(Timer.Timer1)

# 启动定时器，周期性 1s 执行定时器回调。
t.start(period=1000, mode=t.PERIODIC, callback=timer_callback)
```

**时间差**

通过utime.tick接口可以通过时间差确认代码执行效率。

```python
import utime

# 开始执行 print 时间
start = utime.ticks_us()

# 执行print打印语句
print('Hello QuecPython !')

# 结束执行 print 时间
end = utime.ticks_us()

# 计算时间差，单位us
utime.ticks_diff(end, start)

```

#### UTC 时间

UTC 时间（Universal Time Coordinated， 世界标准时间或世界协调时间），以原子时秒长为基础，在时刻上尽量接近于世界时的一种时间计量系统。

全球统一时间，但是由于地球旋转导致不同地域的人看到的日出日落的时间不同，全球同一时间显然不符合各地的作息，因此根据地球的地理位置，人为将地球划分成24个不同的时区。因此在 UTC 的基础上各地区形成本地时间。

本地时间 = UTC 时间 + 时区差，比如北京时区是东八区，领先 UTC 8 个小时，此时如果想获取北京时间，需要将时区配置到东区即可。

QuecPython 提供了本地时间接口以及时区配置接口，系统默认配置在东八区，如果您处在不同的时区，可以根据自己的需求，配置到您所处位置的时区。

```python
import utime

# 配置东八区时区。此处配置可能与 NTP 时间同步里面时区配置产生错乱，我们建议使用 ntptime 配置时区。
utime.setTimeZone(8)

# 获取本地 UTC 时间。
utime.localtime()

```

### 时间同步

所谓时间同步，即要求各点之间的绝对时间相同。在我们生活中可能会遇到，钟表长时间运行后，需要手动对表，防止时间偏差太大。那么长时间设备运行为什么时间会有偏差？在连网设备上是如何进行对时操作呢？在一些场景对各设备间时间一致性要求较高如何处理？

带有时间的设备，都是靠本地时钟源进行控制时间走时，长时间运行时，会受到本身精度或者环境的影响，出现偏差，此时对于时间要求较高的场景下，就需要对设备时间进行同步，保证运行的稳定性，常用的方式通过 NTP 向标准时间服务器进行时间同步（如全球 NTP 授时服务器，pool.ntp.org），或者向自己设计的时间服务器进行时间同步。

QuecPython 提供 NITZ、NTP 时间同步协议，方便客户进行时间同步操作。

#### 基站时间同步

NITZ（Network Identity and Time Zone，网络标识和时区），是一种用于自动配置本地的时间和日期的机制，需要运营商支持。对于蜂窝无线设备，在接入到运营商网络后，运营商网络向移动设备提供本地日期, 时间, 时区, 夏令时偏移等信息。

> 该时间可能与标准时间偏差较大。
> NITZ 未对外提供接口，只是内部同步时间机制，会在开机启动时进行时间同步。

#### NTP 时间同步

NTP（Network Time Protocol， 网络时间协议），是由 RFC 1305 定义的时间同步协议，用来在分布式时间服务器和客户端之间进行时间同步，NTP 基于 UDP 报文进行传输，使用 UDP 端口号为 123。

报文格式如下图所示，我们可以根据协议的报文内容，通过上报时间与服务器的时间比较，以及链路传输时间计算，校准当前设备与服务器时间保持一致。QuecPython 设备对于 NTP 的所有用法参见 [ntptime - 网络时间同步](../../api_reference/networklib/ntptime.md) 章节。

![](../media/system/sys-time/time-ntp-proto.png)

## API 说明

### 时间同步 - API 说明

对于需要时间信息，但是当前时间与标准时间不一致，或者与自己服务器时间不一致的情况下，可以使用 NTP 进行校时。
> 需要注意，对于需要校时的设备，需要考虑在使用的时间会不会因为校时导致本机时间变化而引起程序逻辑混乱。

```python
import ntptime

# 设置 NTP 服务器地址，pool.ntp.org是全球 NTP 授时服务器。
ntptime.sethost('pool.ntp.org')

# 同步 NTP 服务时间
ntptime.settime()

```

### 获取 UTC 时间

QuecPython 系统中，utime时间标准是按照 UTC 时间标准使用的。直接获取本地时间即可。默认表示东八区时间，需要根据需要自己调整时区。

```python
import utime

# 获取本地 UTC 时间。
utime.localtime()

```

### 获取 RTC 时间

RTC 获取年月日、时分秒时间格式，最小精度ms级别。

```python
from machine import RTC
# 创建 RTC 对象。
rtc = RTC()

# 获取 RTC 时间。
rtc.datetime()

```

### 获取时间差

根据平台不同，通过 tick 进行计算，时间差可以获取到 us 级别。
> 时间差请使用接口 utime.ticks_diff 计算，tick 在最大值后会进行重新计数，易造成问题。

```python
# 计算执行 print 语句的时间差。
import utime

# 开始执行 print 时间
start = utime.ticks_us()

# 执行print打印语句
print('Hello QuecPython !')

# 结束执行 print 时间
end = utime.ticks_us()

# 计算时间差，单位us
utime.ticks_diff(end, start)

```

### 时区同步

QuecPython 模组默认使用东八区 UTC 时间, 对于不同的地区，可以根据时区配置调整为自己本地时间。

```python
import utime

# 配置东八区时区。此处配置可能与 NTP 时间同步里面时区配置产生错乱，我们建议使用 ntptime 配置时区。
utime.setTimeZone(8)

# 获取本地 UTC 时间。
utime.localtime()

```

### 休眠

QuecPython 提供秒级、毫秒级、微妙级休眠接口，可以根据使用需求进行调用。
> 注： 实际使用中休眠时间不是一个精准时间。

```python
import utime

# 设置休眠 1s。
utime.sleep(1)
utime.sleep_ms(1000)
utime.sleep_us(1000*1000)
```

### 系统定时器(Timer)

系统定时器用于定时或者周期性执行某任务，该定时器基于硬件定时器功能实现，一般最多可以创建4个定时器。QuecPython 设备对于系统定时器的所有用法参见 [class Timer - 硬件定时器](../../api_reference/peripherals/machine.Timer.md) 章节。

```python
from machine import Timer

# timer 到时回调函数。
def timer_callback(t):
    print('timeout occured !')

# 创建定时器对象。
t = Timer(Timer.Timer1)

# 启动定时器，周期性 1s 执行定时器回调。
t.start(period=1000, mode=t.PERIODIC, callback=timer_callback)
```

### RTC 闹钟

用于时间到期执行指定任务，该功能实现还可用于低功耗下休眠唤醒，以及关机下进行开机唤醒。

```python
from machine import RTC

# 初始化 RTC 实时时钟
rtc = RTC()

# 定义唤醒回调函数接口，唤醒后执行该函数。
def callback(args):
   print('RTC alarm')

# 注册唤醒回调函数。
rtc.register_callback(callback)

# 设置闹钟，在某个时间唤醒。
rtc.set_alarm([2026, 7, 9, 5, 12, 30, 0, 0])

# 使能该闹钟。
rtc.enable_alarm(1)
```

## 常见问题

### 模组关机（不断电），RTC 是否还在运行？

在模块上电后，无论模块状态如何（运行模式、低功耗模式、处于复位状态或者关机状态），只要电源电压保持在工作范围内，RTC 不会停止工作，之前的时间就不会丢失。EC200A系列无法模组关机运行。

### 模组断电，时间从哪同步？

模组断电后，电路无法正常工作，此时时间会丢失，重新上电后，连接网络，模组会通过使用 NITZ 方式与基站进行时间同步。用户也可以通过 NTP 自定义服务器进行时间同步。
