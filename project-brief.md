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
Single-file HTML5 | Chart.js 4.4.0 (jsdelivr CDN) + chartjs-adapter-date-fns
Vanilla JS ES6+ | CSS custom properties (light/dark theme) | No frameworks / build step

## Data sources
| File                | Authority    | Notes                                   |
|---------------------|--------------|-----------------------------------------|
| my_swims.json       | localStorage | User-editable; never overwritten        |
| county_qt.json      | GitHub       | Cached locally; re-fetched if missing   |
| se_london_qt.json   | GitHub       | Cached locally; re-fetched if missing   |
| upcoming_races.json | GitHub       | Cached locally; re-fetched if missing.  |
|                     |              | Also mutated locally by 📅 Add Upcoming |
|                     |              | and ✕ Remove. Download to persist.      |

## Tabs (v26)
1. **Overview** — 6 clickable stat cards, recent PBs, frequency chart, events breakdown, venues
2. **Progression** — Improvement Summary + BvF table, Time Progression chart
3. **Personal Bests** — PB cards with improvement vs configurable reference
4. **Splits** — AI coach, split comparison, pacing profile
5. **All Results** — sortable/filterable; Remove Race per row
6. **County QT** — Middlesex qualifying; stat cards + chart + event table
7. **Regional QT** — SE London qualifying; computed age bracket default
8. **Schedule** — Upcoming race calendar; County + Regional QT status per event; Del per row
9. **Targets** — PBs Due for Renewal + Events Not Yet Swum

## Speed Dial FAB (v26: 5 children)
Single ➕ parent button (bottom-right). Tap to expand 5 child actions upward:
  1. ⏱️ Add Race            (closest to thumb)
  2. 📅 Add Upcoming Race   (NEW v26)
  3. ⚙️ Data Manager
  4. 🌙/☀️ Toggle Theme
  5. 👤 Swimmer Settings    (topmost)
Semi-transparent backdrop closes the dial on tap. Parent rotates 45° (→ ✕) when open.
Children are icon-only, centred on parent axis. closeFabDial() called at top of every action.

## Swimmer Profile Configuration
Set via 👤 → Settings modal:
- **Date of Birth** (swimDash_DOB): drives auto age-bracket selection
- **Gender** (swimDash_GENDER): "Boys" or "Girls" — matches QT JSON field

Championship dates (JS configuration section):
- COUNTY_CHAMPS_DATE: last day of Middlesex County Championships
- REGIONAL_CHAMPS_DATE: last day of SE London Regional Championships
Post-championship, next season's bracket applies automatically.

Season boundary:
- SEASON_START_MONTH = 9 (September, 1-indexed)
  Controls the season start used by getBestSeasonImprovement() and getSeasonStart().

## Add Race (v26: multi-event)
⏱️ child FAB → modal with shared meet fields + per-event rows.
Shared: Date, Venue, Course, Competition Name.
Per-event: Event dropdown, Time (mm:ss.hh or ss.hh), Splits (optional, comma-separated).
Up to 6 event rows per session. Confirmation flash on ⏱️ child.

## Add Upcoming Race (v26)
📅 child FAB → modal. Fields: Date, Competition Name, Venue, Course.
Per-event: Event dropdown, up to 6 events. ✕ to remove row.
Saves to UPCOMING / swimDash_UPCOMING. Merges into existing meet if same date+course.
Confirmation flash on 📅 child. Re-renders Schedule and Targets.

## Remove Upcoming Event (v26)
✕ button in Schedule tab → confirm → removeUpcomingEvent() → localStorage update.
Re-renders Schedule and Targets. Persists locally only — download JSON to make permanent.

## Remove Race
Delete button in All Results row → confirm → updateData(RAW).

## Overview Stat Cards (v26)
All 6 cards now clickable. Card 3 replaced: "Events" → "Best Improvement" showing the
largest time drop this season (first swim of season → current PB across all event+courses).
Displayed as "▼ Xs" in green; "—" in muted if no multi-swim improvement exists this season.

## Persist to GitHub
- Race data: Download my_swims.json from Data Manager → push to GitHub repo.
- Upcoming: Download upcoming_races.json from Data Manager → push to GitHub repo.

## Deployment
Local HTML file (file://) or GitHub Pages. Works offline after first load.
Mobile responsive — @media (max-width: 500px) breakpoint.
