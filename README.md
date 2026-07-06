# Historic Map

An interactive world map of **~448 historic and cultural landmarks** — presidential
libraries and graves, founders' homes, battlefields, forts, national parks, monuments and
memorials, plus a global layer of ancient ruins, cathedrals, castles, museums, volcanoes,
safaris and UNESCO World Heritage sites. Filter by category, search by name, sort by
distance from where you are, and keep a personal **visit log** (Dan / Hannah) for every place.

Part of the Jarvis constellation — a self-contained "standalone" app: one `index.html`,
no build step, no backend.

- **Prod:** https://historic-map.dabrewer.dev/
- **Dev:** https://historic-map.dabrewer.dev/dev/

## Features

- **~448 landmarks across 27 categories** — each a colored dot on the map, colored by type.
- **Category filter** — a grouped sidebar (Presidential, Battlefields, Forts, National Parks,
  World: Ancient & Ruins, Sacred & Castles, Museums, UNESCO, …) toggling types on/off.
- **Search** — filter the list by name.
- **Sort by distance** — type a US place (or use your browser location) and the list sorts by
  how far each site is from you.
- **Visit log** — log in as Dan or Hannah and mark places visited with a date; filter to
  *me / partner / neither / both*. Stored per-user in `localStorage`.
- **Marker clustering** — nearby points cluster at low zoom (Leaflet.markercluster); co-located
  sites share one marker with a pie-slice icon.
- **Switchable basemaps** — Dark / Light / Satellite (Esri) and Streets (CARTO).

## Tech

- **Single file:** everything lives in `index.html` (`<style>` in the head, one `<script>` at
  the bottom). Vanilla JS — no framework, no build.
- **Leaflet 1.9.4 + Leaflet.markercluster 1.5.3**, loaded from the unpkg CDN.
- **Basemap tiles:** Esri ArcGIS (dark/light gray + labels, satellite) and CARTO Voyager.
- **Geocoding:** OpenStreetMap **Nominatim** for the "distance from" origin box.
- **State:** the current user and each person's visits are kept in `localStorage`
  (`historic-map-user`, `historic-map-visits-<name>`). Nothing is fetched at runtime except
  map tiles and geocoding.

## The data

All landmark data is a hardcoded JS array — **`SITES`** — inside `index.html`. Each entry:

```js
{ id:1, name:'Franklin D. Roosevelt Presidential Library & Museum', type:'library',
  state:'NY', lat:41.7714, lng:-73.9366, note:'First presidential library, opened 1941.',
  url:'https://www.fdrlibrary.org' }
```

- `type` must be one of the keys in the **`TYPES`** map (each has a label + color).
- `state` is a US state code *or* a country/region name (e.g. `'Italy'`, `'Moon'`).
- `url` is optional (empty string if none).
- `noPlot: true` marks off-map points (e.g. Apollo 11's Tranquility Base) — they show in the
  list but not on the map.

**To add or edit a site:** add/edit an object in `SITES` with a unique `id` and a `type` that
already exists in `TYPES`. **To add a new category:** add a key to `TYPES` (label + hex color),
then slot it into a `TYPE_GROUPS` section so it appears in the sidebar filter.

## Run locally

Just open `index.html` in a browser (the CDN libraries load over the network). Or serve it:

```bash
python3 -m http.server    # then open the printed URL
```

## Deploy (GitHub Pages)

`.github/workflows/pages.yml` runs on **push to `main`**: it copies `index.html` into the
Pages artifact, writes the `historic-map.dabrewer.dev` CNAME, and publishes to GitHub Pages.

## Workflow

Work on **`dev`**; push there freely. Promote to **`main`** only via a PR the user approves —
never a fast-forward or `git push origin dev:main`. Only `main` triggers the deploy.

## Gotchas

- **Coord grouping (~110 m):** markers group by `lat`/`lng` rounded to 3 decimals, so two
  distinct sites at nearly the same spot merge into one marker/popup — nudge a coordinate if
  they must stay separate.
- **`noPlot` sites** are excluded from the map and from distance sorting but still appear in
  the list (with an "off-map" tag).
- **US-only geocoding:** the origin search is hardcoded to US results (`countrycodes=us`), so
  non-US place names won't resolve.
- **Needs network:** tiles, Leaflet and geocoding all load over the network; opening the file
  fully offline yields a blank map.
