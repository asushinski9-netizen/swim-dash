# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times, not cumulative
RAW splits are individual lap times. Do NOT convert via splits[i] - splits[i-1].
That was incorrectly applied in v6 refactor and caused nonsense values (2.58s laps).

### renderOverview() closing brace
renderOverview() once had its closing } in the wrong place mid-function,
causing chart rendering code to run at parse time. Keep all chart code inside the function.

### getPreviousSwims() — sort must use localeCompare
Sort history descending using b.date.localeCompare(a.date), NOT b.date - a.date
(treating date strings as numbers gives NaN comparisons).

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

## WATCH-OUT AREAS

### fmtDate / fmtDateShort timezone shift
new Date('2025-11-15') parses as UTC midnight → toLocaleDateString() in timezones
behind UTC shows the previous day. Always use parseLocalDate() which constructs
new Date(y, m-1, d) in local time.

### secToTime() for sub-60s values
Returns "38.00" not "0:38.00" for times under 60s — correct swimming notation.
The null/Infinity guard: if (!isFinite(s) || s === null) return '—'

### QT_DATA course strings ≠ DATA course codes
QT uses "Short Course"/"Long Course"; DATA uses "S"/"L". Always convert before matching.

### Null qualify/consider in QT_DATA
10+11 age group has null for 400 IM, 800 Free, 1500 Free.
Filter these out with !(r.qualify === null && r.consider === null).
Guard .toFixed() calls: r.qualify !== null ? parseFloat(r.qualify.toFixed(2)) : null

### renderSplitCharts vs analyzeSwim chart ownership
splitBarChart and paceChart are module-level vars (not in charts{} registry).
renderSplitCharts() creates them fresh. analyzeSwim() mutates datasets[0] and [1]
and calls .update() — it does NOT recreate the chart.

### Memoisation cache invalidation
invalidateCache() must be called whenever DATA is reprocessed (i.e. after any
new data load in init()). Without it, getPBs() etc. return stale results.