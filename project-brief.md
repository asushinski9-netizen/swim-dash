# Swimming Performance Dashboard — Project Brief

## Purpose
Single-file HTML dashboard for tracking a competitive swimmer's race history,
personal bests, split analysis, and qualifying status against county and regional standards.

## Swimmer
Broomfield Park Swimming Club (BPSC), membership 1745801.
Dashboard branding: BPSC royal blue (#1a56c4), club logo embedded as base64 PNG.

## Tech stack
- Single-file HTML5 — no build step, no bundler, no dependencies to install
- Chart.js 4.4.0 (CDN: jsdelivr) + chartjs-adapter-date-fns
- Vanilla JS ES6+, CSS custom properties (light theme)
- No frameworks, no TypeScript

## Data sources
| File              | GitHub path (asushinski9-netizen/swim-dash/main/) | Description              |
|-------------------|---------------------------------------------------|--------------------------|
| my_swims.json     | my_swims.json                                     | All race entries         |
| county_qt.json    | county_qt.json                                    | Middlesex County QT 2026 |
| se_london_qt.json | se_london_qt.json                                 | SE London Regional QT    |

All three fetched in parallel at startup. Falls back to localStorage individually if offline.

## Tabs (v17)
1. 📊 Overview      — stat cards, recent PBs, frequency chart, events breakdown, venues
2. 📈 Progression   — Improvement Summary + BvF table, then Time Progression chart at bottom
3. 🏆 Personal Bests — PB cards per event+course with improvement vs configurable reference
4. ⚡ Splits        — AI pacing coach, split comparison bar chart, pacing profile
5. 📋 All Results   — sortable/filterable table; ✕ Remove Race button per row
6. 🎯 County QT     — Middlesex county qualifying; stat cards list events by status
7. 🏅 Regional QT   — SE London qualifying; defaults to Short Course 25m, age 11/12

## Add Race
＋ FAB → modal (date, event, course, time, venue, competition, splits) →
appended to RAW → localStorage → invalidateCache → all tabs re-render.

## Remove Race
✕ button in All Results row → confirm dialog → RAW.splice → localStorage → all tabs re-render.

## Persist to GitHub
Changes via Add/Remove Race are local only until exported.
⚙️ Data Manager → ⬇️ Download my_swims.json → push file to GitHub repo.
Dashboard re-fetches from GitHub on next load.

## Deployment
- Open directly as a local HTML file (file://) or serve from GitHub Pages
- Works offline after first successful load (localStorage cache)
- Mobile responsive — @media (max-width: 500px) breakpoint

## Key design decisions
- Backdating-robust: benchmark lookups use object identity (r !== pb), not date comparison,
  so races entered out of chronological order are handled correctly throughout
- Debut detection: swimCount === 1 from getSwimCounts(), not deltaPrev === null
- Stable wrapper pattern: splitBarChartWrap, progChartWrap, paceChartWrap always exist in DOM;
  canvas elements inside them are restored before chart operations
- Memoised lookups: getPBs(), getSwimCounts() etc. cache results in cache{} object;
  invalidateCache() clears all on any data change
