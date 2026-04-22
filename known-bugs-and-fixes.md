# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times, not cumulative
RAW splits are individual lap times. Do NOT derive via splits[i] - splits[i-1].

### null.closest() crash in analyzeSwim (v11)
analyzeSwim used getElementById('splitBarChart').closest('.card'). After debut row selected,
canvas replaced by div, so getElementById returned null and .closest() threw TypeError.
Fixed: stable id="splitBarChartWrap" wrapper always exists; both analyzeSwim and
renderSplitCharts reference the wrapper div.

### lapLabels undefined in analyzeSwim (v10)
paceChart.data.labels = lapLabels — variable undeclared in v9. Fixed: const lapLabels declared before use.

### progChart canvas gone after debut callout (v12)
Replacing progChartWrap.innerHTML removed the canvas. Next render got null from getElementById.
Fixed: stable id="progChartWrap" wrapper; canvas restored before destroyChart() each render.

### Progression ghost chart after debut (v13)
After debut callout, charts['progChart'] still held stale reference. On re-select of a different
event the destroyed chart attempted to render. Fixed: delete charts['progChart'] after innerHTML replacement.

### Improvement Summary empty for debut events (v12)
compareMode=prevSwim/prevBest: no refSwim for debut events, silently skipped.
Fixed: debut events render explicit "Debut — no previous benchmark" row.

### indexAxis:y on scatter chart (v10)
Chart.js scatter doesn't support indexAxis. Removed in v10; in v12 scatter replaced with
horizontal bar chart (type:'bar' + indexAxis:'y') which does support it correctly.

### Mobile tab CSS override not applying (v13/v14)
Mobile used a colour that desktop value overrode. Fixed in v14: uses same var(--surface2)
as desktop so cascade is identical on both; !important added for mobile override.

### Stat card event lists — null gap guard (v14)
Outside card shows gap to consideration (gapC). buildQtStatCards uses Math.abs(r.gapC).toFixed(2)
— gapC is guaranteed non-null for Outside status (pbSec != null, consider != null).
Consideration card uses gapQ; also guaranteed non-null for Consideration status.
Guard not needed but be careful if status logic ever changes.

## WATCH-OUT AREAS

### fmtDate / fmtDateShort timezone shift
Always use parseLocalDate() — constructs new Date(y, m-1, d) in local time.
new Date('YYYY-MM-DD') parses as UTC midnight and shows wrong day in UTC- timezones.

### secToTime() for sub-60s values
Returns "38.00" not "0:38.00". Guard: if (!isFinite(s)) return '—'

### QT course strings vs DATA course codes
county_qt / se_london_qt: "Short Course" / "Long Course"
DATA: "S" / "L"
Convert: course === 'Short Course' ? 'S' : 'L'

### SE London QT age groups differ from County QT
County: "10+11", "12", "13"..."16", "17+"
SE London: "11/12", "13", "14"..."17", "18+"
Do not share filter state between the two panels.

### SE London QT has no null values
county_qt has null qualify/consider for some 10+11 events; se_london_qt does not.
buildQtStatCards assumes gapQ/gapC are non-null for Consideration/Outside statuses respectively.

### sortedForPace must be applied in BOTH renderSplitCharts and analyzeSwim
Dataset creation uses sortedForPace; the highlight loop in analyzeSwim also uses
its own sortedForPace sort. If one is changed the other must be updated to match.

### Three localStorage keys
swimDash_RAW, swimDash_QT, swimDash_SE_QT — all three fetched in parallel in init().

### buildQtStatCards is shared by both qualifying tabs
Changes to this function affect both County and Regional displays simultaneously.
Test both tabs after any modification.

### progChartWrap canvas restoration order
canvas check → destroyChart → debut test → replace or create chart.
Canvas must be restored BEFORE destroyChart, which requires the canvas to look up charts['progChart'].
