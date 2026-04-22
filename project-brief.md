# Swimming Performance Dashboard — Project Brief

## Purpose
Single-file HTML dashboard for tracking swimmers's competitive swimming 
history, personal bests, split analysis, and county qualifying status.

## Tech stack
- Single-file HTML5 — no build step, no bundler
- Chart.js 4.4.0 (CDN: jsdelivr)
- chartjs-adapter-date-fns (CDN: jsdelivr)
- Vanilla JS ES6+, CSS custom properties (dark theme)
- No frameworks, no TypeScript

## Add Race workflow
New races entered via the ＋ FAB are stored in localStorage only (swimDash_RAW).
To make them permanent: open ⚙️ Data Manager → ⬇️ Download my_swims.json → push to GitHub.
The dashboard fetches from GitHub on load and falls back to localStorage if offline.

## Data sources
- Race data: GitHub raw JSON → localStorage cache → manual upload fallback
- RACE_DATA_URL: https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/my_swims.json
- QT_DATA_URL: https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/county_qt.json
- Qualifying times source: Middlesex County Championships 2026 PDF

## Deployment
- Opened directly as a local HTML file (file://) OR served from GitHub Pages
- Must work offline after first load (localStorage cache)
- Must work on mobile (has @media 500px breakpoint)