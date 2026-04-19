# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times, not cumulative
RAW splits are individual lap times. Do NOT convert via splits[i] - splits[i-1].
That was incorrectly applied in v6 refactor and caused nonsense values (2.58s laps).

### renderQualifying defined after DOMContentLoaded
renderQualifying() must be declared before showTab() and init(). It was briefly
placed after document.addEventListener('DOMContentLoaded', init) which works due
to hoisting but is structurally wrong and breaks under strict mode / const refactor.

### dataModal inside sticky header
position:fixed inside position:sticky causes rendering bugs on iOS Safari.
dataModal must be a direct child of <body>, not nested inside .header-stats.

### trendHTML scoping in renderProgression
Must be declared with `let trendHTML = ''` inside the forEach callback.
Without it, trendHTML is implicit global and bleeds across loop iterations.

### imp scoping in BvF table
`const imp = refSwim ? refSwim.timeInSec - pb.timeInSec : 0` must be declared
inside the eventsToShow.forEach(), not referenced from the outer forEach scope.

### Split Comparison bar chart frozen
analyzeSwim() must update BOTH datasets[0] (Current Selection) AND datasets[1]
(benchmark). Only updating [1] leaves [0] frozen on the latest swim forever.

### splitDetail.innerHTML double-set + analyzeSwim double-called
renderSplits() once set splitDetail.innerHTML twice and called analyzeSwim twice.
The second innerHTML set wiped the analyzeSwim output. Keep exactly one of each.

### populateSelects() wiping "All Events" option
The progEvent select must include '<option value="">All Events</option>' as
first item — populateSelects() overwrites static HTML options.

### resetPBFilters() commented out
resetPBFilters() was once commented out while the Reset button still called it,
causing ReferenceError on click. Never comment out functions wired to HTML buttons.

### getPreviousSwims() — sort must use localeCompare
Sort history descending using b.date.localeCompare(a.date), NOT b.date - a.date
(treating date strings as numbers gives NaN comparisons).

### renderOverview() closing brace
renderOverview() once had its closing } in the wrong place mid-function,
causing chart rendering code to run at parse time.

### gapQ/gapC NaN when qualify/consider is null
`pbSec - qt.qualify` evaluates to NaN when qt.qualify is null (even if pbSec is
not null). The filter at line ~1826 only excludes rows where BOTH are null.
Guard: `pbSec !== null && qt.qualify !== null ? pbSec - qt.qualify : null`

### borderColor string-replace hack in qualifying chart
Was: `pbColors.map(c => c.replace('0.85','1').replace('0.4','0.8'))`
This is fragile — if any opacity value contains those substrings elsewhere it
corrupts the colour. Now uses a directly constructed pbBorders array instead.

## WATCH-OUT AREAS

### fmtDate / fmtDateShort timezone shift
new Date('2025-11-15') parses as UTC midnight → toLocaleDateString() in timezones
behind UTC shows the previous day. Always use parseLocalDate() which constructs
new Date(y, m-1, d) in local time. Also applies to progression chart x-axis dates.

### secToTime() for sub-60s values
Returns "38.00" not "0:38.00" for times under 60s — correct swimming notation.
The null/Infinity guard: if (!isFinite(s) || s === null) return '—'
Uses pre-computed `sec` variable for the sub-60 branch — do not recompute s - m*60.

### QT_DATA course strings ≠ DATA course codes
QT uses "Short Course"/"Long Course"; DATA uses "S"/"L". Always convert before matching.
Conversion: `course === 'Short Course' ? 'S' : 'L'`

### Null qualify/consider in QT_DATA
10+11 age group has null for 400 IM, 800 Free, 1500 Free.
Filter with !(r.qualify === null && r.consider === null).
Guard .toFixed(): r.qualify !== null ? parseFloat(r.qualify.toFixed(2)) : null
Guard gapQ/gapC: pbSec !== null && qt.qualify !== null ? pbSec - qt.qualify : null

### renderSplitCharts vs analyzeSwim chart ownership
splitBarChart and paceChart are module-level vars (not in charts{} registry).
renderSplitCharts() creates them fresh. analyzeSwim() mutates datasets[0] and [1]
and calls .update() — it does NOT recreate the chart.
analyzeSwim also updates paceChart.data.labels if the lap count changed.

### Memoisation cache invalidation
invalidateCache() must be called whenever DATA is reprocessed.
Cache keys: pbs, firstSwims, latestSwims, previousSwims, previousBestSwims,
swimCounts, uniqueEvents, uniqueVenues, uniqueYears.

### resetQualifyingFilters must use .value not .selectedIndex
.selectedIndex = 1 silently resets to the wrong option if dropdown order changes.
Use .value = '12' (or the correct string) for all qualifying filter resets.

### processData mutates spread-cloned objects
processData writes isPB and deltaPrev onto the objects it creates via spread.
This is safe as long as processData is only called once per data load (which
invalidateCache() + DATA = processData(RAW) enforces). Don't call it twice.

### pbsThisYear threshold
Uses dynamic new Date().getFullYear() — do not hardcode a year here.

### reader.onerror in saveDataToBrowser
FileReader Promises must have both onload AND onerror handlers.
Without onerror, a failed read leaves the Promise permanently pending.