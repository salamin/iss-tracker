# ISS Tracker - Implementation Plan

## Goal
A single-page web app (vanilla JS, no frameworks) that shows the ISS current position on an interactive globe, auto-refreshing in real time.

---

## API

- **Endpoint:** `http://api.open-notify.org/iss-now.json`
- **Method:** GET
- **Response:**
  ```json
  {
    "message": "success",
    "timestamp": 1454357342,
    "iss_position": {
      "latitude": -19.78,
      "longitude": -72.29
    }
  }
  ```
- **Note:** The endpoint is HTTP-only. To avoid mixed-content browser errors, the page must be served over HTTP (not HTTPS), or a CORS proxy must be used.

---

## File Structure

```
demo1/
├── index.html      # Single file containing HTML + CSS + JS
```

A single `index.html` file — no build step, no dependencies to install.

---

## Implementation Steps

### 1. HTML skeleton
- Standard HTML5 boilerplate
- A `<canvas>` element for rendering the globe
- A small info panel (lat/lon + timestamp)
- A status indicator (loading / live / error)

### 2. Globe rendering (Canvas 2D)
- Draw an equirectangular (flat) world map using the Canvas 2D API
- Overlay a marker (blinking dot) at the ISS coordinates, converted from lat/lon to canvas pixel position
- Optionally draw the ground track (last N positions)

  **Coordinate conversion:**
  ```
  x = (longitude + 180) / 360 * canvasWidth
  y = (90 - latitude)  / 180 * canvasHeight
  ```

  A free, license-friendly world map image (Natural Earth or similar SVG/PNG) will be loaded as the background for the canvas.

### 3. Data fetching
- Use the `fetch()` API to call `http://api.open-notify.org/iss-now.json`
- Parse the JSON and extract `iss_position.latitude`, `iss_position.longitude`, and `timestamp`
- Handle network errors gracefully with a visible error message

### 4. Auto-refresh
- Poll the API every **5 seconds** using `setInterval`
- Store the last ~90 positions in memory to draw a trail on the map
- Update the info panel with each new reading

### 5. UI / UX details
- Responsive canvas that scales to the viewport
- ISS marker: bright colored dot with a pulsing CSS animation
- Info panel shows: latitude, longitude, UTC timestamp
- Status badge: "Fetching…" / "Live" / "Error — retrying"

---

## Libraries / Dependencies

| Purpose | Choice | Reason |
|---|---|---|
| Globe/map rendering | Native Canvas 2D API | No framework requirement |
| HTTP requests | Native `fetch()` | Built into all modern browsers |
| Map background image | Public-domain PNG (Natural Earth) | Free, simple to load onto canvas |

No npm, no bundler, no external JS libraries.

---

## CORS Consideration

The Open Notify API does not enforce HTTPS. If the HTML file is opened directly from disk (`file://`) or served over plain HTTP, `fetch()` will work fine. If the page ends up served over HTTPS, the browser will block the mixed-content HTTP request. In that case the plan is to use a lightweight CORS proxy (e.g., `https://corsproxy.io`) as a fallback, noted as a comment in the code.

---

## Open Questions / Your Review

1. **Map style** — flat equirectangular map (simple) vs. a WebGL 3D globe (more impressive but adds complexity)?
2. **Trail length** — show the last 90 positions (~7.5 min of orbit) or a configurable length?
3. **Refresh rate** — 5 s feels responsive; ISS moves ~7.7 km/s so 5 s ≈ 38 km between readings. Is that acceptable?
4. **CORS fallback** — use a public proxy automatically, or just document the HTTP-only constraint?
