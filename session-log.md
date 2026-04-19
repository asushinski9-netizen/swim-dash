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