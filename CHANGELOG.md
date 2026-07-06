# Changelog

All notable changes to BrushForge are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/). This project uses [Semantic Versioning](https://semver.org/).

## [0.14.1] — 2026-07-07
### Fixed
- Hardened the Web Worker fallback for sandboxed contexts (Claude's artifact iframe, some file:// setups). Blob-URL workers are often blocked there, and the failure isn't always a synchronous throw — the worker can construct, then fire an async error or silently never reply. Added an 8s watchdog on every pre-proven dispatch (falls back to the synchronous fold if no first reply lands), a `csgWorkerProven` flag so the watchdog only guards the unproven state, and unified all three failure modes through one `csgWorkerDied()` path.
### Added
- HUD line showing the active CSG mode (`async` / `computing…` / `sync`) so the live path is visible without the console.

## [0.14.0] — 2026-07-07
### Changed
- CSG fold now runs in a Web Worker — edits are instant regardless of scene size; the ~3s rebuild no longer freezes the main thread. Extracted the fold into a pure `csgFoldScene()` (single source of truth); the worker script is assembled at runtime from the live functions via `Function.toString()`, so worker and main-thread code cannot drift. Verified bit-identical to the direct fold on the real scene in an isolated vm context. Stale combined mesh stays visible while computing; newer edits terminate and respawn in-flight folds. Automatic synchronous fallback if Worker/Blob is unavailable.

## [0.13.2] — 2026-07-07
### Fixed
- OOM on the real LVHotelLobby scene (177 brushes). Root cause was a single non-convex `raw` import (a 175-poly geometry-mode staircase): building a BSP from a concave poly soup explodes combinatorially and exhausted a 2GB heap on one brush. Non-convex raws are now excluded from the combined solid (via `csgIsConvex`, cached) — they still render individually, stay editable, and export is unaffected (UE re-runs its own BSP on paste). Added a 4M-split budget per rebuild as a safety net: any future runaway aborts cleanly to individual-brush display instead of crashing the tab.

## [0.13.1] — 2026-07-07
### Fixed
- OOM on open levels (one shared floor, everything touching = a single CSG island, so 0.13.0's partitioning couldn't help). BSP planes are infinite, so clipping against each new brush fragmented all geometry crossing its planes however distant, growing poly count superlinearly. Added `csgClipToBounded`: result polys AABB-disjoint from the clipping brush pass through unclipped (exact — same solid, fewer fragments). Poly growth is now roughly linear.

## [0.13.0] — 2026-07-07
### Changed
- Optimized `rebuildCsg` with island partitioning: brushes are grouped into independent connected components by transitive AABB overlap and folded separately, then concatenated with no cross-clipping (exact — disjoint islands can't interact). ~35x faster at 800 spatially-separated brushes, roughly linear rather than quadratic out past 5000+.

## [0.12.4] — 2026-07-07
### Fixed
- GPU memory leak crashing large scenes. THREE.js doesn't free geometry/material buffers on `scene.remove()` — they need explicit `.dispose()`, which none of the 6 mesh-replacement sites called (per-brush rebuild, undo/redo, scene load, New Scene, and the combined CSG mesh rebuilt on every edit). Added a `disposeMesh()` helper wired into all sites. Pre-existing bug, exposed by import making large scenes common.

## [0.12.3] — 2026-07-07
### Fixed
- "Maximum call stack size exceeded" on large scenes. The BSP tree functions (`csgBuild`/`csgInvert`/`csgClipPolys`/`csgClipTo`/`csgAll`) were recursive, and recursion depth grew linearly with brush count. Converted all five to iterative (explicit work-stack) versions. Verified byte-identical output against the recursive originals across 9 union/subtract/mixed scenarios.

## [0.12.2] — 2026-07-07
### Fixed
- Importing any `raw` brush crashed on commit. `facesFor()` computed `hx=size.x/2` unconditionally before the type check, but raw brushes carry no `b.size` (only `b.polys`). Fixed to derive `hx/hy/hz` only when size exists.

## [0.12.1] — 2026-07-07
### Fixed
- Importing a large scene threw a `RangeError` and aborted the whole batch. Every shape-fit min/max used `Math.min(...arr)` / `Math.max(...arr)`, and spreading a large array into function arguments has an engine-enforced ceiling (lower on iPad's JavaScriptCore than V8). Replaced with reduce-based helpers; moved error handling inside the per-actor loop so one bad actor is skipped, not the batch.

## [0.12.0] — 2026-07-07
### Added
- **Import from Unreal**: paste T3D copied from a UE4.27 viewport and reconstruct brushes. Exact fast-path via `CubeBuilder`/`CylinderBuilder`; otherwise the PolyList is geometrically fit against box/cyl/cone/ramp (tolerant of winding order and float noise). Geometry matching no primitive imports as a new `raw` brush type — exact polygon soup, position/rotation editable, no resize or GEO editing. Export is unaffected (UE re-runs its own BSP on paste).

## [0.11.0] — 2026-07-05

### Changed
- **Unified GEO editing interaction.** Face and edge editing now work like vertex editing: tap an element to select it, then an axis-constrained move gizmo appears — one arrow for a face (along its normal), two for an edge (the perpendicular axes), three for a vertex. This replaces the previous direct-drag on face and edge handles. Edge editing is now axis-constrained rather than a free perpendicular-plane drag. Pristine-box face push still stays fully parametric, so the exact CubeBuilder export is preserved. The number of arrows reflects each element's real degrees of freedom (face 1, edge 2, vertex 3); convexity validation and the flat-box gate are unchanged.

## [0.10.0] — 2026-07-05

### Added
- **Export selected only** — when one or more brushes are selected, the export drawer offers an All/Selected toggle (defaults to Selected). Exports just the chosen brushes as valid T3D with correct indexing and CRLF endings. With no selection, the toggle is hidden and the full scene exports as before.

## [0.9.0] — 2026-07-05

### Added
- **Vertex translation in GEO mode** — a new Face/Edge/Vertex sub-toggle. In Vertex mode, tap a corner to select it, then drag a 3-axis move gizmo to translate that vertex one axis at a time (matching Unreal's Geometry Mode workflow). The sub-toggle shows one handle set at a time instead of face and edge handles together. Vertex moves are convexity-validated and produce the same edited (PolyList-only) geometry as edge slide.

## [0.8.1] — 2026-07-05

### Fixed
- Local/World gizmo now works correctly on brushes with pitch or roll. Local-space move handles and drag axes previously tracked yaw only, so they misaligned on tilted brushes. They now follow the brush's true orientation at any angle, using helpers built on the same rotation routine as display and export. Yaw-only brushes are unchanged.

### Known limitations
- GEO mode remains disabled on tilted brushes; its face-push/edge-slide/vertex transforms are still yaw-only (planned follow-up).

## [0.8.0] — 2026-07-05

### Added
- **Full 3-axis rotation** — brushes now support pitch and roll in addition to yaw. A single shared rotation routine applies UE's FRotator order (yaw→pitch→roll) for both the viewport preview and the exported RelativeRotation, so the two can't diverge. Because rotation rides on the brush component rather than being baked into the polygons, pristine boxes keep their exact CubeBuilder export at any orientation. The rotate gizmo has three colour-coded rings; the inspector has yaw/pitch/roll numeric fields.

### Changed
- Save format gains optional `pitch`/`roll` fields. Additive and backward compatible — scenes saved by earlier versions load unchanged (missing angles default to 0).

### Known limitations
- GEO mode and ground-snap are disabled on brushes with nonzero pitch/roll (they no-op with a toast); their local-space transforms are still yaw-only. Local-space move/scale handles are likewise yaw-aligned. Full 3-axis support for these is a planned follow-up.

## [0.7.1] — 2026-07-05

### Fixed
- Undo/redo now rebuilds the combined CSG mesh. Previously the per-brush meshes updated but the blue CSG result stayed on the pre-undo geometry, because applySnapshot() bypassed the pushHistory hook that normally triggers a CSG rebuild.

## [0.7.0] — 2026-07-05

### Added
- **Two-finger tap to undo** — a quick, near-stationary two-finger tap undoes the last action. Disambiguated from pan/pinch by movement (<12px) and duration (<300ms) thresholds; fires exactly once regardless of whether the two fingers lift together or one at a time.

## [0.6.0] — 2026-07-05

### Added
- **GEO mode** — push/pull box faces and slide box edges with on-geometry handles (single box selection). Face push on a pristine box stays parametric, so the exact CubeBuilder export is preserved; edge slide converts the brush to edited 8-corner geometry, exporting as raw polygons (same limitation as ramp — won't survive a UE builder rebuild). Convexity is validated on every drag; non-planar quads auto-triangulate on the convex diagonal. Inspector shows edited state with a reset-to-parametric-box action.

## [0.5.0] — 2026-07-05

### Added
- **Marquee (box) select** — hold ~300ms then drag on empty space to box-select multiple brushes at once. A quick drag still orbits; a long-press without moving still toggles a single brush. Hit-testing projects each brush's world-space vertices through the existing screen-projection path, so it respects yaw and stays correct for rotated brushes. Replace-only for now — no additive/subtractive drag.

## [0.4.0] — 2026-07-02

First versioned release. The full pipeline from browser blockout to UE4.27 paste is working.

### Added
- Real BSP-tree CSG display (polygon splitting, not centroid approximation) so small subtracts carve accurate holes in large faces.
- UE-style **brush order** — CSG is evaluated top-to-bottom in the outliner; a subtract only cuts brushes above it, an add below a subtract fills the space back in.
- Multi-select: shift-click, long-press (touch), Ctrl+A; gizmo, delete, duplicate, ground-snap and CSG toggle all operate on the full selection.
- Outliner panel: reorder (↑/↓), inline rename, visibility toggle, bidirectional selection.
- Save / load named scenes and debounced autosave via localStorage; session restores on reload.
- Undo / redo (60-step snapshot history) with Cmd/Ctrl+Z and redo shortcuts.
- Version stamping in title, UI badge, and script header.

### Fixed
- **T3D paste did nothing on Windows** — output now uses CRLF line endings, which UE4.27's parser requires.
- **Z position mismatch** — `pos.z` now means the brush centre, matching UE's `RelativeLocation` exactly (was base-anchored, showing e.g. -10 in BrushForge vs 90 in UE).
- Export modal could not be closed; replaced with a proper overlay (backdrop tap / ✕ / Escape).
- New Scene / delete confirmations used `confirm()`, which is blocked in iframes; replaced with double-tap inline confirm.
- iOS file download produced empty files (Blob URL); switched to data: URI.

### Known limitations
- CSG display is view-only; export still sends individual brushes (correct — UE recomputes its own BSP).
- localStorage is scoped to the file's path; moving the HTML starts fresh saves.
