# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times, not cumulative
RAW splits are individual lap times. Do NOT derive via splits[i] - splits[i-1].

### null.closest() crash in analyzeSwim (v11)
After debut row selected, canvas replaced by div, getElementById returned null.
Fixed: stable id="splitBarChartWrap" wrapper. Both analyzeSwim and renderSplitCharts
reference the wrapper div, not the canvas directly.

### lapLabels undefined in analyzeSwim (v10)
paceChart.data.labels = lapLabels — variable undeclared in v9.
Fixed: const lapLabels declared before use.

### progChart canvas gone after debut callout (v12)
Replacing progChartWrap.innerHTML removed the canvas.
Fixed: stable id="progChartWrap"; canvas restored before destroyChart() each render.

### Progression ghost chart after debut (v13)
charts['progChart'] held stale reference after debut callout innerHTML replacement.
Fixed: delete charts['progChart'] after replacement.

### Improvement Summary empty for debut events (v12)
compareMode=prevSwim/prevBest: no refSwim for debut → silent skip.
Fixed: explicit "Debut — no previous benchmark" row rendered instead.

### indexAxis:y on scatter chart (v10)
Chart.js scatter ignores indexAxis. Removed; replaced in v12 with horizontal bar chart.

### Mobile tab CSS override not applying (v13/v14)
Fixed in v14: uses same var(--surface2) CSS variables as desktop. Both use identical
source values; !important on mobile ensures override where needed.

### paceChart canvas not in stable wrapper (v15)
paceChart canvas was a bare <canvas> with no stable wrapper ID. If renderSplitCharts
was called after the empty-state div replaced it, getElementById('paceChart') returned
null. Fixed: added id="paceChartWrap" stable wrapper; renderSplitCharts checks
paceWrap.querySelector('canvas') and restores canvas if missing, same pattern as
splitBarChartWrap.

### Stat card event lists — null gap guard (v14)
Outside card uses Math.abs(r.gapC).toFixed(2); Consideration uses Math.abs(r.gapQ).toFixed(2).
Both are guaranteed non-null for their respective statuses. Safe as long as status
logic remains: Qualified → gapQ<=0, Consideration → gapC<=0 && gapQ>0, Outside → gapC>0.

## WATCH-OUT AREAS

### fmtDate / fmtDateShort timezone shift
Always use parseLocalDate() — new Date('YYYY-MM-DD') parses as UTC midnight.

### secToTime() sub-60s
Returns "38.00" not "0:38.00". Guard: if (!isFinite(s)) return '—'

### QT course strings vs DATA course codes
county_qt / se_london_qt use "Short Course"/"Long Course"; DATA uses "S"/"L".
Convert: course === 'Short Course' ? 'S' : 'L'

### SE London QT age groups differ from County QT
County: "10+11","12","13"–"16","17+"
SE London: "11/12","13"–"17","18+"
These are separate selects in separate panels — do not share filter state.

### SE London QT has no null values; county_qt does (10+11 events)
buildQtStatCards assumes gapQ/gapC are non-null for Consideration/Outside statuses.

### sortedForPace must match in BOTH renderSplitCharts and analyzeSwim
Dataset creation and highlight loop must use the same sort. If one changes, update both.

### Three localStorage keys
swimDash_RAW, swimDash_QT, swimDash_SE_QT — all fetched in parallel in init().

### buildQtStatCards is shared — changes affect both County and Regional tabs
Test both tabs after modifying this function.

### progChartWrap canvas restoration order
Check querySelector('canvas') → restore if missing → destroyChart → debut check → create or replace.
Canvas must exist BEFORE destroyChart so charts['progChart'] lookup succeeds.

### removeRace matches on date+event+course+time
If two entries share all four fields (duplicate race entered), findIndex removes the first.
This is an edge case but worth noting if duplicates are ever entered via Add Race.

### Pacing Profile Y-axis is reversed (v15)
reverse:true means lower values (faster splits) appear at top. This is intentional.
Do NOT remove this without updating the feature description.

### Time Progression chart is last in HTML source (v15)
Improvement Summary and BvF table render before the chart. This replaces the previous
CSS ordering approach (prog-chart-card order:10, etc.) which was fragile. The mobile
media query no longer needs ordering rules for the progression tab.
