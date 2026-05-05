# Session Log

## v23 (latest)
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
