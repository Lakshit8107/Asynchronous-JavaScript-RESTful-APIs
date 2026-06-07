# WeatherGrid — Real-Time Weather Dashboard

A dynamic, AI-powered weather dashboard built to demonstrate asynchronous JavaScript patterns and RESTful API integration. Enter any city worldwide and get live weather metrics rendered instantly in a clean, dark-themed UI.

---

## Features

- **Real-time weather data** — temperature, humidity, wind speed, pressure, visibility, sunrise/sunset
- **5-day forecast** — daily high/low with condition icons
- **Async/await fetch pattern** — non-blocking API calls with proper error handling
- **Dynamic JSON rendering** — nested response objects parsed and mapped to live DOM elements
- **City search** — type any city name or use quick-select pills for popular cities
- **Animated UI** — humidity bar fills on load, smooth state transitions

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Vanilla JavaScript (ES2017+) |
| Async pattern | `async/await` with `fetch()` |
| Data source | Anthropic Claude API (`claude-sonnet-4-20250514`) |
| Styling | CSS custom properties, Google Fonts (Syne + Space Mono) |
| Icons | Emoji-based condition mapping |

---

## How It Works

### 1. Search Triggered

The user types a city name and presses Enter or clicks **Search**. This calls the `fetchWeather(city)` async function.

```js
async function fetchWeather(city) {
  if (!city.trim()) return;
  show('loading-state');
  // ...
}
```

### 2. API Request via Fetch

A structured prompt is sent to the Anthropic `/v1/messages` endpoint using `fetch()` with `async/await`. The prompt instructs the model to return only a valid JSON object containing weather data for the requested city.

```js
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [{ role: "user", content: prompt }]
  })
});
```

### 3. Error Handling (Three Layers)

**Layer 1 — Network errors:** Checks `response.ok` before attempting to parse.

```js
if (!response.ok) throw new Error('API request failed');
```

**Layer 2 — JSON parse errors:** Wraps `JSON.parse()` in its own `try/catch` to catch malformed responses.

```js
try { parsed = JSON.parse(clean); }
catch(e) { throw new Error('Could not parse weather data'); }
```

**Layer 3 — Semantic errors:** The model returns `{ "error": "City not found" }` for invalid city names, which is checked before rendering.

```js
if (parsed.error) {
  show('error-state');
  document.getElementById('error-text').textContent = `City not found: "${city}".`;
  return;
}
```

### 4. JSON Parsing & Rendering

The API response is a nested JSON object. Each field is extracted and injected into the DOM:

```js
// Flat fields
document.getElementById('temp-main').textContent = d.temperature + '°';
document.getElementById('humidity-val').textContent = d.humidity + '%';

// Nested array — 5-day forecast
(d.forecast || []).forEach(f => {
  const div = document.createElement('div');
  div.innerHTML = `
    <p>${f.day}</p>
    <p>${conditionIcon(f.condition)}</p>
    <p>${f.high}° / ${f.low}°</p>
  `;
  fr.appendChild(div);
});
```

---

## JSON Response Schema

The API returns the following structure:

```json
{
  "city": "London",
  "country": "United Kingdom",
  "condition": "Partly Cloudy",
  "temperature": 14,
  "feels_like": 11,
  "humidity": 72,
  "wind_speed": 23,
  "wind_direction": "SW",
  "visibility": 9,
  "pressure": 1008,
  "sunrise": "05:12 AM",
  "sunset": "09:04 PM",
  "forecast": [
    { "day": "Mon", "high": 15, "low": 9, "condition": "Cloudy" },
    { "day": "Tue", "high": 12, "low": 7, "condition": "Rain" },
    { "day": "Wed", "high": 16, "low": 10, "condition": "Sunny" },
    { "day": "Thu", "high": 13, "low": 8, "condition": "Overcast" },
    { "day": "Fri", "high": 11, "low": 6, "condition": "Drizzle" }
  ]
}
```

---

## UI States

The dashboard manages four distinct UI states, toggled via the `show(id)` helper:

| State | When shown |
|---|---|
| `empty-state` | Initial load, before any search |
| `loading-state` | While awaiting API response |
| `weather-content` | Successful data render |
| `error-state` | Network failure or invalid city |

---

## Key Concepts Demonstrated

**`async/await`** — Modern syntax for handling Promises, keeping async code readable and sequential without callback nesting.

**`fetch()` API** — The browser-native way to make HTTP requests, replacing older `XMLHttpRequest`.

**Error propagation** — Errors thrown inside `async` functions bubble up to the nearest `catch` block, enabling clean centralized error handling.

**Dynamic DOM rendering** — JSON arrays (forecast) are iterated with `.forEach()` to programmatically create and append HTML elements.

**State machine UI** — Four named states managed by a single `show()` function, preventing conflicting UI conditions.

---

## Quick Cities

The dashboard ships with 8 preset cities for instant demo:

`London` · `Tokyo` · `New York` · `Sydney` · `Mumbai` · `Paris` · `Dubai` · `Toronto`

---

## Notes

- Weather values are AI-generated based on realistic climate profiles for each city and current season. Values reflect typical conditions, not a live meteorological feed.
- The model returns `{ "error": "City not found" }` for unrecognised or fictional city names.
- Requires an active Anthropic API session to function (handled automatically within Claude.ai artifacts).
