# Device Information

This article introduces how to use the `uos`, `usys`, `modem`, and other features to query the firmware information and remaining memory size of the Quectel QuecPython modules, which are frequently concerned by users. This article will be continuously updated to facilitate users in understanding the basic information of the module.

## Query Firmware Version Information

```python
# Query QuecPython firmware version information (QuecPython unique naming rules)
>>> import uos
>>> uos.uname()
# Return value (taking EC600U model as an example)
# ('sysname=EC600U-CNLB', 'nodename=EC600U', 'release=1.13.0', 'version=v1.12 on Sat_Nov_19_2022_5:29:48_PM', 'machine=EC600U with QUECTEL', 'qpyver=V0002')
```

As shown above, you can query the module model `machine = EC600U with QUECTEL`, which is EC600U. However, the firmware distinguishes the model as `sysname = EC600U-CNLB`, which means that this firmware can be used in the EC600UCNLB sub-model. Generally, it strictly follows the principle of one-to-one correspondence between model letters and numbers, but there are exceptions. The specific applicable module model of the firmware should be based on the firmware description in the downloading page and the official description provided by Quectel technical support. You can also query the firmware compilation date and microPython version (e.g. `version = v1.12 on Sat_Nov_19_2022_5:29:48_PM`), which is generally used to determine the newness of BETA versions (only used for testing and strictly prohibited for mass production). Since BETA versions are only used for testing, the QuecPython version is not officially released, thus the version number information will not change. Therefore, the correct version can be confirmed through the compilation time. It is recommended that users use this method to determine the version when performing version control. It is convenient to perform version control when testing with the test version. `qpyver = V0002` is the official version number of the QuecPython firmware published on the official website, which will not be elaborated here.

Why is it necessary to query the firmware version?

Beacause in the development process, you may encounter some problems and need to consult the official personnel or technical predecessors. At this time, the firmware version is essential information to provide to them. Besides, you can use the following methods to query the firmware version with the uos library:

```python
>>> import uos
>>> uos.uname2()
# Return value (Taking EC600U model as an example. Please note that this method is not supported by old firmware versions)
# (sysname='EC600U-CNLB', nodename='EC600U', release='1.13.0', version='v1.12 on Sat_Nov_19_2022_5:29:48_PM', machine='EC600U with QUECTEL', qpyver='V0002')
```

As shown above, this interface returns the same information as *uos.uname()*, but the return value is compatible with the usage of microPython, making it more convenient for users to access the information on the right side of "="  in the return value in the script. For specific usage, please refer to [uos - Basic System Services](../../api_reference/stdlib/uos.md).

In addition, we can also use the following methods to obtain the firmware version information:

```python
>>> import modem
# Query Quectel firmware version information (Quectel generic naming rules)
>>> modem.getDevFwVersion()
# Common return value types (EC600U model as an example)
# 'EC600UCNLBR03A01M08_OCPU_QPY_BETA1207'
# 'EC600UCNLBR03A02M08_OCPU_QPY'
```

As shown above, the two common return values mainly differ in whether they contain the BETA field. The firmware containing the BETA field is an unofficially released version and can only be used for testing, not for project mass production. BETA is followed by the compilation date, which 1207 represents December 7th. Normally, you should pay attention to whether the BETA field is included and to the date after BETA  in the version number .

Besides the method of querying the version number with the QuecPython scripts, you can also use  **AT+GMR** to query the version number. Here it will not be elaborated here because it is not the focus.

If you use AT commands, you only need to input one line to query the version number, while inputting two lines with scripts. How to solve this? Here is an option:

```python
>>> import modem;modem.getDevFwVersion() # Combine two lines into one, press Enter in the interactive interface to return the result
```

## Query Module Running Memory and File System Remaining Space

```python
import gc
import uos

usr = uos.statvfs("/usr")

print('Get status information of the usr directory:', usr)
print('f_bsize - File system block size in bytes:', usr[0])
print('f_bfree - Number of available blocks', usr[3])
print('Total remaining {} bytes in usr'.format(usr[0] * usr[3]))
print('Total remaining {} KB in usr'.format((usr[0] * usr[3])/1024))
print('Total remaining {} MB in usr'.format((usr[0] * usr[3]) / 1024 / 1024))

bak = uos.statvfs("/bak")

print('Get status information of the bak directory:', bak)
print('f_bsize - File system block size in bytes:', bak[0])
print('f_bfree - Number of available blocks:', bak[3])
print('Total remaining {} bytes in bak'.format(bak[0] * bak[3]))
print('Total remaining {} KB in bak'.format((bak[0] * bak[3])/1024))
print('Total remaining {} MB in bak: '.format((bak[0] * bak[3]) / 1024 / 1024))

mem = gc.mem_free()
print('Remaining available RAM space: {} KB'.format(mem / 1024))
```

As shown above, use the *uos.statvfs* function to query the status information of the *usr* and *bak* folders in the root directory, and obtain the remaining space size of the folders. Here is a brief introduction to the root directory and these two folders. Root directory: It is not allowed for users to operate, so any operation on the root directory will result in an OSerror exception. The *usr* directory: This directory allows customers to read and write files. Usually, customer code and other files are mainly stored here. For expansion, please refer to [External Storage](./memory-device.md#External-Storage). The *bak* directory: This directory is used to store key files that customers need to back up during mass production. It is readable but not writable. For storing key files, please refer to [Use of Backup Partition and Data Security Area](../../api_reference/syslib/securedata.md).

For other uos-related usage, please refer to [uos - Basic System Services](../../api_reference/stdlib/uos.md).

## Query MicroPython Virtual Machine Version
```python
>>> try:import usys as sys
... except ImportError:import sys
>>> sys.implementation
# Output
# (name='micropython', version=(1, 13, 0), mpy=10245)
```

As shown above, you can directly query the version of the MicroPython virtual machine, which is version 1.13.0. Although you can query the version of the MicroPython virtual machine, please note that there are still some differences in usage compared to MicroPython. The reason for using the try-except statement for importation is that the firmware version of the MicroPython virtual machine may be different in different periods, and some modules may have been renamed. To avoid program exit caused by exceptions, the exception handling syntax of Python is used. In the future, the module names of QuecPython will be compatible with historical versions when porting the new MicroPython virtual machine to avoid such situations and facilitate user usage.

## Querying the MicroPython Language Version

```python
>>> try:import usys as sys
... except ImportError:import sys
>>> sys.version
# Output
# '3.4.0'
>>> sys.version_info
# Output
# (3, 4, 0)
```

As shown above, two APIs are used for querying the MicroPython language version, and the main difference between the two return values is the return form. The current microPython language version is 3.4.0, which is compatible with version 3.4.0 of CPython on the computer in terms of syntax. Any changes on the MicroPython language version can be queried through this interface.

## Querying Device IMEI

```python
>>> import modem
>>> modem.getDevImei()
# Output
# '866327040830317'
```

As shown above, although it is very simple to obtain the device IMEI, it is also particularly frequently used, so let's introduce how to query  the device IMEI. So, what is IMEI?

The IMEI of an IoT module stands for International Mobile Equipment Identity. It is a unique identifier used to identify IoT modules. In IoT scenarios, IMEI can be used for the following aspects:

1. Device identification and management: Through IMEI, IoT devices can be uniquely identified and managed, including information such as the manufacturer, model, and version of the device. This information is important for device maintenance and upgrades.
2. Security and anti-theft: IMEI can be used to prevent devices from being stolen or lost. If the IMEI of a device is registered in a central database, the location and usage of the device can be tracked through that database.
3. Remote management and control: IMEI can be used for remote management and control of IoT devices. For example, if a device malfunctions or needs a firmware update, it can be diagnosed and repaired remotely with the IMEI.
4. Data statistics and analysis: IMEI can be used for statistics and analysis of IoT device usage. For example, based on the IMEI, information such as the usage time, location, frequency, and usage patterns of the device can be determined to better understand the device's usage and optimize its performance.

In summary, IMEI is a very important identifier for IoT devices, which can be used for device management, security, remote management, and data analysis.

The details of querying other device information with the modem library is not repeated here. Please refer to [modem](../../api_reference/syslib/modem.md).

## Conclusion

This document has introduced the information of the module that can be queried through microPython scripts , which is highly concerned by the users. If you have any questions or better suggestions, please feel free to contact us. You can also directly submit documents to Gitee. This document will continue to be improved and updated in the future.

