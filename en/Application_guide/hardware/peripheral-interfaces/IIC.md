# IIC

## Briefing on IIC

IIC (Inter-Integrated Circuit) bus is a two-wire serial bus developed by PHILIPS. It's used to connect micro-controller and its peripherals. IIC and I2C are the same thing, it's just the name difference.
The serial bus, composed by Serial Data (SDA) wire and Serial Clock (SCL) wire, is able to send and receive data. An IIC device can either be a master or a slave. Normally, a master would be paired with one or more slaves and responsible for starting and ending a communication session. At the time being, QuecPython cellular modules can be a master only.

IIC Topology：

<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_topology.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">



- Data (SDA): To transmit data. 
  Clock (SCL): To sync the data transmission.
- Every device on the bus owns a unique address. As a result, the communication between micro-controller and devices can be implemented in accordance with the timing of corresponding address. 
- As bidirectional lines, both SDA and SCL are connected to a positive supply voltage via a current-source or pull-up resistor. When the bus is idle, both lines are in high level.
- The transmission rate over the bus is up to 100 kbit/s in the Standard-mode, up to 400 kbit/s in the Fast-mode, and up to 3.4 Mbit/s in the High-speed mode.

## IIC Communication Timing Introduction		

- **Idle Level**: SCL and SDA both are in high level.
- **START Condition**: A HIGH to LOW transition on the SDA line while SCL is HIGH (IDLE). 

<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_start.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- **STOP Condition**: A LOW to HIGH transition on the SDA line while SCL is HIGH (IDLE). 

<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_stop.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- **Acknowledge Condition**: The acknowledge signal sent by the receiving end after it received valid data. After every byte (8 bits) of data, at the ninth clock, the transmitting end releases SDA line to receive acknowledge signal from the receiving end.
If SDA is low, the Acknowledge (ACK) is valid and the transmission is successful. If SDA is high, it will be a Not Acknowledge (NACK) and indicates the transmission has failed.

- **Data Transmission**: Data bits are allowed to change while SCL is LOW. After each 8 bits of data is tramsmitted, the slave would pull down the SDA (ACK) or pull up the SDA  (NACK). 

<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_time_sequence.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

## IIC Support on Different Platforms			
All modules support hardware IIC with both 100k and 400k frequency.

## Software Simulated IIC						
Software Simulated IIC simulates IIC protocol by controlling GPIO pin levels in software.
If the hardware IIC are not enough or are multiplexed as other functions, it is available to use simulated IIC to implement IIC communication.

<!--Some of our modules support simulated IIC with customizable 1~100k frequency. Visit [WIKI-I2C_simulation](../../../api_reference/peripherals/machine.I2C_simulation.md) for more information on setting customized frequency.(https://developer.quectel.com/doc/quecpython/API_reference/zh/peripherals/machine.I2C_simulation.html)。
Below are modules that support IIC simulation. -->

- EC600M-CN Series
- EC800M-CN Series
- EC600U Series
- EG915U Series
- EC200U Series
- EC200A Series
- BG95 Series
- BC25 Series
- EG912N-EN Series
- EG915N Series
- EC600N-CN Series
- EC800N-CN Series

## IIC Function Illustration								

This chapter will introduce how we can use IIC driver's functionalities and data types to build communication between QuecPython series modules and other IIC devices. The typical programming workflow consists of following parts:

1. <a href="#label_creatobject">Create an object</a>
2. <a href="#label_senddata">Send data</a>
3. <a href="#label_receivedata">Receive data</a>


For specific introduction on API, please refer to [machine.IIC](../../../api_reference/peripherals/machine.I2C.md) in detail. 


### <span id="label_creatobject">Create an Object</span>

It is mandatory to specify IIC channel and working mode when creating an IIC object.

```python
class machine.I2C(I2Cn, MODE)
```

For the parameters introduction and pin relationships, please refer to [machine.IIC](../../../api_reference/peripherals/machine.I2C.md)

When creating an object, please note following aspects. 

- In a project, only one object can be created for one channel. When communicating with multiple IIC peripherals via one channel of IIC, it is only necessary to create one IIC object in one of the peripheral drivers. 
- In standard mode, the IIC communication baud-rate is 100k. While In fast mode, it is 400k instead. Note that the baud-rate shall never exceed the maximum supported baudrate of IIC peripheral devices.

### <span id="label_senddata">Send Data</span>

```python
I2C.write(slaveaddress, addr, addr_len, data, datalen)
```

When sending data over IIC, a couple of points should be noted:


- The  `slaveaddress` is a 7-bit address. When the master sends the slave address, the address will be shifted 1 bit to the left. After that, when writing data to the slave, the Least Significant Bit (LSB) will be a zero supplemented. When reading data from the slave, the LSB will be a one supplemented instead. For example, an IIC peripheral device has an address of 0x23, the master should send 0x46 if it want to send data to the slave, otherwise 0x47 shall be sent to read data from the slave. 
- The `addr_len` refers to the address length of the register. You can get this information from the peripheral's datasheet. Commonly used IIC peripherals usually have an register address length of 1 byte such as QMA7981 3-Axis Accelerometer. 
<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_reg_lenth.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

For API introduction, please refer to [machine.UART.write](../../../api_reference/peripherals/machine.UART.md#uart.write).


### <span id="label_receivedata">Receive Data</span>

```python
I2C.read(slaveaddress, addr, addr_len, r_data, datalen, delay)
```

When receiving data over IIC, a couple of points should be noted:


**1. Buffer management：** It is vital to manage buffer correctly. If data is not read before being overwritten by new data, there might be a data loss. Therefore, ensure data are read and processed in time before it's overwritten.

**2. The `delay` parameter：** After receiving commands from the slave, some IIC peripherals may need some delay before returning data correctly. So as a master, a delay is needed between sending command and reading data. See next figure.

<img src="../../media/hardware/peripheral-interfaces/IIC/IIC_read_delay.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

For API and parameter introductions, please refer to [machine.UART.read](../../../api_reference/peripherals/machine.UART.md#uart.read)


## Application Examples							

### 3-Axis Accelerometer QMA7981				

The QMA7981 is a single-chip three-axis accelerometer. It has integrated acceleration transducer with signal conditioning ASIC, sensing tilt, motion, shock and vibration, targeted for applications such as screen rotation, step counting, sleep quality, gaming and personal navigation in mobile and wearable smart devices.

The QMA7981 has following features:


- With custom-designed 14-bit ADC ASIC, it offers the advantages of low noise, high accuracy, low power consumption, and offset trimming.
- Built-In Self-Test feature for low-cost functionality testing during production.
- Built–in motion algorithm for step counting, sleep quality, gaming and personal navigation.
- Working voltage range: 1.71V~3.6V. Working temperature range: -40°C~+85°C. 

#### QMA7981 Diagram							

<img src="../../media/hardware/peripheral-interfaces/IIC/QMA7981_1.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

When using QMA7981, a couple of points should be noted:


- QMA7981 has 1 IIC interface which the module can get data from.
- QMA7981 supports two external interrupts at maximum, configurable LOW or HIGH trigger. Note that the QMA7981's interrupt config should corresponds to that in module. For example: QMA7981 is set to trigger interruption at HIGH, then the module's interrupt pin should be set to normal LOW and trigger by rising edge.

```python
self.extint = ExtInt(INT1, ExtInt.IRQ_RISING, ExtInt.PULL_PD, self.ext_cb)
data = self._read([self.INTPIN_CONF_ADDR], 1)
# set logic high as active level for INT1 pin
data[0] |= 0x01
print('INTPIN_CONF_ADDR: {}'.format(data))
self._write([self.INTPIN_CONF_ADDR], [data[0]])
```

- When AD0 is connected to the ground, IIC slave address is 0x12. When AD0 is connected to VDDIO, IIC slave address is 0x13.
- QMA7981 has two modes: Standby Mode and Active Mode. When powered on, it goes into Standby Mode by default. By setting the PM register (0x11) , it will enter Active Mode.

<img src="../../media/hardware/peripheral-interfaces/IIC/QMA7981_PM_REG.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- When QMA7981 is stationary and level, Z axis will have 1G of acceleration due to gravity. X axis and Y axis should remain 0.

To drive the QMA7981, please refer to [QMA7981](https://github.com/QuecPython/examples/tree/main/hardware/peripheral-interfaces/IIC/qma7981.py). 

### AHT10 Temperature & Humidity Sensor			
The AHT10 sensor is equipped with a newly designed ASIC, an improved MEMS semiconductor capacitive humidity sensing element and a standard on-chip temperature sensing element. It can be used in a variety of application such as: HVAC, Dehumidifiers, Test and Inspection Equipment, Consumer Products, Automotive, Automation, Data Loggers, Weather Stations, Appliances, Humidity Conditioning, Medical, and other related temperature and humidity detection and control. 

AHT10 works by:									

- Powering up the sensor. The sensor will take up to 20ms (SCL will be at HIGH) to get into idle status after being powered, then the sensor is ready to take orders from the master (Cellular Module). 
- The master must communicate with AHT10 via IIC and the slave address of AHT10 is 0x38. After the master issues the initialization command ('11100001'=Initialization, '10101100'=Measurement), it must wait for the measurement till the end. Basic commands are listed in the below table.

<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_cmd.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

Below is a table regarding the return value of the sensor.

<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_state.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

- Issue the collect data command (0xAC) to trigger AHT10 to collect temp & humidity data. Sleep for at least 75ms to wait for the sensor and read collected data from AHT10 afterwards.。

```python
def Trigger_measurement(self):
    # Trigger data conversion
    self.write_data([self.AHT10_START_MEASURMENT_CMD, 0x33, 0x00])
    time.sleep_ms(200)  # At least 75ms
    # check has success
    r_data = self.read_data(6)
    # check bit7
    if (r_data[0] >> 7) != 0x0:
        print("Conversion has error")
    else:
        self.aht10_transformation_temperature(r_data[1:6])
```

- After acquiring the data form AHT10, a conversion should be made on the raw data to get desired information.  
**Relative Humidity (RH)** Can be calculated from the relative humidity signal SRH from SDA using the formula below 

<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_calculate_humidity.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

**Temperature (T)T** can be calculated from the temperature signal ST using the formula below

<img src="../../media/hardware/peripheral-interfaces/IIC/AHT10_calculate_temp.png" style="border-style: solid; border-radius: 10px; color:  #f1f1f1;" border=3 alt="">

```python
def aht10_transformation_temperature(self, data):
    r_data = data
    #　Convert raw data into temperature and humidity
    humidity = (r_data[0] << 12) | (
        r_data[1] << 4) | ((r_data[2] & 0xF0) >> 4)
    humidity = (humidity/(1 << 20)) * 100.0
    print("current humidity is {0}%".format(humidity))
    temperature = ((r_data[2] & 0xf) << 16) | (
        r_data[3] << 8) | r_data[4]
    temperature = (temperature * 200.0 / (1 << 20)) - 50
    print("current temperature is {0}°C".format(temperature))
```

To drive the AHT10, please refer to [Driving AHT10](https://github.com/QuecPython/examples/tree/main/hardware/peripheral-interfaces/IIC/aht10_test.py).

## FAQ 

IIC is a very simple and straightforward communication protocol. There isn't much issue when it comes to practical use.


**1. Fail to receive or send data**

This might be caused by something like wrong baud-rate and wrong HW connection. You may check as below. 

- Check if module IIC baud rate is above the peripheral limit. 
- Check hardware connection. Make sure SDA/SCL and GND lines are connected properly. 

**2. It is normal to transmit/receive, however, the data are wrong**

It could be:

- Voltage level of SDA/SCL in master are different from that in slave. When the master uses 3.3V while the slave uses 1.8V, a level of 1.8V is considered HIGH by peripheral but controller would think a level higher than 1.65V is considered HIGH, resulting in confusion probably. 
- The pull-up resistors in IIC circuit are high enough to hinder its ability to pull up. This will stretch the rising edge as well as the falling edge, leding to error in master. 
