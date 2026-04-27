# Swimming Performance Dashboard — Project Brief

## Purpose
Single-file HTML dashboard for tracking a competitive swimmer's race history,
personal bests, split analysis, and qualifying status vs county and regional standards.

## Swimmer
Broomfield Park Swimming Club (BPSC), membership 1745801.
Dashboard: BPSC royal blue (#1a56c4), club logo as base64 PNG.

## Tech stack
Single-file HTML5 | Chart.js 4.4.0 (jsdelivr CDN) + chartjs-adapter-date-fns
Vanilla JS ES6+ | CSS custom properties (light theme) | No frameworks / build step

## Data sources
| File              | Authority    | Notes                                 |
|-------------------|--------------|---------------------------------------|
| my_swims.json     | localStorage | User-editable; never overwritten      |
| county_qt.json    | GitHub       | Cached locally; re-fetched if missing |
| se_london_qt.json | GitHub       | Cached locally; re-fetched if missing |

## Tabs (v20.1)
1. Overview — stats, recent PBs, frequency chart, events breakdown, venues
2. Progression — Improvement Summary + BvF table, Time Progression chart at bottom
3. Personal Bests — PB cards with improvement vs configurable reference
4. Splits — AI coach (DEBUT/CURRENT PB/ABOVE PB badge), split comparison, pacing profile
5. All Results — sortable/filterable (newest-first default); Remove Race per row
6. County QT — Middlesex qualifying; stat cards with event lists by status
7. Regional QT — SE London qualifying; default Short Course 25m, age 11/12

## Key design decisions
- localStorage-first for RAW: manually added races never overwritten by GitHub
- updateData() central function: single point for all data mutations and re-renders
- Backdating-robust benchmarks: getPreviousSwims/getPreviousBestSwims use r !== pb identity
- Debut detection: four different methods for four UI contexts (see architecture.md)
- refSwim===pb guard is mode-aware (v20.1): shows Debut only in non-latest modes
- swim.isPB for AI Coach badge (v20.1): only the current overall PB gets the green badge
- Stable DOM wrappers: splitBarChartWrap, progChartWrap, paceChartWrap always in DOM
- Responsive QT headers: col-full/col-abbr swap on mobile
- Split input via timeToSec(): accepts both plain seconds and mm:ss.hh format

## Add Race
FAB + → modal → updateData(RAW). Time: mm:ss.hh or ss.hh. Splits: seconds or mm:ss.hh.
Note in modal: "Data is stored in your browser's local storage only."

## Remove Race
Delete button in All Results row → confirm → updateData(RAW).

## Persist to GitHub
Download my_swims.json from Data Manager → push to GitHub repo.
On next load from a clean browser, GitHub data is fetched as the baseline.

## Deployment
Local HTML file (file://) or GitHub Pages. Works offline after first load.
Mobile responsive — @media (max-width: 500px) breakpoint.
