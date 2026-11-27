# FAQ on Hardware Design and Verification 


### **Notes in hardware design**


- Since QuecPython currently ( Till 2023/04/11) does not have a dedicated hardware design document, you can only refer to QuecOpen's hardware design document. Owing to the certain differences between QuecPython and QuecOpen, it is unavailable to refer to document of QuecOpen directly when taking about the pins that can be taken as GPIO. Instead, it is necessary to refer to the pins introduced in the API document -[GPIO](../../api_reference/peripherals/machine.Pin.md) of QuecPython. For Other functions such as UART, which are similar.
- Same as above. Due to differences, the reset level status of module pins may be different from what is described in the document. For status lights, relays and others that may be more concerned about the default status of the level, it is recommended to measure the level of module pin that downloaded QuecPython firmware via oscilloscope or multimeter for further confirmation. 
- It is a must to lead USB_BOOT pin in stage of HW design, otherwise, it will be unavailable to download firmware and debug code. 
- It is recommended to lead out the USB_BOOT pin when designing hardware. Some modules, such as EC600E/EC800E, are mandatory. Otherwise, it will be inconvenient to download the firmware.
- It is recommended to lead the test point of  debug serial port pins in module to capture the underlying APP logs if issue occurs.
- Most of the digital pin voltage domain of the module will range 0-1.8V, which cannot be exceeded, otherwise it may cause irreversible damage. When communicating with devices in different voltage domains, please covert the level via corresponding level conversion circuit .
- For more information, please refer to the Hardware Design Manual and Reference Design Manual.

### **What are the power requirements for hardware design?**

Generally, it is required to have a power supply with current at least 2A and above. Please refer to the hardware design documents of each module for details.

### **What issues generally need to be paid attention to in the hardware design of UART function?**


- Pay attention to the voltage domain. If it does not match, please perform level conversion.
- When the UART baud rate is high, please pay attention to whether the level conversion rate of the hardware level conversion circuit can meet the requirements. It is recommended to convert via a level conversion chip.
- If you need to use the low-power consumption, the module will enter sleep mode. At this time, only the MAIN UART can wake up the module and a few bytes of data will be lost for wake-up. It is recommended to design an external interrupt during hardware design. The pin is used to wake up the module from sleep first, and then send data to the module to ensure that the module can receive complete data.

### **Why is it a failure to display USB port on the computer after powering on the PCB designed by us?**

It is recommended to check according to following steps:

- If you have tested using the EVB, please plug in the same USB cable into EVB and compare it with the computer port to see if it can be identified. Thus, the issue on computer USB driver, computer USB interface and USB cable can be eliminated.
- For USB to be recognized normally by the computer, you need to confirm that the module has been powered on normally. Generally, the VDD_EXT pin will output 1.8V after normal startup. what's more, it is available to send AT commands via MAIN_UART and return normally. If the module is not powered on, please finish it beforehand.
- If it is turned on, the EVB can be recognized normally instead of the self-made PCB. In this situation, it is needed to check whether your USB circuit design meets the hardware design requirements, such as impedance matching requirements, tracing requirements, and whether the junction capacitance of the TVS is unduly huge. wait. A more direct way to solve this problem is to directly connect the four USB communication lines to the module USB pins and remove the original designed circuit.
- If it still cannot be identified after the above investigation, please consider whether the module is damaged and replace it with another PCB for comparison testing.

### **How to design the hardware so that the module can automatically start up when it is powered on?**


Most modules can directly connect the POWERKEY pin to ground through a pull-down resistor, but some models are not allowed to be grounded all the time, such as the BG95 series modules. For details, please refer to the hardware design manual. In addition to the BG95 series, there are some application scenarios that cannot be grounded all the time. For example, if you need to use a scheduled power on and off to reduce power consumption, if it is grounded all the time, it will automatically turn on after software shutdown. At this time, you will need to use a similar circuit of the BG95 series to boot and pull up POWERKEY pin. Finally, it will no longer affect the software shutdown function.