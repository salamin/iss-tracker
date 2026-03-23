# ISS Tracker - Task List

## T1 — HTML skeleton
Create `index.html` with:
- Standard HTML5 boilerplate (`<!DOCTYPE html>`, `<meta charset>`, `<meta viewport>`, `<title>`)
- A `<canvas id="map">` element filling most of the viewport
- An info panel `<div>` containing placeholders for latitude, longitude, and UTC timestamp
- A status badge `<span>` with three possible states: "Fetching…", "Live", "Error — retrying"

## T2 — CSS layout and styling
Inside a `<style>` block in `index.html`:
- Dark background, full-viewport layout
- Canvas sized to fill available space, centered
- Info panel positioned as an overlay (bottom-left corner)
- Status badge color-coded: grey (fetching), green (live), red (error)
- Pulsing keyframe animation (`@keyframes pulse`) for the ISS marker dot

## T3 — Map background image
- Identify a suitable public-domain equirectangular world map PNG (e.g., Natural Earth 110m raster)
- Reference it via a reliable public URL (no local file needed)
- In JS, create an `Image` object, set its `src`, and wait for `onload` before drawing

## T4 — Canvas rendering function
Write a `drawMap(positions)` function that:
- Clears the canvas
- Draws the world map image as the canvas background (scaled to canvas dimensions)
- Implements the coordinate conversion: `x = (lon + 180) / 360 * W`, `y = (90 - lat) / 180 * H`
- Draws the trail: the last 90 positions as small semi-transparent dots, fading older ones
- Draws the current ISS position as a bright dot (e.g., red/orange, radius 6 px)
- Applies the pulse animation effect to the current position marker via canvas arc + globalAlpha

## T5 — ISS data fetch function
Write a `fetchISS()` async function that:
- Calls `fetch('http://api.open-notify.org/iss-now.json')`
- Parses the JSON response
- Extracts `iss_position.latitude`, `iss_position.longitude`, `timestamp`
- Returns the parsed data or throws on error
- Sets the status badge to "Fetching…" before the call and "Live" on success

## T6 — Error handling
In `fetchISS()` and its caller:
- Catch network errors and JSON parse errors
- Set the status badge to "Error — retrying" on failure
- Keep the last known position on the map (do not blank the canvas on error)
- Log the error to `console.error` for debugging

## T7 — Position history and auto-refresh
- Declare a `positions` array (max length 90) in module scope
- After each successful fetch, push the new position and trim the array if needed
- Start a `setInterval` loop calling `fetchISS()` every 5 000 ms
- Call `fetchISS()` once immediately on page load (don't wait 5 s for the first update)
- After each fetch (success or error), call `drawMap(positions)`

## T8 — Info panel update
Write an `updatePanel(lat, lon, timestamp)` function that:
- Formats latitude as `12.3456° N/S` and longitude as `12.3456° E/W`
- Converts the Unix timestamp to a human-readable UTC string
- Updates the relevant DOM elements with these values

## T9 — Responsive canvas resize
- Listen for the `resize` event on `window`
- On resize, update `canvas.width` and `canvas.height` to match the new viewport size
- Re-call `drawMap(positions)` immediately after resize to avoid a blank canvas

## T10 — CORS note in code
- Add a comment block near the `fetch` call explaining the HTTP-only constraint
- Document how to swap in a CORS proxy URL if the page is served over HTTPS

## T11 — End-to-end smoke test
- Open `index.html` in a browser served over HTTP (e.g., `python3 -m http.server`)
- Verify the map loads and the ISS dot appears within 5 s
- Verify the info panel updates on each refresh cycle
- Verify the trail grows over time
- Verify the error state by temporarily breaking the URL and confirming the badge shows "Error — retrying"
