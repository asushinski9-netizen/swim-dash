# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v15
~2698 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL
2.  DATA globals — RAW, QT_DATA, SE_QT_DATA, DATA, cache{}
3.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData()
4.  UTILITIES — timeToSec(), secToTime(), parseLocalDate(), fmtDate(), fmtDateShort(),
                getStroke(), getDistance(), processData()
5.  Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears() (all memoised)
6.  PALETTE array + CC chart colour tokens
7.  MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
8.  FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters(),
                            resetRegionalFilters()
9.  CHARTS REGISTRY — charts{}, destroyChart()
10. POPULATE SELECTS — populateSelects()
11. TAB RENDERS:
    - renderOverview()
    - renderProgression()
    - renderPBs()
    - renderSplits() + renderSplitCharts() + analyzeSwim()
    - renderResults()
    - buildQtStatCards(matched, statsElId)   ← shared helper for County + Regional
    - renderQualifying()
    - renderRegionalQualifying()
12. TAB SWITCHING — showTab()
13. INIT — async init() fetches 3 URLs in parallel → localStorage fallback → modal
14. ADD RACE MODAL — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
15. REMOVE RACE — removeRace(date, event, course, time)
16. DOMContentLoaded → init()

## Tab IDs
tab-overview, tab-progression, tab-pbs, tab-splits, tab-results, tab-qualifying, tab-regional

## Stable DOM wrapper IDs (canvas may be replaced; wrapper always exists)
- splitBarChartWrap — wraps splitBarChart canvas in Splits tab
- progChartWrap — wraps progChart canvas in Progression tab
- paceChartWrap — wraps paceChart canvas in Splits tab (added v15)

Pattern for all three:
  1. check wrapper.querySelector('canvas') → restore <canvas id="..."> if missing
  2. destroyChart / destroy module-level chart ref
  3. check debut/empty condition → replace innerHTML with message OR create new Chart

## Progression tab layout (v15)
HTML source order: filter-bar → grid2 (Improvement Summary + BvF) → prog-chart-card
The chart is last in source order — renders at bottom on both desktop and mobile.
No CSS ordering rules needed. The old prog-chart-card mobile margin-top rule was removed.

## Splits tab empty state (v15)
splitBarChartWrap and paceChartWrap both start with a placeholder "Select an event"
div instead of bare canvas elements. renderSplitCharts() restores canvases into both
wrappers before creating Chart instances.

## Pacing Profile Y-axis (v15)
reverse: true on the Y axis — faster (shorter) split times appear at top, slower at bottom.
Matches the intuitive "better performance = higher on chart" reading direction.

## CSS variables (light theme)
--bg: #f0f4f8        --surface: #ffffff
--surface2: #e8eef5  --surface3: #d6e2ef
--accent: #1a56c4    --accent2: #0ea5d4   --accent3: #7c3aed
--gold: #d97706      --green: #059669     --red: #dc2626
--text: #0f172a      --text2: #334155     --text3: #64748b
--border: #cbd5e1

## Chart colour tokens
CC.grid: '#cbd5e1'   CC.tick: '#475569'   CC.legend: '#334155'

## PALETTE (light-theme optimised)
['#1a56c4','#0891b2','#7c3aed','#059669','#d97706','#dc2626','#db2777','#ea580c','#0d9488','#9333ea']

## Tab CSS (v14+)
Desktop + mobile: var(--surface2) bg / var(--border) border / var(--text2) text = matches .btn.sec
Active: #4f86d8 on both desktop and mobile (explicit !important in mobile block)

## buildQtStatCards(matched, statsElId)
Shared by renderQualifying() and renderRegionalQualifying().
- Qualified card: lists events with checkmark + PB time
- Consideration card: lists events with PB time + "+Xs to QT" gap
- Outside card: lists events with PB time + "+Xs to CT" gap
- No PB card: count only

## removeRace(date, event, course, time)
Matches first RAW entry where all four fields match → splice → persist → re-render all tabs.
Confirm dialog shown before deletion. Relies on date+event+course+time being effectively unique.

## Data sources & localStorage keys
RACE_DATA_URL    → swimDash_RAW    → RAW         my_swims.json
QT_DATA_URL      → swimDash_QT     → QT_DATA     county_qt.json
SE_QT_DATA_URL   → swimDash_SE_QT  → SE_QT_DATA  se_london_qt.json

## Age group differences
County QT:    "10+11","12","13"–"16","17+"
SE London QT: "11/12","13"–"17","18+"
Regional QT defaults to Short Course (25m), age group 11/12.
