# 🏊 Swimming Performance Dashboard

A single-file HTML5 dashboard for competitive swimmers to track race history, analyse performance progression, monitor qualifying status, and optimise pacing strategy.

**👤 Swimmer:** Broomfield Park Swimming Club (BPSC) · Membership #1745801  
**🎨 Tech:** Pure HTML5 + Vanilla JS (ES6+) + Chart.js · No build step, no dependencies to install

---

## 🚀 Quick Start

### Option 1: Open Locally
1. Clone or download the repo
2. Open `index.html` directly in your browser (`file://` protocol)
3. Dashboard works offline after first load (all data cached in localStorage)

### Option 2: Use GitHub Pages
Visit: [https://asushinski9-netizen.github.io/swim-dash](https://asushinski9-netizen.github.io/swim-dash)

### Option 3: Self-Host
Serve `index.html` from any web server (Apache, Nginx, Node.js, etc.)

---

## 📊 Features

### 9 Main Tabs

| Tab | Purpose |
|-----|---------|
| **📊 Overview** | Dashboard snapshot: race count, recent PBs, frequency chart, event breakdown, venues visited |
| **📈 Progression** | Time trend analysis with filters (event, course, compare-to basis); improvement summary table |
| **🏆 Personal Bests** | All-time PBs per event+course with improvement vs. reference time; filterable by stroke & course |
| **⚡ Splits** | AI pacing coach feedback, split comparison bar chart, pacing profile, lap-by-lap detail view |
| **📋 All Results** | Sortable, filterable results table; add/edit/delete races inline |
| **🎯 County QT** | Middlesex County qualifying status; dual SC+LC event cards with progress bars; chart with SC/LC toggles; event-by-event table with SC/LC row toggles; status filter; clickable stat cards |
| **🏅 Regional QT** | SE London Regional qualifying status; same redesigned layout as County tab |
| **🗓️ Schedule** | Upcoming race calendar with County + Regional QT status per event |
| **🎯 Targets** | PBs Due for Renewal + Events Not Yet Swum |

### Quick Actions (Speed Dial FAB)
- **⏱️ Add Race** — Log new swim result with event, time, splits, venue, competition name (multi-event)
- **📅 Add Upcoming Race** — Schedule a future race entry
- **⚙️ Data Manager** — Upload/download JSON data, reset dashboard
- **🌙 Theme Toggle** — Switch between light and dark modes
- **👤 Swimmer Settings** — Set date of birth and gender for auto age-bracket selection

---

## 📱 Device Support

- **Desktop:** Full-featured experience, QT cards in 4-column grid
- **Tablet:** 2-column adaptive layout
- **Mobile:** Optimised for 500px+ screens; QT cards 1-per-row; tables scaled to 0.60rem; abbreviated column headers

---

## 🗂️ Data Files

All data files are JSON-based and fetched from GitHub on startup. Offline fallback uses browser localStorage.

| File | Purpose |
|------|---------|
| **my_swims.json** | Race entries (date, event, time, splits, venue, competition) |
| **county_qt.json** | Middlesex County 2026 qualifying times by age/gender/stroke/course |
| **se_london_qt.json** | SE London Regional qualifying times by age/gender/stroke/course |
| **upcoming_races.json** | Scheduled future races (date, competition, venue, course, events) |

### Data Flow
1. Dashboard fetches all four JSON files in parallel on load
2. If offline or fetch fails, falls back to localStorage cache
3. New races entered via **⏱️ Add Race** are stored locally only
4. To save new races permanently: **⚙️ Data Manager** → **⬇️ Download my_swims.json** → push to GitHub

---

## 🎯 County QT & Regional QT Tabs (v27 redesign)

### Stat Cards (top)
Four clickable summary cards — Qualified / Consideration / Outside / No PB — based on best status across both SC and LC for each event. Each card lists events with the course type (SC/LC badge), best PB, and gap to QT or CT. Clicking a card applies the Status filter instantly.

### Qualification Status Cards
One card per event, displayed in a 4-column grid (desktop) / 1-column (mobile). Each card contains:
- **SC block** (lighter background) — PB, gap chip, per-event-scaled progress bar with CT and QT time labels above markers
- **LC block** (darker background) — same layout; "No PB" badge if event not yet swum in that course
- **Card border colour** — reflects best status across both courses (green / amber / red / grey)
- **Course block left border** — reflects that course's own status

### Filters
Gender · Age Group · Stroke · **Status** (Qualified / Consideration / Outside / No PB)

Status filter logic:
- **Qualified** — at least one course PB is Qualified
- **Consideration** — best status is Consideration (neither course Qualified)
- **Outside** — all course PBs are Outside (none Qualified or Consideration)
- **No PB** — no PB recorded on either course

### Chart
Six datasets: SC PB bars, LC PB bars, SC QT line, SC CT line, LC QT line, LC CT line. **SC / LC toggle buttons** above the chart show/hide each course's datasets without re-rendering.

### Event-by-Event Table
One row per event+course. **SC / LC row toggle buttons** above the table show/hide course rows. "Course" column abbreviated to "C" on mobile.

### Bracket Note
A note below the table confirms the gender, age group, and championship year used for qualifying times, including a notice when next-season standards are not yet published.

---

## 🎯 Key Analytics

### Performance Insights
- **PB Tracking:** Monitor personal bests by event, course, and stroke type
- **Improvement Metrics:** Compare current PB vs. first swim, previous best, latest swim, or previous race
- **Progression Charts:** Visualise time improvement over months/years with configurable baselines
- **Best Season Improvement:** Largest within-season time drop shown on Overview

### Split Analysis
- **Pacing Profile:** Identify whether you're a front-runner or back-half swimmer
- **AI Coaching:** Feedback on consistent pacing vs. observed variability
- **Split Comparison:** Bar chart comparing lap times across multiple swims of the same event

### Qualifying Status
- **County QT:** Dual SC+LC view with progress bars, gap metrics, and status filter
- **Regional QT:** Same analysis for SE London Regional Championships
- **Auto Age Bracket:** Derived from date of birth and championship date

---

## 🛠️ Technology Stack

- **HTML5:** Single-file deployment, no bundler needed
- **Chart.js 4.4.0** (CDN): Lightweight time-series and bar charts
- **chartjs-adapter-date-fns@3.0.0:** Date formatting for timeline charts
- **Vanilla JavaScript (ES6+):** No frameworks; pure DOM manipulation
- **CSS Custom Properties:** Light/dark theme toggle
- **localStorage:** Persistent data caching for offline use

---

## 📝 Adding a Race Result

1. Click **⏱️** button (Speed Dial)
2. Fill in shared fields: Date, Venue, Course, Competition Name
3. Add one or more event rows: Event, Time (mm:ss.hh or ss.hh), Splits (optional, comma-separated)
4. Click **💾 Save Race**
5. To persist: **⚙️ Data Manager** → **⬇️ Download** → commit to GitHub

---

## 🔐 Privacy & Data Storage

- **All data stays on your device.** localStorage is device-specific; no data is sent to any server.
- **To sync across devices:** Download JSON from Data Manager and upload to GitHub or another repo.
- **To backup:** Export my_swims.json regularly.

---

## 🐛 Known Limitations

- Splits input only accepts numeric lap times (no interval notation like "1:23.45")
- Charts require ~1 second to render on first load (Chart.js DOM painting)
- Safari mobile may show reduced chart interactivity due to canvas rendering
- localStorage has a ~5-10MB limit per domain

See [known-bugs-and-fixes.md](known-bugs-and-fixes.md) for detailed workarounds.

---

## 📧 Support & Feedback

For bugs, feature requests, or questions:
1. Check [known-bugs-and-fixes.md](known-bugs-and-fixes.md)
2. Review [architecture.md](architecture.md) for technical context
3. Open an issue on GitHub

---

## 📄 License

Public domain (no restrictions). Feel free to fork, modify, and redistribute.

---

**Last Updated:** May 2026  
**Version:** 27 (dual-course QT redesign with event cards, progress bars, status filter, SC/LC toggles)
