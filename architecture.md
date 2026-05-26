# Code Architecture

## File
Single HTML file: index.html (swimming_dash_v27.html)
Current version: v27
~3831 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL, UPCOMING_DATA_URL,
                    COUNTY_CHAMPS_DATE, REGIONAL_CHAMPS_DATE, SEASON_START_MONTH,
                    ALL_EVENTS (canonical event list — single source of truth)
2.  Swimmer profile functions — getSwimmerDOB(), getSwimmerGender()
3.  DATA globals — RAW, QT_DATA, SE_QT_DATA, UPCOMING, DATA
4.  STATE globals — renderedTabs (Set), sortCol, sortDir, fabDialOpen,
                    currentEventFiltered, contextualPBRef, splitBarChart, paceChart,
                    qtToggleState (SC/LC chart+row toggle state for qt and rq prefixes)
5.  cache{} + invalidateCache() — calls renderedTabs.clear()
6.  THEME — getCC(), getPalette(), applyTheme(), toggleTheme()
7.  STATE MANAGEMENT — updateData(newRaw)
8.  DATA MANAGER MODAL — showDataModal(), hideDataModal(), saveDataToBrowser(),
                          resetDashboard(), downloadRaceData(), downloadUpcomingData()
9.  UTILITIES — escapeHtml(), timeToSec(), secToTime(), parseLocalDate(),
                fmtDate(), fmtDateShort(), getStroke(), getDistance(), processData(),
                getAgeAtEndOfYear(), getQTSeasonYear(), getSeasonStart(),
                getCountyAgeBracket(), getRegionalAgeBracket(),
                getQTStatusForEvent(), renderQTCells()
10. Derived helpers — uniqueEvents(), uniqueVenues(), uniqueYears(),
                      uniqueEventsWithSplits() — all memoised via cache{}
11. PALETTE_LIGHT/DARK + getPalette()
12. MEMOISED LOOKUPS — getPBs(), getFirstSwims(), getLatestSwims(),
                       getPreviousSwims(), getPreviousBestSwims(), getSwimCounts()
13. getBestSeasonImprovement()
14. FILTER RESET HELPERS — forceX() wrappers (forceProgression, forcePBs, forceResults,
                            forceQualifying, forceRegional), then resetX() functions
15. CHARTS REGISTRY — charts{}, destroyChart()
16. POPULATE SELECTS — populateSelects() (snapshots/restores values across rebuilds)
17. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits(), renderSplitCharts(), analyzeSwim(),
                  renderResults(),
                  buildQtStatCards(), buildProgressBar(), applyStatusFilter(),
                  renderQTCards(), toggleQTChart(), toggleQTRows(),
                  renderQualifying(), renderRegionalQualifying(),
                  buildSwumUpcomingSet(), renderSchedule(), renderTargets()
18. RENDER ALL CHART TABS — renderAllChartTabs()
19. TAB SWITCHING — showTab()
20. INIT — async init() — includes keydown (Escape) listener and delegated click
           listeners for .btn-remove-race and .btn-remove-upcoming
21. ADD RACE MODAL — showAddRaceModal(), hideAddRaceModal(),
                     addRaceEventRow(), saveNewRace()
22. REMOVE RACE — removeRace()
23. REMOVE UPCOMING EVENT — removeUpcomingEvent()
24. SPEED DIAL FAB — toggleFabDial(), openFabDial(), closeFabDial()
25. SETTINGS MODAL — showSettingsModal(), hideSettingsModal(), saveSettings()
26. ADD UPCOMING RACE MODAL — showAddUpcomingModal(), hideAddUpcomingModal(),
                               addUpcomingEventRow(), saveNewUpcoming()
27. Theme IIFE + DOMContentLoaded → init()

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

## QT Tab Architecture (v27)

### Two tabs: County QT (#tab-qualifying) and Regional QT (#tab-regional)
Both use the same rendering pipeline. Prefix `qt` = County, `rq` = Regional.

### Filter bar
Gender · Age Group · Stroke · Status (new in v27)
Course filter removed — both courses always shown simultaneously.

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

### Chart — REMOVED
The "PB vs Qualifying & Consideration Times" chart was removed from both QT tabs.
toggleQTChart() is removed. No chart is rendered in renderQTCards().

### qtToggleState — SC/LC row visibility state only
```js
const qtToggleState = { qt: {SC: true, LC: true}, rq: {SC: true, LC: true} };
```
- toggleQTRows(prefix, course) — toggles .qt-tbl-row-sc or .qt-tbl-row-lc via .hidden class
Updates corresponding toggle button active class. No re-render needed.

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
These are SEPARATE from the v27 QT tab rendering pipeline.
They look up a single course at a time and are used only by Schedule and Targets.

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
swimDash_QT       — GitHub authoritative (re-fetched if missing)
swimDash_SE_QT    — GitHub authoritative (re-fetched if missing)
swimDash_UPCOMING — GitHub authoritative on first load; also mutated locally
swimDash_theme    — 'light' or 'dark'
swimDash_DOB      — Swimmer date of birth
swimDash_GENDER   — Swimmer gender
