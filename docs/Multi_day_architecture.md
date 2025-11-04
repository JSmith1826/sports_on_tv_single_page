# Project Architecture

This document provides a high-level overview of how the **Printable Sports TV Listings** scripts work — from fetching live data to producing a formatted Excel file ready for printing.

---

## Overview

The project consists of two main workflows:

1. **Weekly Generator** (`weekly_sports_tv_listings.ipynb`):  
   Builds a multi-day grid (typically 5–7 days) showing sports events across your preferred TV channels.

2. **Single-Day Generator** (e.g. Saturdays):  
   Focuses on one day’s schedule in greater detail, often used for high-volume days like college football Saturdays.

Both share the same overall pipeline:


---

## 1. Configuration Layer

This stage defines all user-facing settings:

- **Date range:** start date and number of days.  
- **Channel lineup:** named profile mapping channel names to numbers.  
- **Favorite teams/school:** highlighted rows at the top of the listing.  
- **Sports & leagues:** which ESPN scoreboard endpoints to query.  
- **Display timezone:** for localizing UTC times.  
- **Aliases and filters:** normalize broadcast strings and exclude streaming services.

> All configuration is defined in the first code block of each notebook for easy editing.

---

## 2. Data Fetching Layer

The data layer connects to ESPN’s public **scoreboard API** for each sport and date.

Example endpoint:
https://site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard?dates=YYYYMMDD


Each response includes a list of games (“events”) with details such as:
- Team names
- Start times (UTC)
- Competition info (league, season)
- Broadcast channels

The script loops over:
1. Each date in the selected range.  
2. Each sport and its league keys.

For every league key, the helper function `fetch_day_sport_multi()` requests the scoreboard data and accumulates all valid events.

---

## 3. Normalization & Mapping Layer

After data retrieval:

1. **Convert timestamps** from UTC to local time (`to_local_timestr()`).
2. **Format matchup titles** like `Michigan State at Minnesota`.
3. **Tag leagues** with short identifiers (e.g. `(NHL)`, `(CFB)`, `(MCH)`).
4. **Normalize broadcast info** using:
   - `CHANNEL_ALIASES` (to map ESPN’s broadcast names to your local channel names)
   - `EXCLUDE_STREAMING_KEYS` (to drop ESPN+, Peacock, etc.)
5. **Attach channel numbers** by looking up the current `CHANNEL_MAPS` profile.

Each event now has standardized columns:
[date, time_str, sport, tag, title, channel, channel_num, fav_row]


---

## 4. Aggregation Layer

The normalized events are flattened into a single pandas `DataFrame`, then restructured into a **grid** suitable for Excel:

- Columns: one per day (`Monday 11/03`, `Tuesday 11/04`, …)  
- Rows: one per channel number  
- Each cell: one or more stacked events (time, sport tag, matchup)

The script builds this grid through:
- Grouping by `(channel_num, channel, col_label)`
- Sorting by time within each cell
- Combining results into `events_by_cell`

Special rows for favorites (school + pro teams) are inserted at the top.

---

## 5. Excel Rendering Layer

Excel output is generated via **XlsxWriter** through pandas’ `ExcelWriter` interface.

### Sheet layout

| Area | Purpose |
|------|----------|
| Title row | Displays main heading (“Sports on TV”) |
| Subtitle | Shows date range (e.g. “Nov 3–7, 2025”) |
| Header row | Channel numbers and date columns |
| Grid body | One row per channel, one column per day |

### Page setup
- Letter size, landscape orientation  
- Margins: 0.2 inches  
- Fit-to-page: 1x1  
- Repeat header row when printing  
- Hidden gridlines for a clean look

### Cell formatting
- Alternating shading for readability  
- Rich text for each cell (colored by sport)  
- Dedicated style for the favorite school row  
- Centered headers and aligned text for visual consistency

Each day’s schedule fits on one printed page, requiring **no manual Excel adjustments**.

---

## 6. Logging & Debugging Layer

Two optional outputs help maintain configuration accuracy:

| File | Description |
|------|--------------|
| `events_flat_<profile>.csv` | Flat event table for debugging or analysis. |
| `unknown_broadcasts.csv` | List of broadcast strings that didn’t match any alias. |

By checking these, you can:
- See which events were fetched and where they map.
- Update `CHANNEL_ALIASES` to capture new or rare broadcast names.

---

## 7. End-to-End Flow Diagram

Below is a text-based diagram of the complete flow:
| Configuration         |
| --------------------- |
| START_DATE            |
| CHANNEL_MAPS          |
| SPORTS                |
| FAVORITE_SCHOOL       |


      |
      v

| ESPN API Fetch          |
| ----------------------- |
| fetch_day_sport_multi() |
| -> JSON events          |


      |
      v
| Normalization         |
| --------------------- |
| Convert timezones     |
| Format titles         |
| Map broadcasts        |
| Assign channels       |


      |
      v

| Aggregation           |
| --------------------- |
| Group by channel      |
| Sort by time          |
| Build grid dict       |


      |
      v

| Excel Rendering       |
| --------------------- |
| Title + Subtitle      |
| Grid cells w/         |
| rich text & color     |


      |
      v

| Final Excel Sheet     |
| --------------------- |
| Single-page view      |
| Print-ready           |



---

## 8. Future Improvements

Potential enhancements for the next iteration:

- CLI interface (`python listings.py --start 2025-11-03 --days 7 --profile "StoryPoint EL"`)
- Add optional PDF export
- Auto-refresh alias mappings via ESPN API logs
- Unit tests for API changes or missing fields
- Optional inclusion of streaming listings as a separate sheet

---

*See also:*  
- [README.md](../README.md) — Overview and usage instructions  
- [configuration.md](configuration.md) — Detailed configuration reference







