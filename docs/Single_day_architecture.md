
---

## ⚙️ **`docs/Single_day_architecture.md`**


# Single-Day Listing Architecture

This document outlines the structure and execution flow of the **Single-Day Sports TV Listing** script.  
It shares most of its logic with the weekly generator but simplifies the aggregation stage and adjusts the Excel layout for high-density scheduling.

---

## Overview

Workflow summary:
Configuration → Fetch ESPN data → Normalize & Map → Build Channel Grid → Format Excel → Print


Because it focuses on a single date, this version avoids any multi-column looping and devotes extra space to displaying multiple events per channel.

---

## 1. Configuration Layer

Handles user-defined constants:

- `TARGET_DATE` – the specific day to fetch events for  
- `CHANNEL_MAPS` and `ACTIVE_CHANNEL_MAP_NAME` – defines which cable lineup to use  
- `SPORTS` – list of ESPN scoreboard endpoints  
- `FAVORITE_PRO_TEAMS` and `FAVORITE_SCHOOL` – control top-section rows and styling  
- `LOCAL_TZ` – localizes all event times for readability

---

## 2. Data Fetching Layer

Uses `fetch_day_sport_multi(target_date, league_keys)` to hit the ESPN scoreboard API once per sport for the selected day.

Example endpoint:
https://site.api.espn.com/apis/site/v2/sports/hockey/mens-college-hockey/scoreboard?dates=20251108


Returned fields include:
- Event ID  
- Team names  
- Start time (UTC)  
- Broadcast network  

All events are gathered into a unified Python list for that day.

---

## 3. Normalization Layer

Each raw ESPN event is transformed through a consistent set of helper functions:

| Function | Purpose |
|-----------|----------|
| `to_local_timestr()` | Converts UTC timestamps to local formatted strings. |
| `build_title()` | Combines home/away names into readable matchup titles. |
| `map_broadcast()` | Maps ESPN broadcast text → canonical channel name. |
| `assign_channel_num()` | Looks up the proper number in the active channel map. |

The resulting dataset includes:
date | time_str | sport | tag | title | channel | channel_num | fav_row


---

## 4. Aggregation Layer

Unlike the weekly version, which groups by `(day, channel)`, the single-day version simplifies grouping to `(channel_num, channel)` only.

- Each cell represents all events for one channel on the chosen date.
- Within each cell, events are sorted by start time.
- Rows are ordered as:
  1. Favorite school row (highlighted)
  2. Favorite pro team rows
  3. All remaining channels in ascending order by number

---

## 5. Excel Rendering Layer

Generated using **XlsxWriter** via pandas’ `ExcelWriter`.

### Sheet Layout
| Section | Description |
|----------|-------------|
| Title | Main heading with the selected date (e.g., “Saturday, November 8, 2025”) |
| Subtitle | Optional short message like “All times Eastern” |
| Columns | `#`, `Channel`, and one single date column |
| Rows | One per channel, containing stacked rich-text entries |

### Styling
- Alternating shaded rows for clarity  
- Color-coded sport tags using the `SPORT_STYLE` dictionary  
- Special background/font for the favorite school row  
- Bold header and title formatting

### Print Setup
- Paper size: Letter  
- Orientation: Landscape  
- Margins: 0.2"  
- Fit to one page wide / one page tall  
- Header/footer show print timestamp  

---

## 6. Logging Layer

The script outputs:
- `events_flat_<profile>.csv` — one-day table for all parsed events  
- `unknown_broadcasts.csv` — unmatched broadcast strings

These support debugging and continuous maintenance of the alias map.

---

## 7. End-to-End Flow Diagram
| Configuration        |
| -------------------- |
| TARGET_DATE          |
| CHANNEL_MAPS         |
| SPORTS               |

     |
     v


ESPN API Fetch
fetch_day_sport_multi
-> JSON events

     |
     v
| Normalization        |
| -------------------- |
| convert timezone     |
| build title          |
| map broadcast        |

     |
     v
| Aggregation          |
| -------------------- |
| group by channel     |
| sort by time         |

     |
     v
| Excel Rendering      |
| -------------------- |
| format & styles      |
| fit to 1 page        |

     |
     v
| Final Excel File     |
| -------------------- |
| single-day view      |




---

## 8. Differences from Weekly Version

| Aspect | Weekly | Single-Day |
|--------|---------|-------------|
| Dates | Multiple (loop over range) | One target date |
| Columns | One per day | Only one column |
| Row density | Moderate | Higher (more stacked events per cell) |
| Output title | “Sports on TV (Nov 3–7, 2025)” | “Sports on TV — Saturday, Nov 8, 2025” |
| Purpose | Broad overview | Deep daily focus |

---

## 9. Future Enhancements

- Option to auto-generate both daily and weekly versions in a single run  
- Add time-based cell color gradients (e.g., early vs. late games)  
- Optional inclusion of streaming platforms in a separate color  
- Interactive HTML export for tablets or kiosks  

---

*See also:*  
- [Single_day_configuration.md](Single_day_configuration.md) – Configuration details  
- [architecture.md](architecture.md) – Full weekly version pipeline





