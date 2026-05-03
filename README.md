# Data Engineering competition

This repository contains a solution for the Data Engineering competition.  
The task is to process raw event data (registrations, session pings, match starts and finishes) along with map metadata, clean it, reconstruct missing information, and build a small match history system. Finally, the data is exposed through a simple REST API and visualized.

**Author:** Djordje Mirosavic

---

## What this notebook does

1. **Loads raw data** – reads `events.jsonl` and `maps.jsonl` files.
2. **Cleans the data**
   - Removes duplicate events (same `id` or same data except `id`).
   - Fills missing `user_id` values using paired match events (`match_start` / `match_finish`).
   - Checks that every type matches with provided description
   - Checks if every event type contains its required fields; missing fields are set to `None`.
   - Removes events that cannot be reconstructed at all (e.g., incomplete match data).
3. **Reconstructs match results**
   - Pairs `match_start` and `match_finish` events for the same game.
   - Computes outcomes from both players perspectives.
   - Builds a complete table of played matches with player names, map, duration, and winner.
4. **Builds a SQLite database** (`match_history.db`) with three tables:
   - `matches` – every played game (both players, map, start/finish time, outcome).
   - `daily_map_leaders` – daily statistics per map: average playtime, match count, and the best player up to that date.
   - Another table for user summaries is generated in memory and served via the API.
5. **Provides a Flask API** with three endpoints:
   - `GET /user-stats` – returns stats for all users (or filtered by country).
   - `GET /map-stats/<map_name>` – daily stats for a specific map, with optional date range.
   - `GET /maps` – lists all available map names.
6. **Visualises** match counts per map over the last 7 days using Plotly.

---

## How to run

The notebook is designed to run in **Google Colab**. You do not need to install anything locally.

### Steps

1. Open the notebook in Colab.
2. Upload the input files when prompted (the first two code cells ask for `events.jsonl` and `maps.jsonl`).
3. Run all cells in order (Runtime → Run all).
   - The Flask server starts in the background thread after the API definition.
4. Once all cells have executed, the API is available at `http://localhost:5000`.

### Making API requests (from inside Colab)

The notebook already includes test calls. You can modify them or add new ones. All requests use the `requests` library.

```python
import requests

# Get stats for all users
requests.get("http://localhost:5000/user-stats").json()

# Filter users by country
requests.get("http://localhost:5000/user-stats?countries=SRB,DEU").json()

# Get daily stats for a map (e.g., "Cobblestone")
requests.get("http://localhost:5000/map-stats/Cobblestone").json()

# With date range
requests.get("http://localhost:5000/map-stats/Cobblestone?date_from=2026-04-03&date_to=2026-04-04").json()

# List all maps
requests.get("http://localhost:5000/maps").json()
```

## Visualisation

Creates a line chart of match counts per map over time. It uses the `/map-stats` endpoint to fetch data.
