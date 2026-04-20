# Tactile Map Braille Editor

A browser-only companion to [Touch Mapper](https://touch-mapper.org/) that adds UEB braille labels to tactile-map STL files. Touch Mapper generates 3D-printable maps from OpenStreetMap data but leaves them text-free. This editor fills that gap: load an STL, let the built-in OSM panel match buildings for you, and export a print-ready STL with embedded braille dots plus a self-contained annotations JSON.

**Fully client-side.** No server, no Python, no install. Open `index.html` directly or host on GitHub Pages.

## Quick start

1. Open `index.html` in any modern browser.
2. `Load STL` (or drag-and-drop). The map panel auto-navigates to the region matching the filename.
3. Click `Fetch this view` to pull OSM buildings for the current map view. The pill at the map's top-left turns green when ready.
4. Label buildings:
   - Click a rooftop in the 3D view, then click its building on the map.
   - Or click the map first, then click the rooftop. Either order works.
   - The sidebar autofills. Tweak rotation, scale, or braille code if needed. `Save Label`.
5. `Review` mode to audit your work. Red dots flag problems.
6. `Export` to download three files in one click: the reimportable JSON bundle plus a legend `.txt` and a map-content `.txt`. `Download Merged STL` to get the print-ready mesh.

## Highlights

- **OSM integration.** Embedded Leaflet map with one-click `Fetch this view`. Pulls building polygons and named POIs via a racing Overpass client (5 mirrors, 20s and 60s rounds), caches to `localStorage` so repeat visits are instant.
- **Bidirectional auto-labelling.** Click a building on the map and a rooftop on the 3D model in either order. English name, description, and UEB braille code autofill. No manual typing per building.
- **Touch Mapper-aligned metadata.** Every annotation carries a rich `osm` record: `name`, precise lat/lon, functional classification (A-E main class with sub-classes like `C1_landmark`, `C2_public`, `D4_leisure_cultural` / "familiar places"), spatial description ("In the north-east corner, covers 2.1% of view"), Wikipedia / Wikidata / OSM links, and selected OSM tags.
- **UEB-correct braille.** Number indicator `⠼` and letter indicator `⠰` are inserted automatically so codes with digits ("S5", "PE2") can't be misread as letter equivalents. Optional Nemeth-style compact digits for tight rooftops.
- **Review mode.** Translucent base + glowing gold dots for a clean audit view, with automatic detection of five geometric problems: pierces the base, overlaps an adjacent label, floats above nothing, rests on map base (no building underneath), or is buried under a rooftop.
- **Print-safe geometry.** Hemisphere dots with a flat base disc (no lower-hemisphere piercing) and adaptive base thickening guarantee at least 1mm of solid material beneath every dot.
- **One-click export.** A single `Export` button produces all three artifacts at once: a reimportable annotations JSON (with the full label data plus `legend_text` and `map_content_text` string fields baked in for redundancy), a standalone `_legend.txt` ready for a braille embosser, and a standalone `_mapcontent.txt` grouped by Touch Mapper classification for a sighted companion.

## Controls

### Mouse
| Action | What it does |
|---|---|
| Left drag | Orbit |
| Scroll | Zoom |
| Right drag | Pan |
| Click rooftop | Select region |
| Click building on map | Attach OSM data to selected region (or queue for next rooftop click) |
| Click braille dots | Select that label for editing |
| Drag braille dots | Reposition label, snaps to top surface |

### Keyboard
| Key | Action |
|---|---|
| `Esc` | Clear selection and any pending OSM match |
| `Delete` / `Backspace` | Delete the current label (ignored while typing in inputs) |
| `Enter` (in confirm modal) | Accept replace |
| `Esc` (in confirm modal) | Keep current |

### Toolbar

| Button | What it does |
|---|---|
| `Load STL` | Open an STL file picker |
| `Top View` | Camera snaps to bird's-eye |
| `+ Free Label` | Click anywhere on the mesh to place a label (for plazas, parks, open ground) |
| `Review` | Enter audit mode (translucent base, glowing dots, red flags on problems) |
| `Clear All` | Remove every label from the current STL. Confirms first. |
| `Import Labels` | Restore an `*_annotations.json` from a previous session |
| `Export` | Download three files at once: `_annotations.json` (reimportable bundle), `_legend.txt`, `_mapcontent.txt` |
| `Download Merged STL` | Export the print-ready STL with braille dots merged into the mesh |



## Export bundle format

`Export` downloads three sibling files per click:

- **`{stl}_annotations.json`** — structured source of truth, reimportable.
- **`{stl}_legend.txt`** — plain-text braille legend table for an embosser.
- **`{stl}_mapcontent.txt`** — grouped description organised by Touch Mapper classification.

The `legend_text` and `map_content_text` fields are also embedded inside the JSON so one file alone is enough if the other two get lost. JSON shape:

```jsonc
{
  "stl_file": "UCSC_East_Fields.stl",
  "generated_at": "2026-04-19T16:00:00.000Z",
  "uid_prefix": "a", "next_uid": 42,
  "blocks": [
    {
      "id": 7, "uid": "a12",
      "cx": 28.4, "cy": -7.3, "z": 3.50,
      "w": 14, "h": 7, "x_min": 21.4, "x_max": 35.4,
      "y_min": -10.8, "y_max": -3.8, "area": 2,
      "name": "Visual and Performing Arts Center",
      "code": "VPA",
      "description": "Visual and Performing Arts Center",
      "braille": "⠧⠏⠁",
      "compactDigits": false, "rotation": 0, "scale": 1.0,
      "osm": {
        "lat": 37.321752, "lon": -122.043705,
        "osm_id": 135174113, "osm_type": "way",
        "name": "Visual and Performing Arts Center",
        "source": "cache",
        "mainClass": "C", "subClass": "C2_public",
        "category": "University building",
        "keyTags": { "building": "university", "name": "..." },
        "spatial": {
          "position": "In the north-east corner",
          "edge": null,
          "coveragePct": 2.13,
          "viewBounds": { "south": 37.319, "west": -122.047, ... }
        }
      }
    }
  ],
  "legend_text": "Braille Legend: UCSC_East_Fields\n==========================...",
  "map_content_text": "Map content: UCSC_East_Fields\n============...\nPublic buildings\n----------------\n  Visual and Performing Arts Center (University building)  [VPA]\n    In the north-east corner · covers 2.1% of view · GPS 37.32175, -122.04371\n..."
}
```

- `blocks[]` is the source of truth. Re-import reads this and nothing else.
- `legend_text` is a ready-to-print ASCII table of code / braille / name. Paste into any UEB embosser input.
- `map_content_text` is a Touch Mapper-style grouped description organised by the classification hierarchy, useful as a sighted-reader reference alongside the tactile map.

## Braille dot specs

| Parameter | Default (scale = 100%) |
|---|---|
| Dot diameter | 1.6 mm |
| Dot height above surface | 0.8 mm |
| Dot spacing within a cell | 2.5 mm |
| Cell gap | 6.0 mm |
| Surface embed | 0.02 mm (just enough for fusion) |
| Base safety margin | 1.0 mm minimum below any dot |

Dot geometry is an upper hemisphere with a flat disc cap on the bottom, sitting on the rooftop. No lower hemisphere to pierce thin shells.

Defaults meet UEB tactile-graphics specifications. The per-label `Size` slider scales from 50% to 200%.

## Print settings

- Layer height: 0.3mm
- Infill: 20%
- Supports: not needed
- Material: PLA or PETG

## How it works

1. **Region detection.** On STL load, the mesh is scanned for connected flat faces grouped by height continuity. The largest strongly dominant region is flagged as the map base.
2. **OSM fetch and caching.** `Fetch this view` sends one Overpass query (buildings plus named POIs in the current map bbox) to multiple mirrors in parallel via `Promise.any`. The winning response populates an in-memory cache that is mirrored into `localStorage` keyed by bbox, so revisits are instant.
3. **POI-inside-polygon enrichment.** Many OSM buildings have no `name` tag because the name lives on a POI node placed inside the building polygon. A point-in-polygon pass links each unnamed building to its enclosing POI's name.
4. **Touch Mapper classification.** A tag-based rule set ported from `touch-mapper-master/converter/map_desc/map-description-classifications.json` assigns each feature a main class (A-E), sub-class, and human category.
5. **Braille rendering.** `codeToBrailleCells` walks the code string and inserts `⠼` and `⠰` indicators per UEB rules. `createBrailleMesh` renders each cell as a cylinder stem plus hemisphere cap plus flat base disc, assembled into a Three.js `Group` per label.
6. **STL synthesis.** On `Download Merged STL`, the label groups are cloned into a fresh scene alongside the original geometry and the adaptive base slab, then serialized to binary STL via Three.js `STLExporter`.

Everything runs in the browser. Three.js and Leaflet are loaded from CDN. No npm, no build step.

## Files

| Path | Purpose |
|---|---|
| `index.html` | Single-page editor. All code inline, no build. |
| `examples/` | Sample STL and annotations JSON files. |
| `CHANGELOG.md` | Per-release notes. |
| `touch-mapper-master/` | Vendored Touch Mapper source, for reference and classification rules. |

## Limitations

- **Overpass latency.** Public Overpass mirrors are rate-limited and sometimes slow. Fetching is manual for this reason. A 5-15 minute cooldown kicks in after 403/429 responses; pre-generating a static GeoJSON bundle per map area is the reliable alternative for heavy use.
- **Vector-tile basemaps not wired in.** The map uses OSM raster tiles, so building names on the basemap are pixels, not queryable data. All name/ID lookups go through Overpass and Nominatim.
- **Spatial hint reference.** The `osm.spatial` position phrase is computed against the Leaflet view bounds at attach time. If you pan the map drastically between attaching and reviewing, the recorded phrase still refers to the original view.

## Dependencies

None at build time. At runtime, loaded from CDN:

- Three.js 0.164.1 (3D viewer, STL loader/exporter)
- Leaflet 1.9.4 (OSM map panel)

## License

See `LICENSE`.
