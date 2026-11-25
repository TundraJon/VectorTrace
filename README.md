# VectorTrace
App to Trace images to Vectors

## Build Queue

**Not yet authorized.** Stage 0 — Visual Design Mockup (standalone, throwaway HTML, not the app shell)

Goal: one self-contained HTML mockup showing two themes side by side so the look can be approved before Stage 1.

Themes (user-supplied spec, schema 2.0.0):
- Blueprint Cyanotype: "Technical Engineering", compact density, 3px radius, IBM Plex Sans, square/miter icons, stroke 1.6.
- Sage Botanical: "Organic Studio", comfortable density, 10px radius, Plus Jakarta Sans, round icons, stroke 1.5.
- Each theme has light and dark modes (4 looks total). Color tokens exactly as supplied: `--vt-bg`, `--vt-surface`, `--vt-surface-alt`, `--vt-ink`, `--vt-ink-muted`, `--vt-border`, `--vt-accent`, `--vt-accent-text`.
- Canvas colors are CSS variables too: `--vt-canvas-path-stroke`, `-node-fill`, `-node-stroke`, `-node-selected-fill`, `-handle-line`, `-selection-fill`, `-grid-color`. Canvas JS reads them with `getComputedStyle`.

Decided with user:
- Fonts: Sans embedded as Latin-subset base64 woff2 (400/500/600). Mono uses the system monospace stack. No CDN.
- Touch targets: both themes `--vt-touch-target: 40px` visual, with a hit area padded to a 44px minimum.
- Theme switched by attributes on `<html>`: `data-theme="blueprint-cyanotype|sage-botanical"` and `data-mode="light|dark"`.

*(Added by Claude. Change before authorizing if wanted.)*
- Hoist fonts, spacing, radius and touch target to theme level so only colors differ per mode (supplied JSON repeats them 4 times).
- Add icon variables: `--vt-icon-stroke-width`, `--vt-icon-linecap`, `--vt-icon-linejoin`.
- Mockup shows: stats strip, four-tab bar, side tool strip, floating context bar, layers-panel row, slider + number pair, swatches, a sample canvas with path, nodes, handles, selection and grid; plus a theme toggle and a light/dark toggle.
- If base64 woff2 can't be fetched from this container, mockup uses the system sans fallback and the font slots stay as marked placeholders.
- Mockup file is not committed into the app; it is a review aid only.

Verification plan: drive the real page, 0 JS errors, screenshot all four looks, confirm every tappable element measures at least 44px hit area.

## Build Planner

Source: `VectorTrace_ProjectDefinition.md` v1.6. Nothing below is authorized. Section numbers refer to that doc.
Stage order is as approved. Stage 0 added at user's request (visual design pass before Stage 1).

### Stage 0 — Visual Design (Sec 1)
- [ ] Two selectable themes: Blueprint Cyanotype and Sage Botanical, each with light and dark mode
- [ ] Theme tokens as CSS variables: colors, fonts, spacing, radius, touch target, canvas colors, icon stroke
- [ ] Fonts: Latin-subset base64 woff2 for UI sans, system monospace stack for mono
- [ ] Touch target 40px visual, 44px minimum padded hit area, both themes
- [ ] Side-by-side mockup approved by user before Stage 1 starts

### Stage 1 — Foundation
- [ ] Single HTML file shell, canvas rendering, mobile-first (Sec 1)
- [ ] Visual design defined: typography, colors, spacing, icons (Sec 1)
- [ ] Persistent viewport; image always behind panels (Sec 2)
- [ ] Four tabs: Load, Auto Trace, Fine Edit, Output; instant switching (Sec 2)
- [ ] Stats strip: `[-/1247 pts] [12 colors] [38 layers] [400×300]` (Sec 2)
- [ ] Settings icon in stats strip opening the settings panel (Sec 2)
- [ ] Settings: tool strip side, default stroke color, stroke width, fill on/off, stroke on/off (Sec 13)
- [ ] Settings: auto-save interval (1-30 min, 5-min snaps) plus on/off (Sec 13)
- [ ] Settings: canvas background Black/White/Checker plus up to 5 custom hex presets, project-persistent, carried to new projects (Sec 13)
- [ ] Quick-cycle background button outside Settings (Sec 13)
- [ ] Settings: symmetry axis mode (Vertical / Horizontal / Horizontal-Vertical, 44° snap) (Sec 13)
- [ ] Settings: dark/light theme, independent of canvas background (Sec 13)
- [ ] Settings: theme choice, Blueprint Cyanotype / Sage Botanical, independent of dark/light and canvas background (new, not in Sec 13)
- [ ] Slider + number input pair component, used on every slider (Sec 4)
- [ ] Tap tolerance 12-15 px on all selectable elements (Sec 4)
- [ ] Animation rules: instant tabs/sub-tabs, smooth slide/fade elsewhere (Sec 3)
- [ ] Reusable port from Ship Builder: file loading, pan/zoom/rotate, transforms, HSL picker, parts system, modal system (Sec 14)

### Stage 2 — Save System and Resume Screen
- [ ] Loading screen on every app open (Sec 12)
- [ ] Empty-state landing page with "Start New Project" (Sec 12)
- [ ] Auto-save, rolling 10 slots (Sec 5)
- [ ] Slot contents: project data, colors with names, timestamp, ~120×80 base64 thumbnail (Sec 5)
- [ ] Manual save into the same 10-slot system (Sec 5)
- [ ] Resume screen: up to 10 cards, editable name, timestamp, delete with confirmation, Start New Project pinned below (Sec 12)
- [ ] Smooth fade into resumed session (Sec 5, 12)
- [ ] Undo stack, 20 levels, covers every action, resets on session load (Sec 8)
- [ ] localStorage/IndexedDB wrapped in try/catch

### Stage 3 — Load Tab
- [ ] Load Image button (file picker), one image at a time (Sec 6)
- [ ] Pinch pan/zoom (Sec 6)
- [ ] 90° rotate button (Sec 6)
- [ ] Fine rotation slider + number input (Sec 6)
- [ ] Canvas width/height inputs and "Match Image" button, still editable after (Sec 6)
- [ ] Viewport acts as crop window; no built-in crop (Sec 6)

### Stage 4 — Auto Trace Tab
**Colors sub-tab (Sec 7A)**
- [ ] Tap-to-flood-fill sampling, additive across taps
- [ ] One combined tolerance slider + number input
- [ ] Add / Subtract mode toggle (green / red)
- [ ] Overlay: bright highlight while selecting, color's own hue once locked, smooth fade
- [ ] Show All checkbox
- [ ] Undo last tap/sample
- [ ] Color entry: base swatch, tolerance gradient bar, tolerance slider + input, delete
- [ ] Max 32 entries; Clear All; Add New Color
- [ ] Focus mode: other entries slide away, controls expand, compact pinned swatch, larger viewport, Done button

**Lines sub-tab (Sec 7B)**
- [ ] Point density slider + input
- [ ] Smoothing slider + input (VTracer penalty-based)
- [ ] Minimum region size slider + input
- [ ] Gap fill toggle
- [ ] Staircase removal (always on) and stacking strategy (always on)
- [ ] Algorithm adapted from VTracer

**Trace sub-tab (Sec 7C)**
- [ ] Run Trace button and progress indicator
- [ ] Result shown as stacked polygon layers over the image
- [ ] Clear Trace button
- [ ] Traced parts available in Fine Edit with no Accept step; Output reachable directly

### Stage 5 — Fine Edit Core
- [ ] Permanent side tool strip, flippable left/right (Sec 8)
- [ ] Floating context bar, slides up on selection (Sec 8)
- [ ] Layers panel: slides from left, tap canvas to dismiss, two-way selection, drag-to-reorder within and across groups (Sec 8)
- [ ] Layer groups named from Tab 2 colors; "Manual" group for drawn shapes; expand/collapse (Sec 8)
- [ ] Part row: drag handle, visibility, lock, fill swatch, stroke swatch, name, type icon (Sec 8)
- [ ] Long-press quick actions: rename, duplicate, delete (Sec 8)
- [ ] Fill and stroke: independent swatches with toggles, match-stroke-to-fill, defaults, Fill OFF + Stroke OFF warning (Sec 8)
- [ ] Color palette: Tab 2 colors on top, HSL picker, eyedropper, recently used, persists in project (Sec 8)
- [ ] Context bar, shape selected: appearance, transform (flip H/V, scale % plus pinch), actions (name, duplicate, simplify, move to group, delete) (Sec 8)
- [ ] Point edit mode: Select / Deselect / Remove, by drag rectangle and tap (Sec 8)
- [ ] Selection actions: Select All, Deselect All, Invert, Delete, Delete Unselected (Sec 8)
- [ ] Simplify Path: 0.1°-45.0° slider, live preview, confirm/cancel (Sec 8)

### Stage 6 — Fine Edit Drawing
- [ ] Polygon tool: tap points, tap near first to close (Sec 8)
- [ ] Rectangle tool: two corners, live preview (Sec 8)
- [ ] Ellipse tool: center, width radius, height radius (Sec 8)
- [ ] Line tool: two points, open path (Sec 8)
- [ ] Auto-commit shapes; each point and each completion is one undo step (Sec 8)
- [ ] Symmetry toggle: visible, draggable centerline, snap indicator, number input for position, live mirrored half (Sec 8)
- [ ] Two named reference points: default Point A / Point B, renamable, visibility toggle, draggable, saved in project and all exports (Sec 8)

### Stage 7 — Output and File Formats
- [ ] Preview: vector only, background toggle black/white/checker (Sec 9)
- [ ] Zone 1: Save Project / Load Project (JONP), visually distinct (Sec 9)
- [ ] Zone 2: SVG export (Sec 9)
- [ ] Zone 2: PNG export with resolution chooser (1x-4x, custom width, live output size, Confirm) (Sec 9)
- [ ] Zone 2: JONV export (Sec 9)
- [ ] JONP encoder/decoder with optional image flag (Sec 10)
- [ ] JONV encoder/decoder (Sec 10)
- [ ] Int16 ×10 coordinate encoding, 3-byte RGB colors (Sec 10)
- [ ] Full opcode table 0x01-0x17 and 0xFF (Sec 10)
- [ ] State-based opcode behavior; mirror stored once with setMirrorAxis, decoder reflects at runtime (Sec 10)

### Needs Decision (logged inconsistencies, not fixed)
- [ ] Doc has no Section 11 (numbering jumps 10 to 12)
- [ ] Header says v1.6; Sec 16 says locked as of v1.5
- [ ] Symmetry: Sec 8 allows a specific non-orthogonal angle; Sec 13 says no free-angle mode and a fixed 44° snap
- [ ] Sec 2 points to Sec 14 for settings; settings are Sec 13
- [ ] Reference-points "included in all export formats" line appears twice in Sec 8
- [ ] Int16 ×10 caps coordinates at ±3276.7; "Match Image" on a large photo (e.g. 4000 px) would overflow
- [ ] Stage 1 item "Visual design defined" now overlaps Stage 0
- [ ] Theme setting is not in the source doc's Sec 13; Sec 13 needs updating if approved
- [ ] Touch target 40/44px vs Sec 4 tap tolerance 12-15 px: confirm Sec 4 applies to canvas elements only

## Build Log

_Empty — nothing here yet._
