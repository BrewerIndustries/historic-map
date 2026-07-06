# historic-map

Interactive Leaflet world map of ~448 historic and cultural landmarks — presidential
libraries/graves, founders' homes, battlefields, forts, national parks/monuments/memorials,
plus global sites (ancient ruins, cathedrals, castles, museums, volcanoes, safaris, UNESCO
World Heritage). Points span 27 category types, and are filterable by category, searchable,
sortable by distance from a typed origin, and each has a per-user visit log (Dan / Hannah).
The whole app is one static `index.html` — no build step, no backend.

## Tech
- **Single file:** `index.html` only (~1400 lines: `<style>` in head, one `<script>` at the bottom). README is empty.
- **Libraries (all via unpkg CDN):** Leaflet 1.9.4 + Leaflet.markercluster 1.5.3. No framework — vanilla JS, `var`-style ES5-ish code.
- **Basemap tiles:** Esri ArcGIS (Dark/Light gray + labels, Satellite) and CARTO Voyager (Streets), switched by an on-map control. See `TILE_DEFS`.
- **Geocoding:** OpenStreetMap **Nominatim** (`/search`, `countrycodes=us`) for the "distance from" origin box. Browser geolocation for "use my location".
- **Data:** all landmark data is a hardcoded JS array (`SITES`) inside `index.html`. Nothing is fetched at runtime except tiles + geocoding.

## Code layout (inside the one `<script>`, ~line 292 on)
- `TYPES` — map of 27 category keys → `{ label, color }`. `TYPE_GROUPS` — how those types are grouped into the sidebar filter sections.
- `SITES` — the data array. Each entry: `{ id, name, type, state, lat, lng, note, url }`. `state` is a US state code OR a country/region name (e.g. `'Italy'`, `'Moon'`). Optional `noPlot:true` marks off-map points (e.g. Apollo 11 Tranquility Base) that show in the list but not on the map.
- **Pre-process** (after `SITES`): builds `locGroups` keyed by `lat.toFixed(3)+','+lng.toFixed(3)` so co-located sites share one marker; sets `s._locKey` and `s._colocated`.
- **User + visit log:** `getCurrentUser`/`loginAs`/`switchUser` (login overlay, stored in `localStorage` key `historic-map-user`). Visits stored per user in `historic-map-visits-<name>` as `{siteId: dateString}`.
- **Map:** `initMap`, `setTile`, `makeGroupIcon` (single = solid dot, multi-type = conic-gradient pie), `buildGroupPopup`, `renderMarkers` (clusters + applies filters).
- **Filters:** `activeFilters` (a Set of active type keys), `setAllFilters`, `initFilters`, `setVisitFilter` (all / me / partner / neither / both).
- **List/sort:** `filteredSites`, `renderList`, `activateItem`; `haversine` for distance, `geocodeOrigin`/`useMyLocation` set the origin. `renderProgress` shows visit counts.
- **Boot:** everything wires up in the `DOMContentLoaded` handler at the bottom.

## Run locally
Just open `index.html` in a browser (CDN libs load over the network). A static server also
works: `python3 -m http.server` in this dir, then visit the printed URL.

## Deploy (GitHub Pages)
`.github/workflows/pages.yml` runs on **push to `main`** only: copies `index.html` into
`_site/`, writes a `CNAME` of `historic-map.dabrewer.dev`, and publishes via GitHub Pages.
- Prod: https://historic-map.dabrewer.dev/  •  Dev: https://historic-map.dabrewer.dev/dev/
- Registered in `.jarvis.json` (both prod and dev lanes = live); the Jarvis dashboard reads that file.

## Where to change what
- **Add/edit landmarks:** edit the `SITES` array. Give a unique `id`; set `type` to an existing `TYPES` key (or add a new one). Coords in decimal `lat`/`lng`. Use `noPlot:true` for anything not on Earth's surface.
- **Add a new category:** add a key to `TYPES` (label + hex color), then slot it into a `TYPE_GROUPS` section so it appears in the sidebar filter.
- **Styling/layout:** the single `<style>` block in `<head>`; theme colors are CSS variables in `:root`.
- **Behavior (filtering, sorting, popups, visit log):** the functions listed above.

## Git workflow
- Work on **`dev`**; push there freely.
- Promote to **`main`** ONLY via a PR the user approves. Never fast-forward or reset-push `main`.
- Only `main` triggers the Pages deploy.
- Keep this repo's **README updated** — it is currently empty; populating it with the summary above would help.

## Gotchas
- Markers are grouped by coordinates rounded to 3 decimals (~110 m). Two truly distinct sites at nearly the same spot will merge into one marker/popup — nudge a coordinate if they must be separate.
- `noPlot` sites are excluded from the map and from distance sorting but still appear in the list (rendered with an "off-map" tag).
- All data and visit state is client-side: `SITES` is baked into the file; visits/user live in `localStorage` (not synced between devices or users).
- Nominatim is rate-limited and hardcoded to US results (`countrycodes=us`) — origin search won't resolve non-US places.
- Tiles, Leaflet, and geocoding all require network access; opening the file fully offline yields a blank map.
