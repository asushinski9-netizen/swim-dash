# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v24.1
~2837 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL, UPCOMING_DATA_URL,
                    COUNTY_CHAMPS_DATE, REGIONAL_CHAMPS_DATE
2.  DATA globals — RAW, QT_DATA, SE_QT_DATA, UPCOMING, DATA, cache{}
3.  THEME — getCC(), getPalette(), applyTheme(), toggleTheme()
4.  STATE MANAGEMENT — invalidateCache(), updateData(newRaw)
5.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData()
6.  UTILITIES — escapeHtml(), timeToSec(), secToTime(), parseLocalDate(),
                fmtDate(), fmtDateShort(), getStroke(), getDistance(), processData(),
                getAgeAtEndOfYear(), getQTSeasonYear(),
                getCountyAgeBracket(), getRegionalAgeBracket(),
                getQTStatusForEvent(), renderQTCells()
7.  Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears(),
                      uniqueEventsWithSplits() — all memoised via cache{}
8.  PALETTE_LIGHT/DARK + getCC()/getPalette() — theme-aware chart tokens
9.  MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
10. FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters(),
                            resetRegionalFilters()
11. CHARTS REGISTRY — charts{}, destroyChart()
12. POPULATE SELECTS — populateSelects()
13. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits(), renderSplitCharts(), analyzeSwim(),
                  renderResults(), buildQtStatCards(),
                  renderQTChartAndTable() [shared QT chart+table helper],
                  renderQualifying(), renderRegionalQualifying(),
                  renderSchedule(), renderTargets()
14. TAB SWITCHING — showTab()
15. INIT — async init()
16. SETTINGS MODAL — showSettingsModal(), hideSettingsModal(), saveSettings()
17. ADD RACE — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
18. REMOVE RACE — removeRace(date, event, course, time)
19. DOMContentLoaded → init()

## Swimmer profile (v24)
SWIMMER_DOB and SWIMMER_GENDER are NOT constants — they are read from localStorage
via `getSwimmerDOB()` and `getSwimmerGender()` functions. Defaults: '2014-10-12' / 'Boys'.
Set via the 👤 Settings modal (fab-settings FAB). Changing them calls saveSettings() which
re-syncs QT filter dropdowns and re-renders County QT, Regional QT, Schedule, Targets, Overview.

## Age bracket calculation (v24)
- `getAgeAtEndOfYear(year)` — age on 31 Dec of that year from DOB
- `getQTSeasonYear(champDateStr)` — if championship has passed, returns next year; else current year
- `getCountyAgeBracket()` — maps age at season year end to "10+11"|"12"|...|"17+"
- `getRegionalAgeBracket()` — maps age to "11/12"|"13"|...|"18+"
These drive default filter values in resetQualifyingFilters() and resetRegionalFilters().

## QT status helpers (v24)
- `getQTStatusForEvent(event, course, pbSec)` — looks up county + regional status for a
  given event/course/time; returns { county: {status, qualify, consider, gapQ, gapC},
  regional: {...} }. Used by Schedule and Targets tabs.
- `renderQTCells(s)` — returns [statusCell, gapCell] HTML strings for two-column layout.
  Gap logic: Outside → gap to CT (closer target); Consideration → gap to QT; Qualified → ✓.
  NO inline font-size — inherits from table CSS for consistent mobile scaling.

## Upcoming races (v24)
- localStorage key: swimDash_UPCOMING — array of meet objects (see data-schema.md)
- Fetched from UPCOMING_DATA_URL on first load if not cached
- Uploadable via Data Manager (4th file slot)
- 48-hour grace period: meets whose date < today-2 are excluded from Schedule/Targets
- upcomingMap: built inside renderSchedule() and renderTargets() as { event: nearestMeet }

## ALL_EVENTS (v24) — Events Not Yet Swum
Complete list of 18 standard British Swimming events:
  50/100/200/400/800/1500 Free | 50/100/200 Back | 50/100/200 Breast
  50/100/200 Fly | 100/200/400 IM
Used in Targets Section 2. Independent of QT data — shows ALL events regardless of age bracket.
Events Not Yet Swum renders as a flex card grid (not a table) with stroke-colour-coded borders.

## processData() sort order (v21)
Stable: date ASC → event name ASC → course ASC (S before L)
deltaPrev and isPB calculations depend on this order being deterministic.

## isPB vs getPBs().includes(swim) — critical distinction (v21)
r.isPB = true for every swim that was the fastest at the time it was swum (historical).
getPBs().includes(swim) = true for exactly one swim per event+course (the current fastest).
Use r.isPB for: Progression chart point colouring.
Use getPBs().includes(swim) for: AI Coach badge, Overview "recent PBs", pbsThisYear count.

## Stable DOM wrapper IDs
splitBarChartWrap / progChartWrap / paceChartWrap
Pattern: querySelector('canvas') → restore → destroyChart → create or replace

## Debut detection — four contexts
| Context                  | Method                                                    | Reason                          |
|--------------------------|-----------------------------------------------------------|---------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                      | First chronological swim        |
| Overview Recent PBs      | swimCount === 1                                           | Only swim ever for event+course |
| Progression Improv / BvF | swims===1 || !refSwim || (ref===pb && mode!='latest')     | No meaningful reference         |
| Splits analyzeSwim       | prevPB===null (date filtered)                             | No earlier swim with splits     |

## refSwim===pb guard by compare mode
latest → "PB is latest"; first/prevSwim/prevBest → Debut

## AI Coach statusLabel
getPBs().includes(swim) → ⭐ CURRENT PB (green)
isDebut → 🚀 DEBUT (gold)
otherwise → no badge

## updateData(newRaw) — central state update
RAW → localStorage → invalidateCache → processData → populateSelects → all tab renders
Does NOT call renderSplits(), renderSchedule(), or renderTargets()
(Schedule/Targets depend on UPCOMING which only changes via file upload/reload)

## Overview stat grid (v24)
6 cards in .grid6: Total Races, Current PBs, Events, Races with Splits,
Upcoming Races (→ Schedule tab), PBs to Renew (→ Targets tab, 6-month threshold).

## Schedule tab note (v24)
Rendered by renderSchedule() into #scheduleNote.
Format: "County QT: age X bracket · Regional QT: age X bracket. [season note if applicable]."
Season note appears when COUNTY_CHAMPS_DATE has passed.

## Targets renewal note (v24)
Rendered by renderTargets() into #renewalNote. Identical wording to schedule note.
Appears below the PBs Due for Renewal table.

## init() data loading priority
1. RAW: localStorage → GitHub fallback → upload modal
2. QT / SE_QT / UPCOMING: localStorage → GitHub fetch if any are missing

## Competition truncation (v24)
Desktop: 30 chars + "…" via comp-full span class.
Mobile: 20 chars + "…" via comp-short span class.
CSS: .comp-full { display: inline } / .comp-short { display: none }
     @mobile: .comp-full { display: none } / .comp-short { display: inline }

## FAB stack (bottom-right, desktop)
  24px — ➕ Add Race (fab-add)
  88px — ⚙️ Data Manager (fab-data)
 152px — 🌙/☀️ Theme toggle (fab-theme)
 216px — 👤 Swimmer Settings (fab-settings)
Mobile offsets: 16px / 74px / 132px / 190px

## DRY QT Rendering (v22)
renderQTChartAndTable(matched, chartId, legendId, tableBodyId) — shared helper
Used by both renderQualifying() and renderRegionalQualifying().

## Progress bar scale (v22)
width = Math.min(impPct * 12.5, 100)%
8% improvement = full bar.

## Security: escapeHtml()
Applied to all user-supplied fields in innerHTML: competition, venue.
Event and course are dropdown-constrained and do not require escaping.

## Responsive QT table headers
col-full (desktop) / col-abbr (mobile) for 4 columns in both County and Regional tables.

## getCC() and getPalette() — call inside render functions (v23)
Never at module level. Every render function that uses chart colours must inject:
  const CC = getCC(); const PALETTE = getPalette();
at its top so re-renders always pick up the active theme.

## Theme is applied in two stages (v23)
1. IIFE before DOMContentLoaded: adds body.dark class immediately from localStorage.
2. init() calls applyTheme(savedTheme): syncs FAB icon once DOM is ready.

## sortDir default and reset: -1 (newest-first)
## Data authority: swimDash_RAW (user) / swimDash_QT / swimDash_SE_QT / swimDash_UPCOMING (GitHub)
