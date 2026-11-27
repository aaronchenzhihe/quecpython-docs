# Apply USB Network Adapter on Linux

This section takes the QuecPython EC200UEU_AB EVB as an example to demonstrate the use of USB network adapter functionality on Linux.

> The ECM mode can be directly loaded and used in Linux, Android, or IOS without a driver. 

1. Prepare the development environment

   Please refer to the [Getting Started](../../../../Getting_started/README.md) section (skip if you are already familiar) to set up the development environment. 
   In this case, the Ubuntu version of the Linux system is used for testing.

2. Open the QPYCOM tool and connect the device

   Power on the module, connect to the computer via a USB port, and use QPYCOM tool to connect to the python interface of USB.
   ![](../../../media/network-comm/nic/network_usbnet_demo_qpycom.png)

3. Write a startup script for the USB network adapter

   Please refer to the corresponding [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) section on the Wiki for specific USB network adapter interface.  
   Create a new file named `main.py` and enter the following content:

   ```python
   # Import required packages
   from misc import USBNET
   from misc import Power
   import sim
   import dataCall
   import utime
   
   def usbnet_start():
       # Check if the USBNET working mode is ECM, if not, set it to ECM and restart the module.
       saved_type = USBNET.get_worktype()
       if saved_type != USBNET.Type_ECM :
           USBNET.set_worktype(USBNET.Type_ECM)
           Power.powerRestart()
       utime.sleep(2)
       # Check SIM card status, exit if no SIM card is inserted.
       sim_info = sim.getStatus()
       if sim_info != 1:
           return
       # Continuously check the network registration status, and enable USBNET when network registration information is obtained.
       while True:
           lte = dataCall.getInfo(1,0)
           if type(lte) == tuple and lte[2][0] == 1:
               if lte[2][2] != '0.0.0.0':
                   break
           utime.sleep(1)
       # Enable USBNET functionality.
       USBNET.open()
   
   usbnet_start()
   ```

4. Import the startup file

   Drag the written script into the module file system's usr partition via the QPYCom file panel. The `main.py` file is executed during startup.

   ![](../../../media/network-comm/nic/network_usbnet_demo_5.png)

5. Connect the USB to Ubuntu

   Connect the USB port of the device to the Ubuntu system.

   ![](../../../media/network-comm/nic/network_usbnet_ubuntu_choose.png)

6. Confirm the USB network adapter connection

   After the network adapter is loaded, check the network adapter status with the `ifconfig` command. You will see the newly added usb0 network adapter. As shown in the following figure:

   ![](../../../media/network-comm/nic/network_usbnet_ubuntu_nic.png)

7. Confirm network connectivity

   Check the connection by pinging `python.quectel.com`.  As shown in the following figure:

   ![](../../../media/network-comm/nic/network_usbnet_ubuntu_ping.png)