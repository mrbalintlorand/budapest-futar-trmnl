# Budapest Futár — TRMNL Private Plugin

Displays live Budapest public transit departures on your TRMNL e-ink display, powered by the [BKK FUTÁR API](https://opendata.bkk.hu/).

Monitor trams, buses, metros, and more — with real-time arrival predictions, route badges, and optional wheelchair accessibility indicators.

---

## Features

- Real-time departure times with predicted arrival data
- Route badges for all BKK transit types (tram, bus, metro, suburban rail, trolleybus, ferry, and more)
- Multi-stop support — combine multiple stops into a single view
- Optional vehicle type emoji (🚋 🚌 🚇 etc.)
- Optional wheelchair accessibility indicator (♿)
- Optional stop name shown per departure row
- Deduplication across stops — the same trip only appears once
- Four layout sizes to fit any TRMNL grid configuration

---

## Setup

### 1. Get a BKK API Key

Register for a free API key at [opendata.bkk.hu](https://opendata.bkk.hu/).

### 2. Find Your Stop ID(s)

Go to [futar.bkk.hu](https://futar.bkk.hu/) and hover over any stop on the map. The stop ID appears in the URL or tooltip as e.g. `#F02297`. Prefix it with `BKK_` → `BKK_F02297`.

### 3. Create a Private Plugin in TRMNL

1. In your TRMNL dashboard, go to **Plugins → Private Plugin**
2. Select the **Polling** strategy
3. Set the polling URL:

```
https://futar.bkk.hu/api/query/v1/ws/otp/api/where/arrivals-and-departures-for-stop.json?stopId={{ stop_id }}&minutesBefore=0&minutesAfter=45&limit=20&includeReferences=compact&key={{ api_key }}
```

4. Paste the Liquid markup from one of the files in the `markup/` folder into the template editor (see [Layouts](#layouts) below)
5. Fill in your **BKK API Key** and **Stop ID(s)** in the plugin's custom fields

---

## Custom Fields

| Field | Required | Description |
|---|---|---|
| `api_key` | Yes | Your API key from [opendata.bkk.hu](https://opendata.bkk.hu/) |
| `stop_id` | Yes | Stop ID(s) — see format below |
| `show_type_emoji` | No | Show a vehicle type emoji next to each route badge |
| `show_accessible_emoji` | No | Show ♿ indicator for wheelchair-accessible trips |
| `show_stop_name` | No | Show the stop name in each departure row |

### Single Stop

```
BKK_F02297
```

### Multiple Stops

Append additional stop IDs using `&stopId=`:

```
BKK_F02297&stopId=BKK_F02298&stopId=BKK_F02299
```

This passes them as separate query parameters to the BKK API, which returns combined departures for all stops. When multiple stops are configured, duplicate trips (same vehicle serving multiple stops in sequence) are automatically deduplicated — only the earliest stop in the trip is shown.

---

## Layouts

Choose the markup file that matches your TRMNL tile size:

| File | Tile size | Max rows |
|---|---|---|
| `markup/full.html` | 780×460 px (full screen) | 15 |
| `markup/half_horizontal.html` | 780×225 px | 8 |
| `markup/half_vertical.html` | 400×460 px | 15 |
| `markup/quadrant.html` | 400×235 px | 8 |

All four layouts share identical logic — only font sizes, spacing, badge widths, and row limits differ.

---

## How It Works

TRMNL polls the BKK FUTÁR API directly (no server or build step required). The API response is passed as-is to the Liquid template, which renders departure rows for the e-ink display.

Departure times shown are real-time predicted times when available, falling back to scheduled times otherwise. All times respect your TRMNL account's UTC offset setting.

---

## License

MIT
