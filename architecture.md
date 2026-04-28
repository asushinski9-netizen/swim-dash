# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v21
~2787 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL
2.  DATA globals — RAW, QT_DATA, SE_QT_DATA, DATA, cache{}
3.  STATE MANAGEMENT — invalidateCache(), updateData(newRaw)
4.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData()
5.  UTILITIES — escapeHtml(), timeToSec(), secToTime(), parseLocalDate(),
                fmtDate(), fmtDateShort(), getStroke(), getDistance(), processData()
6.  Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears(),
                      uniqueEventsWithSplits() — all memoised via cache{}
7.  PALETTE + CC chart colour tokens
8.  MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
9.  FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters(),
                            resetRegionalFilters()
10. CHARTS REGISTRY — charts{}, destroyChart()
11. POPULATE SELECTS — populateSelects()
12. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits(), renderSplitCharts(), analyzeSwim(),
                  renderResults(), buildQtStatCards(), renderQualifying(),
                  renderRegionalQualifying()
13. TAB SWITCHING — showTab()
14. INIT — async init()
15. ADD RACE — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
16. REMOVE RACE — removeRace(date, event, course, time)
17. DOMContentLoaded → init()

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

## Security: escapeHtml()
Applied to all user-supplied fields in innerHTML: competition, venue.
Event and course are dropdown-constrained and do not require escaping.

## Responsive QT table headers
col-full (desktop) / col-abbr (mobile) for 4 columns in both County and Regional tables.

## CSS: light theme variables, CC tokens, PALETTE — unchanged from v14
## sortDir default and reset: -1 (newest-first)
## Tab style: var(--surface2)/var(--border)/var(--text2) inactive; #4f86d8 active
## Data authority: swimDash_RAW (user) / swimDash_QT / swimDash_SE_QT (GitHub)
