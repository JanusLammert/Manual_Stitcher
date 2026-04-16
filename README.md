# 2D Class Stitcher

A browser-based tool for manually assembling 2D class averages from cryo-EM experiments into composite images and computing cross-sectional reconstructions of helical assemblies.

No installation required — open `class_stitcher.html` in any modern browser.

---

## Features

### Data Import
- Load one or more **MRCS stacks** (RELION or CryoSPARC format)
- Supports MRC modes 0 (int8), 1 (int16), 2 (float32), and 6 (uint16)
- **CryoSPARC** classes are automatically rotated 90° on import
- Pixel size is read from the MRC header and can be overridden
- Each stack is assigned a distinct **colour bar** (top and bottom of each class) so you can tell stacks apart at a glance — colour is user-changeable via a colour picker

### Canvas Stitching
- Click thumbnails to add class average **instances** to the canvas
- Add the **same class multiple times** — each instance is fully independent
- **Drag** to move, **drag a corner handle** to crop symmetrically, **drag the rotation handle** (↻) to rotate freely
- Non-square crops supported: set width and height independently
- Per-instance controls: opacity, rotation, mirror X/Y, crop W×H, crop offsets
- Global crop: apply one crop size to all instances simultaneously
- **Snap-to-grid** with configurable grid size
- **Auto Arrange** lays out all instances in a grid and fits the view
- **Undo** (Ctrl+Z / button) with 40-step history

### Display
- Uniform pixel scale across all classes — display scale slider affects all equally
- **8 colormaps**: Grayscale (default), Hot, Cool, Viridis, Plasma, Inferno, Cividis, RdBu
- Contrast controls: min/max in σ units, invert
- Render cache — pixel data is only recomputed when appearance parameters actually change, keeping the canvas responsive even with many instances

### Measurement
- Ruler tool: click-drag to measure distances in pixels, Ångström, and nanometres

### Sinogram Cross-Section
- Draw an **ROI rectangle** over the stitched image (assuming the helix runs along X)
- Drag and resize the ROI with corner handles; enter exact coordinates via the ROI control panel
- **Snap to Fibril Center** moves the ROI to the centroid of all canvas instances
- Two projection modes:
  - **Sum along X (helix axis)** — rotates the patch and sums columns, the physically correct approach for helical assemblies
  - **Full 2D Radon** — standard line-integral transform
- Filter options: Ram-Lak, Hamming, Hann, or unfiltered
- Displays sinogram and **Filtered Back Projection (FBP)** reconstruction side by side
- **Shannon entropy score** of the cross-section — lower entropy indicates a sharper, more structured result

### 3D Volume by Revolution
- Approve the cross-section and extrude it into a **3D volume** by rotating it about the Y or X axis
- Set maximum radius and an optional crossover distance cutoff
- Preview equatorial (XZ) and meridional (XY) slices before computing
- Export as a valid **MRC float32 file** with correct header (readable by UCSF ChimeraX, RELION, Coot, EMAN2)

### Alignment Optimization
- Mark two instances as **REF (A)** and **MOV (B)**
- Brute-force NCC (normalised cross-correlation) search over a configurable translation and rotation range
- Results show best NCC score, ΔX, ΔY, ΔRotation — apply with one click

### Save / Load Session
- **Save State** exports a JSON file capturing all instance positions, crops, rotations, opacity, contrast, colormap, stack colours, ROI, and marks
- **Load State** restores a saved session (reload the same MRCS files first)
- **Export PNG** renders the current canvas at display resolution, including stack colour bars and the selected colormap

---

## Getting Started

1. Download `class_stitcher.html`
2. Open it in Chrome, Firefox, or Edge (no server needed)
3. Click **Load MRCS** to load one or more `.mrcs` / `.mrc` files
4. Click thumbnails in the left panel to add class averages to the canvas
5. Arrange, crop, and rotate as needed
6. For cross-sections: switch to the **ROI tool** (R), draw a rectangle over the fibril, then click **Sinogram X-Sec**

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `V` | Select / Move / Crop / Rotate tool |
| `R` | ROI draw tool |
| `M` | Measure tool |
| `F` | Fit canvas to window |
| `+` / `-` | Zoom in / out |
| `Delete` / `Backspace` | Remove selected instance |
| `Escape` | Deselect / cancel |
| `Ctrl+Z` | Undo |

---

## Workflow Example: Helical Assembly Cross-Section

```
1. Load MRCS files from RELION or CryoSPARC
2. Add class averages of the helical filament to the canvas
3. Arrange them so the filament runs left → right (X direction)
4. Select the ROI tool, draw a rectangle that captures the full filament width
5. Use "Snap to Fibril Center" to centre the ROI precisely
6. Click "Sinogram X-Sec" → adjust filter → inspect sinogram and FBP result
7. Check the entropy score — lower = sharper cross-section
8. Click "Looks Good → Extend to 3D Map"
9. Set the revolution axis and max radius → Export MRC
```

---

## Technical Notes

- **No dependencies** — pure HTML + CSS + JavaScript, single file
- **Pixel rendering** uses a 256-entry LUT per colormap computed at startup; colormaps match matplotlib/scientific conventions
- **Statistics** (contrast normalisation) use Welford's online algorithm — O(N) single pass, no intermediate arrays
- **FBP** uses a custom Cooley–Tukey FFT implementation with ramp, Hamming, or Hann frequency-domain filter
- MRC output follows the MRC2014 standard with correct `MAP ` identifier and machine stamp
- Tested with MRC modes 0, 1, 2, 6 as produced by RELION 3/4 and CryoSPARC v3/v4

---

## Limitations

- Raw pixel data from loaded MRCS files is **not stored in session JSON** — you must reload the same MRCS files before loading a saved state
- The 3D revolution assumes the structure is rotationally symmetric around the chosen axis
- FBP is a simple back-projection; no CTF correction or iterative refinement is performed
- Large stacks (>200 classes, box >512) may be slow to render initially until the cache is populated

---

## License

MIT — free to use, modify, and distribute.
