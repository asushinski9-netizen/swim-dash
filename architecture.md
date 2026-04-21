# Code Architecture

## File
Single HTML file: swimming_dash_vN.html  
Current version: v8 
~2000 lines total

## Script section layout (in order)
1. CONFIGURATION — RACE_DATA_URL, QT_DATA_URL constants
2. DATA globals — RAW, QT_DATA, DATA, cache {}
3. Data Manager Modal — showDataModal(), saveDataToBrowser(), resetDashboard()
4. UTILITIES — timeToSec(), secToTime(), parseLocalDate(), fmtDate(), 
               fmtDateShort(), getStroke(), getDistance(), processData()
5. Derived data helpers — uniqueEvents(), uniqueVenues(), uniqueYears()
6. COLOURS — PALETTE array
7. MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(), 
                      getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
   → All use cache{} object, invalidated by invalidateCache() on data reload
8. FILTER RESET HELPERS — resetProgressionFilters(), resetQualifyingFilters()
9. CHARTS REGISTRY — charts{}, destroyChart()
10. POPULATE SELECTS — populateSelects()
11. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits() + renderSplitCharts() + analyzeSwim(),
                  renderResults(), renderQualifying()
12. TAB SWITCHING — showTab()
13. INIT — async init() with GitHub fetch → localStorage fallback → modal fallback
14. DOMContentLoaded → init()

## Key global state
- DATA: processed swim array (sorted chronologically)
- currentEventFiltered[]: swims shown in Splits tab (used by analyzeSwim)
- splitBarChart, paceChart: Chart instances (module-level, not in charts{} registry)
- sortCol, sortDir: Results tab sort state (reflected in header via updateSortHeaders())
- cache{}: memoisation store, cleared by invalidateCache()
- overviewRendered: boolean — dirty flag, prevents Overview re-rendering on revisit
- contextualPBRef: swim object — the current PB used as benchmark in the pacing chart
- CC: object — chart colour tokens {grid, tick, legend} matching CSS variables

## Tab IDs
tab-overview, tab-progression, tab-pbs, tab-splits, tab-results, tab-qualifying

## CSS variables (dark theme)
--bg, --surface, --surface2, --surface3
--accent (#3b82f6), --accent2 (#06b6d4), --accent3 (#8b5cf6)
--gold (#f59e0b), --green (#10b981), --red (#ef4444)
--text, --text2, --text3, --border

## Modal
dataModal is a direct child of <body> — NOT nested in .header (fixed-inside-sticky iOS bug)

## Mobile breakpoint
@media (max-width: 500px) — all mobile overrides in one block near end of <style>