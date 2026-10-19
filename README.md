# Classroom Environmental Monitoring using SiWG917

An IoT-based classroom monitoring system using the **Silicon Labs SiWG917** wireless SoC to measure environmental parameters such as **temperature, humidity, and noise level**,publish the collected sensor data to a cloud-based MQTT platform for remote monitoring and parameter(s)threshold condition defining and consequent  alert triggering.

## Overview

The system periodically collects sensor data and publishes it through an **MQTT connection** to **ThingSpeak**, where it can be monitored remotely. An Email alert mechanism notifies the operator when predefined environmental conditions are detected.

**Parameters monitored:** Temperature · Humidity · Noise Level

## Objectives

- Monitor classroom environmental conditions in real time
- Collect sensor data using the SiWG917 dev kit's onboard sensors.
- Establish Wi-Fi connectivity via the SiWG917
- Transmit sensor data using MQTT
- Store and visualize data using ThingSpeak
- Generate alerts when conditions exceed predefined thresholds

## System Architecture

```
Classroom Sensors (Temp / Humidity / Noise)
   Purpose: physically sense environmental conditions in the room
            |
            v
        SiWG917
   Purpose: acquire sensor readings, connect to Wi-Fi, and
            publish data as an MQTT client
            |
            v
   ThingSpeak MQTT Broker
   Purpose: receive published MQTT messages and route them
            into the correct channel/fields
            |
            v
   ThingSpeak Channel / Dashboard
   Purpose: store time-series data and provide built-in charts
            for temperature, humidity, and noise
            |
            v
        React
   Purpose: ThingSpeak app that watches channel data and, when
            a condition is met (temp < 15°C OR humidity > 85%),
            triggers an action — here, calling ThingHTTP
            |
            v
       ThingHTTP
   Purpose: receives the trigger from React and sends an
            outbound HTTP request
            |
            v
   Google Apps Script
   Purpose: receives the ThingHTTP request and sends the
            actual email
            |
            v
   Email Notification
   Purpose: notifies the operator (e.g. "AC IS LEFT ON!!",
            "HIGH HUMIDITY!!")
```

## Communication Protocol

```
SiWG917 --Wi-Fi--> MQTT --> ThingSpeak MQTT Broker --> ThingSpeak Channel
```

MQTT is used for its lightweight publish/subscribe model, well suited to IoT and embedded systems.

## Hardware

- Silicon Labs SiWG917 Development Kit
- Onboard temperature and humidity sensor(Si7021:digital relative humidity and temperature sensor)
- Onboard microphone / noise sensing hardware.(ICS43434 I2S :Low-Noise Microphone)
- USB connection for programming and serial monitoring

## Software and Tools

| Tool                    | Purpose                                                     |
| ----------------------- | ----------------------------------------------------------- |
| **Simplicity Studio 6** | Build, debug, and flash the SiWG917 application             |
| **WiseConnect SDK**     | Provides SiWG917 networking, sensor, and MQTT APIs          |
| **Visual Studio Code**  | Source-code editing and project management                  |
| **C**                   | Application and embedded firmware development               |
| **MQTT**                | Transfers sensor data to the cloud                          |
| **ThingSpeak**          | Cloud storage,visualization,monitoring of sensor data,field parameter threshold defining and consequent alert condition triggering.|
| **Git / GitHub**        | Version control and project repository management           |


## System Workflow

1. Initialize the SiWG917 application
2. Initialize onboard sensors
3. Connect to Wi-Fi
4. Obtain network connectivity
5. Establish MQTT connection with the cloud broker
6. Read temperature, humidity, and noise measurements
7. Process sensor values
8. Publish sensor data via MQTT to a ThingSpeak Channel
9. Visualize data on ThingSpeak Channel
10. Check values against alert conditions
11. Trigger notification if an alert condition is met
12. Repeat

## Detailed Procedure

### Step 1: Set up the ThingSpeak channel

1. Log in to the [ThingSpeak](https://thingspeak.com) website and create a new channel.
2. Name the channel and enable the fields:
   - **Field 1**: Temperature
   - **Field 2**: Humidity
3. Save the channel and note the **Channel ID**.

### Step 2: Create the MQTT device (broker credentials)

1. Go to **Devices → MQTT** and create a new MQTT device.
2. Copy the **Client ID**, **Username** and **Password** that ThingSpeak generates.
3. Select the channel the device is allowed to publish to.

### Step 3: Configure the firmware

1. Open the project's `app.c` and update the preprocessor directives with the broker details from Step 2.
2. Open `config/sl_net_default_values.h` and set your local access point's SSID and password so the dev kit can join the Wi-Fi network.
3. Build the project and confirm it compiles successfully.

### Step 4: Set up the alerts

The alert pipeline is **React → ThingHTTP → Google Apps Script → Email**. Create one set of these for each monitored field.

1. In ThingSpeak, go to **Apps → React** and create two React apps for the same channel:
   - **React 1**: monitors the temperature field (Field 1) against its threshold.
   - **React 2**: monitors the humidity field (Field 2) against its threshold.
2. Go to **Apps → ThingHTTP** and create two ThingHTTP apps, one for each React app.
3. In each ThingHTTP app:
   - Paste the deployment URL of the matching Google Apps Script web app.
   - Set the HTTP method to **GET**.
4. In each React app, set the action to trigger its matching ThingHTTP app.

### Summary

| Step | Task | Where |
|------|------|-------|
| 1 | Create channel, enable Field 1 and Field 2, note Channel ID | ThingSpeak |
| 2 | Create MQTT device, copy Client ID / Username / Password | ThingSpeak |
| 3 | Update `app.c` and `sl_net_default_values.h`, then build | Simplicity Studio / VS Code |
| 4 | Create 2 React apps and 2 ThingHTTP apps linked to Apps Script | ThingSpeak |




## Alert System

Example condition used during development:

```
Temperature < 15 °C  OR  Humidity > 85 %
```

```
ThingSpeak Channel --trigger(Sent by React)--> ThingHTTP --HTTP request--> Google Apps Script --> Email Notification
```

## Data Fields

| Field   | Parameter   | Unit                 |
|---------|-------------|-----------------------|
| Field 1 | Temperature | °C                    |
| Field 2 | Humidity    | %                     |
| Field 3 | Noise Level | dB / relative level   |

Exact topic/field mapping depends on your ThingSpeak channel configuration.

## Repository Structure

```
WifiEmbeddedMQTTClient/
├── .vscode/                                  # VS Code workspace settings
├── autogen/                                  # Auto-generated config/init files
├── cmake_gcc/                                # CMake build files (GCC toolchain)
├── cmake_iar/                                # CMake build files (IAR toolchain)
├── cmake_llvm/                               # CMake build files (LLVM toolchain)
├── config/                                   # Project configuration files
├── resources/                                # Project resources
├── simplicity_sdk_2026.6.1/                  # Silicon Labs Simplicity SDK
├── wiseconnect3_sdk_4.1.1/                   # Silicon Labs WiSeConnect 3 SDK
├── .projectlinkstore
├── .silabsExtContext
├── AGENTS.md
├── app.c                                     # Main application logic (Wi-Fi + MQTT)
├── app.h
├── humidity_sensor.c                         # Humidity sensor driver/handling
├── humidity_sensor.h
├── main.c                                    # Application entry point
├── readme.md
├── SEGGER_RTT_Conf.h                         # SEGGER RTT debug configuration
├── vscode.conf
├── wifi_embedded_mqtt_client_soc.pintool     # Pin configuration (Pin Tool)
├── wifi_embedded_mqtt_client_soc.slcp        # Silicon Labs project configuration
├── wifi_embedded_mqtt_client_soc.slpb        # Silicon Labs project build file
├── wifi_embedded_mqtt_client_soc.slps        # Silicon Labs project settings
└── WifiEmbeddedMQTTClient.code-workspace     # VS Code workspace file
```

## Getting Started

### 1. Clone

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd <PROJECT_DIRECTORY>
```

### 2. Install tools

- Simplicity Studio 6
- WiseConnect SDK
- Visual Studio Code
- SiWG917 SDK/toolchain components

### 3. Connect the board

Connect the SiWG917 dev kit via USB and verify the serial/debug interface is detected.
### 4. Configure Wi-Fi and MQTT

1. Initialize the network
2. Connect the SiWG917 to the Wi-Fi network
3. Obtain the device IP address
4. Initialize the MQTT client
5. Configure the MQTT connection parameters
6. Connect to the MQTT broker
7. Wait for the broker to confirm the connection
8. Subscribe to the required topics
9. Publish the initial MQTT message
10. Exchange MQTT messages with the broker
11. Go to `project folder -> config -> sl_net_default_values.h` and update the default Wi-Fi credentials:

```c
#ifndef DEFAULT_WIFI_CLIENT_PROFILE_SSID
#define DEFAULT_WIFI_CLIENT_PROFILE_SSID   "YOURSSID"
#endif

#ifndef DEFAULT_WIFI_CLIENT_CREDENTIAL
#define DEFAULT_WIFI_CLIENT_CREDENTIAL     "Password"
#endif
```
12. For MQTT client initialization, go to `app.c` and make the following changes:

| Parameter | Value in the Project |
|---|---|
| **Client ID** | `ORUKEDwoNSQ5JxQMLB0UIgM` |
| **Broker IP** | `54.81.146.55` |
| **Broker Port** | `1883` |
| **Clean Session** | `1` (Enabled) |
| **Last Will Topic** | `gitam/revanth/monitor/status` |
| **Last Will Message** | `{"device":"SIWG917","status":"OFFLINE"}` |
| **Published Topic** | `channels/3429057/publish` |
| **Published Message** | ` "field1=%ld&field2=%lu",(long)temperature,(unsigned long)humidity` |
| **Publish QoS** | `0` |
| **Last Will QoS** | `1` |
| **Last Will Retained** | `1` |


`config.h` is gitignored — never commit real credentials.

### 5. Build and flash

Build using the SiWG917 toolchain, then flash the firmware to the board.


### 6. Monitor output

Expected serial output:

```
Wi-Fi client interface up Success
Wi-Fi client connected
Init MQTT client Success
MQTT broker connection started
Connected to MQTT broker
Subscribed to Topic: channels/3429057/subscribe
Published message successfully on topic: channels/3429057/publish
```
### 7. Results

**Real-time dashboard**

ThingSpeak visualizes the published temperature, humidity, and noise readings in real time, along with the channel's map location:

![Real-time Dashboard](RealtimeDashboard.jpeg)

**Email alerts**

When an alert condition is triggered (e.g. high humidity or AC left on), the system sends an email notification via the ThingHTTP → Google Apps Script pipeline:

![Email Alerts](EmailAlerts.jpeg)

## Security

Never commit: Wi-Fi passwords, MQTT credentials, API keys, ThingSpeak credentials, or private certificates/keys. Keep them in `config.h` (gitignored) or environment variables.

## Future Enhancements

| Evolution | Future Enhancement | Purpose / Benefit |
|---|---|---|
| **Direct Edge Alerting** | Enable the SiWG917 to detect abnormal temperature/humidity conditions locally and trigger an email alert through a secure communication service. | Provides faster notification by allowing the device to respond immediately to locally detected environmental conditions. |

## Applications

Smart classrooms · smart buildings · indoor air monitoring · labs · offices · libraries · industrial environments · IoT building management

## Project Highlights

| | |
|---|---|
| Platform | Silicon Labs SiWG917 |
| Connectivity | Wi-Fi |
| Protocol | MQTT |
| Cloud Platform | ThingSpeak |
| Language | C |
| Dev Environment | Simplicity Studio 6 / WiseConnect SDK |
| Parameters | Temperature, Humidity, Noise Level |

## Contributors and their Github profile

- Aditya Kolluru(https://github.com/AdityaKolluru05)
- Revanth Dhulipudi(https://github.com/Revanth2k06)

## License

Educational/research use. Add your preferred open-source license before making the repo public.
