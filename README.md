# VectorTrace
App to Trace images to Vectors

## Build Queue

**Not yet authorized.** Stage 1A — App Shell, Themes, Tabs, Stats Strip, Viewport

Goal: the first real app file, `index.html` at repo root. Single file, offline, no CDN, canvas-based, mobile-first, runs on desktop too (Sec 1). Stage 1B (Settings panel contents and persistence) follows as its own build.

From the spec:
- Persistent viewport: one `<canvas>` fills the screen behind all panels; the image is visible on every tab (Sec 2).
- Four tabs, instant switching, no animation: Load, Auto Trace, Fine Edit, Output (Sec 2, 3). Tab bodies are empty placeholders labeled with the tab name; real content arrives in later stages.
- Always-visible stats strip pinned to the very top, above the tab bar: `[-/0 pts] [0 colors] [0 layers] [400×300]`. Point count is `selected/total`, with `-` when nothing is selected. Values come from app state (Sec 2).
- Settings icon in the stats strip opens a slide-down panel shell, reachable from any tab (Sec 2). Panel contents are 1B.
- Slider + number input pair as one reusable component: slider and number stay in sync, number clamps on commit, spinners hidden. Used on every slider in the app from here on (Sec 4).
- Tap tolerance constant `TAP_TOLERANCE_PX = 14` (Sec 4 range 12-15). Real hit-testing arrives with the stages that have selectable shapes.
- Animation rules as CSS classes: tabs instant; panels and overlays use smooth slide or fade (Sec 3).

From the approved Stage 0 mockup:
- Both themes, light and dark: Blueprint Cyanotype and Sage Botanical. Token blocks copied as-is from `mockups/stage0_theme_mockup.html`: theme-level tokens (fonts, radius, spacing, touch target, icon stroke) and mode-level tokens (colors, canvas).
- `data-theme` and `data-mode` go on `<html>`.
- IBM Plex Sans 400/500/600 embedded as base64 woff2 (Latin subset); system monospace stack for mono.
- Controls 40px visual with a 44px hit zone (`.hit` and `.hit-pad` patterns). Canvas colors read from the CSS variables with `getComputedStyle`.

Ported from ShipBuilder (working logic only, no styling):
- Image loading via FileReader.
- `view {tx, ty, scale, cos, sin}` with `screenToImage` and `imageToScreen`; `rotateClockwise`.
- Touch handlers for pan, pinch-zoom and twist-rotate, normal view only.
- Modal system: `showNameModal` generalized (title, default value, callback, multiline).
- Left out: nose/tail calibration (`imageToCanvas`, `canvasToScreen`, `activeAxis`), thrust, `.jon` format, parts-move mode.
- Not in 1A: HSL picker, eyedropper, parts system. They stay in the Planner and get ported when first used (Stage 5).

*(Added by Claude. Change before authorizing if wanted.)*
- Tab bar position: top, as spec Sec 2 says ("across the top"). The Stage 0 mockup showed it at the bottom; see Needs Decision.
- Theme switching until 1B adds Settings controls: `?theme=sage-botanical&mode=dark` URL parameters. Defaults: Blueprint, light.
- A plain "Load Image" button on the Load tab so the viewport can be exercised. The Load tab's full UI (rotate button, fine-rotation slider, canvas size inputs, Match Image) stays Stage 3. Alternative: skip image loading in 1A and test with a sample image.
- Default canvas 400×300; viewport background white (light mode) or near-black (dark mode). The Black/White/Checker choice is 1B.
- Layout uses `dvh`, not `100vh`; bottom-pinned UI uses `position: fixed` (CLAUDE.md App facts).
- Slider pair is verified by a scratch test only, since nothing in 1A uses a slider yet.

Verification plan: drive the real page. 0 JS errors; tabs switch with no transition; stats strip text correct; settings panel slides and closes from every tab; screenshot all four theme looks plus 360px; load an image and check pan, pinch and twist-rotate using synthetic touch events; slider pair sync and clamp.

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
- [ ] Sage layer names can truncate at 360px wide ("Horizo…" for "Horizon line"); row has little room at that width (seen in Stage 0 mockup)
- [ ] Tab bar position: spec Sec 2 says tabs "across the top"; the approved Stage 0 mockup showed them at the bottom. Queue 1A defaults to top per spec

## Build Log

### 1. Stage 0 — Visual design mockup (Blueprint Cyanotype + Sage Botanical)
- [x] `mockups/stage0_theme_mockup.html`: new single-file, offline review mockup (177 KB). Not the app shell.
- [x] Theme tokens per `[data-theme]` (fonts, radius, spacing, touch target 40px, icon stroke/cap/join); color + canvas tokens per `[data-theme][data-mode]`
- [x] Fonts embedded as Latin-subset base64 woff2: IBM Plex Sans 400/500/600, Plus Jakarta Sans 400/500/600 (from npm `@fontsource` 5.3.0); mono is the system stack
- [x] Hit areas: `.hit::after` pads each 40px control to a 44px zone; `.hit-pad` does the same for the slider and number input
- [x] `draw()` paints the canvas from the CSS variables via `getComputedStyle`; helpers `rr()`, `smooth()`
- [x] `bindPair()` slider + number sync; `wireApp()` clones one template into both panels; `apply()` / `loadState()` / `saveState()` (try/catch) for the View, Theme, Mode toggles
- [x] Shows: stats strip, tool strip, floating context bar, layers rows, slider+number pair, palette, buttons, four-tab bar, sample canvas (path, nodes, handles, marquee, grid)
- [x] *(Added by Claude)* Fonts/spacing/radius/touch target hoisted to theme level; icon stroke/cap/join as CSS variables; number-input spinners hidden (they clipped "12.5")
- Verification: Playwright against the real file, 60/60 checks. Token values match the spec in both themes and modes; all 6 font faces loaded; 38 `.hit` controls per theme have a 44px zone with a real hit-test and no overlapping zones; canvas pixel matches `--vt-canvas-node-selected-fill` in all four looks; slider/number sync and clamp; Single view and theme toggle; 0 JS errors; 0 failed requests; no horizontal scroll at 360px. Screenshots checked for all four looks plus 360px. First run showed 2 hit-test failures; both were test errors (points outside the viewport, and pixel snapping at 4px gaps), not app bugs.
- Needs a real-device check: feel of 40px buttons with 44px zones on a phone thumb; font rendering and sizes in Android WebView; canvas sharpness on a high-density screen; custom slider thumb in WebView; Sage layer-name truncation at narrow widths; dark-mode contrast outdoors.

### 2. Stage 0 — Sage Botanical font changed to IBM Plex Sans
- [x] Replaced Plus Jakarta Sans with IBM Plex Sans for Sage Botanical, at the user's request; both themes now share one UI font family (Entry 1 still names Plus Jakarta Sans; this entry supersedes it)
- [x] `mockups/stage0_theme_mockup.html`: removed the three Jakarta `@font-face` blocks; Sage `--vt-font-ui` now the Plex stack; Sage caption updated
- [x] Embedded font data now 3 files (Plex 400/500/600); mockup file 177 KB to 128 KB
- [x] Themes still differ in colors, radius (3px / 10px), spacing density, icon cap/join and stroke width
- Verification: Playwright against the real file, 59/59 checks (was 60; the two per-family font checks collapsed into one). All 3 font faces loaded, both panels render Plex, tokens unchanged, hit areas and canvas checks still pass, 0 JS errors, 0 failed requests. Screenshots checked: side by side light/dark and 360px. At 360px "Sun disc" now fits in Sage; "Horizon line" still truncates (logged in Needs Decision).
- Needs a real-device check: Plex rendering and weights in Android WebView; whether Sage still feels distinct enough from Blueprint with the same font.
