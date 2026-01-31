# DXpeditions Replacement Analysis

**Date:** January 31, 2026  
**File:** dxpeditions.txt  
**Status:** ✅ **VIABLE** - Can use NG3K ADXO page with web scraping or contact for permission

## What is dxpeditions.txt?

A list of upcoming and active DXpeditions (amateur radio expeditions to rare locations).

### Current Format

```text
2
DXNews
https://dxnews.com
NG3K
https://www.ng3k.com/Misc/adxo.html
1771632000,1771804799,Belize,V3T,https://dxnews.com/belize_v31tp/
1771459200,1772236799,Guyana,8R1WA,https://guyana2026.wordpress.com/
1769904000,1772236799,Belize,V31DJ,https://www.qrz.com/db/V31DJ
```

**Format:**

- **Line 1**: Number of credit sources (e.g., "2")
- **Lines 2-N**: Credit pairs (name + URL for each source)
  - Name line (e.g., "DXNews")
  - URL line (e.g., "https://dxnews.com")
- **Remaining lines**: DXpedition entries in CSV format
  - `<start_unix>,<end_unix>,<location>,<callsign>,<info_url>`

**Fields:**

- **start_unix**: Start time (UNIX timestamp, UTC)
- **end_unix**: End time (UNIX timestamp, UTC)
- **location**: Geographic location (e.g., "Belize", "Guyana")
- **callsign**: DXpedition callsign (e.g., "V3T", "8R1WA")
- **info_url**: Link to DXpedition information page

**Update frequency:** Every 24 hours (DXPEDS_MAXAGE = 86400 seconds)

### How HamClock Uses It

From `src/dxpeds.cpp`:

1. **Downloads file** via `/dxpeds/dxpeditions.txt` endpoint
2. **Caches locally** as `dxpeditions.txt` (24-hour cache)
3. **Parses format**:
   - First line = number of credits
   - Next N pairs = credit name + URL
   - Remaining lines = CSV DXpedition entries
4. **Filters**:
   - Removes past expeditions (end_t < now)
   - Optional: Show only active expeditions (start_t < now < end_t)
   - Optional: Hide user-marked expeditions
5. **Displays in pane** with scrolling list
6. **Highlights active expeditions** - Green background if currently active
7. **Highlights spotted expeditions** - Red if spotted on DX cluster
8. **Maps markers** - Shows expedition locations on map
9. **DX cluster integration** - Watches for spots matching expedition callsigns
10. **ADIF integration** - Marks expeditions as "worked" based on logbook

**Features:**

- Scrollable list of upcoming/active DXpeditions
- Toggle date display (show/hide start/end times)
- Toggle "current only" (show only active expeditions)
- Toggle "show hidden" (show user-hidden expeditions)
- Watch DX cluster for expedition spots
- Mark expeditions as hidden
- Open expedition info page in browser
- Set DX location to expedition
- Map markers for expedition locations

---

## Data Sources

### Primary Source: NG3K ADXO (Announced DX Operations)

**Website:** https://www.ng3k.com/Misc/adxo.html

**Owner:** Bill Feidt, NG3K

**Purpose:** Comprehensive list of announced DX operations and DXpeditions

**Format:** HTML table with structured data

**Update frequency:** Updated regularly (last updated timestamp shown on page)

**Data fields:**

- Start Date
- End Date
- DXCC Entity (location)
- Call (callsign)
- QSL via
- Reported by
- Info (details and links)

**Example entry:**

```text
2026 Jan 30 - 2026 Feb 03
St Kitts & Nevis
V47JA
LoTW
W5JON
By W5JON fm Calypso Bay; 160-6m; SSB FT8; yagi, verticals; QSL also OK via W5JON direct
```

**Advantages:**

- ✅ Comprehensive and authoritative
- ✅ Regularly updated
- ✅ Free access
- ✅ Structured HTML (parseable)
- ✅ Includes detail links

**Challenges:**

- ⚠️ HTML format (requires web scraping)
- ⚠️ No explicit API or RSS feed
- ⚠️ Terms of use unclear for automated access

---

### Secondary Source: DXNews.com

**Website:** https://dxnews.com/dxpeditions/

**Purpose:** DXpedition news and announcements

**Format:** WordPress blog with DXpedition posts

**RSS Feed:** Available (https://www.portcars.org/wp/dxnews-rss-feed/ mentions DXNews RSS)

**Advantages:**

- ✅ RSS feed available
- ✅ Detailed expedition information
- ✅ Free access

**Challenges:**

- ⚠️ Blog format (not structured list)
- ⚠️ May not include all expeditions
- ⚠️ Requires parsing blog posts for dates/callsigns
- ⚠️ Less comprehensive than NG3K

---

## Replacement Options

### Option 1: Web Scrape NG3K ADXO Page (RECOMMENDED)

**Approach:** Parse HTML from NG3K ADXO page to extract DXpedition data

**Implementation:**

1. **Fetch HTML** from https://www.ng3k.com/Misc/adxo.html
2. **Parse table** to extract DXpedition entries
3. **Convert dates** to UNIX timestamps
4. **Extract callsigns and locations**
5. **Find info URLs** from "Info" column
6. **Format as dxpeditions.txt**

**Parsing strategy:**

```python
import requests
from bs4 import BeautifulSoup
from datetime import datetime
import re

def fetch_ng3k_dxpeditions():
    """
    Scrape NG3K ADXO page and convert to dxpeditions.txt format
    """
    url = "https://www.ng3k.com/Misc/adxo.html"
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # Find the main table
    table = soup.find('table')  # Adjust selector as needed
    
    dxpeds = []
    
    for row in table.find_all('tr')[1:]:  # Skip header
        cols = row.find_all('td')
        if len(cols) < 6:
            continue
        
        # Extract fields
        start_date = cols[0].text.strip()
        end_date = cols[1].text.strip()
        location = cols[2].text.strip()
        callsign = cols[3].text.strip()
        info = cols[5].text.strip()
        
        # Parse dates to UNIX timestamps
        start_unix = parse_date_to_unix(start_date)
        end_unix = parse_date_to_unix(end_date)
        
        # Extract URL from info column (if present)
        info_url = extract_url_from_info(info) or f"https://www.ng3k.com/Misc/adxo.html"
        
        # Add to list
        dxpeds.append(f"{start_unix},{end_unix},{location},{callsign},{info_url}")
    
    return dxpeds

def parse_date_to_unix(date_str):
    """
    Convert "2026 Jan 30" to UNIX timestamp
    """
    # Parse format: "2026 Jan 30"
    dt = datetime.strptime(date_str, "%Y %b %d")
    return int(dt.timestamp())

def extract_url_from_info(info_text):
    """
    Extract URL from info text if present
    """
    # Look for URLs in info text
    url_match = re.search(r'https?://[^\s]+', info_text)
    return url_match.group(0) if url_match else None
```

**Output format:**

```text
1
NG3K
https://www.ng3k.com/Misc/adxo.html
1769817600,1769903940,St Kitts & Nevis,V47JA,https://www.ng3k.com/Misc/adxo.html
1771632000,1771804799,Belize,V3T,https://www.ng3k.com/Misc/adxo.html
...
```

**Pros:**

- ✅ Comprehensive data source
- ✅ Regularly updated
- ✅ Free access
- ✅ Structured HTML (relatively easy to parse)

**Cons:**

- ⚠️ Web scraping (may break if HTML changes)
- ⚠️ Terms of use unclear (may need permission)
- ⚠️ No official API

**Feasibility:** ✅ **VIABLE** - Technical implementation straightforward

---

### Option 2: Contact NG3K for Permission/API

**Approach:** Email Bill Feidt (NG3K) to request permission or API access

**Email template:**

```text
Subject: Request for DXpedition Data Access for HamClock Software

Dear Bill (NG3K),

I am working on maintaining ESPHamClock, an open-source amateur radio 
display application originally created by Elwood Downey (WB0OEW, SK). 
The software displays DXpedition information from your excellent ADXO 
page.

The original backend server provided DXpedition data in a specific format, 
but we are now building a replacement backend to ensure the software 
continues to function for the ham radio community.

I would like to request permission to:
- Parse your ADXO HTML page programmatically, or
- Access DXpedition data via an API or structured format (if available)

The software is non-profit, open-source, and used by amateur radio 
operators worldwide. We will provide proper attribution ("NG3K ADXO") 
and links back to your website.

Could you please advise on the best way to access your DXpedition data 
for this purpose?

Thank you for maintaining this valuable resource for the DX community.

73,
[Your Name and Callsign]
```

**Pros:**

- ✅ Officially sanctioned
- ✅ Clear legal standing
- ✅ May get better data format

**Cons:**

- ⏱️ Requires communication and response
- ⏱️ May take time
- ❓ May be denied or have conditions

**Feasibility:** ✅ **VIABLE** - Worth attempting

---

### Option 3: Use DXNews RSS Feed

**Approach:** Parse DXNews RSS feed for DXpedition announcements

**RSS Feed URL:** Need to identify (likely https://dxnews.com/feed/ or similar)

**Challenges:**

1. **Blog format** - Posts are narrative, not structured data
2. **Date parsing** - Dates in text, not structured fields
3. **Incomplete** - May not include all expeditions
4. **Callsign extraction** - Requires text parsing

**Example RSS entry:**

```xml
<item>
  <title>V47JA St Kitts and Nevis</title>
  <link>https://dxnews.com/v47ja/</link>
  <description>
    W5JON will be active as V47JA from St Kitts and Nevis, 
    30 January - 3 February 2026. He will operate on 160-6m 
    using SSB and FT8.
  </description>
  <pubDate>Mon, 20 Jan 2026 12:00:00 +0000</pubDate>
</item>
```

**Parsing complexity:** HIGH - Requires NLP to extract dates, callsigns, locations

**Feasibility:** ⚠️ **POSSIBLE** - But complex and incomplete

---

### Option 4: Combine Multiple Sources

**Approach:** Aggregate data from NG3K + DXNews + other sources

**Sources:**

1. **NG3K ADXO** (primary) - Comprehensive list
2. **DXNews RSS** (secondary) - Additional details
3. **ClubLog DXpeditions** - https://clublog.org/dxped.php
4. **QRZ.com DXpedition Calendar** - https://www.qrz.com/dxped/

**Pros:**

- ✅ Most comprehensive
- ✅ Redundancy if one source fails

**Cons:**

- ⚠️ Complex implementation
- ⚠️ Duplicate detection needed
- ⚠️ Multiple parsing strategies

**Feasibility:** ⚠️ **COMPLEX** - Overkill for most use cases

---

## Terms of Use Considerations

### NG3K Website

**Copyright notice:** "Copyright © W.B. Feidt 1996-2026"

**No explicit terms of use found** on the ADXO page

**Common practices:**

- Amateur radio community generally supportive of non-profit software
- Attribution and links back to source typically expected
- Automated access should be reasonable (not excessive)

**Recommendation:**

1. **Contact NG3K first** to request permission
2. **If no response**, proceed with scraping but:
   - Provide clear attribution
   - Link back to NG3K website
   - Implement reasonable rate limiting
   - Cache data (24-hour refresh)
   - Be prepared to stop if requested

---

## Recommendation

### Primary: Contact NG3K for Permission, Then Scrape if Approved

**Rationale:**

1. **NG3K ADXO is the authoritative source** for DXpedition data
2. **Requesting permission is respectful** and may lead to better access
3. **Web scraping is technically feasible** if permission granted
4. **Amateur radio community values** are typically supportive of non-profit software

### Implementation Plan

#### Phase 1: Contact NG3K

1. **Email Bill Feidt (NG3K)** using template above
2. **Wait for response** (1-2 weeks)
3. **If approved**, proceed with implementation
4. **If no response**, proceed cautiously with scraping

#### Phase 2: Build HTML Parser

**Backend service (Python):**

```python
import requests
from bs4 import BeautifulSoup
from datetime import datetime
import time

def fetch_ng3k_dxpeditions():
    """
    Fetch and parse NG3K ADXO page
    """
    url = "https://www.ng3k.com/Misc/adxo.html"
    
    # Add user agent to identify ourselves
    headers = {
        'User-Agent': 'HamClock/1.0 (https://github.com/fang64/hamclock; contact@example.com)'
    }
    
    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # Parse table (implementation depends on actual HTML structure)
    dxpeds = parse_adxo_table(soup)
    
    return format_as_dxpeditions_txt(dxpeds)

def parse_adxo_table(soup):
    """
    Extract DXpedition entries from HTML table
    """
    # Implementation depends on actual HTML structure
    # This is a placeholder
    pass

def format_as_dxpeditions_txt(dxpeds):
    """
    Format as dxpeditions.txt
    """
    output = []
    
    # Credit line
    output.append("1")
    output.append("NG3K")
    output.append("https://www.ng3k.com/Misc/adxo.html")
    
    # DXpedition entries
    for dxped in dxpeds:
        line = f"{dxped['start_unix']},{dxped['end_unix']},{dxped['location']},{dxped['callsign']},{dxped['url']}"
        output.append(line)
    
    return "\n".join(output)
```

#### Phase 3: Backend Proxy Endpoint

```python
@app.route('/dxpeds/dxpeditions.txt')
def dxpeditions():
    # Fetch from NG3K (with caching)
    dxpeds_data = fetch_ng3k_dxpeditions()
    
    # Return as text
    return dxpeds_data, 200, {'Content-Type': 'text/plain'}
```

#### Phase 4: Add Caching

```python
from functools import lru_cache
import time

@lru_cache(maxsize=1)
def cached_fetch_ng3k_dxpeditions():
    """
    Cache for 24 hours
    """
    return fetch_ng3k_dxpeditions(), time.time()

def get_dxpeditions():
    data, timestamp = cached_fetch_ng3k_dxpeditions()
    
    # Refresh if older than 24 hours
    if time.time() - timestamp > 86400:
        cached_fetch_ng3k_dxpeditions.cache_clear()
        data, timestamp = cached_fetch_ng3k_dxpeditions()
    
    return data
```

#### Phase 5: Test and Validate

1. **Compare output** with original dxpeditions.txt
2. **Verify date parsing** accuracy
3. **Test DXpedition display** in HamClock
4. **Verify links work** correctly
5. **Monitor for HTML changes** on NG3K site

---

## Alternative: Fallback to DXNews RSS

If NG3K scraping fails or is denied:

1. **Parse DXNews RSS feed** for DXpedition posts
2. **Use NLP/regex** to extract dates, callsigns, locations
3. **Accept incomplete data** (not all expeditions may be listed)
4. **Supplement with manual entries** for major DXpeditions

---

## Implementation Complexity

### NG3K Scraping Approach

**Estimated effort:** 2-3 days

**Components:**

- HTML parser (BeautifulSoup)
- Date parsing logic
- Format converter
- Backend endpoint
- Caching
- Testing

**Complexity:** **MODERATE** - Depends on HTML structure stability

### DXNews RSS Approach

**Estimated effort:** 3-5 days

**Components:**

- RSS parser
- NLP/regex for data extraction
- Date parsing
- Format converter
- Backend endpoint
- Testing

**Complexity:** **HIGH** - Text parsing is complex and error-prone

---

## Comparison Matrix

| Approach             | Complexity | Data Quality | Completeness | Legal Status | Maintenance |
| -------------------- | ---------- | ------------ | ------------ | ------------ | ----------- |
| NG3K Scraping        | Moderate   | Excellent    | Excellent    | Unclear      | Medium      |
| NG3K with Permission | Moderate   | Excellent    | Excellent    | Clear        | Low         |
| DXNews RSS           | High       | Good         | Moderate     | Clear        | Medium      |
| Multiple Sources     | High       | Excellent    | Excellent    | Varies       | High        |

---

## Final Recommendation

### Contact NG3K for Permission, Then Implement Web Scraping

**Justification:**

1. **NG3K ADXO is the authoritative source** for DXpedition data
2. **Requesting permission is respectful** and aligns with amateur radio values
3. **Web scraping is technically feasible** and straightforward
4. **Data quality is excellent** - comprehensive and regularly updated
5. **Amateur radio community** typically supports non-profit software

### Implementation Steps

1. **Email NG3K** requesting permission to parse ADXO page
2. **Wait for response** (1-2 weeks)
3. **If approved or no response**, implement HTML parser
4. **Build backend endpoint** at `/dxpeds/dxpeditions.txt`
5. **Add caching** (24-hour refresh)
6. **Test with HamClock** to verify compatibility
7. **Monitor HTML structure** for changes
8. **Provide attribution** and links back to NG3K

---

## Conclusion

**Best approach:** Contact NG3K for permission, then implement web scraping of ADXO page.

**Rationale:** NG3K ADXO is the most comprehensive and authoritative source for DXpedition data. Requesting permission respects the owner's work and aligns with amateur radio community values. Web scraping is technically straightforward and provides excellent data quality.

**Fallback:** If NG3K denies permission or scraping becomes unreliable, use DXNews RSS feed with text parsing, accepting reduced completeness.

**Next step:** Email Bill Feidt (NG3K) to request permission for programmatic access to ADXO data.
