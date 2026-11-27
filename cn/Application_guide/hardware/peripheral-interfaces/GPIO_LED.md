# LED

## 场景描述

我们将使用QuecPython创建一个LED类来驱动一个LED。通过这个类，我们可以实现LED的开启、关闭、反转等功能。这种设计使得控制LED变得更加模块化和可扩展

## 设计

### 硬件

 **LED连接**: 将LED的阳极通过一个限流电阻连接到通信模组的GPIO引脚，而LED的阴极连接到（GND）。

### 软件

用户接口定义

```python
LED(gpio)
```

- 功能：创建LED对象
- 返回：LED对象
- LED：LED类
- gpio：控制LED的GPIO编号。 详细请参考[machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin)

```python
LED.on()
```

- 功能：打开LED灯

```python
LED.off()
```

- 功能：关闭LED灯

```python
LED.toggle()
```

- 功能：翻转LED灯状态



软件设计：

- 导入模块
  - 在QuecPython中，你可以使用`machine`模块来控制GPIO。
- 定义LED类
  - 创建一个LED类，其中包含开启、关闭、反转等方法。

```python
import machine
import utime

class LED:
    def __init__(self, pin_number):
        self.pin = machine.Pin(pin_number, machine.Pin.OUT)
        self.state = False  # 初始状态为关闭

    def on(self):
        self.pin.write(1)
        self.state = True
        print("on")

    def off(self):
        self.pin.write(0)
        self.state = False
        print("off")

    def toggle(self):
        if self.state:
            self.off()
        else:
            self.on()

if __name__ == "__main__":
    # 创建一个LED对象，连接到GPIO2
    led = LED(2)

    while True:
        led.toggle()  # 反转LED状态
        utime.sleep(1)  # 等待1秒

```



## 扩展项

调整闪烁速率：

```python
while True:
    led.toggle()
    time.sleep(0.5)  # 改为0.5秒，实现更快的闪烁
```

控制多个LED:

```python
class MultiLED:
    def __init__(self, pin_numbers):
        self.leds = [LED(pin) for pin in pin_numbers]

    def all_on(self):
        for led in self.leds:
            led.on()

    def all_off(self):
        for led in self.leds:
            led.off()

# 创建一个MultiLED对象，连接到GPIO2和GPIO3
multi_led = MultiLED([2, 3])
multi_led.all_on()  # 打开所有LED

```



## 实验

本节主要基于EC600U TE-A和 LTE OPEN-EVB_V1.1，演示如何控制LED灯。

准备材料：LED灯，QuecPython EC600U模组，LTE OPEN-EVB_V1.1 。

为了方便测试，我们可以直接使用 LTE OPEN-EVB_V1.1 上的LED灯，比如NET_STATUS 网络指示灯。

![image-20230705163113093](../../media/hardware/peripheral-interfaces/GPIO/GPIO_EVB_1.png)

> NET_STATUS 默认作为网络指示灯使用，我们需要先通过API关闭网络灯功能。禁止网络灯功能相关API介绍:[misc](../../../api_reference/peripherals/misc.md)
>
> 可以直接在EC600U模组上运行以下代码关闭网络灯
>
> ```python
> import misc
> misc.net_light(0)# 设置关闭网络灯后重启模组,参数:0-关闭,1-开启
> ```



网络灯功能关闭后，按照以下几个步骤执行。

1. 通过的硬件设计手册找到 EC600U 模组控制网络灯的引脚，查找后可以得知EC600U的 NET_STATUS 由引脚号54控制。
2. 获取到引脚号之后，你可以通过 [machine.Pin](../../../api_reference/peripherals/machine.Pin.md) 模块**GPIO引脚编号与物理引脚的映射关系**找到EC600U 模组引脚号54对应的GPIO号为GPIO14。
3. 编写控制LED灯脚本并运行。

使用QPYcom运行以上代码后，我们就可以观察到 NET_STATUS 在以1秒的时间间隔不断闪烁。

> 请注意，模组控制 NET_STATUS 的GPIO的编号可能会因不同的平台而有所不同。

