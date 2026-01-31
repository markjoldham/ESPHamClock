# Data Source Comparison Analysis

**Date:** January 31, 2026  
**Purpose:** Compare HamClock backend data with free public sources (no API keys required)

## Summary

This document compares each HamClock backend data file with publicly available free sources to determine if the software can be modified to use alternative sources.

## Methodology

1. Examine HamClock's expected data format from parsing code
2. Test free public data sources (no API keys)
3. Compare data formats and content
4. Assess feasibility of creating adapters

## Data Files Analysis

### 1. Bz.txt - Magnetic Field Data

**HamClock Format:**

```text
# UNIX        Bx     By     Bz     Bt
1769766240    2.0   -1.9   -0.2    2.7
1769766840    1.3   -2.2    1.8    3.2
```

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/json/rtsw/rtsw_mag_1m.json`
- **Format:** JSON with fields: `time_tag`, `bx_gsm`, `by_gsm`, `bz_gsm`, `bt`
- **Update Frequency:** 1-minute resolution
- **Match:** ✅ **EXCELLENT** - Same data, different format
- **Conversion:** Simple - parse JSON, convert ISO timestamp to UNIX, output as space-delimited

**Recommendation:** **REPLACE** - Easy conversion, reliable source

---

### 2. swind-24hr.txt - Solar Wind Data

**HamClock Expected:** Solar wind speed, density, temperature over 24 hours

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/json/rtsw/rtsw_wind_1m.json`
- **Format:** JSON with `time_tag`, `proton_speed`, `proton_density`, `proton_temperature`
- **Match:** ✅ **EXCELLENT**

**Recommendation:** **REPLACE**

---

### 3. xray.txt - X-Ray Flux

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/json/goes/primary/xrays-6-hour.json`
- **Format:** JSON with `time_tag`, `flux` (short and long wavelength)
- **Match:** ✅ **EXCELLENT**

**Recommendation:** **REPLACE**

---

### 4. kindex.txt - Planetary K-Index

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/json/planetary_k_index_1m.json`
- **Format:** JSON with `time_tag`, `kp_index`
- **Match:** ✅ **EXCELLENT**

**Recommendation:** **REPLACE**

---

### 5. dst.txt - Disturbance Storm Time

**Alternative Sources:**

- **Kyoto University:** <http://wdc.kugi.kyoto-u.ac.jp/dst_realtime/index.html>
- **NOAA:** <https://services.swpc.noaa.gov/products/kyoto-dst.json>
- **Match:** ✅ **GOOD** - NOAA provides JSON format

**Recommendation:** **REPLACE**

---

### 6. solarflux-99.txt - Solar Flux (99 days)

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/json/f107_cm_flux.json`
- **Format:** JSON with date and flux values
- **Match:** ✅ **EXCELLENT**

**Recommendation:** **REPLACE**

---

### 7. ssn-31.txt - Sunspot Number (31 days)

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/json/solar-cycle/observed-solar-cycle-indices.json`
- **Format:** JSON with date and sunspot number
- **Match:** ✅ **EXCELLENT**

**Recommendation:** **REPLACE**

---

### 8. aurora.txt - Aurora Activity

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/json/ovation_aurora_latest.json`
- **Format:** JSON with aurora forecast data
- **Match:** ✅ **GOOD**

**Recommendation:** **REPLACE**

---

### 9. drap-stats.txt - D-Region Absorption Prediction

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/products/animations/d-rap/`
- **Format:** Various formats available
- **Match:** ⚠️ **MODERATE** - May require parsing HTML or different format

**Recommendation:** **EVALUATE** - Needs investigation

---

### 10. noaaswx.txt - NOAA Space Weather Summary

**NOAA SWPC Alternative:**

- **URL:** `https://services.swpc.noaa.gov/products/summary/`
- **Format:** Text summary
- **Match:** ✅ **GOOD**

**Recommendation:** **REPLACE**

---

### 11. rank2_coeffs.txt - Space Weather Ranking Coefficients

**Status:** ✅ **CAN BE REPLICATED**

**What it does:**

This file contains quadratic coefficients (a, b, c) for the formula `y = ax² + bx + c` where:

- `x` = raw space weather value (SSN, X-Ray flux, Solar Flux, Kp, etc.)
- `y` = integer ranking score (roughly -10 to +5) indicating impact severity

The coefficients convert each space weather parameter into a comparable impact score for sorting and display priority.

**File format:**

```text
# y = ax^2 + bx + c
# Column 1 is SPCWX_t index (must be in order)
#       a        b       c
0       0        0.05    -6              // Sunspot_N
1       0        1e6     -2              // X-Ray
2       0        0.1     -15             // Solar_Flux
3       0        3.2     -8.8            // Planetary_K
4       0        1       -2              // Solar_Wind
5       0        1       -20             // DRAP
6       0        -0.8    -2              // Bz_Bt
7       0        3       -3              // NOAA_SpcWx
8       0        0.16    -6              // AURORA
9   -0.04       -0.2      3              // DST
```

**How to replicate:**

These coefficients can be derived from [NOAA Space Weather Scales](<https://www.swpc.noaa.gov/noaa-scales-explanation>) which define impact thresholds for G-scale (geomagnetic), S-scale (solar radiation), and R-scale (radio blackout) events. The quadratic formula maps raw values to impact rankings.

**Replication approach:**

1. **Use NOAA scale thresholds** as calibration points (e.g., Kp=5 is G1-Minor, Kp=9 is G5-Extreme)
2. **Fit quadratic curves** through known impact points for each parameter
3. **Static file approach**: Copy the current rank2_coeffs.txt as a static file (coefficients rarely change)
4. **Dynamic approach**: Calculate coefficients programmatically based on NOAA scale definitions

**Recommendation:** **STATIC COPY** - The coefficients are empirically tuned and stable. Simply include the current rank2_coeffs.txt file as a static resource in any replacement backend. The formula is custom to HamClock's display logic and doesn't need frequent updates.

---

### 12. worldwx.txt - World Weather Grid

**Alternative Sources:**

- **Open-Meteo:** `https://open-meteo.com/` (Free, no API key)
- **Weather.gov:** `https://api.weather.gov/` (US only, free)
- **Match:** ⚠️ **MODERATE** - Different format, would need significant adapter

**Recommendation:** **COMPLEX** - Major refactoring needed

---

### 13. cities2.txt - Cities Database

**Alternative Sources:**

- **GeoNames:** <http://www.geonames.org/> (Free database download)
- **Natural Earth:** <https://www.naturalearthdata.com/downloads/>
- **Match:** ✅ **EXCELLENT** - Standard city databases available

**Recommendation:** **REPLACE** - One-time download and conversion

---

### 14. contests311.txt - Contest Calendar

**Alternative Sources:**

- **WA7BNM:** `https://www.contestcalendar.com/weeklycont.php`
- **Match:** ⚠️ **MODERATE** - Would need web scraping or different format

**Recommendation:** **EVALUATE** - May need permission or different approach

---

### 15. onta.txt - On The Air (POTA/SOTA)

**Alternative Sources:**

- **POTA API:** `https://api.pota.app/` (Free, no key required)
- **SOTA API:** `https://api2.sota.org.uk/` (Free)
- **Match:** ✅ **EXCELLENT** - Official APIs available

**Recommendation:** **REPLACE** - Use official APIs

---

### 16. dxpeditions.txt - DXpedition Information

**Alternative Sources:**

- **NG3K:** `https://www.ng3k.com/Misc/adxo.html`
- **Match:** ⚠️ **MODERATE** - Would need web scraping

**Recommendation:** **EVALUATE** - Check terms of service

---

### 17. cty_wt_mod-ll-dxcc.txt - Country/Prefix Database

**Alternative Sources:**

- **Country Files:** `http://www.country-files.com/`
- **AD1C:** `https://www.country-files.com/cty/`
- **Match:** ✅ **EXCELLENT** - Standard format, regularly updated

**Recommendation:** **REPLACE** - Use community-maintained files

---

## Summary Matrix

| File                   | Free Source Available | Match Quality | Recommendation |
| ---------------------- | --------------------- | ------------- | -------------- |
| Bz.txt                 | ✅ NOAA               | Excellent     | **REPLACE**    |
| swind-24hr.txt         | ✅ NOAA               | Excellent     | **REPLACE**    |
| xray.txt               | ✅ NOAA               | Excellent     | **REPLACE**    |
| kindex.txt             | ✅ NOAA               | Excellent     | **REPLACE**    |
| dst.txt                | ✅ NOAA               | Good          | **REPLACE**    |
| solarflux-99.txt       | ✅ NOAA               | Excellent     | **REPLACE**    |
| ssn-31.txt             | ✅ NOAA               | Excellent     | **REPLACE**    |
| aurora.txt             | ✅ NOAA               | Good          | **REPLACE**    |
| drap-stats.txt         | ⚠️ NOAA               | Moderate      | EVALUATE       |
| noaaswx.txt            | ✅ NOAA               | Good          | **REPLACE**    |
| rank2_coeffs.txt       | ✅ Static Copy        | Excellent     | **STATIC**     |
| worldwx.txt            | ⚠️ Open-Meteo         | Moderate      | COMPLEX        |
| cities2.txt            | ✅ GeoNames           | Excellent     | **REPLACE**    |
| contests311.txt        | ⚠️ WA7BNM             | Moderate      | EVALUATE       |
| onta.txt               | ✅ POTA/SOTA          | Excellent     | **REPLACE**    |
| dxpeditions.txt        | ⚠️ NG3K               | Moderate      | EVALUATE       |
| cty_wt_mod-ll-dxcc.txt | ✅ Country Files      | Excellent     | **REPLACE**    |

## Implementation Priority

### Phase 1: Easy Wins (11 files)

All space weather data from NOAA SWPC - straightforward JSON to text conversion:

1. Bz.txt
2. swind-24hr.txt
3. xray.txt
4. kindex.txt
5. dst.txt
6. solarflux-99.txt
7. ssn-31.txt
8. aurora.txt
9. noaaswx.txt
10. cities2.txt (one-time conversion)
11. cty_wt_mod-ll-dxcc.txt (use existing community file)
12. onta.txt (POTA/SOTA APIs)

### Phase 2: Moderate Effort (3 files)

Require more investigation or complex adapters:

1. drap-stats.txt
2. contests311.txt
3. dxpeditions.txt

### Phase 3: Complex (1 file)

Require significant refactoring:

1. worldwx.txt (major adapter needed)

## Next Steps

1. **Create adapter layer** - Build a backend proxy that:
   - Fetches from NOAA/public sources
   - Converts JSON to HamClock's expected text format
   - Caches data to reduce API calls
   - Serves data on same endpoints HamClock expects

2. **Modify HamClock** - Update `backend_host` to point to new proxy

3. **Test thoroughly** - Ensure all features work with new data sources

4. **Document** - Create setup guide for running the proxy

## Conclusion

**14 out of 17 files** can be easily replaced with free public sources or static copies, primarily from NOAA SWPC. This represents ~82% of the backend functionality. The remaining files either need investigation or represent features that may need to be deprecated or significantly refactored.

The path forward is clear: create a simple backend proxy service that translates NOAA's JSON APIs into HamClock's expected text format, plus serve static files for rank2_coeffs.txt.
