# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v25
~2896 lines total

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
16. SPEED DIAL FAB — toggleFabDial(), openFabDial(), closeFabDial()
17. SETTINGS MODAL — showSettingsModal(), hideSettingsModal(), saveSettings()
18. ADD RACE — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
19. REMOVE RACE — removeRace(date, event, course, time)
20. DOMContentLoaded → init()

## Speed Dial FAB (v25)
Single parent ➕ button (bottom-right, always visible) expands 4 child buttons upward.
State: `let fabDialOpen = false`
Functions: toggleFabDial(), openFabDial(), closeFabDial()
- openFabDial(): adds .open class to #fabChildren, #fabParent, #fabBackdrop
- closeFabDial(): removes .open classes
- Every modal/action opener calls closeFabDial() first
Child button IDs: fabAddRace (⏱️), fabTheme (🌙/☀️), fabSettings (👤)
No ID on Data Manager child (no external reference needed)
fabTheme id retained so applyTheme() can sync the icon text.
Confirmation flash: on #fabAddRace child, not on parent.
Alignment: .fab-child-row has margin-right: 4px so 48px children centre on 56px parent.
Mobile: parent 50px, children 44px, dial anchors bottom:16px right:16px.

## Swimmer profile (v24)
getSwimmerDOB() and getSwimmerGender() read from localStorage (swimDash_DOB / swimDash_GENDER).
Defaults: '2014-10-12' / 'Boys'. Set via 👤 child FAB → Settings modal.
saveSettings() persists and re-renders QT tabs, Schedule, Targets, Overview.

## Age bracket calculation (v24)
getAgeAtEndOfYear(year), getQTSeasonYear(champDateStr),
getCountyAgeBracket(), getRegionalAgeBracket()
Called at render time — NOT cached at module level.

## QT status helpers (v24)
getQTStatusForEvent(event, course, pbSec) → { county: {...}, regional: {...} }
renderQTCells(s) → [statusCell, gapCell] — NO inline font-size, inherits from table CSS.
Gap logic: Outside → CT gap; Consideration → QT gap; Qualified → ✓

## Upcoming races (v24)
localStorage: swimDash_UPCOMING. Fetched from UPCOMING_DATA_URL on first load.
48-hour grace period in renderSchedule() and renderTargets().
upcomingMap: { event: nearestFutureMeet } built inside each render function.

## ALL_EVENTS (v24.1)
18 standard events — independent of QT data:
50/100/200/400/800/1500 Free | 50/100/200 Back | 50/100/200 Breast
50/100/200 Fly | 100/200/400 IM
Used in Targets Section 2 (Events Not Yet Swum flex card grid).
Must stay in sync with Add Race modal dropdown options.

## processData() sort order (v21)
Stable: date ASC → event name ASC → course ASC (S before L)

## isPB vs getPBs().includes(swim)
r.isPB: historical (was fastest at time of swimming)
getPBs().includes(swim): current fastest ever
Use getPBs() for: AI Coach badge, Overview recent PBs, pbsThisYear count.

## Stable DOM wrapper IDs
splitBarChartWrap / progChartWrap / paceChartWrap

## Debut detection — four contexts
| Context                  | Method                                                |
|--------------------------|-------------------------------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                  |
| Overview Recent PBs      | swimCount === 1                                       |
| Progression Improv / BvF | swims===1 || !refSwim || (ref===pb && mode!='latest') |
| Splits analyzeSwim       | prevPB===null (date filtered)                         |

## updateData(newRaw)
RAW → localStorage → invalidateCache → processData → populateSelects → tab renders
Does NOT call renderSplits(), renderSchedule(), renderTargets().

## Overview stat grid (v24)
6 cards in .grid6: Total Races, Current PBs, Events, Races with Splits,
Upcoming Races (→ Schedule), PBs to Renew (→ Targets, 6-month threshold).

## Competition truncation (v24)
.comp-full (30 chars, desktop) / .comp-short (20 chars, mobile).

## FAB stack — Speed Dial (v25)
Parent ➕ at bottom:24px right:24px (desktop), bottom:16px right:16px (mobile).
Children stack upward inside .fab-dial flex column, gap:12px.
No individual position coordinates needed — flex layout handles positioning.

## getCC() and getPalette() — call inside render functions (v23)
Never at module level. Inject at top of every render function using chart colours.

## DRY QT Rendering (v22)
renderQTChartAndTable() shared by renderQualifying() and renderRegionalQualifying().

## sortDir default and reset: -1 (newest-first)
## Data authority: swimDash_RAW (user) / swimDash_QT / swimDash_SE_QT / swimDash_UPCOMING (GitHub)
