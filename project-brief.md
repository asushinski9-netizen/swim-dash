# Swimming Performance Dashboard — Project Brief

## Purpose
Single-file HTML dashboard for tracking a competitive swimmer's race history,
personal bests, split analysis, qualifying status vs county and regional standards,
upcoming race schedule, and performance targets.

## Swimmer
Broomfield Park Swimming Club (BPSC), membership 1745801.
Dashboard: BPSC royal blue (#1a56c4), club logo embedded as base64 JPEG.
DOB and gender are configurable via the 👤 Settings modal (stored in localStorage).

## Tech stack
Single-file HTML5 | Chart.js 4.4.0 (jsdelivr CDN) + chartjs-adapter-date-fns
Vanilla JS ES6+ | CSS custom properties (light/dark theme) | No frameworks / build step

## Data sources
| File                  | Authority    | Notes                                         |
|-----------------------|--------------|-----------------------------------------------|
| my_swims.json         | localStorage | User-editable; never overwritten              |
| county_qt.json        | GitHub       | Cached locally; re-fetched if missing         |
| se_london_qt.json     | GitHub       | Cached locally; re-fetched if missing         |
| upcoming_races.json   | GitHub       | Cached locally; re-fetched if missing         |

## Tabs (v24.1)
1. **Overview** — 6 stat cards (incl. Upcoming Races + PBs to Renew), recent PBs, frequency chart, events breakdown, venues
2. **Progression** — Improvement Summary + BvF table, Time Progression chart
3. **Personal Bests** — PB cards with improvement vs configurable reference
4. **Splits** — AI coach (DEBUT/CURRENT PB badge), split comparison, pacing profile
5. **All Results** — sortable/filterable (newest-first default); Remove Race per row
6. **County QT** — Middlesex qualifying; stat cards with event lists by status
7. **Regional QT** — SE London qualifying; default Short Course 25m, computed age bracket
8. **Schedule** — Upcoming race calendar; one row per event with County + Regional QT status and gap columns. Competition names truncated to 30 chars (desktop) / 20 chars (mobile).
9. **Targets** — PBs Due for Renewal (configurable threshold) + Events Not Yet Swum (flex card grid)

## Key design decisions
- localStorage-first for RAW: manually added races never overwritten by GitHub
- updateData() central function: single point for all data mutations and re-renders
  (does NOT call renderSchedule/renderTargets — those depend on UPCOMING, not RAW)
- Backdating-robust benchmarks: getPreviousSwims/getPreviousBestSwims use r !== pb identity
- Debut detection: four different methods for four UI contexts (see architecture.md)
- refSwim===pb guard is mode-aware: shows Debut only in non-latest modes
- swim.isPB for AI Coach badge: only the current overall PB gets the green badge
- Stable DOM wrappers: splitBarChartWrap, progChartWrap, paceChartWrap always in DOM
- Responsive QT headers: col-full/col-abbr swap on mobile
- Split input via timeToSec(): accepts both plain seconds and mm:ss.hh format
- getCC()/getPalette(): called inside render functions (not module-level) for theme correctness
- renderQTCells(): no inline font-size so mobile CSS cascade works uniformly
- Logo: loaded via GitHub raw URL, not embedded. onerror hides the img if offline.

## Swimmer Profile Configuration
Set via 👤 Settings FAB → modal:
- **Date of Birth** (`swimDash_DOB`): drives age bracket auto-selection for all QT tabs and Schedule/Targets
- **Gender** (`swimDash_GENDER`): matches gender field in QT JSON files ("Boys" / "Girls")

Championship dates (hardcoded in JS configuration section):
- `COUNTY_CHAMPS_DATE`: last day of Middlesex County Championships
- `REGIONAL_CHAMPS_DATE`: last day of SE London Regional Championships
Once a championship date has passed, the next season's age bracket is automatically used.

## Add Race
FAB + → modal → updateData(RAW). Time: mm:ss.hh or ss.hh. Splits: seconds or mm:ss.hh.
Valid events: 50/100/200/400/800/1500 Free | 50/100/200 Back | 50/100/200 Breast |
50/100/200 Fly | 100/200/400 IM
Note in modal: "Data is stored in your browser's local storage only."

## Remove Race
Delete button in All Results row → confirm → updateData(RAW).

## Persist to GitHub
Download my_swims.json from Data Manager → push to GitHub repo.
On next load from a clean browser, GitHub data is fetched as the baseline.
Similarly for upcoming_races.json.

## Deployment
Local HTML file (file://) or GitHub Pages. Works offline after first load.
Mobile responsive — @media (max-width: 500px) breakpoint.

## FABs (bottom-right stack)
| Position (desktop) | Button | Action |
|-------------------|--------|--------|
| 24px              | ➕     | Add Race |
| 88px              | ⚙️     | Data Manager |
| 152px             | 🌙/☀️  | Theme toggle |
| 216px             | 👤     | Swimmer Settings |
