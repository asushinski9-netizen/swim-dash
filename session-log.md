# Session Log

## v20.1 (latest)
- Bug: Progression "Latest Swim" mode showed Debut badge when PB is the most recent swim.
  Root cause: the `refSwim === pb` debut guard (added in v18 for backdated entries) fired
  for ALL compare modes, including 'latest'. When the overall PB is also the latest swim
  (the normal case for an improving swimmer), getLatestSwims() returns the PB itself as
  the reference, refSwim === pb triggers, and Debut is shown incorrectly.
  Fix: the refSwim===pb guard now only fires when compareMode !== 'latest'. For 'latest'
  mode a self-reference means "PB is the most recent swim" — shows "— PB is latest" row.
  Same fix applied to both Improvement Summary forEach and BvF table forEach.
  Genuine debut detection for single-swim events (swims===1) and missing references
  (!refSwim) is unchanged and unaffected.

- Optimization: AI Pacing Coach "New Best" badge replaced with "Current PB" logic.
  Previously: statusLabel showed NEW BEST whenever swim.timeInSec < prevPB.timeInSec
  (faster than the previous benchmark), which is true for every swim that set a new PB
  at the time — including historical PBs that have since been beaten.
  Now: statusLabel uses swim.isPB (set by processData) — only the current fastest-ever
  swim for that event+course gets the green badge. All other non-debut swims show "Above PB".

- Optimization: County QT and Regional QT table headers responsive on mobile.
  Added .col-full (visible on desktop, hidden on mobile) and .col-abbr (hidden on desktop,
  visible on mobile) CSS classes. Applied to four column headers in both QT tables:
  "Qualify Time" → QT, "Consideration Time" → CT,
  "Gap to Qualify" → Gap to QT, "Gap to Consider" → Gap to CT.
  Desktop shows full strings; mobile shows abbreviations.

## v20
- init() localStorage-first: RAW from localStorage authoritative; QT fetched from GitHub
  if not cached. RAW only fetched from GitHub if localStorage empty.
- updateData(newRaw): central state management function for all data mutations.
- Split parsing: timeToSec() replaces parseFloat(); !isFinite guard replaces isNaN.
- Time regex relaxed: allows single-digit seconds.
- renderPBs: removed dead "slower than PB" branch.
- finishDiff: fixed for 50m 2-lap races; midPaceAvg uses slice(1,-1).
- strokeOrder: 'Other' added to both qualifying sort arrays.
- sortDir default: -1 (newest-first).

## v19
- CSS: paired appearance:none with -webkit-appearance in Add Race modal.
- Results: Debut reverted to deltaPrev===null.
- UX: localStorage note in Add Race modal.

## v18
- analyzeSwim reverted to date filter; refSwim===pb debut guard added.

## v17
- getPreviousSwims/getPreviousBestSwims: r !== pb identity filter.

## v16–v9 — see earlier entries
