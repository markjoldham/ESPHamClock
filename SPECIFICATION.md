# ESPHamClock Software Specification

**Version:** 4.22 (Final Release)  
**Author:** Elwood Downey (WB0OEW, SK)  
**Organization:** Clear Sky Institute  
**Date:** January 31, 2026  
**Document Type:** Reverse-Engineered Specification

---

## 1. Overview

### 1.1 Purpose

ESPHamClock is a kiosk-style display application designed for amateur radio operators, providing real-time space weather data, propagation information, DX cluster spots, and ham radio resources on a dedicated display.

### 1.2 Target Platforms

- **ESP8266 Microcontrollers** - Embedded hardware with touchscreen displays
- **UNIX/Linux Systems** - Desktop/server installations with X11 graphics
- **Raspberry Pi** - Popular single-board computer deployment

### 1.3 Key Features

1. **Space Weather Monitoring** - Real-time solar activity, geomagnetic conditions
2. **Propagation Prediction** - Band conditions, grayline, solar terminator
3. **DX Cluster Integration** - Live amateur radio spots from worldwide network
4. **Contest Calendar** - Upcoming amateur radio contests
5. **DXpedition Tracking** - Rare DX operations with map markers
6. **World Map Display** - Azimuthal equidistant projection centered on user location
7. **Customizable Panes** - Modular display areas for different information types
8. **Web Interface** - Remote control and monitoring via HTTP

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    ESPHamClock Frontend                      │
│                      (C++ Application)                       │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Display │  │   Touch  │  │  Network │  │   Web    │   │
│  │  Engine  │  │  Handler │  │  Client  │  │  Server  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTP/TCP
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Backend Server (clearskyinstitute.com)          │
│                      (Perl CGI Scripts)                      │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Space Weather│  │   Weather    │  │  Ham Radio   │     │
│  │   Endpoints  │  │   Endpoints  │  │   Resources  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTP/API
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    External Data Sources                     │
│         (NOAA, DX Cluster, Contest Calendars, etc.)         │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Component Breakdown

#### 2.2.1 Frontend Components

**Display Engine**
- Graphics rendering (Adafruit RA8875 driver for ESP8266, X11 for UNIX)
- Map projection (azimuthal equidistant)
- Font rendering (multiple sizes and styles)
- Color management (RGB565 format)

**Touch Handler**
- Touchscreen input processing
- Menu system navigation
- Interactive map controls
- Gesture recognition (tap, long-press)

**Network Client**
- HTTP client for backend communication
- DX cluster TCP connection
- WiFi management (ESP8266)
- Ethernet support (UNIX)

**Web Server**
- Built-in HTTP server for remote control
- REST-like API endpoints
- Screen capture (BMP format)
- Configuration interface

#### 2.2.2 Backend Components

**Space Weather Endpoints**
- `/spacewx/Bz/Bz.txt` - Magnetic field data
- `/spacewx/xray/xray.txt` - X-ray flux
- `/spacewx/swind/swind-24hr.txt` - Solar wind
- `/spacewx/drap/drap-stats.txt` - D-region absorption
- `/spacewx/kindex/kindex.txt` - Planetary K-index
- `/spacewx/dst/dst.txt` - Disturbance storm time
- `/spacewx/solarflux/solarflux-99.txt` - Solar flux (99 days)
- `/spacewx/ssn/ssn-31.txt` - Sunspot number (31 days)
- `/spacewx/aurora/aurora.txt` - Aurora activity
- `/spacewx/noaaswx/noaaswx.txt` - NOAA space weather summary
- `/spacewx/rank2_coeffs/rank2_coeffs.txt` - Ranking coefficients

**Weather Endpoints**
- `/worldwx/wx.txt` - Global weather grid
- `/wx.pl` - Location-specific weather (CGI script)

**Ham Radio Resource Endpoints**
- `/contests/contests311.txt` - Contest calendar
- `/dxpeds/dxpeditions.txt` - DXpedition list
- `/onta/onta.txt` - On The Air (POTA/SOTA)
- `/prefixes/cty_wt_mod-ll-dxcc.txt` - Country/prefix database
- `/cities/cities2.txt` - Cities database

---

## 3. Functional Requirements

### 3.1 Display Requirements

#### FR-1: Main Display
- **FR-1.1** System shall display a world map in azimuthal equidistant projection
- **FR-1.2** Map shall be centered on user's configured DE (home) location
- **FR-1.3** Map shall show day/night terminator (grayline)
- **FR-1.4** Map shall support pan and zoom operations
- **FR-1.5** Map shall display DX (remote) location marker

#### FR-2: Information Panes
- **FR-2.1** System shall provide 6 configurable information panes
- **FR-2.2** Each pane shall support multiple content types:
  - Space weather plots
  - DX cluster spots
  - Contest calendar
  - DXpedition list
  - On The Air activations
  - RSS feeds
  - SDO solar images
  - DRAP maps
  - NOAA space weather scales
- **FR-2.3** Panes shall auto-update at appropriate intervals
- **FR-2.4** Panes shall support scrolling for long lists

#### FR-3: Clock Display
- **FR-3.1** System shall display UTC time
- **FR-3.2** System shall display DE local time
- **FR-3.3** System shall display DX local time
- **FR-3.4** Clock formats shall be configurable (analog/digital, 12/24 hour)

### 3.2 Space Weather Requirements

#### FR-4: Space Weather Monitoring
- **FR-4.1** System shall display current solar flux (10.7 cm)
- **FR-4.2** System shall display sunspot number
- **FR-4.3** System shall display X-ray flux (short and long wavelength)
- **FR-4.4** System shall display planetary K-index
- **FR-4.5** System shall display solar wind speed, density, temperature
- **FR-4.6** System shall display magnetic field Bz and Bt components
- **FR-4.7** System shall display D-region absorption prediction (DRAP)
- **FR-4.8** System shall display aurora activity
- **FR-4.9** System shall display disturbance storm time (DST)
- **FR-4.10** System shall rank space weather parameters by impact severity

#### FR-5: Space Weather History
- **FR-5.1** System shall plot 24-hour history for real-time parameters
- **FR-5.2** System shall plot 31-day history for sunspot number
- **FR-5.3** System shall plot 99-day history for solar flux
- **FR-5.4** Plots shall auto-scale to data range
- **FR-5.5** Plots shall show current value and trend

### 3.3 Propagation Requirements

#### FR-6: Band Conditions
- **FR-6.1** System shall predict HF band conditions (80m-10m)
- **FR-6.2** Predictions shall be based on solar flux, K-index, and path
- **FR-6.3** System shall display band conditions as color-coded indicators
- **FR-6.4** System shall support VOACAP propagation model

#### FR-7: Grayline Display
- **FR-7.1** System shall calculate solar terminator position
- **FR-7.2** Grayline shall update in real-time
- **FR-7.3** System shall highlight grayline propagation opportunities

### 3.4 DX Cluster Requirements

#### FR-8: DX Cluster Connection
- **FR-8.1** System shall connect to user-configured DX cluster server
- **FR-8.2** System shall support DXSpider, AR-Cluster, and VE7CC protocols
- **FR-8.3** System shall authenticate with callsign
- **FR-8.4** System shall maintain persistent connection
- **FR-8.5** System shall auto-reconnect on connection loss

#### FR-9: Spot Display
- **FR-9.1** System shall display recent DX spots in scrollable list
- **FR-9.2** Spots shall show: callsign, frequency, spotter, time, comment
- **FR-9.3** System shall filter spots by band, mode, or callsign
- **FR-9.4** System shall highlight spots matching watchlist
- **FR-9.5** System shall age-out old spots (configurable timeout)

#### FR-10: Spot Mapping
- **FR-10.1** System shall plot DX spots on world map
- **FR-10.2** Map shall show great circle path from DE to DX
- **FR-10.3** Map shall label spotted callsigns
- **FR-10.4** System shall support spot filtering on map

### 3.5 Contest and DXpedition Requirements

#### FR-11: Contest Calendar
- **FR-11.1** System shall display upcoming amateur radio contests
- **FR-11.2** Contest list shall show: name, start/end times
- **FR-11.3** System shall highlight active contests
- **FR-11.4** System shall support contest detail links
- **FR-11.5** System shall support contest alarms

#### FR-12: DXpedition Tracking
- **FR-12.1** System shall display upcoming and active DXpeditions
- **FR-12.2** DXpedition list shall show: callsign, location, dates
- **FR-12.3** System shall plot DXpedition locations on map
- **FR-12.4** System shall highlight spotted DXpeditions
- **FR-12.5** System shall support DXpedition hiding
- **FR-12.6** System shall integrate with ADIF logbook to show worked status

### 3.6 Configuration Requirements

#### FR-13: User Configuration
- **FR-13.1** System shall support setup wizard on first boot
- **FR-13.2** User shall configure DE location (lat/lng or callsign)
- **FR-13.3** User shall configure DX location
- **FR-13.4** User shall configure timezone preferences
- **FR-13.5** User shall configure display brightness
- **FR-13.6** User shall configure WiFi credentials (ESP8266)
- **FR-13.7** Configuration shall persist in EEPROM/file

#### FR-14: Pane Configuration
- **FR-14.1** User shall select content type for each pane
- **FR-14.2** User shall configure pane-specific options
- **FR-14.3** Pane configuration shall persist across reboots

### 3.7 Web Interface Requirements

#### FR-15: Remote Control
- **FR-15.1** System shall provide HTTP web interface
- **FR-15.2** Web interface shall support screen capture
- **FR-15.3** Web interface shall support configuration changes
- **FR-15.4** Web interface shall support DX location setting
- **FR-15.5** Web interface shall provide status queries

---

## 4. Non-Functional Requirements

### 4.1 Performance Requirements

#### NFR-1: Response Time
- **NFR-1.1** Touch input shall respond within 100ms
- **NFR-1.2** Screen updates shall occur at minimum 1 Hz
- **NFR-1.3** Network requests shall timeout after 30 seconds
- **NFR-1.4** Map rendering shall complete within 2 seconds

#### NFR-2: Resource Usage
- **NFR-2.1** ESP8266 build shall fit in available flash memory
- **NFR-2.2** System shall operate within ESP8266 RAM constraints
- **NFR-2.3** Network bandwidth shall be minimized via caching

### 4.2 Reliability Requirements

#### NFR-3: Availability
- **NFR-3.1** System shall run continuously without manual intervention
- **NFR-3.2** System shall recover from network failures automatically
- **NFR-3.3** System shall handle backend server unavailability gracefully

#### NFR-4: Data Integrity
- **NFR-4.1** System shall validate data from backend before display
- **NFR-4.2** System shall detect and report corrupt data
- **NFR-4.3** Configuration data shall be checksummed

### 4.3 Usability Requirements

#### NFR-5: User Interface
- **NFR-5.1** Interface shall be operable via touchscreen only
- **NFR-5.2** Text shall be readable from 1 meter distance
- **NFR-5.3** Color scheme shall be suitable for 24/7 display
- **NFR-5.4** Menus shall be hierarchical and intuitive

### 4.4 Compatibility Requirements

#### NFR-6: Platform Support
- **NFR-6.1** Code shall compile for ESP8266 (Arduino framework)
- **NFR-6.2** Code shall compile for UNIX (X11 graphics)
- **NFR-6.3** Code shall support multiple display resolutions:
  - 800x480 (standard)
  - 1600x960 (high-res)
  - 2400x1440 (ultra-high-res)

#### NFR-7: Hardware Support
- **NFR-7.1** System shall support Adafruit RA8875 display controller
- **NFR-7.2** System shall support resistive touchscreen
- **NFR-7.3** System shall support optional BME280 sensor (temp/humidity/pressure)
- **NFR-7.4** System shall support optional LTR329 light sensor

---

## 5. Data Requirements

### 5.1 Configuration Data

#### DR-1: User Settings
- Callsign (DE and DX)
- Location (latitude, longitude, grid square)
- Timezone preferences
- Display brightness
- WiFi credentials (ESP8266)
- DX cluster server and credentials
- Pane selections
- Map style and projection
- Clock formats

#### DR-2: Persistent State
- Last known good configuration
- Cached data files (with timestamps)
- Watchlist entries
- Hidden DXpeditions list
- ADIF logbook (optional)

### 5.2 Runtime Data

#### DR-3: Space Weather Data
- Real-time measurements (updated every 1-5 minutes)
- Historical data (24 hours to 99 days)
- Ranking coefficients (static)

#### DR-4: Weather Data
- Global weather grid (updated every 45 minutes)
- Location-specific weather (updated every 30 minutes)

#### DR-5: Ham Radio Data
- DX cluster spots (real-time stream)
- Contest calendar (updated daily)
- DXpedition list (updated daily)
- POTA/SOTA activations (updated hourly)
- Prefix/country database (updated weekly)
- Cities database (static)

---

## 6. Interface Requirements

### 6.1 User Interfaces

#### UI-1: Touchscreen Interface
- Main map display with touch navigation
- Menu system (hierarchical)
- Setup wizard
- Pane interaction (tap to expand, scroll)
- Context menus (long-press)

#### UI-2: Web Interface
- HTTP endpoints for remote control
- Screen capture (BMP format)
- Configuration API
- Status queries

### 6.2 Hardware Interfaces

#### HW-1: Display
- SPI interface to RA8875 controller (ESP8266)
- X11 graphics (UNIX)
- Resistive touchscreen (4-wire)

#### HW-2: Network
- WiFi (ESP8266)
- Ethernet (UNIX)

#### HW-3: Sensors (Optional)
- I2C interface to BME280 (temperature, humidity, pressure)
- I2C interface to LTR329 (light sensor)

### 6.3 Software Interfaces

#### SW-1: Backend Server
- HTTP/1.0 protocol
- Text-based data formats
- Endpoint: `http://clearskyinstitute.com:80/ham/HamClock/`

#### SW-2: DX Cluster
- TCP connection (typically port 7373 or 23)
- Telnet-style protocol
- Line-based text commands

#### SW-3: External Services
- NTP time synchronization
- IP geolocation services
- Radio control (Hamlib, Flrig)
- GPSD (GPS daemon)

---

## 7. Operational Requirements

### 7.1 Installation

#### OP-1: ESP8266 Installation
- Flash firmware via USB
- Configure WiFi via setup wizard
- Mount display in enclosure
- Power via USB (5V)

#### OP-2: UNIX Installation
- Compile from source
- Install dependencies (X11, fonts)
- Run as user application or systemd service
- Configure via command-line options or config file

### 7.2 Maintenance

#### OP-3: Updates
- ESP8266: OTA (Over-The-Air) updates via backend server
- UNIX: Manual recompilation or package update
- Configuration preserved across updates

#### OP-4: Diagnostics
- Serial console logging (ESP8266)
- Syslog support (UNIX)
- Debug levels (configurable)
- Diagnostic upload to backend server (optional)

---

## 8. Security Requirements

### 8.1 Authentication

#### SEC-1: DX Cluster
- Callsign-based authentication
- No password required (per cluster policy)

#### SEC-2: Web Interface
- No authentication (local network only)
- User responsible for network security

### 8.2 Privacy

#### SEC-3: Data Collection
- No telemetry or usage tracking
- Diagnostic uploads are user-initiated only
- Configuration stored locally only

#### SEC-4: Network Security
- HTTP only (no HTTPS) - backend limitation
- User data not transmitted except:
  - Callsign to DX cluster
  - IP address in diagnostic uploads
  - Location for weather queries

---

## 9. Constraints and Assumptions

### 9.1 Constraints

#### C-1: Hardware Constraints
- ESP8266 limited RAM (~80KB available)
- ESP8266 limited flash (~4MB)
- Display resolution fixed at compile time

#### C-2: Network Constraints
- Backend server must be reachable
- DX cluster server must be reachable (for cluster features)
- No offline mode (requires network for data)

#### C-3: Software Constraints
- C++ codebase (Arduino framework for ESP8266)
- Single-threaded execution model
- No dynamic memory allocation in critical paths

### 9.2 Assumptions

#### A-1: Backend Availability
- Backend server at clearskyinstitute.com is available
- Backend data formats remain stable
- Backend update frequency is adequate

#### A-2: User Environment
- User has WiFi or Ethernet connectivity
- User has basic amateur radio knowledge
- User has valid amateur radio callsign

#### A-3: Display Environment
- Display is in fixed location (not mobile)
- Display is visible 24/7
- Ambient lighting is controlled

---

## 10. Future Considerations

### 10.1 Known Limitations

- Backend server is single point of failure
- No HTTPS support
- Limited mobile device support
- No multi-user support
- No cloud synchronization

### 10.2 Potential Enhancements

- Alternative backend servers
- HTTPS support
- Mobile app companion
- Cloud configuration backup
- Multi-language support
- Additional map projections
- Integration with logging software
- Support for additional hardware platforms

---

## 11. Glossary

**ADIF** - Amateur Data Interchange Format (logbook format)  
**ADXO** - Announced DX Operations  
**DE** - Home station location  
**DRAP** - D-Region Absorption Prediction  
**DST** - Disturbance Storm Time (geomagnetic index)  
**DX** - Distant station or location  
**DXpedition** - Amateur radio expedition to rare location  
**EEPROM** - Electrically Erasable Programmable Read-Only Memory  
**ESP8266** - WiFi-enabled microcontroller  
**Grayline** - Solar terminator (day/night boundary)  
**HF** - High Frequency (3-30 MHz amateur radio bands)  
**K-index** - Geomagnetic activity index  
**NOAA** - National Oceanic and Atmospheric Administration  
**OTA** - Over-The-Air (wireless firmware update)  
**POTA** - Parks On The Air (portable operations program)  
**RA8875** - Display controller chip  
**SOTA** - Summits On The Air (mountain operations program)  
**SWPC** - Space Weather Prediction Center (NOAA)  
**VOACAP** - Voice of America Coverage Analysis Program (propagation model)

---

## 12. References

- ESPHamClock Source Code: https://github.com/fang64/hamclock
- Original Website: http://www.clearskyinstitute.com/ham/HamClock/
- NOAA SWPC: https://www.swpc.noaa.gov/
- Amateur Radio Relay League: http://www.arrl.org/

---

**Document History**

| Version | Date       | Author | Description                    |
| ------- | ---------- | ------ | ------------------------------ |
| 1.0     | 2026-01-31 | AI     | Initial reverse-engineered spec |

