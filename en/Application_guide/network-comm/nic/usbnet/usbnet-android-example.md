# Apply USB Network Adapter on Android

This section takes QuecPython EC200UEU_AB EVB as an example to demonstrate how to use a USB network adapter on an Android phone.

> The ECM mode can be directly loaded and used in Linux, Android, or IOS without a driver. 
> In ECM mode,  the MAC address of some module models may not be recognized by the phone. In such cases, you can refer to the corresponding [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) section on the Wiki and update the available MAC address with MAC configuration interface .

1. Prepare the development environment  

   Please refer to the [Getting Started](../../../../Getting_started/README.md) section (skip if you are already familiar) to set up the development environment. 
   You will also need a mobile phone and a cable that can be converted into a mobile phone interface.

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

5. Connect the USB to the phone

   After configuring the startup on the computer, connect the USB port to the Type-C port on the phone and power on the module by pressing the PWRKEY button. Please prepare the adapter cable yourself.

   ![](../../../media/network-comm/nic/network_usbnet_demo_6.png)

6. Connect to the phone network

   Once the module is connected to the phone and powered on, you will see a `<···>` icon in the status bar of the phone, indicating that the USB network adapter has been recognized by the phone. You can now open the browser and access the network.

   ![](../../../media/network-comm/nic/network_usbnet_demo_7.png)