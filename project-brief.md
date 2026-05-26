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
| File                | Authority    | Notes                                   |
|---------------------|--------------|-----------------------------------------|
| my_swims.json       | localStorage | User-editable; never overwritten        |
| county_qt.json      | GitHub       | Cached locally; re-fetched if missing   |
| se_london_qt.json   | GitHub       | Cached locally; re-fetched if missing   |
| upcoming_races.json | GitHub       | Cached locally; re-fetched if missing.  |
|                     |              | Also mutated locally by 📅 Add Upcoming |
|                     |              | and ✕ Remove. Download to persist.      |

## Tabs (v27)
1. **Overview** — 6 clickable stat cards, recent PBs, frequency chart, events breakdown, venues
2. **Progression** — Improvement Summary + BvF table, Time Progression chart
3. **Personal Bests** — PB cards with improvement vs configurable reference
4. **Splits** — AI coach, split comparison, pacing profile
5. **All Results** — sortable/filterable; Remove Race per row
6. **County QT** — Middlesex qualifying; dual SC+LC event cards with progress bars; status filter; clickable stat cards; SC/LC chart toggles; event-by-event table with SC/LC row toggles; bracket note
7. **Regional QT** — SE London qualifying; same redesigned layout as County tab
8. **Schedule** — Upcoming race calendar; County + Regional QT status per event; Del per row
9. **Targets** — PBs Due for Renewal + Events Not Yet Swum

## Speed Dial FAB (5 children)
Single ➕ parent button (bottom-right). Tap to expand 5 child actions upward:
  1. ⏱️ Add Race            (closest to thumb)
  2. 📅 Add Upcoming Race
  3. ⚙️ Data Manager
  4. 🌙/☀️ Toggle Theme
  5. 👤 Swimmer Settings    (topmost)

## County QT & Regional QT — v27 design

### Stat cards (top)
Four clickable cards — Qualified / Consideration / Outside / No PB.
Status based on best across SC+LC per event.
Each event listed with: course badge (SC/LC), best PB time, gap chip.
Clicking a card applies the Status filter and re-renders.

### Qualification status cards
4-column grid (desktop) / 1-column (mobile). One card per event.
- SC block (lighter): course badge, PB, gap chip, progress bar
- LC block (darker): same; "No PB" badge if not swum
- Card border = best status; Course block left border = that course's status
- Progress bar: per-event scaled, always shown, CT/QT time labels above markers
- Gap shown once only in stats row above bar

### Filters
Gender · Age Group · Stroke · Status
Status filter logic:
- Qualified: ≥1 course PB is Qualified
- Consideration: best=Consideration (none Qualified)
- Outside: all course PBs are Outside
- No PB: no PB on either course

### Chart
Removed. The bar chart was confusing due to SC/LC bar overlap issues.

### Event-by-event table
One row per event+course. SC/LC row toggles. "Course"→"C" on mobile.
Bracket note below table: gender, age group, championship year, next-season notice.

## Swimmer Profile Configuration
Set via 👤 → Settings modal:
- **Date of Birth** (swimDash_DOB): drives auto age-bracket selection
- **Gender** (swimDash_GENDER): "Boys" or "Girls"

Championship dates (JS configuration section):
- COUNTY_CHAMPS_DATE: last day of Middlesex County Championships
- REGIONAL_CHAMPS_DATE: last day of SE London Regional Championships

Season boundary:
- SEASON_START_MONTH = 9 (September, 1-indexed)

## Persist to GitHub
- Race data: Download my_swims.json → push to GitHub repo
- Upcoming: Download upcoming_races.json → push to GitHub repo

## Deployment
Local HTML file (file://) or GitHub Pages. Works offline after first load.
Mobile responsive — @media (max-width: 500px) breakpoint.
