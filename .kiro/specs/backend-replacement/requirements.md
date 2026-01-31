# Requirements Document: ESPHamClock Backend Replacement

## Introduction

ESPHamClock is a C++ kiosk application for amateur radio operators that currently depends on a backend server at clearskyinstitute.com. The original author has passed away, creating concerns about long-term viability. This document specifies requirements for a replacement backend system that serves the same data from free public sources.

## Glossary

- **Backend_Proxy**: The replacement HTTP server that fetches data from public sources and converts it to HamClock's expected format
- **HamClock**: The ESPHamClock frontend application (client)
- **NOAA_SWPC**: National Oceanic and Atmospheric Administration Space Weather Prediction Center
- **Endpoint**: An HTTP URL path that serves a specific data file
- **Data_Source**: A free public API or service that provides raw data
- **Format_Converter**: Component that transforms JSON/HTML data to HamClock's text format
- **Cache**: Temporary storage of fetched data to minimize API calls
- **Space_Weather**: Solar and geomagnetic conditions affecting radio propagation
- **D-RAP**: D-Region Absorption Prediction (HF radio absorption forecast)
- **DXpedition**: Amateur radio expedition to a rare location
- **POTA**: Parks On The Air (portable operations program)
- **SOTA**: Summits On The Air (mountain operations program)

## Requirements

### Requirement 1: Space Weather Data Endpoints

**User Story:** As a HamClock user, I want to view real-time space weather data, so that I can understand current radio propagation conditions.

#### Acceptance Criteria

1. WHEN HamClock requests `/ham/HamClock/spacewx/Bz/Bz.txt`, THE Backend_Proxy SHALL fetch data from NOAA SWPC magnetic field API and return it in HamClock's text format
2. WHEN HamClock requests `/ham/HamClock/spacewx/swind/swind-24hr.txt`, THE Backend_Proxy SHALL fetch solar wind data from NOAA SWPC and return 24 hours of measurements
3. WHEN HamClock requests `/ham/HamClock/spacewx/xray/xray.txt`, THE Backend_Proxy SHALL fetch X-ray flux data from NOAA SWPC
4. WHEN HamClock requests `/ham/HamClock/spacewx/kindex/kindex.txt`, THE Backend_Proxy SHALL fetch planetary K-index data from NOAA SWPC
5. WHEN HamClock requests `/ham/HamClock/spacewx/dst/dst.txt`, THE Backend_Proxy SHALL fetch disturbance storm time data from NOAA SWPC
6. WHEN HamClock requests `/ham/HamClock/spacewx/solarflux/solarflux-99.txt`, THE Backend_Proxy SHALL fetch 99 days of solar flux data from NOAA SWPC
7. WHEN HamClock requests `/ham/HamClock/spacewx/ssn/ssn-31.txt`, THE Backend_Proxy SHALL fetch 31 days of sunspot number data from NOAA SWPC
8. WHEN HamClock requests `/ham/HamClock/spacewx/aurora/aurora.txt`, THE Backend_Proxy SHALL fetch aurora activity data from NOAA SWPC
9. WHEN HamClock requests `/ham/HamClock/spacewx/noaaswx/noaaswx.txt`, THE Backend_Proxy SHALL fetch NOAA space weather summary text
10. THE Backend_Proxy SHALL convert all NOAA JSON responses to space-delimited text format with UNIX timestamps

### Requirement 2: D-RAP Data Endpoint

**User Story:** As a HamClock user, I want to view D-region absorption predictions, so that I can assess HF radio propagation quality.

#### Acceptance Criteria

1. WHEN HamClock requests `/ham/HamClock/spacewx/drap/drap-stats.txt`, THE Backend_Proxy SHALL generate D-RAP estimates from X-ray flux data
2. WHEN generating D-RAP estimates, THE Backend_Proxy SHALL use X-ray flux as a proxy with empirical conversion formula
3. WHEN formatting D-RAP data, THE Backend_Proxy SHALL output lines in format "UNIX_TIMESTAMP : MIN MAX MEAN"
4. THE Backend_Proxy SHALL provide 24 hours of D-RAP history at 5-minute intervals

### Requirement 3: Static Data Files

**User Story:** As a HamClock user, I want access to ranking coefficients and prefix databases, so that the application can calculate space weather impact and lookup callsign locations.

#### Acceptance Criteria

1. WHEN HamClock requests `/ham/HamClock/spacewx/rank2_coeffs/rank2_coeffs.txt`, THE Backend_Proxy SHALL serve the static ranking coefficients file
2. WHEN HamClock requests `/ham/HamClock/prefixes/cty_wt_mod-ll-dxcc.txt`, THE Backend_Proxy SHALL serve the country/prefix database file
3. THE Backend_Proxy SHALL serve static files without modification from their original format

### Requirement 4: Contest Calendar Endpoint

**User Story:** As a HamClock user, I want to view upcoming amateur radio contests, so that I can plan my operating activities.

#### Acceptance Criteria

1. WHEN HamClock requests `/ham/HamClock/contests/contests311.txt`, THE Backend_Proxy SHALL fetch contest data from WA7BNM RSS feed
2. WHEN parsing contest data, THE Backend_Proxy SHALL extract contest title, start time, end time, and detail URL
3. WHEN formatting contest data, THE Backend_Proxy SHALL output first line as attribution "WA7BNM Contest Calendar"
4. WHEN formatting contest entries, THE Backend_Proxy SHALL output pairs of lines: "<start_unix> <end_unix> <title>" followed by "<url>"
5. THE Backend_Proxy SHALL convert contest dates to UNIX timestamps in UTC

### Requirement 5: DXpedition Data Endpoint

**User Story:** As a HamClock user, I want to view upcoming DXpeditions, so that I can track rare DX operations and plan contacts.

#### Acceptance Criteria

1. WHEN HamClock requests `/ham/HamClock/dxpeds/dxpeditions.txt`, THE Backend_Proxy SHALL fetch DXpedition data from NG3K ADXO page
2. WHEN parsing DXpedition data, THE Backend_Proxy SHALL extract start date, end date, location, callsign, and info URL
3. WHEN formatting DXpedition data, THE Backend_Proxy SHALL output first line as count of credit sources
4. WHEN formatting DXpedition data, THE Backend_Proxy SHALL output credit pairs (name line, URL line) before entries
5. WHEN formatting DXpedition entries, THE Backend_Proxy SHALL output CSV format: "<start_unix>,<end_unix>,<location>,<callsign>,<url>"

### Requirement 6: POTA/SOTA Data Endpoint

**User Story:** As a HamClock user, I want to view On The Air activations, so that I can find portable operations to contact.

#### Acceptance Criteria

1. WHEN HamClock requests `/ham/HamClock/onta/onta.txt`, THE Backend_Proxy SHALL fetch data from POTA and SOTA APIs
2. WHEN combining POTA and SOTA data, THE Backend_Proxy SHALL merge entries into a single list
3. THE Backend_Proxy SHALL format ONTA data in HamClock's expected text format with activation details

### Requirement 7: Data Caching

**User Story:** As a system administrator, I want the backend to cache data appropriately, so that we minimize API calls and respect rate limits.

#### Acceptance Criteria

1. WHEN fetching space weather data, THE Backend_Proxy SHALL cache responses for 5 minutes
2. WHEN fetching contest data, THE Backend_Proxy SHALL cache responses for 1 hour
3. WHEN fetching DXpedition data, THE Backend_Proxy SHALL cache responses for 24 hours
4. WHEN fetching ONTA data, THE Backend_Proxy SHALL cache responses for 1 hour
5. WHEN serving static files, THE Backend_Proxy SHALL cache them indefinitely until service restart
6. WHEN cache expires, THE Backend_Proxy SHALL fetch fresh data from source APIs
7. IF source API is unavailable, THE Backend_Proxy SHALL serve stale cached data with appropriate HTTP headers

### Requirement 8: HTTP Protocol Compatibility

**User Story:** As a HamClock user, I want the backend to work with HamClock's HTTP client, so that I can use the replacement backend without modifying HamClock.

#### Acceptance Criteria

1. THE Backend_Proxy SHALL listen on HTTP port 80 (not HTTPS)
2. THE Backend_Proxy SHALL serve all endpoints under path prefix `/ham/HamClock/`
3. WHEN HamClock sends HTTP/1.0 requests, THE Backend_Proxy SHALL respond with valid HTTP/1.0 or HTTP/1.1 responses
4. THE Backend_Proxy SHALL return Content-Type header "text/plain" for all data files
5. THE Backend_Proxy SHALL return HTTP 200 status for successful requests
6. THE Backend_Proxy SHALL return HTTP 404 status for unknown endpoints
7. THE Backend_Proxy SHALL return HTTP 503 status when data source is unavailable and no cached data exists

### Requirement 9: Text Format Accuracy

**User Story:** As a HamClock developer, I want data in the exact text format HamClock expects, so that parsing works correctly without code changes.

#### Acceptance Criteria

1. WHEN formatting space weather data, THE Backend_Proxy SHALL use space-delimited columns with consistent spacing
2. WHEN formatting timestamps, THE Backend_Proxy SHALL use UNIX epoch seconds (integer)
3. WHEN formatting floating-point values, THE Backend_Proxy SHALL use appropriate precision matching original format
4. WHEN formatting contest data, THE Backend_Proxy SHALL use exact line pairing (info line, URL line)
5. WHEN formatting DXpedition data, THE Backend_Proxy SHALL use comma-separated values with no spaces after commas
6. THE Backend_Proxy SHALL include comment lines (starting with #) where present in original format
7. THE Backend_Proxy SHALL use UTF-8 encoding for all text output

### Requirement 10: Error Handling

**User Story:** As a system administrator, I want the backend to handle errors gracefully, so that HamClock continues to function even when data sources are temporarily unavailable.

#### Acceptance Criteria

1. WHEN a data source API returns an error, THE Backend_Proxy SHALL log the error with timestamp and details
2. WHEN a data source API is unreachable, THE Backend_Proxy SHALL retry with exponential backoff
3. WHEN fresh data cannot be fetched, THE Backend_Proxy SHALL serve cached data if available
4. WHEN no cached data exists and source is unavailable, THE Backend_Proxy SHALL return HTTP 503 with error message
5. WHEN parsing fails for fetched data, THE Backend_Proxy SHALL log the error and serve cached data
6. THE Backend_Proxy SHALL continue serving other endpoints even if one data source fails

### Requirement 11: Logging and Monitoring

**User Story:** As a system administrator, I want comprehensive logging, so that I can troubleshoot issues and monitor backend health.

#### Acceptance Criteria

1. THE Backend_Proxy SHALL log all incoming HTTP requests with timestamp, path, and response status
2. THE Backend_Proxy SHALL log all outgoing API requests to data sources with timestamp and URL
3. THE Backend_Proxy SHALL log cache hits and misses for each endpoint
4. THE Backend_Proxy SHALL log errors with severity level (ERROR, WARNING, INFO)
5. THE Backend_Proxy SHALL provide a health check endpoint at `/health` returning JSON status
6. THE Backend_Proxy SHALL track and report cache hit ratio for each endpoint

### Requirement 12: Deployment and Configuration

**User Story:** As a system administrator, I want easy deployment and configuration, so that I can run the backend service reliably.

#### Acceptance Criteria

1. THE Backend_Proxy SHALL support configuration via environment variables
2. THE Backend_Proxy SHALL support configuration via configuration file (JSON or YAML)
3. THE Backend_Proxy SHALL allow configuration of cache durations per endpoint
4. THE Backend_Proxy SHALL allow configuration of HTTP listen port
5. THE Backend_Proxy SHALL provide a Docker container image for easy deployment
6. THE Backend_Proxy SHALL include systemd service file for Linux deployment
7. THE Backend_Proxy SHALL document all configuration options in README

### Requirement 13: Attribution and Licensing

**User Story:** As a backend operator, I want to properly attribute data sources, so that we respect terms of use and give credit to data providers.

#### Acceptance Criteria

1. THE Backend_Proxy SHALL include attribution comments in served data files where required by source terms
2. THE Backend_Proxy SHALL maintain a CREDITS file listing all data sources and their terms of use
3. THE Backend_Proxy SHALL include links back to original data sources in documentation
4. THE Backend_Proxy SHALL respect rate limits specified by data source APIs
5. THE Backend_Proxy SHALL include User-Agent header identifying itself in all API requests

### Requirement 14: Performance

**User Story:** As a HamClock user, I want fast response times, so that the application remains responsive.

#### Acceptance Criteria

1. WHEN serving cached data, THE Backend_Proxy SHALL respond within 100 milliseconds
2. WHEN fetching fresh data, THE Backend_Proxy SHALL respond within 5 seconds
3. THE Backend_Proxy SHALL handle at least 10 concurrent requests without degradation
4. THE Backend_Proxy SHALL use connection pooling for outgoing API requests
5. THE Backend_Proxy SHALL implement request timeouts of 30 seconds for data source APIs

### Requirement 15: Data Source Fallback

**User Story:** As a system administrator, I want fallback mechanisms, so that the service remains available even if primary data sources fail.

#### Acceptance Criteria

1. IF NOAA SWPC API is unavailable, THE Backend_Proxy SHALL serve cached data with stale-data warning header
2. IF NG3K ADXO page is unavailable, THE Backend_Proxy SHALL serve cached DXpedition data
3. IF WA7BNM RSS feed is unavailable, THE Backend_Proxy SHALL serve cached contest data
4. THE Backend_Proxy SHALL mark stale data with HTTP header "X-Data-Stale: true"
5. THE Backend_Proxy SHALL include "X-Cache-Age" header indicating seconds since data was fetched

### Requirement 16: Testing and Validation

**User Story:** As a developer, I want comprehensive testing, so that I can verify the backend works correctly with HamClock.

#### Acceptance Criteria

1. THE Backend_Proxy SHALL include unit tests for all format conversion functions
2. THE Backend_Proxy SHALL include integration tests for all endpoints
3. THE Backend_Proxy SHALL include test fixtures with sample data from each source API
4. THE Backend_Proxy SHALL validate output format against HamClock's parsing expectations
5. THE Backend_Proxy SHALL include end-to-end tests with actual HamClock client

### Requirement 17: Documentation

**User Story:** As a system administrator, I want clear documentation, so that I can deploy and maintain the backend service.

#### Acceptance Criteria

1. THE Backend_Proxy SHALL include README with installation instructions
2. THE Backend_Proxy SHALL document all supported endpoints and their data sources
3. THE Backend_Proxy SHALL document configuration options with examples
4. THE Backend_Proxy SHALL document troubleshooting procedures for common issues
5. THE Backend_Proxy SHALL include architecture diagram showing data flow
6. THE Backend_Proxy SHALL document API rate limits and caching strategy
