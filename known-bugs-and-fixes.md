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

### Overview "PBs this year" / "Recent PBs" used isPB not getPBs() (v21)
Fixed: both now use getPBs() which returns only the current fastest per event+course.

### processData sort unstable for same-date SC/LC (v21)
Fixed: course.localeCompare tiebreaker — S always before L.

### Doughnut chart border dark navy (v21)
Fixed to '#ffffff'.

### User-supplied strings unescaped in innerHTML (v21/v22)
Fixed: escapeHtml() applied to competition, venue everywhere.

### Logo MIME type mismatch + base64 corruption (v24.1)
The embedded image data was a JPEG but src said data:image/png.
Additionally a spurious fffd byte was introduced during a repair attempt.
Final fix: removed all embedded base64; replaced with GitHub raw URL:
  src="https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/bpsc_logo.png"
onerror hides the element gracefully if offline.

### Status badge font size inconsistency in Targets renewal table (v24.1)
badge class carries font-size:0.62rem overriding table CSS.
Fixed: plain span with colour/padding only; font-size inherits from table.

### renderQTCells() inline font-size overrode mobile table scaling (v24.1)
All font-size:0.8rem inline styles removed. Spans inherit from table CSS.

### 100 IM missing from ALL_EVENTS and Add Race dropdown (v24.1)
Added to both. Full list now: 50/100/200/400/800/1500 Free,
50/100/200 Back, 50/100/200 Breast, 50/100/200 Fly, 100/200/400 IM.

### Confirmation flash on wrong element (v25)
Was targeting .fab-add (the old parent button). Moved to #fabAddRace child button.

### Four stacked FABs cluttering screen (v25)
Replaced with Speed Dial: single ➕ parent, four child actions fan upward on tap.


## WATCH-OUT AREAS

### isPB vs getPBs().includes() — use the right one for the right purpose
| Check                    | True when                              | Use for                           |
|--------------------------|----------------------------------------|-----------------------------------|
| r.isPB                   | Swim was fastest at time of swimming   | Historical "was this a PB?" check |
| getPBs().includes(swim)  | Swim is the current fastest ever       | "Is this the PB right now?"       |

### Debut detection — four contexts
| Context                  | Method                                                |
|--------------------------|-------------------------------------------------------|
| Results Δ Prev           | r.deltaPrev === null                                  |
| Overview Recent PBs      | swimCount === 1                                       |
| Progression Improv / BvF | swims===1 || !refSwim || (ref===pb && mode!='latest') |
| Splits analyzeSwim       | prevPB===null (date filtered)                         |

### processData sort order (v21)
Stable: date ASC → event name ASC → course ASC (S before L).
deltaPrev and isPB depend on this order.

### escapeHtml must be applied to ALL user-supplied strings in innerHTML
competition, venue from RAW. Any new RAW field rendered into HTML needs escaping.
Event and course are dropdown-constrained — safe without escaping.

### updateData() does NOT call renderSplits(), renderSchedule(), renderTargets()
renderSchedule/renderTargets depend on UPCOMING which only changes via upload/reload.

### split validation: !isFinite not isNaN
### fmtDate/fmtDateShort: always use parseLocalDate()
### QT course strings vs DATA codes: "Short Course"/"Long Course" vs "S"/"L"
### buildQtStatCards shared — changes affect both County and Regional tabs
### renderQTChartAndTable shared — changes affect both County and Regional tabs
### progChartWrap restoration: querySelector → restore → destroyChart → create/replace
### removeRace: first findIndex match on date+event+course+time
### Pacing Profile Y-axis: reverse:true intentional

### getCC() and getPalette() must be called inside render functions (v23)
Never at module level. Every chart render function injects at top:
  const CC = getCC(); const PALETTE = getPalette();

### getSwimmerDOB() / getSwimmerGender() (v24)
NOT constants — functions that read localStorage. Do not cache at module level.
saveSettings() persists and re-renders all affected tabs.

### getCountyAgeBracket() / getRegionalAgeBracket() (v24)
Called at render time — not cacheable. Depend on getSwimmerDOB() which may change.

### renderQTCells() — no inline font-size (v24.1)
Returns [statusCell, gapCell]. No font-size on spans — mobile table CSS must apply.
Gap: Outside → CT gap; Consideration → QT gap; Qualified → ✓.

### ALL_EVENTS list must stay in sync with Add Race dropdown (v24.1)
Current: 50/100/200/400/800/1500 Free, 50/100/200 Back, 50/100/200 Breast,
50/100/200 Fly, 100/200/400 IM (18 events total).

### Speed Dial FAB — closeFabDial() must be called first (v25)
Every function that opens a modal or triggers a UI action must call closeFabDial()
at its very top. Currently: showAddRaceModal, showDataModal, showSettingsModal,
toggleTheme. Any new action added to the dial must follow the same pattern.

### fabTheme id must stay on the child button (v25)
applyTheme() uses document.getElementById('fabTheme') to sync the 🌙/☀️ icon.
Do not remove or rename this id.

### Logo (v24.1+)
Loaded via GitHub raw URL — not embedded. Works online; hidden gracefully offline.
Do not re-embed as base64 without fixing MIME type (must be image/jpeg not image/png)
and verifying the full JPEG is intact using Pillow before encoding.

### Upcoming races 48-hour grace period
cutoff = today - 2 days. Applied in both renderSchedule() and renderTargets().

### Competition column truncation (v24)
.comp-full (30 chars desktop) / .comp-short (20 chars mobile). Adjacent spans in each row.
