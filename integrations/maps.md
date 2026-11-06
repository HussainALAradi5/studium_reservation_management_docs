# Maps and location

## Decision

| Option | Cost | Key needed | Chosen |
| --- | --- | --- | --- |
| Google Maps JavaScript API | Billing account required (monthly free credit, then paid) | yes | no |
| Google Maps Embed / URL links | Free | no (URLs) | yes, for "Open in Google Maps" and "Directions" |
| Leaflet + OpenStreetMap tiles | Free, open source | no | yes, for picking and showing the location |

Leaflet `1.9.4` is loaded from `unpkg.com` only on pages that show a map (stadium form, stadium details), and the module that uses it is imported lazily.

## Picking a location (manager/admin)

`assets/javascripts/stadiums/map-picker.js`

1. The stadium form shows a map centred on the saved coordinates, or on the default centre (`MAP_DEFAULT_LAT`, `MAP_DEFAULT_LNG`, `MAP_DEFAULT_ZOOM` from `.env`, Bahrain by default).
2. Click the map or drag the marker → hidden `latitude` / `longitude` inputs update (7 decimals).
3. "Use my location" asks the browser geolocation API (HTTPS or localhost only).
4. "Clear" removes the coordinates (they are optional).
5. The server validates ranges and that both values are present together.

## Showing a location (everyone)

`assets/javascripts/stadiums/map-view.js` renders a read-only map with a marker. Links:

- Open: `https://www.google.com/maps/search/?api=1&query={lat},{lng}`
- Directions: `https://www.google.com/maps/dir/?api=1&destination={lat},{lng}`

## Tile usage policy

OpenStreetMap's public tile server is for light use and requires attribution, which Leaflet shows. For production traffic, switch `MAP_TILE_URL` in `.env` to a tile provider with a plan.
