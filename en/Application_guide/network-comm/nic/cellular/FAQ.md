# FAQ

**1. What are Network Attach and PDN Connection Establishment?**

In cellular wireless communication, for a device to access the internet, it needs to complete two critical steps: Network Attach and PDN (Packet Data Network) Connection Establishment. The term "network attach" refers to the process of the device initiating the network attach procedure, while "dialing" refers to the process of establishing a PDN connection, which is also known as setting up a bearer. During the "dialing" process, in addition to establishing a PDN connection and obtaining an IP address assigned by the core network, the device also completes the binding of this IP address with the LWIP network protocol stack, essentially equipping the device with a "network card."

The purpose of the device initiating the network attach is to inform the network of its identity, allowing the network to verify the legitimacy of the device's identity. If the verification is successful, the device's registration on the network is completed. However, at this point, the device is not yet capable of performing network services.

As we know, both mobile phones and computers must have a unique IP address to access the internet. This is because the IP address uniquely identifies each device, much like every household has a unique house number or address. Without an IP address, the network cannot determine where the device is, and thus cannot accurately deliver information to it. For the module to obtain an IP address, it must first complete identity verification and registration on the network side and then request the establishment of a PDN connection with the network, which is referred to as "dialing." Once the module completes the PDN connection establishment, the network side assigns an IP address to the device, effectively establishing an internet access channel. At this point, the module can use this IP address to access the internet.



**2. Are the Network Attach and PDN Connection Establishment the Same in 2G/3G and 4G Networks? If Not, What Are the Differences?**

The processes of network attach and PDN connection establishment in 2G and 3G networks are quite similar, with almost identical procedures. However, significant differences arise when it comes to 4G networks. Aside from the considerable differences in signaling, the primary differences from a user’s perspective are as follows:

In 2G/3G networks, network attach and PDN connection establishment are two separate processes. The device must first complete network registration before initiating PDN connection establishment. However, in LTE (4G) networks, the network attach process actually includes the PDN connection establishment, known as establishing a "default bearer." This means that when a device successfully attaches to the LTE network, it has already established a PDN connection and received an IP address assigned by the core network.

Does this imply that a device can directly access the internet once it successfully attaches to the LTE network? Actually, no. Even though the device registers successfully on the LTE network and establishes a default bearer to obtain an IP address from the core network, this IP address is not yet bound to the "network card," meaning the TCP/IP protocol stack cannot use this IP address to perform network operations. Therefore, a "dialing" operation is still necessary.



**3. What is APN?**

APN (Access Point Name) is a crucial concept in mobile networks. It defines how a device accesses external networks (usually the internet) through the carrier's network and specifies what type of network resources the device can access.



**4. Why is it Necessary to Configure an APN?**

APN is a parameter that must be configured when the device joins the network because it determines how the device accesses external networks through the carrier's network and specifies what type of network resources it can access. If the APN is not configured or is configured incorrectly, it may result in the device failing to successfully attach to the network or establish a PDN connection.



**5. Why Can Some SIM Cards Still Successfully Attach to the Network and Establish a PDN Connection Without Configuring an APN or Even with an Incorrect APN?**

This is because the local carrier's base station has an APN auto-correction feature. When the device attempts to register on the network, if the APN is not configured or is configured incorrectly, the carrier's base station automatically corrects it during the network attachment process and sends the correct APN to the device. The device then accepts and uses this APN to continue the attachment process. This functionality depends on whether the carrier's base station supports it, and not all carriers provide this feature.

Currently, most base stations across China have this functionality, though some do not. Since it is impossible to determine in advance which base stations have this feature, we do not recommend relying on the carrier's APN auto-correction function. It is best to proactively configure the correct APN during use.



**6. In What Situations Must a User Configure an APN?**

Here are some situations where configuring an APN is necessary:

- **Lack of APN Auto-Correction by Carrier:** In some regions, the base stations of certain carriers do not have the APN auto-correction feature. In such cases, if the device does not have an APN configured or if the APN is configured incorrectly, the device will fail to attach to the network.
- **Special SIM Cards (e.g., Private Network SIMs):** For certain specialized SIM cards, such as those used for private networks, if the APN is not configured, the device will fail to attach to the network and will be rejected when attempting to initiate an Attach Request within the cell. These types of SIM cards require an APN configuration.
- **Multi-Network SIM Cards:** Some SIM cards can connect to both public networks (e.g., the internet) and private networks. The method for connecting to different networks is by configuring different APNs.

We recommend that users proactively configure the correct APN, regardless of the type of SIM card being used.



**7. How to Confirm the APN Information of a SIM Card?**

You can confirm the APN information by contacting your carrier.



**8. How to Configure the Network Attach APN? Is a Reboot Required? Is the Configuration Saved After Power Loss? Does It Need to Be Configured Every Time the Device Is Powered On?**

- **How to Configure the Network Attach APN**

  The APN can be configured using the `dataCall.setPDPContext` interface, with the `profileID` parameter set to 1. For detailed instructions on using this interface, please refer to the [APN Configuration and Retrieval Functionality](../../../../api_reference/iotlib/dataCall.md#APN-Configuration-and-Retrieval) section of the QuecPython official WIKI.

- **Is a Reboot Required After Configuration?**

  There are two ways to make the configuration take effect:

  - Method 1: Reboot the device.
  - Method 2: Perform a CFUN0/1 switch.

- **Is the Configured APN Saved After Power Loss?**

  Yes, the configuration is saved, meaning the parameters set via software will not be lost due to power loss or device restart.

- **Does It Need to Be Configured Every Time the Device Is Powered On?**

  No, it does not need to be configured every time. For the same single-carrier SIM card, you only need to configure it once.



**9. Why can the SIM card be recognized but fail to register on the network?**

This situation is relatively complex. Please refer to [Module Network Registration Failure](./exception-handling.html#Module Network Registration Failure) in the *Handle Network Exceptions* chapter for details. Here are some common reasons:

- The RF antenna is not connected or the RF antenna performance is poor.
- The SIM card requires APN configuration to register to the network, but no APN is configured.
- The SIM card is bound to a specific device. You need to contact the operator to unbind the SIM card or replace it with a new SIM card.
- The testing location is not covered by the network of the SIM card operator.
- The SIM card only supports specific frequency bands.
- The SIM card only supports specific network modes.
- There are hardware issues with the device, such as interference caused by hardware design flaws, or damaged RF components.



**10. What Is a Multi-Carrier SIM Card, and What Are the Usage Considerations?**

Typically, the SIM cards we use contain configuration information for only one carrier. For example, a China Mobile SIM card contains only China Mobile's carrier configuration information, which we refer to as a profile. However, some SIM cards have multiple carrier profiles built-in, meaning they can support networks from multiple carriers. These SIM cards are known as multi-carrier SIM cards.

A multi-carrier SIM card usually starts by using one of the carrier profiles by default when the device is powered on, searching for and attaching to the network. If the network attachment is not successful within a certain period, the SIM card will automatically switch to another carrier profile and attempt to search for and attach to the network again using the newly switched profile. Additionally, in some cases, even after the device has successfully attached and established a PDN connection using the current profile, the SIM card may still switch profiles. The specific conditions under which this profile switching occurs should be confirmed with the carrier that provides the multi-carrier SIM card.

It is important to note that each profile within such a SIM card corresponds to a unique IMSI (International Mobile Subscriber Identity). When the SIM card switches profiles, the IMSI associated with the SIM card also changes. Every time a profile switch occurs, if the device is connected to the network, this will result in a temporary disconnection from the network until the device successfully attaches and establishes a PDN connection using the new carrier profile.



**11. Why cannot the NIC perform network services after being activated successfully?**

- Your SIM card is in arrears. The module can register on the network and activate the NIC successfully even when the SIM card inserted is in arrears, but the operator will restrict the module from network communication.

- Your SIM card supports access to a special network through a specific APN and you have configured the APN for accessing the special network. In this case, your device can only access the special network instead of the Internet. If you try to access the Internet, the access will fail.



**12. When the automatic reconnection of the NIC is enabled, do Socket, HTTP, and MQTT need to be reinitialized after the NIC is re-connected to the network if the network connection is disconnected due to some reasons during the normal operation of the module?**

You should know that the IP address of the NIC will be changed as long as the NIC is automatically re-connected to the network after disconnecting from the network due to network exceptions. In this case, you need to perform the following steps for Socket, HTTP, and MQTT features.

- Socket

  When creating a socket object, the system automatically binds it to the current IP address of the cellular NIC. When the IP address of the cellular NIC changes, the previous socket becomes invalid and you need to close the previous socket and create a new socket object.

- HTTP

  HTTP is a short connection service based on socket. Each time an HTTP API is called, a new socket object is created. Therefore, the automatic reconnection feature does not affect the HTTP functionality, and you can continue to use the HTTP feature.

  > The socket object created in the HTTP API will be automatically closed by the system after you obtain the response (response.text & response.content & response.json). If you do not obtain the response, you need to manually call `response.close()`  to close the connection and close the socket object.

- MQTT

  The MQTT client in QuecPython solution has the automatic reconnection feature. When the MQTT client detects that the cellular NIC is automatically reconnected to the network after a network disconnection, the MQTT client will be automatically initialized and automatically subscribe to the relevant topics according to the previous subscription information. Therefore, after the cellular NIC is automatically reconnected, you do not need to reinitialize the MQTT client and can continue to use the MQTT feature.