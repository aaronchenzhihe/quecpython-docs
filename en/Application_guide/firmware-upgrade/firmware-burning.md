# Firmware Download

## Overview

If you want to run Python scripts on Quectel modules, the module firmware must support the Python script runtime environment. Therefore, you must download the firmware that supports the Python script runtime environment released by the QuecPython team into the module.

## Downloading Mode

Before downloading the firmware into the module, you need to set a trigger condition for the module to execute a special program. This program is used to receive firmware data sent by the host and write the firmware data into the module's flash. This process is called entering the downloading mode.

Different series of modules ( [click here to view The Correspondence Between Platforms and Modules](../background/hardware-platform.md#The-Correspondence-Between-Platforms-and-Modules) ) have different methods to enter the downloading mode. In general, there are three methods:

### Ground the module's boot pin

Before downloading, ground the module's boot pin and keep it grounded. Then reboot the module until the module enumerates the USB port for downloading. At this time, you can release the grounding level of the boot pin. The module will enter the downloading mode.

> This method is applicable to all modules of ASR platform, Unisoc platform, and Eigencomm platform.

### Send AT+QDOWNLOAD

When the module is running normally, send **`AT+QDOWNLOAD=1`** to the module. The module will write a flag to the relevant register and immediately reboot. After the module reboots and detects the flag, the module will enter the downloading mode.

> This method is applicable to all modules of ASR platform and Unisoc platform

### Download through the serial port

When the module boots, it will perform a short downloading command detection through the serial port. If it detects that the host sends a downloading command to the module through the serial port, the module will enter the downloading mode.

> This method is applicable to Wi-Fi modules and can only be downloaded through specific serial ports.
>
> - The firmware of both FC41D and FCM360W are downloaded through serial port 0.

## Downloading Principle

### ASR Series Modules

After entering the downloading mode, the module waits for the handshake for the download protocol with the downloading tool on the host. After the handshake is completed, the module receives a program called "flasher" from the downloading tool, which is specifically used to download the firmware. The module copies the flasher to the RAM and runs it to receive the firmware data and write the data into the flash storage medium.

First, the flasher writes the partition table to a fixed position, and then reads the partition information from the partition table and writes the code image, NV file system image, user file system image, and other necessary image data into the flash partition at the specific address of the module.

### Unisoc Series Modules

After entering the downloading mode, the module waits for the handshake for the download protocol with the downloading tool on the host. After the handshake is completed, the module receives a program called "FDL" from the downloading tool, which is specifically used to download the firmware. The module copies the FDL to the RAM and runs it to receive the firmware data and write the data into the flash storage medium.

Based on the flash partition information it carries, FDL writes the code image, NV file system image, user file system image, and other necessary image data into the flash partition at the specific address of the module.

### Eigencomm Series Modules

After entering the downloading mode, the module waits for the handshake for the download protocol with the downloading tool on the host. After the handshake is completed, the module obtains the image and corresponding partition information from the downloading tool and writes them into the corresponding flash partitions one by one.

## Firmware Download Tools

### QPYcom

[*QPYcom*](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/QPYcom_V4.1.0.zip) is a QuecPython full-stack development and debugging tool for QuecPython. It supports basic functions such as REPL interaction, file transfer, and firmware combination and download. It also includes extended functions such as DFOTA package making and partition adjustment.

*QPYcom* encapsulates the command-line downloading functionality provided by the baseband processor manufacturers of various series of modules into a unified graphical interface. After you load the firmware in the interface, *QPYcom* can automatically identify the module series and call the corresponding manufacturer's downloading tool in the background for firmware downloading.

> [Click here](../dev-tools/QPYcom/README.md) to view the detailed *QPYcom* user guide.

### Qflash

[*Qflash*](https://developer.quectel.com/wp-content/uploads/2024/09/QFlash_V7.4_CN.zip) is a tool developed by Quectel specifically for firmware downloading. Its implementation principle is similar to the firmware downloading function of *QPYcom*.

However, *Qflash* cannot debug QuecPython development as *QPYcom* does. You are recommended to use *QPYcom* as the downloading and debugging tool.

### QMulti_DL

[*QMulti_DL*](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/QMulti_DL_V3.5_EN.zip) is a firmware downloading tool developed by Quectel for multiple devices downloading firmware simultaneously, which is suitable for batch firmware downloading of terminal products on the production line.

### Baseband Processor Manufacturer Tools

The baseband processor manufacturers also provide downloading tools. These tools can not only download firmware but also have functions such as modifying/erasing system parameters and RF data calibration. Although these tools provide developers with a high degree of development freedom, they are not suitable for those who are not so familiar with the module bottom mechanism, because improper use of the tools can cause abnormal operation of the module and even require factory repair to restore normal operation.

These tools are not open to ordinary developers and are only used by Quectel's internal R&D personnel for development and debugging.

## Downloading Process

The firmware downloading process of *QPYcom* is shown in the following figure. [Click here](../dev-tools/QPYcom/qpycom-dw.md) for detailed instructions.

![](../media/firmware-upgrade/firmware-burning/firmware-burning.gif)

The correspondence between the module and its firmware package suffix is as follows:

| Module                              | Firmware Package Suffix |
| ----------------------------------- | ----------------------- |
| EG912N-EN | .bin/.zip               |
| EC200U-EU/EG912U-GL | .pac                    |
| EC200A-EU/EC200A-AU                 | .blf                    |
| BG95                                | .mbn                    |
| FCM360W                             | .bin                    |
| FC41D                               | .bin                    |

## Frequently Asked Questions

1. Why does "**Failed to download firmware. Please check the module and firmware package**" pop up?

   Most likely because of poor contact of the USB port.

2. Why does "**Module not detected. Please connect the module**" pop up?

   Because the module is not booted, failed to enter the downloading mode, or the driver is not installed correctly.

3. Why does "**Check whether the firmware zip package is decompressed. Please decompress it and try again**" pop up?

   Because you select an incorrect firmware package for the EG912N-EN module. Please decompress the zip package and find the correct firmware package.

5. Which tool is suitable for mass production, and how can a computer download firmware for multiple modules simultaneously?

   [QMulti_DL](https://developer.quectel.com/en/wp-content/uploads/sites/2/2024/11/QMulti_DL_V3.5_EN.zip). Please refer to the user guide included in the tool package for instructions on how to use it.