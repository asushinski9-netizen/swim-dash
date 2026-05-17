# Session Log

## v25 (latest)
- **Feature: Speed Dial FAB** — replaces the four individual floating buttons with a single
  parent ➕ button that expands a fan of 4 child actions on tap.
  - Parent button: ➕ (always visible, bottom-right). Rotates 45° to ✕ when open.
  - Child buttons fan upward (bottom to top, closest to thumb first):
    1. ⏱️ Add Race
    2. ⚙️ Data Manager
    3. 🌙/☀️ Toggle Theme
    4. 👤 Swimmer Settings
  - Semi-transparent backdrop dims dashboard when open; tapping it closes the dial.
  - Staggered entrance animation (0 / 50 / 100 / 150ms delays) via CSS transitions.
  - Children centred on parent axis: margin-right: 4px on .fab-child-row offsets
    48px children to align their centres with the 56px parent button.
  - No text labels — icon-only children.
  - closeFabDial() called at top of every action function so dial collapses before modal opens.
  - Confirmation flash after Add Race moved from parent to child fabAddRace button.
  - JS: fabDialOpen boolean, toggleFabDial(), openFabDial(), closeFabDial().
  - Mobile: dial anchors 16px from bottom-right; parent 50px, children 44px.
  - fabTheme id retained on child so applyTheme() continues to sync icon.
  - Built on clean v24.1 base — no tab overflow code present.

## v24.1
- Logo: GitHub raw URL (bpsc_logo.png). onerror hides if offline.
- Status badge font size (Targets): plain span inherits table font at all breakpoints.
- Upcoming Race venue removed from Targets renewal cell.
- Schedule note rewritten to clean format.
- Renewal note added below Targets PBs Due for Renewal table.
- Events Not Yet Swum: flex card grid with stroke-colour borders.
- 100 IM added to ALL_EVENTS and Add Race dropdown.
- Competition truncation: 30 chars desktop, 20 chars mobile.
- renderQTCells() inline font-size removed for consistent mobile scaling.

## v24
- Schedule tab: upcoming races, one row per event, QT status columns.
- Targets tab: PBs Due for Renewal + Events Not Yet Swum.
- Swimmer Settings modal: DOB + Gender in localStorage.
- Auto age-bracket selection from DOB + championship dates.
- Overview: 6 stat cards (grid6).
- renderQTCells(): two-column Status + Gap layout.

## v23
- Dark/light theme toggle. PALETTE_LIGHT/DARK. getCC()/getPalette().
- applyTheme(), toggleTheme(). swimDash_theme key.

## v22
- escapeHtml(pb.venue). renderQTChartAndTable shared helper. Progress bar 12.5x scale.

## v21
- escapeHtml() across all innerHTML. getPBs() fixes in Overview. processData sort stable.

## v20–v9 — see earlier entries
