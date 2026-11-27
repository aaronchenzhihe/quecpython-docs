# Accelerometer Sensor

The accelerometer sensors include angular acceleration sensors (gyroscopes) and linear acceleration sensors, which can detect the real-time acceleration, angular velocity, and attitude of the device. Here are some typical application scenarios.

- Attitude Perception: Accelerometer sensors can be used to detect the attitude and orientation changes of a device. They can be used in smart wearables, smart security systems, etc., to achieve features such as screen rotation, posture recognition, and motion sensing.
- Motion Tracking: Accelerometer sensors can be used to track and record the acceleration and motion trajectory of objects. In embedded systems, they can be used in motion monitors, sports watches, smart bracelets, etc., to achieve features such as step counting, sleep monitoring, and motion trajectory recording.
- Anti-shake Control: Accelerometer sensors can be used to control the shake and stability of devices. They can be used in applications such as camera stabilization, electronic stabilizers, and robot control to achieve image stabilization and motion control features.

## Supported List

**List of Accelerometer Sensor Models Supported by QuecPython**

| Model    | Interface | Datasheet                                                    | Code Link                                                    |
| -------- | --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| LIS2DH12 | I2C/SPI   | [LIS2DH12 Datasheet](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/lis2dh12/LIS2DH12.pdf) | [Code Link](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/lis2dh12/LIS2DH12.py) |
| ADXL346  | I2C/SPI   | [ADXL346 Datasheet](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/ADXL346/adxl346.pdf) | [Code Link](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/ADXL346/adxl346.py) |
| BMA250   | I2C/SPI   | [BMA250 Datasheet](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/bma250/BMA250-0273141121.pdf) | [Code Link](https://github.com/QuecPython/QuecPython_lib_bundles/blob/master/libraries/bma250/bma250.py) |

## Hardware

This section demonstrates how to debug an accelerometer sensor based on the LTE OPEN-EVB_V1.1, EC600U TE-A and LIS2DH12TR sensor module.

See [LIS2DH12TR Sensor Hardware Connection](./lis2dh12tr.md) for details.

## Software Design

### Workflow

Features supported by LIS2DH12:

1. Single/double-click detection
2. Free fall detection
3. Tilt angle measurement
4. Switch between landscape/portrait mode

Enable the single-click detection feature. When a single-click event occurs, map it to the INT1 pin. The processing logic is roughly as follows:

![media_i2c_lis2dh_3](../../../media/hardware/sensors/gsensor_3.png)

LIS2SH12 initialization steps:

1. Set the CTRL_REG2 register to enable high-pass filtering.
2. Set the CTRL_REG3 register to route the interrupt to the INT1 pin.
3. Set the CTRL_REG4 register to configure the full range.

Single-click interrupt configuration steps:

1. Configure the CLICK_CFG register to enable the required sensing axis, X, Y, Z.
2. Configure the CLICK_THS register to set the threshold.
3. Configure the TIME_LIMIT register to set the time window limit.
4. Configure the TIME_LATENCY register to set the latency.

Enable LIS2SH12 sensor:

1. Configure the CTRL_REG1 register to enable the sensor.

### API Reference

The following APIs provide the abstraction for the lis2dh12 sensor features. Users can directly reference the lis2dh12 class and call its APIs to write sensor applications.

> Different accelerometer sensors may have significant differences. If you want to write your accelerometer sensor class, please read the corresponding datasheet first and develop it based on the source code in this chapter.

**Class Reference**

```python
from lis2dh12 import lis2dh12
```

**Instantiation**

```python
i2c_dev = I2C(I2C.I2C1, I2C.STANDARD_MODE)
g_sensor = lis2dh12(i2c_dev,14)
```

**Parameter Description**

- `i2c_obj` – Object type. I2C object. Required parameter.
- `int_pin` – Integer type. GPIO number connected to the INT1 pin of the sensor. Required parameter.
- `dev_addr` – Integer type. I2C salve address. Optional parameter. Default value: 0x19.

> Note: If the SDO/SA0 pin is at a high level, `dev_addr` is 0x19; if the SDO/SA0 pin is at a low level, `dev_addr` is 0x18.

#### **`lis2dh12.sensor_reset`**

Reset the sensor.

#### **`lis2dh12.int_enable`**

Enable the interrupt.

**Parameter Description:**

- `int_type` – Integer type. Interrupt type. Required parameter.<br />

  0x01: Single-click on the x-axis

  0x04: Single-click on the y-axis

  0x10: Single-click on the z-axis

  0x15: Single-click on all three axes

  0x02: Double-click on the x-axis

  0x08: Double-click on the y-axis

  0x20: Double-click on the z-axis

  0x2A: Double-click on all three axes

  0x95: Free fall

- `int_ths` – Integer type. Interrupt threshold. Required parameter.

- `time_limit` – Integer type. Time limit (single/double-click events). Optional parameter. Default value: 0x18.

- `time_latency` – Integer type. Latency (double-click events). Optional parameter. Default value: 0x12.

- `time_window` – Integer type. Time window (double-click events). Optional parameter. Default value: 0x55. The double-click must be completed within this time.

- `duration` – Integer type. Duration of the event. Optional parameter. Default value: 0x03.
> `int_ths` is used to set the threshold to trigger an interrupt. When the measured value of the sensor (input acceleration) exceeds or falls below this threshold, an interrupt event is triggered.
>
>`time_limit` is used to set the time limit for a click event. If the time interval between two input accelerations exceeding the threshold is less than this time, a click event is considered to have occurred.
>
>`time_latency` is used to set the delay time for a double-click event. After the first click event, wait for a latency time before detecting the second click event. If the second click event occurs within the latency time, a double-click event is considered to have occurred.
>
>`time_window` is used to set the time window for a double-click event. The time window defines the time range for the second triggering event.
>
>`duration` is used to specify the duration threshold for an event. When the duration of an event reaches or exceeds this threshold, the corresponding processing logic or interrupt can be triggered.

#### **`lis2dh12.start_sensor`**

Starts the sensor (enables the `x, y and z` axes).

#### **`lis2dh12.read_acceleration`**

Reads acceleration of three axes.

**Return Value**

- `acceleration` – Tuple type. acceleration of three axes `(x,y,z)`. Unit: `G`.

#### **`lis2dh12.set_int_callback`**

Sets the interrupt callback function ( `INT1`  pin).

#### **`lis2dh12.set_mode`**

Sets the working mode.

**Parameter Description**

- `mode` – Integer type. The acceleration working mode of three axes. Required parameter.

  0: High-resolution mode

  1: Normal mode

  2: Low-power mode.

### Experimental Design

1. Use the INT1 pin of the LIS2DH12 sensor to generate an interrupt.
2. Poll the status of the INT1 pin. When a rising edge is detected, it indicates that an interrupt has occurred. You need to process the interrupt.
3. Read the status of the three axes in the interrupt function.

### Experimental Code

The sensor class code is designed as follows:

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
        # Reset the chip
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
> **Note**
>
> - The parameter of `int_enable` needs to be determined according to the datasheet and specific application scenarios. For example, if you want to avoid frequent false triggering of the interrupt, the parameters `int_ths` and `duration` must be set greater, and the values depend on the test results.
> - When the three-axis sensor is placed vertically with the z-axis upwards and there is no external force, the acceleration of the `x`, `y`, and `z` axes is basically (0,0,1), in units of `G`. This can be used to determine whether the sensor reading and acceleration calculation are normal.
> - It is recommended to use [external interrupt](../../../../api_reference/peripherals/machine.ExtInt.md) to handle interrupts, rather than polling the interrupt register of the sensor. The latter method will prevent entering low-power mode.

The main program code is designed as follows:

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

## Verification

Download the experimental code to the module and run it. When you click the sensor, you can see the content in the figure below printed on the "REPL" interface. The tuples in parentheses are the accelerations of the three axes at the moment of triggering, in units of `G`.

![image-20230630145011109](../../../media/hardware/sensors/gsensor_4.png)