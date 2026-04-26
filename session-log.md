# Session Log

## v19 (latest)
- Bug: CSS linter vendorPrefix warning on line 858 — Add Race modal selects used
  -webkit-appearance:none without the standard appearance:none alongside it.
  Fixed: added appearance:none to all inline-style selects in Add Race modal.
  The global select, .btn rule already had both properties correctly.
- Bug: Results tab Debut badge not showing on first swim of events with multiple swims.
  Root cause: v17 changed debut detection to swimCount===1 (only events swum once ever).
  This broke the original behaviour: the chronologically first swim of any event should
  show Debut in the Δ Prev column regardless of how many later swims exist.
  Fixed: reverted to r.deltaPrev===null — processData() sets deltaPrev=null for the first
  chronological swim of each event+course, which is the correct debut indicator here.
  Removed the now-unused swimCounts declaration from renderResults.
- UX: Added localStorage note below Save Race buttons in Add Race modal:
  "Data is stored in your browser's local storage on this device only. Use ⚙️ → ⬇️ Download
  to save permanently."

## v18
- Bug: Splits tab compared every swim against course PB not previous PB.
  analyzeSwim reverted to r.date < swim.date filter (v17 change overcorrected).
- Bug: Oldest swim no longer showed as Debut in Splits (same v17 overcorrection).
- Bug: Backdated PB (first and fastest swim) showed dash in Progression Improvement
  Summary and compared to itself in BvF.
  Fixed: debut guard extended to !refSwim || swims===1 || refSwim===pb in both
  Improvement Summary forEach and BvF table forEach.

## v17
- Bug: Backdated race broke Progression/Personal Bests benchmark lookups.
  getPreviousSwims: r !== pb (not date filter); newest non-PB swim.
  getPreviousBestSwims: r !== pb; fastest non-PB swim (second-best overall).
  Overview Recent PBs debut: swimCount===1 (not deltaPrev===null).
  Results Δ Prev debut: swimCount===1 [REVERTED in v19 — deltaPrev===null is correct].

## v16
- Bugs: paceChart null crash; saveNewRace missing renderRegionalQualifying;
  stale scatter comment; renderRegionalQualifying used DATA.forEach not getPBs;
  lap delta string comparison; county QT CC.legend on ticks; resetResultsFilters
  sort state; county QT colspan 7 with 8-column table.
- Improvement: renderRegionalQualifying added to init() pre-render.

## v15
- Regional QT default SC; stat card mobile font; Remove Race ✕ button;
  Add Race FAB centred; Time Progression at HTML bottom; Splits empty states;
  Pacing Profile Y-axis reversed.

## v14
- Tabs = btn.sec colour; stat cards centred; BvF/QT table mobile fonts;
  buildQtStatCards() shared helper with event lists.

## v13
- Regional QT tab (se_london_qt.json); QT PB Date column; progression ghost fix;
  sortedForPace; mobile tab override.

## v12
- Progression debut stable wrapper; single-race callout; horizontal bar split chart;
  sorted pacing; Overview charts bigger; grid overflow fix.

## v11
- null.closest() → splitBarChartWrap; downloadRaceData; fab-data.

## v10
- Real BPSC PNG; CC.grid; PALETTE; multiple bug fixes.

## v9
- Light theme; BPSC blue; debut badges; Add Race FAB.
