# Session Log

## v28.1 (latest)
Replaces an initial standalone-editor-tabs attempt with an inline edit mode
within the existing County QT and Regional QT tabs, plus a round of UX fixes.

### Inline QT editor (replaces standalone tabs)
An early implementation added separate "Edit County QT" / "Edit Regional QT"
tabs. This was superseded by an inline edit mode toggled from within the
existing County QT / Regional QT tabs themselves, avoiding duplicate tab
chrome and keeping editing contextually tied to the data being edited.

- Meta banner (`.qt-meta-banner`) added above both QT tabs, always visible,
  showing championship title + date range and a desktop-only
  "✏️ Edit Details & Times" button
- `.qt-view-mode` / `.qt-edit-mode` blocks coexist in the DOM; visibility
  controlled by toggling `.qt-tab-in-edit-mode` on the parent `.panel`
- `openQTEditor(viewPrefix)` / `closeQTEditor(viewPrefix)` — open deep-clones
  live QT_DATA/SE_QT_DATA + meta into a session-scoped `qtEditorStore`; close
  commits the store back to live data via `commitEditorStoreToLive(prefix)`
- Editor uses a distinct element-ID prefix (`cqt`/`rqt`) from the view tab
  (`qt`/`rq`) via `editorPrefixFor()`, avoiding ID collisions
- `renderQTBanner(viewPrefix)` populates the banner; called from `showTab()`
  on entering either QT tab, and once from `init()`
- Editor table (`renderEditorTable()`) filtered by Age Group (required),
  Gender, Stroke, Course; supports add/delete row, inline time editing,
  meta panel edit (title + date range), and JSON download in the v28
  `{meta,times}` schema
- Desktop-only: edit button and editor hidden below 769px viewport width

### Six UX fixes (addressed together within the v28.1 editor pass)
- Time input format unified and validated (`isValidTimeInput()`); empty input
  represents "not offered" rather than an error state
- Duplicate Gender+Course+Event+Age rows flagged across the whole dataset via
  `findDuplicateIndices()`, not just the currently filtered view
- "Not offered" rows (qualify and consider both null) now visually greyed out
  with disabled inputs and an explanatory note instead of looking broken
- Editor filter dropdowns use full stroke names (Freestyle, Backstroke, …)
  to match the view-mode filters, instead of an inconsistent abbreviated set
- Meta banner's "no dates set" hint is responsive: desktop references the
  edit button by name; mobile says to switch to desktop (since the button
  itself is hidden there)
- "← Back" button now both commits the editor store and returns to view mode
  in a single action, rather than requiring a separate save step first

### Banner copy consistency fix
The desktop "no dates set" hint was tightened to quote the edit button's exact
visible label ("Edit Details & Times") instead of a loose paraphrase, so users
can find the right control without guessing.

---

## v28
JSON error handling, QT schema migration, and a data file rename — laying the
groundwork for the v28.1 inline editor.

### JSON load error handling
Every `JSON.parse()` call site in `init()` (8 total: my_swims.json,
county_qt.json, regional_qt.json, upcoming_races.json, each in both their
fresh-fetch and localStorage-cache code paths) wrapped in try/catch. Failures
call `showJsonLoadError(filename, message)`, which renders a persistent,
stackable warning banner at the top of the Overview tab rather than failing
silently or leaving stale/empty data with no explanation.

### se_london_qt.json renamed to regional_qt.json
`SE_QT_DATA_URL` updated to the new filename; Data Manager upload label updated
to match. Internal variable and function names (`SE_QT_DATA`, `SE_QT_META`,
`renderRegionalQualifying()`, etc.) intentionally left unchanged — this was a
file/label rename only, not a variable rename, to minimise the diff.

### QT JSON schema migrated to `{meta, times}`
`county_qt.json` and `regional_qt.json` now carry a `meta` object (`title`,
`dateFrom`, `dateTo`) alongside the existing `times` array, to support the
upcoming championship banner and inline editor.
- New `unwrapQT(raw)` helper normalises either the old bare-array format or
  the new wrapped format to `{meta, times}` — fully backward compatible
- New `QT_META` / `SE_QT_META` globals hold the unwrapped meta object
- `QT_DATA` / `SE_QT_DATA` remain flat arrays (the unwrapped `.times`), so
  every existing consumer (`renderQualifying`, `renderRegionalQualifying`,
  `getQTStatusForEvent`, `renderTargets`) needed zero changes

---

## v27
Major redesign of County QT and Regional QT tabs.

### Core change: dual-course unified view
Both QT tabs previously showed only one course at a time (filtered by a Course select).
v27 removes the Course filter entirely and always shows SC and LC simultaneously,
addressing the UX issue where a swimmer had to switch courses to see their full status.

### Code review fixes (post-v27 iteration)

**B1/I5 — Dead CSS removed**
.qt-chart-toggles, .qt-toggle-btn and active variants removed after chart removal.
.qt-table-toggles and .qt-row-toggle retained and clarified with comments.

**B3 — Mobile course block + progress bar overrides**
Added @media (max-width:500px) overrides for .qt-course-block (padding, margin),
.qt-progress-wrap (height, margin-top), .qt-marker-time (font-size, top).
Corrected .qt-pb-val mobile font from stale 0.92rem to 0.82rem.

**I2 — Persist renewalMonths across sessions**
onchange now calls localStorage.setItem('swimDash_renewalMonths', this.value).
init() restores saved value before first render (fallback to '6' if invalid).

**P2 — Unified empty state messages**
Both QT table empty states now read "No events match the current filters." consistently.

**I3 — Progress bar track always visible**
.qt-progress-wrap background: var(--surface) → var(--surface3) + border:1px solid var(--border).
Track is now visible even when no PB, anchoring the CT/QT markers clearly.

**I6 — qtToggleState renamed to qtRowState**
All 5 references renamed. 3-line comment added clarifying row-only purpose (chart removed).

**P3 — Duplicate race warning in removeRace()**
Now counts all matching entries (date+event+course+time) before deleting.
If >1 match found, confirm dialog warns "⚠️ N identical entries found — only the first
will be removed." Single match behaviour unchanged.

### Chart removed from both QT tabs (post-v27 iteration)
The "PB vs Qualifying & Consideration Times" bar chart was removed from both County
and Regional QT tabs due to confusion over bar colours and SC/LC overlap.
- Chart HTML card divs removed from both tabs
- toggleQTChart() function removed
- Chart-building block (~90 lines) removed from renderQTCards()
- renderQTCards() signature simplified: chartId and legendId params removed (now 6 args)
- destroyChart calls removed from no-data guards
- qtToggleState retained — still used by SC/LC row toggles on the event table

### New rendering pipeline (replaces renderQTChartAndTable + old buildQtStatCards)

**buildQtStatCards(scMatched, lcMatched, statsElId)**
- Counts best status per event across SC+LC
- Stat cards list events with: course badge (SC/LC), best PB, gap chip
- Gap logic: Qualified→largest margin inside QT; Consideration→smallest gap to QT; Outside→smallest gap to CT
- Cards are clickable filter shortcuts — click sets Status filter and re-renders tab
- Prefix derived from statsElId to target correct select (qtStatus or rqStatus)

**buildProgressBar(pbSec, qualify, consider)**
- Per-event scaling with slowAnchor (CT×1.06) and fastAnchor (QT×0.988)
- Always rendered even with no PB, so CT/QT markers are always visible
- CT and QT times labelled above marker lines
- Gap text NOT returned — rendered once in stats row above (prevents LC duplicate)

**applyStatusFilter(allEventNames, scMatched, lcMatched, statusFilter)**
- New function; filters event list by best status across courses
- Qualified: ≥1 course Qualified; Consideration: best=Consideration; Outside: all Outside; No PB: no PB either course

**renderQTCards(scMatched, lcMatched, cardGridId, chartId, legendId, tableBodyId, prefix, statusFilter)**
- Builds event card grid (4-col desktop / 1-col mobile)
- Each card: SC block (surface2 bg) + LC block (surface3 bg), left border = course status colour
- No status chip inside block — status conveyed by border colour only
- Builds 6-dataset chart (SC PB, SC QT, SC CT, LC PB, LC QT, LC CT)
- Builds event-by-event table (one row per event+course, SC/LC row toggles)

**toggleQTChart(prefix, course) / toggleQTRows(prefix, course)**
- SC/LC toggle buttons above chart and table
- Chart toggle: mutates dataset.hidden via qtToggleState; no re-render
- Row toggle: adds/removes .hidden class on qt-tbl-row-sc / qt-tbl-row-lc rows

### New Status filter
Dropdown added to both QT tab filter bars.
Reset functions updated to also clear Status filter.

### Stat card fixes (iterative within v27 session)
- Added course badge (SC/LC) next to PB in event list rows
- Removed "Click to filter" subtitle label (retained as title attr tooltip)
- Removed underline (border-bottom) from event list rows
- All items left-aligned (flex-wrap:wrap, no justify-content:space-between)
- Consistent 0.68rem font on desktop; 0.58rem on mobile for event rows
- No PB events show event name only (no badge/time rendered when bestPB null)
- "— No PB" → "No PB" (dash removed from badge text throughout)
- Mobile stroke abbreviations: Free→FR, Back→BK, Breast→BR, Fly→FL, IM unchanged.
  abbrevEvent() helper inside buildQtStatCards(); col-full/col-abbr pattern applied.
- Mobile alignment fix: align-self:stretch on event list div overrides stat-card
  align-items:center; margin-left:0 !important on gap chips prevents delta right-drift.
- Scroll-to-cards (all screen sizes): clicking a stat card scrolls to the
  Qualification Status — SC & LC section. Uses window.scrollTo with a
  getBoundingClientRect() offset to account for the sticky header height (+ 8px buffer).
  cardSectionId derived from prefix (qtCardSection / rqCardSection).
- "inside" hidden on mobile in Qualified gap text: wrapped in col-full span so desktop
  shows "▼ 0.52s inside QT" and mobile shows "▼ 0.52s QT".

### Course block fixes
- Status chip removed; left border colour conveys status
- Gap chip shown once only in stats row above bar
- LC and SC rows now visually consistent

### Mobile fixes
- QT event-by-event table: added font-size 0.60rem, padding 5px 3px, badge scaling
  at max-width 500px (matching Schedule/Targets pattern, lost in earlier v27 iteration)
- "Course" column header → col-full/col-abbr pattern ("C" on mobile)

### Bracket note
Added #qtNote / #rqNote elements below each event-by-event table.
Populated by renderQualifying() and renderRegionalQualifying() with gender, age,
championship year, and next-season notice if champs date has passed.

### Chart redesign
Old chart: single "best PB" bar vs SC QT/CT lines only.
New chart: SC PB bars + LC PB bars + SC QT/CT lines + LC QT/CT lines (6 datasets).
SC/LC toggle buttons above chart control dataset visibility without re-render.

### Event-by-event table (restored and enhanced)
Previously removed in early v27 implementation. Restored with:
- One row per event+course (not one row per event)
- SC/LC row toggle buttons
- Course column abbreviated to "C" on mobile

---

## v26.3
Addresses logical bugs and architectural issues.

### Fix 1A — getBestSeasonImprovement() season logic corrected
### Fix 1B — Ghost view on active Splits tab resolved
### Fix 3A — Eager rendering anti-pattern removed from updateData()
### Fix 3B — Complete tab guarding strategy
### Filter guard bypass pattern (force wrappers)
### Fix 2A — Stale documentation contradiction removed

---

## v26.2 — Code Quality & Futureproofing Pass
### A. State management refactor — renderedTabs Set
### B. Dead code removal
### C. Single event list source of truth — ALL_EVENTS
### D. Logic fixes (empty DATA guard, UTC offset bug, QT data missing message, etc.)
### E. Security — data-* attributes + event delegation
### F. CSS fixes — badge colours, mobile chart height, regional stat-val
### G. UX — Escape key closes modals, past-dated schedule log prompt, adapter pinned
### H. Data safety — splits normalised to []

---

## v26.1
- FAB parent ＋ centred fix
- Schedule Days column — "Today" only on daysUntil===0
- Overview Recent PBs — Competition in sub-line

---

## v26
- Multi-event Add Race modal
- Add Upcoming Race (📅 Speed Dial child + modal)
- Remove Upcoming Event (✕ in Schedule)
- Download Upcoming Races
- Overview "Best Improvement" stat card
- All Overview stat cards clickable
- Schedule hides already-swum events

---

## v25
- Speed Dial FAB — single ➕ parent expands 4 child actions upward

## v24.1
- Logo: GitHub raw URL. Status badge font fix. 100 IM added. renderQTCells() font-size fix.

## v24
- Schedule tab. Targets tab. Swimmer Settings. Auto age-bracket. Overview 6 stat cards.

## v23
- Dark/light theme toggle. PALETTE_LIGHT/DARK. getCC()/getPalette().

## v22
- escapeHtml(pb.venue). renderQTChartAndTable shared. Progress bar 12.5x scale.

## v21
- escapeHtml() across all innerHTML. getPBs() fixes. processData sort stable.

## v20–v9 — see earlier entries
