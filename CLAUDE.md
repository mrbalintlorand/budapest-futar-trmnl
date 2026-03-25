# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A TRMNL private plugin displaying Budapest public transit departures from the BKK FUTÁR API. Fully serverless — no server, no build step. TRMNL polls the BKK API directly and renders Liquid templates on its e-ink display (800×480px, black & white).

## Architecture

**Data flow:**
1. TRMNL polls `plugin.yaml`'s `polling_url` using the user's `api_key` and `stop_id` custom fields
2. BKK API returns JSON; TRMNL passes it to the Liquid template as-is
3. Template renders to HTML; TRMNL displays on e-ink

**Files:**
- `plugin.yaml` — polling URL, custom fields definition
- `markup/full.html` — 780×460px, up to 15 rows
- `markup/half_horizontal.html` — 780×225px, up to 8 rows
- `markup/half_vertical.html` — 400×460px, up to 15 rows
- `markup/quadrant.html` — 400×235px, up to 8 rows

All four markup files share identical logic; only font sizes, gaps, badge widths, and row limits differ.

## BKK API Response Shape

```
currentTime                        Unix ms, ROOT level (not data.currentTime)
data.entry.stopTimes[]:
  .tripId                          key into data.references.trips
  .stopId                          key into data.references.stops
  .stopSequence                    integer, used for deduplication
  .stopHeadsign                    destination string
  .departureTime                   Unix seconds (scheduled)
  .predictedDepartureTime          Unix seconds (real-time); 0 = no RT data
data.references.trips[tripId]:
  .routeId                         key into data.references.routes
  .wheelchairAccessible            boolean (true/false)
data.references.routes[routeId]:
  .shortName                       e.g. "74"
  .type                            string enum: TRAM, SUBWAY, RAIL, SUBURBAN_RAILWAY,
                                   BUS, COACH, TROLLEYBUS, FERRY, FUNICULAR, GONDOLA,
                                   CABLE_CAR, BICYCLE, CAR, WALK, LOCAL, TRANSIT
data.references.stops[stopId]:
  .name                            e.g. "Ötvenhatosok tere (István utca)"
```

## Liquid Template Conventions

- Custom field values: `trmnl.plugin_settings.custom_fields_values.KEYNAME`
  - Boolean fields arrive as strings `"true"`/`"false"` — always compare with `== "true"`
- UTC conversion: `stopTime.departureTime | plus: trmnl.user.utc_offset` then `| date: "%H:%M"`
- `currentTime` (ms) needs `| divided_by: 1000` before adding UTC offset
- Single-stop detection: check `stop_id contains "&stopId="` — no reliable `size` filter on hashes
- TRMNL grayscale: use `bg--gray-75` (lightest) through `bg--gray-10` (darkest) CSS classes; raw hex `background:` colors do not render on e-ink device

## Trip Deduplication

When querying multiple stops, the same trip appears in `stopTimes` once per stop. The templates keep only the entry with the lowest `stopSequence` per `tripId` using a string-encoded map (`min_seqs`). This logic is identical in all four markup files.

## VS Code

`.vscode/settings.json` disables `html.validate.styles` to suppress false-positive CSS linter errors caused by Liquid `{{ }}` syntax inside `style=""` attributes.
