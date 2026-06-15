---
name: weather
description: This skill should be used when the user asks about the weather, "¿qué clima hace?", "¿cómo está el clima?", "dame el clima", "clima local", "weather", "temperatura", "llueve", "pronóstico", or any variation of weather/climate queries. Use it to fetch current weather conditions for the user's location or a specified city.
version: 1.0.0
---

# Weather Skill

Fetches current weather and forecast using the free [wttr.in](https://wttr.in) service — no API key required.

## How to Execute

### 1. Determine location

- If the user specified a city/location in their message, use that.
- If not, use `wttr.in` without a city — it auto-detects from IP.

### 2. Fetch weather data (JSON)

Use `WebFetch` with one of these URLs:

```
# Auto-detect location (no city given)
https://wttr.in/?format=j1

# Specific city
https://wttr.in/CITY?format=j1
```

Replace `CITY` with the URL-encoded city name (e.g., `Caracas`, `New+York`, `Buenos+Aires`).

### 3. Parse and display

From the JSON response (`weather[0]`), extract and show:

| Field | JSON path |
|---|---|
| Location | `nearest_area[0].areaName[0].value` + country |
| Current temp (°C) | `current_condition[0].temp_C` |
| Feels like (°C) | `current_condition[0].FeelsLikeC` |
| Description | `current_condition[0].weatherDesc[0].value` |
| Humidity % | `current_condition[0].humidity` |
| Wind km/h | `current_condition[0].windspeedKmph` + `winddir16Point` |
| Visibility km | `current_condition[0].visibility` |
| Today max/min | `weather[0].maxtempC` / `weather[0].mintempC` |

### 4. Optional: 3-day forecast

If the user asks for a forecast, iterate `weather[0..2]` and show date + max/min + description from `weather[N].hourly[4].weatherDesc[0].value`.

## Output Format

Present concisely in plain text, no markdown tables unless asked. Example:

```
Caracas, Venezuela — Partly Cloudy
Temp: 26°C (feels like 28°C)
Humidity: 72% | Wind: 18 km/h NE
Today: 22–29°C
```

## Error Handling

- If `WebFetch` fails or returns non-JSON, tell the user wttr.in may be temporarily unavailable and suggest trying again or visiting https://wttr.in directly.
- If the city is not found, wttr.in returns a 404-like response — ask the user to confirm the city name.
