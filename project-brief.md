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
| upcoming_races.json | GitHub       | Cached locally; re-fetched if missing   |

## Tabs (v25)
1. **Overview** — 6 stat cards, recent PBs, frequency chart, events breakdown, venues
2. **Progression** — Improvement Summary + BvF table, Time Progression chart
3. **Personal Bests** — PB cards with improvement vs configurable reference
4. **Splits** — AI coach, split comparison, pacing profile
5. **All Results** — sortable/filterable; Remove Race per row
6. **County QT** — Middlesex qualifying; stat cards + chart + event table
7. **Regional QT** — SE London qualifying; computed age bracket default
8. **Schedule** — Upcoming race calendar; County + Regional QT status per event
9. **Targets** — PBs Due for Renewal + Events Not Yet Swum

## Speed Dial FAB (v25)
Single ➕ parent button (bottom-right). Tap to expand 4 child actions upward:
  1. ⏱️ Add Race  (closest to thumb)
  2. ⚙️ Data Manager
  3. 🌙/☀️ Toggle Theme
  4. 👤 Swimmer Settings  (topmost)
Semi-transparent backdrop closes the dial on tap. Parent rotates 45° (→ ✕) when open.
Children are icon-only (no labels), centred on parent axis.

## Swimmer Profile Configuration
Set via 👤 → Settings modal:
- **Date of Birth** (swimDash_DOB): drives auto age-bracket selection
- **Gender** (swimDash_GENDER): "Boys" or "Girls" — matches QT JSON field

Championship dates (JS configuration section):
- COUNTY_CHAMPS_DATE: last day of Middlesex County Championships
- REGIONAL_CHAMPS_DATE: last day of SE London Regional Championships
Post-championship, next season's bracket applies automatically.

## Add Race
⏱️ child FAB → modal → updateData(RAW). Time: mm:ss.hh or ss.hh.
Valid events: 50/100/200/400/800/1500 Free | 50/100/200 Back | 50/100/200 Breast |
50/100/200 Fly | 100/200/400 IM (18 events total).
Confirmation flash: ⏱️ child turns green with ✓ for 2s after save.

## Remove Race
Delete button in All Results row → confirm → updateData(RAW).

## Persist to GitHub
Download my_swims.json from Data Manager → push to GitHub repo.
Also upload upcoming_races.json for schedule data.

## Deployment
Local HTML file (file://) or GitHub Pages. Works offline after first load.
Mobile responsive — @media (max-width: 500px) breakpoint.
