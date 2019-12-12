# Carelink Device MQTT Specifications

**This document aims to define the specifications of the MQTT control packets
("connect", "publish" and "subscribe"), to be used by the Carelink devices.**

NOTE: The specifications are meant to be device agnostic and thus some
considerations need to be observed, specifically in the standards and
nomenclature used, as well as in the interoperability of some variables and
features across different devices.

*Author*: Jorge Calado (jsc\@uninova.pt)

*Document version*: 3.2.0 – 2019-12-12

----

## Table of Contents

- [Carelink Device MQTT Specifications](#carelink-device-mqtt-specifications)
  - [Standards and nomenclature used](#standards-and-nomenclature-used)
  - [Common Variables](#common-variables)
- [Connect Packet](#connect-packet)
- [MQTT Pub-Sub Topics Dendrogram](#mqtt-pub-sub-topics-dendrogram)
- [MQTT Messages Sequence Diagrams](#mqtt-messages-sequence-diagrams)
  - [Device power-up](#device-power-up)
  - [Normal usage](#normal-usage)
  - [Fall/Wandering](#fallwandering)
- [Publish Topics](#publish-topics)
  - ["[deviceId]/status"](#deviceidstatus)
  - ["[deviceId]/statusWifiAPs"](#deviceidstatuswifiaps)
  - ["[deviceId]/fall/confirmation"](#deviceidfallconfirmation)
  - ["[deviceId]/fall/detected"](#deviceidfalldetected)
  - ["[deviceId]/wandering/confirmation"](#deviceidwanderingconfirmation)
  - ["[deviceId]/wandering/detected"](#deviceidwanderingdetected)
- [Subscribe Topics](#subscribe-topics)
  - ["[deviceId]/active"](#deviceidactive)
  - ["[deviceId]/configuration/energyProfile"](#deviceidconfigurationenergyprofile)
  - ["[deviceId]/configuration/wifi"](#deviceidconfigurationwifi)
  - ["[deviceId]/configuration/lteNB"](#deviceidconfigurationltenb)
  - ["[deviceId]/configuration/notifications"](#deviceidconfigurationnotifications)
  - ["[deviceId]/wandering/notification"](#deviceidwanderingnotification)
  - ["[deviceId]/zones/home"](#deviceidzoneshome)
  - ["[deviceId]/zones/regular"](#deviceidzonesregular)
  - ["[deviceId]/zones/dangerous"](#deviceidzonesdangerous)
  - [TO-DO:](#to-do)
    - ["[deviceId]/configuration/gsm"](#deviceidconfigurationgsm)
    - ["[deviceId]/configuration/bluetooth"](#deviceidconfigurationbluetooth)
    - ["[deviceId]/configuration/lora"](#deviceidconfigurationlora)
    - ["[deviceId]/configuration/gnss"](#deviceidconfigurationgnss)
    - ["[deviceId]/configuration/accelerometer"](#deviceidconfigurationaccelerometer)

----

Standards and nomenclature used
-------------------------------

>   DateTime timestamps using ISO8601 UTC extended format (e.g.
>   YYYY-MM-DDThh:mm:ssZ).

>   Location coordinates using the format WGS 84 (in Decimal Degrees).

>   Json notation for payload messages.

>   Geojson feature collection for definition of geographical areas.

Common Variables
----------------

| *Name*     | *Type* | *Description*                                        | *Required* |
|------------|--------|------------------------------------------------------|------------|
| [deviceId] | string | unique alphanumeric string for device identification | true       |

NOTICE: A user can have multiple devices. A device can only be associated with
one user at a time.

Connect Packet
==============

**Connect Flags:**

| Clean Session | false |
|---------------|-------|
| Will Qos      | 1     |
| Will Flag     | true  |
| Will Retain   | false |
| Keep Alive    | 600   |

**Connect Payload:**

| Client Identifier | unique random mqtt identifier  |
|-------------------|--------------------------------|
| Will Topic        | "[deviceId]/status"            |
| Will Message      | "LW: Unexpected disconnection" |
| Username          | "deviceId"                     |
| Password          | "mqtt password"                |

MQTT Pub-Sub Topics Dendrogram
==============================

![MQTT Pub-Sub Topics Dendrogram](Pub-SubTopicsDendrogram.png)

MQTT Messages Sequence Diagrams
===============================

### Device power-up

![MQTT Messages Sequence Diagrams Device power-up](DevicePower-up.png)

### Normal usage

![MQTT Messages Sequence Diagrams Normal usage](NormalUsage.png)

### Fall/Wandering

(For supported devices only, i.e. Smartphones)

![MQTT Messages Sequence Diagrams Fall-Wandering](FallWandering.png)

Publish Topics
==============

>   *"[deviceId]/status"*

>   *"[deviceId]/statusWifiAPs"* **\***

>   *"[deviceId]/fall/confirmation"* **\***

>   *"[deviceId]/fall/detected"* **\***

>   *"[deviceId]/wandering/confirmation"* **\***

>   *"[deviceId]/wandering/detected"* **\***

**\*** *For supported devices only*

"[deviceId]/status"
--------------------

| QoS    | 0     |
|--------|-------|
| Retain | false |

**(Single Status) Payload (using compressed JsonObject):**

```json
{"timestamp":"YYYY-MM-DDThh:mm:ssZ","location":{"lat":float,"lon":float,"alt":float,"hdop":float,"vdop":float,"pdop":float,"type":string,"accuracy":integer},"batteryLevel":integer,"accompanied":boolean,"sensor":{"accelerometer":{"x":float,"y":float,"z":float}}}
```

**Pretty Json:**

```json
{
    "timestamp": "YYYY-MM-DDThh:mm:ssZ",
    "location": {
        "lat": float,
        "lon": float,
        "alt": float,
        "hdop": float,
        "vdop": float,
        "pdop": float,
        "type": string,
        "accuracy": integer
    },
    "batteryLevel": integer,
    "accompanied": boolean,
    "sensor": {
        "accelerometer": {
            "x": float,
            "y": float,
            "z": float
        }
    }
}
```

| *Name*        | *Type*      | *Description*                                                                             | *Required* |
|---------------|-------------|-------------------------------------------------------------------------------------------|------------|
| timestamp     | string      | timestamp with the dateTime when data was recorded                                        | true       |
| location      | json object | json object with latitude, longitude, altitude coordinates                                | true       |
| lat           | float       | latitude coordinates                                                                      | true       |
| lon           | float       | longitude coordinates                                                                     | true       |
| alt           | float       | altitude coordinates                                                                      | false      |
| hdop          | float       | horizontal dilution of precision                                                          | false      |
| vdop          | float       | vertical dilution of precision                                                            | false      |
| pdop          | float       | positional dilution of precision                                                          | false      |
| type          | string      | origin of gps coordenates; can be: "gps" (default), "here", "opencellid", "google", "lora"| false      |
| accuracy      | integer     | positional dilution of precision                                                          | false      |
| batteryLevel  | integer     | value of battery level in mAh                                                             | true       |
| accompanied   | boolean     | indicator if device is paired with carer smartphone                                       | false      |
| sensor        | json object | json object with the sensors available on the device                                      | false      |
| accelerometer | json object | x, y, z axis acceleration values                                                          | false      |
| x             | float       | x axis acceleration value                                                                 | true*      |
| y             | float       | y axis acceleration value                                                                 | true*      |
| z             | float       | z axis acceleration value                                                                 | true*      |

"[deviceId]/statusWifiAPs"
--------------------

| QoS    | 0     |
|--------|-------|
| Retain | false |

**(Single Status) Payload (using compressed JsonObject):**

```json
{"timestamp":"YYYY-MM-DDThh:mm:ssZ","location":{"lat":float,"lon":float,"alt":float,"hdop":float,"vdop":float,"pdop":float},"batteryLevel":integer,"accompanied":boolean,"sensor":{"accelerometer":{"x":float,"y":float,"z":float}},"wifiAPs":{"mac_1":string,"rssi_1":integer,"mac_2":string,"rssi_2":integer,"mac_3":string,"rssi_3":integer}}
```

**Pretty Json:**

```json
{
    "timestamp": "YYYY-MM-DDThh:mm:ssZ",
    "location": {
        "lat": float,
        "lon": float,
        "alt": float,
        "hdop": float,
        "vdop": float,
        "pdop": float
    },
    "batteryLevel": integer,
    "accompanied": boolean,
    "sensor": {
        "accelerometer": {
            "x": float,
            "y": float,
            "z": float
        }
    },
    "wifiAPs": {
        "mac_1": string,
        "rssi_1": integer,
        "mac_2": string,
        "rssi_2": integer,
        "mac_3": string,
        "rssi_3": integer
    }
}
```

| *Name*        | *Type*      | *Description*                                              | *Required* |
|---------------|-------------|------------------------------------------------------------|------------|
| timestamp     | string      | timestamp with the dateTime when data was recorded         | true       |
| location      | json object | json object with latitude, longitude, altitude coordinates | true       |
| lat           | float       | latitude coordinates                                       | true       |
| lon           | float       | longitude coordinates                                      | true       |
| alt           | float       | altitude coordinates                                       | false      |
| hdop          | float       | horizontal dilution of precision                           | false      |
| vdop          | float       | vertical dilution of precision                             | false      |
| pdop          | float       | positional dilution of precision                           | false      |
| batteryLevel  | integer     | value of battery level in mAh                              | true       |
| accompanied   | boolean     | indicator if device is paired with carer smartphone        | false      |
| sensor        | json object | json object with the sensors available on the device       | false      |
| accelerometer | json object | x, y, z axis acceleration values                           | false      |
| x             | float       | x axis acceleration value                                  | true*      |
| y             | float       | y axis acceleration value                                  | true*      |
| z             | float       | z axis acceleration value                                  | true*      |
| wifiAPs       | json object | json object of the 3 AP's with the strongest signal        | true       |
| mac_1         | string      | bssid of the AP                                            | true       |
| mac_2         | string      | bssid of the AP                                            | true       |
| mac_3         | string      | bssid of the AP                                            | true       |
| rssi_1        | integer     | rssi value of the AP                                       | true       |
| rssi_2        | integer     | rssi value of the AP                                       | true       |
| rssi_3        | integer     | rssi value of the AP                                       | true       |

"[deviceId]/fall/confirmation"
-------------------------------

| QoS    | 2     |
|--------|-------|
| Retain | false |

**Payload (compressed JsonObject):**  
```json
{"status":boolean,"timestamp":"YYYY-MM-DDThh:mm:ssZ","location":{"lat":float,"lon":float,"alt":float},"batteryLevel":integer,"sensor":{"accelerometer":"x,y,z","heartrate":integer,"temperature":integer}}
```

| *Name*        | *Type*      | *Description*                                              | *Required* |
|---------------|-------------|------------------------------------------------------------|------------|
| status        | boolean     | indicator if fall confirmation was positive or negative    | true       |
| timestamp     | string      | timestamp with the dateTime when data was recorded         | true       |
| location      | json object | json object with latitude, longitude, altitude coordinates | true       |
| lat           | float       | latitude coordinates                                       | true       |
| lon           | float       | longitude coordinates                                      | true       |
| alt           | float       | altitude coordinates                                       | true       |
| batteryLevel  | integer     | value of battery level in mAh                              | true       |
| sensor        | json object | json object with the sensors available on the device       | false      |
| accelerometer | string      | x, y, z axis acceleration values                           | false      |
| heartrate     | integer     | heart rate value in bpm                                    | false      |
| temperature   | integer     | Temperature value in degree Celsius                        | false      |

"[deviceId]/fall/detected"
---------------------------

| QoS    | 2     |
|--------|-------|
| Retain | false |

**Payload (compressed JsonObject):**  
```json
{"timestamp":"YYYY-MM-DDThh:mm:ssZ","location":{"lat":float,"lon":float,"alt":float},"batteryLevel":integer,"sensor":{"accelerometer":"x,y,z","heartrate":integer,"temperature":integer}}
```

| *Name*        | *Type*      | *Description*                                              | *Required* |
|---------------|-------------|------------------------------------------------------------|------------|
| timestamp     | string      | timestamp with the dateTime when data was recorded         | true       |
| location      | json object | json object with latitude, longitude, altitude coordinates | true       |
| lat           | float       | latitude coordinates                                       | true       |
| lon           | float       | longitude coordinates                                      | true       |
| alt           | float       | altitude coordinates                                       | true       |
| batteryLevel  | integer     | value of battery level in mAh                              | true       |
| sensor        | json object | json object with the sensors available on the device       | false      |
| accelerometer | string      | x, y, z axis acceleration values                           | false      |
| heartrate     | integer     | heart rate value in bpm                                    | false      |
| temperature   | integer     | Temperature value in degree Celsius                        | false      |

"[deviceId]/wandering/confirmation"
------------------------------------

| QoS    | 2     |
|--------|-------|
| Retain | false |

**Payload (compressed JsonObject):**  
```json
{"status":boolean,"timestamp":"YYYY-MM-DDThh:mm:ssZ","location":{"lat":float,"lon":float,"alt":float},"batteryLevel":integer,"sensor":{"accelerometer":"x,y,z","heartrate":integer,"temperature":integer}}
```

| *Name*        | *Type*      | *Description*                                                | *Required* |
|---------------|-------------|--------------------------------------------------------------|------------|
| status        | boolean     | indicator if wandering confirmation was positive or negative | true       |
| timestamp     | string      | timestamp with the dateTime when data was recorded           | true       |
| location      | json object | json object with latitude, longitude, altitude coordinates   | true       |
| lat           | float       | latitude coordinates                                         | true       |
| lon           | float       | longitude coordinates                                        | true       |
| alt           | float       | altitude coordinates                                         | true       |
| batteryLevel  | integer     | value of battery level in mAh                                | true       |
| sensor        | json object | json object with the sensors available on the device         | false      |
| accelerometer | string      | x, y, z axis acceleration values                             | false      |
| heartrate     | integer     | heart rate value in bpm                                      | false      |
| temperature   | integer     | Temperature value in degree Celsius                          | false      |

"[deviceId]/wandering/detected"
--------------------------------

| QoS    | 2     |
|--------|-------|
| Retain | false |

**Payload (compressed JsonObject):**  
```json
{"timestamp":"YYYY-MM-DDThh:mm:ssZ","location":{"lat":float,"lon":float,"alt":float},"batteryLevel":integer,"sensor":{"accelerometer":"x,y,z","heartrate":integer,"temperature":integer}}
```

| *Name*        | *Type*      | *Description*                                              | *Required* |
|---------------|-------------|------------------------------------------------------------|------------|
| timestamp     | string      | timestamp with the dateTime when data was recorded         | true       |
| location      | json object | json object with latitude, longitude, altitude coordinates | true       |
| lat           | float       | latitude coordinates                                       | true       |
| lon           | float       | longitude coordinates                                      | true       |
| alt           | float       | altitude coordinates                                       | true       |
| batteryLevel  | integer     | value of battery level in mAh                              | true       |
| sensor        | json object | json object with the sensors available on the device       | false      |
| accelerometer | string      | x, y, z axis acceleration values                           | false      |
| heartrate     | integer     | heart rate value in bpm                                    | false      |
| temperature   | integer     | Temperature value in degree Celsius                        | false      |

Subscribe Topics
================

>   *"[deviceId]/active"*

>   *"[deviceId]/configuration/energyProfile"*

>   *"[deviceId]/configuration/[component]"*

>   *"[deviceId]/configuration/notifications"* **\***

>   *"[deviceId]/wandering/notification"* **\***

>   *"[deviceId]/zones/home"* **\***

>   *"[deviceId]/zones/regular"* **\***

>   *"[deviceId]/zones/dangerous"* **\***

**\*** *For supported devices only*

"[deviceId]/active"
--------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**  
```json
bool
```

| *Name* | *Type*  | *Description*                                                                                                       | *Required* |
|--------|---------|---------------------------------------------------------------------------------------------------------------------|------------|
| bool   | boolean | bool==true indicates the device can start normal activity bool==false indicates the device to cease normal activity | true       |

"[deviceId]/configuration/energyProfile"
-----------------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**  
```json
{"gnss":{"active":boolean,"sr":integer},"lteNB":{"active":boolean,"sr":integer},"wifi":{"active":boolean,"sr":integer},"lora":{"active":boolean,"sr":integer}}
```

| *Name* | *Type*  | *Description*                               | *Required* |
|--------|---------|---------------------------------------------|------------|
| active | boolean | Indicator if component is powered on or off | true       |
| sr     | integer | sampling rate (in seconds)                  | true       |

"[deviceId]/configuration/wifi"
-------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**  
```json
{"ssid":"wifi name","wlanpw":"wifi password"}
```

| *Name* | *Type* | *Description*                                   | *Required* |
|--------|--------|-------------------------------------------------|------------|
| ssid   | string | wifi service set identifier of PwD home network | true       |
| wlanpw | string | wifi password                                   | true       |

"[deviceId]/configuration/lteNB"
--------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**  
```json
{"apn":"telecom apn","band":integer}
```

| *Name* | *Type*  | *Description*                | *Required* |
|--------|---------|------------------------------|------------|
| apn    | string  | LTE-NB access point name     | true       |
| band   | integer | LTE-NB frequency band number | true       |

"[deviceId]/configuration/notifications"
-----------------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**  
```json
{"frequency":integer,"persistence":integer,"volume":integer}
```

| *Name*      | *Type*  | *Description*                                          | *Required* |
|-------------|---------|--------------------------------------------------------|------------|
| frequency   | integer | frequency of the notifications displayed on the device | true       |
| persistence | integer | persistence display time of the notifications          | true       |
| volume      | integer | volume of the audio notifications                      | true       |

"[deviceId]/wandering/notification"
------------------------------------

| QoS    | 2     |
|--------|-------|
| Retain | false |

**Payload (compressed JsonObject):**  
```json
{"notification":"Wandering detected! Device: [deviceId];"}
```

| *Name*       | *Type* | *Description*                                                                   | *Required* |
|--------------|--------|---------------------------------------------------------------------------------|------------|
| notification | string | notification of detected wandering event, of user wearing the device [deviceId] | true       |

"[deviceId]/zones/home"
------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed GeoJson Feature Collection):**
```json
{"type":"FeatureCollection","features":[{"type":"Feature","properties":{"id":"home"},"geometry":{"type":"Polygon","coordinates":[[[lon,lat],[lon,lat],[lon,lat],[lon,lat]]]}}]}
```

| *Name* | *Type* | *Description*              | *Required* |
|--------|--------|----------------------------|------------|
| id     | string | identifier of home zone(s) | true       |
| lon    | float  | longitude coordinates      | true       |
| lat    | float  | latitude coordinates       | true       |

"[deviceId]/zones/regular"
---------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed GeoJson Feature Collection):**  
```json
{"type":"FeatureCollection","features":[{"type":"Feature","properties":{"id":"regular1"},"geometry":{"type":"Polygon","coordinates":[[[lon,lat],[lon,lat],[lon,lat],[lon,lat]]]}}]}
```

| *Name* | *Type* | *Description*                 | *Required* |
|--------|--------|-------------------------------|------------|
| id     | string | identifier of regular zone(s) | true       |
| lon    | float  | longitude coordinates         | true       |
| lat    | float  | latitude coordinates          | true       |

"[deviceId]/zones/dangerous"
-----------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed GeoJson Feature Collection):**  
```json
{"type":"FeatureCollection","features":[{"type":"Feature","properties":{"id":"dangerous1"},"geometry":{"type":"Polygon","coordinates":[[[lon,lat],[lon,lat],[lon,lat],[lon,lat]]]}}]}
```

| *Name* | *Type* | *Description*                   | *Required* |
|--------|--------|---------------------------------|------------|
| id     | string | identifier of dangerous zone(s) | true       |
| lon    | float  | longitude coordinates           | true       |
| lat    | float  | latitude coordinates            | true       |


## TO-DO:

"[deviceId]/configuration/gsm"
------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**  
```json
{"a":"a","b":integer}
```

| *Name* | *Type*  | *Description* | *Required* |
|--------|---------|---------------|------------|
| a      | string  |               | true       |
| b      | integer |               | true       |

"[deviceId]/configuration/bluetooth"
------------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**
```json
{"a":"a","b":integer}
```

| *Name* | *Type*  | *Description* | *Required* |
|--------|---------|---------------|------------|
| a      | string  |               | true       |
| b      | integer |               | true       |

"[deviceId]/configuration/lora"
-------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**
```json
{"a":"a","b":integer}
```

| *Name* | *Type*  | *Description* | *Required* |
|--------|---------|---------------|------------|
| a      | string  |               | true       |
| b      | integer |               | true       |

"[deviceId]/configuration/gnss"
-------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**
```json
{"a":"a","b":integer}
```

| *Name* | *Type*  | *Description* | *Required* |
|--------|---------|---------------|------------|
| a      | string  |               | true       |
| b      | integer |               | true       |

"[deviceId]/configuration/accelerometer"
----------------------------------------

| QoS    | 1    |
|--------|------|
| Retain | true |

**Payload (compressed JsonObject):**
```json
{"a":"a","b":integer}
```

| *Name* | *Type*  | *Description* | *Required* |
|--------|---------|---------------|------------|
| a      | string  |               | true       |
| b      | integer |               | true       |
