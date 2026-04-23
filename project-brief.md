# Swimming Performance Dashboard — Project Brief

## Purpose
Single-file HTML dashboard for tracking a competitive swimmer's race history,
personal bests, split analysis, and qualifying status against county and regional standards.

## Swimmer
Broomfield Park Swimming Club (BPSC), membership 1745801.
Dashboard uses BPSC royal blue (#1a56c4) and the club logo (base64 PNG).

## Tech stack
- Single-file HTML5 — no build step, no bundler
- Chart.js 4.4.0 (CDN: jsdelivr) + chartjs-adapter-date-fns
- Vanilla JS ES6+, CSS custom properties (light theme)
- No frameworks, no TypeScript

## Data sources
| File              | GitHub URL (asushinski9-netizen/swim-dash/main/) | Description             |
|-------------------|--------------------------------------------------|-------------------------|
| my_swims.json     | my_swims.json                                    | Race entries            |
| county_qt.json    | county_qt.json                                   | Middlesex County QT 2026|
| se_london_qt.json | se_london_qt.json                                | SE London Regional QT   |

All three fetched in parallel at startup; localStorage fallback if offline.

## Tabs (v15)
1. 📊 Overview — stat cards, recent PBs, frequency chart, events breakdown, venues
2. 📈 Progression — summary cards + BvF table, then Time Progression chart at bottom
3. 🏆 Personal Bests — PB cards per event+course with improvement vs reference
4. ⚡ Splits — AI pacing coach, split comparison bar chart, split detail, pacing profile
5. 📋 All Results — sortable/filterable table with ✕ delete button per row
6. 🎯 County QT — Middlesex county qualifying; detailed stat cards with event lists
7. 🏅 Regional QT — SE London qualifying; defaults to Short Course (25m), age 11/12

## Add Race workflow
＋ FAB → modal → saved to swimDash_RAW localStorage → all tabs re-render.
To make permanent: ⚙️ → ⬇️ Download my_swims.json → push to GitHub.

## Remove Race workflow
All Results tab → ✕ button on any row → confirm dialog → removed from RAW →
persisted to localStorage → all tabs re-render.
Note: Changes are local only until the JSON is downloaded and pushed to GitHub.

## Deployment
- Local HTML file (file://) or GitHub Pages
- Works offline after first load (localStorage cache for all three data files)
- Mobile responsive (@media 500px breakpoint)
