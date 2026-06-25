# Session Log

## v29.2 (latest)
QT editor bug-fixes and UX polish following v29.1 review.

### Fixes

**Refresh alert wording**
"Reloading…" in the post-refresh alert implied the page was already reloading passively. Changed to "Click OK to reload." in both the success and partial-success cases — the reload is triggered by clicking OK on the alert, so this is now accurate.

**New QT rows immediately unusable (bug fix)**
`addEditorRow()` set `qualify: null, consider: null` on new rows, which immediately triggered the `notOffered` guard and greyed them out with disabled inputs. Fixed by stamping `_new: true` on newly added rows and adding `!t._new &&` to the `notOffered` check. The `_new` marker is stripped in both `commitEditorStoreToLive()` and `downloadEditorJSON()` so it never appears in live data or downloaded JSON.

**"Not offered" rows — time inputs now editable**
Previously all inputs in not-offered rows (including the qualify/consider time fields) had `pointer-events: none` and `disabled`, making the note "enter a time to make it available" impossible to follow. Fixed:
- Only the metadata columns (Gender, Course, Event, Age) get `disabled` — they define what the row represents and shouldn't be changed.
- Time inputs are always enabled. CSS updated: `pointer-events: none` now targets `input:not(.time-input)` and selects only; `.time-input` gets background styling only.
- Note text updated to "Not currently offered — enter a time above to activate".
- Delete button re-enabled for not-offered rows (was pointer-events: none; opacity: 0.2 — now just opacity: 0.5).

**Add Row disabled until Age Group selected**
The Add Row button was always active, allowing rows to be added with an empty `age` field that then never appeared in any age-filtered view. Fixed:
- Added `id="cqtAddRowBtn"` / `id="rqtAddRowBtn"` to both Add Row buttons in the HTML.
- `renderEditorTable()` now updates button `disabled`, `opacity`, and `cursor` before its early-return guard, so the state is correct both when no age is selected (button greyed) and when one is (button active).

---

## v29.1
Data Manager refresh, QT editor save flow, and navigate-away protection.

### 1 — Refresh Data from GitHub (Data Manager)
New **🔄 Refresh Data from GitHub** button added to the Data Manager modal (between the Download buttons and Reset). Calls `refreshFromGitHub()`, which:
- Re-fetches `county_qt.json`, `regional_qt.json`, and `upcoming_races.json` with a `?_=timestamp` cache-busting param.
- Updates `localStorage` for each successfully fetched file.
- Does **not** touch `swimDash_RAW` (user-authoritative).
- Stores `swimDash_lastRefresh` (ISO timestamp) on any successful fetch.
- Shows the timestamp as "Last refreshed from GitHub: …" below the buttons whenever the modal opens.
- On full success: shows success alert then reloads. On partial success: shows which files succeeded/failed, reloads only if at least one succeeded.

### 2 — Edit Details & Times button hidden in edit mode
`openQTEditor()` now finds `.btn-edit-qt` inside the tab panel and sets `display: none`. `closeQTEditor()` restores it (`display: ''`). Prevents a second `openQTEditor` call while already in edit mode.

### 3 — Save Changes button + dirty tracking
- `qtEditorDirty: { cqt: false, rqt: false }` — module-level state object, declared alongside `qtEditorStore`.
- All four row-mutation functions (`editorCell`, `editorTimeCell`, `addEditorRow`, `deleteEditorRow`) set `qtEditorDirty[prefix] = true`.
- Meta-panel edits (`saveMetaPanel`) commit immediately to live globals — they do NOT set the dirty flag.
- New **💾 Save Changes** button (`id="cqtSaveBtn"` / `id="rqtSaveBtn"`) in each editor topbar calls `saveEditorChanges(prefix)`:
  - Calls `commitEditorStoreToLive(prefix)`.
  - Clears `qtEditorDirty[prefix]`.
  - Invalidates the corresponding view tab in `renderedTabs`.
  - Shows a 1.8-second green "✅ Saved" flash on the button.
  - Editor stays open.
- "← Back" still commits + closes + clears dirty (existing behaviour, unchanged).

### 4 — Navigate-away protection
Two independent guards:
- **Tab switching** — `showTab()` checks `qtEditorDirty` before switching away from `qualifying` or `regional`. If dirty, shows `confirm("You have unsaved changes … Leave without saving?")`. Confirming discards (removes edit-mode class, restores edit button, clears dirty flag). Cancelling returns to the editor.
- **Page unload** — `init()` registers a `beforeunload` listener that sets `e.preventDefault()` / `e.returnValue = ''` if either editor is open with unsaved changes, triggering the browser's native "Leave site?" prompt.

---

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
upcoming championship banner and inline editor (see v28.1).
- New `unwrapQT(raw)` helper normalises either the old bare-array format or
  the new wrapped format to `{meta, times}` — fully backward compatible
- New `QT_META` / `SE_QT_META` globals hold the unwrapped meta object
- `QT_DATA` / `SE_QT_DATA` remain flat arrays (the unwrapped `.times`), so
  every existing consumer (`renderQualifying`, `renderRegionalQualifying`,
  `getQTStatusForEvent`, `renderTargets`) needed zero changes

---

## v27
Major redesign of County QT and Regional QT tabs. See earlier entries.

## v26.3
Addresses logical bugs and architectural issues. See earlier entries.

## v26.2 — Code Quality & Futureproofing Pass. See earlier entries.

## v26.1 / v26 / v25 / v24.x / v23 / v22 / v21 / v20–v9 — see earlier entries.
