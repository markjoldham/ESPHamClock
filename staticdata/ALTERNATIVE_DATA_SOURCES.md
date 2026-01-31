# Alternative Data Sources for HamClock Backend

Since the original backend server data is no longer available and was not archived, here are potential alternative data sources for each type of information HamClock needs.

## Space Weather Data

### Solar Flux (solarflux-99.txt)

**Alternative Sources:**

- **NOAA SWPC:** <https://services.swpc.noaa.gov/json/f107_cm_flux.json>
- **NOAA FTP:** ftp://ftp.swpc.noaa.gov/pub/indices/old_indices/
- **Space Weather Live:** <https://www.spaceweatherlive.com/en/solar-activity/solar-flux.html> (has API)

### Sunspot Number (ssn-31.txt)

**Alternative Sources:**

- **NOAA SWPC:** <https://services.swpc.noaa.gov/json/solar-cycle/observed-solar-cycle-indices.json>
- **SILSO (Royal Observatory of Belgium):** <https://www.sidc.be/SILSO/datafiles>
- **NOAA FTP:** ftp://ftp.swpc.noaa.gov/pub/indices/

### X-Ray Flux (xray.txt)

**Alternative Sources:**

- **NOAA SWPC Real-time:** <https://services.swpc.noaa.gov/json/goes/primary/xrays-6-hour.json>
- **NOAA SWPC 7-day:** <https://services.swpc.noaa.gov/json/goes/primary/xrays-7-day.json>

### Planetary K-Index (kindex.txt)

**Alternative Sources:**

- **NOAA SWPC:** <https://services.swpc.noaa.gov/json/planetary_k_index_1m.json>
- **GFZ Potsdam:** <https://www.gfz-potsdam.de/en/kp-index/>

### Disturbance Storm Time - DST (dst.txt)

**Alternative Sources:**

- **Kyoto University WDC:** <http://wdc.kugi.kyoto-u.ac.jp/dst_realtime/index.html>
- **NOAA SWPC:** <https://services.swpc.noaa.gov/products/kyoto-dst.json>

### Solar Wind (swind-24hr.txt)

**Alternative Sources:**

- **NOAA SWPC:** <https://services.swpc.noaa.gov/json/rtsw/rtsw_wind_1m.json>
- **NOAA SWPC Mag/Plasma:** <https://services.swpc.noaa.gov/products/solar-wind/>

### Magnetic Field Bz/Bt (Bz.txt)

**Alternative Sources:**

- **NOAA SWPC:** <https://services.swpc.noaa.gov/json/rtsw/rtsw_mag_1m.json>
- Included in solar wind data above

### Aurora (aurora.txt)

**Alternative Sources:**

- **NOAA SWPC Ovation:** <https://services.swpc.noaa.gov/json/ovation_aurora_latest.json>
- **NOAA Aurora 30-min forecast:** <https://services.swpc.noaa.gov/products/animations/ovation/>

### D-Region Absorption Prediction - DRAP (drap-stats.txt)

**Alternative Sources:**

- **NOAA SWPC:** <https://services.swpc.noaa.gov/products/animations/d-rap/>
- **NOAA SWPC Global D-RAP:** <https://services.swpc.noaa.gov/products/d-region-absorption-prediction/>

### NOAA Space Weather (noaaswx.txt)

**Alternative Sources:**

- **NOAA SWPC Alerts:** <https://services.swpc.noaa.gov/products/alerts.json>
- **NOAA SWPC Summary:** <https://services.swpc.noaa.gov/products/summary/>

## Weather Data

### World Weather (worldwx.txt)

**Alternative Sources:**

- **OpenWeatherMap API:** <https://openweathermap.org/api> (requires API key)
- **Weather.gov API:** <https://www.weather.gov/documentation/services-web-api>
- **NOAA NDFD:** <https://graphical.weather.gov/xml/>
- **Open-Meteo:** <https://open-meteo.com/> (free, no API key required)

### Location-specific Weather (wx.pl)

**Alternative Sources:**

- **OpenWeatherMap:** <https://openweathermap.org/current>
- **Weather.gov:** <https://api.weather.gov/points/{lat},{lon}>
- **Open-Meteo:** <https://api.open-meteo.com/v1/forecast>

## Ham Radio Resources

### Country/Prefix Database (cty_wt_mod-ll-dxcc.txt)

**Alternative Sources:**

- **Country Files:** <http://www.country-files.com/>
- **AD1C CTY Files:** <https://www.country-files.com/cty/>
- **K1USN CTY Files:** <http://www.k1usn.com/cty.html>

### Cities Database (cities2.txt)

**Alternative Sources:**

- **GeoNames:** <http://www.geonames.org/> (free database download)
- **Natural Earth Data:** <https://www.naturalearthdata.com/downloads/>
- **SimpleMaps:** <https://simplemaps.com/data/world-cities>

### Contest Calendar (contests311.txt)

**Alternative Sources:**

- **Contest Calendar:** <https://www.contestcalendar.com/>
- **WA7BNM Contest Calendar:** <https://www.contestcalendar.com/weeklycont.php>
- **SM3CER Contest Service:** <http://www.sk3bg.se/contest/>

### On The Air - POTA/SOTA (onta.txt)

**Alternative Sources:**

- **POTA API:** <https://api.pota.app/>
- **SOTA API:** <https://api2.sota.org.uk/>
- **POTA Spots:** <https://pota.app/#/>

### DXpeditions (dxpeditions.txt)

**Alternative Sources:**

- **NG3K DXpedition Calendar:** <https://www.ng3k.com/Misc/adxo.html>
- **DX World:** <https://dx-world.net/>
- **OPDX Bulletin:** <http://www.papays.com/opdx.html>

## Radio Propagation

### VOACAP Band Conditions (fetchBandConditions.pl)

**Alternative Sources:**

- **VOACAP Online:** <https://www.voacap.com/>
- **HamQSL Propagation:** <https://www.hamqsl.com/solar.html>
- **PSKReporter Propagation:** <https://pskreporter.info/pskmap.html>
- **Proppy:** <http://www.proppy.ws/>

## Utility Services

### IP Geolocation (fetchIPGeoloc.pl)

**Alternative Sources:**

- **ipapi.co:** <https://ipapi.co/json/> (free tier available)
- **ip-api.com:** <http://ip-api.com/json/> (free for non-commercial)
- **ipinfo.io:** <https://ipinfo.io/> (free tier available)
- **GeoJS:** <https://get.geojs.io/v1/ip/geo.json> (free)

### RSS Feeds (web15rss.pl)

**Alternative Sources:**

- **ARRL News:** <http://www.arrl.org/news/rss>
- **eHam.net:** <https://www.eham.net/rss/>
- **QRZ News:** <https://www.qrz.com/news/>
- **DX World:** <https://dx-world.net/feed/>

### Version Checking (version.pl)

**Alternative Sources:**

- Could be replaced with GitHub releases API if project moves to GitHub
- Or implement a simple version.json file hosted elsewhere

## Implementation Notes

### API Keys

Many modern APIs require registration and API keys:

- OpenWeatherMap: Free tier allows 1000 calls/day
- ipapi.co: Free tier allows 1000 requests/day
- Most NOAA services are free and don't require keys

### Data Format Conversion

The frontend code expects specific data formats. When implementing alternative sources:

1. Examine the parsing code in the relevant .cpp files
2. Create adapter/translation layers to convert new API responses to expected formats
3. Consider caching to reduce API calls and improve performance

### Rate Limiting

Implement appropriate rate limiting and caching:

- Space weather data: Update every 15-30 minutes
- Weather data: Update every 30-60 minutes
- Static data (cities, prefixes): Update daily or weekly
- Contest calendar: Update daily

### Hosting Options

A replacement backend could be hosted on:

- **Raspberry Pi** at home (simple Python/Node.js server)
- **Cloud services** (AWS Lambda, Google Cloud Functions, Azure Functions)
- **Free hosting** (Heroku free tier, Vercel, Netlify Functions)
- **VPS** (DigitalOcean, Linode, Vultr)

## Next Steps

1. **Reverse-engineer data formats** by analyzing the parsing code in src/
2. **Create API adapters** for each data source
3. **Build a replacement backend** using the alternative sources above
4. **Test with the existing frontend** to ensure compatibility
5. **Document the new backend** for community maintenance

## Legal Considerations

- Respect API terms of service and rate limits
- Attribute data sources as required
- Consider data licensing for redistribution
- NOAA data is public domain (US government)
- Some commercial APIs have restrictions on data usage
