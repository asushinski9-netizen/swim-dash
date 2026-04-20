## v8 (latest)
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
  
## v7 (latest)
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