# Initialization Process

The initialization process of a SIM card involves multiple steps. This document only lists a few main processes and provides explanations. If users want to learn more detailed information, they can refer to the corresponding sections in the 3GPP protocol documents, which mainly involve the following protocols:

* 3GPP TS 11 series

* 3GPP TS 31 series

* 3GPP TS 51 series

## Power On and Reset

The initialization process of a SIM card starts with powering on the device and activating the SIM card through a reset sequence. This reset sequence includes specific voltage levels and timing, which allow the SIM card to know that the device is ready to communicate with it.

SIM card reset can be categorized into cold reset and warm reset.

* Cold reset involves completely disconnecting the power supply and then powering on again, which is a complete reset process.

* Warm reset is a reset performed while the SIM card is still powered on, used to reinitialize the card.

### Activation and Cold Reset

The sequence of activation and cold reset is as follows:

![](../../media/network-comm/sim/SIM卡激活和冷复位时序.png)

* The RST pin is in a low-level state.

* VCC starts to supply power.

* CLK needs to provide an appropriate and stable clock.

* The ME's I/O should be in a receiving state, and during the SIM card activation period, the ME should ignore the status on the I/O. This is because the I/O status of the SIM card is indeterminate during the period from SIM card activation to cold reset.

* The ME sets RST to a high level and waits for the ATR response from the SIM card on the I/O.

  

> The timing after Ta in the above figure represents the cold reset timing.

### Warm Reset

After the SIM card is activated, without powering off the SIM card, a reset initiated by the ME to the SIM card is called a warm reset. The timing diagram is as follows:

![](../../media/network-comm/sim/SIM卡热复位时序.png)

> According to the ISO7816-4 standard, if a warm reset is initiated while the SIM card is in the process of answer to the reset (ATR), it may damage the SIM card and cause problems. Therefore, it is usually necessary to wait for the completion of answering to the reset before performing the next reset operation.

## ATR

After the reset, the SIM card returns a response called Answer To Reset (ATR), which contains some basic information about the SIM card, such as supported protocols and voltage levels.

ATR consists of up to 32 bytes (excluding the start character TS), and its structure is shown in the following figure:

![](../../media/network-comm/sim/ATR帧结构.png)

For detailed explanations of each character in the ATR response frame, please refer to the ISO 7816-4 standard document.

## PPS

PPS, short for Protocol and Parameters Selection, is a process of selecting and setting communication protocols and parameters. The main goal of PPS is to select the optimal protocol and parameters based on the capabilities of the card and the device, in order to achieve the highest communication speed and efficiency.

In the SIM card initialization process, after the device receives the ATR from the SIM card, the PPS process can begin. The ATR contains the protocols and parameters supported by the SIM card. The device will select the optimal protocol and parameters based on this information and its own capabilities.

The basic process of PPS is as follows:

* PPS Request: The device sends a PPS request to the SIM card, which includes the protocols and parameters selected by the device.

* PPS Response: After receiving the request, the SIM card returns a PPS response. The response contains the protocols and parameters accepted by the SIM card.

* Confirmation: After receiving the response, the device confirms the selection of protocols and parameters, and performs subsequent communication according to these protocols and parameters.

The structure of PPS request and response consists of an initial byte PPSS, a format byte PPS0, three optional parameter bytes PPS1, PPS2, PPS3, and a check byte PCK. The details are as follows:

![](../../media/network-comm/sim/PPS请求和响应结构.png)

## File Selection

The SIM card has a file system that stores various critical data and application information. In the initialization process, the device needs to select and read specific files to obtain necessary data. The following are the steps of the file selection process:

* Selecting MF: The device needs to select the MF (Master File) first. MF is the root of the SIM card file system, and selecting MF is a prerequisite for accessing other files. The device usually selects MF by sending a `SELECT` command and specifying the file ID of MF.

* Selecting DF: After selecting MF, the device can select one or more DFs (Dedicated Files). DF is the next level of the file system, similar to a folder, and contains a group of related EFs (Elementary Files). For example, a DF may contain all the phonebook-related EFs. Selecting DF is also done by sending a `SELECT` command, but with the file ID of DF specified.

* Selecting EF: After selecting DF, the device can select specific EFs (Elementary Files). EF is the last level of the file system and directly stores data. For example, an EF may store the user's IMSI or a phone record. Selecting EF is also done by sending a `SELECT` command, but with the file ID of EF specified.

* Reading Files: After selecting EF, the device can send `READ BINARY` or `READ RECORD` commands to read data from EF. For example, the device may read IMSI and use it for subsequent network registration and authentication.

For more information about the SIM card file system, please refer to the next section "SIM Card File System".

## Authentication

After obtaining the IMSI, the device sends a registration request to the network. The network returns a random number, which the device sends to the SIM card. The SIM card uses its internal Ki (Authentication Key) and algorithm to generate a response value, which is then returned to the device. The device sends this response value to the network. If the network verifies the response value correctly, it considers the SIM card valid and completes the registration. Otherwise, the network will reject the service.

## Initialization of Other Applications

If there are other applications on the SIM card, such as the SIM Application Toolkit (STK) application, the device will continue to initialize these applications after authentication. This may involve selecting more files, sending more commands, or performing more interactions.

## Hot Swap Initialization

After the initialization of the SIM card is completed, if the module supports the hot swap function of the SIM card, the initialization of the SIM card hot swap function will be performed.

To implement the hot swap function, the following support is generally required:

* Hardware support for changes in GPIO levels when the SIM card is inserted or removed (hardware design).

* Software needs to monitor the changes in GPIO pins connected to the hardware (monitoring methods can be interrupts or polling, interrupts are generally recommended), and handle debouncing and other processing.

* Software needs to support registering callback functions to notify the upper-layer application when a level change is detected.

* When a SIM card is detected to be inserted or removed, the software needs to handle the SIM card power-off and power-on processes.

If the Quepcython module supports the SIM card hot swap function, the software has already handled steps 2, 3, and 4, so users do not need to worry about them, only hardware support is required in the hardware design.