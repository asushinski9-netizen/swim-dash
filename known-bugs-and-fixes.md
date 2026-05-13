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
ones since beaten. Fixed: both now use getPBs() which returns only the current fastest swim
per event+course.

### processData sort unstable for same-date same-event SC/LC (v21)
Fixed: added course.localeCompare as tiebreaker → stable: same date + same event + S before L.

### Doughnut chart border was dark navy (v21)
Fixed to '#ffffff'.

### User-supplied strings unescaped in innerHTML (v21)
Fixed: escapeHtml() applied everywhere.

### Missing escapeHtml in renderPBs (v21.1 / v22)
pb.venue fixed.

### uniqueEventsWithSplits sort non-deterministic (v22)
localeCompare tiebreaker added.

### Progress bar scale too coarse (v22)
Updated to impPct * 12.5 so 8% fills the bar.

### Logo MIME type mismatch (v24.1)
The embedded image data is a JPEG but the src attribute said `data:image/png;base64`.
Fixed to `data:image/jpeg;base64`. Additionally a spurious `fffd` byte (JPEG COM marker)
had been introduced before the EOI `ffd9` marker during a prior repair attempt; stripped
by scanning backward to the last clean ffd9.

### Status badge font size inconsistency in Targets renewal table (v24.1)
The ⏰ Renew status used `<span class="badge">` which carries `font-size: 0.62rem` via
CSS. This made it too small on desktop and immune to the mobile `0.60rem !important`
override. Replaced with a plain `<span>` using only colour/padding inline styles so
font-size inherits from the table at all breakpoints.

### renderQTCells() inline font-size overrode mobile table scaling (v24.1)
All `font-size:0.8rem` inline styles removed from renderQTCells() spans.
The function now returns clean spans that inherit from the table's CSS rule.

### 100 IM missing from ALL_EVENTS and Add Race dropdown (v24.1)
Added to both. Full event list now: 50/100/200/400/800/1500 Free,
50/100/200 Back, 50/100/200 Breast, 50/100/200 Fly, 100/200/400 IM.


## WATCH-OUT AREAS

### isPB vs getPBs().includes() — use the right one for the right purpose
| Check                    | True when                              | Use for                           |
|--------------------------|----------------------------------------|-----------------------------------|
| r.isPB                   | Swim was fastest at time of swimming   | Historical "was this a PB?" check |
| getPBs().includes(swim)  | Swim is the current fastest ever       | "Is this the PB right now?"       |

### Debut detection — four contexts
| Context                  | Method                                                    | Reason                          |
|--------------------------|-----------------------------------------------------------|---------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                      | First chronological swim        |
| Overview Recent PBs      | swimCount === 1                                           | Only swim ever for event+course |
| Progression Improv / BvF | swims===1 || !refSwim || (ref===pb && mode!='latest')     | No meaningful reference         |
| Splits analyzeSwim       | prevPB===null (date filtered)                             | No earlier swim with splits     |

### refSwim===pb guard is mode-aware (v20.1)
latest mode: refSwim===pb → "PB is latest" (normal case).
first/prevSwim/prevBest modes: refSwim===pb → Debut (backdated first+fastest).

### processData sort order (v21)
Stable: date ASC → event name ASC → course ASC (S before L).
deltaPrev and isPB calculations depend on this order being deterministic.

### escapeHtml must be applied to ALL user-supplied strings in innerHTML (v21)
competition, venue from RAW; any future RAW fields rendered into HTML need escaping.
Event and course are dropdown-constrained and safe.

### analyzeSwim prevPB filters by swim.course
Ensures SC and LC PBs are always separate benchmarks regardless of the tab's course filter.

### init() data priority
1. RAW: localStorage → GitHub fallback → modal
2. QT / SE_QT / UPCOMING: localStorage → GitHub fetch if any are missing

### updateData() does NOT call renderSplits(), renderSchedule(), or renderTargets()
renderSplits() is omitted by design (no full re-render needed on data change).
renderSchedule() and renderTargets() depend on UPCOMING which only changes via
file upload/page reload, not via RAW data edits.

### split validation: !isFinite not isNaN
### fmtDate/fmtDateShort: always use parseLocalDate()
### QT course strings vs DATA codes: "Short Course"/"Long Course" vs "S"/"L"
### buildQtStatCards shared — changes affect both County and Regional tabs
### renderQTChartAndTable shared — changes affect both County and Regional tabs
### progChartWrap restoration: querySelector → restore → destroyChart → create/replace
### removeRace: first findIndex match on date+event+course+time
### Pacing Profile Y-axis: reverse:true intentional
### Time Progression: last in HTML source — no CSS ordering rules needed

### getCC() and getPalette() must be called inside render functions, not at module level (v23)
Every render function that uses chart colours must inject at its top:
  const CC = getCC(); const PALETTE = getPalette();
DO NOT use module-level constants — they evaluate once and miss theme changes.

### toggleTheme() re-renders chart tabs but does NOT call processData()
DATA is unchanged when theme switches. toggleTheme() calls:
renderOverview(), renderProgression(), renderSplits(), renderQualifying(), renderRegionalQualifying()
Does NOT call renderPBs(), renderResults(), renderSchedule(), renderTargets().

### swimDash_DOB and swimDash_GENDER (v24)
Read by getSwimmerDOB() and getSwimmerGender(). NOT module-level constants.
Default values applied if localStorage keys are absent.
saveSettings() persists them and re-renders all affected tabs.

### getCountyAgeBracket() / getRegionalAgeBracket() (v24)
Called at render time — do not cache at module level. They depend on getSwimmerDOB()
which reads from localStorage and may change between renders (after saveSettings()).

### renderQTCells() — no inline font-size (v24.1)
Returns [statusCell, gapCell] pair. Both cells must NOT have inline font-size so
the mobile CSS font-size override applies uniformly across all table cells.
Gap logic: Outside → CT gap; Consideration → QT gap; Qualified → ✓ check.

### ALL_EVENTS list (v24.1)
Must match Add Race modal dropdown options exactly.
Current list (18 events): 50/100/200/400/800/1500 Free, 50/100/200 Back,
50/100/200 Breast, 50/100/200 Fly, 100/200/400 IM.
If a new event is added to either list, update BOTH.

### Competition column truncation (v24)
Uses .comp-full (30 chars, desktop) and .comp-short (20 chars, mobile) CSS classes.
Both are rendered as adjacent <span> elements in each schedule table row.
The CSS display:none/inline swap happens at the 500px breakpoint.

### Upcoming races 48-hour grace period
cutoff = today - 2 days. Meets before cutoff are excluded in both renderSchedule()
and renderTargets(). This prevents stale meets from appearing for ~48hrs after they occur.

### Logo (v24.1)
Embedded as data:image/jpeg;base64 (NOT image/png).
Data is a valid 180x180 JPEG: starts ffd8, ends ffd9, no intermediate fffd bytes.
Logo is fetched at runtime — works when online, hidden gracefully (onerror) when offline.
