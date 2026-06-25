# Known Bugs, Past Fixes & Watch-out Areas

## RESOLVED — do not re-introduce

### v29.2 — Add Row creates immediately-unusable "not offered" rows (bug fix)
`addEditorRow()` pushed `{qualify: null, consider: null}` which triggered the
`notOffered` guard and greyed the row with disabled inputs. Fixed: new rows
carry `_new: true`. The `notOffered` check is `!t._new && qualify===null && consider===null`.
`_new` is stripped by `commitEditorStoreToLive()` and `downloadEditorJSON()`.

### v29.2 — "Not offered" rows had disabled time inputs despite instructing user to enter a time
Note text said "leave blank, or enter a time to make it available" but the
inputs had `pointer-events: none` and `disabled` — impossible to follow.
Fixed: only metadata columns (Gender, Course, Event, Age) are `disabled` for
not-offered rows. Time inputs always enabled. CSS changed from
`.not-offered select, .not-offered input { pointer-events:none }` to
`.not-offered select, .not-offered input:not(.time-input) { pointer-events:none }`.
Note text updated to "Not currently offered — enter a time above to activate".

### v29.2 — Add Row accessible with no Age Group selected
Rows added with `age: ''` never appeared in any age-filtered view. Fixed:
`renderEditorTable()` updates `#cqtAddRowBtn` / `#rqtAddRowBtn` disabled state,
opacity (0.4), and cursor (not-allowed) before its early-return guard, so the
button is always grey when no age is selected and active when one is.

### v29.2 — Refresh alert said "Reloading…" as if automatic
`alert('… Reloading…')` followed immediately by `window.location.reload()` —
the reload only happens after the user clicks OK, so "Reloading…" was misleading.
Changed to "Click OK to reload." in both the success and partial-success paths.

### v29.1 — QT editor "Edit Details & Times" button visible while in edit mode
Clicking the button while already in edit mode would call `openQTEditor()` again.
Fixed: `openQTEditor()` sets `editBtn.style.display = 'none'`;
`closeQTEditor()` restores it with `editBtn.style.display = ''`.

### v29.1 — QT editor navigate-away discarded unsaved row edits silently
Switching tabs while the editor was open with unsaved changes discarded the
editor store without warning. Two guards added:
- `showTab()` checks `qtEditorDirty[prefix]` before switching away; prompts
  "Leave without saving?" — confirming discards, cancelling stays in editor.
- `beforeunload` listener in `init()` fires the browser's native leave-site
  prompt if either editor is open with unsaved row changes.

### v29.1 — No way to save QT editor row changes without closing editor
Only "← Back" committed changes. Fixed: `saveEditorChanges(prefix)` commits
immediately without closing, clears dirty flag, and flashes "✅ Saved" on the
💾 Save Changes button for 1.8 seconds.

### v29.1 — No way to refresh GitHub data without clearing localStorage manually
Fixed: `refreshFromGitHub()` re-fetches `county_qt.json`, `regional_qt.json`,
and `upcoming_races.json` (not `my_swims.json`) with cache-busting. Stores a
`swimDash_lastRefresh` ISO timestamp. Last-refreshed date shown in Data Manager.

### splits[] are lap times not cumulative (v6)
### null.closest() crash in analyzeSwim (v11)
### progChart canvas gone after debut callout (v12)
### Progression ghost chart after debut (v13)
### paceChart null crash (v16)
### Backdated race broke Progression lookups (v17)
### Splits compared vs course PB not previous PB (v18)
### Backdated PB showed dash in Improvement Summary (v18)
### Results Debut badge missing for first swim of multi-swim events (v19)
### Data Killer — init() overwrote localStorage (v20)
### Split parsing with parseFloat (v20)
### finishDiff zero for 50m races (v20)
### strokeOrder indexOf=-1 (v20)
### "Latest Swim" mode showed Debut when PB is most recent swim (v20.1)
### AI Coach tagged historical PBs as Current PB (v20.2)
### resetResultsFilters sorted oldest-first after reset (v20.2)
### Overview "PBs this year" / "Recent PBs" used isPB not getPBs() (v21)
### processData sort unstable for same-date SC/LC (v21)
### User-supplied strings unescaped in innerHTML (v21/v22)
### 100 IM missing from ALL_EVENTS (v24.1)
### Four stacked FABs cluttering screen (v25)
### Schedule table colspan mismatch (v26)
### All items from v26.2 quality pass (see session-log.md)
### getBestSeasonImprovement() false positive (v26.3)
### Ghost view on Splits tab after updateData() (v26.3)
### Incomplete tab guarding (v26.3)

### v27 QT tab — Course filter removed; dual-course unified view
### v27 QT tab — buildQtStatCards() old single-arg signature removed
### v27 QT tab — Chart removed (SC/LC bar overlap)
### v27 QT tab — Status chip removed; left border colour conveys status
### v27 QT tab — Mobile table scaling lost
### v27 — Dead CSS after chart removal (B1/I5)
### v27 — renewalMonths resets on reload (I2)
### v27 — Progress bar track invisible when no PB (I3)
### v27 — qtToggleState renamed to qtRowState (I6)
### v27 — removeRace() silent on duplicate entries (P3)

### v28 — Silent failure on malformed JSON data files
### v28 — se_london_qt.json renamed to regional_qt.json
### v28 — QT JSON schema migrated to {meta, times} wrapper

### v28.1 — Inline QT editor replaces standalone-tab approach
### v28.1 — QT editor UX fixes (six items, see session-log.md)
### v28.1 — Banner copy consistency fix

---

## WATCH-OUT AREAS

### qtEditorDirty — must be cleared in all commit/discard paths
```js
const qtEditorDirty = { cqt: false, rqt: false };
```
Set to `true` by: `editorCell`, `editorTimeCell`, `addEditorRow`, `deleteEditorRow`.
Cleared by: `saveEditorChanges()`, `closeQTEditor()`, `openQTEditor()` (on open),
and the discard path in `showTab()`.
**NOT** set by `saveMetaPanel()` — meta edits commit immediately to live globals.
If you add a new row-mutation function, it must also set `qtEditorDirty[prefix] = true`.

### _new flag on editor rows — strip before persisting
Rows added via `addEditorRow()` carry `_new: true` to exempt them from the
`notOffered` guard while their time fields are still null. This is an internal
UI marker only. Both `commitEditorStoreToLive()` and `downloadEditorJSON()`
strip it via `store.times.map(({_new,...rest})=>rest)`. If you add any other
path that reads from or serialises `qtEditorStore`, strip `_new` there too.

### notOffered check — three conditions, not two
```js
const notOffered = !t._new && (t.qualify===null||t.qualify===undefined)
                            && (t.consider===null||t.consider===undefined);
```
All three conditions must hold. A freshly added row (`_new: true`) with null
times is NOT treated as not-offered. An existing row with empty strings is NOT
treated as not-offered (empty strings are not null). Only rows from the original
dataset with both times genuinely absent are greyed.

### "Not offered" rows — metadata disabled, time inputs enabled
In `renderEditorTable`, not-offered rows use `metaDisabled` (applied to gender/
course/event/age selects and the age text input) but NOT to the two time inputs.
The CSS `.qt-editor-tbl tr.not-offered input:not(.time-input)` targets only
non-time inputs. Do not change this back to targeting all inputs — the whole
point is that users can enter a time to "activate" a not-offered row.

### Add Row button state — updated before early return in renderEditorTable
The `addRowBtn.disabled = !age` update runs before the `if (!age||!store) return`
guard, so the button is always greyed when no age is selected even though the
rest of the function exits early. If you add any new early-return path to
`renderEditorTable`, ensure the button-state update already ran.

### refreshFromGitHub — does NOT refresh my_swims.json
`my_swims.json` is user-authoritative (swimDash_RAW). The refresh function
only touches `swimDash_QT`, `swimDash_SE_QT`, and `swimDash_UPCOMING`.
If you add a new GitHub-authoritative file, add it to the `targets` array
in `refreshFromGitHub()`.

### swimDash_lastRefresh — display only, not used for staleness checks
The timestamp is shown in the Data Manager modal as informational text. It does
not drive any "data is stale" logic and does not affect fetching behaviour.
Do not use it for conditional fetch decisions — the existing "fetch if not in
localStorage" pattern in `init()` is the authority on whether to fetch.

### isPB vs getPBs().includes() — use the right one for the right purpose
| Check                    | True when                              | Use for                           |
|--------------------------|----------------------------------------|-----------------------------------|
| r.isPB                   | Swim was fastest at time of swimming   | Historical "was this a PB?" check |
| getPBs().includes(swim)  | Swim is the current fastest ever       | "Is this the PB right now?"       |

### QT tab — three separate rendering pipelines
1. **QT tab view (cards/table)**: `buildQtStatCards` + `buildProgressBar` +
   `applyStatusFilter` + `renderQTCards` — both courses simultaneously.
2. **Schedule + Targets**: `getQTStatusForEvent` + `renderQTCells` — single
   course at a time. KEEP SEPARATE.
3. **QT editor**: `renderEditorTable` + `editorCell`/`editorTimeCell` etc. —
   operates on `qtEditorStore`, not `QT_DATA`/`SE_QT_DATA` directly.

### qtRowState — module-level, persists within session, row-toggle only
```js
const qtRowState = { qt: {SC: true, LC: true}, rq: {SC: true, LC: true} };
```
Chart was removed in v27. `qtRowState` drives SC/LC row-toggle buttons only.
Resets to all-true on page reload.

### renderedTabs Set — three usage patterns
- `renderedTabs.clear()` — full reset (invalidateCache, renderAllChartTabs)
- `renderedTabs.delete('tabname')` — targeted reset (forceX wrappers, sort/reset fns)
- `renderedTabs.has('tabname')` — guard at top of each render function

### forceX() wrappers — required for all filter onchange handlers
All filter `onchange` attributes must call `forceX()` wrappers, not render
functions directly. Current wrappers: `forceProgression`, `forcePBs`,
`forceResults`, `forceQualifying`, `forceRegional`.

### View-mode prefix vs editor prefix — do not conflate
View-mode IDs use `qt`/`rq`. Editor-mode IDs use `cqt`/`rqt`.
`editorPrefixFor('qt') === 'cqt'`. They are deliberately separate namespaces.

### QT editor — unsaved row edits lost on navigate-away after confirm
If the user confirms "Leave without saving?" in the `showTab()` guard,
the editor store is discarded (not committed). There is no "restore unsaved
changes" facility. This is intentional — the guard gives the user the choice.

### ALL_EVENTS constant — single source of truth
Declared in CONFIGURATION section. 18 events. Do NOT add inline event arrays.

### getCC() and getPalette() — inside render functions only
Always declare at top of render function: `const CC=getCC(); const PALETTE=getPalette();`

### regional_qt.json — do not reintroduce the old filename
`SE_QT_DATA_URL` must point at `regional_qt.json`. Internal variable names
(`SE_QT_DATA`, `SE_QT_META`, `renderRegionalQualifying`) are intentionally
unchanged — only the file and labels changed in v28.

### unwrapQT() — single normalisation point for QT schema
Any code reading `county_qt.json`/`regional_qt.json` must go through
`QT_DATA`/`SE_QT_DATA` (already-flat arrays) or `QT_META`/`SE_QT_META`.
Never re-branch on wrapped vs bare-array format outside `unwrapQT()`.

### escapeHtml() — required for all user-supplied strings in innerHTML
`competition`, `venue` from RAW. Event and course are dropdown-constrained.
Also applied to `filename`/`message` in `showJsonLoadError()`.
