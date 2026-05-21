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
### Confirmation flash on wrong element (v25)
### Four stacked FABs cluttering screen (v25)
### Schedule table colspan mismatch (v26) — fixed to colspan="12"
### Add Race modal: single flat form (v26) — redesigned to multi-event

### All items from v26.2 quality pass
- overviewRendered hoisting risk — replaced by renderedTabs Set in STATE block
- getSwimmerProfile() dead code removed
- Overview upcomingCount inflated by swum events — fixed via buildSwumUpcomingSet()
- Overview crash on empty DATA — guard added
- getBestSeasonImprovement() UTC offset bug — local date components used
- Targets silent when QT data missing — informative message added
- processData() missing splits field — normalised to []
- removeRace/removeUpcomingEvent inline onclick — data attributes + delegation
- .badge.S/.badge.L hardcoded hex in dark theme — CSS variables
- Progression chart too short on mobile — 300px override
- .grid3 unused CSS class removed
- .tab dead border:none removed
- Stale (v25) CSS comment removed
- #tab-regional .stat-val mobile override added
- No Escape key on modals — keydown listener in init()
- No log prompt for past-dated schedule rows — ⚠️ Log? badge added
- Overview PBs to Renew hardcoded at 6 months — reads from #renewalMonths select
- populateSelects() cleared user filters — snapshot/restore added
- toggleTheme() manual render list — renderAllChartTabs() helper
- renderQTCells() No Data silent — now shows italic "Not offered"
- chartjs-adapter-date-fns unpinned — pinned to @3.0.0
- renderQTChartAndTable missing PALETTE — added for pattern consistency
- Overview upcoming count included swum events — fixed
- Best Improvement .toFixed(1) — changed to .toFixed(2)

### FAB parent ＋ not centred (v26.1) — line-height:1; padding:0 added
### Schedule "Today" shown for yesterday's race (v26.1) — exact daysUntil===0 check

### getBestSeasonImprovement() false positive (v26.3)
The function compared firstOfSeason vs all-time currentPB (potentially from a prior season),
producing a positive "improvement" even when the swimmer had not improved within the season.
Fixed: endpoint is now bestInSeason (fastest swim within the current season). Event is
skipped if the season opener is already the season's fastest swim.

### Ghost view on Splits tab after updateData() (v26.3)
updateData() never called renderSplits(), so the Splits tab showed stale data when a race
was added/removed while it was active. Fixed: updateData() now re-renders only the active
tab via showTab(), letting renderedTabs guards handle lazy rendering of all others.

### Eager rendering in updateData() (v26.3)
updateData() forced all 6 tabs to render immediately, defeating the renderedTabs lazy
guard. Fixed: only the active tab re-renders; all others render lazily on next visit.

### Incomplete tab guarding (v26.3)
Only renderOverview() had a renderedTabs guard. All filter-bearing render functions now
have guards. Filter onchange attributes call forceX() wrappers that delete the guard
before rendering. Reset functions and sortResults() also delete the guard before rendering.


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

### escapeHtml must be applied to ALL user-supplied strings in innerHTML
competition, venue from RAW. Event and course are dropdown-constrained — safe without escaping.

### renderedTabs Set — three usage patterns (v26.2)
- `renderedTabs.clear()` — full reset; use in invalidateCache() and renderAllChartTabs()
- `renderedTabs.delete('tabname')` — targeted reset; use when only one tab needs refreshing,
  or in forceX() wrappers and reset/sort functions
- `renderedTabs.has('tabname')` — guard at top of each render function

### forceX() wrappers — required for all filter onchange handlers (v26.3)
Filter onchange attributes must call forceX() wrappers, NOT render functions directly.
The wrappers call renderedTabs.delete() before rendering, bypassing the lazy guard.
Current wrappers: forceProgression(), forcePBs(), forceResults(), forceQualifying(), forceRegional().
Any new filter-bearing tab needs its own forceX() wrapper and must use it in all
onchange attributes, reset functions, and sort functions.

### renderAllChartTabs() — single registration point (v26.2)
New chart-bearing tabs must be added to renderAllChartTabs() only.
toggleTheme() and saveSettings() call this — they need no other changes.

### Intentionally unguarded render functions (v26.3)
- renderSplits() — filter-event driven; always re-renders on event select change
- renderSchedule() — depends on UPCOMING which changes independently of renderedTabs
- renderTargets() — same as Schedule; also has renewal threshold select
These must NOT receive renderedTabs guards.

### updateData() active-tab-only re-render (v26.3)
updateData() re-renders the active tab + Schedule + Targets. All other tabs render lazily.
Do NOT add explicit render calls for other tabs back into updateData() — it defeats the
lazy rendering system. If a tab needs immediate refresh after data change, add a
renderedTabs.delete() + render call there specifically with a comment explaining why.

### ALL_EVENTS constant — single source of truth (v26.2)
Declared in CONFIGURATION section. addRaceEventRow(), addUpcomingEventRow(), and
renderTargets() all reference it. Do NOT add inline event arrays anywhere else.
Adding a new event: one line change in CONFIGURATION, automatically propagates everywhere.

### getBestSeasonImprovement() — within-season only (v26.3)
Uses bestInSeason (fastest within current season) not all-time currentPB as endpoint.
Requires ≥2 swims this season. Skips event if season opener is already season's fastest.
Season boundary: getSeasonStart() → SEASON_START_MONTH (default 9 = September).
Date string built from local components — not toISOString() (UTC offset bug).

### buildSwumUpcomingSet() — called from renderOverview() (v26.2)
Must remain defined before renderOverview() in file order (or remain a function declaration
so it is hoisted). Currently in the Schedule section — safe via hoisting.

### Event delegation for remove buttons (v26.2)
resultsTbody and scheduleBody have delegated click listeners set up once in init().
Buttons use .btn-remove-race and .btn-remove-upcoming classes. No inline onclick.

### UPCOMING data authority (v26)
swimDash_UPCOMING is GitHub-authoritative on first load, also mutated locally by
saveNewUpcoming() and removeUpcomingEvent(). User must download and commit to persist.

### removeUpcomingEvent() — event property format
UPCOMING[i].events is an array of { event: string }. Filter uses `e.event !== event`.
saveNewUpcoming() always pushes { event: select.value } — format is consistent.

### buildSwumUpcomingSet() date window
Uses ±1 day (86400000ms) tolerance. Races logged 2+ days off the meet date may not suppress.

### Schedule Days column labels (v26.1)
daysUntil: 0 → Today (gold), 1 → Tomorrow (accent), -1 → Yesterday (muted),
< -1 → "Xd ago" (muted), > 1 → "in Xd". Never use <= 0 for Today.

### Schedule tab colspan = 12 (v26)
12 columns (added Del column). Empty-state td must use colspan="12".

### Competition column truncation (v24)
.comp-full (30 chars desktop) / .comp-short (20 chars mobile).

### getCC() and getPalette() must be called inside render functions (v23)
Never at module level. Every render function injects at top:
  const CC = getCC(); const PALETTE = getPalette();

### getSwimmerDOB() / getSwimmerGender() — not constants (v24)
Functions that read localStorage. Do not cache at module level.

### getCountyAgeBracket() / getRegionalAgeBracket() — not cacheable (v24)
Called at render time. Depend on getSwimmerDOB() which may change.

### renderQTCells() — no inline font-size (v24.1)
Returns [statusCell, gapCell]. No font-size on spans — table CSS applies.
Gap: Outside → CT gap; Consideration → QT gap; Qualified → ✓.
No Data → italic "Not offered" (v26.2).

### Logo (v24.1+)
Loaded via GitHub raw URL. Works online; hidden gracefully offline via onerror.

### Upcoming races 48-hour grace period
cutoff = today - 2 days. Applied in renderSchedule(), renderTargets(),
upcomingMap construction, and buildSwumUpcomingSet() row filtering.
