# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v14
~2660 lines total

## Script section layout (in order)
1. CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL
2. DATA globals — RAW, QT_DATA, SE_QT_DATA, DATA, cache{}
3. DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                         resetDashboard(), downloadRaceData()
4. UTILITIES — timeToSec(), secToTime(), parseLocalDate(), fmtDate(), fmtDateShort(),
               getStroke(), getDistance(), processData()
5. Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears() (memoised)
6. PALETTE array + CC chart colour tokens
7. MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                      getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
8. FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters(), resetRegionalFilters()
9. CHARTS REGISTRY — charts{}, destroyChart()
10. POPULATE SELECTS — populateSelects()
11. TAB RENDERS:
    - renderOverview()
    - renderProgression()
    - renderPBs()
    - renderSplits() + renderSplitCharts() + analyzeSwim()
    - renderResults()
    - buildQtStatCards(matched, statsElId)  ← shared helper for fixes 6/7/8
    - renderQualifying()
    - renderRegionalQualifying()
12. TAB SWITCHING — showTab()
13. INIT — async init() fetches 3 URLs in parallel → localStorage fallback → modal
14. ADD RACE MODAL — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
15. DOMContentLoaded → init()

## Tab IDs
tab-overview, tab-progression, tab-pbs, tab-splits, tab-results, tab-qualifying, tab-regional

## Stable DOM wrapper IDs
- splitBarChartWrap — wraps splitBarChart canvas; replaced with debut message on debut row
- progChartWrap — wraps progChart canvas; replaced with debut message on single-race selection
Pattern: check querySelector('canvas') → restore if missing → destroyChart → create or message

## CSS variables (light theme)
--bg: #f0f4f8        --surface: #ffffff
--surface2: #e8eef5  --surface3: #d6e2ef
--accent: #1a56c4    --accent2: #0ea5d4   --accent3: #7c3aed
--gold: #d97706      --green: #059669     --red: #dc2626
--text: #0f172a      --text2: #334155     --text3: #64748b
--border: #cbd5e1

## Chart colour tokens
CC.grid: '#cbd5e1'   CC.tick: '#475569'   CC.legend: '#334155'

## PALETTE (light-theme)
['#1a56c4','#0891b2','#7c3aed','#059669','#d97706','#dc2626','#db2777','#ea580c','#0d9488','#9333ea']

## Tab CSS (v14)
Both desktop and mobile use var(--surface2) background / var(--border) border / var(--text2) text
to match the Reset Filter button (btn.sec) style.
Active tab: #4f86d8 on both desktop and mobile.

## Stat cards (v14)
.stat-card has text-align:center globally — applies to Overview, County QT, Regional QT.
Mobile override no longer needs flex centering (inherits from base style).

## buildQtStatCards(matched, statsElId)
Shared function called by both renderQualifying() and renderRegionalQualifying().
Renders 4 stat cards with event lists:
- Qualified: event + PB time
- Consideration: event + PB time + gap to qualify time (+Xs to QT)
- Outside: event + PB time + gap to consideration time (+Xs to CT)
- No PB: count only

## Data sources & localStorage keys
RACE_DATA_URL    → swimDash_RAW    → RAW         (my_swims.json)
QT_DATA_URL      → swimDash_QT     → QT_DATA     (county_qt.json)
SE_QT_DATA_URL   → swimDash_SE_QT  → SE_QT_DATA  (se_london_qt.json)

## Age group differences
County QT:    "10+11", "12", "13"–"16", "17+"
SE London QT: "11/12", "13"–"17", "18+"
