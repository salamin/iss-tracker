# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

Zero-dependency vanilla JS app — no build step required.

```bash
python3 -m http.server 8765
# Open http://localhost:8765
```

> The app must be served over HTTP (not opened as a file://) because the ISS API (`http://api.open-notify.org/iss-now.json`) is HTTP-only, and browsers block mixed-content requests from `file://` origins.

## Architecture

Single-file application: all HTML, CSS, and JS lives in `index.html`.

**Data flow:**
1. On load, `mapReady` (a Promise) resolves once the NASA Blue Marble background image loads
2. Canvas is sized to the viewport, then `tick()` fires immediately
3. `tick()` calls `fetchISS()` → appends `{lat, lon, ts}` to a `positions` array (capped at 90 entries) → calls `updatePanel()`
4. `setInterval(tick, 5000)` polls the API every 5 seconds
5. A separate `requestAnimationFrame` loop (`animate()`) continuously advances `pulsePhase` and calls `drawMap(positions)`, which renders the map background, the fading trail dots, and the pulsing ISS marker

**Coordinate projection** (`project(lat, lon)`): equirectangular — `x = (lon + 180) / 360 * W`, `y = (90 - lat) / 180 * H`.

**Trail:** last 90 positions (~7.5 min at 5s intervals), rendered as small dots with opacity fading from 15% (oldest) to 70% (newest).

**Status badge states:** `"fetching"` (grey, default) → `"live"` (green, on success) → `"error"` (red, on fetch/parse failure; map holds last known position).
