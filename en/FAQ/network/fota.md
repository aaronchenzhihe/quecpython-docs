# FAQ on FOTA

### **How many methods can be used to upgrade module firmware and business scripts? **


Firmware upgrade: differential upgrade, minimum system upgrade

Script upgrade: app_fota upgrade, whole package upgrade, full upgrade

###  **How to perform firmware upgrade?**

1. Create upgrade package via QPYcom
2. Upload the upgrade package to the server.
3. Download the upgrade package according to the link address of server, write it to the device, and verify the upgrade package via HTTP or FTP. 
4. After verifying the upgrade package, please reboot the module. The system will automatically perform the OTA upgrade after the device restarts.
5. After the upgrade is completed, you can check the firmware version through modem.getDevFwVersion().

> For specific, please refer to [fota](../../api_reference/syslib/fota.md): 
>

###  **How to upgrade the business?**

1. Upload the target file to the server.
2. Download the target file to be upgraded to the file system according to the link address of server through HTTP.
3. Set the upgrade flag to record whether file upgrade is required.
4. Reboot the module. After that, upgrade and delete relevant flag at the same time.
5. After the upgrade is completed, it is available to view the upgraded target files in the file system partition.

> For specific, please refer to [app_fota](../../api_reference/syslib/app_fota.md).

###  **What is the mechanism of fota upgrade? Can it be guaranteed that it will not become bricked after the upgrade fails? Can the upgrade be continued after powering off and on again?**

- FOTA upgrade mechanism: start to upgrade - download the target package - download successfully - verify - verify successfully - restart the module - upgrade successfully.
- For minimum system upgrade,  it will not be guaranteed, while for other upgrade methods,  it will not become bricked after the upgrade fails. It lies in that the minimum system will upgrade while refreshing the firmware partition. However, the other firmware upgrade methods can return to the original mode instead of being bricked. 
- There are two situations of power outage:

1. The power outage in downloading: After powering on and running again, it is needed to call the upgrade interface to re-trigger the download, which will start from the beginning.
2. The power outage in upgrading: It is available to record the upgrade flag and the location of the last upgrade. After powering on again, continue the upgrade in the location of last time until the upgrade is completed and restarted. The main principle is: before each upgrade, back up the original firmware content of the partition being upgraded, and record the offset position of the current upgrade. After the next power-on, the upgrade can be continued from position of the last upgrade.

### **Under what circumstances cannot the FOTA upgrade be performed?**

1. The old and new firmware are not the firmware package of the same baseline .
2. Insufficient memory space.
3. The upgrade address is wrong.
4. The minimum system does not support HTTPS upgrade.
5. Network outage.



