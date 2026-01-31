# ESPHamClock Software Design Document

**Version:** 4.22 (Final Release)  
**Author:** Elwood Downey (WB0OEW, SK)  
**Organization:** Clear Sky Institute  
**Date:** January 31, 2026  
**Document Type:** Reverse-Engineered Design

---

## 1. Introduction

### 1.1 Purpose

This document describes the software design of ESPHamClock, a kiosk-style display application for amateur radio operators. The design has been reverse-engineered from the source code to document the architecture, components, and implementation details.

### 1.2 Scope

This document covers:
- System architecture and component design
- Data flow and state management
- Key algorithms and data structures
- Backend communication protocols
- User interface design
- Platform-specific implementations

### 1.3 Intended Audience

- Software developers maintaining or extending ESPHamClock
- System administrators deploying ESPHamClock
- Amateur radio operators interested in technical details
- Developers creating alternative backends

---

## 2. System Architecture

### 2.1 Architectural Overview

ESPHamClock follows a client-server architecture with a monolithic frontend and distributed backend services.

```
┌────────────────────────────────────────────────────────────────┐
│                     ESPHamClock Frontend                        │
│                    (Monolithic C++ Application)                 │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │   Graphics   │  │    Input     │  │   Network    │        │
│  │   Subsystem  │  │   Subsystem  │  │   Subsystem  │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │    State     │  │    Data      │  │     Web      │        │
│  │  Management  │  │   Storage    │  │    Server    │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 Design Principles

1. **Platform Abstraction** - Code shared between ESP8266 and UNIX with platform-specific implementations
2. **Event-Driven** - Main loop processes events (touch, network, timers)
3. **Stateful** - Maintains configuration and runtime state
4. **Modular Panes** - Display areas are independent, pluggable components
5. **Cached Data** - Minimize network requests via local caching
6. **Fail-Safe** - Graceful degradation when services unavailable

---

## 3. Component Design

### 3.1 Graphics Subsystem

#### 3.1.1 Display Abstraction

**Purpose:** Provide unified graphics API across platforms

**Key Classes/Modules:**
- `Adafruit_RA8875` - ESP8266 display driver
- X11 graphics wrapper - UNIX display implementation
- `tft` - Global display object

**Key Functions:**

- `drawPixel(x, y, color)` - Draw single pixel
- `fillRect(x, y, w, h, color)` - Draw filled rectangle
- `drawLine(x1, y1, x2, y2, color)` - Draw line
- `setCursor(x, y)` - Set text cursor position
- `print(text)` - Draw text at cursor
- `fillScreen(color)` - Clear screen

**Color Format:** RGB565 (16-bit color)

#### 3.1.2 Map Rendering

**Purpose:** Render world map with azimuthal equidistant projection

**Key Algorithms:**

**Azimuthal Equidistant Projection:**
```cpp
// Convert lat/lng to screen coordinates
void ll2s(LatLong ll, SCoord &s) {
    // Center map on DE location
    float lat_diff = ll.lat_d - de_ll.lat_d;
    float lng_diff = ll.lng_d - de_ll.lng_d;
    
    // Calculate distance and bearing
    float dist = haversine_distance(de_ll, ll);
    float bearing = calculate_bearing(de_ll, ll);
    
    // Project to screen
    float r = dist * scale_factor;
    s.x = center_x + r * sin(bearing);
    s.y = center_y - r * cos(bearing);
}
```

**Grayline Calculation:**
```cpp
// Calculate solar terminator
void calculateGrayline() {
    // Get sun position
    float sun_lat, sun_lng;
    calculate_sun_position(current_time, sun_lat, sun_lng);
    
    // For each point on map
    for (each map point) {
        // Calculate solar zenith angle
        float zenith = calculate_zenith_angle(point, sun_lat, sun_lng);
        
        // Twilight zone: 90° to 108° zenith
        if (zenith > 90 && zenith < 108) {
            // Draw grayline
        }
    }
}
```

**Map Styles:**
- Solid colors (day/night)
- Terrain (topographic)
- Countries (political boundaries)
- DRAP overlay (absorption prediction)

#### 3.1.3 Font Rendering

**Purpose:** Display text in multiple sizes and styles

**Font System:**
- Adafruit GFX font format
- Multiple font sizes: FAST (small), SMALL, LARGE
- Font styles: LIGHT, BOLD
- Custom fonts: Germano, Courier Prime Sans

**Text Rendering:**
```cpp
void selectFontStyle(FontStyle style, FontSize size);
uint16_t getTextWidth(const char *text);
void drawString(uint16_t x, uint16_t y, const char *text, uint16_t color);
```

### 3.2 Input Subsystem

#### 3.2.1 Touch Input

**Purpose:** Process touchscreen input and gestures

**Touch States:**
- `TS_NONE` - No touch
- `TS_TOUCH` - Initial touch
- `TS_DRAG` - Touch and move
- `TS_RELEASE` - Touch released

**Touch Processing:**
```cpp
void processTouchInput() {
    TouchState ts = readTouch();
    
    if (ts == TS_TOUCH) {
        // Check for menu hits
        if (checkMenuHit(touch_x, touch_y)) {
            handleMenuSelection();
        }
        // Check for pane hits
        else if (checkPaneHit(touch_x, touch_y)) {
            handlePaneInteraction();
        }
        // Check for map hits
        else if (checkMapHit(touch_x, touch_y)) {
            handleMapInteraction();
        }
    }
}
```

**Long-Press Detection:**
```cpp
// Detect long-press for context menus
if (touch_duration > LONG_PRESS_THRESHOLD) {
    showContextMenu(touch_x, touch_y);
}
```

#### 3.2.2 Menu System

**Purpose:** Hierarchical menu navigation

**Menu Structure:**
```cpp
typedef struct {
    const char *label;
    MenuFieldType type;  // MENU_LABEL, MENU_TOGGLE, MENU_1OFN, etc.
    bool *bool_ptr;      // For toggle items
    int *int_ptr;        // For numeric items
    void (*callback)();  // For action items
} MenuItem;
```

**Menu Types:**
- `MENU_LABEL` - Display-only text
- `MENU_TOGGLE` - Boolean on/off
- `MENU_1OFN` - Radio button group
- `MENU_AL1OFN` - Always-on radio button
- `MENU_IGNORE` - Hidden item

### 3.3 Network Subsystem

#### 3.3.1 HTTP Client

**Purpose:** Fetch data from backend server

**Key Functions:**
```cpp
bool httpHCGET(WiFiClient &client, const char *host, const char *path);
bool httpSkipHeader(WiFiClient &client);
bool getTCPLine(WiFiClient &client, char *line, int max_len);
```

**Backend Communication:**
```cpp
// Standard request pattern
WiFiClient client;
if (client.connect(backend_host, backend_port)) {
    // Send HTTP request
    httpHCGET(client, backend_host, "/ham/HamClock/spacewx/Bz/Bz.txt");
    
    // Skip HTTP headers
    httpSkipHeader(client);
    
    // Read response body
    while (getTCPLine(client, line, sizeof(line))) {
        // Parse data
    }
    
    client.stop();
}
```

**Caching Strategy:**
```cpp
FILE *openCachedFile(const char *filename, const char *url, 
                     int max_age, int min_size) {
    // Check if cached file exists and is fresh
    if (file_exists(filename) && file_age(filename) < max_age) {
        return fopen(filename, "r");
    }
    
    // Download fresh data
    if (downloadFile(url, filename)) {
        return fopen(filename, "r");
    }
    
    return NULL;
}
```

#### 3.3.2 DX Cluster Client

**Purpose:** Maintain connection to DX cluster server

**Connection Management:**
```cpp
void connectDXCluster() {
    if (dxc_client.connect(cluster_host, cluster_port)) {
        // Send login
        dxc_client.print(de_call);
        dxc_client.print("-");
        dxc_client.println(SSID);
        
        // Request initial spots
        sendDXCCommand("sh/dx filter real 30");
    }
}
```

**Spot Parsing:**
```cpp
// Parse DX spot line
// Format: DX de CALL: FREQ DX_CALL Comment TIME
bool parseDXSpot(const char *line, DXSpot &spot) {
    // Extract fields using sscanf or string parsing
    if (sscanf(line, "DX de %s %f %s %[^\n]", 
               spot.spotter, &spot.freq, spot.dx_call, spot.comment) == 4) {
        // Lookup DX location
        call2LL(spot.dx_call, spot.ll);
        return true;
    }
    return false;
}
```

#### 3.3.3 Web Server

**Purpose:** Provide HTTP interface for remote control

**Endpoint Handlers:**
```cpp
void handleWebRequest(WiFiClient &client, const char *request) {
    if (strstr(request, "GET /live.bmp")) {
        // Screen capture
        sendScreenCapture(client);
    }
    else if (strstr(request, "GET /set_dx?")) {
        // Set DX location
        parseDXParams(request);
        sendOKResponse(client);
    }
    else if (strstr(request, "GET /get_config")) {
        // Return configuration
        sendConfigJSON(client);
    }
}
```

### 3.4 State Management

#### 3.4.1 Configuration Storage

**Purpose:** Persist user settings across reboots

**Storage Mechanism:**
- ESP8266: EEPROM emulation in flash
- UNIX: File-based storage

**Configuration Structure:**
```cpp
typedef struct {
    char de_call[MAX_CALL_LEN];
    char dx_call[MAX_CALL_LEN];
    LatLong de_ll;
    LatLong dx_ll;
    int de_tz;
    int dx_tz;
    uint8_t brightness;
    uint8_t pane_choices[PANE_N];
    // ... many more fields
    uint16_t checksum;
} NVConfig;
```

**EEPROM Access:**
```cpp
bool NVReadUInt8(NVField field, uint8_t *value);
bool NVWriteUInt8(NVField field, uint8_t value);
bool NVReadString(NVField field, char *str, int max_len);
bool NVWriteString(NVField field, const char *str);
```

#### 3.4.2 Runtime State

**Purpose:** Track current application state

**Key State Variables:**
```cpp
// Display state
PlotChoice plot_ch[PANE_N];  // Current pane content
bool fresh_redraw[PANE_N];   // Pane needs full redraw
time_t next_update[PANE_N];  // Next update time

// Location state
LatLong de_ll;               // DE location
LatLong dx_ll;               // DX location
char de_call[MAX_CALL_LEN];  // DE callsign
char dx_call[MAX_CALL_LEN];  // DX callsign

// Network state
bool wifi_connected;
bool backend_reachable;
bool dxc_connected;

// Time state
time_t now_t;                // Current UTC time
int de_tz_offset;            // DE timezone offset
int dx_tz_offset;            // DX timezone offset
```

### 3.5 Data Storage

#### 3.5.1 Space Weather Data

**Purpose:** Store and manage space weather measurements

**Data Structures:**
```cpp
typedef struct {
    float value;             // Current value
    time_t timestamp;        // Measurement time
    float *history;          // Historical values
    int n_history;           // Number of history points
    float min_val, max_val;  // Range for plotting
} SpaceWxData;

SpaceWxData space_wx[SPCWX_N];  // Array of all space weather parameters
```

**Space Weather Types:**
```cpp
enum SpaceWxType {
    SPCWX_SSN,      // Sunspot number
    SPCWX_XRAY,     // X-ray flux
    SPCWX_FLUX,     // Solar flux
    SPCWX_KP,       // Planetary K-index
    SPCWX_SWIND,    // Solar wind
    SPCWX_DRAP,     // D-region absorption
    SPCWX_BZ,       // Magnetic field Bz
    SPCWX_AURORA,   // Aurora activity
    SPCWX_DST,      // Disturbance storm time
    SPCWX_N         // Count
};
```

#### 3.5.2 DX Cluster Data

**Purpose:** Store DX spots

**Spot Structure:**
```cpp
typedef struct {
    char dx_call[MAX_CALL_LEN];
    char spotter[MAX_CALL_LEN];
    float freq;
    LatLong ll;
    time_t timestamp;
    char comment[MAX_COMMENT_LEN];
    bool on_watchlist;
} DXSpot;

DXSpot *dxc_spots;           // Array of spots
int n_dxspots;               // Number of spots
```

#### 3.5.3 Contest and DXpedition Data

**Purpose:** Store upcoming events

**Contest Entry:**
```cpp
typedef struct {
    time_t start_t;
    time_t end_t;
    char *title;
    char *date_str;
    char *url;
    bool was_active;
} ContestEntry;
```

**DXpedition Entry:**
```cpp
typedef struct {
    time_t start_t;
    time_t end_t;
    char *call;
    char *loc;
    char *title;
    char *date_str;
    char *url;
    LatLong ll;
    int dxcc;
    char prefix[MAX_PREF_LEN];
    bool was_active;
} DXPedEntry;
```

---

## 4. Key Algorithms

### 4.1 Great Circle Path Calculation

**Purpose:** Calculate shortest path between two points on Earth

**Haversine Formula:**
```cpp
float haversine_distance(LatLong p1, LatLong p2) {
    float lat1 = radians(p1.lat_d);
    float lat2 = radians(p2.lat_d);
    float dlat = lat2 - lat1;
    float dlng = radians(p2.lng_d - p1.lng_d);
    
    float a = sin(dlat/2) * sin(dlat/2) +
              cos(lat1) * cos(lat2) *
              sin(dlng/2) * sin(dlng/2);
    
    float c = 2 * atan2(sqrt(a), sqrt(1-a));
    
    return EARTH_RADIUS * c;  // Distance in km
}
```

**Bearing Calculation:**
```cpp
float calculate_bearing(LatLong from, LatLong to) {
    float lat1 = radians(from.lat_d);
    float lat2 = radians(to.lat_d);
    float dlng = radians(to.lng_d - from.lng_d);
    
    float y = sin(dlng) * cos(lat2);
    float x = cos(lat1) * sin(lat2) -
              sin(lat1) * cos(lat2) * cos(dlng);
    
    return atan2(y, x);  // Bearing in radians
}
```

### 4.2 Callsign to Location Lookup

**Purpose:** Convert amateur radio callsign to geographic location

**Algorithm:**
```cpp
bool call2LL(const char *call, LatLong &ll) {
    // 1. Try exact match in prefix database
    if (findExactPrefix(call, ll)) {
        return true;
    }
    
    // 2. Try progressive prefix matching
    // Example: VE3VSM/HR9 -> try HR9, then HR, then H
    for (int len = strlen(call); len > 0; len--) {
        char prefix[MAX_PREF_LEN];
        strncpy(prefix, call, len);
        prefix[len] = '\0';
        
        if (findPrefix(prefix, ll)) {
            return true;
        }
    }
    
    return false;
}
```

**Prefix Database Format:**
```
# Country/prefix database (cty_wt_mod-ll-dxcc.txt)
# Format: Prefix:Country:DXCC:Lat:Lng:TZ:Continent
VE:Canada:1:45.0:-75.0:-5:NA
K:United States:291:38.0:-97.0:-5:NA
G:England:223:52.0:-2.0:0:EU
```

### 4.3 Space Weather Ranking

**Purpose:** Rank space weather parameters by impact severity

**Ranking Formula:**
```cpp
// y = ax² + bx + c
float rankSpaceWx(SpaceWxType type, float value) {
    // Get coefficients from rank2_coeffs.txt
    float a = rank_coeffs[type].a;
    float b = rank_coeffs[type].b;
    float c = rank_coeffs[type].c;
    
    // Calculate rank
    float rank = a * value * value + b * value + c;
    
    return rank;
}
```

**Ranking Scale:**
- Negative values: Below normal (good conditions)
- Zero: Normal conditions
- Positive values: Above normal (poor conditions)
- Higher values: More severe impact

### 4.4 Band Condition Prediction

**Purpose:** Predict HF band propagation quality

**VOACAP Integration:**
```cpp
BandCondition predictBandCondition(int band_m, LatLong from, LatLong to) {
    // Get current space weather
    float solar_flux = space_wx[SPCWX_FLUX].value;
    float kp = space_wx[SPCWX_KP].value;
    
    // Calculate path parameters
    float distance = haversine_distance(from, to);
    float bearing = calculate_bearing(from, to);
    
    // VOACAP model (simplified)
    float muf = calculate_muf(solar_flux, distance);  // Maximum Usable Frequency
    float luf = calculate_luf(kp, distance);          // Lowest Usable Frequency
    
    float band_freq = band_to_freq(band_m);
    
    if (band_freq > muf) {
        return BAND_POOR;      // Too high
    } else if (band_freq < luf) {
        return BAND_POOR;      // Too low
    } else if (band_freq > muf * 0.85) {
        return BAND_FAIR;      // Near MUF
    } else {
        return BAND_GOOD;      // Optimal
    }
}
```

---

## 5. Data Flow

### 5.1 Startup Sequence

```
1. Initialize hardware (display, touch, network)
2. Load configuration from EEPROM/file
3. If first boot:
   - Run setup wizard
   - Save configuration
4. Connect to WiFi/network
5. Sync time via NTP
6. Initialize display (draw map, panes)
7. Start main loop
```

### 5.2 Main Loop

```cpp
void loop() {
    // Update time
    updateTime();
    
    // Process touch input
    processTouchInput();
    
    // Update network connections
    updateDXCluster();
    updateBackendData();
    
    // Update display
    updatePanes();
    updateMap();
    updateClocks();
    
    // Handle web server requests
    handleWebServer();
    
    // Yield to system (ESP8266)
    yield();
}
```

### 5.3 Data Update Flow

```
Backend Server
      │
      │ HTTP GET (periodic)
      ▼
┌─────────────┐
│ HTTP Client │
└─────────────┘
      │
      │ Parse & validate
      ▼
┌─────────────┐
│ Data Cache  │
└─────────────┘
      │
      │ Read
      ▼
┌─────────────┐
│ Display     │
│ Rendering   │
└─────────────┘
```

---

## 6. Platform-Specific Design

### 6.1 ESP8266 Implementation

**Hardware Abstraction:**
- Arduino framework
- Adafruit libraries for display
- ESP8266WiFi for networking
- EEPROM library for storage

**Memory Management:**
- Minimize dynamic allocation
- Use PROGMEM for constants
- Careful buffer sizing
- Periodic garbage collection

**Power Management:**
- Display sleep mode
- WiFi power saving
- Watchdog timer

### 6.2 UNIX Implementation

**Hardware Abstraction:**
- X11 for graphics
- Standard sockets for networking
- File-based configuration
- POSIX threads (optional)

**Resource Management:**
- Larger buffers allowed
- More aggressive caching
- Background threads for network I/O

---

## 7. Error Handling

### 7.1 Network Errors

**Strategy:** Retry with exponential backoff

```cpp
time_t nextWiFiRetry(const char *label) {
    static int retry_count = 0;
    
    // Exponential backoff: 1, 2, 4, 8, 16 minutes (max)
    int delay = min(60 * (1 << retry_count), 16 * 60);
    retry_count++;
    
    Serial.printf("%s: retry in %d sec\n", label, delay);
    
    return myNow() + delay;
}
```

### 7.2 Data Validation

**Strategy:** Validate before use, fallback to cached data

```cpp
bool validateSpaceWxData(const char *data) {
    // Check format
    if (!checkFormat(data)) {
        return false;
    }
    
    // Check value ranges
    if (!checkRanges(data)) {
        return false;
    }
    
    // Check timestamp
    if (!checkTimestamp(data)) {
        return false;
    }
    
    return true;
}
```

### 7.3 Display Errors

**Strategy:** Show error messages, continue operation

```cpp
void showError(const char *message) {
    // Draw error box
    tft.fillRect(error_box.x, error_box.y, error_box.w, error_box.h, RA8875_RED);
    tft.setCursor(error_box.x + 5, error_box.y + 5);
    tft.setTextColor(RA8875_WHITE);
    tft.print(message);
    
    // Auto-dismiss after timeout
    error_dismiss_time = millis() + ERROR_TIMEOUT;
}
```

---

## 8. Performance Optimization

### 8.1 Display Optimization

**Dirty Rectangle Tracking:**
```cpp
// Only redraw changed areas
void updatePane(int pane_idx) {
    if (fresh_redraw[pane_idx]) {
        // Full redraw
        drawPaneComplete(pane_idx);
        fresh_redraw[pane_idx] = false;
    } else {
        // Incremental update
        drawPaneIncremental(pane_idx);
    }
}
```

**Double Buffering (UNIX):**
```cpp
// Render to off-screen buffer
render_to_buffer();

// Swap buffers
XdbeSwapBuffers(display, &swap_info, 1);
```

### 8.2 Network Optimization

**Connection Reuse:**
```cpp
// Keep DX cluster connection alive
if (!dxc_client.connected()) {
    reconnectDXCluster();
}
```

**Batch Requests:**
```cpp
// Fetch multiple data files in one session
WiFiClient client;
if (client.connect(backend_host, backend_port)) {
    fetchFile(client, "/spacewx/Bz/Bz.txt");
    fetchFile(client, "/spacewx/xray/xray.txt");
    fetchFile(client, "/spacewx/kindex/kindex.txt");
    client.stop();
}
```

### 8.3 Memory Optimization

**String Pooling:**
```cpp
// Reuse string buffers
static char line_buffer[256];
static char temp_buffer[128];
```

**Lazy Loading:**
```cpp
// Load data only when pane is visible
if (isPaneVisible(PANE_CONTESTS)) {
    if (!contests_loaded) {
        loadContests();
        contests_loaded = true;
    }
}
```

---

## 9. Testing Considerations

### 9.1 Unit Testing

**Testable Components:**
- Coordinate conversion functions
- Data parsing functions
- Validation functions
- Calculation algorithms

**Example Test:**
```cpp
void test_haversine_distance() {
    LatLong nyc = {40.7128, -74.0060};
    LatLong london = {51.5074, -0.1278};
    
    float dist = haversine_distance(nyc, london);
    
    assert(abs(dist - 5570.0) < 10.0);  // ~5570 km
}
```

### 9.2 Integration Testing

**Test Scenarios:**
- Backend server unavailable
- DX cluster connection failure
- Invalid data formats
- Network interruption
- Touch input sequences
- Configuration changes

### 9.3 Platform Testing

**ESP8266:**
- Memory usage monitoring
- Long-term stability (days/weeks)
- WiFi reconnection
- OTA updates

**UNIX:**
- Multiple display resolutions
- Different X11 configurations
- Network interface changes

---

## 10. Maintenance and Evolution

### 10.1 Code Organization

**Directory Structure:**
```
src/
├── ESPHamClock.cpp      # Main application
├── HamClock.h           # Common definitions
├── wifi.cpp             # Network functions
├── spacewx.cpp          # Space weather
├── wx.cpp               # Weather
├── dxcluster.cpp        # DX cluster
├── contests.cpp         # Contests
├── dxpeds.cpp           # DXpeditions
├── plotmap.cpp          # Map rendering
├── touch.cpp            # Touch input
├── webserver.cpp        # Web server
├── setup.cpp            # Setup wizard
└── ArduinoLib/          # Platform abstraction
```

### 10.2 Adding New Features

**Pane Content Type:**
1. Define new `PlotChoice` enum value
2. Implement `updateXXX()` function
3. Implement `drawXXX()` function
4. Add to pane selection menu
5. Add update scheduling

**Backend Data Source:**
1. Define endpoint URL
2. Implement fetch function
3. Implement parse function
4. Add caching logic
5. Add error handling

### 10.3 Porting to New Platform

**Requirements:**
1. Graphics library (pixel, line, rectangle, text)
2. Touch input (optional)
3. Network stack (TCP/IP, HTTP)
4. Persistent storage
5. Time functions

**Steps:**
1. Create platform-specific `ArduinoLib` implementation
2. Implement display driver wrapper
3. Implement network wrapper
4. Implement storage wrapper
5. Test and optimize

---

## 11. Security Considerations

### 11.1 Network Security

**Threats:**
- Man-in-the-middle attacks (HTTP only)
- DX cluster impersonation
- Web interface unauthorized access

**Mitigations:**
- User responsible for network security
- Web interface on local network only
- No sensitive data transmitted

### 11.2 Input Validation

**Threats:**
- Malformed backend data
- Buffer overflows
- Format string attacks

**Mitigations:**
- Validate all input data
- Use safe string functions (strncpy, snprintf)
- Bounds checking on arrays

---

## 12. Future Design Considerations

### 12.1 Modularity Improvements

- Plugin architecture for pane types
- Configurable backend endpoints
- Theme system for colors/fonts
- Localization support

### 12.2 Performance Improvements

- Hardware acceleration (GPU)
- Multi-threading (UNIX)
- Compressed data formats
- Delta updates

### 12.3 Feature Additions

- Mobile app companion
- Cloud configuration sync
- Multi-user support
- Voice control
- Integration with logging software

---

## 13. References

- ESPHamClock Source Code: https://github.com/fang64/hamclock
- Adafruit RA8875 Library: https://github.com/adafruit/Adafruit_RA8875
- ESP8266 Arduino Core: https://github.com/esp8266/Arduino
- X11 Programming Manual: https://www.x.org/releases/current/doc/

---

**Document History**

| Version | Date       | Author | Description                  |
| ------- | ---------- | ------ | ---------------------------- |
| 1.0     | 2026-01-31 | AI     | Initial reverse-engineered design |
