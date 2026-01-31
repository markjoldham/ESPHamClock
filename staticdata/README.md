# Static Data Directory

## Purpose
This directory was created to capture static data files from the HamClock backend server at clearskyinstitute.com.

## Status: Backend Server Data Unavailable

**Date Checked:** January 31, 2026

The backend server at `clearskyinstitute.com` is still online and responding, but all the data endpoints that HamClock depends on are returning **404 Not Found** errors.

## Attempted Endpoints

The following endpoints were attempted but are no longer available:

### Space Weather Data
- `/Bz/Bz.txt` - Magnetic field Bz/Bt data
- `/solar-wind/swind-24hr.txt` - Solar wind information (24 hour)
- `/ssn/ssn-31.txt` - Sunspot numbers (31 day)
- `/solar-flux/solarflux-99.txt` - Solar flux data (99 day)
- `/drap/stats.txt` - D-Region Absorption Prediction statistics
- `/geomag/kindex.txt` - Planetary K-index
- `/xray/xray.txt` - X-ray flux data
- `/NOAASpaceWX/noaaswx.txt` - NOAA space weather
- `/NOAASpaceWX/rank2_coeffs.txt` - Space weather ranking coefficients
- `/aurora/aurora.txt` - Aurora data
- `/dst/dst.txt` - Disturbance Storm Time index

### Weather Data
- `/worldwx/wx.txt` - Gridded world weather table

### Ham Radio Resources
- `/cty/cty_wt_mod-ll-dxcc.txt` - Country/prefix database with lat/lng
- `/cities2.txt` - Cities database
- `/contests/contests311.txt` - Contest calendar
- `/ONTA/onta.txt` - "On The Air" activities (POTA/SOTA)
- `/dxpeds/dxpeditions.txt` - DXpedition information

### Dynamic Endpoints (CGI Scripts)
These were not attempted as they require parameters:
- `/fetchBandConditions.pl` - VOACAP band conditions/propagation models
- `/wx.pl` - Weather data for specific lat/lng coordinates
- `/fetchIPGeoloc.pl` - IP geolocation service
- `/version.pl` - Version checking for updates
- `/RSS/web15rss.pl` - RSS feed aggregator

## Implications

This confirms the warning in the main README.md: **"Much of the functionality of HamClock is provided by a back-end server, the code for which is/was owned by Elwood Downey."**

Without access to:
1. The backend server code
2. The data files
3. The CGI scripts that generate dynamic content

The HamClock frontend cannot function as originally designed.

## Path Forward

To preserve HamClock functionality, the community would need to:

1. **Recreate the backend services** by:
   - Reverse-engineering the expected data formats from the frontend parsing code
   - Finding alternative data sources for each type of information
   - Implementing replacement endpoints

2. **Modify the frontend** to:
   - Use alternative data sources directly
   - Implement fallback mechanisms when data is unavailable
   - Cache data locally when possible

3. **Document the data formats** by:
   - Analyzing the parsing code in the frontend
   - Creating specifications for each data file format
   - Documenting the expected API contracts

## Server Status
- **Domain:** clearskyinstitute.com
- **IP Address:** 72.167.43.150
- **Server:** Apache/2.4.41 (Ubuntu)
- **HTTPS:** Working
- **HTTP:** Returns empty responses
- **Data Endpoints:** All returning 404 Not Found

## Internet Archive / Wayback Machine Status

**Checked:** January 31, 2026

The Wayback Machine has archived the main clearskyinstitute.com website going back to 1998, and has numerous snapshots of the HamClock project page from 2017-2020. However:

- **None of the data endpoints were archived** - The Wayback Machine did not crawl the dynamic data files (.txt files) or CGI scripts (.pl files) that HamClock depends on
- **Main HamClock page:** Archived from September 2017 through at least August 2020
- **Data directories:** No snapshots found for /dst/, /xray/, /geomag/, /ssn/, /solar-flux/, /solar-wind/, /Bz/, /aurora/, /drap/, /NOAASpaceWX/, /worldwx/, /contests/, /ONTA/, /dxpeds/, /cty/, or cities2.txt

This is likely because:
1. The data files were dynamically generated and updated frequently
2. The Wayback Machine typically doesn't archive frequently-changing data files
3. The endpoints may have been excluded via robots.txt or other crawl restrictions
4. These were backend API endpoints not linked from the main website pages

**Conclusion:** The Wayback Machine cannot be used as a source to recover the backend data files.
