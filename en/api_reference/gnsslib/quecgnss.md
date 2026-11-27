# quecgnss - Internal GNSS Resolver

This feature provides the APIs of the built-in GNSS feature.

> QuecPython modules list that support built-in GNSS:<br />
> ECX00U AA/LA series<br />
> EG91XU AA series<br />
> EC800M GA/GB/GC/GD series<br />EG810M GA series<br />EC800G GA series<br />
> EG800AKXX_11LG/LN series<br />
> EG800ZCN_LG series<br />
> EC200AXX_V1GA series<br />
> EC200A GA series<br />
> BG95M1/M3 model

**Example**

```python
import quecgnss
import utime 

def main():
    quecgnss.configSet(0,1)#Set the location galaxy toGPS+Beidou
    quecgnss.configSet(2,1)#open AGPS
    quecgnss.configSet(4,1)#Turn on the backup power
    ret = quecgnss.init()
    if ret == 0:
        print('GNSS init ok.')
    else:
        print('GNSS init failed.')
        return -1
    for i in range(1000):
        data = quecgnss.read(4096)
        print(data[1].decode())
        utime.sleep(3)

    quecgnss.gnssEnable(0)


if __name__ == '__main__':
    main()

#===================================================================================================
#Execution Result
167,169,170,,,,,,,,1.773,1.013,1.455*15
$GPGSV,2,1,8,3,23,303,34,16,32,219,28,22,74,98,26,25,16,43,25*77
$GPGSV,2,2,8,26,70,236,28,31,59,12,38,32,55,127,34,4,5,,21*49
$BDGSV,2,1,8,163,51,192,32,166,70,11,31,167,52,197,32,169,59,334,31*61
$BDGSV,2,2,8,170,40,205,31,161,5,,31,164,5,,27,165,5,,29*59
$GNRMC,022326.000,A,3149.324624,N,11706.921702,E,0.000,261.541,180222,,E,A*38
$GNGGA,022326.000,3149.324624,N,11706.921702,E,1,12,1.013,-8.580,M,0,M,,*47
$GNGLL,3149.324624,N,11706.921702,E,022326.000,A,A*44
$GNGSA,A,3,31,32,3,16,22,25,26,,,,,,1.773,1.013,1.455*1C
$GNGSA,A,3,163,166,167,169,170,,,,,,,,1.773,1.013,1.455*15
$GPGSV,2,1,8,3,23,303,34,16,32,219,27,22,74,98,26,25,16,43,25*78
$GPGSV,2,2,8,26,70,236,28,31,59,12,37,32,55,127,34,4,5,,20*47
$BDGSV,2,1,8,163,51,192,32,166,70,11,31,167,52,197,32,169,59,334,31*61
$BDGSV,2,2,8,170,40,205,31,161,5,,31,164,5,,27,165,5,,29*59
$GNRMC,022327.000,A,3149.324611,N,11706.921713,E,0.000,261.541,180222,,E,A*3F
$GNGGA,022327.000,3149.324611,N,11706.921713,E,1,12,1.013,-8.577,M,0,M,,*48
$GNGLL,3149.324611,N,11706.921713,E,022327.000,A,A*43
...... # Omit some data due to large data quantity.
$GNGSA,A,3,31,32,3,16,22,25,26,,,,,,1.837,1.120,1.456*11
$GNGSA,A,3,163,166,167,169,170,,,,,,,,1.837,1.120,1.456*18
$GPGSV,2,1,8,3,23,302,27,16,32,220,26,22,73,101,27,25,16,43,27*45
$GPGSV,2,2,8,26,70,237,28,31,59,13,33,32,54,128,28,4,5,,24*44
$BDGSV,2,1,8,163,51,192,33,166,71,11,35,167,52,198,33,169,59,334,34*6E
$BDGSV,2,2,8,170,40,205,32,161,5,,33,164,5,,28,165,5,,30*5F
$GNRMC,022507.000,A,3149.324768,N,11706.922344,E,0.000,261.541,180222,,E,A*31
$GNGGA,022507.000,3149.324768,N,11706.922344,E,1,12,1.120,-8.794,M,0,M,,*48
$GNGLL,3149.324768,N,11706.922344,E,022507.000,A,A*4D
$GNGSA,A,3,31,32,3,16,22,25,26,,,,,,1.837,1.120,1.455*12
$GNGSA,A,3,163,166,167,169,170,,,,,,,,1.837,1.120,1.455*1B
$GPGSV,2,1,8,3,23,302,26,16,32,220,26,22,73,101,27,25,16,43,26*45
$GPGSV,2,2,8,26,70,237,28,31,59,13,32,32,54,128,28,4,5,,24*45
$BDGSV,2,1,8,163,51,192,24,166,71,11,35,167,52,198,33,169,59,334,34*68
$BDGSV,2,2,8,170,40,205,31,161,5,,33,164,5,,28,165,5,,30*5C
$GNRMC,022508.000,A,3149.324754,N,11706.922338,E,0.002,261.541,180222,,E,A*38
$GNGGA,022508.000,3149.324754,N,11706.922338,E,1,12,1.120,-8.750,M,0,M,,*4B
$GNGLL,3149.324754,N,11706.922338,E,022508.000,A,A*46
$GNGSA,A,3,31,3
```

##  Overview of GNSS Positioning Data

```python
NMEA Port Data Classification:：
                $GPGGA Global Positioning System Fix Data
                $GPGSA GNSS DOP and Active Satellites 
                $GPGSV GNSS Satellites in View
                $GPRMC Recommended Minimum Specific GNSS Data
                $GPVTG Course Over Ground and Ground Speed 
                $GPDTM Datum Reference
                $GPGNS GNSS Fix Data
NMEA Sentence Parsing：
The basic format of the GSV sentence is as follows:

                $GPGSV,(1),(2),(3),(4),(5),(6),(7),...,(4),(5),(6),(7)*hh(CR)(LF)

                Field 1: Total number of GSV sentences.
            
                Field 2: Sequence number of the current GSV sentence.

                Field 3: Total number of satellites in view (00–12, leading zeros are transmitted).

                Field 4: Satellite PRN number (01–32, leading zeros are transmitted).

                Field 5: Elevation of the satellite (00–90 degrees, leading zeros are transmitted).

                Field 6: Azimuth of the satellite (000–359 degrees, leading zeros are transmitted).

                Field 7: Signal-to-noise ratio (00–99 dB; left blank if the satellite is not being tracked) — also referred to as the C/N0 value.

The basic format of the GGA sentence is as follows:

                $GPGGA,(1),(2),(3),(4),(5),(6),(7),(8),(9),(10),(11),(12)*hh(CR)(LF)

                Field 1: UTC time, hhmmss.sss (hours, minutes, seconds format).

                Field 2: Latitude ddmm.mmmm (degrees and minutes format, leading zeros transmitted).

                Field 3: Latitude hemisphere — N (North) or S (South).

                Field 4: Longitude dddmm.mmmm (degrees and minutes format, leading zeros transmitted).

                Field 5: Longitude hemisphere — E (East) or W (West).

                Field 6: GPS status — 0 = Invalid, 1 = GPS fix, 2 = DGPS fix, 3 = Invalid PPS, 6 = Estimated.

                Field 7: Number of satellites used (00–12, leading zeros transmitted).

                Field 8: HDOP (Horizontal Dilution of Precision), range 0.5–99.9.

                Field 9: Altitude (-9999.9 to 99999.9).

                Field 10: Altitude unit, meters.

                Field 11: Height of geoid (ellipsoid relative to mean sea level).

                Field 12: Height unit of geoid separation, meters.

The basic format of the GSA sentence is as follows:

                $GPGSA,(1),(2),(3),(3),,,,,,,,,,(3),(4),(5),(6),(7)*hh(CR)(LF)

                Field 1: Mode — A = Automatic 2D/3D, M = Manual 2D/3D.

                Field 2: Fix type — 1 = No fix, 2 = 2D fix, 3 = 3D fix.

                Field 3: PRN codes — Satellite PRN number(s) being used in the current fix (00; leading zeros transmitted, up to 12 satellites).

                Field 4: PDOP (Position Dilution of Precision), range 0.0–500.0.

                Field 5: HDOP (Horizontal Dilution of Precision), range 0.0–500.0.

                Field 6: VDOP (Vertical Dilution of Precision), range 0.0–500.0.

                Field 7: Satellite system ID.

The basic format of the VTG sentence is as follows:

                $GPVTG,(1),(2),(3),(4),(5),(6),(7),(8),(9)*hh(CR)(LF)

                Field 1: Course over ground (000–359, leading zeros transmitted).

                Field 2: Reference — T = True North.

                Field 3: Course over ground (000–359, leading zeros transmitted).

                Field 4: Reference — M = Magnetic North.

                Field 5: Horizontal speed (0.00, leading zeros transmitted).

                Field 6: Unit — N = Knots.

                Field 7: Horizontal speed (0.00, leading zeros transmitted).

                Field 8: Unit — K = Kilometers per hour (km/h).

                Field 9: Mode indicator — E = Estimated, A = Autonomous (non-DGPS).

The basic format of the RMC sentence is as follows:
        
                $GPRMC,(1),(2),(3),(4),(5),(6),(7),(8),(9),(10),(11),(12),(13)*hh(CR)(LF)

                Field 1: UTC time, hhmmss.sss format.

                Field 2: Status — A = Valid (fix), V = Invalid (no fix).

                Field 3: Latitude ddmm.mmmm (degrees and minutes format, leading zeros transmitted).

                Field 4: Latitude hemisphere — N (North) or S (South).

                Field 5: Longitude dddmm.mmmm (degrees and minutes format, leading zeros transmitted).

                Field 6: Longitude hemisphere — E (East) or W (West).

                Field 7: Speed over ground, in knots.

                Field 8: Course over ground, in degrees.

                Field 9: UTC date, DDMMYY format.

                Field 10: Magnetic variation (000–180 degrees, leading zeros transmitted).

                Field 11: Magnetic variation direction — E (East) or W (West).

                Field 12: Mode indicator — E = Estimated, A = Autonomous (non-DGPS).

                Field 13: Navigation status — V = Navigation status not provided.

The basic format of the DTM sentence is as follows:

                 $GNDTM,<1>,<2>,<3>,<4>,<5>,<6>,<7>,<8>*<9>(CR)(LF)

                <1>Local datum code — W84, P90.

                <2>Datum subcode — blank.

                <3>Latitude offset.

                <4>Latitude hemisphere — N (North) or S (South).

                <5>Longitude offset.

                <6>Longitude hemisphere — E (East) or W (West).

                <7>Altitude offset.

                <8>Reference datum code — W84.

                <9>Checksum.

The basic format of the GNS sentence is as follows:
        
                $GNGNS,<1>,<2>,<3>,<4>,<5>,<6>,<7>,<8>.<9>,<10>,<11>,<12>,<13>,*<14>(CR)(LF)

                <1> UTC time: Fix time hhmmss.ss — 000000.00 ~ 235959.99
                
                <2> Latitude: ddmm.mmmmm — 0000.00000 ~ 8959.9999
                
                <3> Latitude hemisphere: N = North, S = South
                
                <4> Longitude: dddmm.mmmmm — 0000.00000 ~ 17959.99999
                
                <5> Longitude hemisphere: E = East, W = West
                
                <6> Mode indicator: N = No fix; A = Autonomous; D = Differential; P = Precise; R = RTK Fixed; F = RTK Float; E = Estimated; M = Manual input; S = Simulator
                
                <7> Satellites used: Number of satellites used in fix, 00–99
                
                <8> HDOP: Horizontal Dilution of Precision, 0.5–99.9
                
                <9> Altitude: Unit = meters
                
                <10> Geoid separation: Height of geoid relative to WGS84 ellipsoid
                
                <11> Differential data age: blank if prefix is “GN”
                
                <12> Reference station ID: blank if prefix is “GN”
                
                <13> Navigation status: C = Caution, S = Safe, U = Unsafe, V = Invalid
                
                <14> Checksum
```



## Initialize GNSS 

### **`quecgnss.init`**

```python
quecgnss.init()
```

Initializes the built-in GNSS feature.

**Return Value**
`0` - Successful execution

`-1` - Failed execution

## Get GNSS Working Status

### **`quecgnss.get_state`**

```python
quecgnss.get_state()
```

Gets the current working status of the built-in GNSS feature.

**Return Value**

| Value | Type | Description                                                  |
| ----- | ---- | ------------------------------------------------------------ |
| 0     | int  | GNSS feature is disabled.                                    |
| 1     | int  | GNSS firmware is being updated.                              |
| 2     | int  | GNSS is positioning. In this mode, the module can read the GNSS location data. After obtaining the location data, the user needs to analyze the corresponding sentence to determine whether the location data is effective. For example, if the status of GNRMC sentences is A or V, A indicates valid positioning and V indicates invalid positioning. |

## Enable or Disable GNSS

### `quecgnss.gnssEnable`

```python
quecgnss.gnssEnable(opt)
```

Enables or disables GNSS feature. If you use the built-in GNSS feature for the first time after powering the module on, you need not call this function to enable GNSS feature, but call *quecgnss.init()* directly. *quecgnss.init()* will automatically enable the GNSS feature when GNSS feature is initialized.

**Parameter**

`opt` - Integer type. Enable or disable GNSS.

 `0` - Disable the GNSS feature.

 `1` - Enable the GNSS feature.

**Return Value**

 `0` - Successful execution

 `-1` - Failed execution

## Get GNSS Location Data

### `quecgnss.read`

```python
quecgnss.read(size)
```

Gets GNSS location data.

**Parameter**

`size` - Integer type. Size of the data to be read. Unit: byte.

**Return Value**

A tuple `(size, data)`  - Successful execution

`size` - Size of the data read.
`data` - GNSS location data.

-1 - Failed execution

## GNSS/LTE Priority Settings

> Currently, this function is only supported by BG95M1/M3 models.

### `quecgnss.setPriority`

```python
quecgnss.setPriority(Priority)
```

On some modules, GNSS and LTE share RF resources. This interface is used to control the priority of RF resource usage between GNSS and LTE.

**Parameter**

`Priority`， integer type, sets the priority value between GNSS and LTE.

Correspondence of priority levels for GNSS/LTE on different platforms:

| Priority | BG95M1/M3   |
| ------ | -------- |
| 0      | GNSS>LTE |
| 1      | LTE>GNSS |

**Return Value**

 `0` - Successful execution

 `-1` - Failed execution

## Get GNSS/LTE Priority

> Currently, this function is only supported by BG95M1/M3 models.

### `quecgnss.getPriority`

```python
quecgnss.getPriority()
```

On some modules, GNSS and LTE share RF resources. This interface is used to read the priority of RF resource usage between GNSS and LTE.

**Return Value**

Returns an integer representing the GNSS/LTE priority.

Correspondence of GNSS/LTE priority across different platforms:

| Priority | BG95M1/M3   |
| ------ | -------- |
| 0      | GNSS>LTE |
| 1      | LTE>GNSS |

## Built-in GNSS Configuration Parameter Settings

> Currently, this function is supported only by the EC800M GA/GB/GC/GD series,<br> EG810M GA series, ECX00U AA/LA series,EG800AKXX_11LG/LN series,EG800ZCN_LG series,<br> EC200AXX_V1GA series and EC200A GA series.


### `quecgnss.configSet`

```python
quecgnss.configSet(config_type, config_value)
```

Sets GNSS parameters, including satellite system, NMEA sentence types, whether to use AGNSS, APFLASH, etc.

**Parameter Description**

* `config_type`，int，The types of GNSS configuration options to be set are as follows:

| Value | Description        |
| ---- | ---------------------- |
| 0    | Configure the satellite system to be used |
| 1    | Configure the types of NMEA sentences to output |
| 2    | Configure whether to enable AGPS |
| 3    | Configure whether to enable APFLASH |
| 4    | Configure whether to enable GNSS backup power |

* `config_value`，int，The values of GNSS configuration options and their meanings are as follows:

Satellite System（config_type = 0）：

| Value | Description             |
| ----- | ----------------------- |
| 0     | GPS                     |
| 1     | GPS + BeiDou            |
| 3     | GPS + GLONASS + Galileo |
| 4     | GPS + GLONASS           |
| 5     | GPS + BeiDou + Galileo  |
| 6     | GPS + Galileo           |
| 7     | BeiDou                  |

EG800AKXX_11LG/LN EG800ZCN_LG Series Satellite System(EG800AKCN_11LG only supports BeiDou, and does not support other constellations.)（config_type = 0）：

NOTICE:The selection of the satellite system is related to the GNSS hardware.
The GNSS hardware within different modules comes from different manufacturers,
and the satellite systems supported by the GNSS hardware from different manufacturers are also different.

| Value | Description                      |
| ----- | ---------------------------------|
| 0    | GPS                               |
| 1    | GPS + BeiDou                      |
| 2    | GPS + BeiDou + Galileo + GLONASS  |
| 3    | GPS + GLONASS + Galileo           |
| 4    | GPS + GLONASS                     |
| 5    | GPS + BeiDou + Galileo            |
| 6    | GPS + Galileo                     |
| 7    | BeiDou                            |
| 8    | GPS + BeiDou + GLONASS            |

NMEA Sentence Types (config_type = 1): This field is represented as a bitmap. Options to be enabled should be set to 1.
For example, to output GGA + RMC + GSV + GSA + VTG, the bitmap should be `011111`, which corresponds to a decimal value of 31.

| Value | Description           |
| ----- | --------------------- |
| 0     | Disable all (000000). |
| 1     | GGA(000001)           |
| 2     | RMC(000010)           |
| 4     | GSV(000100)           |
| 8     | GSA(001000)           |
| 16    | VTG(010000)           |
| 32    | GLL(100000)           |
| 63    | All outputs (111111). |

Enable AGPS (config_type = 2): When enabled, GNSS will download ephemeris data from the network and attempt to use it at startup to accelerate positioning.

| Value | Description |
| ----- | ----------- |
| 0     | Disable     |
| 1     | Enable      |

Enable APFLASH (config_type = 3): When enabled, GNSS will store ephemeris data on the Quecpython module and attempt to use it at startup to accelerate positioning.

| Value | Description |
| ----- | ----------- |
| 0     | Disable     |
| 1     | Enable      |

Enable Backup Power (config_type = 4): When enabled and backup power is properly connected, the GNSS chip can autonomously store ephemeris data.

>Backup power is supported on the EC800M GB/GD series EG800AKXX_11LG/LN series.

| Value | Description |
| ---- | ----------- |
| 0    | Disable |
| 1    | Enable |

**Return Value**

int: 0 indicates success; any other value indicates failure.

## Read Built-in GNSS Configuration Parameters

> Currently, this function is supported only by the EC800M GA/GB/GC/GD series,<br> EG810M GA series, ECX00U AA/LA series,EG800AKXX_11LG/LN series,EG800ZCN_LG series,<br> EC200AXX_V1GA series and EC200A GA series.
### `quecgnss.configGet`

```python
quecgnss.configGet(config_type)
```

Reads GNSS parameters, including satellite system, NMEA sentence types, whether AGNSS is used, APFLASH, etc.

**Parameter Description**

* `config_type`，int，The types of GNSS configuration options to be read are as follows：

| Value | Description        |
| ---- | ---------------------- |
| 0    | Configure the satellite system to be used |
| 1    | Configure the types of NMEA sentences to output |
| 2    | Configure whether to enable AGPS |
| 3    | Configure whether to enable APFLASH |
| 4    | Configure whether to enable GNSS backup power |

**Return Value**

int，Meanings are as follows (–1 indicates failure).

Satellite System（config_type = 0）：

| Value | Description             |
| ----- | ----------------------- |
| 0     | GPS                     |
| 1     | GPS + BeiDou            |
| 3     | GPS + GLONASS + Galileo |
| 4     | GPS + GLONASS           |
| 5     | GPS + BeiDou + Galileo  |
| 6     | GPS + Galileo           |
| 7     | BeiDou                  |


EG800AKXX_11LG/LN EG800ZCN_LG Series Satellite System(EG800AKCN_11LG only supports BeiDou, and does not support other constellations.)（config_type = 0）：

NOTICE:The selection of the satellite system is related to the GNSS hardware.
The GNSS hardware within different modules comes from different manufacturers,
and the satellite systems supported by the GNSS hardware from different manufacturers are also different.

| Value | Description                      |
| ----- | ---------------------------------|
| 0    | GPS                               |
| 1    | GPS + BeiDou                      |
| 2    | GPS + BeiDou + Galileo + GLONASS  |
| 3    | GPS + GLONASS + Galileo           |
| 4    | GPS + GLONASS                     |
| 5    | GPS + BeiDou + Galileo            |
| 6    | GPS + Galileo                     |
| 7    | BeiDou                            |
| 8    | GPS + BeiDou + GLONASS            |

NMEA Sentence Types (config_type = 1): This field is represented as a bitmap. Options that are enabled are set to 1.
For example, if GGA + RMC + GSV + GSA + VTG are output, the returned value is 31, which corresponds to the bitmap 011111 in binary.

| Value | Description           |
| ----- | --------------------- |
| 0     | Disable all (000000). |
| 1     | GGA(000001)           |
| 2     | RMC(000010)           |
| 4     | GSV(000100)           |
| 8     | GSA(001000)           |
| 16    | VTG(010000)           |
| 32    | GLL(100000)           |
| 63    | All outputs (111111). |

Enable AGPS (config_type = 2): When enabled, GNSS will download ephemeris data from the network and attempt to use it at startup to accelerate positioning.

| Value | Description |
| ----- | ----------- |
| 0     | Disable     |
| 1     | Enable      |

Enable APFLASH (config_type = 3): When enabled, GNSS will store ephemeris data on the Quecpython module and attempt to use it at startup to accelerate positioning.

| Value | Description |
| ----- | ----------- |
| 0     | Disable     |
| 1     | Enable      |

Enable Backup Power (config_type = 4): When enabled and backup power is properly connected, the GNSS chip can autonomously store ephemeris data.

>Backup power is supported on the EC800M GB/GD series EG800AKXX_11LG/LN series.

| Value | Description |
| ---- | ----------- |
| 0    | Disable |
| 1    | Enable |