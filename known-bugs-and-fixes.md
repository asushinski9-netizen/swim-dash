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

### v27 QT tab — Stat card mobile centring override
Mobile CSS applies align-items:center to the whole stat card (for icon/number/label).
This centred the event list div too. Fixed with align-self:stretch on the event list
container, and margin-left:0 !important on .qt-gap-chip to prevent delta right-drift.

### v27 QT tab — Stat card mobile stroke abbreviations
abbrevEvent() helper inside buildQtStatCards() maps: Free→FR, Back→BK, Breast→BR,
Fly→FL, IM unchanged. Applied via col-full/col-abbr pattern inside evList rows.
Scoped to stat card event list only — event names elsewhere are unaffected.

### v27 — Dead CSS after chart removal (B1/I5)
.qt-chart-toggles, .qt-toggle-btn, .qt-toggle-btn.active-sc/.active-lc were defined
in CSS but no longer referenced after chart removal. Removed. .qt-row-toggle retained.

### v27 — Mobile course block and progress bar missing overrides (B3)
@media (max-width:500px) had no overrides for .qt-course-block or .qt-progress-wrap,
causing cramped layout and potential marker overlap on small screens. Fixed.

### v27 — renewalMonths resets to 6 on every reload (I2)
Threshold preference not persisted. Fixed: localStorage key swimDash_renewalMonths
saved on change, restored in init() before first render.

### v27 — Empty state messages inconsistent (P2)
"No events found." vs "No events match the current filters." used in different places.
Unified to "No events match the current filters." throughout.

### v27 — Progress bar track invisible when no PB (I3)
background: var(--surface) made the track white/dark (same as card background), so
CT/QT markers appeared to float. Fixed: background: var(--surface3) + border added.

### v27 — qtToggleState misleading name after chart removal (I6)
Renamed to qtRowState throughout (5 references). Comment added.

### v27 — removeRace() silent on duplicate entries (P3)
Two identical entries (same date/event/course/time) — only first deleted, no warning.
Fixed: counts matches first; confirm dialog warns if >1 found.

### v27 QT tab — Chart removed from County and Regional tabs
"PB vs Qualifying & Consideration Times" bar chart removed from both QT tabs.
Root cause: SC and LC bars were overlapping (not grouped side-by-side), causing the
LC "no PB" grey bars to cover correctly-coloured SC bars. Decision taken to remove
the chart entirely rather than continue debugging visual confusion.
renderQTCards() signature simplified to 6 args (chartId, legendId removed).
toggleQTChart() function removed. qtToggleState retained for row toggles.

### v27 QT tab — Stat card scroll to Qualification Status section
Clicking any stat card scrolls to the Qualification Status — SC & LC section on all
screen sizes. Uses window.scrollTo() with a live getBoundingClientRect() offset to
clear the sticky header (header height + 8px). Targets qtCardSection / rqCardSection
(the .card wrapper containing the section title), NOT the grid element itself.
scrollIntoView avoided because it does not account for sticky header height.

### v27 QT tab — Qualified gap chip "inside" hidden on mobile
"inside" in "▼ Xs inside QT" wrapped in <span class="col-full"> so it is hidden
at max-width 500px, showing "▼ Xs QT" instead to prevent wrapping.

### v28 — Silent failure on malformed JSON data files
Previously, a syntax error in any of the four data files (e.g. a trailing comma)
either threw uncaught or silently fell back to an empty dataset with no
indication to the user. Fixed: every JSON.parse() call site in init() (8 total,
covering fetch + localStorage paths for all four files) is wrapped in try/catch
and reports failures via showJsonLoadError(filename, message), which renders a
persistent, stackable banner at the top of the Overview tab.

### v28 — se_london_qt.json renamed to regional_qt.json
SE_QT_DATA_URL updated to the new filename. Internal variable/function names
(SE_QT_DATA, SE_QT_META, renderRegionalQualifying, etc.) deliberately left
unchanged — only the GitHub filename and user-facing labels changed. The Data
Manager modal's upload label was updated to reference regional_qt.json.

### v28 — QT JSON schema migrated to {meta, times} wrapper
county_qt.json and regional_qt.json now carry a meta object (title, dateFrom,
dateTo) alongside the existing times array, to support the new championship
banner and inline editor (see v28.1). Backward compatible: unwrapQT() accepts
either the old bare-array format or the new wrapped format and always returns
{meta, times}. QT_DATA/SE_QT_DATA remain flat arrays after unwrapping, so no
downstream rendering code needed to change.

### v28.1 — Inline QT editor replaces earlier standalone-tab approach
An initial attempt at separate, dedicated "Edit County QT" / "Edit Regional QT"
tabs was superseded by an inline edit mode within the existing County QT /
Regional QT tabs (toggled via an "✏️ Edit Details & Times" button in a
persistent meta banner). Avoids duplicating tab chrome and keeps the editor
contextually tied to the data it's editing. Desktop-only (hidden below 769px)
since the editing table layout doesn't fit comfortably on mobile.

### v28.1 — QT editor UX fixes (six items addressed together)
- Time inputs use a single shared format (`(\d{1,3}:)?\d{1,3}(\.\d{1,2})?`,
  empty = not-offered) validated via isValidTimeInput() — previously time entry
  had no validation feedback.
- Duplicate Gender+Course+Event+Age combinations are now detected across the
  full dataset (findDuplicateIndices()) and flagged with a ⚠ Duplicate badge,
  rather than silently allowing ambiguous duplicate rows.
- Rows representing "not offered" age groups (both qualify and consider null)
  are now visually greyed out with disabled inputs and an explanatory note,
  instead of showing blank/editable fields that looked broken.
- Stroke names in the editor's filter dropdowns use full words (Freestyle,
  Backstroke, etc.) matching the view-mode filters, rather than an inconsistent
  abbreviated set.
- The meta banner's "no dates set" message is now mobile-aware: desktop shows
  a hint referencing the "Edit Details & Times" button; mobile shows a
  "switch to Desktop mode to edit" message instead, since the edit button
  itself is hidden on mobile.
- "← Back to County/Regional QT" button flow clarified: clicking it commits
  the editor store to live data and returns to view mode in one action,
  rather than requiring a separate explicit save step.

### v28.1 — Banner copy consistency fix
The desktop "no dates set" hint originally paraphrased the edit button's label
loosely ("click Edit to configure"). Fixed so the hint text quotes the button's
exact visible label ("Edit Details & Times") to avoid user confusion about
which control to look for.

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
1. **QT tab cards/table (view mode)**: buildQtStatCards() + buildProgressBar() +
   applyStatusFilter() + renderQTCards()
   - Takes scMatched + lcMatched arrays; shows both courses simultaneously
   - Used only by renderQualifying() and renderRegionalQualifying()
2. **Schedule + Targets**: getQTStatusForEvent() + renderQTCells()
   - Looks up a single course at a time from QT_DATA / SE_QT_DATA
   - KEEP SEPARATE — do not merge these pipelines
3. **QT editor (v28.1, edit mode)**: renderEditorTable() + editorCell()/editorTimeCell()
   - Operates on a session-scoped clone (qtEditorStore), not on QT_DATA/SE_QT_DATA directly
   - Only touches live data via commitEditorStoreToLive() — see caveat below

### qtRowState — module-level, persists within session
```js
const qtRowState = { qt: {SC: true, LC: true}, rq: {SC: true, LC: true} };
```
toggleQTRows() mutates this object. (Its predecessor, qtToggleState, also drove
a chart that was removed in v27 — qtRowState is row-toggle-only.)
State is NOT reset when filters change or tabs re-render — intentional so the
user's toggle preference is preserved across filter changes within a session.
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

### View-mode prefix vs editor prefix — do not conflate
View-mode IDs use `qt`/`rq`. Editor-mode IDs use `cqt`/`rqt`
(editorPrefixFor('qt') === 'cqt'; anything else maps to 'rqt'). They are
deliberately different namespaces to avoid element-ID collisions between the
two modes living in the DOM simultaneously. When adding new editor elements,
always use the `cqt`/`rqt` prefix, never `qt`/`rq`.

### QT editor — unsaved edits lost on navigate-away (known caveat, unresolved)
qtEditorStore is committed to the live QT_DATA/SE_QT_DATA globals only when
closeQTEditor() runs (i.e. clicking "← Back"), or immediately for meta-panel
edits via saveMetaPanel(). If the user switches to a different top-level tab
(or otherwise navigates away) while edit mode is open WITHOUT clicking
"← Back" first, any row-level edits made since the store was cloned are lost —
they were never committed and the store itself is not persisted. This is a
known limitation, not yet addressed. Do not assume editor edits are safe until
"← Back" has been clicked.

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
- renderQTBanner() — cheap; always reflects current QT_META/SE_QT_META

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
Competition column in Schedule/Overview, and the meta banner's mobile/desktop
note text (.qt-banner-note-desktop/.qt-banner-note-mobile, toggled at the 769px
breakpoint used by .btn-edit-qt). Add to any new table column or banner text
that would overflow or mismatch on mobile.

### escapeHtml() — required for all user-supplied strings in innerHTML
competition, venue from RAW. Event and course are dropdown-constrained.
Also applied to filename/message in showJsonLoadError() (v28).

### unwrapQT() — must be the only place that branches on QT schema shape
Any new code that reads county_qt.json/regional_qt.json should go through
QT_DATA/SE_QT_DATA (already-flat arrays) or QT_META/SE_QT_META — never
re-parse or re-branch on whether the source was wrapped or bare-array. Keep
that logic centralised in unwrapQT().

### UPCOMING data authority
swimDash_UPCOMING is GitHub-authoritative on first load, also mutated locally.
User must download and commit to persist.

### removeUpcomingEvent() — event property format
UPCOMING[i].events is array of { event: string }. Filter uses `e.event !== event`.

### Logo — GitHub raw URL
Loaded via GitHub raw URL. Hidden gracefully offline via onerror.

### regional_qt.json — do not reintroduce the old filename
SE_QT_DATA_URL must point at regional_qt.json. If a future PR adds a new fetch
or upload reference to se_london_qt.json, that is a regression of the v28 rename.
