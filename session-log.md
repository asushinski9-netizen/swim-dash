# Session Log

## v14 (latest)
- Tabs: Colour changed to match Reset Filter buttons (var(--surface2) bg, var(--border) border, var(--text2) text) on both desktop and mobile; active tab stays #4f86d8
- Stat cards: Added text-align:center globally to .stat-card — centres Overview, County QT, and Regional QT metrics on desktop without changing mobile (already flex-centred)
- BvF mobile table: th and td both set to 0.62rem !important with matching padding — previously inconsistent
- County QT / Regional QT event tables: th and td both explicitly set to 0.60rem !important for consistency
- QT stat cards (County + Regional): Replaced static sub-text with event lists:
  - Qualified card: lists events with checkmark and PB time
  - Consideration card: lists events with PB time and "+Xs to QT" gap to qualify time
  - Outside card: lists events with PB time and "+Xs to CT" gap to consideration time
  - Shared buildQtStatCards(matched, statsElId) helper avoids duplication between renderQualifying() and renderRegionalQualifying()

## v13
- Mobile tabs: Explicit hex on mobile overrides desktop; active retains #4f86d8 via dedicated mobile rule
- Progression ghost chart: delete charts['progChart'] after debut callout prevents orphan reference
- Progression debut callout: Shows competition name instead of venue
- Pacing Profile: Lines sorted fastest-slowest via sortedForPace in both renderSplitCharts and analyzeSwim
- QT table: PB Date moved to its own column
- Regional Qualifying tab added using se_london_qt.json (age groups 11/12, 13-17, 18+)
- SE_QT_DATA_URL, SE_QT_DATA global, swimDash_SE_QT localStorage, third upload field in Data Manager
- init() fetches all three data URLs in parallel

## v12
- Tabs: #b8cde0 inactive, #a0bbd4 border
- Grid: min-width:0 on children; chart-wrap overflow:hidden
- Progression debut: stable progChartWrap wrapper; single-race debut callout
- Progression improvement: Debut row for null refSwim in prevSwim/prevBest modes
- Overview charts: Frequency 280px, Breakdown 320px
- Splits debut message: context-aware
- Splits chart: horizontal bar (indexAxis:y) replaces scatter
- Splits pacing: sorted fastest-slowest
- Splits lap delta label: "vs Prev PB"
- Results/QT tables: mobile font scaling
- Data FAB: 56px

## v11
- Tabs: #4f86d8 active, surface3+border inactive
- Debut badge: right side under time
- Mobile panel bleed: removed display:flex from #tab-progression
- Bug: null.closest() fixed with id="splitBarChartWrap"
- Download JSON: downloadRaceData() + button
- Data FAB: .fab-data button

## v10
- Logo: real BPSC PNG base64
- CC.grid: #cbd5e1; PALETTE updated for light bg
- Bugs: lapLabels, showError order, indexAxis, renderSplits in save, white line, last-child, progression mobile order, tooltip float, CC.legend, hsl()

## v9
- Light theme, BPSC blue accent, logo
- Debut badges everywhere
- Add Race FAB + modal
- Splits debut suppression
