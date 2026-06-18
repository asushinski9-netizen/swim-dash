# Code Architecture

## File
Single HTML file: index.html
Current version: v28.1
~4512 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL (now points to
                    regional_qt.json — see "QT file rename" below), UPCOMING_DATA_URL,
                    COUNTY_CHAMPS_DATE, REGIONAL_CHAMPS_DATE, SEASON_START_MONTH,
                    ALL_EVENTS (canonical event list — single source of truth)
2.  Swimmer profile functions — getSwimmerDOB(), getSwimmerGender()
3.  DATA globals — RAW, QT_DATA, SE_QT_DATA (flat arrays only), QT_META, SE_QT_META
                    (new in v28 — see "QT schema migration"), UPCOMING, DATA
4.  unwrapQT(raw) — schema-compatibility helper (new in v28, see below)
5.  STATE globals — renderedTabs (Set), sortCol, sortDir, fabDialOpen,
                    currentEventFiltered, contextualPBRef, splitBarChart, paceChart
5b. qtRowState — declared later in file (near QT render pipeline), not here;
                    SC/LC row toggle state for qt and rq prefixes
6.  cache{} + invalidateCache() — calls renderedTabs.clear()
7.  THEME — getCC(), getPalette(), applyTheme(), toggleTheme()
8.  STATE MANAGEMENT — updateData(newRaw)
9.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData(), downloadUpcomingData()
10. UTILITIES — escapeHtml(), timeToSec(), secToTime(), parseLocalDate(),
                fmtDate(), fmtDateShort(), getStroke(), getDistance(), processData(),
                getAgeAtEndOfYear(), getQTSeasonYear(), getSeasonStart(),
                getCountyAgeBracket(), getRegionalAgeBracket(),
                getQTStatusForEvent(), renderQTCells()
11. Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears(),
                      uniqueEventsWithSplits() — all memoised via cache{}
12. PALETTE_LIGHT/DARK + getPalette()
13. MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
14. getBestSeasonImprovement()
15. FILTER RESET HELPERS — forceX() wrappers (forceProgression, forcePBs, forceResults,
                            forceQualifying, forceRegional), then resetX() functions
16. CHARTS REGISTRY — charts{}, destroyChart()
17. POPULATE SELECTS — populateSelects() (snapshots/restores values across rebuilds)
18. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits(), renderSplitCharts(), analyzeSwim(),
                  renderResults(),
                  buildQtStatCards(), buildProgressBar(), applyStatusFilter(),
                  qtRowState, toggleQTRows(),
                  renderQTCards(), renderQualifying(), renderRegionalQualifying(),
                  buildSwumUpcomingSet(), renderSchedule(), renderTargets()
19. RENDER ALL CHART TABS — renderAllChartTabs()
20. TAB SWITCHING — showTab() (now also calls renderQTBanner('qt')/('rq')
                    when entering the qualifying/regional tabs — see QT Editor section)
21. JSON LOAD ERROR DISPLAY — showJsonLoadError(filename, message) (new in v28,
                    see below)
22. INIT — async init() — includes keydown (Escape) listener and delegated click
           listeners for .btn-remove-race and .btn-remove-upcoming
23. ADD RACE MODAL — showAddRaceModal(), hideAddRaceModal(),
                     addRaceEventRow(), saveNewRace()
24. REMOVE RACE — removeRace()
25. REMOVE UPCOMING EVENT — removeUpcomingEvent()
26. SPEED DIAL FAB — toggleFabDial(), openFabDial(), closeFabDial()
27. SETTINGS MODAL — showSettingsModal(), hideSettingsModal(), saveSettings()
28. ADD UPCOMING RACE MODAL — showAddUpcomingModal(), hideAddUpcomingModal(),
                               addUpcomingEventRow(), saveNewUpcoming()
29. QT EDITOR (new in v28.1, ~345 lines) — see dedicated section below
30. Theme IIFE + DOMContentLoaded → init()

---

## QT schema migration (v28)

`county_qt.json` and `regional_qt.json` now use a wrapped schema instead of a bare
array:

```json
{
  "meta": { "title": "Middlesex County Championships", "dateFrom": "2027-02-01", "dateTo": "2027-02-14" },
  "times": [ { "gender": "Boys", "course": "Short Course", "event": "50 Free", "age": "12", "qualify": 32.10, "consider": 34.90 }, ... ]
}
```

### unwrapQT(raw)
```js
function unwrapQT(raw) {
  if (!raw) return { meta: {}, times: [] };
  try {
    const parsed = typeof raw === 'string' ? JSON.parse(raw) : raw;
    if (Array.isArray(parsed)) return { meta: {}, times: parsed };   // old flat format
    if (parsed && Array.isArray(parsed.times)) return parsed;        // new schema
    return { meta: {}, times: [] };
  } catch { return { meta: {}, times: [] }; }
}
```
Accepts either the legacy bare-array format or the new `{meta,times}` format and
always normalises to `{meta, times}`. Used in `init()` for both fresh GitHub fetches
and localStorage-cached reads, so older cached JSON keeps working without a
migration step.

`QT_DATA` and `SE_QT_DATA` are always flat arrays (the unwrapped `.times`) — every
existing consumer (`renderQualifying`, `renderRegionalQualifying`,
`getQTStatusForEvent`, `renderTargets`) is unchanged and unaware of the wrapper.
`QT_META` / `SE_QT_META` hold `{title, dateFrom, dateTo}` and are consumed only by
`renderQTBanner()` (see QT Editor section).

## QT file rename (v28)
`SE_QT_DATA_URL` now points to `regional_qt.json` (was `se_london_qt.json`).
Internal JS variable/function names (`SE_QT_DATA`, `SE_QT_META`, `renderRegionalQualifying`,
etc.) were deliberately left unchanged — only the filename and user-facing labels changed.

## JSON load error handling (v28)

### showJsonLoadError(filename, message)
Creates (or reuses) a persistent `#jsonLoadErrorBanner` div inserted as the first
child of `#tab-overview`. Each call appends one stacked entry:
`⚠ Failed to load <filename> — <message>`, plus a hint to check for trailing
commas/syntax errors and re-upload via the Data Manager. Both `filename` and
`message` are passed through `escapeHtml()`.

Called from every `JSON.parse()` site in `init()` — covers the four data files,
in both their "fresh GitHub fetch" and "localStorage cache" code paths (8 call
sites total: my_swims.json ×2, county_qt.json ×2, regional_qt.json ×2,
upcoming_races.json ×2). A malformed file degrades to an empty/cached dataset
for that file rather than blocking the rest of init().

---

## Lazy rendering system

### renderedTabs Set
Central state variable. Tracks which tabs have completed an initial render since the
last data or theme change.

| Operation | When to use |
|---|---|
| `renderedTabs.clear()` | Full reset — in invalidateCache() and renderAllChartTabs() |
| `renderedTabs.delete('tab')` | Targeted reset — in forceX() wrappers, reset fns, sortResults() |
| `renderedTabs.has('tab')` | Guard at top of each guarded render function |
| `renderedTabs.add('tab')` | At the bottom of each guarded render function |

### Guarded render functions
renderOverview, renderProgression, renderPBs, renderResults,
renderQualifying, renderRegionalQualifying.

### Intentionally unguarded
- renderSplits() — filter-event driven; always re-renders on event select change
- renderSchedule() — depends on UPCOMING which changes independently
- renderTargets() — same; also has renewal threshold select
- renderQTBanner() — cheap, always reflects current QT_META/SE_QT_META

### forceX() wrappers
All filter onchange attributes call force wrappers, NOT render functions directly.
```js
function forceProgression()  { renderedTabs.delete('progression'); renderProgression(); }
function forcePBs()          { renderedTabs.delete('pbs');         renderPBs(); }
function forceResults()      { renderedTabs.delete('results');     renderResults(); }
function forceQualifying()   { renderedTabs.delete('qualifying');  renderQualifying(); }
function forceRegional()     { renderedTabs.delete('regional');    renderRegionalQualifying(); }
```

---

## QT Tab Architecture (v27, view-mode unchanged in v28.1)

### Two tabs: County QT (#tab-qualifying) and Regional QT (#tab-regional)
Both use the same rendering pipeline. Prefix `qt` = County, `rq` = Regional.
Since v28.1 each tab is split into a **view mode** and an **edit mode** (see QT
Editor section) — everything in this section describes view mode, which is
unchanged from v27.

### Filter bar (view mode)
Gender · Age Group · Stroke · Status
Course filter removed (v27) — both courses always shown simultaneously.

### Stat cards — buildQtStatCards(scMatched, lcMatched, statsElId)
- Counts best status per event across SC+LC
- Each card lists events with: course badge (SC/LC), best PB time, gap chip
- Gap logic: Qualified → largest margin inside QT; Consideration → smallest gap to QT;
  Outside → smallest gap to CT (both across SC and LC)
- Clicking a card sets the Status filter select and re-renders the tab
- Prefix derived from statsElId to target correct select ('qtStatus' or 'rqStatus')
- Mobile: font-size 0.58rem for event rows; desktop: 0.68rem

### Status filter — applyStatusFilter(allEventNames, scMatched, lcMatched, statusFilter)
Filters the unified event list by best status:
- Qualified  → at least one course is Qualified
- Consideration → best across courses is Consideration (none Qualified)
- Outside → both course PBs are Outside (or No PB is not Outside by definition)
- No PB → no PB on either course (status No PB or No Data)
Returns filtered event name array consumed by renderQTCards().

### Progress bar — buildProgressBar(pbSec, qualify, consider)
Per-event scaling: slowAnchor = CT * 1.06, fastAnchor = QT * 0.988.
Faster times position further right. Always rendered even when pbSec is null,
so CT/QT markers are visible for events with no PB.
CT and QT times shown as small labels above their marker lines.
Gap text NOT included in bar output — rendered once only in the stats row above.

### Course block design — buildCourseBlock (inner fn in renderQTCards)
- SC block: background var(--surface2); LC block: background var(--surface3)
- Left border coloured by that course's own status (green/amber/red/grey)
- Stats row: course badge · PB time · gap chip — all left-aligned, consistent font
- Status shown via left border colour only (no chip label)
- No PB: shows "No PB" badge (no dash prefix)
- Not offered at age group: shows "Not offered" badge

### Card grid
4-column grid on desktop (≥1200px), 2-column at 1200px, 1-column on mobile (≤640px).
Card border colour = best status across both courses.
Card header badge = best status label (no dash prefix for No PB/No Data).

### Chart — REMOVED (v27)
The "PB vs Qualifying & Consideration Times" chart was removed from both QT tabs.
toggleQTChart() is removed. No chart is rendered in renderQTCards().

### qtRowState — SC/LC row visibility state (row toggles only)
```js
// Drives event-by-event table SC/LC row visibility only. Chart was removed in v27.
const qtRowState = { qt: {SC: true, LC: true}, rq: {SC: true, LC: true} };
```
- toggleQTRows(prefix, course) — toggles .qt-tbl-row-sc or .qt-tbl-row-lc via .hidden class
Updates corresponding toggle button active class. No re-render needed.
Do NOT add chart-related keys — use separate state if a chart is re-added.

### Event-by-event table
One row per event+course combination.
Row classes: qt-tbl-row-sc (plain bg) / qt-tbl-row-lc (subtle teal tint).
.hidden class applied when toggle is off.
"Course" column header uses col-full/col-abbr pattern (shows "C" on mobile).
Table respects the status filter (only filtered events appear).

### Stat card mobile helpers (inside buildQtStatCards)
**abbrevEvent(ev)** — maps stroke words for mobile display:
  Free→FR, Back→BK, Breast→BR, Fly→FL, IM unchanged.
  Applied via col-full/col-abbr spans inside evList rows. Scoped to stat cards only.

**Mobile alignment** — event list div uses align-self:stretch to override the mobile
  stat-card align-items:center. Gap chips use margin-left:0 !important to stay left.

**Scroll on click (all screen sizes)** — makeCard() uses window.scrollTo() with a
  getBoundingClientRect() offset to scroll to the Qualification Status section,
  clearing the sticky header (header height + 8px buffer). 150ms delay for DOM settle.
  cardSectionId = prefix === 'rq' ? 'rqCardSection' : 'qtCardSection'
  Uses the .card wrapper element (includes section title), not the grid element.
  scrollIntoView not used — it ignores sticky header height.

**Qualified gap chip mobile** — "inside" in "▼ Xs inside QT" wrapped in col-full span;
  hidden on mobile so chip reads "▼ Xs QT" and does not wrap.

### Bracket note
Populated by renderQualifying() and renderRegionalQualifying() into #qtNote / #rqNote.
Format: "Qualifying times shown for {gender} · age group {age} · {Championship} {year}."
Includes next-season notice if champs date has passed.

### updateData() active-tab-only re-render
updateData() re-renders the active tab + Schedule + Targets. All other tabs render lazily.

---

## QT Editor Architecture (v28.1)

Both QT tabs now have an inline **edit mode** alongside the existing view mode,
replacing an earlier (now superseded) attempt at separate standalone editor tabs.
Edit mode is desktop-only — the entry button is hidden below the 769px breakpoint.

### View/edit toggle (CSS-driven)
```css
.qt-view-mode  { display: block; }
.qt-edit-mode  { display: none;  }
.qt-tab-in-edit-mode .qt-view-mode { display: none; }
.qt-tab-in-edit-mode .qt-edit-mode { display: block; }
```
`openQTEditor(viewPrefix)` / `closeQTEditor(viewPrefix)` toggle the
`.qt-tab-in-edit-mode` class on the parent `.panel` (`#tab-qualifying` /
`#tab-regional`). No DOM is created or destroyed — both `.qt-view-mode` and
`.qt-edit-mode` blocks exist in the HTML at all times; the class controls which
is visible.

### Meta banner — always visible (both modes)
A `.qt-meta-banner` sits above the view/edit blocks in both tabs, showing the
championship title and date range, plus an "✏️ Edit Details & Times" button
(`.btn-edit-qt`, desktop-only via the 769px media query) that calls
`openQTEditor()`.

**renderQTBanner(viewPrefix)** populates the banner title/dates from
`QT_META`/`SE_QT_META`. If no dates are set, shows a responsive hint via two
classes toggled at the same 769px breakpoint as `.btn-edit-qt`:
- `.qt-banner-note-desktop` — "No dates set — click "Edit Details & Times" to
  configure" (must match the button's exact label)
- `.qt-banner-note-mobile` — "No dates set — switch to Desktop mode to edit"

Called from `showTab()` (when entering the qualifying/regional tab) and from
`init()` (once, after data load, before the first render).

### Editor prefix mapping
The editor uses a **separate element ID prefix** from the view tab to avoid ID
collisions between view-mode and edit-mode elements:
- View prefix: `qt` (County) / `rq` (Regional)
- Editor prefix: `cqt` (County) / `rqt` (Regional)
- `editorPrefixFor(viewPrefix)` — maps `'qt'→'cqt'`, anything else→`'rqt'`
- `qtEditorSource(prefix)` — given an editor prefix, returns the matching data
  accessors and target filename (`county_qt.json` / `regional_qt.json`)

### Open/close/commit lifecycle
- `openQTEditor(viewPrefix)` — adds `.qt-tab-in-edit-mode`; on first open per
  session, deep-clones the live `QT_DATA`/`SE_QT_DATA` (+ meta) into a separate
  `qtEditorStore` so edits don't mutate live data until committed
- `closeQTEditor(viewPrefix)` — removes `.qt-tab-in-edit-mode`; calls
  `commitEditorStoreToLive(prefix)` to write the editor store back into the live
  globals and re-render the view tab
- `commitEditorStoreToLive(prefix)` — the only place editor edits become "real"

**⚠ Known caveat** — the editor store is only committed on "← Back" click or on
meta-panel "Save" (which commits immediately). Navigating away from the QT tab
by any other route (switching tabs without clicking Back) does not commit
row-level edits made since the store was cloned. See known-bugs-and-fixes.md.

### Editor table — renderEditorTable(prefix)
Renders one editable row per QT entry matching the editor's own filter bar
(Age Group required, Gender/Stroke/Course optional). Each row uses:
- `editorCell(prefix, idx, field, value)` — plain text/select cell
- `editorTimeCell(prefix, idx, field, inputEl)` — time-format input cell
- `deleteEditorRow(prefix, idx)` / `addEditorRow(prefix)` — row mutation
- `populateEditorAgeSelect(prefix)` — builds the Age Group dropdown from
  distinct ages present in that dataset
- `resetEditorFilters(prefix)` — clears Gender/Stroke/Course filters

Rows where both `qualify` and `consider` are `null` ("not offered" for that
age group) get a `.not-offered` class, disabled time inputs, and an
explanatory note instead of editable fields.

### Time input validation
- `secToTimeInput(sec)` — wraps `secToTime()` for editor display
- `isValidTimeInput(str)` — regex `^(\d{1,3}:)?\d{1,3}(\.\d{1,2})?$`;
  empty string is valid (represents "not offered")
- `timeInputToSec(str)` — wraps `timeToSec()` for committing edits

### Duplicate detection
`findDuplicateIndices(times)` scans the full dataset (not just the filtered
view) for rows sharing the same Gender+Course+Event+Age combination, and flags
each with a `.has-duplicate` class + `⚠ Duplicate` badge.

### Meta edit panel
`toggleMetaPanel(prefix)` shows/hides a small form (Championship Title, Date
From, Date To) above the editor table. `saveMetaPanel(prefix)` commits the
meta fields **immediately** to the live `QT_META`/`SE_QT_META` globals and
refreshes the banner — it does not wait for "← Back".

### Download
`downloadEditorJSON(prefix)` serialises the editor store back into the
`{meta, times}` wrapped schema and downloads it as `county_qt.json` or
`regional_qt.json`. Warns (via confirm-style alert) before downloading if any
row is missing Event/Age, or if unresolved duplicates remain.

---

## Speed Dial FAB (5 children)
Single parent ➕ button (bottom-right). Expands 5 child actions upward.
State: `let fabDialOpen = false`
Child button order (bottom→top):
  1. ⏱️ fabAddRace    — Add Race (multi-event)
  2. 📅 fabAddUpcoming — Add Upcoming Race
  3. ⚙️ (no id)        — Data Manager
  4. 🌙 fabTheme        — Toggle Theme
  5. 👤 fabSettings     — Swimmer Settings

---

## Swimmer profile
getSwimmerDOB() and getSwimmerGender() read from localStorage.
Defaults: '2014-10-12' / 'Boys'. Set via 👤 Settings modal.
saveSettings() persists and calls renderAllChartTabs() + renderSchedule() + renderTargets().

## Age bracket calculation
getAgeAtEndOfYear(year), getQTSeasonYear(champDateStr),
getCountyAgeBracket(), getRegionalAgeBracket()
Called at render time — NOT cached at module level.

## QT status helpers (Schedule + Targets)
getQTStatusForEvent(event, course, pbSec) → { county: {...}, regional: {...} }
renderQTCells(s) → [statusCell, gapCell]
These are SEPARATE from the QT tab rendering pipeline.
They look up a single course at a time from the flat QT_DATA/SE_QT_DATA arrays
and are used only by Schedule and Targets. Unaffected by the v28 schema migration
since QT_DATA/SE_QT_DATA remain flat arrays after unwrapQT().

## Upcoming races
localStorage: swimDash_UPCOMING. Fetched from UPCOMING_DATA_URL on first load.
Also mutated locally by saveNewUpcoming() and removeUpcomingEvent().
48-hour grace period applied in renderSchedule(), renderTargets(), buildSwumUpcomingSet().

## ALL_EVENTS constant — single source of truth
18 standard events in CONFIGURATION section.
Used in: addRaceEventRow(), addUpcomingEventRow(), renderTargets().

## processData() sort order
Stable: date ASC → event name ASC → course ASC (S before L).
Includes splits normalisation: Array.isArray(r.splits) ? r.splits : [].

## isPB vs getPBs().includes(swim)
r.isPB: historical (was fastest at time of swimming)
getPBs().includes(swim): current fastest ever

## escapeHtml() — required for all user-supplied strings in innerHTML
competition, venue from RAW. Event and course are dropdown-constrained — safe without escaping.
Also used by showJsonLoadError() for filename/message text (v28).

## getCC() and getPalette() — must be called inside render functions
Never at module level. Every render function injects at top:
  const CC = getCC(); const PALETTE = getPalette();

## getBestSeasonImprovement() — within-season only
Uses bestInSeason (fastest within current season) not all-time currentPB as endpoint.
Season boundary: getSeasonStart() → SEASON_START_MONTH (default 9 = September).

## renderAllChartTabs() — single registration point
New chart-bearing tabs must be added here only.
toggleTheme() and saveSettings() call this.

## Data authority
swimDash_RAW      — USER AUTHORITATIVE (never overwritten by GitHub)
swimDash_QT       — GitHub authoritative (re-fetched if missing); stored in the
                     {meta,times} wrapper as fetched, unwrapped on read via unwrapQT()
swimDash_SE_QT    — GitHub authoritative (re-fetched if missing); same wrapper handling.
                     Sourced from regional_qt.json (renamed in v28, see above)
swimDash_UPCOMING — GitHub authoritative on first load; also mutated locally
swimDash_theme    — 'light' or 'dark'
swimDash_DOB      — Swimmer date of birth
swimDash_GENDER   — Swimmer gender
