# Handle Common Exception

For SIM card status exceptions and network registration and dialing exceptions, please refer to the [Handle Network Exceptions](../nic/cellular/exception-handling.md) section of the QuecPython official website wiki for cellular wireless network cards.

## Unable to send SMS
* Check if the SIM card status is abnormal.
* Check if the device is registered to the network properly.
* Verify if the SMS center number is correctly set.
* Confirm if the SMS service is activated.
* Check if the device is within the service area.
* Check if the signal is too weak.

## Unable to make phone calls
* Check if the SIM card status is normal.
* Check if the device is registered to the network properly.
* Verify if the voice call service is activated.
* Check if the device is within the service area.
* Check if the signal is too weak.
* Check if the voice-related functions are working properly.

## Successful network registration and dialing, but unable to access the URL
* Check if the SIM card is overdue, causing the operator does not forward the response data or request message to the other end. If overdue, please recharge.
* For other exceptional cases, please refer to the [Handle Network Exceptions](../nic/cellular/exception-handling.md) section of the QuecPython official website wiki.

## Some protocol data cannot be exchanged properly
* For SIM cards of this type, check if the operator has restricted certain protocols. For example, for the NTP protocol, some operators restrict it to be sent only to specific NTP service addresses, otherwise it cannot be exchanged. Verify with the operator and activate this type of service.
* For other exceptional cases, please refer to the [Handle Network Exceptions](../nic/cellular/exception-handling.md) section of the QuecPython official website wiki.