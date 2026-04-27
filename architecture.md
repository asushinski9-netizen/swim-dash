# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v20.2
~2768 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL
2.  DATA globals — RAW, QT_DATA, SE_QT_DATA, DATA, cache{}
3.  STATE MANAGEMENT — invalidateCache(), updateData(newRaw)
4.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData()
5.  UTILITIES — timeToSec(), secToTime(), parseLocalDate(), fmtDate(), fmtDateShort(),
                getStroke(), getDistance(), processData()
6.  Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears() (memoised)
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

## Tab IDs
tab-overview, tab-progression, tab-pbs, tab-splits, tab-results, tab-qualifying, tab-regional

## Stable DOM wrapper IDs
splitBarChartWrap / progChartWrap / paceChartWrap
Pattern: querySelector('canvas') → restore → destroyChart → create or replace

## Debut detection — four contexts

| Context                  | Method                                                  | Reason                            |
|--------------------------|---------------------------------------------------------|-----------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                    | First chronological swim          |
| Overview Recent PBs      | swimCount === 1                                         | Only swim ever                    |
| Progression Improv / BvF | swims===1 \|\| !refSwim \|\| (ref===pb && mode!='latest')  | No meaningful reference       |
| Splits analyzeSwim       | prevPB===null (r.date < swim.date filter)               | No earlier swim with splits       |

## refSwim===pb guard behaviour by mode (v20.1)
| compareMode | refSwim===pb means           | UI shows              |
|-------------|------------------------------|-----------------------|
| first       | backdated entry is first+fastest | Debut             |
| prevSwim    | only one swim exists         | Debut                 |
| prevBest    | only one swim exists         | Debut                 |
| latest      | PB is the most recent swim   | "— PB is latest"      |

## AI Coach statusLabel (v20.1)
swim.isPB === true  → ⭐ CURRENT PB (green) — the overall fastest swim for event+course
isDebut === true    → 🚀 DEBUT (gold)
otherwise           → ABOVE PB (grey)
swim.isPB is set by processData() running minimum scan. True for exactly one swim per
event+course combination at any given time.

## updateData(newRaw) — central state update
  RAW = newRaw → localStorage → invalidateCache() → processData() → populateSelects()
  → renderOverview/Progression/PBs/Results/Qualifying/RegionalQualifying/updateSortHeaders
  Does NOT call renderSplits() — would clear tab if no event selected.

## init() data loading priority
1. RAW: localStorage authoritative; GitHub fetch only if localStorage empty
2. QT/SE_QT: localStorage first; GitHub fetch if either key missing
3. If RAW empty after both sources: show upload modal

## Responsive QT table headers (v20.1)
.col-full { display: inline } — desktop: full text, e.g. "Qualify Time"
.col-abbr { display: none  } — desktop: hidden
@media (max-width: 500px):
.col-full hidden, .col-abbr shown — mobile: abbreviated text, e.g. "QT"
Applied to 4 headers in both County QT and Regional QT tables.

## CSS variables (light theme)
--bg:#f0f4f8  --surface:#ffffff  --surface2:#e8eef5  --surface3:#d6e2ef
--accent:#1a56c4  --gold:#d97706  --green:#059669  --red:#dc2626
--text:#0f172a  --text2:#334155  --text3:#64748b  --border:#cbd5e1

## Chart tokens: CC.grid #cbd5e1 / CC.tick #475569 / CC.legend #334155
## PALETTE: ['#1a56c4','#0891b2','#7c3aed','#059669','#d97706','#dc2626','#db2777','#ea580c','#0d9488','#9333ea']
## sortDir default: -1 (newest-first in All Results)
## Data sources: swimDash_RAW (user authoritative) / swimDash_QT / swimDash_SE_QT (GitHub)
