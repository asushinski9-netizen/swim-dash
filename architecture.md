# Code Architecture

## File
Single HTML file: swimming_dash_vN.html
Current version: v13x
~2676 lines total

## Script section layout (in order)
1. CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL constants
2. DATA globals — RAW, QT_DATA, SE_QT_DATA, DATA, cache {}
3. Data Manager Modal — showDataModal(), hideDataModal(), saveDataToBrowser(),
                        resetDashboard(), downloadRaceData()
4. UTILITIES — timeToSec(), secToTime(), parseLocalDate(), fmtDate(),
               fmtDateShort(), getStroke(), getDistance(), processData()
5. Derived data helpers — uniqueEvents(), uniqueVenues(), uniqueYears()
6. PALETTE array + CC chart colour tokens
7. MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                      getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
   → All use cache{} object, invalidated by invalidateCache() on data reload
8. FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters(),
                           resetRegionalFilters()
9. CHARTS REGISTRY — charts{}, destroyChart()
10. POPULATE SELECTS — populateSelects()
11. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits() + renderSplitCharts() + analyzeSwim(),
                  renderResults(), renderQualifying(), renderRegionalQualifying()
12. TAB SWITCHING — showTab()
13. INIT — async init() — fetches all 3 URLs in parallel → localStorage fallback → modal
14. ADD RACE MODAL — showAddRaceModal(), hideAddRaceModal(), saveNewRace()
15. DOMContentLoaded → init()

## Key global state
- DATA: processed swim array (sorted chronologically)
- QT_DATA: county qualifying times array
- SE_QT_DATA: SE London regional qualifying times array
- currentEventFiltered[]: swims shown in Splits tab (used by analyzeSwim)
- contextualPBRef: reference to the PB swim object for the current splits selection
- splitBarChart, paceChart: Chart instances (module-level, not in charts{} registry)
- sortCol, sortDir: Results tab sort state
- cache{}: memoisation store, cleared by invalidateCache()
- overviewRendered: dirty flag, reset by invalidateCache() to force re-render

## Tab IDs
tab-overview, tab-progression, tab-pbs, tab-splits, tab-results, tab-qualifying, tab-regional

## Stable DOM wrapper IDs (canvas may be replaced; wrapper always exists)
- splitBarChartWrap — wraps #splitBarChart canvas in Splits tab
- progChartWrap — wraps #progChart canvas in Progression tab
Both follow the same pattern: check querySelector('canvas') before destroyChart(),
replace innerHTML for debut message, restore canvas when switching back.

## CSS variables (light theme — v9+)
--bg: #f0f4f8        --surface: #ffffff
--surface2: #e8eef5  --surface3: #d6e2ef
--accent: #1a56c4    --accent2: #0ea5d4   --accent3: #7c3aed
--gold: #d97706      --green: #059669     --red: #dc2626
--text: #0f172a      --text2: #334155     --text3: #64748b
--border: #cbd5e1

## Chart colour tokens (CC object)
CC.grid:   '#cbd5e1'   — grid lines (matches --border)
CC.tick:   '#475569'   — axis tick labels
CC.legend: '#334155'   — legend labels (matches --text2)

## PALETTE (light-theme optimised)
['#1a56c4','#0891b2','#7c3aed','#059669','#d97706','#dc2626','#db2777','#ea580c','#0d9488','#9333ea']

## Tab CSS
Desktop inactive: background #b8cde0, border #a0bbd4
Desktop active:   background #4f86d8, color white
Mobile inactive:  background #9ab4cc !important, border #7a9ab8 !important, color #1e3a52 !important
Mobile active:    background #4f86d8 !important (explicit mobile override required)

## Mobile breakpoint
@media (max-width: 500px) — all mobile overrides in one block near end of <style>

## Data sources & localStorage keys
| URL constant       | localStorage key  | Variable    | Description              |
|--------------------|-------------------|-------------|--------------------------|
| RACE_DATA_URL      | swimDash_RAW      | RAW         | my_swims.json            |
| QT_DATA_URL        | swimDash_QT       | QT_DATA     | county_qt.json           |
| SE_QT_DATA_URL     | swimDash_SE_QT    | SE_QT_DATA  | se_london_qt.json        |

All three fetched in parallel via Promise.all in init(). Each falls back to localStorage
individually. RAW.length === 0 triggers the data upload modal.

## Add Race workflow
New races from the ＋ FAB are appended to RAW and written to swimDash_RAW in localStorage.
To make permanent: ⚙️ → ⬇️ Download my_swims.json → push to GitHub.

## Age group differences between QT sources
County (county_qt.json):     "10+11", "12", "13", "14", "15", "16", "17+"
SE London (se_london_qt.json): "11/12", "13", "14", "15", "16", "17", "18+"
County has null values for some 10+11 events. SE London has no nulls.
