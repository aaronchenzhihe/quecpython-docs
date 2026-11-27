# EC800X_DP-DTU-Q600R Series Development Board Introduction

## Quick Start

Supported module list

- EC800K-CN
- [EC800M-CN](https://www.quectel.com.cn/product/ec800m-cn)

## Feature List

### Basic Overview

The EC800X_DP-DTU-Q600R series development board is a DTU development board using the LTE CAT1 network. Its physical dimensions are 98 mm x 46 mm x 26 mm, making it easy to integrate into industrial PCs, industrial equipment, and sensing devices.

- The development board features wide network coverage and low transmission latency, and supports access to the three major Chinese operators;
- It has three indicator LEDs to help customers confirm device status;
- It supports FOTA remote upgrade, improving product reliability;
- It supports DC 5 - 28 V wide-voltage power supply;
- It provides a standard RS485 interface;
- It comes with a hardware watchdog;
- It suits various industrial scenarios and is easy to install;
- The development board also supports cloud access such as TCP, MQTT, Alibaba Cloud, and Telecom Cloud, with future support for Huawei Cloud, Tencent Cloud, OneNET, and others.

## Development Board Renderings

![](./media/ec800x-dp-dtu-q600r/渲染图.png)

## Document Downloads

[DP-DTU-Q600R Series Development Board Product Specification and User Guide](https://developer.quectel.com/wp-content/uploads/2024/11/EC800X_DP-DTU-Q600R系列开发板产品规格及用户指导.pdf)

[DP-DTU-Q600R Series Development Board Schematic](https://developer.quectel.com/wp-content/uploads/2024/11/EC800X_DP-DTU-Q600R系列开发板原理图.pdf)

[DP-DTU-Q600R Series Development Board Silkscreen](https://developer.quectel.com/wp-content/uploads/2024/11/EC800X_DP-DTU-Q600R系列开发板丝印.pdf)

## Development Board Resources

The main interface layout of the development board is shown in the figure below.

![](./media/ec800x-dp-dtu-q600r/dp-dtu-q600r.png)

- VCC: Power input, DC 5 - 28 V.
- GND: Ground.
- 485A/RXD: Supports RS485 and TTL. TTL: 3.3 V.
- 485B/TXD: Supports RS485 and TTL. TTL: 3.3 V.
- POW: Red LED, the power indicator; it lights up once power is applied.
- NET: Red LED.
  - Steady on: Working normally;
  - Off: Faulty;
  - Fast blinking: Good signal;
  - Slow blinking: Poor signal.
- USER: Red LED.
  - Steady on: Connected to the server;
  - Off: Not connected.
- SIM card slot: The NANO SIM card slot supports USIM compliant with ETSI and IMT-2000 specifications, and supports 1.8 V and 3.0 V USIM cards.
- USB port.
- Antenna connector.

## Quick Start Guide

### Hardware Preparation

The following are the hardware preparation steps after you receive the development board.

- <font color='red'>**Step 1: Hardware Preparation**</font>

    The following accessories need to be prepared before starting.

    1. **Development board**
    2. **USB cable** (USB-A to USB-Type-C)
    3. **PC** (Windows 10)
    4. **SIM card**
    5. **4G antenna**

- <font color='red'>**Step 2: Antenna & SIM Card Installation**</font>

    Install the antenna supplied with the development board at the LTE antenna connector, and insert the SIM card into the SIM card slot on the development board.

- <font color='red'>**Step 3: Development Board Connection**</font>

    Connect the Type-C port of the development board to the USB port of the PC with a USB Type-C cable to power the board.

- <font color='red'>**Step 4: Power On the Development Board**</font>

    Set the power selection switch for USB and DC on the development board to the USB position.

    Long-press PWRKEY to power on. Make sure the supply voltage is stable before pressing the PWRKEY button. It is recommended that the interval from power-up to pressing PWRKEY be no less than 30 ms. After power-on, the NET LED on the development board blinks, indicating that the board has started normally.

### Driver Installation

- Open the [QuecPython official driver download link](https://www.quectel.com.cn/download-zone).

- Select the driver that matches the module model. The module used on this development board is EC800M. Taking EC800M as an example, search for the USB driver for this model on the download page.

  ![](media/readme/driver_eg912u.png)

- [QuecPython_USB_Driver_Win10_ASR](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_Windows_USB_DriverA_Customer_V1.1.13.zip) — click download. If the development board carries another model, follow the same steps.

- After downloading, extract the driver package and find **"setup.exe"** or **"setup.bat"**. Double-click to run it. After installation, open Device Manager and the yellow exclamation mark on the port will disappear, indicating a successful installation and normal communication.

>  Devices such as <font color='red'>Mobile ECM Network Adapter </font> or <font color='red'>CDC Ethernet Control Model (ECM) </font> being unrecognized is normal. It does not affect firmware flashing or subsequent development, and can be ignored.

- Right-click 【**My Computer**】 — select 【**Manage**】 — select 【**Device Manager**】, then select 【**Ports**】 in Device Manager. After opening the page as shown, if a serial port starting with **Quectel USB** is refreshed, the USB driver has been installed successfully.


### Tool Installation

- Obtaining QPYcom

Developing with QuecPython requires a dedicated development and debugging tool — **QPYcom** — which supports debugging code, analyzing logs, file transfer, flashing firmware, merging firmware, and more.

Open the [QuecPython tool download page](https://www.quectel.com.cn/download-zone) and download the tool for your operating system.

*Note: This tool requires no installation; it runs after extraction.*

- VSCode Extension

Editing Python code usually involves an IDE dedicated to Python or compatible with multiple languages, which can effectively improve development efficiency. VSCode is recommended here.

For VSCode, QuecPython provides a dedicated extension that implements code hints, code completion, and serial port debugging. See the figure below for installation.

In VSCode, click the extension marketplace in the sidebar, search for "QuecPython" in the marketplace, and download the extension from the search results.

![](./media/readme/VScode_1.png)

### Firmware Flashing

**Obtaining Firmware**

Modules are usually shipped with standard AT firmware or QuecOpen firmware. To develop the module based on QuecPython, you need to manually flash dedicated QuecPython firmware.

[Official firmware download address](https://www.quectel.com.cn/download-zone)

![](media/readme/Firmware.png)

Given the wide variety of firmware on the official website, to choose the right firmware package you first need to know the module model, which can be obtained from the module's laser marking or by sending AT commands.

Once you have the module model, download the corresponding firmware for that module from the official website. Firmware names are generally in the form `QPY_OCPU_version_module_model_FW`. Unless there is a specific requirement, download the latest version by default.

Click the download button in the download column to download. After a successful download you will get a zip package named after the firmware.

The firmware package downloaded from the official website is in compressed format. After downloading it locally, extract it. You will get two files: the `.bin` file is the QuecPython firmware itself, and the `.md` file is the changelog.

![](media/readme/firmware_zip.png)

> Be sure to extract the package contents to a path that does **not contain Chinese characters, spaces, or other special characters**; otherwise the download tool may fail to recognize the firmware. The download tool path must also **not contain Chinese characters, spaces, or other special characters**.

**Firmware Flashing**

- <font color='red'>**Step 1: Create a Project**</font>

First make sure the module is properly connected and powered on. Open QPYcom and go to the download page. Click **"Create"** to create a new firmware download project. The project name is user-defined; save it persistently after configuration.

- <font color='red'>**Step 2: Select Firmware**</font>

Select the firmware to download to the module (choose the firmware corresponding to the module model to be flashed; for new-format firmware, select the .bin file for all module models).

- <font color='red'>**Step 3: Set the Download Mode**</font>

Click the drop-down arrow to the right of **"Download script"** and select **"Download FW"**.

- <font color='red'>**Step 4: Start Flashing Firmware**</font>

After clicking **"Download FW"**, firmware downloading begins. A progress bar and percentage are displayed during the process, and a pop-up indicates success when finished.

![](./media/readme/firmware.gif)

### REPL Debugging

REPL stands for <font color='red'>**Read-Eval-Print-Loop (interactive interpreter)**</font>. QuecPython programs can be debugged in the REPL.

Run the **QPYcom** tool, select the correct serial port (baud rate need not be specified), and open it to start Python command-line interaction.

- <font color='red'>**Step 1: Enter the Interactive Page**</font>

Open QPYcom, select **Quectel USB REPL Port** as the port, and choose the "Interactive" view.

- <font color='red'>**Step 2: Open the Serial Port**</font>

Click "Open Serial Port", enter **print('hello world')** in the interactive page, and press Enter to see the execution result.

```python
>>> print('hello world')
hello world
```

![](./media/readme/hello_world.png)

<font color='red'>*Note: Input in the tool's interactive page must use English characters; Chinese characters will be blocked.*</font>

## Developing Your First Script

### Writing the First Script File

Create a *helloworld.py* file that outputs "hello world". The script is as follows:

```python
print("hello world")
```

Download the file you just edited to the module via QPYcom and run it.

![](./media/readme/QPYcom_hello.jpg)

### File Transfer Between PC and Module

**Download method 1:**

<img src="./media/readme/QPYcom_drag.jpg" alt="image-2021081301" style="zoom:150%;" />

- <font color='red'>**Step 1: Open the Serial Port**</font>

First select the module's interactive port and click the "Open Serial Port" button.

- <font color='red'>**Step 2: Download via Tool Buttons**</font>

Use the "**+**" and "**-**" buttons at the top right of the file page to upload and delete files.

- <font color='red'>**Step 3: Download by Drag and Drop**</font>

You can also drag local files shown on the left side of the file page directly to the module on the right (folders can also be dragged).

- <font color='red'>**Step 4: Download Progress and Result**</font>

During download, the file name and download progress are shown in the status bar.

**Download method 2:**

<img src="./media/readme/QPYcom_sc_1.jpg" alt="image-2021081301" style="zoom:150%;" />

- <font color='red'>**Step 1: Create a Project**</font>

Create a user project as needed (click the "Create" button). The steps are the same as firmware flashing above.

- <font color='red'>**Step 2: Configure the Files to Download**</font>

Select the user scripts to download to the module (add them in the "User Scripts" area via the right-click menu).

- <font color='red'>**Step 3: Set the Download Mode**</font>

Click the drop-down arrow and select "Download Script".

- <font color='red'>**Step 4: Start Downloading the Script**</font>

Click "Download Script" to start. A progress bar is shown during the download.

### Running a Script File

- <font color='red'>**Step 1: Download the Script**</font>

Download the script file to run to the module. Refer to the steps above.

- <font color='red'>**Step 2: Run the Script via REPL**</font>

After opening the serial port, enter the following code in the QPYcom interactive page to execute the script file.

```python
import example
example.exec("/usr/helloworld.py") # filePath is the path of the script file to execute
```

- <font color='red'>**Step 3: Run the Script via the GUI Tool**</font>

Alternatively, run the script file with the **Run** button on the QPYcom file page: select the script file to run and click the "**▷**" button.

The execution result is shown below.

![](./media/readme/QPYcom_repl_hello.png)

### Stopping a Running Program

How to stop a running program depends on the type of script file being run, as follows:

<table valign="center">
	<thead>
    <tr>
		<th>Is the program<br/>named main.py</th>
		<th>Does the program<br/>contain an infinite loop</th>
		<th>Does the program<br/>use multithreading</th>
		<th valign="middle">Stop Steps</th>
	</tr>
	 </thead>
    <tr>
		<td rowspan="4">✓</td>
		<td rowspan="2">✓</td>
		<td rowspan="1">✓</td>
		<td>(1) Press Ctrl + A to enter RAW mode<br/>(2) Press Ctrl + D to restart the QuecPython VM<br/>(3) Press Ctrl + B to return to normal interactive mode<br/>(4) If the above methods do not work, re-flash the firmware</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>(1) Press Ctrl + C to interrupt the program<br/>(2) If the above methods do not work, re-flash the firmware</td>
	</tr>
	<tr>
		<td rowspan="2">✗</td>
		<td rowspan="1">✓</td>
		<td>(1) Press Ctrl + A to enter RAW mode<br/>(2) Press Ctrl + D to restart the QuecPython VM<br/>(3) Press Ctrl + B to return to normal interactive mode<br/>(4) If the above methods do not work, wait patiently for the program to finish</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>(1) Press Ctrl + C to interrupt the program<br/>(2) If the above methods do not work, re-flash the firmware</td>
	</tr>
	<tr>
		<td rowspan="4">✗</td>
		<td rowspan="2">✓</td>
		<td rowspan="1">✓</td>
		<td>(1) Press Ctrl + D to restart the QuecPython VM<br/>(2) If the above methods do not work, restart the module directly</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>(1) Press Ctrl + D to restart the QuecPython VM<br/>(2) If the above methods do not work, restart the module directly</td>
	</tr>
	<tr>
		<td rowspan="2">✗</td>
		<td rowspan="1">✓</td>
		<td>(1) Press Ctrl + C to interrupt the program<br/>(2) If the above methods do not work, re-flash the firmware or restart the module directly</td>
	</tr>
	<tr>
		<td rowspan="1">✗</td>
		<td>(1) Press Ctrl + C to interrupt the program<br/>(2) If the above methods do not work, re-flash the firmware or restart the module directly</td>
	</tr>
</table>
