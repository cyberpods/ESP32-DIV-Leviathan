<div align="center">

  <h1>ESP32-DIV v3.8</h1>

  <p>
    Enhanced Wireless Security Research Toolkit
  </p>

<!-- Badges -->
<a href="https://github.com/cyberpods/ESP32-DIV-Leviathan
" title="Go to GitHub repo"><img src="https://img.shields.io/static/v1?label=cyberpods&message=ESP32-DIV-Leviathan
&color=purple&logo=github" alt="cyberpods - ESP32-DIV-Leviathan
"></a>

</div>

<br />

## About

This is an **enhanced fork** of the original [CiferTech ESP32-DIV](https://github.com/cifertech/ESP32-DIV) project with additional features across WiFi, Bluetooth, 2.4GHz, and SubGHz categories. Includes bug fixes and ESP32 core 3.x compatibility.


### Credits
- **[CiferTech](https://github.com/cifertech/ESP32-DIV)** - Original ESP32-DIV project
- **[JesseCHale](https://github.com/JesseCHale/ESP32-DIV)** - SPI bus architecture fix (HSPI for touch, VSPI for radios) and stability improvements

> This project is intended for **authorized security research, penetration testing, and educational purposes only**. Always obtain proper authorization before testing on networks or devices you don't own.

<div>&nbsp;</div>


<div>&nbsp;</div>

## SPI Bus Architecture Fix

Thanks to **[JesseCHale](https://github.com/JesseCHale/ESP32-DIV)** for identifying and documenting the SPI bus conflict that caused touchscreen failures after radio operations.

**The Fix:**
| SPI Bus | Pins | Peripherals |
|---------|------|-------------|
| **VSPI** | 18, 19, 23 | NRF24, CC1101, SD Card (shared) |
| **HSPI** | 25, 32, 33, 35 | Touchscreen (dedicated) |

Previously, touchscreen shared VSPI with the radios, causing the touch controller to die after SubGHz/2.4GHz operations. Now touchscreen has its own dedicated SPI bus.

**Hardware Note (V1 Boards):** GPIO4/5 are shared between IR module and NRF24 radio U2. You cannot use IR and 2.4GHz Scanner/ProtoKill features simultaneously.

<div>&nbsp;</div>

## v3.1 Enhancement Update

Major enhancements focused on making the device more useful for security analysts with improved information display and expanded detection capabilities.

### OUI Vendor Lookup
All MAC addresses now show manufacturer names:
- **WiFi Scanner** - Shows AP manufacturer (Cisco, Ubiquiti, TP-Link, etc.)
- **BLE Scanner** - Shows device vendor (Apple, Samsung, etc.)
- **Probe Sniffer** - Identifies device manufacturers
- Detects **MAC randomization** (locally administered addresses)
- Covers 400+ common vendors

### Enhanced Signal Visualization
- **Visual signal bars** (4-bar indicators) replace raw dBm numbers
- **Color coding**: Green (excellent) → Yellow (good) → Orange (fair) → Red (weak)
- **Distance estimation** based on RSSI values
- **Encryption color coding**: Red (OPEN) → Orange (WEP) → Yellow (WPA) → Green (WPA2+)

### Expanded Tracker Detection
Detects 12 tracker types (up from 5):
| Tracker | Company ID |
|---------|------------|
| Apple AirTag | 0x004C |
| Samsung SmartTag | 0x0075 |
| Tile | 0x0131 |
| Chipolo | 0x05A0 |
| **Google Find My Device** | 0x00E0 |
| **Pebblebee** | 0x0544 |
| **Motorola Moto Tag** | 0x0078 |
| **Amazon Sidewalk** | 0x0171 |
| **Eufy SmartTrack** | 0x02D5 |
| **Nut Tracker** | 0x0382 |

Features AirTag-specific detection (type 0x12) and left-behind notification alerts.

### Enhanced Skimmer Detection
Detects 40+ suspicious Bluetooth module patterns (up from 16):
- **HC-series**: HC-05, HC-06, HC-08 (threat level 5)
- **BT04/BT05**: BT04-A, BT05, BT-SPP (threat level 4)
- **JDY series**: JDY-08, JDY-10, JDY-31 (threat level 4)
- **HM series**: HM-10, HM-11, HM-19 (threat level 3)
- **MLT series**: MLT-BT05 (threat level 4)
- **Generic SPP**: Devices advertising SPP service

Threat calculation considers: proximity, pattern severity, hidden names, short names, numeric-only names.

### Enhanced Camera Detection
Detects 50+ camera patterns (up from 22):
- **Consumer brands**: Wyze, Ring, Blink, Arlo, Nest, Yi, Eufy, Reolink, Amcrest
- **Professional brands**: Hikvision, Dahua, Foscam, Axis
- **Spy camera patterns**: SPY, HIDDEN, NANNY, LTK-, A9-, SQ-, XD-
- **Smart home**: TUYA, SMARTLIFE patterns
- **Protocol indicators**: P2P, ONVIF, RTSP

### Weather Station Protocols
Decodes 433MHz weather sensors:
| Protocol | Sensors |
|----------|---------|
| **Acurite** | Tower, 5n1, Rain gauge |
| **LaCrosse** | TX141 series |
| **Oregon Scientific** | THGR122N, THGR228N, WGR800 |
| **Fine Offset** | WH24, WS1900, Ambient Weather |

Displays: Temperature (C/F), Humidity, Wind speed/direction, Rainfall, Battery status

### HVAC Database Expansion
IR codes for 12 AC brands (up from 4):
Samsung, LG, Daikin, Mitsubishi, Haier, Gree, Midea, Carrier, Panasonic, Toshiba, Fujitsu, Generic

### Export & Logging
- **Session Logger** - Continuous logging to SD card with timestamps
- **CSV Export** - WiFi, BLE, tracker, SubGHz scan results
- **JSON Export** - Programmatic analysis format
- Log files saved to `/logs/` and `/exports/` directories

<div>&nbsp;</div>

## Complete Feature List

### WiFi Tools (20 features)
| # | Feature | Description |
|---|---------|-------------|
| 1 | Packet Monitor | Real-time waterfall visualization of WiFi activity across all 14 channels. Useful for finding congested channels and detecting wireless activity patterns. |
| 2 | Beacon Spammer | Broadcast multiple fake SSIDs to flood WiFi scanners. Used for testing how devices handle beacon floods and network list pollution. |
| 3 | WiFi Deauther | Send deauthentication frames to disconnect clients from access points. Tests network resilience against deauth attacks and client reconnection behavior. |
| 4 | Deauth Detector | Passive monitor that alerts when deauthentication attacks are detected on nearby networks. Early warning system for active attacks. |
| 5 | WiFi Scanner | Comprehensive network scanner showing SSID, BSSID, RSSI with signal bars, channel, encryption type, and **vendor name from OUI lookup**. Color-coded encryption and signal strength. Essential for site surveys and network discovery. |
| 6 | Captive Portal | Create a fake access point with a login page to test user awareness. Captures credentials entered into the portal for security assessments. |
| 7 | Evil Twin Detect | Scans for duplicate SSIDs with different BSSIDs that may indicate evil twin attacks. Protects against rogue access point threats. |
| 8 | Probe Sniffer | Captures probe requests to reveal what networks nearby devices are searching for. Useful for understanding device behavior and tracking. |
| 9 | Hidden AP Detect | Discovers cloaked wireless networks that don't broadcast their SSID. Finds "hidden" networks through probe response analysis. |
| 10 | Rogue AP Scanner | Compares detected APs against a whitelist to identify unauthorized access points on your network. Enterprise security auditing tool. |
| 11 | Channel Analyzer | Bar graph visualization of AP distribution across channels 1-14. Helps select the least congested channel for optimal performance. |
| 12 | EAPOL Capture | Captures WPA/WPA2 4-way handshakes and saves as PCAP files. Required for offline password auditing with hashcat/aircrack-ng. |
| 13 | **PMKID Capture** | Captures PMKID from the first EAPOL message without requiring a full handshake. Faster than traditional handshake capture for password auditing. |
| 14 | **Evil Twin AP** | Creates an open clone of a target network. Clients may connect to the stronger signal, allowing traffic interception for security testing. |
| 15 | **Karma Attack** | Responds to ALL probe requests by creating APs matching whatever SSID devices search for. Tests client susceptibility to rogue networks. |
| 16 | **WiFi Geiger** | Signal strength meter with real-time RSSI graph for tracking down specific access points. Useful for locating hidden routers or signal sources. |
| 17 | **Pwnagotchi Detect** | Detect Pwnagotchi devices by monitoring for characteristic beacon frame patterns. |
| 18 | **Probe Flood** | Flood network with probe requests for wireless stress testing. |
| 19 | **Ping Scanner** | Scan local network for active hosts via ICMP ping sweep. |
| 20 | **Station Tracker** | Track WiFi client devices and monitor their AP associations over time. |

### Bluetooth Tools (14 features)
| # | Feature | Description |
|---|---------|-------------|
| 1 | BLE Jammer | Floods BLE advertising channels (37, 38, 39) to disrupt Bluetooth Low Energy connections. Tests device behavior under interference. |
| 2 | BLE Spoofer | Broadcasts fake BLE advertisements mimicking various devices (AirPods, fitness trackers, etc.). Tests how apps handle spoofed devices. |
| 3 | Sour Apple | Spoof Apple BLE proximity pairing packets to trigger AirDrop/Handoff popups on iOS devices. Demonstrates Apple ecosystem vulnerabilities. |
| 4 | Sniffer | Passive BLE advertisement monitor showing device names, MAC addresses, RSSI, and manufacturer data. General BLE reconnaissance tool. |
| 5 | BLE Scanner | Active scanner discovering nearby BLE devices with **vendor name from OUI**, company name from BLE data, signal bars, distance estimation, TX power, and service UUIDs decoded to human-readable names. |
| 6 | Skimmer Detect | Detects **40+ Bluetooth skimmer module patterns** (HC-05/06, BT04/05, JDY, HM, MLT series) with threat level scoring. Analyzes proximity, pattern severity, and suspicious naming. Personal security tool for ATMs and gas pumps. |
| 7 | Camera Finder | Detects **50+ camera BLE patterns** including Wyze, Ring, Blink, Arlo, Nest, Hikvision, Dahua, and spy camera signatures (SPY, HIDDEN, NANNY). Identifies smart home cameras and P2P/ONVIF devices. |
| 8 | **GATT Explorer** | Connects to BLE devices and enumerates all services, characteristics, and descriptors. Read/write values for device analysis. |
| 9 | **Android Spam** | Broadcasts Samsung Galaxy Buds and Google FastPair packets to trigger pairing popups on Android devices. Similar to Sour Apple for Android. |
| 10 | **Beacon Cloner** | Scans for iBeacons, captures their UUID/Major/Minor values, and rebroadcasts them. Test beacon-based systems and location spoofing. |
| 11 | **BLE Fuzzer** | Sends malformed and randomized BLE packets to test device robustness. Useful for finding crashes and vulnerabilities in BLE stacks. |
| 12 | **Flipper Detect** | Detect Flipper Zero devices nearby by BLE advertisement patterns. |
| 13 | **Flipper Spam** | Mimic Flipper Zero BLE advertisements. |
| 14 | **AirTag Spoof** | Spoof Apple AirTag BLE signals for tracking awareness testing. |

### 2.4GHz Tools (8 features)
| # | Feature | Description |
|---|---------|-------------|
| 1 | Scanner | NRF24-based spectrum analyzer scanning all 128 channels (2400-2527 MHz). Visualizes all 2.4GHz activity beyond just WiFi. |
| 2 | 2.4GHz Analyzer | Enhanced spectrum view with waterfall display showing signal history over time. Better for identifying intermittent transmissions. |
| 3 | Tracker Detector | Detects **12 tracker types**: AirTag, Tile, SmartTag, Chipolo, Google Find My, Pebblebee, Motorola Moto Tag, Amazon Sidewalk, Eufy, Nut. AirTag-specific detection and left-behind alerts. Threat level indicators. |
| 4 | Proto Kill | Disrupts various 2.4GHz protocols (WiFi, Zigbee, wireless mice/keyboards) by flooding specific channels. Protocol-aware jamming. |
| 5 | Drone Detector | Identifies drone control signals and FPV video feeds in the 2.4GHz band. Detects DJI, FrSky, and other common drone protocols. |
| 6 | MouseJack Detect | Scans for vulnerable wireless keyboards and mice using unencrypted protocols. Identifies devices susceptible to keystroke injection. |
| 7 | WiFi Cam Detect | Detects hidden WiFi cameras by analyzing traffic patterns and identifying camera-specific network behaviors. |
| 8 | **Multi-Channel 24** | Uses all 3 NRF24 modules simultaneously to monitor 3 different channels at once. Presets for BLE advertising, WiFi channels, Drone FPV, Zigbee, and more. Real-time activity visualization with packet counting. |

### SubGHz Tools (23 features + Security Sensors)
| # | Feature | Description |
|---|---------|-------------|
| 1 | Replay Attack | Capture SubGHz signals and replay them. Works with fixed-code devices like older garage doors, gates, and simple remotes. |
| 2 | SubGHz Jammer | Broadband jamming across common SubGHz frequencies (315/433/868/915 MHz). Tests device behavior when RF communication is blocked. |
| 3 | Saved Profile | Browse, manage, and replay previously captured SubGHz signals stored on SD card. Organize your captured codes library. |
| 4 | Freq Analyzer | Real-time spectrum analyzer for SubGHz bands. Visual display helps identify active frequencies and signal characteristics. |
| 5 | TPMS Decoder | Decodes tire pressure monitoring system signals (typically 315/433 MHz). Shows tire ID, pressure, and temperature data. |
| 6 | Weather Decoder | Decodes **4 weather protocols**: Acurite, LaCrosse TX141, Oregon Scientific v2.1/v3, Fine Offset/Ambient Weather. **Frequency selectable** (433/868/915 MHz). Shows temperature, humidity, wind, rainfall, battery status. |
| 7 | **Smart Meter** | Decodes AMR smart meter transmissions (902-928 MHz). **Protocols**: SCM, SCM+, IDM, R900, NETIDM. Shows meter ID, consumption, tamper flags, leak/backflow alerts for electric, gas, and water meters. |
| 8 | Relay Detector | Detects car key fob relay attack equipment by monitoring for suspicious signal amplification patterns. Vehicle security tool. |
| 9 | Bug Sweeper | Scans SubGHz spectrum for hidden RF transmitters, wireless microphones, and surveillance bugs. Counter-surveillance tool. |
| 10 | Bruteforce | Iterates through code ranges for simple fixed-code systems. Useful for testing security of basic RF remotes and receivers. |
| 11 | ProtoView | Advanced protocol analyzer that decodes and displays raw signal data with timing analysis. Identify unknown protocols. |
| 12 | Rolling Flaws | Tests for known vulnerabilities in rolling code implementations (KeeLoq, etc.). Detects weak implementations and resyncs. |
| 13 | Tesla Charge Port | Transmits the 315MHz signal that opens Tesla charge port doors. Documented Tesla feature for charging access. |
| 14 | POCSAG Decoder | Decodes POCSAG pager transmissions. Displays pager messages, addresses, and function codes from nearby pager networks. |
| 15 | Genie Recorder | Specialized recorder for Genie Intellicode garage door openers. Captures and manages Genie-specific signals. |
| 16 | **RAW Recorder** | Records any SubGHz signal with precise timing data for later analysis or replay. Works with unknown protocols. |
| 17 | **Freq Scanner** | Automated scanner that sweeps through SubGHz bands looking for active transmissions. Discovers what frequencies are in use. |
| 18 | **Garage Protocols** | Pre-loaded codes for common garage door systems: Linear, Chamberlain, LiftMaster, Genie, Craftsman. Quick access to standard protocols. |
| 19 | **Car Key Analyzer** | Multi-frequency capture tool for automotive key fobs. Identifies vehicle manufacturer (Toyota, Honda, Ford, GM, BMW, etc.), detects button functions (Lock/Unlock/Trunk/Panic/Remote Start), analyzes rolling codes, and includes Virtual Remote to save/replay captured signals. |
| 20 | **Doorbell Trigger** | Frequency presets for common wireless doorbells. Includes learn mode to capture and replay doorbell signals. |
| 21 | **Doorbell Brute** | Optimized brute force for wireless doorbells. Modes: Quick patterns (common codes), Last 4/8/12 bits, Full 24-bit. Includes 12+ manufacturer prefixes (Byron, Friedland, Heath Zenith, Honeywell, etc.) to reduce search space. |
| 22 | **FZ Analyzer** | Flipper Zero compatible frequency analyzer for cross-platform signal analysis. |
| 23 | **Security Sensors** | Decode wireless security sensor transmissions — door/window contacts, motion detectors, glass break sensors. |

### Tools (4 features)
| # | Feature | Description |
|---|---------|-------------|
| 1 | Serial Monitor | Built-in serial terminal displaying debug output. Useful for troubleshooting and development without a computer connected. |
| 2 | Update Firmware | Over-the-air firmware updates via WiFi or from SD card. Keep your device updated without USB connection. |
| 3 | Button Checker | Hardware diagnostic tool that tests all buttons and displays their state. Verify button functionality after assembly. |
| 4 | WiFi Setup | On-screen keyboard for configuring WiFi SSID and password without serial connection. |

### Settings (5 options)
| # | Setting | Description |
|---|---------|-------------|
| 1 | Brightness | Adjust TFT display brightness (saves to EEPROM). Balance visibility with battery life. |
| 2 | Screen Timeout | Configure automatic screen-off timer to preserve battery. Screen wakes on button press. |
| 3 | LED Control | Enable or disable the status LED. Turn off for stealth operation or to save power. |
| 4 | Sound | Enable or disable button click feedback sounds. Useful for quiet operation. |
| 5 | Device Info | System information display showing firmware version, ESP32 chip info, free RAM, CPU temperature, and battery voltage. |

<div>&nbsp;</div>

## Hardware

> **Note:** This version is for **V1 boards only** (ESP32-U). The main fork v1.5.0+ is for V2 boards (ESP32-S3) which are not compatible.

ESP32-DIV consists of two boards:

### Main Board
- **ESP32-U (16MB)** - Main microcontroller with Wi-Fi and BLE
- **ILI9341 TFT Display** - 2.4" touchscreen
- **PCF8574** - I/O expander for buttons
- **SD Card Slot** - Stores logs and captured signals
- **TP4056** - Lithium battery charging
- **CP2102** - USB to serial

### Shield
- **3x NRF24 Modules** - 2.4GHz operations
- **1x CC1101 Module** - SubGHz operations (315/433/868/915 MHz)

<div>&nbsp;</div>

## Button Controls

| Button | Function |
|--------|----------|
| UP/DOWN | Navigate menus |
| LEFT/RIGHT | Adjust settings / Navigate within features |
| SELECT | Confirm / Enter feature |
| ENCODER (click) | **Exit current feature** (reliable exit) |

The encoder button provides instant exit from any feature - no more "hold to exit" confusion.

<div>&nbsp;</div>

## Installation

1. Install Arduino IDE with ESP32 board support
2. Install required libraries:
   - TFT_eSPI
   - PCF8574
   - RF24
   - ELECHOUSE_CC1101
   - IRremoteESP8266
   - ArduinoFFT
   - XPT2046_Touchscreen
3. Configure TFT_eSPI for ILI9341 with your pin configuration
4. Upload the sketch to your ESP32

<div>&nbsp;</div>

## Total Feature Count

| Category | Features |
|----------|----------|
| WiFi | 20 |
| Bluetooth | 14 |
| 2.4GHz | 8 |
| SubGHz | 23 |
| Tools | 4 |
| Settings | 5 |
| **Total** | **74 features** |

<div>&nbsp;</div>

## Credits

- Original project by [CiferTech](https://github.com/cifertech/ESP32-DIV)

<div>&nbsp;</div>

## Version History

### v3.8
- Smart Meter: increased capacity from 12 to 25 simultaneous meters
- Smart Meter: added on-screen LOG toggle (touch icon + SELECT button) for SD card CSV logging
- Smart Meter: fixed SD card SPI bus release after logging
- Fixed WiFi Setup keyboard ghost input (wrong PCF pins)
- Fixed serial API SD commands not releasing SPI bus
- Fixed status bar: battery ADC, WiFi RSSI, SD card probe, temperature comparison bug

### v3.7
- Fixed CC1101 lockup when switching SubGHz tools
- Added Security Sensors decoder
- Fixed screenshot SD save (wrong CS pin GPIO 5 → 15)
- Fixed Freq Analyzer CC1101 initialization
- Improved OUI lookup speed (binary search)
- Fixed Pwnagotchi Detector buffer overflow and false positives
- Fixed WiFi scan recursion and memory leaks
- Fixed Hidden AP Detector crash, added detail view
- Fixed BLE Scanner watchdog crash, Beacon Cloner detection
- Fixed Deauth Detector ISR crash
- Optimized DRAM usage

### v3.6
- Added WiFi: Pwnagotchi Detect, Probe Flood, Ping Scanner, Station Tracker
- Added BLE: Flipper Detect, Flipper Spam, AirTag Spoof
- Fixed DRAM overflow, Deauth Detector crashes, menu navigation

## Disclaimer

This tool is provided for authorized security testing and educational purposes only. Users are responsible for ensuring they have proper authorization before using any features on networks or devices. Unauthorized access to computer systems is illegal.
