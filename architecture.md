# Code Architecture

## File
Single HTML file: index.html
Current version: v29.2
~2,345 lines total

## Script section layout (in order)
1.  CONFIGURATION — RACE_DATA_URL, QT_DATA_URL, SE_QT_DATA_URL (regional_qt.json),
                    UPCOMING_DATA_URL, COUNTY_CHAMPS_DATE, REGIONAL_CHAMPS_DATE,
                    SEASON_START_MONTH, ALL_EVENTS (canonical 18-event list)
2.  Swimmer profile functions — getSwimmerDOB(), getSwimmerGender()
3.  DATA globals — RAW, QT_DATA, SE_QT_DATA (flat arrays), QT_META, SE_QT_META, UPCOMING, DATA
4.  unwrapQT(raw) — schema-compatibility helper
5.  STATE globals — renderedTabs (Set), sortCol, sortDir, fabDialOpen,
                    currentEventFiltered, contextualPBRef, splitBarChart, paceChart
6.  cache{} + invalidateCache()
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
15. FILTER RESET HELPERS — forceX() wrappers, resetX() functions
16. CHARTS REGISTRY — charts{}, destroyChart()
17. POPULATE SELECTS — populateSelects()
18. TAB RENDERS — renderOverview(), renderProgression(), renderPBs(),
                  renderSplits(), renderSplitCharts(), analyzeSwim(),
                  renderResults(),
                  buildQtStatCards(), buildProgressBar(), applyStatusFilter(),
                  qtRowState, toggleQTRows(),
                  renderQTCards(), renderQualifying(), renderRegionalQualifying(),
                  buildSwumUpcomingSet(), renderSchedule(), renderTargets()
19. RENDER ALL CHART TABS — renderAllChartTabs()
20. TAB SWITCHING — showTab() — includes v29.1 navigate-away guard for dirty editor
21. JSON LOAD ERROR DISPLAY — showJsonLoadError(filename, message)
22. INIT — async init() — includes keydown (Escape), beforeunload (v29.1),
           last-refresh display (v29.1), delegated click listeners
23. ADD RACE MODAL — showAddRaceModal(), hideAddRaceModal(), addRaceEventRow(), saveNewRace()
24. REMOVE RACE — removeRace()
25. REMOVE UPCOMING EVENT — removeUpcomingEvent()
26. SPEED DIAL FAB — toggleFabDial(), openFabDial(), closeFabDial()
27. SETTINGS MODAL — showSettingsModal(), hideSettingsModal(), saveSettings()
28. ADD UPCOMING RACE MODAL — showAddUpcomingModal(), hideAddUpcomingModal(),
                               addUpcomingEventRow(), saveNewUpcoming()
29. QT EDITOR (~400 lines) — see QT Editor section below
30. REFRESH FROM GITHUB — refreshFromGitHub() (v29.1)
31. SAVE EDITOR CHANGES — saveEditorChanges(prefix) (v29.1)
32. Theme IIFE + DOMContentLoaded → init()

---

## New functions in v29.x

### refreshFromGitHub() — async, v29.1
Re-fetches `county_qt.json`, `regional_qt.json`, `upcoming_races.json` from
GitHub with a `?_=Date.now()` cache-busting suffix. Stores results to
`swimDash_QT`, `swimDash_SE_QT`, `swimDash_UPCOMING`. Does NOT touch
`swimDash_RAW` (user-authoritative). On completion:
- Full success → `alert('Click OK to reload.')` → `window.location.reload()`
- Partial success → alert listing successes and failures → reload if any succeeded
- Network error → alert with error message, button re-enabled, no reload
Stores `swimDash_lastRefresh` (ISO string) whenever at least one file is updated.
Displayed in Data Manager modal on next open via `#dataModalLastRefresh`.

### saveEditorChanges(prefix) — v29.1
Commits `qtEditorStore[prefix]` to live globals (`QT_DATA`/`SE_QT_DATA`/
`QT_META`/`SE_QT_META`) immediately without closing the editor. Clears
`qtEditorDirty[prefix]`. Invalidates the corresponding view tab in `renderedTabs`.
Refreshes the meta banner. Shows a 1.8-second green "✅ Saved" flash on
`#cqtSaveBtn` / `#rqtSaveBtn`.

---

## QT Editor Architecture (v28.1 + v29.x)

### State
```js
const qtEditorStore = { cqt: null, rqt: null };  // session-scoped working copies
const qtEditorDirty = { cqt: false, rqt: false }; // v29.1: uncommitted row-level changes
```

`qtEditorDirty` is set to `true` by any row-level mutation:
`editorCell`, `editorTimeCell`, `addEditorRow`, `deleteEditorRow`.
It is cleared by: `openQTEditor()` (on first open), `saveEditorChanges()`,
`closeQTEditor()`, and the discard path in `showTab()`.
Meta-panel edits (`saveMetaPanel`) commit immediately to live globals and do
NOT set the dirty flag.

### View/edit toggle (CSS-driven, unchanged from v28.1)
```css
.qt-view-mode  { display: block; }
.qt-edit-mode  { display: none;  }
.qt-tab-in-edit-mode .qt-view-mode { display: none; }
.qt-tab-in-edit-mode .qt-edit-mode { display: block; }
```

### openQTEditor(viewPrefix) — v29.1 additions
- Adds `.qt-tab-in-edit-mode` to the tab panel (unchanged)
- **NEW**: finds `.btn-edit-qt` inside the tab panel, sets `display: none`
- Deep-clones source data into `qtEditorStore` on first call per session (unchanged)
- **NEW**: clears `qtEditorDirty[prefix] = false`

### closeQTEditor(viewPrefix) — v29.1 additions
- Removes `.qt-tab-in-edit-mode` (unchanged)
- **NEW**: finds `.btn-edit-qt`, restores `display: ''`
- Calls `commitEditorStoreToLive(prefix)` (unchanged)
- **NEW**: clears `qtEditorDirty[prefix] = false`
- Refreshes banner and re-renders view tab (unchanged)

### commitEditorStoreToLive(prefix) — v29.2 addition
```js
// Strips _new markers before writing to live globals
const cleanTimes = store.times.map(({_new, ...rest}) => rest);
```

### Navigate-away protection — v29.1

**showTab() guard:**
```js
if (qtPanel.classList.contains('qt-tab-in-edit-mode') && qtEditorDirty['cqt'] && name !== 'qualifying') {
  if (!confirm('You have unsaved changes in the County QT editor.\n\nLeave without saving?')) return;
  // discard: remove edit-mode class, restore edit button, clear dirty flag
}
```
Confirming discards (does NOT commit). Cancelling keeps the editor open.
Same pattern for Regional (`rqt`).

**beforeunload listener (in init()):**
```js
window.addEventListener('beforeunload', e => {
  const qtDirty = panel.classList.contains('qt-tab-in-edit-mode') && qtEditorDirty['cqt'];
  const rqDirty = panel.classList.contains('qt-tab-in-edit-mode') && qtEditorDirty['rqt'];
  if (qtDirty || rqDirty) { e.preventDefault(); e.returnValue = ''; }
});
```

### Not-offered rows — v29.2 changes

**notOffered check (in renderEditorTable):**
```js
// _new rows are never not-offered, regardless of null times
const notOffered = !t._new && (t.qualify===null||t.qualify===undefined)
                            && (t.consider===null||t.consider===undefined);
```

**Row rendering — split disabled state:**
```js
const metaDisabled = notOffered ? ' disabled' : '';
// Applied to: gender select, course select, event select, age text input
// NOT applied to: qualify time input, consider time input, delete button
```

**CSS:**
```css
/* Only non-time inputs get pointer-events:none in not-offered rows */
.qt-editor-tbl tr.not-offered select,
.qt-editor-tbl tr.not-offered input:not(.time-input) { pointer-events:none; background:var(--surface3); }
/* Time inputs get grey bg only — pointer events remain active */
.qt-editor-tbl tr.not-offered .time-input { background: var(--surface3); }
/* Delete button re-enabled (was pointer-events:none; opacity:0.2) */
.qt-editor-tbl tr.not-offered .btn-row-del { opacity: 0.5; }
```

**Note text:** "Not currently offered — enter a time above to activate"

### _new flag lifecycle (v29.2)
1. `addEditorRow()` pushes `{..., qualify:null, consider:null, _new:true}`
2. `renderEditorTable()` checks `!t._new` before marking `notOffered`
3. When user enters a time, `editorTimeCell()` sets `row[field] = timeInputToSec(raw)` —
   qualify/consider are no longer null, so `notOffered` becomes false on next render
4. `commitEditorStoreToLive()` strips `_new` via `map(({_new,...rest})=>rest)` before writing
5. `downloadEditorJSON()` strips `_new` similarly before serialising

### Add Row button state (v29.2)
`renderEditorTable()` updates `#cqtAddRowBtn` / `#rqtAddRowBtn` before its
early-return guard so the button state is always correct even when no age is selected:
```js
const addRowBtnId = prefix==='cqt' ? 'cqtAddRowBtn' : 'rqtAddRowBtn';
const addRowBtn = document.getElementById(addRowBtnId);
if (addRowBtn) {
  addRowBtn.disabled = !age;
  addRowBtn.style.opacity = age ? '' : '0.4';
  addRowBtn.style.cursor = age ? '' : 'not-allowed';
}
```

### downloadEditorJSON(prefix) — v29.2 addition
```js
// Strip _new before serialising
const cleanStore = {...store, times: store.times.map(({_new,...rest})=>rest)};
const json = JSON.stringify(cleanStore, null, 2);
```

---

## Data Manager — refreshFromGitHub (v29.1)

### HTML additions
```html
<!-- Between download buttons and Reset -->
<button id="btnRefreshGitHub" onclick="refreshFromGitHub()">🔄 Refresh Data from GitHub</button>
<div id="dataModalLastRefresh"></div>  <!-- populated in init() from swimDash_lastRefresh -->
```

### init() additions (v29.1)
```js
// beforeunload guard
window.addEventListener('beforeunload', e => { ... });

// Last-refresh display
const lastRefresh = localStorage.getItem('swimDash_lastRefresh');
const refreshNote = document.getElementById('dataModalLastRefresh');
if (refreshNote && lastRefresh) {
  refreshNote.textContent = `Last refreshed from GitHub: ${d.toLocaleString(...)}`;
}
```

---

## Lazy rendering system (unchanged from v28.1)

### renderedTabs Set
| Operation | When |
|-----------|------|
| `renderedTabs.clear()` | Full reset — invalidateCache(), renderAllChartTabs() |
| `renderedTabs.delete('tab')` | Targeted reset — forceX() wrappers, resetX(), sortResults() |
| `renderedTabs.has('tab')` | Guard at top of each guarded render function |
| `renderedTabs.add('tab')` | Bottom of each guarded render function |

### Guarded render functions
renderOverview, renderProgression, renderPBs, renderResults, renderQualifying,
renderRegionalQualifying.

### Intentionally unguarded
renderSplits, renderSchedule, renderTargets, renderQTBanner.

### forceX() wrappers
All filter `onchange` attributes call force wrappers, not render functions directly.

---

## QT schema (v28, unchanged)

### unwrapQT(raw)
Accepts old bare-array or new `{meta, times}` format. Always returns `{meta, times}`.
`QT_DATA`/`SE_QT_DATA` are always flat arrays after unwrapping. All consumers
downstream read from these flat arrays, not from the raw file format.

### Backward compatibility
Old bare-array `county_qt.json` / `regional_qt.json` files still load correctly.

---

## localStorage keys
```
swimDash_RAW           — USER AUTHORITATIVE (never overwritten by GitHub)
swimDash_QT            — GitHub authoritative; {meta,times} JSON text
swimDash_SE_QT         — GitHub authoritative; same as above
swimDash_UPCOMING      — GitHub authoritative on first load; also mutated locally
swimDash_theme         — 'light' or 'dark'
swimDash_DOB           — Swimmer date of birth (YYYY-MM-DD)
swimDash_GENDER        — 'Boys' or 'Girls'
swimDash_renewalMonths — '3','6','9','12'
swimDash_lastRefresh   — ISO timestamp of last successful refreshFromGitHub() call (v29.1)
```

---

## GitHub URLs
```
RACE_DATA_URL:      .../asushinski9-netizen/swim-dash/main/my_swims.json
QT_DATA_URL:        .../asushinski9-netizen/swim-dash/main/county_qt.json
SE_QT_DATA_URL:     .../asushinski9-netizen/swim-dash/main/regional_qt.json
UPCOMING_DATA_URL:  .../asushinski9-netizen/swim-dash/main/upcoming_races.json
```

---

## Speed Dial FAB (5 children, unchanged from v26)
Order bottom→top: ⏱️ Add Race · 📅 Add Upcoming · ⚙️ Data Manager · 🌙 Theme · 👤 Settings

## Swimmer profile
getSwimmerDOB() and getSwimmerGender() read from localStorage.
Defaults: '2014-10-12' / 'Boys'. Set via 👤 Settings modal.

## ALL_EVENTS constant — single source of truth
18 events in CONFIGURATION section. Used in addRaceEventRow(), addUpcomingEventRow(),
renderTargets(). EDITOR_EVENTS in the QT editor section is a subset (17, excludes 100 IM
which is not a QT event). Do NOT add inline event arrays anywhere else.

## processData() sort order
Stable: date ASC → event name ASC → course ASC (S before L).

## getBestSeasonImprovement() — within-season only
Uses bestInSeason (fastest within current season) not all-time PB.
Season boundary: getSeasonStart() → SEASON_START_MONTH (default 9 = September).
Uses local date components to avoid UTC offset bug (not toISOString()).

## getCC() and getPalette() — inside render functions only
Never at module level. Always declare at top: `const CC=getCC(); const PALETTE=getPalette();`
