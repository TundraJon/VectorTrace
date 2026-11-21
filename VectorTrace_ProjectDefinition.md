# VectorTrace App — Project Definition Document
**Version:** 1.6  
**Date:** June 16, 2026  
**Status:** Planning Phase — Complete (pre-build)

---

## 1. PROJECT OVERVIEW

### What It Is
A standalone HTML-based vector illustration tool that loads a raster image as a tracing reference, auto-traces it into vector shapes using color sampling and adjustable settings, allows full manual fine-tuning of the resulting vector paths and points, and exports the result in multiple formats.

### What Started This
The need to create a vector copy of a complex illustration (hunter running through snow with mountains and trees) for use in a mobile game app. Existing tools didn't provide enough control over the vectorization process. This app is designed to give the user direct, precise control over every aspect of the conversion.

### Relationship to Ship Builder
This is a standalone app, separate from Ship Builder, and a complete rewrite — not a fork or visual continuation. Ship Builder is referenced once here as context: certain proven interaction logic (pan/zoom math, coordinate transforms, touch handling, the general concept of parts/layers) is being adapted because it already works well, not because this app inherits Ship Builder's structure or appearance. Functionally this app is a superset of Ship Builder's capabilities — the only Ship Builder feature NOT included is the thrust system, replaced by the more flexible two named reference points concept (Section 8).

### Visual Style
Clean and modern — explicitly NOT a continuation of Ship Builder's dark terminal/blue-outline aesthetic. Full visual design (typography, color system, spacing, iconography) to be defined separately before build. Dark mode and Light mode are both required (see Settings, Section 13).

### Tech Stack
- Single HTML file (HTML/CSS/JavaScript)
- No server required
- Canvas-based rendering
- Runs on mobile (primary) and desktop

---

## 2. OVERALL STRUCTURE

### Viewport
The loaded image always sits in the background viewport. All four tabs work on top of the same persistent image. The viewport is always visible regardless of which tab is active — panels float on top.

### Main Navigation
Four tabs across the top — instant switching (no animation):

| Tab | Purpose |
|-----|---------|
| 1. Load | Input — load and position the source image |
| 2. Auto Trace | Process — automated color detection and vectorization |
| 3. Fine Edit | Process — manual point editing and shape management |
| 4. Output | Output — export and save options |

### Flexible Workflow — No Tab Is Mandatory
Each tab is a station the user can stop at or skip entirely. All four paths through the app are valid:
- **Full auto:** Load → Auto Trace → Output
- **Auto then refine:** Load → Auto Trace → Fine Edit → Output
- **Full manual:** Load → Fine Edit (draw everything by hand) → Output
- **Hybrid:** Load → Auto Trace → Fine Edit (clean up auto results) → Output

### Always-Visible Stats Strip ✅ LOCKED
A thin strip pinned to the very top of the screen, above the main tab bar. Always visible regardless of active tab. Separate and distinct from the status bar (which reacts to actions/events).

Format: `[-/1247 pts]  [12 colors]  [38 layers]  [400×300]`
- Point count shown as `selected/total`. Shows `-` for selected when nothing is selected (e.g. `[-/1247 pts]`)
- Color count — total color entries in palette
- Layer count — total parts/layers on canvas
- Canvas size — current width × height

### Settings Access
A settings icon lives in the persistent stats strip at the top. Tapping it opens the settings panel (overlay/slide-down). Accessible from any tab at any time. Full settings panel contents detailed in Section 14.

---

## 3. ANIMATION & TRANSITION RULES

| Transition Type | Animation |
|----------------|-----------|
| Tab switches | Instant — no animation |
| Sub-tab switches | Instant — no animation |
| Focus mode enter/exit | Smooth slide |
| Overlay highlights appearing | Smooth fade in |
| Color entries appearing/disappearing | Smooth slide |
| Save/restore session screen | Smooth fade |
| Any panel expanding or collapsing | Smooth slide |
| Layers panel slide in/out | Smooth slide |
| Context bar slide up/down | Smooth slide |

---

## 4. UNIVERSAL UI RULES

### Slider + Number Input (MANDATORY EVERYWHERE)
Every single slider in the entire app must have a paired number input box next to it. No exceptions.
- Slider for quick approximate adjustment
- Number input for precise value entry (critical for use on turbulent airplane)
- They stay in sync — moving slider updates number, typing number updates slider
- Applies to: rotation, tolerance, point density, smoothing, canvas size, opacity, stroke width, simplify path degrees, and everything else

### Tap Tolerance
- Forgiving tap detection on all selectable elements — shapes, lines, points
- Tap NEAR a line, point, or shape edge is sufficient — never requires pixel-perfect accuracy
- Generous hit radius of approximately 12-15 pixels around any selectable element

---

## 5. SAVE SYSTEM

### Auto-Save
- Triggers every 5 minutes automatically (user-adjustable in Settings)
- Silent — no user action required
- Rolling 10 save slots

### Save Slots
Each of the 10 slots stores:
- Full project data (image, all parts, all settings, canvas size)
- All sampled and eyedropped colors with names and organization
- Timestamp
- Thumbnail screenshot of canvas state at time of save (~120×80px, stored as base64)

### Session Resume Screen
- On app open: if saved sessions exist, show resume screen
- Displays up to 10 thumbnail cards with timestamps
- User taps desired slot to restore
- Smooth fade transition into resumed session
- Undo stack resets fresh on every session load

### Manual Save
- Same format as auto-save slot
- User-triggered
- Stored in same rolling 10-slot system

### Project File
- The [JONP] project file format (full spec in Section 10)
- Single downloadable file containing everything needed to resume work
- Colors travel with the file — no loss when moving between devices

### Output Formats (Tab 4)
- SVG — clean vector, scalable
- PNG — flattened raster, resolution chooser at export time
- [JONV] — lean binary vector format for game engine use (full spec in Section 10)

---

## 6. TAB 1 — LOAD ✅ LOCKED

### Purpose
Input tab. Load and position the source image before any tracing begins.

### Features
- **Load Image button** — standard file picker
- **Image in viewport** — sits behind all panels, always visible
- **Pan/Zoom** — pinch gesture
- **Rotation controls:**
  - 90° increment button (used ~80% of the time)
  - Fine-tune slider + number input for precise rotation (e.g., correcting a slightly askew image)
- **Canvas Size:**
  - Manual width/height number inputs
  - "Match Image" button — auto-fills canvas dimensions from loaded image dimensions, remains manually editable after
- One image at a time
- No built-in crop — user handles cropping externally before loading
- The viewport acts as the effective crop window — what is visible in viewport is what gets traced

### Code Source
Pan/zoom/rotate logic — see Section 14 for code reuse details.

---

## 7. TAB 2 — AUTO TRACE ✅ LOCKED

### Purpose
Automated color detection and region tracing with full user control over parameters.

### Sub-Tab Structure (3 sub-tabs — instant switching)

---

### Sub-Tab A: Colors ✅ LOCKED

#### Color Sampling — Flood Fill System
- Tap image pixel → flood fill starts, captures initial connected region of similar color
- Tap additional areas to additively expand that color's captured region
- App uses flood fill outward from tap point, finding all connected pixels within similarity threshold
- One combined tolerance slider + number input controls overall color similarity (not separate H/S/L sliders)
- Flood fill naturally handles all three dimensions of color similarity at once

#### Add / Subtract Modes
- **Add mode** (default) — tap to flood fill and add to current color's region
- **Subtract mode** — tap to flood fill and remove that area from current color's region
- Toggle button between modes — clearly labeled, color coded (green = add, red = subtract)
- Useful for removing accidental captures (e.g., purple edge bleeding into blue sky selection)

#### Visual Overlay Feedback
- While actively selecting: captured region shows as bright contrasting highlight color so selection is obvious against image
- When color entry is locked in: overlay switches to that color's own sampled hue
- **Show All toggle** (checkbox):
  - OFF — shows only active color's overlay while selecting
  - ON — shows all color overlays simultaneously, each using its own sampled color as tint

#### Undo
- Undo button removes the last tap/sample action
- Works in both add and subtract modes

#### Color Entry List
Each color in the list displays:
- **Base swatch** — the exact sampled color
- **Gradient bar** — shows full tolerance range, darkest matching shade on one end, lightest on the other
- **Tolerance slider + number input** — fine-tunes the flood fill range
- **Delete button**

#### List Capacity
- Up to 32 color entries maximum

#### List Controls
- Clear All button
- Add New Color button

#### Focus Mode (per color entry)
- Tap a color entry → triggers focus mode with smooth slide animation
- Other color entries slide away
- Active color's controls expand — larger gradient bar, bigger tolerance slider/input, add/subtract toggle, undo button
- Small compact swatch pinned near top showing which color is active
- Image viewport expands to fill more screen for precise sampling
- **Done button** — exits focus mode, smooth slide back to full list view

---

### Sub-Tab B: Lines ✅ LOCKED

#### Controls
- **Point density** slider + number input
- **Smoothing** slider + number input (VTracer penalty-based algorithm)
- **Minimum region size** slider + number input — ignore regions smaller than X pixels
- **Gap fill** toggle — stacking strategy for borders between adjacent regions

#### Under The Hood
- Staircase removal — always on, automatic
- Stacking strategy — always on, architectural
- Algorithm adapted from VTracer (visioncortex)

---

### Sub-Tab C: Trace ✅ LOCKED

#### Controls & Behavior
- **Run Trace button**
- **Progress indicator** — spinner or progress bar
- **Result appears in viewport** as stacked polygon layers over image
- **Clear Trace button**
- Traced parts automatically available in Fine Edit — no Accept step required
- User can go straight to Output without opening Fine Edit

---

## 8. TAB 3 — FINE EDIT ✅ LOCKED

### Purpose
Manual vector editing. Fully optional — use it to clean up auto-trace results or draw everything from scratch.

---

### Layout

#### Permanent Side Tool Strip
- Right side by default, flippable to left in Settings
- Always visible regardless of selection state
- Contains: Select tool, Point Edit tool, Polygon, Rectangle, Ellipse, Line, Symmetry toggle, Layers panel toggle, Reference Points, Eyedropper, Settings access

#### Floating Context Bar
- Slides up from bottom when a shape or point is selected (smooth slide animation)
- Slides back down when deselected
- Content changes based on what is selected — shape mode vs point edit mode
- Coexists with permanent side strip — they answer different questions

#### Layers Panel
- Slides in from LEFT edge (opposite tool strip — no collision)
- Tap canvas to dismiss, smooth slide animation
- Two-way selection: tap part in list = highlights on canvas; tap shape on canvas = scrolls list to that part
- Full drag-to-reorder within and across groups — stacking order in list = render order on canvas

---

### Layer Groups
- Auto-trace results organized into named groups matching Tab 2 color entry names
- Manually drawn shapes go into "Manual" group automatically
- Each group: expandable/collapsible with triangle toggle
- Each part row shows: drag handle, visibility toggle, lock toggle, fill swatch, stroke swatch, name, type icon
- Long press part row for quick actions: rename, duplicate, delete

---

### Fill and Stroke — Every Shape

#### Two Independent Color Properties
- **Fill color swatch** with on/off toggle — controls inside color
- **Stroke color swatch** with on/off toggle — controls outline color
- **Match stroke to fill checkbox** — when checked, stroke follows fill automatically
- **Stroke defaults to black, ON**
- **Fill defaults to current palette color, ON**

#### Four Valid States
- Fill ON + Stroke ON — filled shape with outline (most common)
- Fill ON + Stroke OFF — solid filled shape, no outline
- Fill OFF + Stroke ON — outline only, transparent inside (ship outline use case)
- Fill OFF + Stroke OFF — not permitted, app warns user

#### Both swatches visible in context bar and layers panel rows

---

### Color Palette in Fine Edit
- Tab 2 sampled colors appear automatically at top, named and organized by color entry names
- HSL color picker for custom colors
- Eyedropper — tap any pixel on background image to sample
- Recently used colors shown
- All colors persist across saves as part of project file
- Colors travel with project file between devices

---

### Context Bar — Shape Selected
**Left — Appearance:**
- Fill color swatch (with on/off toggle)
- Stroke color swatch (with on/off toggle)
- Match stroke to fill checkbox
- Opacity slider + number input
- Stroke width slider + number input

**Middle — Transform:**
- Flip Horizontal
- Flip Vertical
- Scale number input (percentage) — pinch gesture on canvas for quick scaling

**Right — Actions:**
- Name button — tap to rename inline
- Duplicate
- Simplify Path — opens 0.1° to 45.0° slider with live preview
- Move to Group
- Delete

---

### Context Bar — Point Edit Mode

#### Selection Tools (all work via drag rectangle AND individual tap)
- **Select** — adds points to active selection
- **Deselect** — removes points from active selection, still in play for invert
- **Remove** — permanently excludes points from working set, invert ignores them

#### Selection Actions
- Select All
- Deselect All
- Invert Selection — flips selected/deselected, ignores removed points
- Delete — removes currently selected points
- Delete Unselected — removes everything not selected, ignores removed points

#### Path Simplification
- Simplify Path button — opens slider 0.1° to 45.0°
- Degree value = angle difference threshold between consecutive segments
- Live preview as slider moves
- Confirm or cancel

---

### Manual Drawing Tools

#### Available Tools
- **Polygon** — tap to place points, tap near first point to close
- **Rectangle** — tap two corners, auto-closes, previews as you place second point
- **Ellipse** — three points: center, width radius, height radius
- **Line** — two points, open path

#### Drawing Behavior
- All shapes commit automatically — no confirm button needed
- Undo is the only way to walk back any action
- Each point placement is one undo step
- Shape completion is one undo step
- Works back through the undo stack one step at a time

#### Undo Stack
- 20 levels deep
- Covers every action: points placed, shapes completed, shapes deleted, points deleted, points moved, colors changed, opacity changed, layer reordered, groups of points deleted
- Resets fresh on every session load — undo does not persist across saves

#### Symmetry
- Toggle on/off in side strip
- Visible centerline on canvas when active, hidden when off
- Centerline is draggable to any position
- Snaps to perfect vertical or horizontal with visual indicator when dragging near 0° or 90°
- Can also be set to specific angle for non-orthogonal symmetry
- Precise position settable via number input
- Draw one half of a shape — mirrored half appears simultaneously on other side of centerline
- Move centerline to new position for next shape — workflow for drawing multiple symmetric objects

---

### Two Named Reference Points
- Any drawing can have exactly two reference points
- Each point is user-named — defaults to "Point A" and "Point B"
- Examples: Nose/Tail, Horizon Left/Right, Trunk Top/Bottom, Entry/Exit
- Appear as small labeled markers on canvas
- Toggle visibility on/off without deleting
- Repositionable by dragging at any time
- Stored in project file
- Included in all export formats
- Included in all export formats, including the [JONV] binary format (Section 10)

---

## 9. TAB 4 — OUTPUT ✅ LOCKED

### Purpose
Preview the finished vector and either save the working project or export a finished deliverable.

### Preview Section
- Renders the finished vector only — source/background image is hidden
- Background toggle: black / white / checker (same pattern as viewport background cycling elsewhere in the app)

### Zone 1 — Save Project
Visually distinct from the export zone (separate color treatment / labeled divider) so there's no confusion between "continue working later" and "finished deliverable."
- **Save Project** button — writes full [JONP] format (everything: image flag, palette, parts, groups, ref points, app state, draw codes)
- **Load Project** button — opens a saved [JONP] file and resumes exactly where it left off

### Zone 2 — Export Finished Vector
- **SVG export** — standard scalable vector file
- **PNG export** — opens a resolution chooser before exporting:
  - Current canvas size displayed (e.g. "Canvas: 400 × 300")
  - Quick-scale buttons: 1x / 2x / 3x / 4x — tapping shows calculated output resolution live (e.g. 3x → "Output: 1200 × 900")
  - Custom width number input — height auto-calculates to preserve canvas aspect ratio, output resolution updates live as user types
  - Confirm button triggers the actual export at the chosen resolution
- **[JONV] export** — lean binary vector-only file (see format spec below), for feeding into the game engine

---

## 10. THE .JON BINARY FILE FORMAT ✅ LOCKED

### Concept
One file format, two modes, distinguished by a 4-byte magic header. Designed for maximum compactness — floats are converted to Int16 (×10 scale, 1 decimal precision preserved), colors are packed as 3-byte RGB instead of 7-character hex strings, and mirrored/symmetric shapes are stored once with a runtime mirror flag instead of being duplicated in the file.

### File Type Headers
- **[JONP]** — Project file. Fully round-trip editable. Contains everything needed to resume work.
- **[JONV]** — Vector file. Lean export. Contains only what the game engine / renderer needs to draw the object.

### [JONP] Project File Structure
```
[JONP]          — 4 bytes, magic identifier
[version]       — 1 byte, format version number
[canvasW]       — Int16, canvas width
[canvasH]       — Int16, canvas height
[colorCount]    — 1 byte, number of palette colors
[colors...]     — 3 bytes each (RGB)
[colorNames...] — variable length strings, null terminated
[groupCount]    — 1 byte, number of layer groups
[groupNames...] — variable length strings, null terminated
[refPointA]     — 2x Int16 coordinates + null terminated name string
[refPointB]     — 2x Int16 coordinates + null terminated name string
[imageFlag]     — 1 byte, 0 = no image stored, 1 = image included
[imageData...]  — present only if imageFlag = 1; base64 encoded source image
[appState...]   — zoom, pan, rotation, active tab, current tool
[partCount]     — Int16, number of parts
[parts...]      — full editable part structure with all properties
[drawCodes...]  — binary opcodes and coordinates (see opcode table)
```
Source image storage is optional and user-controlled at save time via the imageFlag — keeps the file lean when the image isn't needed, but available when exact orientation/size verification matters.

### [JONV] Vector Export File Structure
```
[JONV]          — 4 bytes, magic identifier
[version]       — 1 byte, format version number
[canvasW]       — Int16, canvas width
[canvasH]       — Int16, canvas height
[refPointA]     — 2x Int16 + null terminated name string
[refPointB]     — 2x Int16 + null terminated name string
[drawCodes...]  — binary opcodes and coordinates, nothing else
```

### Coordinate Encoding
All coordinates are multiplied by 10 and rounded, then stored as a signed 16-bit integer (Int16). Example: -68.3 → -683 → stored in 2 bytes. A full x,y coordinate pair = 4 bytes. Range covers -3276.8 to +3276.7 in original units, more than sufficient for any canvas size used in this app.

### Opcode Table (Complete)

| Opcode | Command | Payload |
|--------|---------|---------|
| 0x01 | beginPath() | none |
| 0x02 | moveTo(x,y) | 4 bytes (2x Int16) |
| 0x03 | lineTo(x,y) | 4 bytes (2x Int16) |
| 0x04 | closePath() | none |
| 0x05 | stroke() | none |
| 0x06 | fill() | none |
| 0x07 | setLineWidth(w) | 1 byte |
| 0x08 | setStrokeColor(r,g,b) | 3 bytes (RGB) |
| 0x09 | setFillColor(r,g,b) | 3 bytes (RGB) |
| 0x0A | fillRect(x,y,w,h) | 8 bytes (4x Int16) |
| 0x0B | drawText(str,x,y) | 1 byte length + string bytes + 4 bytes (2x Int16) |
| 0x0C | strokeRect(x,y,w,h) | 8 bytes (4x Int16) |
| 0x0D | ellipse(cx,cy,radiusX,radiusY) | 8 bytes (4x Int16) |
| 0x0E | setOpacity(alpha) | 1 byte (0-255) |
| 0x0F | beginGroup(name) | 1 byte length + string bytes |
| 0x10 | endGroup() | none |
| 0x11 | refPointA(x,y,name) | 4 bytes + 1 byte length + string bytes |
| 0x12 | refPointB(x,y,name) | 4 bytes + 1 byte length + string bytes |
| 0x13 | mirrorModeOn() | none |
| 0x14 | mirrorModeOff() | none |
| 0x15 | setFillOff() | none |
| 0x16 | setStrokeOff() | none |
| 0x17 | setMirrorAxis(x1,y1,x2,y2) | 8 bytes (4x Int16) — defines centerline as two points |
| 0xFF | endOfFile | none |

### State-Based Behavior Rules
- **setLineWidth, setStrokeColor, setFillColor, setOpacity** all behave as persistent state — once set, the value applies to all subsequently drawn shapes until changed again by another call to that same opcode.
- **Rotation** is baked directly into point coordinates — no separate rotation opcode exists for individual shapes.
- **Mirror/Symmetry** is the one exception to "rotation baked into coordinates" — because the centerline is user-repositionable during a drawing session, it must be stored explicitly via setMirrorAxis so the decoder knows exactly where to reflect across. Shapes drawn while mirrorModeOn() is active are stored once; the mirrored copy is generated at runtime by the decoder, cutting file size roughly in half for symmetric objects.

### Design Rationale
Replaces verbose canvas.js text calls (e.g. `ctx.beginPath(); ctx.moveTo(x,y);`) with single-byte opcodes, replaces 7-character hex color strings with 3-byte RGB values, and eliminates duplicated mirror-shape data via a runtime mirror flag. Result is a highly compact binary file relative to an equivalent text-based format.

---

## 12. SESSION RESUME SCREEN ✅ LOCKED

### App Open Flow
- Brief loading screen on every app open (app name/logo)
- Then lands on a landing page — content depends on whether saved sessions exist

### No Saved Sessions (first launch or all sessions cleared)
- Empty state landing page
- "Start New Project" button only
- Tapping it → smooth fade transition into Tab 1 Load with an empty canvas
- Only Black/White/Checker background options available — no custom presets exist yet

### Saved Sessions Exist
- Vertical list of up to 10 session cards
- "Start New Project" button pinned at the bottom, below all cards
- Tapping a card → smooth fade transition into that resumed session, exactly as left

### Session Card Contents
- Thumbnail screenshot (~120×80px)
- Editable name — defaults to timestamp, can be renamed (e.g. "Hunter Image")
- Timestamp
- Delete button — always requires confirmation before removing (destructive action)
- Tapping anywhere else on the card loads/resumes that session

---

## 13. SETTINGS PANEL ✅ LOCKED

### Access
Settings icon lives in the persistent stats strip at the top, accessible from any tab at any time.

### Full Settings List

**Tool strip side**
- Left or Right toggle
- Default: Right

**Default stroke color**
- Customizable, applies to all new shapes going forward
- Default: Black

**Default stroke width**
- Slider + number input
- Default: 2

**Default fill on/off**
- Toggle, applies to all new shapes going forward
- Default: ON

**Default stroke on/off**
- Toggle, applies to all new shapes going forward
- Default: ON

**Auto-save interval**
- Slider + number input, range 1-30 minutes
- Snaps to 5-minute intervals from 5 to 30 (5,10,15,20,25,30); 1 minute available as a low-end option
- Default: 5 minutes
- On/Off toggle — auto-save can be fully disabled

**Canvas background**
- Built-in options: Black, White, Checker
- Plus up to 5 custom user-defined hex color presets
- Each custom preset displays its hex code underneath the swatch for reference
- Custom presets are project-persistent — saved inside the [JONP] project file, not stored app-wide
- Starting a NEW project automatically carries forward the custom presets from the most recently active project (editable independently from that point forward)
- Loading an existing saved project restores that project's exact custom presets
- A truly fresh app start with no project history shows only Black/White/Checker
- Removable via delete button in settings
- Quick-cycle button available elsewhere in the app (outside Settings) for fast switching during work — cycles through Black → White → Checker → Custom 1 → Custom 2 → etc.

**Symmetry axis mode**
- Three radio button options:
  - **Vertical** — axis always perfectly vertical; tap sets horizontal position of the line
  - **Horizontal** — axis always perfectly horizontal; tap sets vertical position of the line
  - **Horizontal/Vertical** — drag freely; snaps to whichever of the two (0° or 90°) falls within a fixed 44° tolerance; no adjustable slider
- No free-angle/custom-angle mode — no identified use case for it

**App theme**
- Dark mode / Light mode toggle
- Applies to the app's own UI chrome (panels, tool strip, context bar, settings, etc.) — independent from the canvas background color setting, which is separate and project-specific
- Default: TBD when visual design is defined

### Explicitly NOT Settings (fixed, not user-adjustable)
- Tap tolerance radius — fixed at approximately 12-15 pixels, not exposed to the user

---

## 14. PROVEN LOGIC BEING ADAPTED (reference: Ship Builder)

Certain interaction logic is being carried over because it already works well on touch devices — not the visual style, just the underlying mechanics.

| System | Reuse Plan |
|--------|-----------|
| File loading (fileInput, shipImg) | Direct adapt |
| Pan/zoom/rotate touch handlers | Direct adapt |
| Canvas coordinate transforms | Direct adapt |
| .jon file output | Direct adapt |
| HSL color picker | Direct adapt |
| Parts management system | Direct adapt |
| Manual drawing tools | Direct adapt |
| Modal system | Direct adapt |
| Eyedropper/color sampler | Extend significantly |

---

## 15. TEST CASE IMAGE

Hunter in fur clothing running through snow with mountains and pine trees in background. Complex illustration with distinct color regions, clear outlines, multiple layered elements (sky, mountains, trees, figure, snow). Benchmark for auto-trace quality.

---

## 16. PLANNING STATUS

All sections of this document are locked as of v1.5. Planning phase is complete — ready to proceed to build phase upon explicit instruction.

**This document is a living reference, not a fixed contract.** Build-phase discoveries, technical constraints, or simply seeing something rendered and wanting it different are all expected and normal. When changes are needed mid-build, this document gets updated and both parties confirm agreement before proceeding — no unilateral changes by either side.

### Reminder of Working Agreement
- Nothing gets built without explicit instruction to build
- Every decision is discussed and agreed upon before implementation
- No assumptions get made and acted upon without confirmation
- This is a collaborative partnership, not independent execution by either party

---

*End of planning document.*
