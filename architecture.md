# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v26.3
~3307 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL, UPCOMING_DATA_URL,
                    COUNTY_CHAMPS_DATE, REGIONAL_CHAMPS_DATE, SEASON_START_MONTH,
                    ALL_EVENTS (canonical event list — single source of truth)
2.  Swimmer profile functions — getSwimmerDOB(), getSwimmerGender()
3.  DATA globals — RAW, QT_DATA, SE_QT_DATA, UPCOMING, DATA
4.  STATE globals — renderedTabs (Set), sortCol, sortDir, fabDialOpen,
                    currentEventFiltered, contextualPBRef, splitBarChart, paceChart
5.  cache{} + invalidateCache() — calls renderedTabs.clear()
6.  THEME — getCC(), getPalette(), applyTheme(), toggleTheme()
7.  STATE MANAGEMENT — updateData(newRaw)
8.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData(), downloadUpcomingData()
9.  UTILITIES — escapeHtml(), timeToSec(), secToTime(), parseLocalDate(),
                fmtDate(), fmtDateShort(), getStroke(), getDistance(), processData(),
                getAgeAtEndOfYear(), getQTSeasonYear(), getSeasonStart(),
                getCountyAgeBracket(), getRegionalAgeBracket(),
                getQTStatusForEvent(), renderQTCells()
10. Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears(),
                      uniqueEventsWithSplits() — all memoised via cache{}
11. PALETTE_LIGHT/DARK + getCC()/getPalette()
12. MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
13. getBestSeasonImprovement()
14. FILTER RESET HELPERS — forceX() wrappers (forceProgression, forcePBs, forceResults,
                            forceQualifying, forceRegional), then resetX() functions
15. CHARTS REGISTRY — charts{}, destroyChart()
16. POPULATE SELECTS — populateSelects() (snapshots/restores values across rebuilds)
17. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits(), renderSplitCharts(), analyzeSwim(),
                  renderResults(), buildQtStatCards(),
                  renderQTChartAndTable(),
                  renderQualifying(), renderRegionalQualifying(),
                  buildSwumUpcomingSet(), renderSchedule(), renderTargets()
18. RENDER ALL CHART TABS — renderAllChartTabs()
19. TAB SWITCHING — showTab()
20. INIT — async init() — includes keydown (Escape) listener and delegated click
           listeners for .btn-remove-race and .btn-remove-upcoming
21. ADD RACE MODAL — showAddRaceModal(), hideAddRaceModal(),
                     addRaceEventRow(), saveNewRace()
22. REMOVE RACE — removeRace()
23. REMOVE UPCOMING EVENT — removeUpcomingEvent()
24. SPEED DIAL FAB — toggleFabDial(), openFabDial(), closeFabDial()
25. SETTINGS MODAL — showSettingsModal(), hideSettingsModal(), saveSettings()
26. ADD UPCOMING RACE MODAL — showAddUpcomingModal(), hideAddUpcomingModal(),
                               addUpcomingEventRow(), saveNewUpcoming()
27. Theme IIFE + DOMContentLoaded → init()

---

## Lazy rendering system (v26.2 + v26.3)

### renderedTabs Set
Central state variable. Tracks which tabs have completed an initial render since the
last data or theme change. Declared in the STATE globals block.

| Operation | When to use |
|---|---|
| `renderedTabs.clear()` | Full reset — in invalidateCache() and renderAllChartTabs() |
| `renderedTabs.delete('tab')` | Targeted reset — in forceX() wrappers, reset fns, sortResults() |
| `renderedTabs.has('tab')` | Guard at top of each guarded render function |
| `renderedTabs.add('tab')` | At the bottom of each guarded render function |

### Guarded render functions (v26.3)
renderOverview, renderProgression, renderPBs, renderResults,
renderQualifying, renderRegionalQualifying.

### Intentionally unguarded (v26.3)
- renderSplits() — filter-event driven; always re-renders on event select change
- renderSchedule() — depends on UPCOMING which changes independently
- renderTargets() — same; also has renewal threshold select

### forceX() wrappers (v26.3)
All filter onchange attributes call force wrappers, NOT render functions directly.
Wrappers call renderedTabs.delete() before rendering to bypass the lazy guard.
```js
function forceProgression()  { renderedTabs.delete('progression'); renderProgression(); }
function forcePBs()          { renderedTabs.delete('pbs');         renderPBs(); }
function forceResults()      { renderedTabs.delete('results');     renderResults(); }
function forceQualifying()   { renderedTabs.delete('qualifying');  renderQualifying(); }
function forceRegional()     { renderedTabs.delete('regional');    renderRegionalQualifying(); }
```

### updateData() active-tab-only re-render (v26.3)
```
invalidateCache()  →  renderedTabs.clear()
processData()
populateSelects()
updateSortHeaders()
showTab(activeId)  →  re-renders only the currently active tab
renderSchedule()   →  always, race changes affect upcoming/renewal display
renderTargets()    →  always
```
All other tabs render lazily when the user navigates to them.

---

## getBestSeasonImprovement() (v26.3 corrected)
Returns { improvement (seconds), event, course } or null.
Season start: getSeasonStart() → Sept 1 of current season year (SEASON_START_MONTH=9).
Requires ≥2 swims this season per event+course.
Endpoint: bestInSeason (fastest swim within the current season) — NOT the all-time PB.
If season opener is already the season's fastest, the event is skipped.
Baseline: first chronological swim of the season for that event+course.

```
improvement = firstOfSeason.timeInSec - bestInSeason.timeInSec
```

This ensures only genuine within-season progress is reported, regardless of historical PBs
from prior seasons.

---

## Speed Dial FAB (v26: 5 children)
Single parent ➕ button (bottom-right). Expands 5 child buttons upward.
State: `let fabDialOpen = false` (in STATE globals block)
Functions: toggleFabDial(), openFabDial(), closeFabDial()
Child button order (bottom→top, CSS nth-child 1→5):
  1. ⏱️ fabAddRace    — Add Race (multi-event)
  2. 📅 fabAddUpcoming — Add Upcoming Race
  3. ⚙️ (no id)        — Data Manager
  4. 🌙 fabTheme        — Toggle Theme
  5. 👤 fabSettings     — Swimmer Settings
Stagger delays: nth-child(1)=0s, (2)=0.05s, (3)=0.10s, (4)=0.15s, (5)=0.20s
fabTheme id retained so applyTheme() can sync the icon text.
closeFabDial() MUST be called first in every action function.

---

## Swimmer profile (v24)
getSwimmerDOB() and getSwimmerGender() read from localStorage.
Defaults: '2014-10-12' / 'Boys'. Set via 👤 Settings modal.
saveSettings() persists and calls renderAllChartTabs() + renderSchedule() + renderTargets().

## Age bracket calculation (v24)
getAgeAtEndOfYear(year), getQTSeasonYear(champDateStr),
getCountyAgeBracket(), getRegionalAgeBracket()
Called at render time — NOT cached at module level.

## QT status helpers (v24/v26.2)
getQTStatusForEvent(event, course, pbSec) → { county: {...}, regional: {...} }
renderQTCells(s) → [statusCell, gapCell]
  No Data  → italic "Not offered" (v26.2)
  No PB    → "No PB"
  Qualified / Consideration / Outside → coloured status + gap

## Upcoming races (v24/v26)
localStorage: swimDash_UPCOMING. Fetched from UPCOMING_DATA_URL on first load.
Also mutated locally by saveNewUpcoming() and removeUpcomingEvent().
48-hour grace period applied in renderSchedule(), renderTargets(), buildSwumUpcomingSet().
upcomingMap: { event: nearestFutureMeet } built inside renderTargets().

## ALL_EVENTS (v26.2)
18 standard events — single source of truth in CONFIGURATION section.
Used in: addRaceEventRow(), addUpcomingEventRow(), renderTargets().
Do NOT define inline event arrays anywhere else.

## processData() sort order (v21)
Stable: date ASC → event name ASC → course ASC (S before L).
Includes splits normalisation: Array.isArray(r.splits) ? r.splits : [].

## isPB vs getPBs().includes(swim)
r.isPB: historical (was fastest at time of swimming)
getPBs().includes(swim): current fastest ever
Use getPBs() for: AI Coach badge, Overview recent PBs, pbsThisYear count.

## Add Race modal — multi-event (v26)
Shared fields: Date, Venue, Course, Competition (entered once per session).
Per-event rows: dynamically added by addRaceEventRow(), max 6.
saveNewRace() validates all rows, pushes one RAW entry per row.
Duplicate event detection within a batch.

## Add Upcoming Race modal (v26)
saveNewUpcoming() merges into existing meet on same date+course, or pushes new.
UPCOMING sorted by date. Persists to swimDash_UPCOMING.
Calls renderedTabs.delete('overview') + renderOverview() + renderSchedule() + renderTargets().

## Remove Upcoming Event (v26)
removeUpcomingEvent(date, event, course) splices from matching meet's events[].
If meet.events becomes empty, the meet is removed from UPCOMING.
Calls renderedTabs.delete('overview') + renderOverview() + renderSchedule() + renderTargets().

## Schedule tab — already-swum filter (v26)
buildSwumUpcomingSet(rows) — cross-references upcoming rows against DATA.
Match: same event, same course, date within 1 day (±86400000ms).
renderSchedule() filters visibleRows = rows not in swumSet.
hiddenCount reported in scheduleNote. Empty-state colspan = 12.

## downloadUpcomingData() (v26)
Serialises UPCOMING to JSON blob, triggers download as upcoming_races.json.

## Stable DOM wrapper IDs
splitBarChartWrap / progChartWrap / paceChartWrap

## DRY QT Rendering (v22)
renderQTChartAndTable() shared by renderQualifying() and renderRegionalQualifying().
Both receive const CC = getCC(); const PALETTE = getPalette() at top (v26.2).

## Security: escapeHtml() + data attributes (v21/v26.2)
escapeHtml() applied to all user-supplied fields in innerHTML: competition, venue.
Remove buttons use data-* attributes + delegated listeners on resultsTbody / scheduleBody.
Set up once in init(). No inline onclick with user data.

## Responsive QT table headers
col-full (desktop) / col-abbr (mobile) for gap columns in both QT tabs.

## sortDir default and reset: -1 (newest-first)
sortResults() and resetResultsFilters() both call renderedTabs.delete('results').

## Data authority
swimDash_RAW      — USER AUTHORITATIVE (never overwritten by GitHub)
swimDash_QT       — GitHub authoritative (re-fetched if missing)
swimDash_SE_QT    — GitHub authoritative (re-fetched if missing)
swimDash_UPCOMING — GitHub authoritative on first load; also mutated locally (v26)
swimDash_theme    — 'light' or 'dark'
swimDash_DOB      — Swimmer date of birth
swimDash_GENDER   — Swimmer gender
