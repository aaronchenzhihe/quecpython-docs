# FAQ on BSP Development

###  **What is the load capacity of GPIO?**

GPIO output current capability: 4mA

###  **What interrupt modes are supported?**

*   Rising edge interrupt mode

*   Falling edge interrupt mode

*   Rising and falling edge interrupt modes

*   High and low level interrupt mode

###   **Which pins can be used as GPIO?**

For specific pins that can be used as GPIO, please refer to the Section machine.Pin on the wiki.

> Specific link: [GPIO pin description](../../api_reference/peripherals/machine.Pin.md)

###  **Which pins can be used as external interrupts?**

The external interrupt number will go along with the number of GPIO pins.  For details, please refer to machine.Pin and machine.Extlnt on the GPIO pin wiki.

> Specific link: [GPIO pin description](../../api_reference/peripherals/machine.Pin.md);
>
> [Extlnt-External Interrupt Description](../../api_reference/peripherals/machine.ExtInt.md)

###  **Which pins can be used as wake-up interrupts?**

*   Unisoc platform: PSM pin wakes up PSM_EXT_INT (pull high)&#x20;

*   Eigencomm platform: AGPIOWU pin wake-up

### **How does the pin level state change during this period from the hardware is powered on to the script can control the GPIO state? **

Power-on initialization state: When the hardware is powered on, the GPIO pin is usually in an undefined state, which means that the level of the pin may be high level (VCC), low level (GND) or floating state.

Before script control: Before executing script,  the level state of the GPIO pin may remain unchanged, or may change based on hardware or other external factors.

Script control stage: Once the script starts to set the GPIO, the level status of the pin will be changed according to the relevant codes and instructions in the script. Moreover, according to the specific script logic and requirements, it is available to change the pin level via controlling the input and output direction of the GPIO and writing high or low level. 

### **How to query the quantity of BSP peripheral resources in module？**

Check the Chapter [machine](../../../API_reference/en/peripherals/machine.md) in wiki under the official website documentation center, including pins. This chapter includes an introduction to peripheral resources like interrupt serial ports.

### **What is the maximum driving pixel supported by current LCD peripherals?**

Screen pixels must not exceed 500*500.

### **Is the DAC supported?**

The DAC function is currently not supported.

### **What is the voltage range of the ADC?**

EC600U: 0~VBAT;

EC600N: 0~1.3 V;

EC600S: 0~1.3 V;

Other models can be confirmed by querying the corresponding hardware design manual of the module.

### **What is the communication speed of I2C?**

The communication mode supports standard mode and fast mode. 

 Standard mode: 100KHz 

Fast mode: 400KHz

### **Can the USB serial port carry out data exchange?**

UART3 (USB_CDC port) can be used for data interaction, which is the interaction port of the module.

### **How many bytes of data can the serial port receive at most at one time?**

According to the actual measurement, 2000 bytes can be received normally, and the maximum is 512. If it exceeds, it will be automatically sent by package.

### **Is there any documentation on the default levels of the module’s pins?**

Users can refer to the *QuecPython_GPIO configuration* document of the corresponding model to learn about the default levels of each GPIO pin.

In addition, when the module is powered on, the levels of some pins may change. It is recommended that users measure the level changes of the pins before designing their own hardware to avoid losses caused by selecting GPIO in an improper way.

### **Can the RTC time of the EC600S module be maintained when the power is off?**

There is no way to continue the battery life duration when the power is off, because the EC600S does not have an RTC backup battery pin.
