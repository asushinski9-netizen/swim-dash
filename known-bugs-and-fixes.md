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
### All items from v26.2 quality pass (see session-log.md)
### getBestSeasonImprovement() false positive (v26.3) — endpoint is bestInSeason
### Ghost view on Splits tab after updateData() (v26.3)
### Eager rendering in updateData() (v26.3)
### Incomplete tab guarding (v26.3)

### v27 QT tab — Course filter removed
Old single-course filter replaced by dual-course unified view.
resetQualifyingFilters() and resetRegionalFilters() no longer reference qtCourse/rqCourse.

### v27 QT tab — buildQtStatCards() old single-arg signature removed
Now takes (scMatched, lcMatched, statsElId). Old call sites removed.
Stat cards are clickable filter shortcuts — prefix derived from statsElId.

### v27 QT tab — renderQTChartAndTable() removed
Replaced entirely by renderQTCards() which handles cards, chart, and table.

### v27 QT tab — Stat card "SC or LC" / "Both courses" subtitle inconsistency
Fixed: subtitle removed entirely; "Click to filter" in title attr only.

### v27 QT tab — Duplicate gap text on LC progress bar
Gap text rendered once only in stats row above bar; removed from buildProgressBar() output.

### v27 QT tab — Status chip inside course block
Replaced by left border colour on the course block div. No chip label rendered.

### v27 QT tab — "— No PB" dash prefix
Removed from course block badge and card header badge. Now just "No PB" / "No Data".

### v27 QT tab — Mobile table scaling lost
Re-added: font-size 0.60rem, padding 5px 3px, badge scaling for both QT tab tables
at max-width 500px, matching Schedule and Targets pattern.

### v27 QT tab — Stat card event list alignment
Items flow left-to-right (flex, flex-wrap:wrap, no justify-content:space-between).
No flex:1 on event name — all items naturally left-aligned.

---

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

### QT tab — two separate rendering pipelines
1. **QT tab cards/chart/table**: buildQtStatCards() + buildProgressBar() + applyStatusFilter() + renderQTCards()
   - Takes scMatched + lcMatched arrays; shows both courses simultaneously
   - Used only by renderQualifying() and renderRegionalQualifying()
2. **Schedule + Targets**: getQTStatusForEvent() + renderQTCells()
   - Looks up a single course at a time from QT_DATA / SE_QT_DATA
   - KEEP SEPARATE — do not merge these pipelines

### qtToggleState — module-level, persists within session
```js
const qtToggleState = { qt: {SC: true, LC: true}, rq: {SC: true, LC: true} };
```
toggleQTChart() and toggleQTRows() mutate this object.
State is NOT reset when filters change or tabs re-render — intentional so user's
toggle preference is preserved across filter changes within a session.
Resets to {SC:true, LC:true} on page reload only.

### applyStatusFilter() — operates on event names, not matched rows
Takes allEventNames array and both scMatched/lcMatched to determine best status.
Returns filtered event name array. renderQTCards() then uses this to filter
both the card grid and table (but NOT the stat cards — those always show all events).

### buildProgressBar() — gap text NOT returned
Gap text is intentionally excluded from buildProgressBar() output.
It is rendered once only in the stats row above the bar (inside buildCourseBlock).
Do NOT add gap text back to buildProgressBar() — it would duplicate on LC rows.

### Course block border colour — status of that specific course
The left border on each qt-course-block reflects that course's own status, not the
overall card best status. This is intentional — it shows per-course qualification.

### Card header badge — best status across both courses
The badge in .qt-card-header reflects the best status across SC+LC.
"No PB" and "No Data" show without a dash prefix.

### Stat card clickable shortcut — prefix derivation
buildQtStatCards() derives prefix from statsElId:
  prefix = statsElId.startsWith('rq') ? 'rq' : 'qt'
The Status select ID is then `${prefix}Status` and the force function is
`forceRegional` or `forceQualifying`. If a new QT tab is added, ensure its
statsElId starts with a unique prefix.

### renderedTabs Set — three usage patterns
- `renderedTabs.clear()` — full reset; invalidateCache() and renderAllChartTabs()
- `renderedTabs.delete('tabname')` — targeted reset; forceX() wrappers, reset/sort fns
- `renderedTabs.has('tabname')` — guard at top of each render function

### forceX() wrappers — required for all filter onchange handlers
Filter onchange attributes must call forceX() wrappers, NOT render functions directly.
Current wrappers: forceProgression(), forcePBs(), forceResults(), forceQualifying(), forceRegional().
Any new filter-bearing tab needs its own forceX() wrapper.

### renderAllChartTabs() — single registration point
New chart-bearing tabs must be added here only.
toggleTheme() and saveSettings() call this — they need no other changes.

### Intentionally unguarded render functions
- renderSplits() — filter-event driven
- renderSchedule() — depends on UPCOMING independently
- renderTargets() — same; also has renewal threshold select

### updateData() active-tab-only re-render
updateData() re-renders the active tab + Schedule + Targets. All other tabs render lazily.

### ALL_EVENTS constant — single source of truth
Declared in CONFIGURATION section. Do NOT add inline event arrays anywhere else.

### getBestSeasonImprovement() — within-season only
Uses bestInSeason (fastest within current season) not all-time currentPB.
Requires ≥2 swims this season. Skips event if season opener is already season's fastest.

### buildSwumUpcomingSet() — date window
Uses ±1 day (86400000ms) tolerance. Races logged 2+ days off the meet date may not suppress.

### Schedule Days column labels
daysUntil: 0→Today, 1→Tomorrow, -1→Yesterday, <-1→"Xd ago", >1→"in Xd".
Never use <= 0 for Today.

### Schedule tab colspan = 12
12 columns (includes Del column). Empty-state td must use colspan="12".

### getCC() and getPalette() — inside render functions only
Never at module level. Always declare at top of render function:
  const CC = getCC(); const PALETTE = getPalette();

### Mobile col-abbr/col-full pattern
Used for: QT/CT columns in both QT tabs, "Course"→"C" in QT event tables,
Competition column in Schedule/Overview. Add to any new table column that
would overflow on mobile.

### escapeHtml() — required for all user-supplied strings in innerHTML
competition, venue from RAW. Event and course are dropdown-constrained.

### UPCOMING data authority
swimDash_UPCOMING is GitHub-authoritative on first load, also mutated locally.
User must download and commit to persist.

### removeUpcomingEvent() — event property format
UPCOMING[i].events is array of { event: string }. Filter uses `e.event !== event`.

### Logo — GitHub raw URL
Loaded via GitHub raw URL. Hidden gracefully offline via onerror.
