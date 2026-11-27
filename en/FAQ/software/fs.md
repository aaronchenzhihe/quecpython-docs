# FAQ on File System

### **What is the function of ql_fs?**
- Provide [`path_exists`](../../api_reference/syslib/ql_fs.md#ql_fs.path_exists)  to determine the existence of file.

- Provide [`file_copy`](../../api_reference/syslib/ql_fs.md#ql_fs.file_copy) to copy files quickly .

- Provide [`path_dirname`](../../api_reference/syslib/ql_fs.md#ql_fs.path_dirname)  to obtain the prefix pathname of last level.

- Provide [`path_getsize`](../../api_reference/syslib/ql_fs.md#ql_fs.path_getsize)  to obtain the file size.

- Provide [`mkdirs`](../../api_reference/syslib/ql_fs.md#ql_fs.mkdirs) to create nested folders .

- Provide [`rmdirs`](../../api_reference/syslib/ql_fs.md#ql_fs.rmdirs) to delete nested folders .

- Provides [`touch`](../../api_reference/syslib/ql_fs.md#ql_fs.touch) to create files quickly.

- Provide `write_json` and [`read_json`](../../api_reference/syslib/ql_fs.md#ql_fs.read_json )  to read and write json files quickly.

### **Why does it fail to delete files using uos.remove()?**
Please check whether the file is closed. Files that have been opened cannot be deleted and must be closed before they can be deleted.

### **What are the interface types for mounting SD cards on platform?**
The Unisoc 8910 platform supports mounting SDIO-SD cards.

### **What are the types of built-in file systems?**

- The built-in file system type of ECxxxU series modules is littlefs 1.x.


### **What will happen if the unclosed files are deleted directly or the power is turned off? **


If the unclosed files are deleted directly or the system is powered off at this time, file fragments will appear. There is currently no script to clean up these file fragments.

### **Is there anything we need to pay attention to regarding flash life? **

Minimize write operations in the file system. It is best to perform data synchronization processing before writing data. Do not write the same data repeatedly.

Writing files is an erasing and writing operation on the flash, which will aggravate the loss of the flash life.

> APIs that can write flash operations include file operations, net.setapn, dataCAll.setapn, system.setRelEnable, fota, appfota, sms storage, etc.; AT commands that embedded with storage and  restart function will be capable to write flash as well.

### **Will the firmware upgrade clear file system data?**
Modules can only be upgraded with differential firmware and do not change or erase file system data. 
