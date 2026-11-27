# 加速度传感器

加速度传感器包含角加速度传感器（陀螺仪）、线加速度传感器等，能够实时检测设备的加速度、角速度和姿态等数据，以下是一些典型应用场景。

- 姿态感知：加速度传感器可以用于检测设备的姿态和方向变化。它们可以用于智能穿戴，摄影摄像等设备中，实现屏幕旋转、姿态识别和动作感应等功能。
- 运动追踪：加速度传感器可以用于追踪和记录物体的加速度和运动轨迹。在嵌入式系统中，它们可以用于运动监测器、运动手表、智能手环等设备中，实现步数统计、睡眠监测和运动轨迹记录等功能。
- 防抖动控制：加速度传感器可以用于控制设备的抖动和稳定性。它们可以用于相机防抖功能、电子稳定器、机器人控制等应用中，实现图像稳定和运动控制功能。

## 支持列表

**Quecpython目前已支持的加速度传感器型号表**

| 型号     | 接口    | 规格书                                                       | 代码链接                                                     |
| -------- | ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| LIS2DH12 | I2C/SPI | [规格书](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/lis2dh12/LIS2DH12.pdf) | [代码链接](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/lis2dh12/LIS2DH12.py) |
| ADXL346  | I2C/SPI | [规格书](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/ADXL346/adxl346.pdf) | [代码链接](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/ADXL346/adxl346.py) |
| BMA250   | I2C/SPI | [规格书](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/bma250/BMA250-0273141121.pdf) | [代码链接](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/bma250/bma250.py) |

## 硬件介绍

本章节主要基于 LTE OPEN-EVB_V1.1和EC600U TE-A和LIS2DH12TR传感器模块，演示如何调试一款加速度传感器。

详见[LIS2DH12TR传感器硬件连接](./lis2dh12tr.md)。

## 软件设计

### 工作流程

LIS2DH12 支持的功能：

1.  单双击检测

2.  自由落体检测

3.  倾斜角测量

4.  切换横屏/竖屏模式

我们使用其单击检测功能，出现单击事件，将其映射到INT1 引脚上面，其处理逻辑大致如下：

![media_i2c_lis2dh_3](../../../media/hardware/sensors/gsensor_3.png)

LIS2SH12 初始化步骤如下：

1.  设置 CTRL_REG2 寄存器，开启高通滤波。
2.  设置 CTRL_REG3 寄存器，将中断引到INT1 引脚上面。
3.  设置 CTRL_REG4 寄存器，配置为满量程。

配置单击中断步骤如下：

1.  配置 CLICK_CFG 寄存器，使能需要检测的感应轴，X,Y,Z。
2.  配置 CLICK_THS 寄存器，设置阈值。
3.  配置 TIME_LIMIT 寄存器，设置窗口限制。
4.  配置 TIME_LATENCY 寄存器，设置延时。

LIS2SH12 使能传感器

1.  配置CTRL_REG1 寄存器，开始使能传感器。

### API参考

以下 API 实现了对lis2dh12传感器的功能抽象，用户可直接引用lis2dh12类，调用其API编写传感器应用程序。

> 不同加速度传感器的区别可能较大，用户若想编写自己的加速度传感器类，请先阅读对应规格书，可仿照本章源码进行开发。

**类引用：**

```python
from lis2dh12 import lis2dh12
```

**实例化：**

```python
i2c_dev = I2C(I2C.I2C1, I2C.STANDARD_MODE)
g_sensor = lis2dh12(i2c_dev,14)
```

**参数描述：**

- `i2c_obj` - I2C对象，必传，obj类型。
- `int_pin` - 连接传感器int1的引脚gpio号，必传，int类型。
- `dev_addr` - I2C从设备地址，非必传，int类型，缺省值0x19。

> 注意：SDO/SA0 引脚为高电平，`dev_addr`为0x19；SDO/SA0 引脚为低电平，`dev_addr`为0x18。

#### **`lis2dh12.sensor_reset`**

重置传感器。

#### **`lis2dh12.int_enable`**

中断使能。

**参数描述：**

- `int_type` - 中断类型（可配单双击中断，自由落体中断），必传，int类型。<br />x轴单击：0x01，y轴单击：0x04，z轴单击：0x10，三轴单击：0x15<br />x轴双击：0x02，y轴双击：0x08，z轴双击：0x20，三轴双击：0x2A<br />自由落体：0x95。
- `int_ths` - 中断阈值，必传，int类型。
- `time_limit` - 时间限制(单双击事件)，非必传，int类型，缺省值0x18。
- `time_latency` - 延时(双击事件设置)，非必传，int类型，缺省值0x12。
- `time_window` - 时间窗口(双击事件设置)，非必传，int类型，缺省值0x55。双击得在该段时间内完成。
- `duration` - 事件的持续时间，非必传，int类型，缺省值0x03。

> `int_ths`用于设置触发中断的阈值。当传感器测量值（输入加速度）超过或低于该阈值时，会触发中断事件。
>
> `time_limit`用于设置点击事件的时间限制。如果检测到两次输入加速度超过阈值的时间间隔小于该时间，则认为发生了点击事件。
>
> `time_latency`用于设置双击事件的延时时间。在第一次点击事件之后，延时一段时间后开始检测第二次点击事件，如果第二次点击事件在延时时间内发生，则认为发生了双击事件。
>
> `time_window`用于设置双击事件的时间窗口。时间窗口定义了第二次触发事件的时间范围。
>
> `duration`用于指定某个事件的持续时间阈值。当事件的持续时间达到或超过该阈值时，可以触发相应的处理逻辑或中断。

#### **`lis2dh12.start_sensor`**

启动传感器（使能`xyz`轴）。

#### **`lis2dh12.read_acceleration`**

读取三轴加速度。

**返回值描述：**

- `acceleration` - 三轴加速度`(x,y,z)`，tuple类型，单位：`G`。

#### **`lis2dh12.set_int_callback`**

设置中断回调。（`int1`脚）。

#### **`lis2dh12.set_mode`**

设置工作模式。

**参数描述：**

- `mode` - 三轴加速度工作模式，必传，int类型，0：高分辨率模式，1：普通模式，2：低功耗模式。

### 实验设计

1.  使用LIS2DH12 传感器的 INT1 引脚产生中断。
2.  轮询此引脚的状态，检测到上升沿以后， 表示中断产生，处理中断。
3.  在中断函数里面读取三轴的状态。

### 实验代码

传感器类代码设计如下：

```python
class lis2dh12(object):
    '''
    lis2dh12 class
    '''
    def __init__(self, i2c_dev, int_pin, slave_address=0x19):
        '''
        :param i2c_dev: i2c object
        :param int_pin: gpio of pin which is connected with int1_pin
        :param slave_address: device address
        '''
        self._address = slave_address
        self._i2c_dev = i2c_dev
        self._int_pin = int_pin
        self._extint = None
        self._sensor_init()
    
    def _read_data(self, regaddr, datalen):
        '''
        i2c read data
        :param regaddr: register address
        :param datalen: length of reading data
        :return: data
        '''
        r_data = bytearray(datalen)
        reg_addres = bytearray([regaddr])
        self._i2c_dev.read(self._address, reg_addres, 1, r_data, datalen, 1)
        ret_data = list(r_data)
        return ret_data

    def _write_data(self, regaddr, data):
        '''
        i2c write data
        :param regaddr: register address
        :param data: data to write
        '''
        addr = bytearray([regaddr])
        w_data = bytearray([data])
        self._i2c_dev.write(self._address, addr, len(addr), w_data, len(w_data))

    def sensor_reset(self):
        '''
        reset the sensor
        '''
        # 重置chip
        self._write_data(LIS2DH12_CTRL_REG5, 0x80)

        print('reboot already. {}'.format(self._read_data(LIS2DH12_CTRL_REG5,1)))
        utime.sleep_ms(100)
        r_data = self._read_data(LIS2DH12_WHO_AM_I, 1)
        while r_data[0] != 0x33:
            r_data = self._read_data(LIS2DH12_WHO_AM_I, 1)
            utime.sleep_ms(5)

    def _sensor_init(self):
        '''
        initialize the sensor
        '''
        self.sensor_reset()

        self._write_data(LIS2DH12_CTRL_REG1, 0x77)  # set ODR 400HZ ,enable XYZ.
        utime.sleep_ms(20)  # (7/ODR) = 18ms
        self._write_data(LIS2DH12_CTRL_REG4, 0x08)  # ±2g

        self._write_data(LIS2DH12_CLICK_CFG, 0)  # clear click_cfg
        self._write_data(LIS2DH12_INT1_CFG, 0)  # clear int1_cfg
        self._write_data(LIS2DH12_INT2_CFG, 0)  # clear int2_cfg

    def int_enable(self,int_type,int_ths=0x12,time_limit=0x18,time_latency=0x12,time_window=0x55,duration=0x03):
        '''
        interrupt enable
        :param int_type: type of interrupt
        :param int_ths: threshold
        :param time_limit: click_int to send this parameter, time window limit
        :param time_latency: click_int to send this parameter, set the time_latency
        :param duration:
        '''
       # single_click int
        if int_type in (XYZ_SINGLE_CLICK_INT, X_SINGLE_CLICK_INT, Y_SINGLE_CLICK_INT, Z_SINGLE_CLICK_INT):
            self._write_data(LIS2DH12_CTRL_REG2, 0x07)  # Enable high pass filter for click function
            self._write_data(LIS2DH12_CTRL_REG3, 0x80)  # Bind interrupt to INT1 pin, default high level is valid
            self._write_data(LIS2DH12_CTRL_REG5, 0x08)  # INT1 latch
            self._write_data(LIS2DH12_CLICK_CFG, int_type)  # enable click_int
            self._write_data(LIS2DH12_CLICK_THS, int_ths)  # set threshold
            self._write_data(LIS2DH12_TIME_LIMIT, time_limit)  # set time_limit
        # double_click int
        elif int_type in (XYZ_DOUBLE_CLICK_INT, X_DOUBLE_CLICK_INT, Y_DOUBLE_CLICK_INT, Z_DOUBLE_CLICK_INT):
            self._write_data(LIS2DH12_CTRL_REG2, 0x07)
            self._write_data(LIS2DH12_CTRL_REG3, 0x80)
            self._write_data(LIS2DH12_CTRL_REG5, 0x08)
            self._write_data(LIS2DH12_CLICK_CFG, int_type)
            self._write_data(LIS2DH12_CLICK_THS, int_ths)
            self._write_data(LIS2DH12_TIME_LIMIT, time_limit)
            self._write_data(LIS2DH12_TIME_LATENCY, time_latency)
            self._write_data(LIS2DH12_TIME_WINDOW, time_window)
        # 6d int
        elif int_type in (MOVE_RECOGNIZE, X_MOVE_RECOGNIZE, Y_MOVE_RECOGNIZE, Z_MOVE_RECOGNIZE,POSI_CHANGE_RECOGNIZE,
                          X_POSI_CHANGE_RECOGNIZE,Y_POSI_CHANGE_RECOGNIZE,Z_POSI_CHANGE_RECOGNIZE,FF_RECOGNIZE):
            self._write_data(LIS2DH12_CTRL_REG2, 0x00)  # switch off the high pass filter
            self._write_data(LIS2DH12_CTRL_REG3, 0x40)
            self._write_data(LIS2DH12_CTRL_REG5, 0x08)
            self._write_data(LIS2DH12_INT1_CFG, int_type)  # enable 6d int
            self._write_data(LIS2DH12_INT1_THS, int_ths)
            self._write_data(LIS2DH12_INT1_DURATION, duration)  # set duration

    def start_sensor(self):
        '''
        start the sensor
        '''
        self._write_data(LIS2DH12_CTRL_REG1, 0x77)  # ODR 100HZ ,enable XYZ.
        utime.sleep_ms(20)  # (7/ODR) = 18ms

    def process_xyz(self):
        '''
        Read registers and convert x-axis, y-axis, and z-axis data
        :return: x,y,z data
        '''
        data = []
        ctl4 = self._read_data(LIS2DH12_CTRL_REG4, 1)[0]
        big_endian = ctl4 & 0x40
        # read xl,xh,yl,yh,zl,zh
        for i in range(6):
            r_data = self._read_data(LIS2DH12_OUT_X_L + i, 1)
            data.append(r_data[0])
        if big_endian:
            x = data[0] * 256 + data[1]
            y = data[2] * 256 + data[3]
            z = data[4] * 256 + data[5]
        else:
            x = data[1] * 256 + data[0]
            y = data[3] * 256 + data[2]
            z = data[5] * 256 + data[4]
        return (x, y, z)

    def int_processing_data(self):
        '''
        handle int_processing
        :return: x,y,z-axis acceleration
        '''
        acc = self.read_acceleration
        int_src = self._read_data(LIS2DH12_INT1_SRC,1)  # read INT1_SRC，clear interrupt request
        return acc

    @property
    def _resolution(self):
        """
        resolution range.
        :return: range_2_G, range_4_G, range_8_G,, range_16_G.
        """
        ctl4 = self._read_data(LIS2DH12_CTRL_REG4,1)[0]
        return (ctl4 >> 4) & 0x03

    @property
    def _acceleration(self):
        """
        x,y,z-axis acceleration
        :return: x,y,z-axis acceleration
        """
        divider = 1
        accel_range = self._resolution
        if accel_range == 3:        # range_16_G
            divider = 2048
        elif accel_range == 2:      # range_8_G
            divider = 4096
        elif accel_range == 1:      # range_4_G
            divider = 8192
        elif accel_range == 0:      # range_2_G
            divider = 16384
        x, y, z = self.process_xyz()
        x = x / divider
        y = y / divider
        z = z / divider
        if accel_range == 3:        # range_16_G
            print('range_16_G')
            x = x if x <= 16 else x - 32
            y = y if y <= 16 else y - 32
            z = z if z <= 16 else z - 32
        elif accel_range == 2:      # range_8_G
            print('range_8_G')
            x = x if x <= 8 else x - 16
            y = y if y <= 8 else y - 16
            z = z if z <= 8 else z - 16
        elif accel_range == 1:      # range_4_G
            print('range_4_G')
            x = x if x <= 4 else x - 8
            y = y if y <= 4 else y - 8
            z = z if z <= 4 else z - 8
        elif accel_range == 0:      # range_2_G
            print('range_2_G')
            x = x if x <= 2 else x - 4
            y = y if y <= 2 else y - 4
            z = z if z <= 2 else z - 4
        return (x, y, z)

    @property
    def read_acceleration(self):
        '''
        read acceleration
        :return: x,y,z-axis acceleration
        '''
        while 1:
            status = self._read_data(LIS2DH12_STATUS_REG,1)[0]
            xyzda = status & 0x08   # if xyz data exists, set 1
            xyzor = status & 0x80
            if not xyzda:
                continue
            else:
                x,y,z = self._acceleration
                return (x, y, z)


    def set_mode(self,mode):
        """
        set work mode
        :param mode: 0: High resolution mode; 1: Normal mode; 2: Low power mode;
        :return: None
        """
        if mode == 0:
            self._write_data(LIS2DH12_CTRL_REG1, 0x77)  # ODR 400HZ ,enable XYZ.
            self._write_data(LIS2DH12_CTRL_REG4, 0x08)  # ±2g， High resolution mode
        elif mode == 1:
            self._write_data(LIS2DH12_CTRL_REG1, 0x57)  # ODR 100HZ ,enable XYZ.
            self._write_data(LIS2DH12_CTRL_REG4, 0x08)  # ±2g， Normal mode
        elif mode == 2:
            self._write_data(LIS2DH12_CTRL_REG1, 0x8f)
            self._write_data(LIS2DH12_CTRL_REG4, 0x08)  # ±2g， Low power mode
        else:
            print("wrong mode.")

    def set_int_callback(self, cb):
        self._extint = ExtInt(self._int_pin, ExtInt.IRQ_FALLING, ExtInt.PULL_PU, cb)

```

> **注意**
>
> - 中断使能接口`int_enable`的参数需结合规格书和具体应用场景决定，比如希望中断不会经常误触发，则阈值参数`int_ths`和持续时间参数`duration`需要稍微设置较大，数值随具体测试效果而定。
> - 三轴传感器在z轴竖直向上放置且没有外力作用的时候，`x`,`y`,`z`轴的加速度基本是(0,0,1)，单位`G`。据此可以判断传感器读取计算加速度是否正常。
> - 开发的时候尽量用[外部中断](../../../../api_reference/peripherals/machine.ExtInt.md)的方式来处理中断，而非轮询传感器中断寄存器的方式，后者会导致进不了低功耗模式。

主程序代码设计如下：

```python
def int_cb(args):
    print('click just happened...')
    acc = dev.int_processing_data()
    print("read_acceleration result: ", acc)

if __name__ == "__main__":
    # initialize i2c
    i2c_dev = I2C(I2C.I2C1, I2C.STANDARD_MODE)
    # initialize the lis2dh12 object
    dev = lis2dh12(i2c_dev, 14)
    # enable single_click_interrupt
    dev.int_enable(XYZ_SINGLE_CLICK_INT)
    # set interrupt callback
    dev.set_int_callback(int_cb)
    # start sensor
    dev.start_sensor()
    # LP mode
    dev.set_mode(2)

    print("interrupt detecting...")
```

## 效果验证

​	将实验代码下载至模组中运行，单击传感器，可见交互界面出现如下图的打印，括号内打印的是触发瞬间的三个轴的加速度，单位为`G`：

![image-20230630145011109](../../../media/hardware/sensors/gsensor_4.png)



