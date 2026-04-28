# Session Log

## v21 (latest)
Full audit of v20.2 — six bugs fixed, one inconsistency resolved, one security improvement:

- Bug: "PBs this year" stat card counted historical isPB swims (any swim that was a PB
  when swum), not current PBs. An improving swimmer with a 200 Back PB beaten in Feb 2026
  still had their Jan 2025 swim counted. Fixed: pbs.filter(r => r.date >= currentYearStart)
  where pbs = getPBs() — counts only current overall PBs set this year.

- Bug: "Recent Personal Bests" in Overview showed up to 6 historical isPB swims (same
  issue — DATA.filter(r => r.isPB) includes old beaten PBs). Fixed: getPBs().sort by date
  descending, take first 6 — always shows the 6 most recently set current PBs.

- Bug: processData sort was unstable for same-date same-event entries across SC/LC.
  Two swims on the same date for the same event (e.g. 200 Back S and L) would sort
  indeterminately, causing non-deterministic deltaPrev and isPB values. Fixed: added
  || a.course.localeCompare(b.course) as a final sort tiebreaker.

- Bug: Doughnut chart in Overview used borderColor '#111827' (dark navy from the old dark
  theme). On the white light-theme surface this showed as dark segment dividers. Fixed to
  '#ffffff' so segment borders blend cleanly with the white card background.

- Improvement: eventsWithSplits was recomputed on every populateSelects() call (which runs
  on every updateData()). Added uniqueEventsWithSplits() memoised helper alongside the
  other unique* functions; populateSelects() now uses the cached result.

- UX: Del column header in All Results given cursor:default so it doesn't appear clickable
  like the sortable headers. All other th elements inherit cursor:pointer from the CSS rule.

- Security: Added escapeHtml() utility function. Applied to all user-supplied strings
  rendered into innerHTML: competition and venue in AI Coach, Results table (competition
  and venue columns), Overview Recent PBs (venue), and Splits tab Split Detail (venue).
  Prevents XSS if a malformed entry containing HTML tags is entered via Add Race modal.
  Note: low real-world risk since this is a personal dashboard with no server.

Changes NOT applied (reviewed and kept as-is):
- eventPieChart groups SC+LC together (display choice, not a bug)
- analyzeSwim prevPB already filters by swim.course, not the tab filter (correct)
- sortDir behaviour when switching columns via header click (acceptable UX tradeoff)
- getPBs().includes() O(n) scan is fine at this data scale

## v20.2
- AI Coach: getPBs().includes(swim) for CURRENT PB badge (not swim.isPB)
- Removed ABOVE PB badge — no badge for ordinary swims
- resetResultsFilters: sortDir set to -1 (newest-first consistent with default)

## v20.1
- Progression: latest mode self-reference shows "PB is latest" not Debut
- QT table headers: col-full/col-abbr responsive classes

## v20
- init() localStorage-first; updateData(); timeToSec splits; regex relaxed;
  dead PBs branch removed; finishDiff fixed; strokeOrder Other; sortDir -1

## v19–v9 — see earlier entries
