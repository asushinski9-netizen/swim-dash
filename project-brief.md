# Swimming Performance Dashboard — Project Brief

## Purpose
Single-file HTML dashboard for tracking a competitive swimmer's race history,
personal bests, split analysis, qualifying status vs county and regional standards,
upcoming race schedule, and performance targets.

## Swimmer
Broomfield Park Swimming Club (BPSC), membership 1745801.
Dashboard: BPSC royal blue (#1a56c4), club logo loaded from GitHub (bpsc_logo.png).
DOB and gender configurable via 👤 Swimmer Settings (stored in localStorage).

## Tech stack
Single-file HTML5 | Chart.js 4.4.0 (jsdelivr CDN) + chartjs-adapter-date-fns@3.0.0
Vanilla JS ES6+ | CSS custom properties (light/dark theme) | No frameworks / build step

## Data sources
| File                | Authority    | Notes                                    |
|---------------------|--------------|------------------------------------------|
| my_swims.json       | localStorage | User-editable; never overwritten         |
| county_qt.json      | GitHub       | Cached locally; re-fetched if missing.   |
|                     |              | `{meta,times}` schema since v28; editable via inline editor |
| regional_qt.json    | GitHub       | Cached locally; re-fetched if missing.   |
|                     |              | Renamed from se_london_qt.json in v28; same schema as county |
| upcoming_races.json | GitHub       | Cached locally; re-fetched if missing.   |
|                     |              | Also mutated locally by 📅 Add Upcoming  |
|                     |              | and ✕ Remove. Download to persist.       |

GitHub-authoritative files (all except my_swims.json) can also be force-refreshed
via ⚙️ Data Manager → 🔄 Refresh Data from GitHub without clearing user race data.

A malformed JSON file in any of the above surfaces a visible warning banner
on the Overview tab (rather than failing silently) — see "Error handling" below.

## Tabs (v29.2)
1. **Overview** — 6 clickable stat cards, recent PBs, frequency chart, events breakdown,
   venues; also hosts the JSON load error banner if any data file fails to parse
2. **Progression** — Improvement Summary + BvF table, Time Progression chart
3. **Personal Bests** — PB cards with improvement vs configurable reference
4. **Splits** — AI coach, split comparison, pacing profile
5. **All Results** — sortable/filterable; Remove Race per row
6. **County QT** — Middlesex qualifying; meta banner with championship title/dates;
   dual SC+LC event cards with progress bars; status filter; clickable stat cards;
   event-by-event table with SC/LC row toggles; bracket note; desktop-only inline
   editor for times and championship details
7. **Regional QT** — Regional qualifying; same layout and editor as County tab
8. **Schedule** — Upcoming race calendar; County + Regional QT status per event; Del per row
9. **Targets** — PBs Due for Renewal + Events Not Yet Swum

## Speed Dial FAB (5 children)
Single ➕ parent button (bottom-right). Tap to expand 5 child actions upward:
  1. ⏱️ Add Race            (closest to thumb)
  2. 📅 Add Upcoming Race
  3. ⚙️ Data Manager
  4. 🌙/☀️ Toggle Theme
  5. 👤 Swimmer Settings    (topmost)

## County QT & Regional QT — design (v27 view mode, v28.1/v29.x editor)

### Meta banner
Sits above both QT tabs at all times, showing the championship title and date
range (sourced from QT_META/SE_QT_META). Includes a desktop-only "✏️ Edit
Details & Times" button. The button is hidden while edit mode is active (v29.1)
and restored when the editor is closed. If no dates are set, shows a responsive
hint pointing the user at the button (desktop) or telling them to switch to
desktop (mobile).

### Stat cards (top, view mode)
Four clickable cards — Qualified / Consideration / Outside / No PB.
Status based on best across SC+LC per event.
Each event listed with: course badge (SC/LC), best PB time, gap chip.
Clicking a card applies the Status filter and re-renders.

### Qualification status cards (view mode)
4-column grid (desktop) / 1-column (mobile). One card per event.
- SC block (lighter): course badge, PB, gap chip, progress bar
- LC block (darker): same; "No PB" badge if not swum
- Card border = best status; Course block left border = that course's status
- Progress bar: per-event scaled, always shown, CT/QT time labels above markers
- Gap shown once only in stats row above bar

### Filters (view mode)
Gender · Age Group · Stroke · Status

### Inline editor (v28.1 + v29.x)
Desktop-only (hidden below 769px). Toggled via the meta banner button.

**v29.1 additions:**
- Edit button hidden while editor is open; restored on close
- 💾 Save Changes button: commits store to live globals without closing; clears dirty flag; flashes "✅ Saved"
- `qtEditorDirty { cqt, rqt }` tracks uncommitted row-level changes
- Navigate-away guard in `showTab()`: confirm dialog before leaving a dirty editor
- `beforeunload` listener: browser-native prompt if page is closed/navigated with unsaved changes

**v29.2 additions:**
- New rows (`addEditorRow`) start with `_new: true`, exempting them from the `notOffered` guard
- `_new` stripped by `commitEditorStoreToLive()` and `downloadEditorJSON()` before persisting
- Not-offered rows: only metadata columns disabled; time inputs remain active
- Note text: "Not currently offered — enter a time above to activate"
- Add Row button disabled (greyed, `cursor: not-allowed`) until Age Group is selected

## Swimmer Profile Configuration
Set via 👤 → Settings modal:
- **Date of Birth** (swimDash_DOB): drives auto age-bracket selection
- **Gender** (swimDash_GENDER): "Boys" or "Girls"

Championship dates (JS configuration section):
- COUNTY_CHAMPS_DATE: last day of Middlesex County Championships
- REGIONAL_CHAMPS_DATE: last day of Regional Championships

Season boundary:
- SEASON_START_MONTH = 9 (September, 1-indexed)

## Error handling (v28)
Any of the four JSON data files failing to parse surfaces a persistent, stackable
warning banner at the top of the Overview tab naming the file and the parse
error, with a hint to check for syntax errors and re-upload. The dashboard
continues to function using whatever data is available rather than blocking.

## Persist to GitHub
- Race data: Download my_swims.json → push to GitHub repo
- Upcoming: Download upcoming_races.json → push to GitHub repo
- Qualifying times: Download county_qt.json / regional_qt.json from the inline
  editor (⬇ Download JSON) → push to GitHub repo
- Or use 🔄 Refresh Data from GitHub to pull the latest versions from GitHub
  into localStorage without clearing race data

## Deployment
Local HTML file (file://) or GitHub Pages. Works offline after first load.
Mobile responsive — @media (max-width: 500px) for general layout;
@media (min-width: 769px) gates the QT inline editor to desktop only.
