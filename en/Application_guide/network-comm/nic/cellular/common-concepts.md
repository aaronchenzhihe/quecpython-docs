# Basic Concepts of Cellular Network

## APN

### What is APN

APN (Access Point Name) refers to a network access technology that is a required parameter for terminal access. It determines the access method for the terminal to access the network.

For users, many types of external networks can be accessed, such as [Internet](https://en.wikipedia.org/wiki/Internet), [WAP websites](https://en.wikipedia.org/wiki/Wireless_Application_Protocol), internal networks of corporate groups, and industry-specific networks. Different access points have different access ranges and access methods. How does the network know which network the terminal should access and allocate the corresponding IP address? The answer is APN. It is APNs that determine how a terminal accesses which network.

All operators use specific APNs (Access Point Names), which are usually pre-configured on your SIM card, but you may need to adjust them manually if necessary.

For APN configuration interface description, please refer to the following sections:

[APN configuration interface description](../../../../api_reference/iotlib/dataCall.md#APN-Configuration-and-Retrieval)

APN configuration example:

* [Automatically Activate One NIC at Startup and Configure APN](application.md#Automatically-Activate-One-NIC-at-Startup-and-Configure-APN)

* [Automatically Activate Multiple NICs at Startup and Configure APN](application.md#Automatically-Activate-Multiple-NICs-at-Startup-and-Configure-APN)

* [Manually Activate One NIC](application.md#Manually-Activate-One-NIC)

* [Manually Activate Multiple NICs](application.md#Manually-Activate-Multiple-NICs)

### Confirm Which APN to Use

All operators have their own APNs. Generally, for regular SIM cards (also known as public network cards), their APNs are publicly available and can be found online or by contacting the corresponding operator. However, for IoT SIM cards or other specialized network cards, you need to contact the corresponding operator to confirm which APN to use.



## CFUN

CFUN (Cellular Functionality) refers to the functional mode of a mobile terminal. When talking about CFUN, we generally refer to the APIs of `net` feature or the AT command `AT+CFUN` to set or get the functional mode of the mobile terminal. For how to get and set CFUN, please refer to [Get/Set Module Work Mode](./api-instruction.md#Get/Set-Module-Work-Mode) section in the *Cellular Network API* chapter. Mobile terminals usually have the following functional modes:

* Minimum functionality mode: In this mode, the entire radio frequency network protocol stack is turned off, and the SIM card is powered off. In this mode, the power consumption of the terminal device is the lowest when it is not turned off.

* Full functionality mode: In this mode, all wireless radio frequency functions of the terminal device are enabled, and the device can perform network-related operations.

* Airplane mode: In this mode, the wireless radio frequency functions of the terminal device are disabled, that is, the device is prohibited from sending and receiving RF signals, but the SIM card is still powered on and can be recognized.



## MCC/MNC

MCC (Mobile Country Code) and MNC (Mobile Network Code) are used to distinguish a mobile network operator in a mobile communication network.

* MCC: Mobile Country Code. MCC is a 3-digit number used to identify the country or region where the mobile device is located. For example, the MCC for China is 460.

* MNC: Mobile Network Code. MNC is a 2-digit or 3-digit number used to identify a specific mobile network operator within a particular country or region. For example, the MNC for China Mobile is 00 or 02, the MNC for China Unicom is 01, and the MNC for China Telecom is 03 or 05.

The combination of MCC and MNC forms a globally unique code used to identify every mobile network operator in the world. The identifier consisting of MCC and MNC is called PLMN (Public Land Mobile Network).



## Cell

In a mobile network, a cell represents a specific geographical area covered by a base station. Each cell is covered by a base station and receives network services from the base station. The coverage area of each cell is different and can be adjusted according to needs and environment.

For a UE, cells are divided into serving cells and neighboring cells.

* Serving Cell: The serving cell is the cell currently providing services to the mobile device. In other words, the device is currently communicating with the base station of this cell, and all activities such as calls and data transmission are performed through this cell.

* Neighboring Cell: Neighboring cells are cells adjacent to the serving cell and may become new serving cells when the mobile device moves. The mobile device periodically measures the signal quality of neighboring cells to facilitate cell switching when necessary.



## Signal Quality

### Overview

In cellular mobile networks, signal quality is determined by different measurement values, not a single value of a parameter. The parameters used to measure signal quality and their ranges generally vary in different network modes. Here are some common measurement values:

* [RSSI](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength)

* [CSQ](../../../../api_reference/iotlib/net.md#Get-Signal-Strength)

* [RSRP](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength)

* [RSRQ](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength)

* [RSCP](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength)

* [SINR](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength)

### Factors Affecting Signal Quality

Many factors can affect signal strength and signal quality, such as:

* Distance to the base station: The farther the distance between the mobile device and the base station, the weaker the signal strength and the lower the signal quality. This is because the signal attenuates as it travels.

* Obstacles: Buildings, trees, hills, and other obstacles can block signal transmission, resulting in decreased signal strength and signal quality. This kind of signal attenuation is particularly serious indoors.

* Antenna performance: The antenna performance of the device and the base station can also affect signal strength and signal quality.

* Interference: Signals emitted by other devices may conflict with the target signal, resulting in decreased signal quality. This kind of signal attenuation is particularly serious in places with frequent use of wireless devices (such as public Wi-Fi areas).

* Multipath effects: Signals may reflect, refract, or scatter before reaching the receiver, resulting in multiple signal paths. These signals from different paths may interfere with each other, resulting in decreased signal quality.

* Weather conditions: Certain weather conditions such as rain, fog, and snow can also affect wireless signals, reducing signal strength and quality.

* Network congestion: During periods of high network usage, such as peak hours, a large number of network users may cause network congestion, which can affect signal quality.

* Base station load: If a base station carries too many users or a large amount of data traffic within a certain period, it may also affect signal quality.

* Power control: Mobile communication systems usually use power control mechanisms to optimize signal strength and avoid interference caused by excessively strong signals or degraded service quality caused by weak signals.

When measuring signal strength and signal quality, high signal strength values or good signal quality values cannot guarantee a fast and stable data transmission. For example, in a location where the RSSI value is high, indicating good signal strength, the communication quality of the user device may not be good due to network congestion or heavy base station load.

### RSSI

[RSSI](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength) (Received Signal Strength Indicator) refers to the total power (in dBm) of all received signals, including pilot signals, data signals, neighboring interference signals, and background noise signals. The parameter range of RSSI varies in different network modes, but a larger value indicates better signal strength.

RSSI is usually a relative value, and its measurement is highly dependent on the receiving device. Therefore, the measurement standards for RSSI may not be completely uniform for different devices. The RSSI measurement standards provided below are for reference only:

**Measurement Standards**

| RSSI (dBm)         | Signal Strength | Description                                                  |
| ------------------ | --------------- | ------------------------------------------------------------ |
| RSSI < -100        | Poor            | Very weak signal, unable to communicate normally.            |
| -100 <= RSSI < -90 | Fair            | Weak signal, having packet loss or latency.                  |
| -90 <= RSSI < -80  | Good            | Good signal, supporting most applications.                   |
| -80 <= RSSI < -70  | Very good       | Very good signal, suitable for HD video and real-time voice. |
| RSSI > -70         | Excellent       | Extremely good signal, suitable for high-speed data transmission and applications requiring high network quality. |

<br>

### CSQ

[CSQ](../../../../api_reference/iotlib/net.md#net.csqQueryPoll) (Carrier Signal Quality) refers to the signal strength, used to indicate RSSI level. Range: 0 – 31. Larger values indicate better signal strength. If the CSQ value is less than 6, the terminal may have difficulty establishing network communication. There is a corresponding relationship between CSQ and RSSI:
$$
CSQ = (RSSI + 113) / 2
$$

| CSQ  | RSSI (dBm)              |
| ---- | ----------------------- |
| 0    | -113                    |
| 1    | -111                    |
| 2~30 | -109 ~ -53              |
| 31   | >= -51                  |
| 99   | Unknown or undetectable |

<br>

### RSRP

[RSRP](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength) (Reference Signal Received Power) refers to the received power of reference signals. It is the average power of the signals received on all REs (resource elements) carrying reference signals in a symbol. It reflects the path loss intensity in the current channel and is used for cell coverage measurements, cell selection, and re-selection. Range: -140 dBm to -44 dBm. Larger values indicate better signal strength.

Please note that RSRP is a concept introduced in LTE, so it is used to measure LTE network signal strength, equivalent to RSCP in WCDMA networks. The following RSRP standards are for reference only:

| RSRP (dBm)         | Signal Strength | Description                                                  |
| ------------------ | --------------- | ------------------------------------------------------------ |
| RSRP <= -105       | 6               | Very poor coverage, unable to provide service.               |
| -105 < RSRP <= -95 | 5               | Poor coverage. Outdoor voice calls can be initiated but with low success rate and high DCR (Drop Call Rate). Basically unable to initiate service indoors. |
| -95 < RSRP <= -85  | 4               | Fair coverage. Various outdoor services can be initiated, including low-speed data services, but indoor call success rate is low and DCR is high. |
| -85 < RSRP <= -75  | 3               | Good coverage. Various outdoor services can be initiated, including medium-speed data services, and indoor services can be initiated, including low-speed data services. |
| -75 < RSRP <= -65  | 2               | Very good coverage. Various outdoor services can be initiated, including high-speed data services, and indoor services can be initiated, including medium-speed data services. |
| RSRP > -65         | 1               | Excellent coverage.                                          |

<br>

### RSRQ

[RSRQ](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength) (Reference Signal Received Quality) refers to the quality of the received reference signals. It reflects the signal-to-noise ratio and interference levels of the current channel. Range: -20 dB to -3 dB. Larger values indicate better signal strength. RSRQ is a concept introduced in LTE, so it is used to measure LTE network signal strength

RSRQ is the ratio of RSRP to RSSI, but adjusted by a coefficient since their measurements may be based on different bandwidths, i.e. RSRQ = N*RSRP/RSSI. 

The following RSRQ standards are for reference only:

| RSRQ (dB)         | Signal Quality | Description                                                  |
| ----------------- | -------------- | ------------------------------------------------------------ |
| -20 <= RSRQ < -15 | Poor           | Very poor signal quality, unable to communicate normally.    |
| -15 <= RSRQ < -10 | Fair           | Fair signal quality. Communication may be affected by interference. |
| -10 <= RSRQ < -7  | Good           | Good signal quality, providing normal communication.         |
| -7 <= RSRQ <= -3  | Excellent      | Excellent signal quality, providing fast-speed data communication. |

### RSCP

[RSCP](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength) (Receive Signal Code Power) refers to the received power of the code signals. It is a concept in UMTS networks and represents the power measured on a specific physical channel by the receiver. It is used as an indication of signal strength, handover criteria, and path loss calculation in downlink power control.  Range: -120 dBm to -25 dBm. Larger values indicate better signal strength.

The following RSCP standards are for reference only:

| RSCP (dBm)          | Signal Strength | Description                                                  |
| ------------------- | --------------- | ------------------------------------------------------------ |
| RSCP < -110         | Very poor       | Very poor signal quality, unable to maintain stable calls and data transmission. |
| -110 <= RSCP < -100 | Poor            | Poor signal, having difficulty maintaining stable calls and data transmission. |
| -100 <= RSCP < -85  | Fair            | Fair signal. Call and data quality may be affected.          |
| -85 <= RSCP < -75   | Good            | Good signal, supporting high-quality calls and data transmission. |
| RSCP >= -75         | Excellent       | Excellent signal, ensuring the highest-quality calls and data transmission. |

<br>

### SINR

[SINR](../../../../api_reference/iotlib/net.md#Get-Detailed-Signal-Strength) (Signal to Interference plus Noise Ratio) refers to the ratio of the received useful signal strength to the received interference signal strength. It is an important parameter for measuring signal quality in mobile network communication. Range: -10 dB to 40 dB.

The following SINR standards are for reference only:

| SINR (dB)      | Signal Quality | Description                                           |
| -------------- | -------------- | ----------------------------------------------------- |
| SINR < 3       | Very Poor      | Unable to establish or maintain a connection. |
| 3 <= SINR < 10 | Poor           | Unstable communication with high packet loss and low data transmission rates. |
| 11 <= SINR < 15 | Fair           | Communication is relatively stable, but data transmission rates may be limited. |
| 16 <= SINR < 25 | Good           | Communication is stable with higher data transmission rates. |
| SINR >= 25     | Excellent      | Communication is very stable with high data transmission rates. |

<br>

## Band

In mobile communication, a band refers to a frequency band, which is a specific range of frequencies in the radio spectrum. Each band consists of a certain frequency range and bandwidth. The radio spectrum is a limited and valuable resource that needs to be shared among various wireless communication services worldwide. To ensure harmonious coexistence and avoid interference between different services, the International Telecommunication Union (ITU) and telecom authorities divide the radio spectrum into multiple bands, each assigned to one or more specific services. These communication bands are usually numbered, such as Band 1, Band 2, Band 3, etc.

Different frequency bands have different transmission characteristics. For example, signals in lower frequency bands can penetrate buildings better, while signals in higher frequency bands are more suitable for transmission in open areas or line-of-sight conditions. Therefore, network operators need to balance and decide which bands to support in order to optimize network coverage and capacity. Users can refer to Part [Set and Get Band](../../../../api_reference/iotlib/net.md#Set-and-Get-Band) of the QuecPython wiki documentation to set and query the band of the module.



## Network Mode

Network mode, also called radio access technology (RAT), refers to the wireless technology for devices to connect to the network. Technologies like GSM, GPRS, WCDMA, CDMA2000 and LTE are network modes. Users can refer to Part [Network Mode and Roaming Configuration](../../../../api_reference/iotlib/net.md#Network-Mode-and-Roaming-Configuration) of the QuecPython wiki documentation to set and query the network mode of the module.



## Network Technology

Network technology refers to a set of technologies or protocols used to establish and manage connections between devices and networks. 

For example:

COMPACT is an optimization technology for GSM networks. It improves the spectrum efficiency and capacity of the network by changing the processing of control channels. Therefore, it is classified as a GSM network technology.

EMTC (Enhanced Machine-Type Communication) is an LTE technology aimed at improving network support for a large number of low-power devices. Therefore, it is classified as an LTE network technology. Users can refer to Part [Get Network Configuration Mode](../../../../api_reference/iotlib/net.md#Get-Network-Configuration-Mode) of the QuecPython wiki documentation to query the network technology of the module.



## Default Bearer

A bearer, also known as an EPS bearer, is a concept introduced in LTE. It refers to the channel through which information is transmitted in an LTE wireless network. If a bearer is analogous to a road, the information is like the vehicles traveling back and forth on the road.

In an LTE network, bearers are generally divided into default bearers and dedicated bearers.

* Default bearer: A bearer created during the initial UE attachment according to the default QoS level in the user subscription is a default bearer. This default bearer is maintained throughout the existence of the PDN connection, providing an "always-on" IP connection to the UE. It can be understood as a bearer that provides a best-effort IP connection.

* Dedicated bearer: Other EPS bearers connected to the same PDN are called dedicated bearers. Dedicated bearers are established to provide specific QoS transmission requirements. For example, a dedicated bearer for VoLTE.



## Base Station Time

Base station time usually refers to the internal clock of a wireless base station, which provides an accurate time reference for the mobile communication network. In a wireless communication system, sending and receiving data needs to be done within precise time windows. For example, data transmission needs to be synchronized with the base station's time accurately, otherwise, data loss or errors may occur. Therefore, base stations need a precise internal clock to control these time-sensitive operations.

In addition, base station time is commonly used for automatic time calibration of mobile devices, especially when devices move between different time zones. This process is achieved through Network Identity and Time Zone (NITZ). Users can refer to Part [Get Current Base Station Time](../../../../api_reference/iotlib/net.md#Get-Current-Base-Station-Time) of the QuecPython wiki documentation to query the base station time.