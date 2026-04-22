## v12 (latest)
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
  → Fixed by adding stable id="splitBarChartWrap" wrapper; both analyzeSwim and
  renderSplitCharts reference wrapper div, not the canvas that may have been replaced
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
- Logo: Replaced swimmer emoji with actual BPSC PNG logo (base64 embedded) in header
- Data button: Removed from header (modal still accessible via init fallback)
- Debut badges: Added .badge.debut CSS; shown in Overview Recent PBs (right side under time),
  Progression Improvement Summary, Progression BvF Δ column, Results Δ Prev column
- Mobile progression: prog-chart-card class added; chart moves below summary on mobile via order CSS
- Splits chart: Bar chart replaced with horizontal scatter/dot chart (inverted X = faster right)
- Splits debut: analyzeSwim suppresses chart for debut races, shows message instead
- Add Race FAB: ＋ fixed floating button opens modal; saves to localStorage + re-renders all tabs

## v8
- Bug: renderQualifying status logic now guards qt.qualify/consider !== null before
  comparison — swimmer faster than consideration-only threshold was showing as "Outside"
- Bug: populateSelects now uses memoised uniqueEvents()/uniqueVenues() — eliminates
  duplicated iteration and a sort inconsistency between the two code paths
- Bug: renderProgression has else fallback for unrecognised compareMode — previously
  refSwims would be undefined and throw on refSwims[key]
- Bug: contextualPB now picks lowest timeInSec across all matched PBs (was taking
  first in alphabetical sort order — LC before SC, giving wrong benchmark)
- Bug: splitChartTitle now updates in analyzeSwim on every row click to show the
  selected swim's actual course — was stuck on the last chronological swim's course
- Bug: splitChartTitle in renderSplits shows "SC/LC" when both courses are selected
- Bug: meets count key uses | separator — prevents name+date string collision
- Bug: SC/LC stat card counts computed once via getSwimCounts() not inline DATA.filter
- Pacing chart: selected swim's line highlighted (white/thick) in analyzeSwim,
  others dimmed; contextualPBRef module var added to track current PB swim
- overviewRendered dirty flag — Overview charts no longer re-render on every revisit;
  reset in invalidateCache() on data reload
- All tabs pre-rendered in init — first switch to any tab is now instant
- hideDataModal() named function added — modal close button no longer uses inline call
- renderQualifying: empty QT_DATA guard shows informative message directing user to
  upload county_qt.json; !matched.length guard now fires before chart creation
- resetProgressionFilters uses .value not .selectedIndex
- renderProgression refSwims/refLabel block properly indented; innerHTML → textContent
- Stale numbered comment removed from renderSplits
- Dead .active-coach CSS rule removed
- updateSortHeaders moved to sortResults only — no longer runs on every filter change
- secToTime unreachable s===null guard removed; sub-60 branch simplified
- populateSelects blank-line formatting fixed
- Dead .sec-hdr and .trend CSS classes removed
- CC constant added for chart colours — 18 hardcoded hex strings replaced with
  CC.tick, CC.legend, CC.grid
- contextualPB lookup uses getPBs() — no longer sorts a full copy of DATA
- Label accessibility: modal file inputs now have matching for attributes on labels
- Rollback: fix-18 chart/table empty ordering reverted — guard correctly placed
  after stat cards, before chart creation, with labels var restored
  
## v7
- Structural: renderQualifying moved before showTab/init (was after DOMContentLoaded)
- Structural: dataModal moved to direct child of <body> (fixes iOS Safari fixed-inside-sticky bug)
- Removed: dead renderedTabs Set, orphaned POPULATE SELECTS section comment
- Bugs: gapQ/gapC null guard for partially-null QT rows, pacing chart labels now
  update when a swim with different lap count is selected, secToTime double-compute eliminated
- Qualifying: now uses memoised getPBs() instead of iterating DATA directly
- Qualifying: borderColor constructed directly as pbBorders array (no string-replace hack)
- resetQualifyingFilters: uses .value not .selectedIndex (safe if dropdown order changes)
- splitChartTitle: updated dynamically in renderSplits with event name and course
- Reset Dashboard modal button now calls resetDashboard() consistently (confirm dialog)
- pbsThisYear uses dynamic new Date().getFullYear() — was hardcoded to 2025
- Progression chart x-axis uses parseLocalDate() for timezone consistency
- uniqueEvents(), uniqueVenues(), uniqueYears() all memoised via cache{}
- reader.onerror handler added in saveDataToBrowser (Promise no longer hangs silently)
- Loading … placeholders in header stats during initial GitHub fetch
- QT-empty warning logged in init catch block
- Results table sort headers show ↑/↓ on active column, ↕ on others

## v6
- Fixed 25 issues from full code review
- Bugs: renderOverview brace, getPreviousSwims sort, trendHTML scope, imp scope,
  splitBarChart dataset[0] not updating, double render in renderSplits
- Perf: memoised all lookup functions with cache{}, added getSwimCounts()
- CSS: moved qualifying mobile styles into @media block, removed duplicate active rule
- UX: secToTime sub-60s formatting, parseLocalDate timezone fix, removed 'securely' text
- Then: reverted incorrect cumulative-splits conversion (splits are already lap times)
- Then: fixed trendHTML/imp scoping bugs introduced during refactor
- Then: fixed split bar chart Current Selection frozen (analyzeSwim now updates both datasets)

## v5
- Added County Qualifying tab with QT_DATA, gender/age/course/stroke filters,
  bar chart + table, stat cards

## v4
- Added missing QT events (200 Breast, 200 Fly, 400 IM, 800 Free, 1500 Free)
- Fixed qualifying chart event ordering (stroke then distance)