# SIF Communication Protocol				

International standard SIF communication protocol. Universal, convenient, single-wire, single-direction data transmission.

## Protocol Introduction					

 ![GPIO SIF Bus](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif_bus_en.png)



Characteristics about the SIF protocol:

- SIF protocol is simple and cheap to implement. Suitable for cases where the data is small and the communication quality is not strict; 
- It's a single-wire, master-slave, one-direction communication protocol. Similar but different from the 1-Wire protocol. One-direction means that a device can either be a sender or a receiver. For example, in the above figure, "BMS" is the sender, "Controller" and "Charger" is the receiver; 
- The baud rate is either predetermined as UART or detected by the receiving end based on the sync signal sent by the sender.

Transmit one frame of data at a time, each frame consists of three parts: synchronization signal (the leading signal for sending the main message) + data signal (valid data content of 8bit * 12 data bits, sent according to a certain duty cycle) + end signal (the signal marks the end of the transmission of a complete frame of data). When the transmission is over, idle state of the line must be LOW and all data should be transmitted at once with each transmission.

The voltage level of the data complies with the TTL standard.

Master (BMS) output is pulled-down. Normally there's internal pull-down resistor in the IO or you can add an external one. But the slave (receiving end) should be pulled-up by default. At 5V, the pull-up resistor should be 2.2K. And at 3.3V, the resistor should be 1K.


SIF logical level signal is defined as follows:

Tosc definition: 250us < Tosc <2ms. Recommended: 500us.

Sync signal:

![GPIO SIF sync signal](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif_sync.png)

Bit(0):

![Clock of Bit 0](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif-bit0.png)

Bit(1):

![Clock of Bit 1](????????Insert bit one image link here.????????)

A couple of points should be noted regarding signal definition:

- 32Tosc has a range of 0.5ms. It should not exceed 1ms for sake of the response time. 
- Sync signal: >992Tosc of LOW + 32Tosc of HIGH. Idle bit time > 15ms (0.5ms*992/32=15ms)
- Data bit logical 1 should meet: HIGH time > LOW time + 0.5ms
- Data bit logical 0 should meet: LOW time > HIGH time + 0.5ms
- The high and low levels will be 0.5ms and 1ms respectively, and the duty cycle is 1:2 (75% is appropriate).

Examples:

Send two bytes of data: 11000010, 11000010 (Binary)

![sif_example](../../media/hardware/peripheral-interfaces/GPIO/GPIO_sif_example.png)



## Function Description

Among QuecPython modules, EC600M/EC800M support SIF slave function.

```python
import sif
sif.init(gpio, cb)
```

For mapping between pin number and physical pins, please refer to [machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin).

It is available in multiple modules. As a result, please do import it beforehand. Then you can use the SIF slave function by calling methods below:


```python
import sif
def sif_cb(data):
    print("SIF data:",data)

sif.init(2, sif_cb)
```

The "2" here indicates the SIF protocol is applied on GPIO2.


## Design									

At present, QuecPython only supports SIF slave function.


### Hardware								

For reference only since it shall rely on specific device. 

![SIF1](../../media/hardware/peripheral-interfaces/GPIO/GPIO_SIF1.png)



### Software								

User API design

```python
class SifProtocol(gpio)
```

- Function: Create SifProtocol object.
- Return: SifProtocal object.
- SifProtocal: SifProtocal class.
- gpio: the GPIO number of 1-wire bus. Please refer to [machine.Pin](../../../api_reference/peripherals/machine.Pin.md#machine.Pin)

Software design:

- Import modules 
  - In QuecPython, you can import the `sif` module to use the SIF protocol. 
- Define SifProtocol class 
  - Create a SifProtocol class, including the creation. 

```python
import sif
import ubinascii


class SifProtocol():
    """This class is the protocol of xingheng SIF"""

    def __init__(self, gpio):
        """
        Args:
            gpio (int): SIF communication gpio port
        """
        sif.init(gpio, self.__recv_sif_data_callback)

    """
    Function:  The SIF will initialize the registered callback function. The SIF data will be parsed to byte data and passed to the callback function in low layer. In addition, the __parse_sif_data() will parse the data of different manufacturer and get the battery voltage, current, temperature, etc.
    """
    def __recv_sif_data_callback(self, data):
        print("SIF data: ", ubinascii.hexlify(data, ' '))
        self.__parse_sif_data(data)
    """
    Function: Parse the data based on the protocol. Take the common WuXi standard protocol as example, word and double words should be transmitted in little endian (Intel). Lower bytes are transmitted first.
    """    
    def __parse_sif_data(self, data):
        if data != b'':
            try:
                if len(data) == 20 and data[0] == 1:  # public message
                    check_sum = sum(data[:-1])
                    if check_sum & 0xff == data[19]:
                        print("SIF public data sum check success")
                        self.__manufacturer = data[2]
                        self.__bat_type = data[3]
                        self.__battery_cell_material = data[4]
                        self.__rated_volt = (data[5] + (data[6] << 8)) / 10
                        self.__rated_capacity = (data[7] + (data[8] << 8)) / 10
                        self.__remain_capacity = data[9] / 2
                        self.__bat_volt = (data[10] + (data[11] << 8)) / 10
                        self.__electric_current = (
                            data[12] + (data[13] << 8)) / 10 - 500
                        self.__max_temperatiure = data[14] - 40
                        self.__min_temperatiure = data[15] - 40
                        self.__mos_temperatiure = data[16] - 40
                        self.__battery_fault = data[17]
                        self.__battery_work_state = data[18]
            except Exception as e:
                print("SIF receive data fault:", e)

```

The actual physical data and the data transmitted by the bus are correlated as follows:

$$
P = \left ( C \times R \right ) +  F
$$

In the formula：
P: Actual physical data；
C: Data range transmitted by the bus;
R: Precision；
F: Offset.

Public message content


| Signal Name | Ordinal | Length | Unit | Offset | Precision | Range |
| ---------------- | ---- | ------ | ---- | ------ | ---- | -------- |
| Message ID   ID | 0    | 8 bit  | -    | 0      | 0    | -        |
| Protocol Version | 1    | 8 bit  | -    | 0      | 0    | -        |
| Battery Manufacturer Code | 2    | 8 bit  | -    | -      | -    | -        |
| Battery Model | 3    | 8 bit  | -    | 0      | 1    | 0~15     |
| Cell Material | 4    | 8 bit  | -    | 0      | 1    | 0~15     |
| Rated Voltage | 5    | 16 bit | V    | 0      | 0.1  | 0~1000   |
| Rated Capacity | 7    | 16 bit | Ah   | 0      | 0.1  | 0~500    |
| Remaining Battery SOC | 9    | 8 bit  | %    | 0      | 0.5  | 0~100    |
| Current Voltage | 10   | 16 bit | V    | 0      | 0.1  | 0~1000   |
| Current Current | 12   | 16 bit | A    | -500   | 0.1  | -500~500 |
