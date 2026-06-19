## hgh-airport-api

> Use when querying Hangzhou Xiaoshan International Airport (HGH/ZSHC) flight departures, arrivals, timetables (including day-of-week schedules), or city autocomplete. Documents the complete set of JSON APIs reverse-engineered from hzairport.com with curl recipes, response schemas, and usage patterns.


# Hangzhou Xiaoshan Airport (HGH/ZSHC) API

## Overview

The official Hangzhou airport website (`hzairport.com`) exposes a set of JSON APIs that return real-time flight data and static schedules. All endpoints are **unauthenticated**, accept GET (and POST), and return clean JSON. They were reverse-engineered from the site's own `ajax.js` (located at `/static/front/js/ajax.js`), which defines jQuery AJAX wrappers calling these endpoints server-side.

**Base URL:** `https://www.hzairport.com`

**Architecture:** The frontend is a traditional jQuery server-rendered site (~2018 era). Flight data is loaded asynchronously via synchronous `$.ajax({async:false})` POST calls to `_more.html` endpoints that return paginated JSON. All endpoints also accept GET with query-string parameters — no auth tokens, CSRF, or session cookies required.

---

## When to Use

- User asks about a specific flight from/to Hangzhou (HGH)
- User wants to know today's departures/arrivals at HGH
- User asks "what flights leave Hangzhou at X time" or similar time-based queries
- User asks about flight schedules for a specific weekday (e.g. "Friday 12:00")
- User asks about a flight's status, gate, aircraft type, or airline
- User needs to find valid city/airport names for filtering (autocomplete)

Do NOT use for:
- Other Chinese airports (each has its own site/API)
- Booking or purchasing tickets (status/schedule only)
- Historical flight data (only current-day live data + static schedule)
- Future-date live status (the live endpoints ignore any `date` parameter)

---

### Quick Decision: Which Endpoint?

| Question type | Endpoint | Key detail |
|---------------|----------|------------|
| "What flights leave Hangzhou today at X:XX?" | `/flight/index_more.html` | `md_name` = destination |
| "Flight CA4598 — real-time status?" | `/flight/index_more.html?identity=CA4598` | `operation_type_en` |
| "What flights land in Hangzhou today at X:XX?" | `/flight/arrive_more.html` | ⚠️ `sf_name` = origin (NOT `md_name`) |
| "Does flight X operate on Fridays?" | `/flight/more_time.html?cates=arrive&keywords=X` | `schedule` field: "5"=Friday |
| "What flights operate on day X at time Y?" | `/flight/more_time.html` + manual filter | `schedule` + `arrive`/`takeoff` |
| "Give me the full schedule for route A→B" | `/flight/more_time.html?cates=arrive&keywords=B` | Paginated scan |
| "What are valid city names for filtering?" | `/flight/out_air_more.html?city=X` | Returns `[{title: "城市/机场"}]` |
| "Find a flight number by partial match" | `/flight/out_air_more.html?identity=X` | Returns matching flight numbers |

---

## Endpoints

### 1. Departures — `GET /flight/index_more.html`

**Live departure board for the current day only.** The `date` parameter, if passed, is ignored by the backend.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `p` | int | No | Page number (default 1, 10 items/page, ordered by scheduled time ascending) |
| `identity` | string | No | Filter by flight number (e.g. `CA4598`) |
| `city` | string | No | Filter by destination city in Chinese (e.g. `成都`) |
| `airline` | string | No | Filter by airline name in Chinese (e.g. `国际航空`) |

**Minimal example:**
```bash
curl -s "https://www.hzairport.com/flight/index_more.html?p=1"
```

**Response schema:**
```json
{
  "flag": 1,
  "msg": "OK",
  "data": [
    {
      "flight_identity": "CA4598",
      "airline_name": "国际航空",
      "iata_flight_number": "CA4598",
      "md_name": "成都/双流",
      "md_name_en": "Chengdu/Shuangliu",
      "jhsj_time": "2026-05-19 20:00:00",
      "sjsj_time": null,
      "bgsj_time": null,
      "operation_type": "计划",
      "operation_type_en": "On Schedule",
      "flight_sector_code": "国内",
      "gate_id": "B14",
      "check_indesk_range": "F17-F29",
      "air_craftty": "空客A321(窄体大型)",
      "jt_name": "",
      "jt_name_en": ""
    }
  ],
  "extra": {}
}
```

**Key fields:**

| Field | Type | Description |
|-------|------|-------------|
| `flight_identity` | string | Flight number (primary identifier) |
| `airline_name` | string | Airline in Chinese (see Airline Mapping table) |
| `md_name` / `md_name_en` | string | Destination city/airport (中/EN) |
| `jhsj_time` | string | **Scheduled time** (`YYYY-MM-DD HH:MM:SS`) |
| `sjsj_time` | string\|null | **Actual time** — null until operated |
| `bgsj_time` | string\|null | **Estimated time** (预估时间) — populated when en-route, useful fallback |
| `operation_type_en` | string | `On Schedule` \| `Departed` \| `Delayed` \| `Cancelled` |
| `flight_sector_code` | string | `国内` \| `国际` \| `地区` |
| `gate_id` | string | Boarding gate (often empty for future flights) |
| `check_indesk_range` | string | Check-in counter range |
| `air_craftty` | string | Aircraft type in Chinese |
| `jt_name` / `jt_name_en` | string | Stopover/via city (empty if direct) |

### 2. Arrivals — `GET /flight/arrive_more.html`

**Live arrival board for the current day only.** Same parameter interface as departures.

```bash
curl -s "https://www.hzairport.com/flight/arrive_more.html?p=1"
```

**⚠️ Critical: different field names from departures!**

The arrivals endpoint stores the origin city in `sf_name` / `sf_name_en` ("sf" = 始发 = departure/origin), **NOT** `md_name` / `md_name_en`. The `md_*` fields will be **empty strings** in arrivals responses.

| Arrivals field | Departures equivalent | Meaning |
|---------------|----------------------|---------|
| `sf_name` | `md_name` | Origin city (中文) |
| `sf_name_en` | `md_name_en` | Origin city (English) |
| `jt_name` | `jt_name` | Stopover city (same name) |

All other fields (`flight_identity`, `jhsj_time`, `sjsj_time`, `bgsj_time`, `operation_type_en`, `flight_sector_code`, `airline_name`, etc.) are identical to departures.

### 3. Timetable — `GET /flight/more_time.html`

**Static flight schedule with day-of-week granularity.** This is a completely different data model from the live endpoints — it represents the scheduled pattern, not real-time status. A single flight number can appear **multiple times** with different `schedule` values and slightly different times per day.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `p` | int | No | Page number |
| `keywords` | string | No | Search by flight number or city name (Chinese) |
| `types` | string | No | `0` = domestic, `1` = international (⚠️ may not filter reliably — verify manually) |
| `cates` | string | **Yes** | `leave` = departures, `arrive` = arrivals |

**Example — find all arrivals from Chengdu:**
```bash
curl -s "https://www.hzairport.com/flight/more_time.html?cates=arrive&keywords=成都"
```

**Response schema (different from live endpoints!):**
```json
{
  "flag": 1,
  "msg": "OK",
  "data": [
    {
      "air_no": "MU6171",
      "departure": "成都双流",
      "takeoff": "0920",
      "arrive": "1200",
      "schedule": "123467",
      "air_type": "32N",
      "cates": "arrive",
      "throught": "",
      "departure_en": "",
      "throught_en": ""
    }
  ],
  "extra": {}
}
```

| Field | Type | Description |
|-------|------|-------------|
| `air_no` | string | Flight number (e.g. `MU6171`) |
| `departure` | string | Origin/destination city in Chinese |
| `takeoff` | string | Departure time — **HHMM string** (e.g. `"0920"`) |
| `arrive` | string | Arrival time — **HHMM string** (e.g. `"1200"`) |
| `schedule` | string | **Day-of-week codes** (see below) |
| `air_type` | string | Aircraft IATA code (`738`, `32N`, `320`, `333`, `JET`, `3D`) |
| `cates` | string | `leave` or `arrive` |
| `throught` | string | Stopover city in Chinese (empty = direct) |
| `departure_en` | string | Origin in English (often empty) |
| `throught_en` | string | Stopover in English (often empty) |

**Schedule field decoding:**

The `schedule` field uses digit concatenation: `1`=Mon, `2`=Tue, `3`=Wed, `4`=Thu, `5`=Fri, `6`=Sat, `7`=Sun.

| schedule | Meaning |
|----------|---------|
| `"1234567"` | Daily |
| `"5"` | Friday only |
| `"1357"` | Mon + Wed + Fri + Sun |
| `"246"` | Tue + Thu + Sat |
| `"123467"` | All days EXCEPT Friday |
| `"37"` | Wed + Sun |

```python
days_map = {'1':'Mon','2':'Tue','3':'Wed','4':'Thu','5':'Fri','6':'Sat','7':'Sun'}
# Decode: "1357" → ['Mon','Wed','Fri','Sun']
days = [days_map[d] for d in schedule]
```

**⚠️ Multiple entries per flight:** A single flight (e.g. MU6171) can have **multiple timetable records** with different `schedule` values and slightly different times — e.g. `schedule:"123467" arr:"1200"` AND `schedule:"5" arr:"1145"`. Always retrieve ALL entries for a flight number (use `keywords=<flight_no>`) and check each one's schedule, not just the first result.

### 4. City Autocomplete — `GET /flight/out_air_more.html` & `/flight/arr_air_more.html`

Fuzzy-search endpoints used by the website's search boxes. Returns simple `[{title: "..."}]` arrays.

| Endpoint | Purpose | Key param |
|----------|---------|-----------|
| `/flight/out_air_more.html` | Search departure destinations | `city` (Chinese) |
| `/flight/arr_air_more.html` | Search arrival origins | `city` (Chinese) |

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `city` | Partial city name in Chinese (e.g. `成` matches 成都/双流, 成都/天府) |
| `identity` | Partial flight number (e.g. `CA17` matches CA1727, CA1741, etc.) |

**Behavior:**
- With `city` param: returns matching **city names** — useful for discovering valid filter values
- With `identity` param: returns matching **flight numbers**
- With no params: returns recent/popular **flight numbers** (not city names)
- Max 20 results per call

**Example:**
```bash
# Find cities matching "成"
curl -s "https://www.hzairport.com/flight/out_air_more.html?city=成"
# → [{"title":"成都/双流"},{"title":"成都/天府"},{"title":"北海/福成"}]

# Find flights matching "CA17"
curl -s "https://www.hzairport.com/flight/out_air_more.html?identity=CA17"
# → [{"title":"CA1727"},{"title":"CA1741"},{"title":"CA1747"},...]
```

### 5. Timetable Keyword Search — `GET /flight/air_time_more.html`

Fuzzy search for timetable entries. Similar to the autocomplete endpoints but operates on the timetable dataset.

| Parameter | Description |
|-----------|-------------|
| `keywords` | Partial city name or flight number |
| `cates` | `leave` or `arrive` (optional) |

```bash
curl -s "https://www.hzairport.com/flight/air_time_more.html?keywords=成都"
```

---

## Bonus: Non-Flight Endpoints

The `ajax.js` file also defines endpoints for the airport's corporate website content. These are outside the flight-data scope but documented here for completeness.

| Function | Endpoint | Params | Status | Description |
|----------|----------|--------|--------|-------------|
| `getReportList` | `/about/about_more.html` | `cid`, `p` | ✅ Works (cid=10001) | CSR/sustainability reports. Fields: `title`, `detail`, `addtime`, `pic`, `pdf_file` |
| `getNoticeList` | `/notice/list_more.html` | `cid`, `p`, `keywords`, `times` | ✅ Works (cid=10001) | Airport notices. Fields: `title`, `detail`, `addtime`, `hits` |
| `getMediaList` | `/media/list_more.html` | `cid`, `p`, `keywords`, `year` | ✅ Works (cid=10001) | Media center news. Fields: `title`, `detail`, `addtime`, `pic`, `video` |
| `getPartyList` | `/party/list_more.html` | `cid`, `p`, `keywords`, `year` | ✅ Works | Party/党建 news. Fields: `title`, `detail`, `addtime`, `hits`, `pic` |
| `getTalentList` | `/talent/list_more.html` | `cid`, `p`, `keywords` | ⚠️ Returns empty | Job listings — likely no current openings |
| `getTenderList` | `/tender/list_more.html` | `cid`, `p`, `types`, `keywords`, `tender_time` | ❌ Broken | Tender/bidding — SQL error in backend (SQL injection in `cid` parameter) |

`cid` values appear to be category IDs (`10001` is a common one). These endpoints also accept POST with body parameters (matching the jQuery `ajaxMain` usage).

---

## Usage Patterns

### Finding Flights by Time (Live Endpoints)

The live endpoints paginate by scheduled time ascending. Each page has exactly 10 entries, but due to **codeshares** (one physical flight listed under 5-10 airline codes), there are typically only 2-5 distinct flights per page.

**Page-to-time estimation:**
- First flights of the day: ~06:00 on page 1
- Page progression: ~10 pages per real-world hour
- 12:00 noon ≈ page 30-35
- 20:00 evening ≈ page 130-135
- Full day span: ~140-200 pages

**Strategy:**
1. Estimate target page from time
2. Jump directly (don't iterate from page 1)
3. Walk ±3 pages to capture all codeshares at target time
4. Group results by `jhsj_time` + destination

```bash
# One-shot: find all 20:00 departures
for p in 130 131 132 133 134 135; do
  curl -s "https://www.hzairport.com/flight/index_more.html?p=$p" | \
    python3 -c "import sys,json;d=json.load(sys.stdin);[print(f'{f[\"flight_identity\"]} | {f[\"md_name\"]} | {f[\"jhsj_time\"]}') for f in d['data']]"
done
```

### Finding Flights by Weekday (Timetable)

```python
import json, subprocess

def find_flights_by_day(cates="arrive", target_day="5", time_range=("1150","1210")):
    """Find all flights on a given day within a time window.
    
    Args:
        cates: "arrive" or "leave"
        target_day: "5"=Fri, "1"=Mon, etc.
        time_range: (min_HHMM, max_HHMM) as strings
    """
    lo, hi = time_range
    for p in range(1, 80):
        cmd = f'curl -s "https://www.hzairport.com/flight/more_time.html?cates={cates}&p={p}"'
        data = json.loads(subprocess.check_output(cmd, shell=True).decode())
        if not data.get('data'):
            break
        for f in data['data']:
            t = f.get('arrive' if cates == 'arrive' else 'takeoff', '')
            schedule = f.get('schedule', '')
            if lo <= t <= hi and target_day in schedule:
                yield f

# Example: Friday arrivals 11:50-12:10
for f in find_flights_by_day("arrive", "5", ("1150", "1210")):
    print(f"{f['air_no']} | from: {f['departure']} | arr: {f['arrive']} | days: {f['schedule']}")
```

---

## Common Pitfalls

1. **Arrivals use `sf_name`, not `md_name`.** The arrivals endpoint stores origin in `sf_name`/`sf_name_en`. If you read `md_name` from arrivals, you'll get empty strings. This is the #1 mistake.

2. **Timetable has a completely different schema.** No `jhsj_time`, no `md_name`. Use `takeoff`/`arrive` (HHMM strings) and `schedule` (day codes). The `departure` field in timetable = origin city (always in Chinese).

3. **Codeshares inflate page counts 5-10x.** A single A321 can appear as 8 separate entries (CA + ZH + SC + 3U + MF + MU + CZ + G5). Group by `jhsj_time` + destination before reporting.

4. **One flight, multiple timetable entries.** MU6171 has `schedule:"123467" arr:"1200"` AND `schedule:"5" arr:"1145"` — the Friday variant arrives 15 min earlier. Always retrieve ALL entries with `keywords=<flight_no>`.

5. **Live endpoints are today-only.** The `date` parameter (if accepted) is ignored. For future dates or recurring schedules, use the timetable endpoint.

6. **`sjsj_time` is null until operated.** Use `bgsj_time` (预估时间) as a fallback for in-flight status when `sjsj_time` is still null.

7. **`airline_name` uses Chinese.** See mapping table below. "国际航空" = Air China, not "International Airlines".

8. **Empty pages mean end-of-data.** When `data` returns `[]`, there are no more scheduled flights (~01:00-02:00 for live endpoints).

9. **Timetable `types` filter is unreliable.** `types=0` (domestic) and `types=1` (international) may return the same data. Always verify `departure` values manually rather than trusting the filter.

10. **Autocomplete returns flight numbers without a `city` param.** Call `/flight/out_air_more.html` with no params → flight numbers. Call with `city=成` → city names. The behavior depends on which param you provide.

---

## Airline Name Mapping

| Chinese (API value) | IATA | English |
|---------------------|------|---------|
| 国际航空 | CA | Air China |
| 东方航空 | MU | China Eastern |
| 南方航空 | CZ | China Southern |
| 海南航空 | HU | Hainan Airlines |
| 深圳航空 | ZH | Shenzhen Airlines |
| 厦门航空 | MF | Xiamen Airlines |
| 山东航空 | SC | Shandong Airlines |
| 长龙航空 | GJ | Loong Air |
| 春秋航空 | 9C | Spring Airlines |
| 吉祥航空 | HO | Juneyao Air |
| 首都航空 | JD | Capital Airlines |
| 华夏航空 | G5 | China Express |
| 四川航空 | 3U | Sichuan Airlines |
| 成都航空 | EU | Chengdu Airlines |
| 西藏航空 | TV | Tibet Airlines |
| 河北航空 | NS | Hebei Airlines |
| 西部航空 | PN | West Air |
| 祥鹏航空 | 8L | Lucky Air |
| 昆明航空 | KY | Kunming Airlines |
| 天津航空 | GS | Tianjin Airlines |
| 重庆航空 | OQ | Chongqing Airlines |
| 联合航空 | KN | China United Airlines |
| 桂林航空 | GT | Air Guilin |
| 全日本航空 | NH | ANA (All Nippon Airways) |
| 泰国狮子蒙特里航空 | SL | Thai Lion Air |

---

## Verification Checklist

- [ ] API returns `flag: 1` and `msg: "OK"` (or "ok")
- [ ] `jhsj_time` is `YYYY-MM-DD HH:MM:SS` (live endpoints only)
- [ ] Timetable `takeoff`/`arrive` are HHMM strings, not datetime
- [ ] Timetable `schedule` decoded: 1=Mon ... 7=Sun
- [ ] Timetable: checked ALL entries for a flight (not just the first)
- [ ] Arrivals use `sf_name` field, NOT `md_name`
- [ ] Codeshare duplicates grouped by time+destination before reporting
- [ ] `bgsj_time` used as fallback when `sjsj_time` is null
- [ ] Autocomplete: `city` param for city names, `identity` param for flight numbers

---
> Source: [janebooom/hgh-airport-api](https://github.com/janebooom/hgh-airport-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
