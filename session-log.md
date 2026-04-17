# Session Log

## v6 (latest)
- Fixed 25 issues from full code review
- Bugs: renderOverview brace, getPreviousSwims sort, trendHTML scope, imp scope,
  splitBarChart dataset[0] not updating, double render in renderSplits
- Perf: memoised all lookup functions with cache{}, added getSwimCounts()
- CSS: moved qualifying mobile styles into @media block, removed duplicate active rule
- UX: secToTime sub-60s formatting, parseLocalDate timezone fix, removed 'securely' text
- Then: reverted incorrect cumulative-splits conversion (splits are already lap times)

## v5
- Added County Qualifying tab with QT_DATA, gender/age/course/stroke filters,
  bar chart + table, stat cards

## v4  
- Added missing QT events (200 Breast, 200 Fly, 400 IM, 800 Free, 1500 Free)
- Fixed qualifying chart event ordering (stroke then distance)