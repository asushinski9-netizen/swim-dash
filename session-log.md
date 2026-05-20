# Session Log

## v26.2 (latest) — Code Quality & Futureproofing Pass

### A. State management refactor
- STATE GLOBALS block introduced immediately after DATA GLOBALS — all mutable `let`
  variables (sortCol, sortDir, fabDialOpen, currentEventFiltered, contextualPBRef,
  splitBarChart, paceChart) moved here from their scattered locations.
- `overviewRendered` boolean replaced by `renderedTabs` Set throughout. `renderedTabs.clear()`
  is the single reset point; `renderedTabs.delete('overview')` for targeted Overview refreshes.
- `renderAllChartTabs()` helper added before showTab() — toggleTheme() and saveSettings()
  call it instead of manual render lists. Add new chart tabs here only.

### B. Dead code removal
- `getSwimmerProfile()` removed (declared but never called).
- `.grid3` CSS class removed (defined but never used in HTML).
- Dead `border: none` removed from `.tab` ruleset (immediately overridden by the next line).
- Stale `(v25)` version annotation removed from FAB CSS comment.
- `const PALETTE = getPalette()` added to `renderQTChartAndTable()` for pattern consistency.

### C. Single event list source of truth
- `ALL_EVENTS` constant declared in CONFIGURATION section.
- `addRaceEventRow()`, `addUpcomingEventRow()`, and `renderTargets()` all reference it —
  three separate inline arrays eliminated.
- `renderQTCells()` 'No Data' case now renders as italic muted "Not offered" — visually
  distinct from "No PB" so users understand the event isn't available at their age group.

### D. Logic fixes
- `renderOverview()` empty-DATA guard added — no crash if called before races are loaded.
- `getBestSeasonImprovement()` builds season start string from local date components
  instead of toISOString() — fixes UTC offset bug in BST/UTC+ timezones.
- `renderTargets()` shows informative message when QT data is not loaded, instead of
  silently showing the "all current" green message.
- `removeUpcomingEvent()` and `saveNewUpcoming()` both call renderedTabs.delete('overview')
  + renderOverview() so the Upcoming Races stat card updates immediately.
- Overview upcomingCount now calls buildSwumUpcomingSet() to filter already-swum events —
  matches the Schedule tab count exactly.
- Overview renewalCount reads from #renewalMonths select instead of hardcoded 6 months.
  Stat card sub-label is now dynamic (e.g. "Older than 3 months").
- #renewalMonths onchange also triggers renderedTabs.delete('overview') + renderOverview().
- populateSelects() snapshots and restores select values — filters no longer reset when
  a race is added or removed.

### E. Security
- removeRace and removeUpcomingEvent buttons use data-* attributes instead of inline
  onclick string interpolation with user data.
- Delegated click listeners on resultsTbody and scheduleBody set up once in init().

### F. CSS fixes
- .badge.S and .badge.L use var(--accent) / var(--accent2) — adapt correctly to dark theme.
- #progChartWrap gets height: 300px !important on mobile (was compressed to 240px by
  global .chart-wrap override).
- #tab-regional .stat-val gets font-size: 1.3rem !important on mobile — matches qualifying tab.

### G. UX
- Escape key closes any open modal or Speed Dial (single keydown listener in init()).
- Past-dated unlogged schedule rows show a subtle ⚠️ Log? gold badge on the event cell.
- chartjs-adapter-date-fns pinned to @3.0.0.

### H. Data safety
- processData() normalises splits to [] if the field is missing or not an array.

### Fix (post v26.2)
- Best Improvement stat card used .toFixed(1) — changed to .toFixed(2) to match the
  2 decimal place precision used throughout the dashboard.

## v26.1
- Fix: FAB parent ＋ glyph centred — added line-height:1; padding:0 to .fab-parent.
- Fix: Schedule Days column — "Today" only on daysUntil===0; "Yesterday" for -1; "Xd ago" for older.
- UX: Schedule column order — Event, Days, Course, PB, County Status, County Gap,
  Regional Status, Regional Gap, Date, Competition, Venue, Del.
- UX: Overview Recent PBs — sub-line shows Competition (full desktop / 30-char mobile)
  instead of Venue. Reuses existing .comp-full / .comp-short CSS classes.

## v26

### Feature: Multi-event Add Race modal
The flat single-entry Add Race form was redesigned so Date, Competition, Venue, and Course
are shared at the top, with a repeating row for each event's Event, Time, and Splits.
- `addRaceEventRow()` dynamically appends a styled card row with a select, time input, and splits input.
- Max 6 event rows per session. Each row has its own ✕ remove button.
- `saveNewRace()` iterates all rows, validates each time field, and pushes one RAW entry per row.
- Duplicate event detection within a batch: errors if the same event appears twice.
- Confirmation flash moves to the ⏱️ child FAB as in v25.
- Mobile: shared fields sit in a 1×2 grid above the event list; event rows are full-width cards.

### Feature: Add Upcoming Race (📅 Speed Dial child + modal)
A new 📅 child button was added to the Speed Dial (2nd from bottom, between ⏱️ and ⚙️).
- `showAddUpcomingModal()` / `hideAddUpcomingModal()` — standard open/close pattern.
- `addUpcomingEventRow()` — appends a simple select + ✕ row; max 6 events per modal session.
- `saveNewUpcoming()` — validates and pushes to UPCOMING; merges into existing meet if same date+course.
  Persists to `swimDash_UPCOMING` in localStorage. Calls `renderSchedule()` + `renderTargets()`.
- Confirmation flash on `#fabAddUpcoming`.
- Speed Dial now has 5 children (bottom→top): ⏱️ Add Race, 📅 Add Upcoming, ⚙️ Data Manager, 🌙 Theme, 👤 Settings.
- CSS stagger delay: `.fab-child-row:nth-child(5) { transition-delay: 0.20s; }` added.

### Feature: Remove Upcoming Event (✕ button in Schedule table)
- Schedule table gains a 12th column (Del) with a ✕ button per row.
- `removeUpcomingEvent(date, event, course)` splices the event from the matching UPCOMING meet,
  removing the meet entirely if its events array becomes empty.
- Persists to localStorage and re-renders Schedule + Targets.
- Confirm dialog reminds user to download and push JSON for permanent effect.
- Empty-state `<td>` colspan updated from 11 → 12.
- Mobile CSS: `#tab-schedule .tbl-wrap .badge` scaling added.

### Feature: Download Upcoming Races from Data Manager
- `downloadUpcomingData()` serialises `UPCOMING` and triggers browser download as `upcoming_races.json`.
- Button added to Data Manager modal immediately after the existing ⬇️ Download my_swims.json button.

### Feature: Overview "Best Improvement" stat card (replaces "Events" card)
- 3rd stat card now shows the **Best Season Improvement** — the largest absolute time drop
  (first swim of season → current PB) across all event+course pairs with ≥2 swims this season.
- `getBestSeasonImprovement()` computes this; `getSeasonStart()` returns Sept 1 of the
  current swim season year (configurable via `SEASON_START_MONTH = 9`).
- Card navigates to Progression tab on click. Displays `▼ Xs` in green; `—` in muted if no improvement found.
- Sub-label shows the winning event+course (e.g. "200 Back (S)").

### Feature: All Overview stat cards now clickable
All 6 stat cards now have `onclick="showTab(...)"` and `cursor:pointer`. Previously only
"Upcoming Races" and "PBs to Renew" were clickable. New link targets:
  - Total Races → All Results tab
  - Current PBs → Personal Bests tab
  - Best Improvement → Progression tab
  - Races with Splits → Splits tab

### Feature: Schedule tab hides already-swum events
`buildSwumUpcomingSet()` cross-references each upcoming row against DATA (within a 1-day
window of the meet date). Swum events are filtered out of the visible table. A note line
appended to `scheduleNote` reports how many events were hidden.

### Configuration
`SEASON_START_MONTH = 9` (September, 1-indexed) — controls the season boundary used by
`getSeasonStart()`. Change to adjust when the swim season begins.

---

## v25
- **Feature: Speed Dial FAB** — single ➕ parent expands 4 child actions upward.
  Children (bottom→top): ⏱️ Add Race, ⚙️ Data Manager, 🌙 Theme, 👤 Settings.
  Backdrop, staggered entrance animation, parent rotates to ✕ when open.
  closeFabDial() called at the top of every action function.

## v24.1
- Logo: GitHub raw URL. Status badge font fix. 100 IM added. renderQTCells() inline
  font-size removed. Competition truncation 30/20 chars.

## v24
- Schedule tab. Targets tab. Swimmer Settings modal. Auto age-bracket from DOB.
- Overview 6 stat cards (grid6). renderQTCells() two-column layout.

## v23
- Dark/light theme toggle. PALETTE_LIGHT/DARK. getCC()/getPalette().

## v22
- escapeHtml(pb.venue). renderQTChartAndTable shared helper. Progress bar 12.5x scale.

## v21
- escapeHtml() across all innerHTML. getPBs() fixes in Overview. processData sort stable.

## v20–v9 — see earlier entries
