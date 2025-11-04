# Configuration Guide

This document explains all major configuration options used by the **Printable Sports TV Listings** scripts.  
The goal is to make it easy to adapt this project for different households, cable systems, or sports preferences.

---

### 1. Date and Output Settings

These control what days appear in the listings and where the generated Excel file is saved.

```python
from datetime import date, timedelta

START_DATE = date(2025, 11, 3)   # First day shown in the grid
NUM_DAYS   = 5                   # Number of consecutive days to include
DAY_DATES  = [START_DATE + timedelta(days=i) for i in range(NUM_DAYS)]

Variable	Type	Description
START_DATE	datetime.date	First date to include in the listings.
NUM_DAYS	int	How many days to include (e.g., 5 for a Mon–Fri view).
DAY_DATES	list[date]	Automatically derived list of dates used internally.
```

| Variable     | Type            | Description                                              |
| ------------ | --------------- | -------------------------------------------------------- |
| `START_DATE` | `datetime.date` | First date to include in the listings.                   |
| `NUM_DAYS`   | `int`           | How many days to include (e.g., `5` for a Mon–Fri view). |
| `DAY_DATES`  | `list[date]`    | Automatically derived list of dates used internally.     |




#### Output Settings
```
OUTPUT_DIR  = "output"
OUTPUT_XLSX = f"StoryPointEL_Listings_{START_DATE}_{NUM_DAYS}.xlsx"
```
| Variable      | Type  | Description                                                                         |
| ------------- | ----- | ----------------------------------------------------------------------------------- |
| `OUTPUT_DIR`  | `str` | Directory where the Excel file will be written.                                     |
| `OUTPUT_XLSX` | `str` | Filename for the generated Excel sheet. You can include date variables for clarity. |



### 2. Time Zone

All event times from the ESPN API are UTC. Set your preferred display timezone here:
```
from zoneinfo import ZoneInfo
LOCAL_TZ = ZoneInfo("America/Detroit")

# US Time Zones
# "America/New_York" <---- Eastern Time (ET)
# "America/Chicago"	<---- Central Time (CT)
# "America/Denver"	<---- Mountain Time (MT)
# "America/Los_Angeles"	<---- Pacific Time (PT)
```

### 3. Channel Lineups

The project supports multiple channel lineup profiles, making it easy to generate listings for different cable systems or households.
```
CHANNEL_MAPS = {
    "StoryPoint EL": {
        "CBS": 3, "NBC": 4, "FOX": 6, "ABC": 7,
        "ESPN": 11, "ESPNews": 12, "ESPNU": 13, "ESPN2": 14,
        "FS1": 15, "Big Ten": 47,
        "USA": 18, "TNT": 19, "truTV": 20, "TBS": 21,
    },
    "Example Household": {
        "CBS": 2, "NBC": 5, "FOX": 11, "ESPN": 35, ...
    },
}

ACTIVE_CHANNEL_MAP_NAME = "StoryPoint EL"
```

| Variable                  | Type                        | Description                                        |
| ------------------------- | --------------------------- | -------------------------------------------------- |
| `CHANNEL_MAPS`            | `dict[str, dict[str, int]]` | Maps *profile names* to channel name/number pairs. |
| `ACTIVE_CHANNEL_MAP_NAME` | `str`                       | Selects which profile to use for the current run.  |



### 4. Favorite Teams and School Highlighting

These control which rows get special formatting at the top of the Excel sheet.
```
FAVORITE_PRO_TEAMS = [
    "Detroit Lions", "Detroit Red Wings", "Detroit Tigers", "Detroit Pistons"
]

FAVORITE_SCHOOL = {
    "name": "Michigan State",
    "fill_color": "#18453B",   # Dark green background
    "font_color": "#FFFFFF",   # White text
}
```
| Variable             | Type        | Description                                                                        |
| -------------------- | ----------- | ---------------------------------------------------------------------------------- |
| `FAVORITE_PRO_TEAMS` | `list[str]` | Names of pro teams to always include, even if they’re not on your normal channels. |
| `FAVORITE_SCHOOL`    | `dict`      | Configures one “highlight” row for a specific college program.                     |

### 5. Sports to Include

Each entry maps a display label to one or more ESPN API league paths.
```
SPORTS = [
    ("NCAA Hockey", ["hockey/mens-college-hockey"]),
    ("NCAA Football", ["football/college-football"]),
    ("NCAA Basketball (M)", ["basketball/mens-college-basketball"]),
    ("NHL", ["hockey/nhl"]),
    ("NBA", ["basketball/nba"]),
    ("NFL", ["football/nfl"]),
    ("Soccer (Intl)", ["soccer/eng.1", "soccer/uefa.champions"]),
]
```
| Column        | Description                                               |
| ------------- | --------------------------------------------------------- |
| Display Label | Human-readable sport name shown in the grid.              |
| League Keys   | One or more ESPN scoreboard API endpoints for that sport. |

### 6. Broadcast Mapping and Filtering

ESPN returns a broadcast field for each game, often with inconsistent formatting.
These settings standardize those values and filter out unwanted entries.
```
CHANNEL_ALIASES = {
    "ESPN2": "ESPN2",
    "ESPNU": "ESPNU",
    "BTN": "Big Ten",
    "NBC": "NBC",
    "ABC": "ABC",
    "FOX": "FOX",
    "FS1": "FS1",
    "FS2": "FS2",
    "TNT": "TNT",
    "truTV": "truTV",
    "TBS": "TBS",
    # Add more mappings here as you encounter new strings
}

EXCLUDE_STREAMING_KEYS = [
    "ESPN+", "SEC+", "Peacock Premium", "Paramount+", "Bally App"
]
```
| Variable                 | Description                                                    |
| ------------------------ | -------------------------------------------------------------- |
| `CHANNEL_ALIASES`        | Normalizes ESPN broadcast strings to match your channel names. |
| `EXCLUDE_STREAMING_KEYS` | Filters out streaming-only listings from the final grid.       |

### 7. Styling and Formatting

Styling is handled inside the script via the xlsxwriter engine, but a few constants can be adjusted easily:
```
| Style           | Variable                | Default            | Description                         |
| --------------- | ----------------------- | ------------------ | ----------------------------------- |
| Title row       | `title_fmt`             | bold, size 18      | Main header (e.g. “Sports on TV”)   |
| Subtitle        | `subtitle_fmt`          | italic, size 12    | Secondary header (date range)       |
| Grid header     | `header_fmt`            | bold, shaded       | Column labels (#, Channel, Days)    |
| Channel numbers | `number_fmt`            | bold               | Channel numbers in the left column  |
| Alternate rows  | `base_fmt`, `shade_fmt` | white / light gray | Improves readability                |
| Favorite school | `fav_school_fmt`        | custom colors      | Highlighted row for favorite school |
```
Rich text colors per sport can be customized in the SPORT_STYLE mapping.

### 8. Debug Files
```
| Output File                 | Description                                            |
| --------------------------- | ------------------------------------------------------ |
| `events_flat_<profile>.csv` | Raw event-level table with all details.                |
| `unknown_broadcasts.csv`    | Summary of broadcast strings that couldn’t be matched. |
```
These help fine-tune your configuration and expand your channel alias list.

### 9. Common Customizations
| Task                           | What to Edit                        |
| ------------------------------ | ----------------------------------- |
| Switch to a new cable provider | Add a new entry to `CHANNEL_MAPS`.  |
| Add or remove sports           | Update the `SPORTS` list.           |
| Highlight a different college  | Change the `FAVORITE_SCHOOL` block. |
| Use a different timezone       | Modify `LOCAL_TZ`.                  |
| Hide streaming games           | Adjust `EXCLUDE_STREAMING_KEYS`.    |
| Fix a missing channel name     | Add to `CHANNEL_ALIASES`.           |

### 10. Tips
- Keep your CHANNEL_ALIASES up to date — ESPN occasionally changes their broadcast string formatting.

- When expanding to more days, test-print to make sure the grid still fits on one page.

- You can copy this entire config block into another notebook (like the single-day version) to maintain consistency between scripts.

- If Excel print margins drift, re-check your Excel version’s default paper settings — all layout settings are programmatically defined, but some local defaults can override them.


