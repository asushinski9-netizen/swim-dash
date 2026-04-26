# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v19
~2712 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL
2.  DATA globals — RAW, QT_DATA, SE_QT_DATA, DATA, cache{}
3.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData()
4.  UTILITIES — timeToSec(), secToTime(), parseLocalDate(), fmtDate(), fmtDateShort(),
                getStroke(), getDistance(), processData()
5.  Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears() (memoised)
6.  PALETTE + CC chart colour tokens
7.  MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
8.  FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters(),
                            resetRegionalFilters()
9.  CHARTS REGISTRY — charts{}, destroyChart()
10. POPULATE SELECTS — populateSelects()
11. TAB RENDERS:
    renderOverview(), renderProgression(), renderPBs(),
    renderSplits(), renderSplitCharts(), analyzeSwim(),
    renderResults(), buildQtStatCards(), renderQualifying(), renderRegionalQualifying()
12. TAB SWITCHING — showTab()
13. INIT — async init()
14. ADD RACE — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
15. REMOVE RACE — removeRace(date, event, course, time)
16. DOMContentLoaded → init()

## Tab IDs
tab-overview, tab-progression, tab-pbs, tab-splits, tab-results, tab-qualifying, tab-regional

## Stable DOM wrapper IDs (canvas replaced on debut/empty; wrapper always exists)
splitBarChartWrap / progChartWrap / paceChartWrap
Pattern: querySelector('canvas') check → restore → destroyChart → create or replace

## Debut detection — four contexts, four methods

| Context                  | Method                           | Reason                             |
|--------------------------|----------------------------------|------------------------------------|
| Results Δ Prev column    | r.deltaPrev === null             | First chronological swim per event |
| Overview Recent PBs      | swimCount === 1                  | Only swim ever for event+course    |
| Progression Improv / BvF | !refSwim || swims===1 || ref===pb| No meaningful reference available  |
| Splits analyzeSwim       | prevPB === null (date filtered)  | No earlier swim with splits        |

## Benchmark lookups — by context

| Function                | Filter used        | Returns                                |
|-------------------------|--------------------|----------------------------------------|
| analyzeSwim prevPB      | r.date < swim.date | Fastest earlier dated swim with splits |
| getPreviousSwims()      | r !== pb           | Newest non-PB swim                     |
| getPreviousBestSwims()  | r !== pb           | Fastest non-PB swim (2nd best overall) |
| getFirstSwims()         | DATA order (chron) | Earliest dated swim per event+course   |
| getLatestSwims()        | DATA order (chron) | Latest dated swim per event+course     |

## processData()
Sorts RAW chronologically. Computes timeInSec, isPB (running min), deltaPrev (vs prior swim).
deltaPrev===null marks the chronological first swim — used only in Results tab for debut.

## CSS variables (light theme)
--bg:#f0f4f8  --surface:#ffffff  --surface2:#e8eef5  --surface3:#d6e2ef
--accent:#1a56c4  --gold:#d97706  --green:#059669  --red:#dc2626
--text:#0f172a  --text2:#334155  --text3:#64748b  --border:#cbd5e1

## Chart tokens: CC.grid #cbd5e1 / CC.tick #475569 / CC.legend #334155

## PALETTE: ['#1a56c4','#0891b2','#7c3aed','#059669','#d97706','#dc2626','#db2777','#ea580c','#0d9488','#9333ea']

## Tabs: var(--surface2)/var(--border)/var(--text2) inactive; #4f86d8 active (both desktop + mobile)

## Data sources & localStorage
RACE_DATA_URL → swimDash_RAW → RAW / QT_DATA_URL → swimDash_QT → QT_DATA
SE_QT_DATA_URL → swimDash_SE_QT → SE_QT_DATA

## Add / Remove Race
Add: ＋ FAB → modal (with localStorage note) → RAW.push → localStorage → re-render all
Remove: ✕ in Results → confirm → findIndex → RAW.splice → localStorage → re-render all
Persist: ⚙️ → ⬇️ Download my_swims.json → push to GitHub
