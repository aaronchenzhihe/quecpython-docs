# Log Tool User Guide

This document introduces the usage of log capture tools for different QuecPython platforms, including ASR and UNISOC8910.

## Purpose of Module Log

During the use of the module, you may encounter various issues. In order to identify the cause of the problem and the running status of the module code, it is usually necessary to capture the running logs of the module. These logs can help you analyze and locate problems, as well as determine whether the operation of certain functions meets expectations.

## Module Log Types

Module logs are generally classified into AP logs, CP logs, pcap logs, and dump files.

* AP logs: Logs output by the SDK and Apps. AP logs are mainly used to confirm the running status of the system software and can be used to locate software bugs.

* CP logs: Logs related to the wireless protocol stack output by the Modem, such as air interface logs. CP logs are mainly used to locate the bugs related to wireless networks, such as module network registration problems and abnormal network disconnections. To locate the device's network registration problem, you need to capture the CP logs during the device's boot stage or the CP logs containing the CFUN0/1 switching process.

  > CFUN0/1 switching refers to calling `net.setModemFun(0)` to switch the device to mode 0 (minimum functionality mode), and then calling `net.setModemFun(1)` to switch the device to mode 1 (full functionality mode). The purpose is to make the device initiate the network registration process again. If you did not capture the CP logs during the device's boot stage, you can also capture the CP logs containing the CFUN0/1 switching process and provide them to Quectel software engineers for analysis.

* pcap logs: pcap logs can be used to analyze and debug network traffic. Through pcap logs, you can know the working status of the device or application at the network level to identify performance issues or faults. By analyzing pcap logs, you can view the details of network communication, such as the process of establishing connections, the format of transmitted packets, and the parameters of requests and responses. For example, when analyzing TCP/IP, MQTT, HTTP/HTTPS issues, pcap logs are often used. You can use *Wireshark* to open pcap files.
* dump files: When some unrecoverable errors have occurred in the operating system or application, it will save certain parts of the memory as files to record the status and other relevant information at that time. These files are called dump files. For example, program crashes, illegal memory access, memory exhaustion, stack overflow, and other abnormal issues may cause the module program to crash and reboot abnormally. If you encounter situations where the module reboots abnormally, it is likely that a dump problem has occurred. At this time, it is necessary to capture and export the dump file and provide it to Quectel software engineers for problem localization and analysis.

## Log Tools for Each Platform

Below are the tools used to capture AP/CP logs, pcap logs, and dump files for each platform.

| Platform   | Module Model              | AP Log                                                       | CP Log                                                       | pcap Log                                         | Dump File                                        | Icon                                                         |
| ---------- | ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| ASR        | EG915N                    | [CATStudio](./ASR.md#Using CATStudio)<br>[QWinLog](./ASR.md#Using QWinLog) | [CATStudio](./ASR.md#Using CATStudio)<br/>[QWinLog](./ASR.md#Using QWinLog ) | [CATStudio](./ASR.md#Using CATStudio)            | [CATStudio](./ASR.md#Using CATStudio)            | ![](../../media/dev-tools/LogTool/CatStudio.png)![](../../media/dev-tools/LogTool/QWinLog.png) |
| UNISOC8910 | EC200U/EG912U/<br/>EG915U | [coolwatcher](./UNISOC8910.md#Capture AP Logs)               | [ArmTracer](./UNISOC8910.md#Capture CP Logs)                 | [coolwatcher](./UNISOC8910.md#Export Pcap Files) | [coolwatcher](./UNISOC8910.md#Export Dump Files) | ![](../../media/dev-tools/LogTool/cooltools.png) ![](../../media/dev-tools/LogTool/ArmTracer.png) |

## Table of Contents

[ASR Platform Log Tool User Guide](./ASR.md)

[UNISOC8910 Platform Log Tool User Guide](./UNISOC8910.md)