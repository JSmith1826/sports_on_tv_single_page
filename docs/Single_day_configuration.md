# Single-Day Listing Configuration Guide

This file explains how to configure and customize the **Single-Day Sports TV Listing** notebook (`daily_sports_tv_listing.ipynb`).  
It uses the same ESPN API data pipeline as the weekly version, but focuses on a **single date** to allow for more detailed, compact layouts ideal for busy sports days (e.g., college football Saturdays).

---

## 1. Date and Output Settings

Specify the single target date and output file name.

```python
from datetime import date

TARGET_DATE = date(2025, 11, 8)
OUTPUT_DIR  = "output"
OUTPUT_XLSX = f"StoryPointEL_Listings_{TARGET_DATE}.xlsx"
```

| Variable      | Type            | Description                                         |
| ------------- | --------------- | --------------------------------------------------- |
| `TARGET_DATE` | `datetime.date` | The specific day you want to generate listings for. |
| `OUTPUT_DIR`  | `str`           | Directory where the Excel file will be saved.       |
| `OUTPUT_XLSX` | `str`           | Filename pattern for the generated Excel file.      |

## 2. Time Zone

As with the weekly version, ESPN provides UTC timestamps.
Set your local display timezone here:
```
from zoneinfo import ZoneInfo
LOCAL_TZ = ZoneInfo("America/Detroit")
```

## 3. Channel Lineups

The same channel lineup logic applies as in the multi-day notbook — define one or more named channel maps and select the active one.
```
CHANNEL_MAPS = {
    "StoryPoint EL": {
        "CBS": 3, "NBC": 4, "FOX": 6, "ABC": 7,
        "ESPN": 11, "ESPNews": 12, "ESPNU": 13, "ESPN2": 14,
        "FS1": 15, "Big Ten": 47, "USA": 18, "TNT": 19,
    },
}

ACTIVE_CHANNEL_MAP_NAME = "StoryPoint EL"
```

## 4. Favorite Teams & Highlight Row

You can still highlight your favorite college team and display pro teams first.
This layout emphasizes that top section since there’s more vertical space available on a single-day sheet.
```
FAVORITE_PRO_TEAMS = ["Detroit Lions", "Detroit Red Wings"]
FAVORITE_SCHOOL = {
    "name": "Michigan State",
    "fill_color": "#18453B",
    "font_color": "#FFFFFF",
}
```

## 5. Sports Configuration

Since this version covers one day, the SPORTS list can be more extensive without overwhelming the grid.
You can include all the same ESPN scoreboard endpoints as the weekly generator.
```
SPORTS = [
    ("College Football", ["football/college-football"]),
    ("College Hockey", ["hockey/mens-college-hockey"]),
    ("College Basketball (M)", ["basketball/mens-college-basketball"]),
    ("NFL", ["football/nfl"]),
    ("NBA", ["basketball/nba"]),
    ("NHL", ["hockey/nhl"]),
    ("Soccer", ["soccer/eng.1", "soccer/uefa.champions"]),
]
```

## 6. Broadcast Mapping & Filtering
```
CHANNEL_ALIASES = {
    "ESPN2": "ESPN2", "ESPNU": "ESPNU", "BTN": "Big Ten",
    "NBC": "NBC", "ABC": "ABC", "FOX": "FOX", "FS1": "FS1",
    "FS2": "FS2", "TNT": "TNT", "truTV": "truTV", "TBS": "TBS",
}

EXCLUDE_STREAMING_KEYS = ["ESPN+", "Peacock Premium", "Paramount+", "Bally App"]
```

## 7. Styling & Print Layout
The single-day version modifies the grid for denser vertical content:
| Feature           | Description                                                            |
| ----------------- | ---------------------------------------------------------------------- |
| Title             | “Sports on TV — Saturday, November 8, 2025”                            |
| Fewer columns     | Only one day, so the layout emphasizes rows (channels and time slots). |
| Rich text cells   | Multiple games stacked per cell, with colored sport tags.              |
| Fit-to-page       | Configured to fit **exactly one landscape page** in Excel.             |
| Narrow margins    | 0.2″ default.                                                          |
| Alternate shading | Keeps rows readable in long lists.                                     |


## 8. Debugging Outputs
| File                        | Description                                        |
| --------------------------- | -------------------------------------------------- |
| `events_flat_<profile>.csv` | Flat event list for the selected day.              |
| `unknown_broadcasts.csv`    | Unmatched broadcast strings for alias maintenance. |


## 9. Typical Customizations
| Task                           | Edit This                                                |
| ------------------------------ | -------------------------------------------------------- |
| Change target date             | Update `TARGET_DATE`.                                    |
| Use a different channel lineup | Modify `CHANNEL_MAPS` and set `ACTIVE_CHANNEL_MAP_NAME`. |
| Add or remove sports           | Adjust the `SPORTS` list.                                |
| Add new broadcast aliases      | Extend `CHANNEL_ALIASES`.                                |
| Highlight a different school   | Update `FAVORITE_SCHOOL`.                                |

For a full  walkthrough of the data pipeline, see [Single_day_architecture.md](Single_day_configuration.md)

