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
- time: "mm:ss.hh" or "ss.hh" for sub-60s
  Regex: /^(\d{1,2}:)?\d{1,2}\.\d{2}$/ — single-digit seconds allowed (e.g. 1:2.50)
- splits: INDIVIDUAL LAP TIMES in seconds — NOT cumulative
  Accepted formats in Add Race modal: plain seconds (41.88) or mm:ss.hh (1:45.32)
  Parsed via timeToSec(); validated with !isFinite(s) || s <= 0

### Valid event strings
50/100/200/400/800/1500 Free | 50/100/200 Back | 50/100/200 Breast
50/100/200 Fly | 200/400 IM

## Processed DATA row (after processData())
Same as RAW plus:
- timeInSec: number — total time in seconds
- isPB: boolean — true only for the current fastest-ever swim of that event+course
  (running minimum; exactly one swim per event+course has isPB=true at any time)
- deltaPrev: number|null — seconds vs chronologically previous swim (negative = faster)
  null = first chronological swim of that event+course
  deltaPrev===null means debut only in Results tab context — see architecture.md

## County QT entry (county_qt.json)
```json
{"gender":"Boys","course":"Short Course","event":"50 Back","age":"12","qualify":38.10,"consider":40.80}
```
- course: "Short Course"/"Long Course" (NOT "S"/"L")
- age: "10+11","12","13","14","15","16","17+"
- qualify/consider: float seconds; null for events not offered to 10+11 age group

## SE London Regional QT entry (se_london_qt.json)
```json
{"gender":"Boys","course":"Long Course","event":"50 Back","age":"11/12","qualify":36.76,"consider":38.60}
```
- age: "11/12","13","14","15","16","17","18+"
- qualify/consider: always float, never null

## Course code conversion
RAW/DATA: "S"/"L" ↔ QT files: "Short Course"/"Long Course"
Conversion: course === 'Short Course' ? 'S' : 'L'

## GitHub URLs
RACE_DATA_URL:  .../asushinski9-netizen/swim-dash/main/my_swims.json
QT_DATA_URL:    .../asushinski9-netizen/swim-dash/main/county_qt.json
SE_QT_DATA_URL: .../asushinski9-netizen/swim-dash/main/se_london_qt.json

## localStorage authority
swimDash_RAW    — USER AUTHORITATIVE (never overwritten by GitHub)
swimDash_QT     — GitHub authoritative (re-fetched from GitHub if missing)
swimDash_SE_QT  — GitHub authoritative (re-fetched from GitHub if missing)
