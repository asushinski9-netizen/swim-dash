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
- course: "S" = Short Course 25m, "L" = Long Course 50m
- time: "mm:ss.hh" or "ss.hh" for sub-60s
- splits: INDIVIDUAL LAP TIMES in seconds — NOT cumulative

## CRITICAL: splits[] are lap times, not cumulative totals
Do NOT use splits[i] - splits[i-1]. Direct values only.

## Valid stroke values: Free, Back, Breast, Fly, IM

## Valid event strings
50/100/200/400/800/1500 Free, 50/100/200 Back, 50/100/200 Breast,
50/100/200 Fly, 200/400 IM

## Processed DATA row (after processData())
Same as RAW plus:
- timeInSec: number  — total time in seconds
- isPB: boolean      — PB for event+course at time of swim
- deltaPrev: number|null — diff vs previous swim (negative = faster)

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
Age values: "10+11", "12", "13", "14", "15", "16", "17+"
qualify/consider can be null for events not offered to 10+11 age group.

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
Age values: "11/12", "13", "14", "15", "16", "17", "18+"
No null values — all events offered for all age groups.

## Age group comparison
| County QT | SE London QT |
|-----------|--------------|
| 10+11     | 11/12        |
| 12        | (none)       |
| 13–16     | 13–17        |
| 17+       | 18+          |

## Course code mapping
RAW/DATA: "S" / "L"
QT files: "Short Course" / "Long Course"
Conversion: course === 'Short Course' ? 'S' : 'L'

## GitHub URLs
RACE_DATA_URL:  https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/my_swims.json
QT_DATA_URL:    https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/county_qt.json
SE_QT_DATA_URL: https://raw.githubusercontent.com/asushinski9-netizen/swim-dash/main/se_london_qt.json
