# Quick Start

## Hardware preparation

- A Windows computer, recommended for the 'Win10' system
- A set of [BG95M3 QuecPython standard development board ](../../../dev_board_guide/bg95-evb.md)
- A functional Nano SIM card

## Environment construction

- Download and install BG95M3 series module driver: [QuecPython_USB_Driver_Win10_U_G]( https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverQ_NDIS_V2.7.14.zip)
- Download and install [VSCode](https://code.visualstudio.com/ )
- Download and unzip [QPYcom](https://developer.quectel.com/wp-content/uploads/2024/09/QPYcom_V4.1.0.zip) Tool to the appropriate location on the computer
- Use QPYcom to burn [firmware package](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_V0003_BG95M3_FW.zip )
- Download [Experimental Source Code](https://github.com/QuecPython/tracker-box)

## Set up your smart tracker

Power on the smart tracker by following these steps:

1. Before turning on the device, insert the nano SIM card (data service must be activated) into the SIM card slot. If the SIM card is inserted after the device is turned on, the device needs to be restarted.

2. Turn on the device by any of the following methods:

   Method 1: Press the power button on the side of the device.

   <img src="../../media/solutions/tracker_box(BG95M3)/power_button.PNG" style="zoom: 20%;" />

   Method 2: Connect the smart tracker to a laptop or power bank using the included USB-C cable, and the device will automatically turn on.

   <img src="../../media/solutions/tracker_box(BG95M3)/usb-c_sim.jpg" style="zoom: 20%;" />

## Connect to the Asset Management SaaS Platform

The asset management SaaS platform developed by Acceleronix provides comprehensive solutions for specific industries. It covers the entire business process from TSL model function definition to SaaS platform management and mobile application device control.

This asset management SaaS platform supports users in operating devices and can dynamically analyze hardware data based on the TSL model. It integrates multiple sensors such as water quality, temperature, humidity, light, and carbon dioxide to adapt to different scenarios. Functions include real-time data viewing, historical operation curves, data aggregation, as well as device maps and trajectory playback for location-based devices.

### Device development

### Power on

After completing the hardware connection work, when PWR and SCK1 are lit up or a COM port containing the words "Quectel USB" appears in the port list of the computer's Device Manager, it indicates that the device has been successfully powered on.

<img src="../../media/solutions/tracker_box(BG95M3)/USB.png" style="zoom: 100%;" />

### Burn firmware package

Refer to [this chapter](../../../Getting_started/flash_firmware.md) to flash the [firmware package](https://developer.quectel.com/wp-content/uploads/2024/09/QPY_OCPU_V0003_BG95M3_FW.zip ) to the development board.

## Script import and run

1. Refer to [this chapter](../../../Getting_started/first_python.md#Writing the First Script File)  to import all files in the code folder under the source code directory into the module file system according to the original directory structure, as shown in the figure below.

<img src="../../media/solutions/tracker_box(BG95M3)/Qpycom.png" style="zoom: 100%;" />

2. Refer to [this chapter](../../../Getting_started/first_python.md#Executing the Script File) Execute the script file and execute the main program file `main.py`. (During normal startup, the module will automatically execute the `main.py` file)

3. Refer to [this chapter](../../../Getting_started/first_python.md#Stop-a-Running-Program), stop program running.

### Business debugging

### Program launch

 After executing the _main.py script, the program starts running and prints dialing information, including dialing status, IP Address, DNS server address, device number, etc.

<img src="../../media/solutions/tracker_box(BG95M3)/860ce14a-e915-4207-9f29-36ac5b1a7a64.png" style="zoom: 100%;" />

### Data detection

After starting to run, startup information will be printed.

<img src="../../media/solutions/tracker_box(BG95M3)/60eb6249-67ea-40d3-a59b-0612dbc9f308.png" style="zoom: 100%;" />

### View device location (LBS)

1. Click "Details" in the device list to open the device details page.
2. Go to the Location tab to view LBS location data.

<img src="../../media/solutions/tracker_box(BG95M3)/saas_location.png" style="zoom: 100%;" />

**Note**: The location is updated every 30 minutes. If you need to manually refresh it, please click the refresh button in the application control panel, then reload the page to view the updated location.

### View and manage sensor data

1. When the device is online, click on the device in the Wonderfree app to enter the control panel. 
2. View the real-time sensor data and LBS location data of the smart tracker. 
3. If you need to update the sensor data or LBS location, click the refresh button in the upper right corner of the control panel.

<img src="../../media/solutions/tracker_box(BG95M3)/wonderfree4.png" style="zoom: 25%;" />

<img src="../../media/solutions/tracker_box(BG95M3)/wonderfree5.png" style="zoom: 25%;" />