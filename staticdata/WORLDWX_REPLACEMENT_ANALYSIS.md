# World Weather Grid Replacement Analysis

**Date:** January 31, 2026  
**File:** worldwx.txt  
**Status:** ⚠️ **COMPLEX** - Requires significant adapter or feature deprecation

## What is worldwx.txt?

A global weather grid providing fast weather lookups for any location on Earth without making individual API calls.

### Current Format

```text
#   lat     lng  temp,C     %hum    mps     dir    mmHg    Wx           TZ
 -90    -180   -31.6    83.0     0.0    94.0  1002.0 Clouds          46800
 -86    -180   -28.2    86.0     4.6   134.0   997.0 Clouds          -43200
 -82    -180    -9.6    84.0     6.1   149.0   982.0 Clouds          -43200
...
```

**Fields:**

- **lat**: Latitude (-90 to 90)
- **lng**: Longitude (-180 to 180)
- **temp,C**: Temperature in Celsius
- **%hum**: Humidity percentage
- **mps**: Wind speed in meters per second
- **dir**: Wind direction in degrees
- **mmHg**: Pressure in millibars (despite header saying mmHg)
- **Wx**: Weather conditions (text: "Clear", "Clouds", "Rain", "Snow")
- **TZ**: Timezone offset in seconds from UTC

**Grid Structure:**

- **Dimensions**: 3433 lines = ~46 latitudes × ~72 longitudes
- **Latitude spacing**: 4° steps (46 points from -90 to 90)
- **Longitude spacing**: 5° steps (72 points from -180 to 175)
- **Coverage**: Global
- **Update frequency**: Every 45 minutes

### How HamClock Uses It

From `src/wx.cpp`:

1. **Downloads entire grid** into memory as 2D table (`wwt.table`)
2. **Parses format**: `sscanf (line, "%g %g %g %g %g %g %g %31s %d", &lat, &lng, &temp, &hum, &wind_speed, &windir, &pressure, conditions, &timezone)`
3. **Validates regular spacing** - confirms constant lat/lng steps
4. **Stores in memory** as array of `WXInfo` structures
5. **Fast lookups** via `findWXFast(lat, lng)` - O(1) grid index calculation

**Used by:**

- **Roaming cursor weather display** - Shows weather when hovering over map locations
- **Timezone lookups** - `getFastTZ()` for any lat/lng
- **Info boxes** - Quick weather display for arbitrary locations

**Key advantage:** No API calls needed for cursor hover - instant weather for any location.

---

## Data Structure

```cpp
typedef struct {
    char city[32];              // Not used in grid (empty)
    float temperature_c;        // Temperature in Celsius
    float humidity_percent;     // Humidity 0-100
    float pressure_hPa;         // Pressure in hectopascals
    float wind_speed_mps;       // Wind speed meters/sec
    char wind_dir_name[4];      // "N", "NE", "E", etc. (converted from degrees)
    char clouds[32];            // Not used in grid
    char conditions[32];        // "Clear", "Clouds", "Rain", "Snow"
    char attribution[32];       // Not used in grid
    int8_t pressure_chg;        // Not used in grid
    int timezone;               // Seconds from UTC
} WXInfo;
```

---

## Alternative Data Sources

### Option 1: Open-Meteo API (Free, No Key Required)

**URL:** `https://api.open-meteo.com/v1/forecast`

**Capabilities:**

- Global coverage
- Free tier: 10,000 API calls/day
- No API key required
- Supports bulk requests (multiple lat/lng in one call)

**Example request:**

```text
GET https://api.open-meteo.com/v1/forecast?latitude=40.7&longitude=-74.0&current=temperature_2m,relative_humidity_2m,wind_speed_10m,wind_direction_10m,weather_code,pressure_msl
```

**Response format:** JSON

```json
{
  "current": {
    "time": "2026-01-31T12:00",
    "temperature_2m": 15.2,
    "relative_humidity_2m": 65,
    "wind_speed_10m": 5.4,
    "wind_direction_10m": 180,
    "weather_code": 3,
    "pressure_msl": 1013.2
  }
}
```

**Weather codes:** WMO standard (0=clear, 1-3=clouds, 51-67=rain, 71-77=snow, etc.)

**Challenges:**

1. **Grid generation required** - Must make ~3400 API calls to build full grid
2. **Rate limiting** - 10,000 calls/day = can update grid ~3 times/day
3. **Timezone data** - Open-Meteo doesn't provide timezone offsets
4. **Format conversion** - JSON to text, weather codes to names

**Feasibility:** ⚠️ **MODERATE** - Possible but requires significant adapter logic

---

### Option 2: Weather.gov API (Free, US Only)

**URL:** `https://api.weather.gov/`

**Limitations:**

- **US coverage only** - Cannot provide global grid
- Not suitable for HamClock's worldwide audience

**Feasibility:** ❌ **NOT SUITABLE** - Geographic limitation

---

### Option 3: OpenWeatherMap (Requires API Key)

**URL:** `https://api.openweathermap.org/data/2.5/weather`

**Limitations:**

- **Requires API key** - Not free/anonymous
- Free tier: 1,000 calls/day (insufficient for 3400-point grid)
- Paid tier required for bulk access

**Feasibility:** ❌ **NOT SUITABLE** - Violates "no API key" requirement

---

### Option 4: Static/Cached Grid

Use a pre-generated weather grid with typical/average values.

**Approach:**

- Generate grid once with real data
- Serve as static file
- Update periodically (daily/weekly) via manual process

**Pros:**

- Simple to implement
- No API rate limits
- Works offline

**Cons:**

- Not real-time weather
- Misleading to users
- Defeats purpose of weather display

**Feasibility:** ⚠️ **POSSIBLE** - But compromises feature value

---

### Option 5: Deprecate Feature

Remove world weather grid functionality from HamClock.

**Impact analysis:**

**Features affected:**

1. **Roaming cursor weather** - Would no longer show weather on hover
2. **Fast timezone lookups** - Would need alternative TZ source
3. **Info box weather** - Would need per-location API calls

**Workarounds:**

1. **Timezone data** - Use separate timezone database (e.g., timezone-boundary-builder)
2. **Cursor weather** - Disable or make API call on demand (slower)
3. **Info boxes** - Use cached DE/DX weather only

**User impact:**

- Loss of convenience feature (hover weather)
- Slightly slower timezone lookups
- Core functionality (DE/DX weather) unaffected

**Feasibility:** ✅ **VIABLE** - Feature is convenience, not critical

---

## Recommendation

### Hybrid Approach: Deprecate Grid + Separate Timezone Database

**Rationale:**

1. **World weather grid is a convenience feature**, not core functionality
2. **No free API can efficiently provide 3400-point global grid** without rate limits
3. **Timezone data can be sourced separately** from dedicated TZ databases
4. **Core weather features** (DE/DX locations) use different endpoints and remain functional

### Implementation Plan

#### Phase 1: Replace Timezone Lookups

**Use:** Timezone boundary database (GeoJSON format)

**Source:** `https://github.com/evansiroky/timezone-boundary-builder`

**Approach:**

1. Download timezone boundary GeoJSON (one-time, ~50MB)
2. Implement point-in-polygon lookup for lat/lng → timezone
3. Replace `getFastTZ()` with new lookup function
4. Cache results for performance

**Pros:**

- Accurate timezone data
- No API calls needed
- Works offline
- One-time download

**Cons:**

- Larger file size (~50MB vs ~1.3MB)
- More complex lookup (point-in-polygon vs grid index)

#### Phase 2: Modify Cursor Weather Behavior

**Option A: Disable cursor weather**

- Remove weather display on map hover
- Show only lat/lng coordinates

**Option B: On-demand API calls**

- Make API call when user hovers (with debounce)
- Cache recent lookups
- Show "Loading..." indicator

**Option C: Show DE/DX weather only**

- Only display weather for configured DE and DX locations
- No arbitrary location weather

**Recommendation:** **Option A** - Simplest, least confusing

#### Phase 3: Update Documentation

- Explain that world weather grid is no longer available
- Document timezone database source
- Update user guide

---

## Alternative: Open-Meteo Grid Generator

If maintaining the grid feature is critical, implement a backend service that:

### Architecture

```text
┌─────────────────┐
│   HamClock      │
│   (client)      │
└────────┬────────┘
         │ GET /worldwx/wx.txt
         │
┌────────▼────────┐
│  Backend Proxy  │
│  (Python/Node)  │
└────────┬────────┘
         │ Bulk API calls
         │
┌────────▼────────┐
│  Open-Meteo API │
│  (free tier)    │
└─────────────────┘
```

### Backend Service Logic

1. **Cache grid in memory** - Refresh every 45 minutes
2. **Bulk API calls** - Request multiple points per call (Open-Meteo supports this)
3. **Rate limit management** - Spread 3400 calls over 45 minutes (~1.3 calls/sec)
4. **Format conversion** - JSON → text format HamClock expects
5. **Timezone lookup** - Use separate TZ database for timezone field

### Implementation Complexity

**Estimated effort:** 2-3 days

**Components:**

- Backend service (Python Flask or Node Express)
- Open-Meteo API client
- Timezone database integration
- Grid caching logic
- Format converter
- Rate limiter

**Pros:**

- Maintains full feature compatibility
- Uses free API
- Transparent to HamClock

**Cons:**

- Requires running backend service
- More complex than deprecation
- Dependent on Open-Meteo availability
- Rate limits may cause delays

---

## Comparison Matrix

| Approach                  | Complexity | Cost | Feature Loss | Accuracy | Offline |
| ------------------------- | ---------- | ---- | ------------ | -------- | ------- |
| Open-Meteo Grid Generator | High       | Free | None         | Good     | No      |
| Static/Cached Grid        | Low        | Free | Real-time    | Poor     | Yes     |
| Deprecate + TZ Database   | Moderate   | Free | Cursor WX    | N/A      | Yes     |
| OpenWeatherMap            | High       | Paid | None         | Good     | No      |

---

## Final Recommendation

### Deprecate World Weather Grid + Add Timezone Database

**Justification:**

1. **No free API can efficiently provide global weather grid** without significant complexity
2. **Cursor weather is a convenience feature**, not core functionality
3. **Timezone data is more critical** and can be sourced from dedicated databases
4. **Simplifies backend replacement** - One less complex file to replicate
5. **Core weather features remain functional** - DE/DX weather uses different endpoints

### Implementation Steps

1. **Download timezone boundary database** (GeoJSON format)
2. **Implement point-in-polygon TZ lookup** to replace `getFastTZ()`
3. **Modify cursor behavior** to show lat/lng only (no weather)
4. **Update documentation** explaining the change
5. **Test thoroughly** to ensure TZ lookups work correctly

### Alternative Path

If cursor weather is deemed critical:

1. **Implement Open-Meteo grid generator** as backend service
2. **Accept rate limiting constraints** (grid updates every 45 min may be delayed)
3. **Monitor API availability** and have fallback to deprecation if service becomes unreliable

---

## Conclusion

**Best approach:** Deprecate world weather grid, replace timezone lookups with dedicated TZ database.

**Rationale:** The world weather grid is a convenience feature that cannot be efficiently replicated with free APIs. The timezone functionality (more critical) can be preserved using dedicated timezone databases. This approach balances feature preservation with implementation complexity.

**If grid must be preserved:** Implement Open-Meteo backend service with bulk API calls and caching, accepting the complexity and rate limit constraints.
