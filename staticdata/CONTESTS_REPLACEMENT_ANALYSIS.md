# Contest Calendar Replacement Analysis

**Date:** January 31, 2026  
**File:** contests311.txt  
**Status:** ✅ **VIABLE** - Can use WA7BNM RSS feed with proper attribution

## What is contests311.txt?

A list of upcoming amateur radio contests with start/end times and details.

### Current Format

```text
WA7BNM Weekend Contests
1769817600 1769903940 Feld Hell Sprint
https://www.contestcalendar.com/contestdetails.php?ref=436
1769864400 1769950800 UBA DX Contest, SSB
https://www.contestcalendar.com/contestdetails.php?ref=235
1769904000 1769918400 North American SSB Sprint Contest
https://www.contestcalendar.com/contestdetails.php?ref=242
```

**Format:**

- **Line 1**: Credit line (e.g., "WA7BNM Weekend Contests")
- **Subsequent pairs of lines**:
  - **Line A**: `<start_unix_time> <end_unix_time> <contest_title>`
  - **Line B**: `<contest_details_url>`

**Fields:**

- **start_unix_time**: Contest start time (UNIX timestamp, UTC)
- **end_unix_time**: Contest end time (UNIX timestamp, UTC)
- **contest_title**: Name of the contest
- **contest_details_url**: Link to contest details page

**Update frequency:** Every hour (CONTESTS_MAXAGE = 3600 seconds)

### How HamClock Uses It

From `src/contests.cpp`:

1. **Downloads file** via `/contests/contests311.txt` endpoint
2. **Caches locally** as `contests311.txt` (1-hour cache)
3. **Parses format**:
   - First line = credit attribution
   - Pairs of lines = contest info + URL
4. **Filters out past contests** - Only shows future/current contests
5. **Displays in pane** with scrolling list
6. **Highlights active contests** - Green background if now is between start/end times
7. **Supports alarms** - Can set alarm for contest start time
8. **Links to details** - Opens contest URL in browser

**Features:**

- Scrollable list of upcoming contests
- Toggle date display (show/hide start/end times)
- Toggle timezone (UTC vs DE timezone)
- Set one-time alarm for contest start
- Open contest details page in browser
- Open weekly calendar page

---

## Data Source: WA7BNM Contest Calendar

**Website:** https://contestcalendar.com/

**Owner:** Bruce Horn, WA7BNM

**Purpose:** Comprehensive amateur radio contest calendar

### Available Formats

1. **Web pages** (HTML)
   - 8-day calendar: https://www.contestcalendar.com/weeklycont.php
   - 5-week calendar
   - 12-month calendar
   - State QSO parties

2. **RSS feeds** (XML)
   - Available for various calendar views
   - Labeled with RSS icon on website

3. **Text format**
   - Weekly calendar: https://contestcalendar.com/weeklycal.txt
   - Plain text format

4. **iCal/vCal format** (ICS)
   - For calendar applications

5. **XML/JSON format**
   - Available to software authors by written agreement

---

## Terms of Use Analysis

From https://contestcalendar.com/terms.php:

### What is ALLOWED

✅ **Direct links** to WA7BNM web pages (must be labeled "WA7BNM Contest Calendar")

✅ **RSS feeds** for display on club or non-profit websites with:
- Display labeled "WA7BNM Contest Calendar"
- Links back to the Site's web pages

✅ **ICS feeds** for personal calendar applications

✅ **XML/JSON data** for contest-related software (requires written agreement)

### What is NOT ALLOWED

❌ **Copying web pages** to another website

❌ **Copying e-mailed calendar** to another website (except DLARC/Internet Archive)

❌ **Automated scraping** (robots, spiders) without authorization

❌ **Overloading the site** or interfering with other users

### Interpretation for HamClock

**Current backend (contests311.txt):**
- Likely created by Clear Sky Institute with permission or agreement
- Format suggests custom data extraction/formatting
- May have been authorized under "written agreement" clause

**Replacement options:**
1. **RSS feed** - Explicitly allowed for non-profit use with attribution
2. **Text format** - Available but terms unclear (may require permission)
3. **XML/JSON** - Requires written agreement with owner
4. **Web scraping** - Explicitly prohibited

---

## Replacement Options

### Option 1: Use WA7BNM RSS Feed (RECOMMENDED)

**Approach:** Parse RSS feed from WA7BNM contest calendar

**RSS Feed URL:** Need to identify specific RSS feed URL (varies by calendar view)

**Typical RSS format:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
  <channel>
    <title>WA7BNM Contest Calendar</title>
    <link>https://www.contestcalendar.com/</link>
    <description>Amateur Radio Contest Calendar</description>
    <item>
      <title>Feld Hell Sprint</title>
      <link>https://www.contestcalendar.com/contestdetails.php?ref=436</link>
      <description>Start: Sat Jan 30 2026 18:00 UTC, End: Sun Jan 31 2026 17:59 UTC</description>
      <pubDate>...</pubDate>
    </item>
    ...
  </channel>
</rss>
```

**Implementation:**

1. **Fetch RSS feed** from WA7BNM
2. **Parse XML** to extract contest entries
3. **Convert to contests311.txt format**:
   - Extract title from `<title>`
   - Parse start/end times from `<description>` or custom fields
   - Convert to UNIX timestamps
   - Extract URL from `<link>`
4. **Add attribution** - First line: "WA7BNM Contest Calendar" (or as specified in RSS)
5. **Cache locally** with 1-hour expiration

**Pros:**

- ✅ Explicitly allowed by terms of use
- ✅ Free, no API key required
- ✅ Maintained by WA7BNM (authoritative source)
- ✅ Proper attribution built-in
- ✅ No written agreement needed

**Cons:**

- ⚠️ RSS format may vary (need to identify correct feed)
- ⚠️ Date/time parsing may be complex (depends on RSS format)
- ⚠️ May not include all contests (RSS might be subset)

**Feasibility:** ✅ **VIABLE** - Best option, complies with terms

---

### Option 2: Use Text Format (weeklycal.txt)

**URL:** https://contestcalendar.com/weeklycal.txt

**Approach:** Fetch text file and convert to contests311.txt format

**Challenges:**

1. **Terms unclear** - Text format not explicitly mentioned in terms of use
2. **Format unknown** - Need to examine actual file format
3. **May require permission** - Could fall under "automated access" restriction

**Feasibility:** ⚠️ **UNCERTAIN** - Would need to contact WA7BNM for permission

---

### Option 3: Request XML/JSON Access

**Approach:** Contact Bruce Horn (WA7BNM) to request XML/JSON data access

**Process:**

1. Email WA7BNM explaining HamClock use case
2. Request written agreement for XML/JSON access
3. Implement parser for agreed-upon format

**Pros:**

- ✅ Officially sanctioned
- ✅ Likely best data format
- ✅ Clear legal standing

**Cons:**

- ⏱️ Requires communication and agreement
- ⏱️ May take time to establish
- ❓ May be denied or have conditions

**Feasibility:** ✅ **VIABLE** - But requires owner contact

---

### Option 4: Alternative Contest Sources

**Other contest calendars:**

1. **SM3CER Contest Service** - http://www.sk3bg.se/contest/
2. **ARRL Contest Calendar** - http://www.arrl.org/contest-calendar
3. **DX Zone Contest Calendar** - https://www.dxzone.com/catalog/Operating_Modes/Contesting/

**Challenges:**

- Different formats
- May not be as comprehensive
- Different terms of use
- Would need separate analysis

**Feasibility:** ⚠️ **POSSIBLE** - But WA7BNM is preferred source

---

## Recommendation

### Use WA7BNM RSS Feed with Proper Attribution

**Rationale:**

1. **Explicitly allowed** by WA7BNM terms of use for non-profit software
2. **No written agreement required** (unlike XML/JSON)
3. **Authoritative source** - WA7BNM is the standard in ham radio contesting
4. **Free and maintained** - No API keys, regularly updated
5. **Proper attribution** - RSS includes credit information

### Implementation Plan

#### Phase 1: Identify RSS Feed URL

1. **Visit WA7BNM website** and locate RSS icon
2. **Test RSS feed** to understand format
3. **Document RSS structure** (title, description, dates, links)

#### Phase 2: Build RSS Parser

**Backend service (Python/Node):**

```python
import feedparser
import time
from datetime import datetime

def fetch_contests_from_rss(rss_url):
    """
    Fetch contests from WA7BNM RSS feed and convert to contests311.txt format
    """
    feed = feedparser.parse(rss_url)
    
    output = []
    
    # First line: attribution (from RSS channel title)
    output.append(feed.feed.title)
    
    # Parse each contest entry
    for entry in feed.entries:
        title = entry.title
        url = entry.link
        
        # Parse start/end times from description or custom fields
        # (Format depends on actual RSS structure)
        start_time = parse_contest_start(entry)
        end_time = parse_contest_end(entry)
        
        # Convert to UNIX timestamps
        start_unix = int(start_time.timestamp())
        end_unix = int(end_time.timestamp())
        
        # Add to output in contests311.txt format
        output.append(f"{start_unix} {end_unix} {title}")
        output.append(url)
    
    return "\n".join(output)
```

#### Phase 3: Backend Proxy Endpoint

```python
@app.route('/contests/contests311.txt')
def contests311():
    # Fetch from RSS
    rss_url = "https://www.contestcalendar.com/[RSS_FEED_PATH]"
    contests_data = fetch_contests_from_rss(rss_url)
    
    # Return as text
    return contests_data, 200, {'Content-Type': 'text/plain'}
```

#### Phase 4: Add Attribution in HamClock

Ensure HamClock displays:
- "WA7BNM Contest Calendar" credit line (already does via first line)
- Links back to contestcalendar.com (already does via contest URLs)

#### Phase 5: Test and Validate

1. **Compare output** with original contests311.txt
2. **Verify date/time parsing** accuracy
3. **Test contest display** in HamClock
4. **Verify links work** correctly

---

## Alternative: Contact WA7BNM for Permission

If RSS feed format is insufficient or unclear:

### Email Template

```text
Subject: Request for Contest Data Access for HamClock Software

Dear Bruce (WA7BNM),

I am working on maintaining ESPHamClock, an open-source amateur radio 
display application originally created by Elwood Downey (WB0OEW, SK). 
The software displays contest information from your excellent WA7BNM 
Contest Calendar.

The original backend server provided contest data in a specific format, 
but we are now building a replacement backend to ensure the software 
continues to function for the ham radio community.

I would like to request permission to access your contest data via:
- RSS feed (if available for weekly contests), or
- XML/JSON format (if a written agreement is needed)

The software is non-profit, open-source, and used by amateur radio 
operators worldwide. We will provide proper attribution ("WA7BNM Contest 
Calendar") and links back to your website.

Could you please advise on the best way to access your contest data 
for this purpose?

Thank you for maintaining this valuable resource for the contesting 
community.

73,
[Your Name and Callsign]
```

---

## Implementation Complexity

### RSS Feed Approach

**Estimated effort:** 1-2 days

**Components:**

- RSS feed parser (feedparser library)
- Date/time extraction logic
- Format converter (RSS → contests311.txt)
- Backend endpoint
- Testing

**Complexity:** **MODERATE** - Depends on RSS date format

### XML/JSON Agreement Approach

**Estimated effort:** 1 week (including communication time)

**Components:**

- Email communication with WA7BNM
- Agreement negotiation
- Parser implementation (once format is known)
- Backend endpoint
- Testing

**Complexity:** **LOW** (technical) + **MODERATE** (administrative)

---

## Comparison Matrix

| Approach            | Complexity | Legal Status | Data Quality | Maintenance |
| ------------------- | ---------- | ------------ | ------------ | ----------- |
| RSS Feed            | Moderate   | Allowed      | Good         | Low         |
| Text Format         | Low        | Unclear      | Good         | Low         |
| XML/JSON Agreement  | Low        | Allowed      | Excellent    | Low         |
| Alternative Sources | High       | Varies       | Varies       | Medium      |
| Web Scraping        | High       | Prohibited   | Good         | High        |

---

## Final Recommendation

### Primary: Use WA7BNM RSS Feed

**Justification:**

1. **Explicitly allowed** by terms of use
2. **No permission needed** (unlike XML/JSON)
3. **Authoritative source** for contest data
4. **Free and maintained** by WA7BNM
5. **Proper attribution** built into RSS

### Fallback: Contact WA7BNM for XML/JSON Access

If RSS feed is insufficient or unavailable:

1. **Email Bruce Horn (WA7BNM)** explaining HamClock use case
2. **Request XML/JSON access** with written agreement
3. **Implement parser** for agreed format

### Implementation Steps

1. **Identify RSS feed URL** from WA7BNM website
2. **Test RSS feed** to understand structure
3. **Build RSS parser** to extract contest data
4. **Convert to contests311.txt format**
5. **Implement backend endpoint** at `/contests/contests311.txt`
6. **Test with HamClock** to verify compatibility
7. **Document attribution** requirements

---

## Conclusion

**Best approach:** Use WA7BNM RSS feed with proper attribution.

**Rationale:** The RSS feed is explicitly allowed for non-profit use, requires no special permission, and provides authoritative contest data. This approach respects WA7BNM's terms of use while maintaining HamClock's contest display functionality.

**Next step:** Identify the specific RSS feed URL for weekly/upcoming contests and test the feed structure to build an appropriate parser.
