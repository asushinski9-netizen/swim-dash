# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times, not cumulative
RAW splits are individual lap times. Do NOT convert via splits[i] - splits[i-1].
That was incorrectly applied in v6 refactor and caused nonsense values (2.58s laps).

### renderOverview() closing brace
renderOverview() once had its closing } in the wrong place mid-function,
causing chart rendering code to run at parse time.

### getPreviousSwims() — sort must use localeCompare
Sort history descending using b.date.localeCompare(a.date), NOT b.date - a.date.

### trendHTML scoping in renderProgression
Must be declared with `let trendHTML = ''` inside the forEach callback.

### imp scoping in BvF table
`const imp = ...` must be declared inside eventsToShow.forEach(), not outer scope.

### Split Comparison bar chart frozen
analyzeSwim() must update BOTH datasets[0] and datasets[1]. Only updating [1] left [0] frozen.

### splitDetail.innerHTML double-set + analyzeSwim double-called
renderSplits() once set splitDetail.innerHTML twice and called analyzeSwim twice.

### populateSelects() wiping "All Events" option
The progEvent select must include '<option value="">All Events</option>' as first item.

### null.closest() crash in analyzeSwim (v11)
analyzeSwim used getElementById('splitBarChart').closest('.card'). After a debut row was
selected, the canvas was replaced with a div, so getElementById returned null and .closest()
threw TypeError. Fixed with stable id="splitBarChartWrap" wrapper div that always exists.
Both analyzeSwim and renderSplitCharts reference the wrapper, not the canvas.

### lapLabels undefined in analyzeSwim (v10)
paceChart.data.labels = lapLabels referenced an undeclared variable (existed in v8, dropped
in v9 scatter refactor). Fixed: const lapLabels = swim.splits.map(...) declared before use.

### progChart canvas gone after debut callout (v12)
Replacing progChartWrap.innerHTML with a debut message removed the canvas from DOM. Next
renderProgression() call got null from getElementById('progChart'). Fixed with stable
id="progChartWrap" wrapper — canvas is restored inside it before destroyChart() each render.

### Improvement Summary empty for debut events (v12)
With compareMode=prevSwim or prevBest, no ref swim exists for debut events. The
`if (!refSwim) return` silently skipped them. Fixed: debut events now render an explicit
"Debut — no previous benchmark" row.

### indexAxis:y on scatter chart (v10)
Chart.js scatter charts don't support indexAxis. Silently ignored. Removed in v10.
In v12, scatter was replaced with a grouped horizontal bar chart (type:'bar' + indexAxis:'y').

### Progression ghost chart after debut (v13)
After showng debut callout in progChartWrap, charts['progChart'] still held a stale reference.
On next render, destroyChart found no canvas but the reference remained. Fixed by explicitly
calling `delete charts['progChart']` after replacing the innerHTML with the debut message.

### Mobile tab override not applying (v13)
Mobile CSS used var(--surface2) for inactive tab background, which the desktop value #b8cde0
overrode. Fixed by using explicit hex #9ab4cc in the @media block with !important.

## WATCH-OUT AREAS

### fmtDate / fmtDateShort timezone shift
new Date('2025-11-15') parses as UTC midnight → toLocaleDateString() in timezones
behind UTC shows the previous day. Always use parseLocalDate() which constructs
new Date(y, m-1, d) in local time.

### secToTime() for sub-60s values
Returns "38.00" not "0:38.00" for times under 60s — correct swimming notation.
The null/Infinity guard: if (!isFinite(s)) return '—'

### QT_DATA course strings ≠ DATA course codes
County QT uses "Short Course"/"Long Course"; DATA uses "S"/"L".
SE London QT uses the same "Short Course"/"Long Course" convention.
Always convert: course === 'Short Course' ? 'S' : 'L'

### SE London QT age groups differ from County QT
County: "10+11", "12", "13"..."16", "17+"
SE London: "11/12", "13", "14"..."17", "18+"
These are different select options in different panels — do not share filter state.

### SE London QT has no null values
Unlike county_qt.json which has null qualify/consider for some 10+11 events,
se_london_qt.json has no nulls. The null-filter guard in renderQualifying is not
needed in renderRegionalQualifying but was kept safe with direct value access.

### Null qualify/consider in QT_DATA (county only)
10+11 age group has null for 400 IM, 800 Free, 1500 Free.
Filter with !(r.qualify === null && r.consider === null).
Guard .toFixed() calls: r.qualify !== null ? parseFloat(r.qualify.toFixed(2)) : null

### renderSplitCharts vs analyzeSwim chart ownership
splitBarChart and paceChart are module-level vars (not in charts{} registry).
renderSplitCharts() creates them fresh. analyzeSwim() mutates datasets and calls .update().
sortedForPace must be applied in BOTH places — dataset creation and the highlight loop.

### Memoisation cache invalidation
invalidateCache() must be called whenever DATA is reprocessed. Also resets overviewRendered.

### progChartWrap canvas restoration
Before every destroyChart('progChart') call, the code checks progWrap.querySelector('canvas')
and restores the canvas if the debut message replaced it. This must happen BEFORE destroyChart,
not after, because destroyChart looks up charts['progChart'] which requires the canvas to exist.

### Three localStorage keys for data
swimDash_RAW — race entries (my_swims.json)
swimDash_QT  — county qualifying times (county_qt.json)
swimDash_SE_QT — SE London regional times (se_london_qt.json)
All three are fetched in parallel in init() and stored to localStorage on success.
