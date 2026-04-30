# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### splits[] are lap times not cumulative (v6)
### null.closest() crash in analyzeSwim (v11) — splitBarChartWrap stable wrapper
### lapLabels undefined in analyzeSwim (v10) — declared before use
### progChart canvas gone after debut callout (v12) — progChartWrap stable wrapper
### Progression ghost chart after debut (v13) — delete charts['progChart']
### paceChart null crash (v16) — if (paceChart) guard
### Backdated race broke Progression lookups (v17) — r !== pb identity filter
### Splits compared vs course PB not previous PB (v18) — date filter in analyzeSwim
### Backdated PB showed dash in Improvement Summary (v18) — refSwim===pb guard
### Results Debut badge missing for first swim of multi-swim events (v19) — deltaPrev===null
### CSS -webkit-appearance without standard appearance (v19)
### Data Killer — init() overwrote localStorage (v20)
### Split parsing with parseFloat (v20) — timeToSec + !isFinite guard
### finishDiff zero for 50m races (v20) — legs[1]-legs[0]
### strokeOrder indexOf=-1 (v20) — 'Other' appended
### Dead "slower than PB" branch (v20) — removed
### "Latest Swim" mode showed Debut when PB is most recent swim (v20.1)
### AI Coach tagged historical PBs as Current PB (v20.2) — getPBs().includes(swim)
### resetResultsFilters sorted oldest-first after reset (v20.2) — fixed to sortDir=-1

### Overview "PBs this year" and "Recent PBs" included historical isPB swims (v21)
isPB is set by processData() for every swim that was fastest at time of swimming, including
ones since beaten. The Overview stat card and Recent PBs list both used DATA.filter(r=>r.isPB)
which returned historical beaten PBs. Fixed: both now use getPBs() which returns only the
current fastest swim per event+course.
"PBs this year": pbs.filter(r => r.date >= currentYearStart).length
"Recent PBs": getPBs().sort by date descending, take first 6.

### processData sort unstable for same-date same-event SC/LC (v21)
Two swims on the same date for the same event but different courses (S vs L) sorted
indeterminately. deltaPrev and isPB depend on order. Fixed: added course.localeCompare
as tiebreaker → stable: same date + same event + S always before L.

### Doughnut chart border was dark navy (v21)
borderColor '#111827' left over from dark theme. On light white background this showed dark
divider lines between segments. Fixed to '#ffffff'.

### User-supplied strings unescaped in innerHTML (v21)
competition and venue fields from RAW were rendered directly into innerHTML without escaping,
creating an XSS vector. Fixed: escapeHtml() utility applied to all user-supplied string
rendering: AI Coach, Results table, Overview Recent PBs, Splits Split Detail.
### Missing escapeHtml in renderPBs (v21.1)
The v21 escapeHtml() sweep covered AI Coach, Results table, Overview Recent PBs, and
Splits Split Detail — but missed pb.venue in the renderPBs() pb-info line.
Fixed: escapeHtml(pb.venue) applied.


## WATCH-OUT AREAS

### isPB vs getPBs().includes() — use the right one for the right purpose
| Check                    | True when                              | Use for                           |
|--------------------------|----------------------------------------|-----------------------------------|
| r.isPB                   | Swim was fastest at time of swimming   | Historical "was this a PB?" check |
| getPBs().includes(swim)  | Swim is the current fastest ever       | "Is this the PB right now?"       |

Specifically: Overview recent PBs, stat counts, and AI Coach badge all need getPBs().
Only use r.isPB where historical tracking is semantically correct (e.g. chart point colouring
in Progression — showing which races were PBs when swum is correct there).

### Debut detection — four contexts
| Context                  | Method                                                    | Reason                          |
|--------------------------|-----------------------------------------------------------|---------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                      | First chronological swim        |
| Overview Recent PBs      | swimCount === 1                                           | Only swim ever for event+course |
| Progression Improv / BvF | swims===1 \|\| !refSwim \|\| (ref===pb && mode!='latest') | No meaningful reference         |
| Splits analyzeSwim       | prevPB===null (date filtered)                             | No earlier swim with splits     |

### refSwim===pb guard is mode-aware (v20.1)
latest mode: refSwim===pb → "PB is latest" (normal case).
first/prevSwim/prevBest modes: refSwim===pb → Debut (backdated first+fastest).

### processData sort order (v21)
Stable: date ASC → event name ASC → course ASC (S before L).
deltaPrev and isPB calculations depend on this order being deterministic.

### escapeHtml must be applied to ALL user-supplied strings in innerHTML (v21)
competition, venue from RAW; any future fields added to RAW that render into HTML.
Fields set by dropdown (event, course) are safe — constrained to known values.

### analyzeSwim prevPB filters by swim.course
Ensures SC and LC PBs are always separate benchmarks regardless of the tab's course filter.

### init() data priority
1. RAW: localStorage → GitHub fallback → modal
2. QT/SE_QT: localStorage → GitHub fetch if either missing

### updateData() does NOT call renderSplits()
### split validation: !isFinite not isNaN
### fmtDate/fmtDateShort: always use parseLocalDate()
### QT course strings vs DATA codes: "Short Course"/"Long Course" vs "S"/"L"
### buildQtStatCards shared — changes affect both County and Regional tabs
### progChartWrap restoration: querySelector → restore → destroyChart → create/replace
### removeRace: first findIndex match on date+event+course+time
### Pacing Profile Y-axis: reverse:true intentional
### Time Progression: last in HTML source — no CSS ordering rules needed

### Missing escapeHtml in renderPBs (v22)
pb.venue was rendered directly into innerHTML in Personal Bests tab cards.
Fixed: escapeHtml(pb.venue). All user-supplied strings now escaped everywhere.

### uniqueEventsWithSplits sort non-deterministic for same-distance events (v22)
Sort used only getDistance() with no tiebreaker. uniqueEvents() had localeCompare(b) as
tiebreaker but uniqueEventsWithSplits() did not. Fixed: added localeCompare(b) tiebreaker
for consistency and deterministic ordering.

### Progress bar scale too coarse (v22)
impPct * 5 cap 100% → any improvement ≥ 20% filled the bar. 20% is an unrealistic
threshold for competitive swimming. Updated to impPct * 12.5 so 8% fills the bar.
Better reflects the practical competitive improvement range and gives visual granularity.

## WATCH-OUT AREAS (additions for v22)

### escapeHtml must be applied to all user-supplied strings in innerHTML
Fully applied as of v22: pb.venue (renderPBs), swim.competition/swim.venue (analyzeSwim),
r.competition/r.venue (renderResults), r.venue (renderOverview recent PBs), swim.venue
(renderSplits split detail). Any new field from RAW rendered into innerHTML needs escaping.

### renderQTChartAndTable is the single source of truth for QT chart and table (v22)
Used by both renderQualifying() and renderRegionalQualifying(). Changes to chart format,
table columns, or tooltip text only need to be made here. Do NOT duplicate logic back into
the two tab functions.
