# Data Schemas

## RAW swim entry (my_swims.json)
```json
{
  "course": "S",
  "event": "200 Back",
  "date": "2025-11-15",
  "competition": "Watford Swimming Club County Qualifier 2025",
  "venue": "Watford",
  "time": "2:55.68",
  "splits": [41.88, 46.55, 46.78, 40.47]
}
```

### Field rules
- course: "S" = Short Course 25m, "L" = Long Course 50m
- date: ISO 8601, YYYY-MM-DD
- time: "mm:ss.hh" or "ss.hh" for sub-60s events
- splits: INDIVIDUAL LAP TIMES in seconds — NOT cumulative
  - [43.46, 44.09] means lap 1 = 43.46s, lap 2 = 44.09s, total = 87.55s
  - Do NOT use splits[i] - splits[i-1]

### Valid event strings
50/100/200/400/800/1500 Free
50/100/200 Back
50/100/200 Breast
50/100/200 Fly
200/400 IM

## Processed DATA row (after processData())
Same fields as RAW plus:
- timeInSec: number  — total time in seconds (from timeToSec(r.time))
- isPB: boolean      — true if fastest time for this event+course at the point it was swum
                       (recalculated from scratch on every processData() call)
- deltaPrev: number|null — seconds vs chronologically previous swim of same event+course
                           negative = faster, null = first chronological swim of that event
                           NOTE: deltaPrev === null does NOT reliably indicate a debut;
                           use getSwimCounts() === 1 for debut detection

## County QT entry (county_qt.json)
```json
{
  "gender": "Boys",
  "course": "Short Course",
  "event": "50 Back",
  "age": "12",
  "qualify": 38.10,
  "consider": 40.80
}
```

### Field rules
- course: "Short Course" or "Long Course" (NOT "S"/"L")
- age: "10+11", "12", "13", "14", "15", "16", "17+"
- qualify / consider: seconds as float; can be null for events not offered to 10+11 age group

## SE London Regional QT entry (se_london_qt.json)
```json
{
  "gender": "Boys",
  "course": "Long Course",
  "event": "50 Back",
  "age": "11/12",
  "qualify": 36.76,
  "consider": 38.60
}
```

### Field rules
- course: "Short Course" or "Long Course"
- age: "11/12", "13", "14", "15", "16", "17", "18+"
- qualify / consider: always a float, never null

## Age group comparison
| County QT | SE London QT | Notes                        |
|-----------|--------------|------------------------------|
| 10+11     | 11/12        | Different grouping           |
| 12        | (none)       | County only                  |
| 13–16     | 13–16        | Same                         |
| 16        | 16           | Same                         |
| 17+       | 17           | County groups 17+; SE splits |
| (none)    | 18+          | SE London only               |

## Course code conversion
RAW / DATA uses "S" / "L"
QT files use "Short Course" / "Long Course"
Conversion used everywhere: course === 'Short Course' ? 'S' : 'L'

## GitHub raw URLs
RACE_DATA_URL:   https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/my_swims.json
QT_DATA_URL:     https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/county_qt.json
SE_QT_DATA_URL:  https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/se_london_qt.json

## localStorage keys
swimDash_RAW    — race entries
swimDash_QT     — county qualifying times
swimDash_SE_QT  — SE London regional qualifying times
