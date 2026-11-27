# FAQ on GNSS

### **What are the modules used by QuecPython to develop built-in GNSS functions?**

*   ASR Platform: EG915NEUAG/EC800MCNGA/EC800MCNGB/EC800MCNGC/EC800MCNGD/EG810MCNGA
*   Unisoc Platform: EC200UCNAA/EC200UCNLA/EC200UEUAA/EC200UAUAA/EG912UGLAA/EC800GCNGA
*   Qualcomm Platform: BG95/BG600L

### **What are cold start, warm start and hot start of GNSS?**

*   Cold start: Perform a cold start when the GNSS receiver is in a brand new state or has not been used for a long time. During a cold start, the receiver needs to obtain satellite ephemeris data, precise time information, and receive sufficient satellite signals to perform positioning. This process usually takes several minutes to more than 10 minutes.

*   Warm Start: A warm start is performed when the GNSS receiver restarts within a relatively short period of time. During a warm start, the receiver has already stored ephemeris data and time information from the last time it was used. Therefore, it can acquire satellite signals and perform positioning faster. This process usually takes tens of seconds to several minutes.

*   Hot start: A hot start is performed when the GNSS receiver restarts within a relatively short period of time and is near the same location. During a hot start, the receiver has stored ephemeris data, time information and position information from the last time of use, so it can acquire satellite signals and perform positioning faster. This process usually only takes a few seconds to tens of seconds.

### **How to use hot start in built-in GNSS?**

*   GNSS hot start requires the backup power function, which is currently only supported by EC800MCNGB\EC800MCNGD.

### **The problem of inaccurate GNSS positioning**

*   View satellite searching status in GNSS (visible satellites and available satellites)

*   Check the current environment, whether it is indoors or blocked by buildings

*   The position of the GNSS antenna, whether it is in half sky

*   Whether the conversion has been performed when inputting longitude and latitude in the coordinate system

### **What is the difference between visible satellites and available satellites?**

*   Visible satellites are satellites that can be seen in the sky above the receiver's location at a specific time and place
*   Available satellites refer to satellites that can provide effective signals for positioning in GNSS system

### **What are GGA and RMC and which should be used?**

*   The GGA (Global Positioning System Fix Data) sentence provides information about position, time and positioning quality. It contains data such as longitude, latitude, altitude, positioning quality indicator, and so on. The GGA sentences are usually used to display the current position and time instead of speed and heading information.

*   The RMC (Recommended Minimum Navigation Information) sentence provides information about position, speed, heading and time. It contains data such as longitude, latitude, speed, heading, date and time. The RMC sentences are commonly used in navigation applications, which will provide more comprehensive positioning information.

*   Whether the GGA or RMC is selected depends on the application scenario and requirements. If you only need to display the current location and time, the GGA statement is available; if you need more comprehensive positioning information, including speed and heading, the RMC statement will be preferred. Normally, the RMC sentence more popular because it provides more location information.

### **How does the module output raw NMEA data?**

The built-in GNSS calls quecgnss.read(size), and the log output by the QPYcom is the raw NMEA data. For details, see [quecgnss](../../api_reference/gnsslib/quecgnss.md).

### **How to interpret the meaning of each field in NMEA messages?**

The NMEA message of GNSS is a commonly used data format in transmitting positioning and time information of Global Navigation Satellite System (GNSS) receivers. The meaning of each field can be interpreted according to the NMEA protocol. The following are common NMEA message fields and annotations:

* \$GPGGA: GPS positioning message 

  * UTC Time: UTC time (hour/minute/second) when the message is generated 

  * Latitude: latitude value (In a format of ddd°mm′)

  * Latitude direction: N (Northern Hemisphere) or S (Southern Hemisphere)

  * Longitude: longitude value (In a format of dd°mm')

  * Longitude direction: E (Eastern Hemisphere) or W (Western Hemisphere)

  * Positioning quality indication: 

      0 - invalid positioning 

      1 - positioning fix 

      2 - differential positioning fix 

      6 - estimated positioning fix

  * Number of used satellites: the number of satellites used for positioning

  * HDOP: Horizontal Dilution of Precision

  * Altitude: height relative to sea level

  * Geodesic ellipsoid separation: the height difference between the geodesic ellipsoid and the WGS84 ellipsoid

  * Differential GPS data age: The age of differential GPS data (only applicable to differential positioning fix)

  * Differential base station ID: Base station ID used for differential positioning (only applicable to differential positioning fix)

  * \$GPGGA: GPS positioning information message

* GPRMC: Recommended minimum positioning message
  *   UTC Time: UTC time (hour/minute/second) when the message is generated 
  *   Positioning status: A-valid positioning, V-invalid positioning
  *   Latitude: latitude value (In a format of ddd°mm′)
  *   Latitude direction: N (Northern Hemisphere) or S (Southern Hemisphere)
  *   Longitude: longitude value (In a format of dd°mm')
  *   Longitude direction: E (Eastern Hemisphere) or W (Western Hemisphere)
  *   Ground speed: in knot (nautical miles/hour)
  *   Ground heading: heading angle (degree)
  *   UTC date: UTC date(Year/Month/Day) when the message is generated
  *   Magnetic declination direction: E (eastward) or W (westward)
  *   Mode indication: A-autonomous positioning, D-differential positioning, E-estimated positioning, N- invalid data 

* \$GPGLL: Geo-positioning message
  * Latitude: latitude value (In a format of ddd°mm′)

  * Latitude direction: N (Northern Hemisphere) or S (Southern Hemisphere)

  * Longitude: longitude value (In a format of dd°mm')

  * Longitude direction: E (Eastern Hemisphere) or W (Western Hemisphere)

  * UTC Time: UTC time (hour/minute/second) when the message is generated 

  * Positioning status: A-valid positioning, V-invalid positioning

  * Mode indication: A-autonomous positioning, D-differential positioning, E-estimated positioning, N-invalid data 

    

* \$GPGSA：GNSS DOP and active satellite message
  * Mode: M-Manual 2D/3D mode, A-Auto 2D/3D mode

  * Positioning mode

      1-No Positioning

      2-2D Positioning

      3-3D Positioning

  * PRN number: Satellite PRN number used for positioning

  * PDOP: Position Dilution of Precision

  * HDOP: Horizontal Dilution of Precision

  * VDOP: Vertical Dilution of Precision

    

* \$GPGSV: Visible satellite information message
  *   Total number of messages: Total number of messages from visible satellites

  * Ordinal number of message: the sequence number of the current message

  * Number of visible satellites: the number of visible satellites in the current message

  * PRN number: Satellite PRN number

  * Elevation angle: Satellite elevation angle (degrees)

  * Azimuth: Satellite azimuth (degrees)

  * Signal-to-noise ratio: Satellite signal-to-noise ratio (dBHz)

    

### **Whether RTD, RTK, and ntrip protocol are supported?**

No 

### **How to calculate the distance between two already-known longitude and latitude?**

If the radian of the earth is not taken into consideration, the Pythagorean theorem can be applied to calculate the straight-line distance, which is a simple question to calculate distance between x1y1 and x2y2. If the distance is more than 10km, you need to calculate the radian of the earth.

### **How to solve static drift?**

better GPS modules (better algorithm, better hardware and stronger anti-interference), better antennas, wider skies (Receive signal) and movement trace shall be used. In later stage, it is available to fit route of map via algorithm.

### **What to do if It is a failure to locate after entering tunnel when used as vehicle locator?**

When the vehicle enters the tunnel, the GPS signal will be lost immediately. At this time, the current location of the vehicle, driving speed and other information can be calculated roughly according to the coordinates of the last precise positioning as well as the data of the accelerometer, gyroscope, and electronic compass.

### **Can it be used to make personnel locator and positioning watch?**

Yes, most commonly used model for locator is EC800MCNGA/GC, and the model used for positioning watch and other wearable devices is EG810MCNGA.

### **Which maps use the GCJ02 coordinate system?**

Google Maps (China), Amap and Tencent Maps

### **Which maps can be applied to  view WGS84 positioning directly?**

Google Maps, Bing Maps and Here Maps

### **When using it in vehicle, does it need to lead the antenna to the roof of the vehicle?**

Yes, the car body is a metal cage structure. If the antenna is not lead, the signal strength will be weakened.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
