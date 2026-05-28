# Data Schemas

## RAW swim entry (my_swims.json)
```json
{
  "course": "S",
  "event": "200 Back",
  "date": "2025-11-15",
  "competition": "Watford Swimming Club County Qualifier 2025",
  "venue": "Watford",
  "time": "2:55.68",
  "splits": [41.88, 46.55, 46.78, 40.47]
}
```
- course: "S" = Short Course 25m, "L" = Long Course 50m
- date: ISO 8601, YYYY-MM-DD
- time: "mm:ss.hh" or "ss.hh". Regex: /^(\d{1,2}:)?\d{1,2}\.\d{2}$/
- splits: INDIVIDUAL LAP TIMES in seconds (not cumulative). Parsed via timeToSec().
  If missing or not an array, processData() normalises to [].

### Valid event strings (18 total — defined in ALL_EVENTS constant)
50/100/200/400/800/1500 Free | 50/100/200 Back | 50/100/200 Breast
50/100/200 Fly | 100/200/400 IM

## Processed DATA row (after processData())
Same as RAW plus:
- timeInSec: number
- isPB: boolean — fastest at time of swimming (historical)
- deltaPrev: number|null — seconds vs previous swim of same event+course
- splits: always an array (normalised from raw)

## County QT entry (county_qt.json)
```json
{"gender":"Boys","course":"Short Course","event":"50 Back","age":"12","qualify":38.10,"consider":40.80}
```
- course: "Short Course"/"Long Course" (NOT "S"/"L")
- age: "10+11","12","13","14","15","16","17+"
- qualify/consider: float seconds; null for events not offered to 10+11

## SE London Regional QT entry (se_london_qt.json)
```json
{"gender":"Boys","course":"Long Course","event":"50 Back","age":"11/12","qualify":36.76,"consider":38.60}
```
- age: "11/12","13","14","15","16","17","18+"
- qualify/consider: always float, never null

## Upcoming races entry (upcoming_races.json)
```json
{
  "date": "2026-06-14",
  "competition": "Hillingdon Swimming Club Summer Spectacular Meet 2026",
  "venue": "Uxbridge",
  "course": "S",
  "events": [
    { "event": "100 Back" },
    { "event": "50 Breast" }
  ]
}
```
- course: "S" or "L"
- events: array of { "event": "<event string>" }
  Each element MUST have an `event` property (string).
  removeUpcomingEvent() relies on `e.event` for filtering.
  saveNewUpcoming() always pushes { event: select.value }.
- Meets are flattened to one row per event in the Schedule table.
- 48-hour grace: meets with date < today-2 days are excluded everywhere.
- buildSwumUpcomingSet(): cross-references DATA vs upcoming rows (±1 day date window).
- UPCOMING is mutated by saveNewUpcoming() and removeUpcomingEvent().
  Changes persist to swimDash_UPCOMING in localStorage but do NOT push to GitHub.
  User must download upcoming_races.json from Data Manager and commit.

## Swimmer profile (localStorage)
- swimDash_DOB: ISO 8601 date, e.g. "2014-10-12". Default: '2014-10-12' if unset.
- swimDash_GENDER: "Boys" or "Girls". Must match QT JSON gender field exactly.
Both read via getSwimmerDOB() / getSwimmerGender() — not module-level constants.

## Season configuration (JS constant)
- SEASON_START_MONTH: integer 1–12, default 9 (September, 1-indexed).
  Used by getSeasonStart() to determine swim season start date.
  Date string built from local components (not toISOString()) to avoid UTC offset bug.

## ALL_EVENTS constant (JS constant — CONFIGURATION section)
Canonical list of all 18 supported events. Single source of truth.
Used by: addRaceEventRow(), addUpcomingEventRow(), renderTargets().
Do NOT define inline event arrays elsewhere — edit here only.

## Course code conversion
RAW/DATA: "S"/"L" ↔ QT files: "Short Course"/"Long Course"
Conversion: course === 'Short Course' ? 'S' : 'L'

## QT tab matched row structure (built inside renderQualifying / renderRegionalQualifying)
```js
{
  gender, course, event, age,       // from QT JSON
  qualify: float|null,              // qualifying time in seconds
  consider: float|null,             // consideration time in seconds
  pb: DATA row | null,              // swimmer's PB for this event+course
  pbSec: float | null,              // PB in seconds (null if no PB)
  gapQ: float | null,               // pbSec - qualify (negative = inside QT)
  gapC: float | null,               // pbSec - consider (negative = inside CT)
  status: 'Qualified'|'Consideration'|'Outside'|'No PB'
}
```
Two arrays built per render: scMatched (Short Course) and lcMatched (Long Course).
Both passed to buildQtStatCards() and renderQTCards().

## qtToggleState (module-level JS object)
```js
const qtToggleState = { qt: {SC: true, LC: true}, rq: {SC: true, LC: true} };
```
Tracks SC/LC visibility for chart datasets and table rows.
Mutated by toggleQTChart() and toggleQTRows().
Persists within the session; resets on page reload.

## GitHub URLs
RACE_DATA_URL:    .../asushinski9-netizen/swim-dash/main/my_swims.json
QT_DATA_URL:      .../asushinski9-netizen/swim-dash/main/county_qt.json
SE_QT_DATA_URL:   .../asushinski9-netizen/swim-dash/main/se_london_qt.json
UPCOMING_DATA_URL:.../asushinski9-netizen/swim-dash/main/upcoming_races.json

## localStorage keys
swimDash_RAW      — USER AUTHORITATIVE (never overwritten by GitHub)
swimDash_QT       — GitHub authoritative (re-fetched if missing)
swimDash_SE_QT    — GitHub authoritative (re-fetched if missing)
swimDash_UPCOMING — GitHub authoritative on first load; also mutated locally
swimDash_theme         — 'light' or 'dark'
swimDash_DOB           — Swimmer date of birth
swimDash_GENDER        — Swimmer gender
swimDash_renewalMonths — PB renewal threshold ('3','6','9','12'). Restored in init().
