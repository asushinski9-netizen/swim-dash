# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v23
~2733 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL
2.  DATA globals — RAW, QT_DATA, SE_QT_DATA, DATA, cache{}
3.  THEME — getCC(), getPalette(), applyTheme(), toggleTheme()
4.  STATE MANAGEMENT — invalidateCache(), updateData(newRaw)
5.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData()
6.  UTILITIES — escapeHtml(), timeToSec(), secToTime(), parseLocalDate(),
                fmtDate(), fmtDateShort(), getStroke(), getDistance(), processData()
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
                  renderQualifying(), renderRegionalQualifying()
14. TAB SWITCHING — showTab()
15. INIT — async init()
16. ADD RACE — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
17. REMOVE RACE — removeRace(date, event, course, time)
18. DOMContentLoaded → init()

## processData() sort order (v21)
date ASC → event name ASC → course ASC (S before L)
Stable tiebreaker on course prevents non-deterministic deltaPrev/isPB for same-date entries.

## isPB vs getPBs().includes(swim) — critical distinction (v21)
r.isPB = true for every swim that was the fastest at the time it was swum (historical).
getPBs().includes(swim) = true for exactly one swim per event+course (the current fastest).

Use r.isPB for: Progression chart point colouring (historical PB dots), deltaPrev-adjacent logic.
Use getPBs().includes(swim) for: AI Coach badge, Overview "recent PBs", pbsThisYear count.
NEVER mix these up — see known-bugs-and-fixes.md for details.

## Stable DOM wrapper IDs
splitBarChartWrap / progChartWrap / paceChartWrap
Pattern: querySelector('canvas') → restore → destroyChart → create or replace

## Debut detection — four contexts
| Context                  | Method                                                    | Reason                          |
|--------------------------|-----------------------------------------------------------|---------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                      | First chronological swim        |
| Overview Recent PBs      | swimCount === 1                                           | Only swim ever for event+course |
| Progression Improv / BvF | swims===1 \|\| !refSwim \|\| (ref===pb && mode!='latest') | No meaningful reference         |
| Splits analyzeSwim       | prevPB===null (date filtered)                             | No earlier swim with splits     |

## refSwim===pb guard by compare mode
latest → "PB is latest"; first/prevSwim/prevBest → Debut

## AI Coach statusLabel
getPBs().includes(swim) → ⭐ CURRENT PB (green)
isDebut → 🚀 DEBUT (gold)
otherwise → no badge

## updateData(newRaw) — central state update
RAW → localStorage → invalidateCache → processData → populateSelects → all tab renders
Does NOT call renderSplits().

## init() data loading priority
1. RAW: localStorage → GitHub fallback → upload modal
2. QT/SE_QT: localStorage → GitHub fetch if either missing


## DRY QT Rendering (v22)
renderQTChartAndTable(matched, chartId, legendId, tableBodyId) — shared helper
Contains all chart creation, legend, and table row rendering for qualifying tabs.
renderQualifying() and renderRegionalQualifying() handle only filter/data setup (~15 lines
each) and call this helper. Any change to chart format or table columns: edit here only.

## Progress bar scale (v22)
width = Math.min(impPct * 12.5, 100)%
8% improvement = full bar. Reflects practical competitive swimming improvement range.

## Security: escapeHtml()
Applied to all user-supplied fields in innerHTML: competition, venue.
Event and course are dropdown-constrained and do not require escaping.

## Responsive QT table headers
col-full (desktop) / col-abbr (mobile) for 4 columns in both County and Regional tables.

## CSS: light theme variables, CC tokens, PALETTE — unchanged from v14
## sortDir default and reset: -1 (newest-first)
## Tab style: var(--surface2)/var(--border)/var(--text2) inactive; #4f86d8 active
## Data authority: swimDash_RAW (user) / swimDash_QT / swimDash_SE_QT (GitHub)
