# ESPHamClock External Write Operations Analysis

**Date:** January 31, 2026  
**Purpose:** Document all external write operations (POST, database writes, etc.)

## Summary

ESPHamClock performs **LIMITED** write operations to external services. The software is primarily a **read-only client** that fetches data from various sources. The few write operations that exist are:

1. **Optional diagnostic uploads** (user-initiated)
2. **DX cluster login** (read-only connection)
3. **Radio control commands** (local network only)

**No databases are written to.** All configuration and state is stored locally.

---

## 1. Diagnostic File Upload (Optional)

### What it does

Uploads diagnostic logs and configuration files to the backend server for troubleshooting.

### When it happens

- **User-initiated only** via menu: "Setup → Post diagnostics"
- **User-initiated only** via web API: `GET /postDiags`
- **Automatic on fatal errors** (crash diagnostics)

### What is sent

- Log files from the local system
- EEPROM configuration file (settings)
- Public IP address (used in filename for identification)

### Destination

```text
POST /ham/HamClock/diags/<public_ip>.txt HTTP/1.0
Host: clearskyinstitute.com:80
```

### Code location

`src/ESPHamClock.cpp` - `postDiags()` function (lines ~2190-2250)

### Privacy considerations

- Contains your callsign, location, and configuration settings
- Includes your public IP address
- Only sent when explicitly requested by user or on crash
- Can be disabled by not using the feature

### Can it be disabled?

Yes - simply don't use the "Post diagnostics" menu option. The automatic crash upload could be removed by modifying the code.

---

## 2. DX Cluster Connection (Read-Only)

### What it does

Connects to a DX cluster server to receive real-time amateur radio spot information.

### When it happens

- When DX cluster feature is enabled in settings
- Maintains persistent connection to receive spots

### What is sent

**Login sequence:**

```text
<callsign>-<SSID>
```

Example: `W1ABC-55`

**Optional commands:**

- Request recent spots: `sh/dx filter real 30` (DXSpider)
- Request recent spots: `show/dx/30 @` (AR-Cluster)
- Request recent spots: `show/myfdx` (VE7CC)

### What is NOT sent

- **No spot posting** - HamClock does not post spots to the cluster
- **No QSO logging** - No contact information is uploaded
- **No user data** - Only login and spot requests

### Destination

User-configurable DX cluster server (e.g., `dxc.nc7j.com:7373`)

### Code location

`src/dxcluster.cpp` - `dxcSendMsg()` function (lines ~360-375)

### Privacy considerations

- Your callsign is sent during login
- Connection is read-only (receive spots only)
- No personal information beyond callsign

### Can it be disabled?

Yes - disable DX cluster in settings or don't configure a cluster server.

---

## 3. Radio Control (Local Network Only)

### What it does

Sends frequency change commands to amateur radio transceivers via Hamlib or Flrig protocols.

### When it happens

- When user clicks on a DX spot to tune the radio
- Only if radio control is configured

### What is sent

**Hamlib protocol:**

```text
F <frequency_in_Hz>
```

Example: `F 14074000`

**Flrig protocol (XML-RPC):**

```xml
<?xml version="1.0"?>
<methodCall>
  <methodName>rig.set_frequency</methodName>
  <params>
    <param><value><double>14074000</double></value></param>
  </params>
</methodCall>
```

### Destination

- **Local network only** - typically `localhost:4532` (Hamlib) or `localhost:12345` (Flrig)
- Does not communicate with external servers
- Communicates with radio control software running on the same machine or local network

### Code location

`src/radio.cpp` - Hamlib and Flrig client functions

### Privacy considerations

- Local network communication only
- No data sent to external servers
- Only frequency change commands

### Can it be disabled?

Yes - don't configure radio control in settings.

---

## 4. Web Server (Incoming Only)

### What it does

HamClock includes a built-in web server that **receives** HTTP requests for remote control and monitoring.

### Direction

**INCOMING ONLY** - The web server listens for connections but does not initiate outbound writes to external services.

### What it provides

- Screen capture (BMP image)
- Status queries (time, location, DX info)
- Configuration changes
- Remote control commands

### Code location

`src/webserver.cpp`

### Privacy considerations

- Exposes information about your station (callsign, location, etc.)
- Should be firewalled if not needed
- Does not send data outbound

---

## 5. GPSD Client (Local Network Only)

### What it does

Connects to a local GPSD daemon to receive GPS position data.

### Direction

**READ-ONLY** - Sends watch command to enable reporting, then receives data.

### What is sent

```text
?WATCH={"enable":true,"json":true};?POLL;
```

### Destination

- **Local network only** - typically `localhost:2947`
- Does not communicate with external GPS services

### Code location

`src/gpsd.cpp`

---

## What HamClock Does NOT Do

### No Database Writes

- Does not write to MySQL, PostgreSQL, MongoDB, or any external database
- All configuration stored locally in EEPROM file

### No Cloud Services

- Does not upload data to cloud services
- Does not sync with external accounts
- Does not use OAuth or API keys (except for optional features like weather APIs)

### No Telemetry

- Does not send usage statistics
- Does not phone home (except optional diagnostic upload)
- Does not track user behavior

### No Social Media

- Does not post to Twitter, Facebook, etc.
- Does not share spots or QSOs publicly

### No QSO Logging Services

- Does not upload to QRZ.com, eQSL, LoTW, etc.
- Does not maintain online logbook

---

## Summary Table

| Operation              | Type      | Destination           | Frequency      | User Control |
| ---------------------- | --------- | --------------------- | -------------- | ------------ |
| Diagnostic Upload      | POST      | Backend server        | User-initiated | Optional     |
| DX Cluster Login       | TCP       | User-configured       | On connect     | Optional     |
| DX Cluster Commands    | TCP       | User-configured       | As needed      | Optional     |
| Radio Control (Hamlib) | TCP       | Local network         | On spot click  | Optional     |
| Radio Control (Flrig)  | TCP       | Local network         | On spot click  | Optional     |
| GPSD Watch             | TCP       | Local network         | On connect     | Optional     |
| Web Server             | HTTP      | Incoming only         | N/A            | Optional     |

---

## Privacy Assessment

### Low Privacy Risk

HamClock is designed as a **display and monitoring tool**, not a data collection or sharing platform.

**What is shared:**

- Callsign (to DX cluster during login)
- Public IP address (in diagnostic uploads, if used)
- Configuration settings (in diagnostic uploads, if used)

**What is NOT shared:**

- QSO logs
- Personal information beyond callsign
- Usage statistics
- Location data (except in diagnostic uploads)

### Recommendations

1. **Disable diagnostic uploads** if privacy is a concern
2. **Firewall the web server** if not needed for remote access
3. **Use a VPN** if connecting to public DX clusters
4. **Review DX cluster privacy policies** before connecting

---

## Conclusion

ESPHamClock is primarily a **read-only client** that fetches data from various sources. The limited write operations are:

- **Optional** (user-initiated or configurable)
- **Transparent** (documented in code and settings)
- **Privacy-conscious** (minimal data sharing)

**No databases are written to.** All state is stored locally. The software does not engage in telemetry, tracking, or cloud synchronization.

For a completely offline installation, disable:

- DX cluster connections
- Diagnostic uploads
- Web server (if not needed)

The software will continue to function with all display and monitoring features intact.
