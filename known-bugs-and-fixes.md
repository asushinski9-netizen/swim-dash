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
### processData sort unstable for same-date SC/LC (v21)
### Doughnut chart border dark navy (v21)
### User-supplied strings unescaped in innerHTML (v21/v22)
### Logo MIME type mismatch + base64 corruption (v24.1)
### Status badge font size inconsistency in Targets (v24.1)
### renderQTCells() inline font-size overrode mobile scaling (v24.1)
### 100 IM missing from ALL_EVENTS and Add Race dropdown (v24.1)
### Confirmation flash on wrong element (v25) — moved from parent to child fabAddRace
### Four stacked FABs cluttering screen (v25) — replaced with Speed Dial

### Schedule table colspan mismatch (v26)
Empty-state <td> had colspan="11". After adding the Del column the table has 12 columns.
Fixed: colspan="12".

### Add Race modal: single flat form (v26)
The modal's flat form did not support adding multiple events in one sitting.
Redesigned: Date/Venue/Course/Competition shared at top; per-event rows with Event, Time,
Splits, and ✕ remove button. saveNewRace() iterates all rows and pushes one RAW entry each.

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
Stable: date ASC → event name ASC → course ASC (S before L)

### escapeHtml must be applied to ALL user-supplied strings in innerHTML
competition, venue from RAW. Any new RAW field rendered into HTML needs escaping.
Event and course are dropdown-constrained — safe without escaping.

### updateData() does NOT call renderSplits(), renderSchedule(), renderTargets()
renderSchedule/renderTargets depend on UPCOMING which only changes via:
  - saveNewUpcoming() (v26)
  - removeUpcomingEvent() (v26)
  - saveDataToBrowser() + reload
  - saveSettings() (re-renders for age bracket changes)

### UPCOMING data authority (v26)
swimDash_UPCOMING is GitHub-authoritative on first load, but is also mutated locally by
saveNewUpcoming() and removeUpcomingEvent(). These changes persist to localStorage but do
NOT push to GitHub. User must download via ⬇️ Download upcoming_races.json and commit.

### removeUpcomingEvent() — event property format (v26)
UPCOMING[i].events is an array of { event: string } objects.
The filter `e => e.event !== event` relies on this schema.
saveNewUpcoming() always pushes { event: select.value }, so the format is guaranteed
consistent for user-added entries. GitHub-sourced entries must also match this schema.

### buildSwumUpcomingSet() date window (v26)
Uses ±1 day (86400000ms) to account for meets that span multiple days or date-entry drift.
If a race is logged with a date 2+ days off from the meet date it may not be suppressed.

### getBestSeasonImprovement() season boundary (v26)
Uses getSeasonStart() → SEASON_START_MONTH (default 9 = September).
"First swim of season" is the chronologically earliest DATA entry for that event+course
on or after Sept 1. If there are no swims from before the current PB in this season,
that event+course is skipped (improvement = 0 or negative → excluded).

### Speed Dial FAB — closeFabDial() must be called first (v25/v26)
Every function that opens a modal or triggers a UI action must call closeFabDial()
at its very top: showAddRaceModal, showAddUpcomingModal, showDataModal,
showSettingsModal, toggleTheme. Any new dial action must follow the same pattern.

### fabTheme id must stay on the child button (v25/v26)
applyTheme() uses document.getElementById('fabTheme') to sync the 🌙/☀️ icon.

### Add Race modal — multi-event (v26)
addRaceEventRow() caps at 6 rows. Each row is a styled card div.
saveNewRace() validates per-row: time format + splits. Batch duplicate event check.
The shared "Course" field applies to ALL event rows in that modal session.
If different events need different courses, open the modal twice.

### Add Upcoming modal — merge behaviour (v26)
If a meet with the same date+course already exists in UPCOMING, events are merged
(new events appended, existing ones skipped). Competition/venue fields are NOT updated
if they already exist on the target meet. No confirmation is shown for the merge —
the modal just closes and re-renders.

### Schedule "hidden events" note (v26)
buildSwumUpcomingSet() is called fresh on every renderSchedule(). It is NOT cached.
The note appended to scheduleNote will update automatically when new results are added
via updateData() → the schedule is re-rendered via showTab() on next visit.
(Schedule is NOT re-rendered by updateData() directly — only on tab switch.)

### getCC() and getPalette() must be called inside render functions (v23)
Never at module level. Every chart render function injects at top:
  const CC = getCC(); const PALETTE = getPalette();

### getSwimmerDOB() / getSwimmerGender() (v24)
NOT constants — functions that read localStorage. Do not cache at module level.

### getCountyAgeBracket() / getRegionalAgeBracket() (v24)
Called at render time — not cacheable.

### renderQTCells() — no inline font-size (v24.1)
Returns [statusCell, gapCell]. No font-size on spans — mobile table CSS must apply.

### ALL_EVENTS list must stay in sync with Add Race dropdown (v24.1/v26)
Current (18 events): 50/100/200/400/800/1500 Free, 50/100/200 Back,
50/100/200 Breast, 50/100/200 Fly, 100/200/400 IM.
addRaceEventRow() and addUpcomingEventRow() both define ALL_EVENT_OPTIONS inline —
keep these in sync with the renderTargets() ALL_EVENTS array if new events are added.

### Logo (v24.1+)
Loaded via GitHub raw URL. Works online; hidden gracefully offline via onerror.

### Upcoming races 48-hour grace period
cutoff = today - 2 days. Applied in renderSchedule(), renderTargets(), and
the upcomingMap construction inside renderTargets(). Also in buildSwumUpcomingSet()
(via the rows array which has already been cutoff-filtered by renderSchedule()).

### Competition column truncation (v24)
.comp-full (30 chars desktop) / .comp-short (20 chars mobile). Adjacent spans per row.

### Schedule tab colspan = 12 (v26)
Table now has 12 columns (added Del column). Empty-state td must use colspan="12".
