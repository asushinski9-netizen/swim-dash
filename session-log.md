# Session Log

## v13 (latest)
- Mobile tabs: Explicit #9ab4cc background on mobile overrides desktop value; active tab retains #4f86d8 via dedicated mobile rule
- Progression ghost chart: Added `delete charts['progChart']` after debut callout to prevent Chart.js orphan reference causing empty renders on event re-select
- Progression debut callout: Shows competition name instead of venue
- Pacing Profile: Lines now sorted fastest→slowest (sortedForPace) in both renderSplitCharts and analyzeSwim highlight loop
- QT table: PB Date moved to its own column (was inline below PB time, inflating row height)
- Regional Qualifying tab added: New 🏅 Regional QT tab using se_london_qt.json
  - SE London age groups: 11/12, 13, 14, 15, 16, 17, 18+ (different from county 10+11…17+)
  - Long Course default (SE London is primarily LC); no null values in SE QT data
  - Full matching renderRegionalQualifying() function: stat cards, bar+line chart, 8-column table with PB Date column
  - resetRegionalFilters() helper
  - SE_QT_DATA_URL constant; SE_QT_DATA global with swimDash_SE_QT localStorage key
  - Data Manager: third upload field for se_london_qt.json (uploadSEQT)
  - init() fetches all three URLs in parallel; falls back to localStorage individually
  - showTab() dispatches to renderRegionalQualifying() for 'regional'
  - Mobile: regional table scaled same as county (0.62rem); grid4 2-column

## v12
- Tabs: Inactive tabs further darkened to #b8cde0 with border #a0bbd4 for mobile contrast
- Grid: Added min-width:0 to all grid children to prevent chart blowout; chart-wrap gets overflow:hidden
- Progression debut: Single-race selection replaces chart with debut callout (stable progChartWrap wrapper)
- Progression improvement: Debut row shown when refSwim is null (prevSwim/prevBest modes)
- Overview charts: Race Frequency 220→280px, Events Breakdown 220→320px
- Splits debut message: Context-aware — "Select a later race" if others exist, "Swim again" if truly first
- Splits chart: Scatter replaced with grouped horizontal bar chart (indexAxis:y, shorter=faster — intuitive)
- Splits pacing: sortedForPace — datasets ordered fastest→slowest so fastest line leads legend
- Splits lap delta label: "Lap Deltas vs PB" → "Lap Deltas vs Prev PB"
- Results table: Mobile font scaled to 0.65rem with tighter padding
- QT table: Mobile font scaled to 0.62rem with tighter padding
- Data FAB: Matched to 56px (same as ＋ FAB)

## v11
- Tabs: Active tab lightened to #4f86d8; inactive tabs darkened to var(--surface3) + border
- Debut badge: Moved to right side under time in Recent PBs (was left side next to event name)
- Mobile bleed: Removed display:flex from #tab-progression (was causing other panels to bleed)
- Bug (critical): analyzeSwim crashed with null.closest() when canvas replaced by debut message
  → Fixed with stable id="splitBarChartWrap" wrapper; both analyzeSwim and
    renderSplitCharts reference the wrapper div, not the canvas
- renderSplitCharts: Checks splitBarChartWrap for canvas before getting ctx
- Download JSON: Added downloadRaceData() and ⬇️ Download button in Data Manager modal
- Data FAB: Added .fab-data ⚙️ button above ＋ FAB to restore Data Manager access

## v10
- Logo: Replaced hand-drawn SVG dolphin with actual uploaded BPSC PNG (base64 <img>)
- Colours: CC.grid darkened to #cbd5e1 (was #e2e8f0 — invisible on white)
- PALETTE: Updated to deeper tones suitable for light background
- Bug: lapLabels undefined in analyzeSwim → declared as const before paceChart update
- Bug: showError declared at top of saveNewRace (was after its call sites)
- Bug: indexAxis:y removed from scatter chart (unsupported on scatter type)
- Bug: renderSplits() removed from saveNewRace (cleared splits tab if no event selected)
- Bug: Selected pacing line was #ffffff (invisible on light bg) → now #1a56c4
- Bug: Mobile :last-child rule targeting removed data button → removed
- Bug: Progression mobile ordering used flex order on tab panel → replaced with simpler approach
- Bug: Scatter tooltip ctx.parsed.y wrapped in Math.round()
- Bug: CC.legend used for tick colour on pacing chart → CC.tick
- Pacing chart: Uses PALETTE instead of hsl() formula

## v9
- Theme: Full light theme overhaul (--bg:#f0f4f8, --surface:#fff, --accent:#1a56c4 BPSC blue)
- Logo: Replaced swimmer emoji with BPSC PNG logo (base64 embedded) in header
- Data button: Removed from header (modal still accessible via init fallback and later FAB)
- Debut badges: Added .badge.debut CSS; shown in Overview Recent PBs (right side under time),
  Progression Improvement Summary, Progression BvF Δ column, Results Δ Prev column
- Mobile progression: prog-chart-card class added; chart moves below summary on mobile
- Splits chart: Bar chart replaced with horizontal scatter/dot chart
- Splits debut: analyzeSwim suppresses chart for debut races, shows message instead
- Add Race FAB: ＋ fixed floating button opens modal; saves to localStorage + re-renders all tabs

## v8 and earlier — see original session-log
