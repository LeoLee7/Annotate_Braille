# Tactile Map Braille Editor

Add braille labels to 3D-printed tactile maps (Touch Mapper STL files) so blind users can identify buildings by touch.

**Fully client-side.** No server, no Python, no install. Host on GitHub Pages or open `index.html` directly.

## Live Demo

Open `index.html` in any modern browser, or host via GitHub Pages.

## Workflow

### 1. Load a tactile map

Click **Load STL** or drag-and-drop an STL file. The editor renders the mesh in 3D and automatically detects building regions (connected flat surfaces).

### 2. Label buildings

1. **Click a building rooftop** to select it (highlights yellow)
2. Type **Braille Code** (e.g. "ADM"), **English Name**, and **Description** in the sidebar
3. Adjust **Rotation** (0 to 360) and **Size** (50% to 200%) sliders as needed
4. Click **Save Label**. White braille dots appear on the surface.
5. Use the OSM map on the right for geographic reference.
6. Repeat for all buildings.

Controls:
- **Left drag**: orbit
- **Scroll**: zoom
- **Right drag**: pan
- **Top View**: snap to bird's-eye view
- **+ Free Label**: click anywhere to place a label at an arbitrary point
- **Click braille dots**: select that label for editing
- **Drag braille dots**: reposition the label (snaps to top surface)

### 3. Export

- **Download Merged STL**: merges all braille dots into the STL mesh and downloads `{name}_braille.stl`. Ready for 3D printing.
- **Legend TXT**: downloads a text file listing each code, braille unicode, and name.
- **Export Labels JSON**: saves label data for later editing. Re-import with **Import Labels**.

### 4. Resume editing

Import a saved `_labels.json` to restore all labels at their exact coordinates.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Single-page 3D braille editor with built-in STL synthesis |
| `examples/` | Sample labels and blocks JSON files |

## How It Works

Everything runs in the browser:

1. **Region detection**: analyzes STL mesh faces, groups connected surfaces by height continuity
2. **Braille rendering**: converts letter codes to UEB dot patterns, renders as 3D cylinder+sphere geometry
3. **STL synthesis**: merges dot geometry with the original mesh using Three.js STLExporter
4. **No backend**: all file I/O uses browser File API and Blob downloads

## Braille Dot Specifications

| Parameter | Default (100%) | Range with scale slider |
|-----------|---------------|------------------------|
| Dot diameter | 1.6mm | 0.8 to 3.2mm |
| Dot height | 0.8mm | 0.4 to 1.6mm |
| Dot spacing | 2.5mm | 1.25 to 5.0mm |
| Cell gap | 6.0mm | 3.0 to 12.0mm |
| Surface embed | 0.3mm | 0.15 to 0.6mm |

Default values meet UEB (Unified English Braille) tactile standards. Dots are dome-shaped (cylinder stem with sphere cap) for optimal tactile readability.

## Print Settings

- Layer height: 0.3mm
- Infill: 20%
- Supports: not needed (dots are embedded into surface)
- Material: PLA or PETG

## Dependencies

None. Three.js and Leaflet.js are loaded from CDN. No npm, no build step.
