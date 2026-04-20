# Changelog

All notable changes to the Tactile Map Braille Editor.

## Unreleased

### Added

- **OSM integration.** Right-hand map panel with Leaflet + raster tiles. One-click `Fetch this view` button pulls all building polygons and named POIs in the current view via Overpass. Results cache to `localStorage` so repeat visits to the same area are instant with no network call.
- **Parallel Overpass racing.** `Promise.any` against five mirrors (`overpass.kumi.systems`, `overpass-api.de`, `lz4.overpass-api.de`, `z.overpass-api.de`, `overpass.osm.ch`) with two-round escalating timeout (20s, then 60s). Wins take the first successful response; losers are aborted. Per-mirror 5-minute cooldown on 403/429.
- **Bidirectional STL to OSM binding.**
  - Forward: select a rooftop on the 3D model, click a building on the map. Name, description, and braille code autofill.
  - Reverse: click a building on the map first, then click any rooftop. The pending OSM match attaches when the rooftop is selected.
  - Conflict prompt: a custom modal asks to confirm replacement when a region is already linked to a different OSM entity.
- **POI-inside-polygon name enrichment.** Many OSM buildings carry no `name` tag while the name lives on a POI node inside the polygon. The prefetch pulls both and uses point-in-polygon to attach the POI name to its enclosing building.
- **Touch Mapper-aligned classification.** Each labelled feature is tagged with `mainClass` (A linear / B areal / C buildings / D POIs / E boundaries), `subClass` (e.g. `C1_landmark`, `C2_public`, `D4_leisure_cultural`), a human category label (e.g. `Library building`, `Landmark building`), and a key-tags subset (`building`, `amenity`, `shop`, `tourism`, `historic`, `wikidata`, `wikipedia`, ...). Rules ported from `touch-mapper-master/converter/map_desc/map-description-classifications.json`.
- **Spatial hints.** Each OSM record gets a Touch Mapper-style position phrase: `In the north-east corner`, `touches west edge`, `covers 2.1% of view`. Computed against the Leaflet view bounds at attach time.
- **Nominatim search autocomplete.** Debounced 350ms live suggestions with keyboard navigation (up/down/enter/escape). Powered by `nominatim.openstreetmap.org/search`.
- **Auto-navigate on STL load.** The filename (minus `.stl`, underscores replaced with spaces) is fed to Nominatim, and the map pans to the matching coordinates automatically. Saves a search step.
- **UEB number indicator and letter indicator compliance.** `codeToBrailleCells` inserts `⠼` before digit runs and `⠰` before a-j letters following digits, so "S5" renders as `⠎⠼⠑` (3 cells) instead of the ambiguous `⠎⠑`.
- **Optional Nemeth-style compact digits.** Per-label checkbox switches to lowered-digit patterns (dots shifted to rows 2-3) that need no number indicator, saving one cell per digit run. `S5` becomes `⠎⠢` (2 cells). Trade-off: not strict UEB. Default off; author opts in when space is tight.
- **Review mode.** Dedicated button. Makes the STL base translucent and paints every braille dot with glowing gold material. Runs geometric checks on every label and paints problem dots red, listing issues in the detail panel:
  - `pierces base (Xmm below)` — dot geometry extends below the STL floor.
  - `overlaps #N` — two labels' rectangles share XY footprint.
  - `floating (no surface below)` or `floating (Xmm above surface)` — raycast down returns nothing or the hit is far below the dot.
  - `on map base (no building underneath)` — the dot rests on the ground region rather than a rooftop. Ground detection is now z-based: every region within 0.5mm of the mesh's lowest region and covering 5%+ of total faces is treated as ground, so dense downtown STLs no longer slip through the old area-ratio heuristic.
  - `buried under surface (Xmm above)` — raycast up finds a building rooftop above the dot's position.
  - `extends off map edge` — any sample point of the label rectangle falls off the mesh entirely.
  - `spans N regions` — the label rectangle's 8 edge samples (4 corners + 4 midpoints) hit more than one distinct region, meaning the label is straddling a rooftop/rooftop or rooftop/ground boundary. Fires whenever a label is wider or taller than the building it's trying to label.
- **Background click guard.** Clicking the giant connected map base no longer selects it. Mode hint turns red and the `+ Free Label` button flashes three times to suggest the correct path for open-ground labels.
- **Keyboard shortcuts.** `Esc` clears selection and pending OSM match. `Delete` or `Backspace` runs `clearLabel()` on the current selection. Ignored inside text inputs and while the confirm modal is open.
- **Clear All button** (dark red, in the toolbar). Asks for confirmation then removes every braille label and free label on the current STL in one step. Rooftops stay selectable, STL stays loaded — only the annotation layer is wiped.
- **Auto-reset on STL load.** Loading a new STL now fully purges any leftover dots, free labels, selections, pending OSM matches, OSM markers, and UID counter from the previous STL. Previous behavior could leave stale braille dots floating in the scene because their region IDs no longer matched anything.
- **Legend row highlight.** When a region is selected in the 3D view (including labelled regions via braille dot click), the matching row in the sidebar Labels table highlights amber and scrolls into view. Two-way link with the existing row-click-to-select.
- **Wikipedia / Wikidata / OSM links** in the sidebar detail for every attached OSM feature.
- **Unified Export button, three files per click.** One `Export` click downloads all three artifacts in a single batch: `{stl}_annotations.json` (the reimportable source of truth, also containing `legend_text` and `map_content_text` string fields for redundancy), `{stl}_legend.txt` (embosser-ready braille legend), and `{stl}_mapcontent.txt` (Touch Mapper-style grouped description for sighted readers). No more clicking three separate buttons and forgetting which one you missed.

### Changed

- **Braille dot geometry.** Full sphere cap replaced with upper hemisphere plus a flat circular disc underneath, so every dot is a watertight hemisphere sitting on the rooftop. Surface embed reduced from 0.3mm to 0.02mm.
- **Adaptive base thickening.** At STL export, the lowest point of any braille dot is compared to the STL's `boundingBox.min.z`. If the gap is below `BRAILLE_BASE_SAFETY_MM` (1mm), a bottom slab is added to guarantee 1mm of solid material beneath any dot. Idempotent across repeated merges of an already-annotated STL.
- **Status pill.** Replaced the noisy bottom status-bar text with an unobtrusive pill at the top-left of the map: grey dot when idle, orange pulsing pill with `Parsing map… ~Ns` countdown and a progress bar while fetching, green when ready, red on failure. Countdown estimate adapts to a rolling median of the last 5 fetch durations stored in `localStorage`.
- **Manual fetching.** Auto-fetch on map move was removed. Fetching is now fully manual via the `Fetch this view` button, so panning and zooming never consumes Overpass quota behind the user's back.

### Fixed

- **3D-print base piercing.** Earlier full-sphere dots pierced through thin rooftop shells and broke the bottom slab during printing. With hemisphere+disc dots, 0.02mm embed, and adaptive base thickening, this class of failure is eliminated.
- **Digit/letter ambiguity in braille.** Codes containing digits no longer render as letter-equivalent patterns (e.g. `S5` used to emboss identically to `SE`). UEB number indicator prefix is now inserted automatically; Nemeth lowered digits available as an opt-in.
- **Background accidental selection.** Clicking the map base no longer attaches labels or triggers OSM matching.
- **Overpass rate-limit thrashing.** Per-mirror cooldown on 403/429 responses prevents the client from repeatedly hammering a mirror that has already blocked it.
- **Garbage Nominatim fallback.** The click handler no longer falls back to `display_name.split(',')[0]` when no OSM `name` is present; that field is a postal address ("21250 Stevens Creek Blvd") and was filling the name field with street numbers.

### Removed

- `Legend TXT` and `Map Content` toolbar buttons (their content is now embedded inside the unified JSON bundle as `legend_text` and `map_content_text` fields).
- The Python aggregation script `scripts/map_content.py` (replaced by the in-browser `buildMapContentText` that writes the same format directly into the export JSON).
