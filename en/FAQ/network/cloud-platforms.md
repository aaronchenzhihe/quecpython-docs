# FAQ on IoT Platform

### **What is the underlying verification method- signature verification, temporary token authority verification or unique-certificate-per-device verification?**

Unique-certificate-per-device verification, unique-certificate-per-product verification and temporary Token authority verification are all suitable. Additionally, it also supports registration-free;

### **After running the code once on Alibaba IoT platform, it failed to connect the second time**
It is needed  to confirm whether the firmware has been re-burnt. After using dynamic connection, the secret key will be saved in the usr of the module. If lost, it will be unable to connect.

### **What functions does the module with umqtt support?**
The module with umqtt provides basic connection and publication and subscription for creating MQTT client terminal and encapsulated functions such as heartbeat keep-alive, reconnection and etc.

### **How to connect to Alibaba IoT platform, Tencent IoT platform, etc. via MQTT?**

For details to connect to the Alibaba IoT platform and Tencent IoT platform via MQTT, please refer to the QuecPython IoT platform in [wiki community](../../api_reference/README.md) on the official website. 

### **When the IoT platform runs the demo, it occurs "Not subscribed"**

Reason: failed Network registration, and the subscribed and published topics are not created on the IoT platform.

### **The reason why the MQTT connection is abnormally disconnected for a period of time**

Reason: The MQTT server has a heartbeat detection mechanism. If there is no communication between the device and the IoT platform for a period of time, the connection will be actively disconnected.

Solution: The disconnection is based on the timeout value-keepalive when configuring mqtt. It will disconnect actively if it exceeded the due time. We can use the timer according to the time set in keepalive to actively send ping packets to the IoT platform before the activity time is exceeded. , the data packet returned by the server does not need to be processed by the client.

### **MQTT disconnection is obtained by catching exceptions, but after disconnection, the exception cannot be caught**

An exception will be thrown and processed by umqtt. The try in upper layer will be a failure to catch the exception below. Once disconnected, it will try to reconnect and output log without throwing exception directly.

