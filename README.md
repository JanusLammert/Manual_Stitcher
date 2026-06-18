# 2D Class Stitcher

A browser-based tool that turns RELION or cryoSPARC 2D class averages of helical assemblies into an interactive sinogram solver, producing an initial 3D helical model directly from 2D classes.

No installation required — download `class_stitcher.html` and open it in any modern browser (Chrome, Firefox, or Edge).

---

## Overview

A 2D class average spanning a full crossover of a helical filament is itself a sinogram: each horizontal position encodes an azimuthal projection of the fibril. Assembling a clean gallery of such classes into a usable cross-section has traditionally been a manual, trial-and-error task. This tool automates that process while still allowing full manual control, implementing the approach described by Scheres (2020) for amyloid structure determination in RELION.

Classes can be assembled either by hand (drag, crop, rotate, mirror) or automatically: the tool solves the cyclic order and mirror orientation of a multi-selected set of classes by maximizing edge correlation around the full periodic boundary, places them with soft-edge blending, and then iteratively refines the placement through a reconstruct → reproject → cross-correlate cycle until convergence. The resulting cross-section can be extruded into an initial 3D helical volume by revolution.

---

## Features

### Data Import
- Load one or more MRCS/MRC stacks from **RELION** or **CryoSPARC** (CryoSPARC classes are automatically rotated 90° on import)
- Supports MRC modes 0 (int8), 1 (int16), 2 (float32), and 6 (uint16)
- Pixel size is read from the MRC header and can be overridden
- Each loaded stack gets a distinct, user-changeable colour bar so classes from different stacks remain visually distinguishable

### Manual Canvas Stitching
- Click thumbnails to add class average instances to the canvas; the same class can be added multiple times, each instance fully independent
- Drag to move, drag a corner handle to crop (including non-square crops), drag the rotation handle to rotate freely
- Per-instance controls: opacity, rotation, mirror X/Y, crop dimensions and offsets, z-order
- Snap-to-grid with configurable grid size, and an Auto Arrange function that lays out all instances in a grid
- Undo with 40-step history

### Automatic Helix Stitch
- Multi-select a set of classes (Shift+click or Shift+drag) and let the tool solve their placement automatically
- **Step 1 — initial placement:** classes are placed at equal intervals based on a specified crossover distance; cyclic order is solved via greedy + 2-opt optimization, with optional mirror-flip testing per class
- **Step 2 — iterative refinement:** repeatedly reconstructs a cross-section from the current arrangement, reprojects it, cross-correlates each class against the reprojection, and updates its X/Y position — repeated for a configurable number of iterations. The reconstruction step here uses a fast SART pass internally for a sharper reprojection reference, though the panel still labels this setting "FBP iterations" (see the Sinogram Cross-Section section below for details)
- Configurable search range per iteration, fade width for soft-edge blending between classes, and Y-alignment mode (centre on axis, align top edges, or keep existing Y positions)

### Sinogram Cross-Section
- Draw a rectangular ROI over the stitched image (assuming the helix runs along X) with corner-handle resizing or exact coordinate entry
- "Snap to Fibril Center" moves the ROI to the centroid of all canvas instances
- Two reconstruction methods, selectable in the modal:
  - **SART** (Simultaneous Algebraic Reconstruction Technique) — the default. An iterative algebraic reconstruction (Andersen & Kak, 1984) that forward-projects the current estimate at each angle, computes a correction from the residual, and back-projects that correction with a configurable relaxation factor (λ). Angle order is randomized each iteration for faster convergence. Both the number of iterations and λ are user-adjustable
  - **FBP** (filtered back-projection) — the classical direct reconstruction, selectable as an alternative via the method toggle
- Displays the sinogram and reconstruction side by side, with a Shannon entropy score of the cross-section (lower entropy indicates a sharper, more structured result)
- X-periodicity scan via autocorrelation to help identify the correct crossover distance

> **Note:** the iterative refinement loop inside the automatic Helix Stitch panel (see above) also reconstructs a cross-section at each refinement step. Internally, this now runs a fast 1–2 iteration SART pass rather than a single-shot FBP, since it gives a sharper reprojection reference at negligible extra cost — even though the panel's UI still labels this setting "FBP iterations" for historical reasons. No action is needed; this is purely an internal implementation detail of how the reprojection reference is computed during automatic refinement.

### 3D Volume by Revolution
- Once a cross-section looks correct, extrude it into a 3D volume by rotating it about the chosen helix axis
- Set helical rise and twist per subunit, handedness, and an optional Cn symmetry order
- Preview equatorial and meridional slices before committing
- Export as a valid MRC float32 file with a correct header, readable by RELION, UCSF ChimeraX, Coot, and EMAN2

### Alignment Optimization
- Mark two instances as REF and MOV, then run a brute-force normalized cross-correlation search over a configurable translation and rotation range to align them precisely

### Save / Load Session
- Save the full canvas state (positions, crops, rotations, opacity, contrast, colormap, stack colours, ROI, marks) as JSON
- Reload a saved session (after re-loading the same MRCS files) to continue exactly where you left off
- Export the current canvas view as a PNG at display resolution

---

## Getting Started

1. Download `class_stitcher.html` and open it in a browser — no server needed
2. Click **Load MRCS** to load one or more `.mrcs` / `.mrc` files (or try the built-in **Test Dataset**)
3. Click thumbnails to add class averages to the canvas, or Shift-select classes and use **Helix Stitch** for automatic placement
4. Use the **ROI** tool to draw a rectangle over the assembled fibril, then compute the sinogram cross-section
5. If the cross-section looks correct, extend it to a 3D initial model and export as MRC

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `V` | Select / Move / Crop / Rotate tool |
| `R` | ROI draw tool |
| `M` | Measure tool |
| `F` | Fit canvas to window |
| `+` / `-` | Zoom in / out |
| `Delete` / `Backspace` | Remove selected instance |
| `Escape` | Deselect / cancel |
| `Ctrl+Z` | Undo |

---

## Technical Notes

- No external dependencies — pure HTML, CSS, and JavaScript in a single file
- Pixel rendering uses a precomputed 256-entry lookup table per colormap (8 colormaps available: Grayscale, Hot, Cool, Viridis, Plasma, Inferno, Cividis, RdBu)
- Contrast normalization uses Welford's online algorithm for a single-pass, memory-efficient statistics computation
- SART reconstruction uses a sequential, randomized-angle-order update scheme following the classic Andersen & Kak (1984) formulation (forward-project at angle a, compute the residual-based correction normalized by ray weight, back-project with relaxation factor λ)
- FBP reconstruction uses a custom Cooley–Tukey FFT implementation for the filtering step
- MRC output follows the MRC2014 standard with a correct `MAP` identifier and machine stamp

---

## Limitations

- Raw pixel data from loaded MRCS files is not stored in the session JSON — reload the same MRCS files before loading a saved state
- The 3D revolution step assumes the structure is rotationally symmetric around the chosen helix axis
- The 2D cross-section reconstruction (whether SART or FBP) is a 2D-only method; no CTF correction or true iterative 3D refinement is performed
- Large stacks (>200 classes, box size >512) may render slowly until the internal cache is populated

---

## License

```
This program allows manual or automatic stitching of 2D classes and generation of an
initial 3D volume based on the stitching.
Copyright (C) 2026 Janus Lammert

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see <https://www.gnu.org/licenses/>.
```

Contact: Janus Lammert (j.lammert@fz-juelich.de)
