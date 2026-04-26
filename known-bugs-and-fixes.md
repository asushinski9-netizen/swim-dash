# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times, not cumulative (v6)
Do NOT derive via splits[i] - splits[i-1].

### null.closest() crash in analyzeSwim (v11)
Fixed: stable id="splitBarChartWrap". Canvas replaced on debut; wrapper always exists.

### lapLabels undefined in analyzeSwim (v10)
Fixed: const lapLabels declared before paceChart.data.labels reference.

### progChart canvas gone after debut callout (v12)
Fixed: stable id="progChartWrap"; canvas restored before destroyChart() each render.

### Progression ghost chart after debut (v13)
Fixed: delete charts['progChart'] after debut callout innerHTML replacement.

### indexAxis:y on scatter chart (v10)
Fixed in v12: replaced with horizontal bar chart (type:'bar' + indexAxis:'y').

### paceChart null crash in analyzeSwim (v16)
Fixed: entire pacing update block wrapped in if (paceChart) { ... }.

### Backdated race broke Progression/Personal Bests benchmark lookups (v17)
getPreviousSwims / getPreviousBestSwims used r.date < pb.date — returned nothing
when backdated entry became PB. Fixed: r !== pb identity filter.
getPreviousSwims    → newest non-PB swim (sorted newest-first, [0])
getPreviousBestSwims → fastest non-PB swim (min timeInSec reduce)

### Splits compared every swim against course PB not previous PB (v18)
v17 changed analyzeSwim prevPB to "fastest of all other swims" — always the overall PB.
REVERTED: analyzeSwim uses r.date < swim.date. Semantically correct: "best time before
this swim was swum". Backdated first swim correctly shows as Debut.

### Backdated PB showed dash in Improvement Summary; compared to itself in BvF (v18)
When backdated entry is both first and fastest, getFirstSwims() === getPBs() for that key.
imp = 0 → dash. Fixed: debut guard extended to !refSwim || swims===1 || refSwim===pb.
Applied in Improvement Summary forEach and BvF table forEach.

### Results tab Debut badge missing for first swim of multi-swim events (v19)
v17 changed Results debut detection to swimCount===1 — only showed Debut for events
swum exactly once total, breaking debut display for all events with multiple swims.
Fixed: reverted to r.deltaPrev===null, which processData() sets for the chronologically
first swim of each event+course. This is the correct per-row debut indicator.

### CSS -webkit-appearance without standard appearance (v19)
Add Race modal selects had -webkit-appearance:none in inline styles without appearance:none.
VS Code CSS linter flagged line 858. Fixed: added appearance:none to all inline selects.

## WATCH-OUT AREAS — DEBUT DETECTION: THREE CONTEXTS, THREE APPROACHES

| Context                    | Method                          | Rationale                                |
|----------------------------|---------------------------------|------------------------------------------|
| Results Δ Prev column      | r.deltaPrev === null            | First chronological swim of event        |
| Overview Recent PBs badge  | swimCount === 1                 | Only swim ever for that event+course     |
| Progression Improv/BvF     | !refSwim || swims===1 || ref===pb | No meaningful reference to compare to  |
| Splits analyzeSwim         | prevPB === null (date filtered) | No earlier swim with splits existed      |

Do NOT unify these. Each answers a different question about debut status.

### analyzeSwim prevPB uses date filter — intentional (v18)
prevPB = fastest swim with r.date < swim.date AND splits.length > 0.
Backdated first swim → no earlier dates → prevPB null → Debut. Correct.

### getPreviousSwims / getPreviousBestSwims use r !== pb identity
Works because getPBs() returns references into same DATA array. Do not convert to index.

### refSwim === pb guard in Progression (v18)
Catches backdated entry that is both first and fastest. Without it, imp=0 → dash not Debut.
Essential for correct display of backdated PBs in Improvement Summary and BvF table.

### fmtDate/fmtDateShort: always use parseLocalDate()
new Date('YYYY-MM-DD') parses UTC midnight → wrong day in non-UTC timezones.

### secToTime() sub-60s returns "38.00" not "0:38.00". Guard: if (!isFinite(s)) return '—'

### QT course strings vs DATA codes: "Short Course"/"Long Course" vs "S"/"L"

### SE London QT age groups differ from County QT (see data-schema.md)

### buildQtStatCards shared — changes affect both County and Regional tabs

### progChartWrap restoration order: querySelector → restore → destroyChart → create/replace

### removeRace matches date+event+course+time (first findIndex). Duplicates: first match removed.

### Pacing Profile Y-axis: reverse:true intentional (faster splits at top)

### Time Progression: last in HTML source — no CSS ordering rules needed
