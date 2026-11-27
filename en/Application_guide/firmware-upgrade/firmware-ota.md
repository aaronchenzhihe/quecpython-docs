# Firmware OTA Upgrade

## Overview

OTA (Over The Air) refers to wireless download technology. OTA upgrade is the process of downloading and upgrading the device's software over a wireless network, allowing for updates such as bug fixes, version upgrades, etc., without the need for a wired connection.

## Categories of OTA Upgrades

OTA upgrades mainly consist of wireless network downloading of upgrade packages, writing the contents of the upgrade package to flash storage, package verification, and rebooting for upgrade. According to the technical implementation, they can be categorized as full upgrade, differential upgrade, and minimal system upgrade.

### Full Upgrade

A full upgrade involves replacing all the content of the current version on the device with the complete content of the target version.

To be compatible with the software process and the differential upgrade mentioned later, a full upgrade is not simply placing the original upgrade package on the cloud for the device to download. It also needs to format the original data by adding headers, footers, verification data, etc. We refer to this new data package as a full package.

### Differential Upgrade

A differential upgrade extracts the differences between the current version on the device and the target version. After downloading the differential file to the device, using a differential restoration algorithm, the differential data is applied to the current version in a patch-like manner, thereby restoring the data of the target version. This differential file is called a differential package.

This process can be broadly divided into 4 stages:

- Differential package creation: Extraction and compression of differential data.
- Upload of the differential package to the cloud.
- Download of the differential package to the device.
- Decompression of the differential package and data restoration.

### Minimal System Upgrade

A minimal system upgrade combines both differential and full upgrades, making it a customized solution. It is mainly designed for devices with limited flash storage space that cannot accommodate the upgrade package file. When creating the upgrade package, two upgrade package files are generated, and multiple automatic reboots are required.

> Comparison of the three upgrade methods:
>
> - The full upgrade package requires more storage space compared to the differential upgrade, resulting in higher data consumption during downloads.
>
> - Full upgrades are faster than differential upgrades.
>
> - Full upgrades do not consider the currently running version and can be upgraded to any other version. In contrast, differential upgrades can only be upgraded to a specific version and require maintenance of differential packages and target firmware packages for different versions, making firmware maintenance a bit more complicated.
>
> - Minimal system upgrades fall between full and differential upgrades in terms of features.

## QuecPython OTA Solution

### System Architecture

Quectel's modules generally consist of multiple partitions, used to store program images, factory parameters, file systems, etc.

Even if some partition data has changed, because a full upgrade is an overwrite upgrade, a full upgrade can still be used. The data most likely to change is the file system.

For a differential upgrade, it is achieved by generating a patch on the data of the current version and applying the patch to the device. Therefore, differential upgrades can only be applied to partitions where data will not change. For program images, factory parameters, etc., which cannot be modified without special tools after leaving the factory, these partitions can be differentially upgraded.

For details on the QuecPython partition table, refer to [Storage Device Application Note](../system/memory-device.md)。

### Solution Design

The storage space occupied by the firmware program image is relatively  large, and the device's storage space is limited. Therefore, the  firmware uses a differential upgrade solution. However, the file system  partition's content changes after the device starts, making it  unsuitable for differential upgrades. Thus, user application scripts can only undergo a full upgrade. Consequently, firmware and application  scripts cannot be upgraded simultaneously. Therefore, QuecPython OTA  upgrades are also divided into firmware upgrades and app upgrades. In  actual use, to achieve remote control and automatic OTA upgrades of  devices, a complete OTA solution involving both the device and the cloud platform is typically implemented, as shown in the diagram below.

![](../media/firmware-upgrade/firmware-ota/aws_ota_flow_chart.png)

### Firmware Upgrade Steps

#### Device-Side Preparation

##### Firmware Preparation

Prepare firmware for both the new and old versions. The firmware of the  old version must match the one flashed to the module; otherwise, it may  result in a failed upgrade.

##### Delta Package Preparation

The preparation method may vary for different device models. Refer to [FOTA Upgrade Package Making](./firmware-ota-delta-package-making.md) for specific details.

##### Code Preparation

###### 1. Initialize Cloud Platform Functionality

The role of the OTA cloud platform is to:

​	a. Store upgrade package files.

​	b. Facilitate message notifications, such as the start of an upgrade.

​	c. Manage upgrade status, such as success or failure.

The OTA cloud platform enables web-based control for automatic OTA  upgrades. Before use, initialize the functionality related to the cloud  platform, such as filling in certificates and connecting to the cloud  platform.

###### 2. Receive Cloud Platform Upgrade Messages

Users create and deploy OTA jobs on the cloud platform webpage. The  device periodically polls for message notifications and obtains the  upgrade package URL from the message. After initializing the cloud  platform functionality, subscribe to topics related to OTA jobs and  periodically publish specific topics to receive message notifications  sent by the cloud platform.

###### 3. Download Upgrade Package

Once the upgrade package URL is obtained, the device uses firmware  upgrade-related APIs for package download, writing, and verification.  For API details, refer to [fota - Firmware Upgrade](../../api_reference/syslib/fota.md).

###### 4. Restart

After successfully verifying the upgrade package, the module needs to be restarted (either automatically or manually). The system will  automatically perform the upgrade after the device restarts.

###### 5. Run the New Firmware

After the upgrade is complete, the new firmware runs automatically.  Compare the current version number with the version number in the OTA  job document on the cloud platform. If they match, publish a specific  topic and report the OTA success status to the cloud platform.

#### Cloud Platform Preparation

##### Amazon Cloud

If using the OTA feature of the Amazon Cloud platform, you need to first connect to the Amazon Cloud.

Here, we use Amazon Cloud as an example to demonstrate the steps for using the OTA platform:

1. Create an S3 storage bucket. In the image below, `qpyota` is the created storage bucket.

![](../media/firmware-upgrade/firmware-ota/aws_s3.png)

![](../media/firmware-upgrade/firmware-ota/aws_s3_create.png)

2. Upload the upgrade package file and OTA job description document to the S3 storage bucket. Click the `Upload` button in the image below to perform the upload. In the image below, "EG915UEUAB_V0002-V0003.pack" is the differential package for FOTA upgrade, and `fota.json` is the description document for the FOTA upgrade job.

![](../media/firmware-upgrade/firmware-ota/aws_s3_fota_upload.png)

The format of the upgrade job description document is as follows:

```json
[
  {
     "operation":"FOTA",
     "files":[
        {
           "fileName":"EG915UEUAB_V0002-V0003.pack",
		   "fileVersion": "EG915UEUABR03A21M08_OCPU_QPY",
           "fileSource":{
              "url":"${aws:iot:s3-presigned-url:https://qpyota.s3.us-west-2.amazonaws.com/EG915UEUAB_V0002-V0003.pack}"
           }
        }
     ]
  }
]
```

3. Create OTA Upgrade Job

Navigate to the AWS IoT section, select `Manage` -> `Remote actions` ->` Jobs`, and click the `Create Job` button.

![](../media/firmware-upgrade/firmware-ota/aws_ota_jobs_create.png)

After entering the job name and description in step 1, proceed to step 2 for file configuration:

a. Choose the target items for the job. You can select specific items like QuecPython_OTA or choose item groups containing multiple devices. These items should have been created beforehand.

b. Next, select the job description document, which is the previously uploaded `fota.json` file in the S3 storage bucket.

c. Afterward, create an IAM role for the pre-signed URL. The purpose of this pre-signed URL is to generate a time-limited upgrade package file URL for device downloading. Due to security restrictions, devices cannot directly download the upgrade package file from the S3 storage bucket. Here, the URL is set to be valid for 60 minutes.

![](../media/firmware-upgrade/firmware-ota/aws_ota_jobs_create1.png)

![](../media/firmware-upgrade/firmware-ota/aws_ota_jobs_create2.png)

After completing the above, proceed to step 3 for job configuration. 

Choose job run type and click `submit` button to complete the job creation. The `jobfota` is the created job.

![](../media/firmware-upgrade/firmware-ota/aws_ota_jobs_create3.png)

![](../media/firmware-upgrade/firmware-ota/aws_ota_jobs_create4.png)

Clicking on the `jobfota` allows you to view job details. The `Job  executions` page displays the status of the upgrade job, with one item,  QuecPython_OTA, currently queued and waiting for the upgrade job to be  executed. The `Job document` page provides details of the current upgrade job, such as the upgrade mode being "FOTA," the upgrade package name  being "EG915UEUAB_V0002-V0003.pack," the target version to upgrade to  being "EG915UEUABR03A21M08_OCPU_QPY," and the URL of the upgrade package file.

![](../media/firmware-upgrade/firmware-ota/aws_ota_jobs_create5.png)

![](../media/firmware-upgrade/firmware-ota/aws_ota_jobs_create6.png)

4. Run the upgrade program on the device, wait for the device to upgrade, and check the upgrade result.

After completing the download, report the download result.

![](../media/firmware-upgrade/firmware-ota/aws_ota_result.png)

Reporting a successful upgrade after completion and reboot.

![](../media/firmware-upgrade/firmware-ota/aws_ota_result1.png)

### APP Upgrade Steps

These steps are for upgrading QuecPython client application scripts,  including Python files, JSON files, and audio files. The chosen upgrade  approach is a full upgrade, meaning that after downloading the target  files, they replace the existing files on the device.

#### Device-Side Preparation

##### Upgrade Package Preparation

Since it is a full upgrade, only the target files to be upgraded need to be prepared as the upgrade package.

##### Code Preparation

###### 1. Initialize Cloud Platform Functionality

Utilize the OTA cloud platform to achieve web-based control for  automatic OTA upgrades. Before use, it is necessary to initialize the  functions related to the cloud platform, such as filling in certificates and connecting to the cloud platform.

###### 2. Receive Cloud Platform Upgrade Messages

When users create and deploy OTA tasks on the cloud platform's web page, the device retrieves message notifications through periodic polling,  obtaining the upgrade package URL from the messages. After initializing  the cloud platform functions as mentioned earlier, subscribe to topics  related to OTA tasks and periodically publish specific topics to receive message notifications sent by the cloud platform.

###### 3. Download the Upgrade Package

Once the upgrade package URL is obtained, the device downloads the  target files to be upgraded into the file system based on the URL. Both  single-file and multi-file batch download methods are supported. For API details, refer to [app_fota - User File Upgrade](../../api_reference/syslib/app_fota.md).

###### 4. Set Upgrade Flag

Set the upgrade flag to record whether an upgrade is to be performed.

###### 5. Reboot

Reboot the device (either automatically or manually). The system will  automatically perform the OTA upgrade after the device restarts.

###### 6. Run the New Application

After the upgrade is complete, the upgrade flag is cleared, and the new  application is launched. Compare the current version number with the  version number in the OTA task document on the cloud platform. If they  match, publish a specific topic and report the success status of OTA to  the cloud platform.

#### Cloud Platform Preparation

##### Amazon Cloud

Taking Amazon Cloud as an example, here are the steps for using the OTA platform:

1. Upload the upgrade package files and OTA task description document to  the S3 storage bucket. Click the `upload` button in the figure below to  perform the upload. In the figure below, `a.py` and `b.py` are target Python script files for app_fota upgrade, and `app_fota.json` is the description document for the app_fota upgrade task.

![](../media/firmware-upgrade/firmware-ota/aws_s3_upload.png)

The format of the upgrade task description document is as follows:

```json
[
  {
     "operation":"app_fota",
     "files":[
        {
           "fileName":"/usr/a.py",
		   "fileVersion": "2.0",
           "fileSource":{
			  "url":"${aws:iot:s3-presigned-url:https://qpyota.s3.us-west-2.amazonaws.com/a.py}"
           }
        },
		{
           "fileName":"/usr/b.py",
		   "fileVersion": "2.0",
           "fileSource":{
			  "url":"${aws:iot:s3-presigned-url:https://qpyota.s3.us-west-2.amazonaws.com/b.py}"
           }
        }
     ]
  }
]
```

2. Create an OTA Upgrade Job

The specific steps are the same as  those for firmware upgrade job creation. The only difference is the  selection of the task description document. Choose the `app_fota.json` file that was uploaded earlier for the app upgrade.

![](../media/firmware-upgrade/firmware-ota/aws_appota_jobs_create.png)

3. Run the upgrade program on the device, wait for the device to upgrade, and check the upgrade result.

After completing the download, report the download result.

![](../media/firmware-upgrade/firmware-ota/aws_appota_result.png)

Report the upgrade results after completion and reboot.

![](../media/firmware-upgrade/firmware-ota/aws_appota_result1.png)

## Sample Code

Code of `aws.py` script:

```python
from umqtt import MQTTClient
import ujson
import utime as time
import _thread
import log

class Aws(object):
    """
    Azure client
    """
    def __init__(self, client_id, username, password, server, port, keep_alive=60, ssl=False, ssl_params=None):
        self.client_id = client_id
        self.username = username
        self.password = password
        self.server = server
        self.port = port
        self.ssl = ssl
        self.ssl_params = ssl_params
        self.client = None
        self.connected = False
        self.keep_alive = keep_alive
        self.callback = None
        self.logging = log.getLogger("AWS")

    def _connect(self):
        try:
            self.client = MQTTClient(self.client_id,
                                     self.server,
                                     port=self.port,
                                     user=self.username,
                                     password=self.password,
                                     keepalive=self.keep_alive,
                                     ssl=self.ssl,
                                     ssl_params=self.ssl_params)
            self.client.connect()
            self.connected = True
            if self.callback:
                self.client.set_callback(self.callback)
            self.logging.info("Connected to Aws IoT Hub")
            return True
        except Exception as e:
            self.logging.error("Failed to connect to Aws IoT Hub: %s", e)
            return False

    def set_callback(self, callback):
        self.callback = callback

    def connect(self):
        if not self.check_connection():
            self._connect()

    def check_connection(self):
        return self.connected

    def disconnect(self):
        try:
            if self.client:
                self.client.disconnect()
                self.client = None
                self.connected = False
                self.logging.info("Disconnected from Aws IoT Hub")
                return True
        except Exception as e:
            self.logging.error("Failed to disconnect from Aws IoT Hub: %s", e)
            return False

    def subscribe(self, topic):
        try:
            if self.client:
                self.client.subscribe(topic)
                self.logging.info("subscribe success")
                return True
        except Exception as e:
            self.logging.error("Failed to subscribe to Aws IoT Hub: %s", e)
            return False

    def publish(self, topic, payload):
        try:
            if self.client:
                self.client.publish(topic, ujson.dumps(payload))
                self.logging.info("publish success")
        except Exception as e:
            self.logging.error("Failed to publish message: %s", e)

    def loop(self):
        try:
            while True:
                if self.client:
                    self.client.wait_msg()
                else:
                    time.sleep_ms(100)
        except Exception as e:
            self.logging.error("Error in MQTT loop: %s", e)

    def loop_forever(self):
        _thread.start_new_thread(self.loop, ())
```

Code of `aws_ota_test.py` script:

```python
from usr.aws import Aws
import modem
import ujson
import uos
import fota
import app_fota
from misc import Power
import utime

# certificate.crt
certificate_content = """
# (content of certificate.pem.crt)
"""
# private.pem
private_content = """
# (content of private.pem.key)
"""

# device name
client_id = 'QuecPython_OTA'
# server address
server = 'ambzd54j67b7h-ats.iot.us-west-2.amazonaws.com'
# port
port = 8883
# user
user = None
# pwd
pwd = None

# get firmware version
FIRMWARE_VERSION = modem.getDevFwVersion()

import usr.a as a
APP_VERSION = a.getversion()

def fota_callback(args):
    print("Download status: %s, Download process: %s" % tuple(args))
    status = "IN_PROGRESS"
    progress = args[1]
    process_data = {"status": status,"statusDetails":{"progress":progress, "step":"downloading"}}
    aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobfota/update", process_data)

def event_callback(topic, data):
    """
    aws callback
    """
    import ujson
    global operation
    receive_data = ujson.loads(data.decode())
    # print("Subscribe Recv: Topic={},Msg={}".format(topic.decode(), data.decode()))
    operation = receive_data["execution"]["jobDocument"][0]["operation"]
    version = receive_data["execution"]["jobDocument"][0]["files"][0]["fileVersion"]
    
    if operation == "FOTA":
        # FOTA
        if version == FIRMWARE_VERSION:
            print("fota SUCCEEDED")
            process_data = {"status": "SUCCEEDED","statusDetails":{"operation": 'install', "state": 'package installed and started'}}
            aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobfota/update", process_data)
        else:
            print("start fota")
            _fota = fota()
            ret = _fota.httpDownload(url1=receive_data["execution"]["jobDocument"][0]["files"][0]["fileSource"]["url"], callback=fota_callback)
            if ret == 0:
                print("fota download completely", ret)
            else:
                status = "FAILED"
                print("fota FAILED", ret)
                process_data = {"status": status,"statusDetails":{"progress":100}}
                aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobfota/update", process_data)
    elif operation == "app_fota":
        # app_fota
        if version == APP_VERSION:
            print("app_fota SUCCEEDED")
            process_data = {"status": "SUCCEEDED","statusDetails":{"operation": 'install', "state": 'package installed and started'}}
            aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobappota/update", process_data)
        else:
            print("start app_fota")
            _app_fota = app_fota.new()
            ota_data = [{"url": i["fileSource"]["url"], "file_name": i["fileName"]} for i in receive_data["execution"]["jobDocument"][0]["files"]]
            res = _app_fota.bulk_download(ota_data)
            ota_process = 100 if not res else -1
            if ota_process == 100:
                _app_fota.set_update_flag()
                print("app_fota download completely")
                process_data = {"status": "IN_PROGRESS","statusDetails":{"progress":ota_process, "step":"downloading"}}
                aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobappota/update", process_data)
                utime.sleep(5)
                # restart device
                Power.powerRestart()
            else:
                status = "FAILED"
                print("app_fota FAILED", res)
                process_data = {"status": status,"statusDetails":{"progress":100}}
                aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobappota/update", process_data)

# create aws obj
aws_obj = Aws(client_id, user, pwd, server, port, keep_alive=60,ssl=True,ssl_params={"cert": certificate_content,"key": private_content})
print("create aws obj")
# register callback
aws_obj.set_callback(event_callback)
print("aws set callback")

# connect mqtt server
print("aws connect start")
aws_obj.connect()
print("aws connect end")

# subscribe jobs topic
aws_obj.subscribe("$aws/things/QuecPython_OTA/jobs/notify")
aws_obj.subscribe("$aws/things/QuecPython_OTA/jobs/get/accepted")

# start
aws_obj.loop_forever()
print("aws loop_forever")

# get ota jobs
utime.sleep(5)
a = {"key":0}
aws_obj.publish("$aws/things/QuecPython_OTA/jobs/get", a)
utime.sleep(5)
aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobfota/get", a)
utime.sleep(5)
aws_obj.publish("$aws/things/QuecPython_OTA/jobs/jobappota/get", a)
```

## Support Status for Different Module Models

| Model                                   | **Differential Upgrade** | **Minimal System Upgrade** |
| --------------------------------------- | ------------------------ | -------------------------- |
| EC600NCNLC&EC600NCNLF&EGx00N Series     | Supported                | Supported                  |
| EC600NCNLA&EC600NCNLE&EC800NCNLA Series | Not Supported            | Supported                  |
| ECx00M&EGx00M Series                    | Not Supported            | Supported                  |
| EC200A Series                           | Supported                | Not Supported              |
| ECx00U&EGx00U&ECx00G Series             | Supported                | Not Supported              |
| ECx00E Series                           | Supported                | Not Supported              |
| BG95&BG600L Series                      | Supported                | Not Supported              |
