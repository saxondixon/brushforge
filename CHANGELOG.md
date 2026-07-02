# Changelog

All notable changes to BrushForge are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/). This project uses [Semantic Versioning](https://semver.org/).

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
