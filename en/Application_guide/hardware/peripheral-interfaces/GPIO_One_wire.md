# One-Wire Protocol

## Protocol Introduction		
<!-- https://www.engineersgarage.com/what-is-the-1-wire-protocol/ -->
The 1-Wire protocol is a single-wire interface, half-duplex, bidirectional, low-speed and power, long-distance serial-data communication protocol. Although this protocol is classified as a 1-wire standard, at least two wires are required in the 1-wire standard bus — one for the data and/or power supply and another for the ground return. An additional wire might be necessary, depending on the power mode.

A 1-wire standard has a controller and slave configuration, where there can only be one controller device, a computer or micro-controller, and several slave devices. It’s possible to connect up to 100-slave, 1-wire devices, with a 1-wire standard bus. However, as slave devices are added to the bus, their polling by the controller device might take more time.

This protocol does not use a clock signal. Instead, the slave devices are internally clocked and synchronized with a signal from the controller device. The controller device is solely responsible for the read and write operations of the slave devices, so they cannot initiate a data transfer on their own. What they can do is indicate their presence over the bus when the controller resets. Every controller device is identified by a 64-bit address, stored within each one-wire slave device’s ROM.

This is a low-speed serial communication standard, with a typical data speed of 15.4 kbps. The bus can be overdriven to a maximum data speed of 125 kbps. The data speeds in the 1-Wire protocol are low compared to other standard serial-data communication protocols (like UART, I2C, and SPI), but the 1-wire bus is extremely economical in production and operation. It offers a simple hardware implementation and an extremely low-power footprint.

Although the hardware is simple, the software implementation on the micro-controller side is highly complex. And despite a low power consumption, it can communicate data over relatively long distances.

The 1-Wire protocol is used by temperature sensors, real-time clocks, timers, EEPROMs, and the popular iButton. Most of these 1-wire slave devices are products of (what’s now) Maxim Integrated.


### The Standard Bus							

The 1-wire standard bus has at least two wires. One is the data line and the other is the ground return. Both master and slave have an open-drain (open-collector) connection with the data line. This is why a 4.7 K resistor typically pulls the data line up. There are two possible powering modes for the 1-wire peripheral devices: parasitic and conventional.

In parasitic mode, only the data line and ground return must be traced to the 1-wire slave device. If the conventional power mode is used, an additional positive supply line must be traced to each 1-wire slave connected to the bus.

Therefore, the 1-wire bus on a PCB might have two or three lines. The conventional powering with three lines in the 1-wire bus is more reliable.  

### Parasitic and Conventional Power			

As mentioned, 1-wire slave devices can be powered in parasitic and conventional modes. All 1-wire slave devices have three terminals: VDD, GND and data. In the parasitic mode, the VDD and GND pins are connected to the ground, so the signal and power are supplied to the slave devices on the same line (i.e. the data line).

The slave devices have an internal capacitor of 800 pF, which gets charged when the data line is HIGH. The stored charge keeps the slave live when the data line is LOW. The data line is typically pulled up by a 4.7 K resistor.

Parasitic powering requires strict timing and supply to the exact specification to maintain the slave device live without fail. This is why this mode is less reliable. Often, an additional hard pull-up is used to ascertain the power supply.

Parasitic powering:									

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_parasitic_power.png)

Parasitic powering with an additional hard pull-up:	

  ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_parasitic_power_with_pullup.png)

In a conventional powering mode, 1-wire slave devices are externally powered. An additional wire is traced to each 1-wire slave. The external power supply to the slaves ensures safe operation even in harsh, high-temperature conditions.

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_conventional_powering.png) 

Typical 1-wire devices operate in voltages from – 1.71~1.89V, 1.71~3.63V, 2.97~6.63V, and 2.97~5.25V. The current drawn ranges between 1.06~5mA. The pull-up resistor sets the current level whether devices are supplied parasitic or conventional power. 


### How it works								

This interface is not usually used in micro-controllers or microcomputers. It’s often implemented by software using bit-banging or a universal asynchronous receiver-transmitter (UART).

Communication over the data line is initiated by the controller using a reset. It pulls down the data line for 480 us and then releases it, allowing the typical pull-up resistor to pull the data line HIGH. If the slave devices are connected to the bus, they respond to the reset signal by pulling the data line LOW for 60~240 us. If the line is pulled down by the slave(s), the master confirms their presence over the bus. After 60~240 us, the peripheral(s) release the data line, so the master can start writing.

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_one_wire_rest.png)

After a reset, the master is capable to write and read data with the slave devices. Initially, it sends ROM commands such as the search ROM command (0xF0), to access the ROM address of the slave devices. After reading the ROM addresses of all the connected 1-wire slave devices, the master device can access one by sending the Match ROM command (0x55). The ROM commands are followed by the function ones.

For example, if a 1-wire temperature sensor is connected to the bus, the micro-controller can send the function commands to start the temperature conversion, read the temperature, etc. The ROM and function commands are 8-bit long.

Since the 1-Wire standard does not use any clock signal, communication of the ‘0’ and ‘1’ bits occur by setting the logical level of the data line for a specific time slot. Usually, the time slot is 60 us long. There’s also an interval of 1us between each time slot, so that the data line is pulled HIGH again by the pull-up resistor. During each 60 us time slot, 1 bit is communicated between the master and the slave. The time slot can be up to 10 times shorter if the bus is over-driven.

When the master has to write bits over the data line, it pulls the data line down.

- To write ‘0,’ the master pulls down the data line for the full 60 us time slot, then releases it for a 1 us interval between the time slots. 
- To write ‘1,’ the master pulls the data line down for a shorter period of 15 us, during the entire time slot, then releases it for a 1us interval between the time slots. 

The slave devices pulse at about the mid-time slot (i.e. 30 us in the 60 us time slot). They have a basic mono-stable multi-vibrator to detect the duration of the pulses. The ROM and function commands are 8-bit long. The data communicated is also in groups of 8 bits. The error detection is performed by an 8-bit cyclic redundancy check.

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_one_wire_write.png)

The master reads from the slave device after sending a ROM search or function command. The read operation is controlled by the master device. The master reads from the slave bit-by-bit while data is communicated to the master in groups of 8 bits. Each bit is read in a 60 us time slot (or shorter if the bus is over-driven).

The master pulls the data line down for 1 us and releases it. Then, it samples the data from the bus after 15 us. If the slave writes ‘0’ over the bus, it keeps the line pulled down for the full 60 us of time slot, and then releases the data line for a 1 us interval between the time slots. If the slave writes ‘1’ over the bus, it keeps the line pulled down for 15 us, and then releases the data line for the pull-up resistor to pull the data line HIGH.

The master samples each bit after 15 us. If the bit sent by the slave is ‘0,’ the line is pulled LOW at the time of sampling. If the bit sent by the slave is ‘1,’ the line is pulled HIGH at the time of sampling.

 ![img](../../media/hardware/peripheral-interfaces/GPIO/GPIO_one_wire_read.png)

The master can communicate with up to 100 slaves on a 1-wire standard bus. However, the greater the number of 1-wire slaves connected to the bus, the more time the master needs to pull data from them. The software libraries typically use bit-banging or the UART to time pulse durations. The LSB is always sent first in the 1-Wire protocol.


## Functionality Briefing					

Among QuecPython modules, EC200U/EC600U/EG912U/EG915U support GPIO 1-Wire Protocol. Please refer to the API introduction: [machine.OneWire](../../../api_reference/peripherals/machine.OneWire.md)


### Object Creation								

```python
class machine.OneWire(GPIOn)
```
For relationship between pin number and physical pins, please refer to [machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin).

It is available in the module, so please make sure to import it first. You can then create an one-wire like this:

```python
one_wire = machine.OneWire(2)
```

The "2" here means the protocol uses GPIO2.


### Reset & Device Detection					

```python
OneWire.reset()
```

Resets the bus and detects device. Returning 0 indicates the device has responded correctly while -1 means no device responds.

Reset is used to start communication with the slave. The master will send a reset pulse, and then wait for slave's response. See following steps:

- The master pulls down the Data Line (or 1-Wire line) for at least 480 microseconds, and then releases it.
- The master will wait for a moment so that the slave can respond to the reset pulse. Therefore, the slave should pull down the Data Line in 15~60 microseconds to send an acknowledgement signal.
- The master checks the acknowledgement signal from the slave to determine if such slave  exists. If acknowledgement signal is visible, the communication is considered a failure.

### Write										

```python
OneWire.write(buf)
```

The write operation is used to write data to the slave. The write operation transmits data bit-by -bit. The basic steps of the write operation are as follows:

- The master sends a write command to signal the slave to get ready to receive data.
- The slave receives data sent by the master bit-by-bit from the Least Significant Bit.
- The master will wait for while before transmitting every bit.

### Read										

```python
OneWire.read(len)
```

The read operation is used to read data from a slave device. In the 1-Wire protocol, read operation receives data bit-by-bit. The basic steps of read operation are as follows:

- The master sends a read command to signal the slave to get ready to send data.
- The slave sends data to dataline internally bit-by-bit from the Least Significant Bit.
- The master will wait for a while for each bit before reading level from the data line.

## Design										

This chapter will introduce `machine.OneWire` by using temperature sensor DS18B20 with QuecPython EC600U module.

Scenario：

Design a class that can read the temperature sensor DS18B20. DS18B20,  a widely used temperature sensor, is suited for multiple embedded applications. In this case,  a `DS18B20Sensor` class will be created to implement initialization, temperature reading and other functions.


### Hardware									

Materials: DS18B20 Temperature Sensor, 3 jumper wires.

<img src="../../media/hardware/peripheral-interfaces/GPIO/GPIO_DS18B20.png" alt="DS18B20" style="zoom:67%;" />



First of all, connect the hardware correctly. DS18B20 has three pins, which are VCC for power supply, DQ single line for data transmission and GND for ground respectively. 

Connect the hardware according to the following figure, the DQ is connected to GPIO15 on the EC600U module.


![EVB_DS18B20](../../media/hardware/peripheral-interfaces/GPIO/GPIO_EVB_DS18B20.png)



### Software								

API Design	

```python
class DS18B20Sensor(onewire_pin)
```

- Functionality: Create a DS18B20 object.
- Returns: DS18B20 object.
- DS18B20Sensor: DS18B20Sensor class.
- onewire_pin: GPIO number of the One-Wire bus. Please refer to [machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin).

```python
DS18B20Sensor.read_temperature()
```

- Functionality: Reads temperature.

Software Design	

- Import module
  - In QuecPython, it is valid to use `machine.OneWire` module to use the One-Wire protocol.
- Define DS18B20Sensor Class 
  - Create a DS18B20Sensor class which is embedded with methods like creating and reading temperature.

```python
import machine

class DS18B20Sensor:
    def __init__(self, onewire_pin):
        self.onewire = machine.OneWire(onewire_pin)
        # Assuming only one DS18B20 on the bus
        self.device_address = self.onewire.scan()[0]

    def reset(self):
        self.onewire.reset()

    def write(self, data):
        self.onewire.write(data)

    def read(self, len=1):
        return self.onewire.read(len)

    def read_temperature(self):
        # Skip ROM command and Convert temperature command
        write_data1 = bytearray([0xcc, 0x44])

        # Skip ROM command and Read scratchpad command
        write_data2 = bytearray([0xcc, 0x44])
        
        self.reset()
        self.write(write_data1)

        self.reset()
        self.write(write_data2)

        data = []
        for _ in range(9):
            data.append(self.read())

        raw_temp = (data[1] << 8) | data[0]
        temperature = (raw_temp / 16) if (raw_temp & 0x8000 ==
                                          0) else ((raw_temp - 65536) / 16)
        return temperature


if __name__ == "__main__":
    ds18b20 = DS18B20Sensor(15)
    print("temperature = ", ds18b20.read_temperature())
```

>This class can be expanded to support multiple DS18B20 sensors with a slight modifications to allow the initialization function to recognize multiple device addresses and select the specific sensor to be read when the temperature is requested.

This class provides basic functionality, but more error handling and parameter configuration may be needed in real applications. In addition, checksum and calibration can be supplemented to ensure accuracy.
