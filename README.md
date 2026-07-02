# BrushForge

A single-file, touch-first CSG blockout tool for **Unreal Engine 4.27**. Design BSP brush geometry in the browser (works on iPad via Safari → Add to Home Screen) and paste it straight into the UE4.27 viewport as native brushes.

## What it does

- Place and edit convex primitives: **box, cylinder, cone, ramp**
- **Add / Subtract** CSG with real BSP-tree boolean display
- **Brush order** controls CSG priority, exactly like UE (subtracts only cut brushes above them in the outliner)
- Move / rotate / scale gizmo with world & local space, grid + rotation snapping
- Multi-select (shift-click, long-press, Ctrl+A)
- Outliner with reorder, rename, visibility, per-brush selection
- Undo / redo (60 steps)
- Save / load named scenes + autosave (localStorage)
- **T3D export** → paste into the UE4.27 viewport (verified byte-compatible)

## Usage

Open `brushforge.html` in a browser. No build step, no dependencies beyond Three.js (loaded from CDN).

### Exporting to UE4.27

1. Tap **Export**
2. **Copy** (same machine) or **Share / Save to Cloud** (iPad → Windows)
3. Click the UE4.27 viewport to focus it
4. **Ctrl+V**

BSP brushes can only enter UE via clipboard paste — File → Import Into Level does not accept T3D brush data.

## Coordinate system

Brush data is authored in **Unreal coordinates** (X forward, Y right, Z up; left-handed; units = uu/cm). `pos.z` is the brush **centre**, matching UE's `RelativeLocation`. Display maps to Three.js (Y up) only for rendering.

## Versioning

Current version is stamped in three places, all kept in sync on release:
- `const VERSION` near the top of the `<script>` block
- the `<title>` tag
- the changelog comment block at the top of the script

See [CHANGELOG.md](CHANGELOG.md) for history.

## License

Private project — all rights reserved.
