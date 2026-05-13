# Session Log

## v24.1 (latest)

### New features (v24)
- **Schedule tab (🗓️)**: Upcoming races from `upcoming_races.json`; one row per event.
  - Columns: Date, Days Until, Competition (truncated), Venue, Course, Event, Current PB, County Status, County Gap, Regional Status, Regional Gap.
  - 48-hour grace period — past meets drop off automatically.
  - Footer note shows which age bracket was used for each competition.
- **Targets tab (🎯)**: Two sections.
  - Section 1 — PBs Due for Renewal: one row per event (fastest PB across all courses), flagged if older than configurable threshold (3/6/9/12 months). Columns: Event, Fastest PB, Course, Set date, Age, County Status, County Gap, Regional Status, Regional Gap, Upcoming Race, Status badge. Oldest PBs sort first.
  - Section 2 — Events Not Yet Swum: all 18 standard events with no recorded result, displayed as a responsive flex card grid with stroke-colour-coded left borders. Shows next scheduled race if available.
- **Swimmer Settings modal (👤 FAB)**: DOB and Gender stored in localStorage (`swimDash_DOB`, `swimDash_GENDER`). Saving re-syncs QT dropdowns and re-renders Schedule, Targets, QT tabs, and Overview.
- **Auto age-bracket selection**: `getCountyAgeBracket()` and `getRegionalAgeBracket()` compute correct age bracket from DOB + championship last-day dates. Post-championship, next season's bracket is used automatically.
- **Upcoming races data**: New `swimDash_UPCOMING` localStorage key; fetched from `UPCOMING_DATA_URL` on first load; uploadable via Data Manager (4th file slot).
- **Overview expanded to 6 stat cards** (`.grid6`): added Upcoming Races (→ Schedule) and PBs to Renew (→ Targets, 6-month threshold).
- **`renderQTCells(s)`**: Returns `[statusCell, gapCell]` pair for two-column QT layout. Gap logic: Outside → gap to CT; Consideration → gap to QT; Qualified → ✓. No inline font-size — inherits from table CSS for consistent mobile scaling.
- **Competition column truncation**: 30 chars on desktop, 20 chars on mobile (`.comp-full` / `.comp-short` CSS classes).
- **QT filter defaults** now call `resetQualifyingFilters()` / `resetRegionalFilters()` which use computed age brackets rather than hardcoded values.

### Fixes (v24.1 rounds)
- **Logo**: MIME type corrected from `image/png` to `image/jpeg`; spurious `fffd` byte before JPEG EOI stripped.
- **Schedule stat cards removed**: four key-metric cards gone from Schedule tab (info is in QT tabs).
- **"Upcoming Events" → "Upcoming Races"** on Overview stat card.
- **PB column (Schedule)**: Now one line — `0:49.19 (S)` format.
- **Schedule note**: Rewritten to `County QT: age X bracket · Regional QT: age X bracket. [season note if applicable].`
- **Renewal note**: Identical wording added below Targets renewal table.
- **Venue removed** from Targets upcoming race cell.
- **Status badge font size (Targets)**: Replaced `<span class="badge">` (which carried its own `font-size: 0.62rem`) with a plain `<span>` that inherits table font at all breakpoints — fixes both desktop (too small) and mobile (too large) inconsistencies.
- **Never-swum table** replaced with flex card grid for better desktop UX.
- **`100 IM` added** to `ALL_EVENTS` and to the Add Race dropdown.
- **All inline `font-size` removed** from `renderQTCells()` spans so mobile CSS `0.60rem !important` rule applies uniformly.

## v23
- Feature: Dark / light theme toggle
  - body.dark CSS class overrides all :root variables with dark equivalents
  - Two new CSS variables --tab-active and --header-start/--header-end so tab
    and header colours respond to theme without hardcoded hex in rules
  - PALETTE split into PALETTE_LIGHT and PALETTE_DARK; getPalette() returns the
    correct one at render time
  - CC constant replaced with getCC() function returning correct grid/tick/legend
    colours for the active theme
  - const CC = getCC(); const PALETTE = getPalette(); injected at the top of every
    render function that uses chart colours so re-renders always pick up the active theme
  - applyTheme(theme): sets body.dark class, updates 🌙/☀️ button icon, persists to
    localStorage as swimDash_theme
  - toggleTheme(): switches theme, resets overviewRendered, re-renders all chart tabs
    (Overview, Progression, Splits, County QT, Regional QT). Does NOT call processData —
    data is unchanged, only colours need refreshing.
  - IIFE before DOMContentLoaded sets body.dark class immediately from localStorage to
    prevent flash of wrong theme on load. Button icon is synced inside init() once DOM ready.
  - Theme FAB: 🌙/☀️ button at bottom:152px (desktop) / bottom:132px (mobile),
    same size as other FABs (56px desktop, 50px mobile), sits above ⚙️ Data button
  - Default: light theme. Preference persists across sessions via swimDash_theme key.

## v22
- Security: escapeHtml(pb.venue) in renderPBs.
- Comments: two stale Fix N comments cleaned.
- Bug: uniqueEventsWithSplits sort tiebreaker added.
- DRY: renderQTChartAndTable shared helper; 150 lines removed.
- Progress bar scale: 5→12.5 multiplier (8% = full bar).

## v21
- Security: escapeHtml() utility across all user-supplied innerHTML.
- Bug: Overview PBs this year / Recent PBs used isPB; fixed to getPBs().
- Bug: processData sort unstable for same-date SC/LC; course tiebreaker added.
- Bug: Doughnut border dark navy; fixed to #ffffff.
- Improvement: uniqueEventsWithSplits() memoised.
- UX: Del column header cursor:default.

## v20.2
- getPBs().includes(swim) for CURRENT PB badge; ABOVE PB badge removed.
- resetResultsFilters sortDir -1.

## v20.1
- Progression latest mode self-reference shows "PB is latest" not Debut.
- QT table headers: col-full/col-abbr responsive.

## v20
- init() localStorage-first; updateData(); timeToSec splits; regex relaxed;
  finishDiff fixed; strokeOrder Other; sortDir -1.

## v19–v9 — see earlier entries
