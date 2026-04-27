# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times not cumulative (v6)
### null.closest() crash in analyzeSwim (v11) — splitBarChartWrap stable wrapper
### lapLabels undefined in analyzeSwim (v10) — declared before use
### progChart canvas gone after debut callout (v12) — progChartWrap stable wrapper
### Progression ghost chart after debut (v13) — delete charts['progChart']
### paceChart null crash (v16) — if (paceChart) guard
### Backdated race broke Progression lookups (v17) — r !== pb identity filter
### Splits compared vs course PB not previous PB (v18) — reverted to date filter
### Backdated PB showed dash in Improvement Summary (v18) — refSwim===pb guard

### Results tab Debut badge missing for first swim of multi-swim events (v19)
swimCount===1 broke debut for events with multiple swims. Reverted to deltaPrev===null.

### CSS -webkit-appearance without standard appearance (v19)
Fixed: added appearance:none to inline selects in Add Race modal.

### Data Killer — init() overwrote localStorage with GitHub fetch (v20)
Fixed: init() reads localStorage first; RAW never overwritten by GitHub.

### Split parsing corruption with parseFloat (v20)
Fixed: timeToSec(s.trim()); validation uses !isFinite(s) || s <= 0.

### finishDiff always zero for 50m 2-lap races (v20)
Fixed: 2-lap races use legs[1]-legs[0]; longer races use legs.slice(1,-1).

### strokeOrder indexOf=-1 for unrecognised strokes (v20)
Fixed: 'Other' appended to strokeOrder in both qualifying functions.

### "Latest Swim" mode showed Debut when PB is the most recent swim (v20.1)
Root cause: refSwim===pb guard from v18 fired for ALL compare modes. For 'latest' mode,
refSwim===pb simply means the swimmer's PB is also their most recent race — entirely
normal. Fix: guard only fires when compareMode !== 'latest'. Self-reference in 'latest'
mode shows "— PB is latest" row instead.
Applied in both Improvement Summary forEach and BvF table forEach.

### AI Coach "New Best" shown on historical PBs since beaten (v20.1)
statusLabel used swim.timeInSec < prevPB.timeInSec — true for every swim that was a PB
at the time it was swum, including ones now beaten. Fixed: uses swim.isPB (processData
flag) — only the current overall fastest swim gets the green badge.

## WATCH-OUT AREAS

### Debut detection — four contexts (unchanged from v19)

| Context                  | Method                                          | Reason                                |
|--------------------------|-------------------------------------------------|---------------------------------------|
| Results Δ Prev column    | r.deltaPrev === null                            | First chronological swim per event    |
| Overview Recent PBs      | swimCount === 1                                 | Only swim ever for event+course       |
| Progression Improv / BvF | swims===1 \|\| !refSwim \|\| (ref===pb && mode!='latest') | No meaningful reference      |
| Splits analyzeSwim       | prevPB === null (date filtered)                 | No earlier swim with splits           |

### refSwim===pb guard in Progression (v18, refined v20.1)
For 'first'/'prevSwim'/'prevBest' modes: refSwim===pb means backdated entry is both
first and fastest → show Debut.
For 'latest' mode: refSwim===pb means PB is also the most recent swim → show neutral row.
Never show Debut in 'latest' mode when multiple swims exist.

### swim.isPB vs swim.timeInSec < prevPB.timeInSec
swim.isPB (set by processData running minimum) = true only for the current overall
fastest swim. Use this for "is this still the best ever?" checks.
swim.timeInSec < prevPB.timeInSec = true for any swim that was a PB at the time.
Use this only if you need "was this a PB when it was swum?" — e.g. historical PB display.

### analyzeSwim prevPB uses date filter (r.date < swim.date) — intentional
Backdated first swim → no earlier dates → prevPB null → Debut. Correct.
Do NOT change this to "all other swims" — that broke the Splits tab in v17.

### getPreviousSwims / getPreviousBestSwims use r !== pb identity
Both use DATA array references. Do not convert to index or value comparison.

### init() data priority
1. RAW: localStorage → GitHub fallback if empty → modal if both fail
2. QT/SE_QT: localStorage → GitHub fetch if either missing
Never overwrite localStorage RAW from GitHub.

### updateData() does NOT call renderSplits()
Would clear Splits tab if no event selected. Splits tab re-renders on showTab().

### split validation: !isFinite not isNaN
timeToSec('') returns Infinity. isNaN(Infinity)===false → passes validation.
Always use !isFinite(s) || s <= 0.

### col-abbr / col-full classes
Used only in QT table headers. .col-abbr hidden on desktop, shown on mobile.
The mobile @media block swaps them with !important.

### fmtDate/fmtDateShort: always use parseLocalDate()
### QT course strings vs DATA codes: "Short Course"/"Long Course" vs "S"/"L"
### buildQtStatCards shared — changes affect both County and Regional tabs
### progChartWrap restoration order: querySelector → restore → destroyChart → create/replace
### removeRace matches date+event+course+time (first findIndex match)
### Pacing Profile Y-axis: reverse:true intentional
### Time Progression: last in HTML source — no CSS ordering rules needed
