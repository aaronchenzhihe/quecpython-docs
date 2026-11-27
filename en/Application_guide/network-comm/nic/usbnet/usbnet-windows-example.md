# Apply USB Network Adapter on Windows 

This section takes the QuecPython EC200UEU_AB EVB as an example to demonstrate the use of USB network adapter functionality on Windows.

> RNDIS mode can be directly loaded and used on Windows without driver. If ECM mode is needed to be used on Windows, we also provide ECM driver for adaptation on Windows.

1. Prepare the development environment

   Please refer to the [Getting Started](../../../../Getting_started/README.md) section (skip if you are already familiar) to set up the development environment.

2. Open the QPYCOM tool and connect the device

   Power on the module, connect to the computer via a USB port, and use QPYCOM tool to connect to the python interface of USB.
   ![](../../../media/network-comm/nic/network_usbnet_demo_qpycom.png)

3. Enable the USB network adapter by entering interactive commands through the QPYCOM tool.

   Please refer to the corresponding [USBNET](../../../../api_reference/peripherals/misc.USBNET.md) section on the Wiki for specific USBNET interfaces.

   In the interactive interface, enter the following commands:
   ```python
   # First, import the USBNET package from misc
   >>> from misc import USBNET
   >>> from misc import Power

   # Check the current USBNET working mode. 1 represents ECM mode and 3 represents RNDIS mode
   >>> USBNET.get_worktype()
   1

   # Set it to RNDIS mode
   >>> USBNET.set_worktype(USBNET.Type_RNDIS)
   0

   # Restart the module and apply the previously configured mode
   >>> Power.powerRestart()

   # After restarting the module, import the required packages
   >>> from misc import USBNET
   >>> import dataCall

   >>> USBNET.get_worktype()
   3

   # Check if the module has successfully dialed
   >>> dataCall.getInfo(1,0)
   (1, 0, [1, 0, '10.62.213.130', '211.138.180.4', '211.138.180.5'])

   # Open the USB network adapter after the dialing information is obtained
   >>> USBNET.open()
   0
   ```

4. Check the USB network adapter connection

   You can see that a RNDIS mode network interface has been added.

   ![](../../../media/network-comm/nic/network_usbnet_demo_3.png)

   Open cmd and enter `ipconfig`. You can also see that basic network information has been obtained. At this point, close all other network interfaces and only keep the network interface of the USB network adapter. The computer can now access the Internet normally.

   ![](../../../media/network-comm/nic/network_usbnet_demo_4.png)

5. Check the network connection

   At this point, you can access web pages through a computer browser. For example, you can entering https://developer.quectel.com/en/ in the browser to access the web page.

   ![](../../../media/network-comm/nic/network_eth_gateway_web.png)