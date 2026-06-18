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
- If the file fails to parse (invalid JSON), init() calls
  `showJsonLoadError('my_swims.json', message)` and falls back to an empty array
  or whatever was already cached — see "JSON load error handling" below.

### Valid event strings (18 total — defined in ALL_EVENTS constant)
50/100/200/400/800/1500 Free | 50/100/200 Back | 50/100/200 Breast
50/100/200 Fly | 100/200/400 IM

## Processed DATA row (after processData())
Same as RAW plus:
- timeInSec: number
- isPB: boolean — fastest at time of swimming (historical)
- deltaPrev: number|null — seconds vs previous swim of same event+course
- splits: always an array (normalised from raw)

---

## QT JSON schema (v28 — wrapped format)

`county_qt.json` and `regional_qt.json` (renamed from `se_london_qt.json` — see
below) both now use a wrapped `{meta, times}` shape instead of a bare array:

```json
{
  "meta": {
    "title": "Middlesex County Championships",
    "dateFrom": "2027-02-06",
    "dateTo": "2027-02-14"
  },
  "times": [
    {"gender":"Boys","course":"Short Course","event":"50 Back","age":"12","qualify":38.10,"consider":40.80}
  ]
}
```
- `meta.title`: string, championship name shown in the QT tab banner
- `meta.dateFrom` / `meta.dateTo`: ISO 8601 dates (YYYY-MM-DD); either can be
  absent/empty, in which case the banner shows a "No dates set" hint
- `times`: array of QT entries in the same row shape as before (see below)

### Backward compatibility
The old bare-array format (no `meta` wrapper) is still accepted. Both formats
are normalised by `unwrapQT(raw)`, which returns `{meta: {}, times: [...]}` for
old-format files and passes new-format files through unchanged. `QT_DATA` and
`SE_QT_DATA` are always flat arrays after this step — nothing downstream needs
to know which format the source file used.

### County QT entry (inside `times`, county_qt.json)
```json
{"gender":"Boys","course":"Short Course","event":"50 Back","age":"12","qualify":38.10,"consider":40.80}
```
- course: "Short Course"/"Long Course" (NOT "S"/"L")
- age: "10+11","12","13","14","15","16","17+"
- qualify/consider: float seconds; null for events not offered to 10+11

### Regional QT entry (inside `times`, regional_qt.json)
```json
{"gender":"Boys","course":"Long Course","event":"50 Back","age":"11/12","qualify":36.76,"consider":38.60}
```
- age: "11/12","13","14","15","16","17","18+"
- qualify/consider: float, or null for events not offered to an age group
  (the regional file previously always supplied a float, but the editor can
  now produce null rows via "not offered" — see architecture.md QT Editor section)

## File rename (v28)
`se_london_qt.json` → **`regional_qt.json`**. `SE_QT_DATA_URL` was updated to
point at the new filename. The internal JS variable name `SE_QT_DATA` (and
`SE_QT_META`, `renderRegionalQualifying()`, etc.) were intentionally left
unchanged — only the file on GitHub and user-facing labels changed.

## QT_META / SE_QT_META (new in v28)
```js
let QT_META    = {};  // { title, dateFrom, dateTo } for County
let SE_QT_META = {};  // { title, dateFrom, dateTo } for Regional
```
Populated from `meta` by `unwrapQT()` during `init()`. Consumed only by
`renderQTBanner(viewPrefix)` to populate the championship title/date-range
banner shown above both QT tabs. Editable via the inline QT editor's meta
panel (`toggleMetaPanel()` / `saveMetaPanel()`), which commits changes to
these globals immediately on Save.

## JSON load error handling (new in v28)
Any `JSON.parse()` failure on a fetched or cached data file calls
`showJsonLoadError(filename, message)`, which renders a persistent, stackable
red banner at the top of the Overview tab (`#jsonLoadErrorBanner`) rather than
failing silently or throwing. Covers `my_swims.json`, `county_qt.json`,
`regional_qt.json`, and `upcoming_races.json`, in both their GitHub-fetch and
localStorage-cache code paths. The affected dataset falls back to an empty
array (or stays at whatever was already loaded) rather than blocking the rest
of `init()`.

---

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
- If the file fails to parse, init() calls showJsonLoadError('upcoming_races.json', message)
  and UPCOMING falls back to [] or the existing cache.

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
  gender, course, event, age,       // from QT JSON (the unwrapped .times entry)
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
This structure is unaffected by the v28 schema migration — it's built from the
already-unwrapped flat QT_DATA/SE_QT_DATA arrays.

## qtRowState (module-level JS object)
```js
const qtRowState = { qt: {SC: true, LC: true}, rq: {SC: true, LC: true} };
```
Tracks SC/LC row visibility for the event-by-event tables only (the QT tab
chart was removed in v27; this is row-only state, despite the similarly-named
predecessor `qtToggleState` once also covering chart datasets).
Mutated by toggleQTRows(). Persists within the session; resets on page reload.

## QT editor store (v28.1, session-scoped, not persisted to localStorage)
When `openQTEditor(viewPrefix)` is first called in a session, the live
`QT_DATA`/`SE_QT_DATA` + meta are deep-cloned into an in-memory `qtEditorStore`.
Edits (add/delete row, time edits, meta panel Save) mutate this store. Row-level
edits are written back to the live globals only when `closeQTEditor()` runs
`commitEditorStoreToLive(prefix)` — i.e. on clicking "← Back". Meta panel edits
commit immediately on "Save" regardless of Back. See architecture.md and
known-bugs-and-fixes.md for the caveat this creates.

## GitHub URLs
RACE_DATA_URL:    .../asushinski9-netizen/swim-dash/main/my_swims.json
QT_DATA_URL:      .../asushinski9-netizen/swim-dash/main/county_qt.json
SE_QT_DATA_URL:   .../asushinski9-netizen/swim-dash/main/regional_qt.json   (renamed in v28)
UPCOMING_DATA_URL:.../asushinski9-netizen/swim-dash/main/upcoming_races.json

## localStorage keys
swimDash_RAW      — USER AUTHORITATIVE (never overwritten by GitHub)
swimDash_QT       — GitHub authoritative (re-fetched if missing); stores the raw
                     {meta,times} (or legacy flat-array) JSON text as fetched
swimDash_SE_QT    — GitHub authoritative (re-fetched if missing); same storage
                     convention, sourced from regional_qt.json
swimDash_UPCOMING — GitHub authoritative on first load; also mutated locally
swimDash_theme         — 'light' or 'dark'
swimDash_DOB           — Swimmer date of birth
swimDash_GENDER        — Swimmer gender
swimDash_renewalMonths — PB renewal threshold ('3','6','9','12'). Restored in init().
