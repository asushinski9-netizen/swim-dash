# Session Log

## v15 (latest)
- Regional QT: Default course changed to Short Course (25m); resetRegionalFilters also defaults to SC
- Stat card mobile: Event lists inside QT stat cards (Qualified/Consideration/Outside) reduced to 0.58rem / line-height 1.35 so more text fits without overflow; applied to both County and Regional tabs
- Remove Race: ✕ button added to every row in All Results table; removeRace(date,event,course,time) finds row in RAW by matching those four fields, splices it, persists to localStorage, and re-renders all tabs. Confirm dialog before deletion. Results table colspan updated from 9 to 10.
- Add Race FAB: font-size 1.8rem + line-height:1 ensures ＋ is visually centred
- Time Progression chart: Moved to bottom of Progression tab in HTML source order — works on both desktop and mobile without CSS ordering. Removed now-redundant mobile prog-chart-card margin-top rule.
- Splits empty state: Split Comparison (splitBarChartWrap) and Pacing Profile (paceChartWrap) now show "Select an event to begin analysis." placeholder on initial load. renderSplitCharts restores both canvases from their stable wrapper IDs.
- Pacing Profile Y-axis: Added reverse:true so faster (shorter) split times appear at top, slower at bottom — matches intuitive reading direction

## v14
- Tabs: Colour = btn.sec (var(--surface2) / var(--border) / var(--text2)) on desktop and mobile
- Stat cards: text-align:center globally — centres Overview, County QT, Regional QT on desktop
- BvF mobile table: th and td both 0.62rem !important with matching padding
- QT tables: th and td both 0.60rem !important for County and Regional
- QT stat cards: buildQtStatCards(matched, statsElId) shared helper; Qualified lists events, Consideration shows +Xs to QT, Outside shows +Xs to CT

## v13
- Mobile tabs: hex override + dedicated active rule; Regional QT tab added (se_london_qt.json)
- Progression ghost chart: delete charts['progChart'] after debut callout
- Progression debut: competition name instead of venue
- Pacing Profile: sortedForPace in renderSplitCharts and analyzeSwim
- QT PB Date: own column
- SE_QT_DATA_URL, SE_QT_DATA, swimDash_SE_QT, uploadSEQT, renderRegionalQualifying()

## v12
- Progression debut stable wrapper, single-race callout, debut row in improvement summary
- Overview charts: bigger; Splits: horizontal bar, sorted pacing, "vs Prev PB" label
- Mobile table font scaling; Data FAB 56px

## v11
- null.closest() fixed with splitBarChartWrap; Download JSON; Data FAB

## v10
- Real BPSC PNG; CC.grid fixed; PALETTE updated; multiple bugs fixed

## v9
- Light theme; BPSC blue; logo; debut badges; Add Race FAB
