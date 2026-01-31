# How HamClock Fetches Bz.txt

## Source Code Location
File: `src/spacewx.cpp` (lines 975-1050)

## Backend Configuration
From `src/wifi.cpp`:
```cpp
const char *backend_host = "clearskyinstitute.com";
int backend_port = 80;
```

## Endpoint
```cpp
static const char bzbt_page[] = "/Bz/Bz.txt";
```

## HTTP Request
The software constructs this HTTP request:

```http
GET /ham/HamClock/Bz/Bz.txt HTTP/1.0
Host: clearskyinstitute.com
User-Agent: HamClock/<version>
Connection: close

```

## Equivalent curl Command
```bash
curl -H "User-Agent: HamClock" \
     http://clearskyinstitute.com/ham/HamClock/Bz/Bz.txt
```

Or with HTTPS:
```bash
curl -H "User-Agent: HamClock" \
     https://clearskyinstitute.com/ham/HamClock/Bz/Bz.txt
```

## Expected Data Format
The file should contain lines in this format:
```
# UNIX        Bx     By     Bz     Bt
1684087500    1.0   -2.7   -3.2    4.3
1684087800    0.8   -2.5   -3.0    4.1
...
```

Where:
- **UNIX**: Unix timestamp (seconds since epoch)
- **Bx**: Magnetic field X component (nT)
- **By**: Magnetic field Y component (nT)
- **Bz**: Magnetic field Z component (nT) - **this is the primary value used**
- **Bt**: Total magnetic field magnitude (nT)

## Parsing Logic
From `src/spacewx.cpp`:

```cpp
// Expected number of data points
#define BZBT_NV  (number of values expected)

// Parse each line
long unix;
float this_bz, this_bt;
if (sscanf(line, "%ld %*f %*f %f %f", &unix, &this_bz, &this_bt) != 3) {
    continue;  // Skip invalid lines
}

// Store values
bzbt_cache.bz[i] = this_bz;
bzbt_cache.bt[i] = this_bt;
bzbt_cache.x[i] = (unix - current_time) / 3600.0;  // Hours from now
```

## Key Points

1. **Path Construction**: The code prepends `/ham/HamClock` to all backend requests
   - Defined in `httpHCGET()` function
   - So `/Bz/Bz.txt` becomes `/ham/HamClock/Bz/Bz.txt`

2. **Data Validation**: 
   - Must receive all expected data points (BZBT_NV)
   - Latest data must be recent (within 15 minutes: `-0.25` hours)
   - If validation fails, data is marked as invalid

3. **Update Interval**: 
   - Defined as `BZBT_INTERVAL` in the code
   - Typically updates every 15-30 minutes

4. **Error Handling**:
   - Connection failures trigger retry with backoff
   - Invalid data is logged but doesn't crash the app
   - Old data is rejected if too stale

## Testing the Endpoint

Try fetching it yourself:
```bash
# Test if endpoint exists (currently returns 404)
curl -v https://clearskyinstitute.com/ham/HamClock/Bz/Bz.txt

# Expected result currently:
# HTTP 404 Not Found
```

## Alternative Data Source

NOAA SWPC provides equivalent data:
```bash
# Real-time magnetic field data (1-minute resolution)
curl https://services.swpc.noaa.gov/json/rtsw/rtsw_mag_1m.json
```

This returns JSON with Bx, By, Bz, Bt values that could be converted to the expected format.
