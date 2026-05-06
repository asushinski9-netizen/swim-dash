# 🏊 Swimming Performance Dashboard

A single-file HTML5 dashboard for competitive swimmers to track race history, analyze performance progression, monitor qualifying status, and optimize pacing strategy.

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

### 7 Main Tabs

| Tab | Purpose |
|-----|---------|
| **📊 Overview** | Dashboard snapshot: race count, recent PBs, frequency chart, event breakdown, venues visited |
| **📈 Progression** | Time trend analysis with filters (event, course, compare-to basis); improvement summary table |
| **🏆 Personal Bests** | All-time PBs per event+course with improvement vs. reference time; filterable by stroke & course |
| **⚡ Splits** | AI pacing coach feedback, split comparison bar chart, pacing profile, lap-by-lap detail view |
| **📋 All Results** | Sortable, filterable results table; add/edit/delete races inline |
| **🎯 County QT** | Middlesex County qualifying status vs. PBs; gap-to-qualify metrics; event-by-event qualification chart |
| **🏅 Regional QT** | SE London Regional qualifying status; same analysis as County tab |

### Quick Actions
- **➕ Add Race** (bottom-right FAB): Log new swim result with event, time, splits, venue, competition name
- **⚙️ Data Manager** (second FAB): Upload/download JSON data, reset dashboard
- **🌙 Theme Toggle** (third FAB): Switch between light and dark modes

---

## 📱 Device Support

- **Desktop:** Full-featured experience on 1+ column layouts
- **Tablet:** 2-column grid adaptive layout
- **Mobile:** Optimized for 500px+ screens with vertical stacking; all tables scale for small viewports

---

## 🗂️ Data Files

All data files are JSON-based and fetched from GitHub on startup. Offline fallback uses browser localStorage.

| File | Purpose |
|------|---------|
| **my_swims.json** | Race entries (date, event, time, splits, venue, competition) |
| **county_qt.json** | Middlesex County 2026 qualifying times by age/gender/stroke/course |
| **se_london_qt.json** | SE London Regional qualifying times by age/gender/stroke/course |

### Data Flow
1. Dashboard fetches all three JSON files in parallel on load
2. If offline or fetch fails, falls back to localStorage cache
3. New races entered via the **➕ Add Race** modal are stored **locally only** (not persisted to GitHub)
4. To save new races permanently: **⚙️ Data Manager** → **⬇️ Download my_swims.json** → push to GitHub

---

## 🎯 Key Analytics

### Performance Insights
- **PB Tracking:** Monitor personal bests by event, course, and stroke type
- **Improvement Metrics:** Compare current PB vs. first swim, previous best, latest swim, or previous race
- **Progression Charts:** Visualize time improvement over months/years with configurable baselines

### Split Analysis
- **Pacing Profile:** Identify whether you're a front-runner (fast early splits) or back-half swimmer
- **AI Coaching:** Feedback on consistent pacing vs. observed variability
- **Split Comparison:** Bar chart comparing lap times across multiple swims of the same event

### Qualifying Status
- **County QT:** Gap-to-qualify and consideration-time metrics for Middlesex Championships
- **Regional QT:** Same analysis for SE London Regional Championships
- **Age/Gender Filters:** Select relevant age group and gender to match competition categories

---

## 🛠️ Technology Stack

- **HTML5:** Single-file deployment, no bundler needed
- **Chart.js 4.4.0** (CDN): Lightweight time-series and bar charts
- **chartjs-adapter-date-fns:** Date formatting for timeline charts
- **Vanilla JavaScript (ES6+):** No frameworks; pure DOM manipulation
- **CSS Custom Properties:** Light/dark theme toggle
- **localStorage:** Persistent data caching for offline use

### Why Single-File?
✅ Zero build complexity  
✅ Easy to customize and deploy  
✅ Works on any web server  
✅ Mobile-friendly out of the box  
✅ All code auditable in one place  

---

## 📝 Adding a Race Result

1. Click **➕** button (bottom-right)
2. Fill in:
   - **Date:** Competition date
   - **Event:** 50–1500m stroke events
   - **Course:** Short (25m) or Long (50m)
   - **Time:** Format as `mm:ss.hh` or `ss.hh`
   - **Venue:** Pool location
   - **Competition:** Meet name
   - **Splits** (optional): Comma-separated lap times in seconds
3. Click **💾 Save Race**
4. Data is stored in browser localStorage
5. To persist: **⚙️ Data Manager** → **⬇️ Download** → commit to GitHub

---

## 🔐 Privacy & Data Storage

- **All data stays on your device.** localStorage is device-specific; no data is sent to any server.
- **To sync across devices:** Download JSON from Data Manager and upload to GitHub or another repo.
- **To backup:** Export my_swims.json regularly.

---

## 🎨 Customization

### Edit Branding
- Open `index.html` in a text editor
- Search for `"Broomfield Park SC"` and replace with your club name
- Search for the base64 logo (`data:image/png;base64,...`) and replace with your club logo

### Change Color Scheme
- Modify CSS custom properties in the `<style>` section (`:root` selector)
- `--accent`: Primary blue color (currently #1a56c4)
- `--gold`, `--green`, `--red`: Status and emphasis colors

### Add New Events
- Edit the `<select id="ar-event">` options in the Add Race modal
- Update `county_qt.json` and `se_london_qt.json` if needed

---

## 📚 Reference Documentation

- **[project-brief.md](project-brief.md)** — Original requirements and scope
- **[architecture.md](architecture.md)** — JavaScript module breakdown and data structures
- **[data-schema.md](data-schema.md)** — JSON field definitions and validation rules
- **[known-bugs-and-fixes.md](known-bugs-and-fixes.md)** — Current issues and workarounds

---

## 🐛 Known Limitations

- Splits input only accepts numeric lap times (no interval notation like "1:23.45")
- Charts require ~1 second to render on first load (Chart.js DOM painting)
- Safari mobile may show reduced chart interactivity due to canvas rendering
- localStorage has a ~5-10MB limit per domain (shouldn't be a problem unless you log 10,000+ races)

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
**Version:** 23 (7-tab dashboard with AI coaching, full offline support, light/dark theme)
