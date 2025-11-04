```markdown
# Project Diagrams

This page shows how the **Weekly** and **Single-Day** Sports TV Listing pipelines relate to each other.

Both workflows share the same core steps (configuration → ESPN fetch → normalization → Excel export), but diverge in how they handle dates and layout.

---

## 1. Weekly vs Single-Day Pipelines (Side by Side)

```mermaid
flowchart TB

  %% ==== WEEKLY PIPELINE ====
  subgraph W[Weekly Listings Pipeline]
    W1[Config:\nSTART_DATE / NUM_DAYS\nCHANNEL_MAPS\nSPORTS\nFAVORITES\nLOCAL_TZ]
    W2[Generate DAY_DATES\n(e.g. Mon–Fri)]
    W3[Loop over days × sports\nFetch ESPN scoreboard JSON]
    W4[Normalize events:\nUTC → local time\nmatchup titles\nbroadcast → channels]
    W5[Build multi-day grid:\n(channel, day) cells\nsorted by start time]
    W6[Render Excel sheet:\nOne column per day\nFit to 1 page landscape]
  end

  %% ==== SINGLE-DAY PIPELINE ====
  subgraph D[Single-Day Listings Pipeline]
    D1[Config:\nTARGET_DATE\nCHANNEL_MAPS\nSPORTS\nFAVORITES\nLOCAL_TZ]
    D2[Use single date\n(TARGET_DATE only)]
    D3[Loop over sports\nFetch ESPN scoreboard JSON]
    D4[Normalize events:\nUTC → local time\nmatchup titles\nbroadcast → channels]
    D5[Build single-day grid:\nOne column, many rows\nsorted by start time]
    D6[Render Excel sheet:\nHigh-density single day\nFit to 1 page landscape]
  end

  %% ==== COMMON FLOW HIGHLIGHT ====
  classDef shared fill=#e0f7ff,stroke=#0077aa,stroke-width:1px;
  classDef normal fill=#ffffff,stroke=#999,stroke-width:1px;

  class W3,W4,W5,W6,D3,D4,D5,D6 normal;
  class W3,W4,D3,D4 shared;

  %% Align steps roughly in rows
  W1 --> W2 --> W3 --> W4 --> W5 --> W6
  D1 --> D2 --> D3 --> D4 --> D5 --> D6

flowchart LR

  subgraph Core[Shared Core]
    C1[Configuration\n(date(s), channels, sports,\nfavorites, timezone)]
    C2[ESPN Scoreboard Fetch\n(one call per sport/league)]
    C3[Normalization\nUTC → local time\nmatchup titles\nbroadcast → channel map]
  end

  subgraph Weekly[Weekly Layout]
    Wg1[Generate DAY_DATES\n(start + N days)]
    Wg2[Build grid by\n(channel, day)]
    Wg3[Excel: multi-day sheet\n1 column per day]
  end

  subgraph Daily[Single-Day Layout]
    Dg1[Use single TARGET_DATE]
    Dg2[Build grid by\n(channel only)]
    Dg3[Excel: single-day sheet\n1 column, more rows]
  end

  C1 --> C2 --> C3
  C3 --> Wg1 --> Wg2 --> Wg3
  C3 --> Dg1 --> Dg2 --> Dg3

