# Swimming Performance Dashboard — Project Brief

## Purpose
Single-file HTML dashboard for tracking a competitive swimmer's race history,
personal bests, split analysis, and qualifying status against both county and
regional standards.

## Swimmer
Broomfield Park Swimming Club (BPSC), membership 1745801.
Dashboard branding uses BPSC royal blue (#1a56c4) and the club logo (base64 PNG embedded).

## Tech stack
- Single-file HTML5 — no build step, no bundler
- Chart.js 4.4.0 (CDN: jsdelivr)
- chartjs-adapter-date-fns (CDN: jsdelivr)
- Vanilla JS ES6+, CSS custom properties (light theme)
- No frameworks, no TypeScript

## Data sources
| File               | GitHub URL                                                                               | Description                    |
|--------------------|------------------------------------------------------------------------------------------|--------------------------------|
| my_swims.json      | .../asushinski9-netizen/swim-dash/main/my_swims.json                                     | Race entries                   |
| county_qt.json     | .../asushinski9-netizen/swim-dash/main/county_qt.json                                    | Middlesex County QT 2026       |
| se_london_qt.json  | .../asushinski9-netizen/swim-dash/main/se_london_qt.json                                 | SE London Regional QT          |

All three fetched in parallel at startup; localStorage fallback if offline.

## Qualifying time sources
- County: Middlesex County Championships 2026 PDF (harrowswim.com)
- Regional: SE London Regional Championships (se_london_qt.json uploaded directly)

## Deployment
- Opened directly as a local HTML file (file://) OR served from GitHub Pages
- Must work offline after first load (localStorage cache for all three data files)
- Must work on mobile (has @media 500px breakpoint)

## Add Race workflow
New races entered via the ＋ FAB are stored in localStorage only.
To make permanent: open ⚙️ Data Manager → ⬇️ Download my_swims.json → push file to GitHub.
The dashboard re-fetches from GitHub on next load.

## Tabs (v13)
1. 📊 Overview — stat cards, recent PBs, frequency chart, events breakdown, venues
2. 📈 Progression — time chart, improvement summary, best vs reference table
3. 🏆 Personal Bests — PB cards per event+course with improvement vs reference
4. ⚡ Splits — AI pacing coach, split comparison bar chart, pacing profile, split detail
5. 📋 All Results — sortable filterable results table with all swims
6. 🎯 County QT — Middlesex county qualifying status with chart and table
7. 🏅 Regional QT — SE London regional qualifying status with chart and table
