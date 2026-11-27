# FAQs on Signal

###  **How to check signal strength?**

Two methods are available to query:

- CSQ signal strength: net.csqQueryPoll()
- RSRP signal strength: net.getSignal()

> For specific, see the link: [CSQ](../../api_reference/iotlib/net.md)

### **The relationship between signal strength and the stability of network data services **

Signal strength represents the network coverage of the current camped cell. Generally speaking, signal strength is directly proportional to the network communication stability rate. If the signal strength is poor, the network data communication will be relatively unstable; if the signal strength is good, the network data communication will be relatively stable; but it cannot be absolutely assumed that if the signal strength is good, the network data communication will be stable since it is also related to other factors such as the block network. The signal strength reference values are as follows:

CSQ signal strength: The signal strength value ranges from 0 to 31. The larger the value, the better the signal strength.

| Parameter interval | 0~15 | 15~31 |
| :----------: | :--: | :---: |
| Signal strength description | Weaker | Better |

RSRP signal strength: The received power of the downlink reference signal, ranging from -140 ~ -44 dBm. The 99 means unknown or undetectable.

| Parameter interval | <-114 | -114~-105 | -104~-95 | -94~-85 | >-85 |
| :-----------: | :---: | :-------: | :------: | :------: | :-- : |
| Signal strength description | Very weak | Weak | Fair | Strong | Excellent |

###  **Relationship between signal strength and power consumption**

- For Weak signal strength, it may need to increase power when communicating between device and network to enhance the sensitivity of signal reception in order to maintain a stable communication. Therefore, the device may consume more power in an environment with weak signal strength so as to maintain communication quality.
- For device, the stronger signal strength may mean that it is closer to the base station, has better signal quality, and the communication connection is relatively more stable. In this case, the power consumption of the device will be lower under the circumstance that the signal strength is good. 

