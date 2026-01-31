# Understanding rank2_coeffs.txt - Space Weather Ranking System

**Date:** January 31, 2026  
**Status:** ✅ **CAN BE REPLICATED**

## Overview

The `rank2_coeffs.txt` file contains quadratic coefficients used by HamClock to convert raw space weather values into comparable impact scores. This allows the software to rank and prioritize which space weather parameters are most important to display.

## Purpose

HamClock displays up to 4 space weather parameters in the NCDXF box. When set to "Auto" mode, it needs to determine which parameters are most impactful at any given time. The ranking system converts disparate measurements (sunspot numbers, X-ray flux, Kp index, etc.) into a unified impact score.

## The Formula

Each space weather parameter uses a quadratic formula:

```text
y = ax² + bx + c
```

Where:

- **x** = raw space weather value (e.g., Kp index = 7, Solar Flux = 150)
- **y** = impact ranking score (integer, roughly -10 to +5)
- **a, b, c** = coefficients specific to each parameter

Higher scores indicate greater impact/severity.

## Current Coefficients

From the downloaded `rank2_coeffs.txt`:

```text
# y = ax^2 + bx + c, where x = raw space weather value, y = small integer for ranking
# N.B. column 1 is SPCWX_t index and must be in this order.
#       a        b       c      // Parameter (example values => score)
0       0        0.05    -6      // Sunspot_N      60 => -3       200 => 4
1       0        1e6     -2      // X-Ray          (C)1e-6 => -2  (M)1e-5 => 8
2       0        0.1     -15     // Solar_Flux
3       0        3.2     -8.8    // Planetary_K    1 => -5.6   4 =>  4   9 => 20
4       0        1       -2      // Solar_Wind
5       0        1       -20     // DRAP
6       0        -0.8    -2      // Bz_Bt          0 => -2         -10 => 6
7       0        3       -3      // NOAA_SpcWx     0 => -3   1 => 0  3 => 6
8       0        0.16    -6      // AURORA         50 => 2         100 => 10
9   -0.04       -0.2      3      // DST           -10 => 1  0 => 3  5 => 1
```

## How It Works in HamClock

From `src/spacewx.cpp`:

```cpp
static int computeSWRank (const SpaceWeather_t *sp)
{
    return ((int) roundf ((sp->a * sp->value + sp->b) * sp->value + sp->c));
}
```

The software:

1. Fetches current values for all 10 space weather parameters
2. Applies the quadratic formula to each using its coefficients
3. Sorts parameters by score (highest = most impactful)
4. Displays the top 4 in the NCDXF box

## Coefficient Derivation

The coefficients are empirically tuned based on:

### NOAA Space Weather Scales

NOAA defines three scale types:

- **G-Scale (Geomagnetic Storms)**: G1 (Minor) to G5 (Extreme), based on Kp index
- **S-Scale (Solar Radiation Storms)**: S1 (Minor) to S5 (Extreme), based on particle flux
- **R-Scale (Radio Blackouts)**: R1 (Minor) to R5 (Extreme), based on X-ray flux

Reference: <https://www.swpc.noaa.gov/noaa-scales-explanation>

### Example: Planetary Kp Index

From NOAA scales:

- Kp = 5: G1 (Minor) - 1700 events per solar cycle
- Kp = 6: G2 (Moderate) - 600 events per cycle
- Kp = 7: G3 (Strong) - 200 events per cycle
- Kp = 8: G4 (Severe) - 100 events per cycle
- Kp = 9: G5 (Extreme) - 4 events per cycle

The coefficients `a=0, b=3.2, c=-8.8` map these to impact scores:

- Kp=1: y = 3.2(1) - 8.8 = -5.6 (low impact)
- Kp=4: y = 3.2(4) - 8.8 = 4.0 (moderate impact)
- Kp=9: y = 3.2(9) - 8.8 = 20.0 (extreme impact)

### Example: X-Ray Flux

X-ray flux is measured in watts/m². The R-scale uses:

- M1 (10⁻⁵): R1 (Minor)
- M5 (5×10⁻⁵): R2 (Moderate)
- X1 (10⁻⁴): R3 (Strong)
- X10 (10⁻³): R4 (Severe)
- X20 (2×10⁻³): R5 (Extreme)

The coefficients `a=0, b=1e6, c=-2` convert flux to score:

- C-class (1e-6): y = 1e6(1e-6) - 2 = -1 (low impact)
- M-class (1e-5): y = 1e6(1e-5) - 2 = 8 (high impact)

## How to Replicate

### Option 1: Static Copy (Recommended)

The simplest approach is to use the current coefficients as-is:

1. Copy the downloaded `rank2_coeffs.txt` file
2. Serve it as a static file from your replacement backend
3. These coefficients are stable and rarely need updating

**Pros:**

- Simple, no computation needed
- Preserves original HamClock behavior
- Coefficients are empirically tuned

**Cons:**

- No automatic updates if NOAA scales change

### Option 2: Dynamic Generation

Generate coefficients programmatically based on NOAA scales:

1. Define impact thresholds for each parameter (from NOAA scales)
2. Choose 2-3 calibration points (e.g., minor, moderate, extreme)
3. Fit a quadratic curve through those points
4. Output in rank2_coeffs.txt format

**Pros:**

- Can adapt to updated NOAA scales
- Transparent methodology

**Cons:**

- More complex implementation
- May not match original tuning exactly

### Option 3: Simplified Linear

Most coefficients have `a=0` (linear, not quadratic). You could simplify:

```text
y = bx + c
```

This makes the math easier while preserving most functionality.

## Implementation Example (Python)

```python
def generate_rank2_coeffs():
    """Generate rank2_coeffs.txt with current coefficients"""
    
    coeffs = [
        (0, 0,      0.05,   -6,     "Sunspot_N"),
        (1, 0,      1e6,    -2,     "X-Ray"),
        (2, 0,      0.1,    -15,    "Solar_Flux"),
        (3, 0,      3.2,    -8.8,   "Planetary_K"),
        (4, 0,      1,      -2,     "Solar_Wind"),
        (5, 0,      1,      -20,    "DRAP"),
        (6, 0,      -0.8,   -2,     "Bz_Bt"),
        (7, 0,      3,      -3,     "NOAA_SpcWx"),
        (8, 0,      0.16,   -6,     "AURORA"),
        (9, -0.04,  -0.2,   3,      "DST"),
    ]
    
    output = []
    output.append("# y = ax^2 + bx + c, where x = raw space weather value, y = small integer for ranking roughly -10..5")
    output.append("# N.B. column 1 is SPCWX_t index and must be in this order.")
    output.append("#       a        b       c")
    
    for idx, a, b, c, name in coeffs:
        output.append(f"{idx}       {a}        {b}       {c}              // {name}")
    
    return "\n".join(output)
```

## Recommendation

**Use the static copy approach.** The coefficients in rank2_coeffs.txt are:

- Empirically tuned for HamClock's display
- Stable (haven't changed in years)
- Based on well-established NOAA scales
- Simple to deploy

Simply include the current file in any replacement backend as a static resource. No need to regenerate or fetch from external sources.

## References

- **NOAA Space Weather Scales:** <https://www.swpc.noaa.gov/noaa-scales-explanation>
- **HamClock Source:** `src/spacewx.cpp` (initSWFit, computeSWRank functions)
- **Space Weather Impacts:** <https://www.weather.gov/wrn/winter-space-sm>

## Conclusion

The rank2_coeffs.txt file is **fully replicable** and should be treated as a **static configuration file** rather than dynamic data. It can be included directly in any HamClock backend replacement without modification.
