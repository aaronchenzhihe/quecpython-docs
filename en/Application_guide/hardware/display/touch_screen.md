
## Touch Screen

In embedded devices, a touchscreen is a common input device used to replace a mouse or other pointing devices for interacting with and controlling the device's interface. It is typically a flat surface on which users can touch and slide their fingers to perform corresponding actions.

### Types

Here are some common types of touchscreens, along with their working principles, characteristics, and applications:

1. **Resistive Touchscreen**:
   - **Working Principle**: A resistive touchscreen consists of two layers of resistive material. When touched, the two layers make electrical contact at the point of contact, causing a change in resistance. By measuring the change in resistance, the touchscreen controller can determine the touch position.
   - **Characteristics**: Low cost, high durability, and can withstand various environmental factors such as dust and water. The disadvantage is its lower touch accuracy and support for single-touch only. Transparency and color reproduction may not be as good as other types.
   - **Applications**: Commonly used in low-cost devices and industrial equipment, such as POS systems and industrial control panels.
2. **Capacitive Touchscreen**:
   - **Working Principle**: Capacitive touchscreens detect touch positions by measuring changes in capacitance on the screen. When a finger touches the screen, it causes a change in capacitance, which is detected as a touch event.
   - **Characteristics**: High touch accuracy, support for multi-touch, good color reproduction and high transparency. The disadvantage is its relatively higher cost and limited touch input with only fingers or special styluses.
   - **Applications**: Capacitive touchscreens are widely used in smartphones, tablets, and laptops.
3. **Infrared Touchscreen**:
   - **Working Principle**: Infrared touchscreens detect touch positions by measuring the interruption of infrared light caused by the touch. The screen edges are equipped with infrared emitters and receivers, and when a touch event occurs, it blocks some of the infrared light.
   - **Characteristics**: Supports multi-touch and can be touched with any object. The downside is lower accuracy compared to capacitive and resistive touchscreens and susceptibility
- **Applications**: Infrared touchscreens are typically used in large display devices, such as interactive TVs, electronic whiteboards, and more..

4. **Surface Acoustic Wave (SAW) Touchscreen**:

- **Working Principle**: SAW touchscreens use ultrasonic waves to detect touch positions. When you touch the screen, it absorbs or disrupts some of the ultrasonic waves, and by measuring this change, the touch location can be determined.
- **Features**: It has very high touch accuracy and excellent image quality, but it is sensitive to environmental changes, such as temperature, humidity, and liquid splashes, which can affect its performance.
- **Applications**: Commonly found in gaming devices and devices requiring high precision and high-quality images.

Each type of touchscreen has its advantages and suitable applications, and the choice of which type to use depends on specific application requirements.

### Driver Block Diagram

The relationship between the MCU/CPU, driver IC, and interface is as follows:

![1688127951394](../../media/hardware/display/touch_controller.png)

1. **Touch Controller**: Also known as the main controller, it is the core component of the touchscreen system. It is responsible for receiving signals from the touchscreen sensor and converting these signals into understandable coordinate information. The touch controller can be a separate IC or a module integrated into a microprocessor or microcontroller.

2. **Touch Sensor**: This is the physical part of the touchscreen system, which can be a film or a glass panel. When we touch this sensor, it converts our touch actions into electrical signals and sends them to the touch controller.

3. **Interface**: This is the way the touch controller communicates with other components, such as microprocessors or microcontrollers. Common interface types include I2C, SPI, and USB etc.

4. **Microprocessor or Microcontroller**: It processes the data from the touch controller and translates this data into useful commands, such as moving the cursor or opening applications.

Therefore, the operation of this system can be simplified into the following steps:

Touch -> Touch Sensor -> Electrical Signal -> Touch Controller -> Data -> Interface -> Microprocessor or Microcontroller -> Commands

### Compatibility Notes

For detailed interface description, please refer to [WIKI_Touch](../../../api_reference/peripherals/machine.LCD.md).

The support status on QuecPython series modules is as follows:

| Name    | Function         | Interface | Datasheet          |
| ------- | ---------------- | --------- | ------------------ |
| xpt2046 | Resistive Touch  | SPI       | [xpt2046.pdf](../../../../cn/Application_guide/media/hardware/display/XPT2046.PDF) |
| gt911   | Capacitive Touch | I2C       | [gt911.pdf](../../../../cn/Application_guide/media/hardware/display/GT911.PDF)   |
| cst816  | Capacitive Touch | I2C       | [cst816.pdf](https://datasheet4u.com/pdf/1576442/CST816D.pdf)  |



### Calibrating the Touch Screen

In practical applications, resistive touch screens need to be calibrated before use, while capacitive touch screens are usually calibrated by the control chip and do not require additional calibration steps. The calibration algorithm for resistive touch screens is already integrated in the driver. During the calibration process, the screen will light up four points, and you need to touch them one by one to calibrate. The collected data will be compared, and if the error is too large, the calibration will fail and you need to touch again.

Calling the calibration function *xtp2046.adjust()* will start the calibration process on the screen. After the calibration is completed, the parameters will be saved in a file. The next time you initialize xtp2046, the calibration data will be read to avoid repeated calibration before each use.

### Touch Screen Press

Whether it is a resistive or capacitive touch screen, the touch controller chip usually has an interrupt pin to notify touch events, and the level can be read through this pin interrupt. However, the signal provided by the touch controller is often less accurate than the judgment made by the program through the register. When creating a touch object, you need to pass in the interrupt pin number.

For resistive touch screens, the criterion for judging a press is that the pressure in the Z direction is greater than the configured threshold; for capacitive touch screens, it is judged that at least one touch point exists.

### Touch Screen Rotation

The touch screen has 8 directions, just like the display screen. It is defined in the parameters when creating the touch object (x_inv, y_inv, xy_swap). The rotation is achieved through software calculation.

The resolution setting of the touch screen is also important because the rotation calculation of the touch screen depends on the width and height resolution of the touch screen. Improper settings will result in incorrect rotation effects.

<font color='red'>**Note:**</font>

**When using a resistive touch screen, the resistance values in each direction may not be evenly distributed, which may cause inaccurate touch positions after rotation calculation. Therefore, it is recommended not to rotate the resistive touch screen after calibration.**

### Example

QuecPython has already adapted three touch drivers, which can be <font color='red'>**used directly**</font>.

Taking the touch screen (GT911) of the [Uranium EVB](touch_screen.md) as an example:

```python
from tp import gt9xx
tp_gt911 = gt9xx(irq=40,reset=20)
tp_gt911.activate()
tp_gt911.init()
tp_gt911.read_xy()
```

The effect is as follows:

![1688129750452](../../media/hardware/display/touch_resulut_show.png)

### Common Issues

#### Inaccurate Touch

Inaccurate touch may be caused by the following reasons:

1. **Calibration Issues**: If the touch screen is not calibrated correctly or there are errors during the calibration process, the touch screen may not accurately recognize the touch position. In this case, you need to calibrate the touch screen.
2. **Driver Issues**: If there are problems with the touch screen driver program, it may cause inaccurate touch positioning. In this case, you may need to update or fix the touch screen driver program.
3. **Hardware Issues**: Hardware problems can also cause inaccurate touch. For example, the touch screen sensor may be damaged, or there may be connection issues between the touch screen and the controller.
4. **Electromagnetic Interference**: In certain environments, electromagnetic interference may cause inaccurate touch positioning. This may require changing the device's operating environment or adding electromagnetic shielding measures.
5. **Software Issues**: In some cases, issues with the application may affect the accuracy of touch. For example, some applications may not handle touch input correctly, resulting in deviation in touch positioning.

#### Touch Malfunction

There can be several reasons for touch screen malfunction:

1. **Hardware Failure**: The touch screen itself or the hardware connected to it (such as the controller, data cables, etc.) may have failures, including physical damage, aging, poor contact, etc.
2. **Driver Issues**: The touch screen driver software may have failures, such as defects in the driver program, compatibility issues, incorrect installation or update of the driver, etc.
3. **Software Conflict**: In the operating system, other software or drivers may conflict with the touch screen driver.
4. **Power Issues**: The power supply to the touch screen may be unstable or insufficient, which can cause touch screen malfunction.
5. **Environmental Factors**: Extreme environmental conditions such as temperature, humidity, static electricity, etc., can also affect the normal operation of the touch screen.

#### Touch Response Delay

Several factors can cause touch response delay:

1. **Processor Overload**: If the processor of the embedded device is busy processing a large number of tasks, it may not have enough resources to process touch input quickly, resulting in touch response delay.
2. **Software Issues**: The driver program of the touch screen or operating system may have issues that cause delay in touch event processing. In addition, if the application running on the device is poorly designed or optimized, it may also cause touch response delay.
3. **Hardware Issues**: If there are problems with the touch screen or its related hardware, it may cause touch response delay. For example, the touch screen sensor or controller may have performance issues.
4. **Communication Delay**: In some embedded systems, there may be delays in data transmission between the touch screen and the processor, which can be caused by slow interface speed or poor quality of data cables, etc.
