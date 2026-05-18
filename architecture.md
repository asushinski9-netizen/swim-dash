# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v26
~3150 lines total (approx)

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL, UPCOMING_DATA_URL,
                    COUNTY_CHAMPS_DATE, REGIONAL_CHAMPS_DATE, SEASON_START_MONTH
2.  DATA globals — RAW, QT_DATA, SE_QT_DATA, UPCOMING, DATA, cache{}
3.  THEME — getCC(), getPalette(), applyTheme(), toggleTheme()
4.  STATE MANAGEMENT — invalidateCache(), updateData(newRaw)
5.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData(), downloadUpcomingData()
6.  UTILITIES — escapeHtml(), timeToSec(), secToTime(), parseLocalDate(),
                fmtDate(), fmtDateShort(), getStroke(), getDistance(), processData(),
                getAgeAtEndOfYear(), getQTSeasonYear(), getSeasonStart(),
                getCountyAgeBracket(), getRegionalAgeBracket(),
                getQTStatusForEvent(), renderQTCells()
7.  Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears(),
                      uniqueEventsWithSplits() — all memoised via cache{}
8.  PALETTE_LIGHT/DARK + getCC()/getPalette() — theme-aware chart tokens
9.  MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
10. v26: getBestSeasonImprovement() — season-start PB delta across all event+course pairs
11. FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters(),
                            resetRegionalFilters()
12. CHARTS REGISTRY — charts{}, destroyChart()
13. POPULATE SELECTS — populateSelects()
14. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits(), renderSplitCharts(), analyzeSwim(),
                  renderResults(), buildQtStatCards(),
                  renderQTChartAndTable() [shared QT chart+table helper],
                  renderQualifying(), renderRegionalQualifying(),
                  buildSwumUpcomingSet(), renderSchedule(), renderTargets()
15. TAB SWITCHING — showTab()
16. INIT — async init()
17. ADD RACE MODAL (multi-event) — showAddRaceModal(), hideAddRaceModal(),
                                   addRaceEventRow(), saveNewRace()
18. REMOVE RACE — removeRace(date, event, course, time)
19. REMOVE UPCOMING EVENT — removeUpcomingEvent(date, event, course)
20. SPEED DIAL FAB — toggleFabDial(), openFabDial(), closeFabDial()
21. SETTINGS MODAL — showSettingsModal(), hideSettingsModal(), saveSettings()
22. ADD UPCOMING RACE MODAL — showAddUpcomingModal(), hideAddUpcomingModal(),
                               addUpcomingEventRow(), saveNewUpcoming()
23. DOMContentLoaded → init()

## Speed Dial FAB (v26: 5 children)
Single parent ➕ button (bottom-right, always visible) expands 5 child buttons upward.
State: `let fabDialOpen = false`
Functions: toggleFabDial(), openFabDial(), closeFabDial()
- openFabDial(): adds .open class to #fabChildren, #fabParent, #fabBackdrop
- closeFabDial(): removes .open classes
Child button order (bottom to top, CSS nth-child 1→5):
  1. ⏱️ fabAddRace    — Add Race (multi-event)
  2. 📅 fabAddUpcoming — Add Upcoming Race (NEW v26)
  3. ⚙️ (no id)        — Data Manager
  4. 🌙 fabTheme        — Toggle Theme
  5. 👤 fabSettings     — Swimmer Settings
Stagger delays: nth-child(1)=0s, (2)=0.05s, (3)=0.10s, (4)=0.15s, (5)=0.20s
fabTheme id retained so applyTheme() can sync the icon text.
closeFabDial() MUST be called first in every action function.

## Overview stat cards (v26)
6 cards in .grid6, ALL now clickable (onclick → showTab):
  1. Total Races        → All Results tab
  2. Current PBs        → Personal Bests tab
  3. Best Improvement   → Progression tab     (NEW v26: replaces "Events" card)
  4. Races with Splits  → Splits tab
  5. Upcoming Races     → Schedule tab
  6. PBs to Renew       → Targets tab

## getBestSeasonImprovement() (v26)
Returns { improvement (seconds), event, course } or null.
Season start: getSeasonStart() → Sept 1 of current season year (SEASON_START_MONTH=9).
Requires ≥2 swims this season per event+course. Uses first swim of season as baseline,
current PB as endpoint. Largest positive delta wins.
NOT memoised — called in renderOverview() only, which is guarded by overviewRendered flag.

## getSeasonStart() (v26)
Returns Date for September 1 of the current swim season.
If today is in Sept-Dec: current year. If Jan-Aug: previous year.
Controlled by SEASON_START_MONTH constant (1-indexed, default 9).

## Add Race modal — multi-event (v26)
Shared fields: Date, Venue, Course, Competition Name (entered once).
Per-event rows: dynamically added by addRaceEventRow(), max 6.
Each row contains: Event select, Time input, Splits input, ✕ remove button.
saveNewRace() validates all rows, pushes one RAW entry per row.
Duplicate event detection within a batch → error if same event appears twice.

## Add Upcoming Race modal (v26)
showAddUpcomingModal() — resets and shows modal, calls addUpcomingEventRow() once.
addUpcomingEventRow() — appends select + ✕ row, max 6.
saveNewUpcoming() — validates, merges into existing meet on same date+course, or pushes new.
UPCOMING sorted by date after each new entry. Persists to swimDash_UPCOMING.
Calls renderSchedule() + renderTargets() after save. Confirmation flash on #fabAddUpcoming.

## Remove Upcoming Event (v26)
removeUpcomingEvent(date, event, course) — splices from matching UPCOMING meet's events[].
If meet.events becomes empty after removal, the meet is also removed from UPCOMING.
Persists to localStorage. Confirms with user and reminds to download JSON for permanence.
Re-renders Schedule and Targets.

## Schedule tab — already-swum filter (v26)
buildSwumUpcomingSet(rows) — cross-references each upcoming row against DATA.
Match criteria: same event, same course, date within 1 day of meet date (±86400000ms).
Matched entries are added to swumSet as "date|event|course" strings.
renderSchedule() filters visibleRows = rows.filter not in swumSet.
hiddenCount reported in scheduleNote. Empty-state colspan = 12.

## Schedule table — Delete column (v26)
New 12th column "Del" with a ✕ button per row calling removeUpcomingEvent().
Empty-state colspan updated from 11 to 12.
Mobile CSS: #tab-schedule .tbl-wrap .badge scaling (0.50rem).

## downloadUpcomingData() (v26)
Serialises UPCOMING to JSON blob and triggers download as upcoming_races.json.
Button in Data Manager modal, immediately after ⬇️ Download my_swims.json.

## processData() sort order (v21)
Stable: date ASC → event name ASC → course ASC (S before L)

## isPB vs getPBs().includes(swim)
r.isPB: historical (was fastest at time of swimming)
getPBs().includes(swim): current fastest ever
Use getPBs() for: AI Coach badge, Overview recent PBs, pbsThisYear count.

## Stable DOM wrapper IDs
splitBarChartWrap / progChartWrap / paceChartWrap

## Debut detection — four contexts (unchanged from v24)
| Context                  | Method                                                |
|--------------------------|-------------------------------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                  |
| Overview Recent PBs      | swimCount === 1                                       |
| Progression Improv / BvF | swims===1 || !refSwim || (ref===pb && mode!='latest') |
| Splits analyzeSwim       | prevPB===null (date filtered)                         |

## updateData(newRaw)
RAW → localStorage → invalidateCache → processData → populateSelects → tab renders
Does NOT call renderSplits(), renderSchedule(), renderTargets().
renderSchedule/renderTargets are also called by: saveNewUpcoming(), removeUpcomingEvent(),
saveSettings(). They are NOT triggered by race data changes.

## getCC() and getPalette() — call inside render functions (v23)
Never at module level. Every render function injects at top:
  const CC = getCC(); const PALETTE = getPalette();

## DRY QT Rendering (v22)
renderQTChartAndTable() shared by renderQualifying() and renderRegionalQualifying().

## sortDir default and reset: -1 (newest-first)
## Data authority: swimDash_RAW (user) / swimDash_QT / swimDash_SE_QT / swimDash_UPCOMING (GitHub)
## swimDash_UPCOMING is also mutated locally by saveNewUpcoming() and removeUpcomingEvent().
