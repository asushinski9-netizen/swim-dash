# Data Schemas

## RAW swim entry (my_swims.json)
```json
{
  "course": "S",          // "S" = Short Course 25m, "L" = Long Course 50m
  "event": "200 Back",    // "{distance} {stroke}" — stroke values below
  "date": "2025-11-15",   // ISO 8601, YYYY-MM-DD
  "competition": "Watford Swimming Club County Qualifier 2025",
  "venue": "Watford",
  "time": "2:55.68",      // mm:ss.hh or ss.hh for sub-60s
  "splits": [41.88, 46.55, 46.78, 40.47]  // INDIVIDUAL LAP TIMES in seconds
                                            // NOT cumulative — [43.46, 44.09] sums to 87.55
}
```

## CRITICAL: splits[] are already lap times, not cumulative totals
Do NOT derive legs via splits[i] - splits[i-1]. This was a past bug.

## Valid stroke values
Free, Back, Breast, Fly, IM

## Valid event strings
"50 Free", "100 Free", "200 Free", "400 Free", "800 Free", "1500 Free"
"50 Back", "100 Back", "200 Back"
"50 Breast", "100 Breast", "200 Breast"
"50 Fly", "100 Fly", "200 Fly"
"200 IM", "400 IM"

## Processed DATA row (after processData())
Same as RAW plus:
- timeInSec: number  — total time in seconds
- isPB: boolean      — true if this is the best time for event+course at time of swim
- deltaPrev: number|null — difference vs previous swim of same event+course (negative = faster)

## QT_DATA entry (county_qt.json)
```json
{
  "gender": "Boys",
  "course": "Short Course",   // "Short Course" or "Long Course" (NOT "S"/"L")
  "event": "50 Back",         // matches RAW event string exactly
  "age": "12",                // "10+11", "12".."16", "17+"
  "qualify": 38.10,           // seconds (float), null if event not offered for age
  "consider": 40.80           // seconds (float), null if event not offered for age
}
```

## Course code mapping
RAW/DATA uses "S"/"L" — QT_DATA uses "Short Course"/"Long Course"
Conversion: course === 'Short Course' ? 'S' : 'L'