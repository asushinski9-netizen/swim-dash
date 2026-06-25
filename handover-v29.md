# Handover Document — Swimming Performance Dashboard v29.2

**Date:** June 2026  
**Versions covered:** v29.1 and v29.2  
**Base:** v28.1 (inline QT editor)

---

## What was built in this session

### v29.1 — Data Manager refresh + QT editor save flow
Four features added to the existing v28.1 codebase:

1. **🔄 Refresh Data from GitHub** button in Data Manager  
   Re-fetches `county_qt.json`, `regional_qt.json`, `upcoming_races.json` with cache-busting. Never touches `my_swims.json`. Stores a `swimDash_lastRefresh` timestamp shown in the modal. Alerts "Click OK to reload." after success.

2. **Edit button hides in edit mode**  
   `openQTEditor()` sets `editBtn.style.display = 'none'`; `closeQTEditor()` restores it. Prevents double-entry into the editor.

3. **💾 Save Changes + dirty tracking**  
   `qtEditorDirty { cqt, rqt }` tracks whether the editor store has uncommitted row-level changes. All four mutation functions mark dirty. New Save Changes button commits without closing. "← Back" commits and closes (unchanged). Both clear the dirty flag.

4. **Navigate-away protection**  
   `showTab()` prompts before leaving a dirty editor (discard/cancel). `beforeunload` listener fires the browser's native leave-site prompt for page closes or navigation.

### v29.2 — QT editor bug-fixes and UX polish
Four issues found during review of v29.1:

1. **Alert wording** — "Reloading…" changed to "Click OK to reload." (the reload is user-triggered by clicking OK, not automatic).

2. **New rows immediately unusable** — `addEditorRow()` set `qualify: null, consider: null`, triggering the `notOffered` guard. Fixed with `_new: true` flag that exempts new rows from the guard. Stripped before committing or downloading.

3. **"Not offered" rows had disabled time inputs** — The note said "enter a time to activate" but inputs were `disabled`. Fixed: only metadata columns (Gender/Course/Event/Age) are disabled; time inputs always remain active. CSS narrowed to `input:not(.time-input)`.

4. **Add Row active without Age Group selected** — Rows added with empty `age` were invisible in all filtered views. Fixed: `renderEditorTable()` updates the Add Row button's disabled state before its early-return guard.

---

## Current state of the project

### What works well
- All 9 tabs render correctly with the full dataset
- County QT and Regional QT tabs show dual SC+LC status accurately
- QT inline editor (desktop): full CRUD on qualifying times, meta panel, download, save-without-close, navigate-away protection
- Data Manager: upload, download, refresh from GitHub, reset
- Race entry, upcoming race management, schedule, targets — all stable
- Dark/light theme, mobile responsiveness, lazy rendering

### Known remaining limitations
- QT editor is desktop-only (hidden below 769px). No mobile editing path.
- Editor row edits are discarded (not auto-saved) if the user confirms "Leave without saving?" — there is no undo/restore.
- `swimDash_lastRefresh` is display-only; it does not drive any staleness or conditional-fetch logic.
- Safari mobile has reduced chart interactivity (Chart.js canvas rendering limitation).

---

## File inventory

| File | Status |
|------|--------|
| `index.html` | ✅ v29.2 — primary deliverable |
| `my_swims.json` | Unchanged — swimmer's race data |
| `county_qt.json` | Unchanged — Middlesex County QT data |
| `regional_qt.json` (formerly `se_london_qt.json`) | Unchanged — SE London Regional QT data |
| `upcoming_races.json` | Unchanged — scheduled races |
| `README.md` | ✅ Updated — v29.2, Refresh section, editor UX notes |
| `project-brief.md` | ✅ Updated — v29.x features documented |
| `architecture.md` | ✅ Updated — all new functions, QT editor v29.x changes |
| `session-log.md` | ✅ Updated — v29.1 and v29.2 entries added |
| `known-bugs-and-fixes.md` | ✅ Updated — 7 new resolved items, 6 new watch-out entries |
| `data-schema.md` | No change needed — schemas unchanged in v29.x |

---

## Key architectural decisions made in v29.x

### Why `_new` flag rather than empty strings for new rows
Using `qualify: ''` instead of `null` would avoid the `notOffered` check but
would require changes throughout the time conversion pipeline (`timeInputToSec`,
`secToTimeInput`, QT rendering). The `_new: true` flag is purely internal — it
touches only `renderEditorTable`, `commitEditorStoreToLive`, and `downloadEditorJSON` —
with zero impact on any rendering logic outside the editor.

### Why discard (not commit) on navigate-away confirm
Committing on discard would be surprising ("I said leave without saving but my
changes were saved anyway"). The user explicitly chose not to save. The editor
store simply stops being referenced; the next `openQTEditor()` call will
deep-clone from the live globals again.

### Why `beforeunload` + `showTab()` guard are independent
`showTab()` handles in-app navigation (the common case). `beforeunload` handles
browser-level navigation (address bar, close tab, browser back). Each fires in
different circumstances and they don't interfere with each other.

### Why refreshFromGitHub does not touch my_swims.json
`my_swims.json` is user-authoritative — the swimmer adds races locally and
periodically pushes the file to GitHub. Overwriting it on refresh could silently
destroy locally added races that haven't been committed yet. The three
GitHub-authoritative files (QT data, upcoming) are safe to overwrite because
they are managed centrally and any local edits are downloaded explicitly before
committing.

---

## Suggested next features

These were not in scope for v29.x but would be natural extensions:

### Short-term
- **Season selector on Targets tab** — currently always uses the current season; allow viewing previous seasons
- **QT editor mobile fallback** — a simplified read-only view below 769px that links to desktop for editing
- **Bulk-import upcoming races** — paste a CSV or paste from a club fixture list
- **Export results as CSV** — alongside the existing JSON download

### Medium-term
- **Multi-swimmer support** — switch between swimmer profiles (useful for coaching)
- **Club aggregate view** — compare PBs across multiple swimmers (requires server-side storage)
- **Relay splits** — current split model assumes individual events only
- **Meet comparison** — overlay two meets side by side on the Progression chart

### Infrastructure
- **GitHub Actions auto-deploy** — currently manual push; a workflow to auto-deploy on push to `main` would simplify the update cycle
- **Automated QT data updates** — scrape official British Swimming standards and update `county_qt.json` / `regional_qt.json` automatically each season

---

## How to pick up from here

1. **Load the dashboard** — open `index.html` locally or visit the GitHub Pages URL
2. **Upload data files** — via ⚙️ Data Manager if starting fresh on a new device
3. **Check the age bracket** — 👤 Swimmer Settings → confirm DOB and gender; the QT tabs will auto-select the right age group
4. **Update championship dates** — in `COUNTY_CHAMPS_DATE` and `REGIONAL_CHAMPS_DATE` constants in the JS configuration section when the next season's dates are known
5. **Update QT data** — use ✏️ Edit Details & Times in the County/Regional QT tabs or upload new JSON files via Data Manager, then download and commit the updated JSON to GitHub

---

## Handover checklist

- [x] `index.html` v29.2 delivered
- [x] `README.md` updated
- [x] `project-brief.md` updated  
- [x] `architecture.md` updated
- [x] `session-log.md` updated (v29.1 + v29.2 entries)
- [x] `known-bugs-and-fixes.md` updated (7 resolved + 6 watch-outs)
- [x] `data-schema.md` — no changes needed
- [x] All v29.2 verification checks passed (23/23)
- [x] Emoji fix: Regional QT tab 🎍 → 🏅 (&#127885; → &#127941;)
