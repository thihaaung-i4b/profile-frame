# Architecture

**Analysis Date:** 2026-04-08

## Pattern Overview

**Overall:** Single-page application (SPA) with monolithic client-side rendering

**Key Characteristics:**
- Entire application contained in a single `index.html` file
- HTML, CSS, and JavaScript embedded inline
- Canvas-based image composition and manipulation
- Immediate feedback with no backend dependencies
- Fully responsive design with mobile-first approach
- Event-driven state management

## Layers

**Presentation Layer:**
- Purpose: Render UI and display application state to user
- Location: `index.html` (lines 1-372)
- Contains: HTML structure, CSS styling, DOM elements
- Depends on: State and business logic
- Used by: Application initialization and event handlers

**Logic Layer:**
- Purpose: Handle business logic for frame selection, image processing, zoom, pan, and download
- Location: `index.html` (lines 374-581)
- Contains: Event handlers, state management, canvas rendering, file I/O
- Depends on: HTML DOM, Canvas API, File Reader API
- Used by: All user interactions

**State Management:**
- Purpose: Track application state across interactions
- Location: `index.html` (lines 388-390)
- Contains: Selected frame index, photo/frame images, zoom percentage, pan offsets, drag state
- Depends on: Nothing (pure state)
- Used by: All logic functions

## Data Flow

**Frame Selection Flow:**

1. User clicks frame card in grid (`.frame-card`)
2. `pick(idx)` handler called with frame index
3. Selected frame class updated, card highlighted with gold border
4. Frame image loaded from `preloaded` array
5. If frame already loaded, `render()` called immediately
6. Toast notification shown
7. State updated: `selIdx` and `frameImg`

**Photo Upload Flow:**

1. User selects image file via file input (`.upload-area input`)
2. `loadPhoto(event)` handler reads file
3. FileReader converts file to data URL
4. Image object created and loaded
5. Upon image load, state updated: `photoImg`, `panX`, `panY` reset
6. Upload UI updated with checkmark and confirmation
7. `render()` called to generate preview
8. Toast notification shown

**Preview Rendering Flow:**

1. Canvas obtained from DOM
2. Canvas dimensions set to frame size (1080×1080)
3. Photo clipped to circle shape (radius 352px at 1080px size)
4. Photo scaled by zoom percentage and positioned by pan offsets
5. Photo drawn in center of canvas
6. Frame overlay drawn on top
7. Placeholder hidden, canvas displayed
8. Download button enabled

**Zoom Adjustment Flow:**

1. User clicks zoom +/- buttons
2. `zoom(delta)` handler updates `zoomPct` (clamped 40-220%)
3. UI label updated
4. `render()` called with new scale

**Pan/Drag Flow:**

1. User touches/clicks canvas (mouse or touch events)
2. `onStart()` records initial position and current pan values
3. On move, `onMove()` calculates delta from drag start
4. Delta scaled by canvas scale ratio (actual pixels vs display pixels)
5. New pan values calculated and `render()` called
6. `onEnd()` disables dragging flag

**Download Flow:**

1. User clicks download button
2. `download()` gets canvas element
3. Canvas converted to blob (PNG format)
4. Object URL created from blob
5. Temporary anchor element created and clicked
6. Object URL revoked, anchor removed
7. Toast notification shown
8. Fallback: Open data URL in new tab if blob creation fails

**Reset Flow:**

1. User clicks reset button (↺)
2. `reset()` clears all state variables
3. All DOM elements reset to initial state
4. Form input cleared
5. UI returned to placeholder state
6. Toast notification shown

**State Management:**

Global state held in JavaScript variables:
- `selIdx`: Currently selected frame index (-1 if none)
- `frameImg`: Preloaded frame Image object
- `photoImg`: User-selected photo Image object
- `zoomPct`: Zoom level percentage (40-220)
- `panX`, `panY`: Photo offset in canvas space pixels
- `dragging`: Boolean flag for active drag state
- `dragStartX/Y`: Initial pointer position during drag
- `panStartX/Y`: Pan values at drag start

## Key Abstractions

**Frame Configuration:**
- Purpose: Centralized definition of available frames with metadata
- Examples: `FRAME_CONFIG` array (lines 381-385)
- Pattern: Array of objects with `src` (data URI), `name` (label) properties
- Frames are pre-rendered SVGs embedded as base64 data URIs for cross-origin safety

**Canvas Composition:**
- Purpose: Combine photo and frame into final image
- Examples: `render()` function (lines 465-499)
- Pattern: 
  - Create circular clip region
  - Draw photo with scale and offset
  - Draw frame overlay on top
  - Display result

**Event Delegation:**
- Purpose: Respond to user interactions
- Examples: Inline onclick handlers, addEventListener calls
- Pattern: Direct event binding to DOM elements with handler functions

## Entry Points

**HTML Page Load:**
- Location: `index.html`
- Triggers: Browser loading the file
- Responsibilities: 
  - Render initial HTML structure
  - Apply CSS styling
  - Preload all frame images
  - Build frame selection grid dynamically
  - Attach event listeners for canvas drag/pan

**Frame Selection:**
- Location: `.frame-card` onclick handlers (line 402)
- Triggers: User clicking a frame thumbnail
- Responsibilities: 
  - Update selection state
  - Update UI highlighting
  - Trigger render if both frame and photo exist

**Photo Upload:**
- Location: `.upload-area input` onchange handler (line 337)
- Triggers: User selecting a file
- Responsibilities: 
  - Read file using FileReader API
  - Decode to image
  - Update upload UI with checkmark
  - Reset pan offset
  - Trigger render

**Zoom Control:**
- Location: `.zbtn` onclick handlers (lines 347-349)
- Triggers: User clicking +/- buttons
- Responsibilities: 
  - Increment/decrement zoom percentage
  - Clamp to valid range
  - Update display
  - Re-render

**Canvas Interaction:**
- Location: Mouse and touch event listeners on canvas (lines 550-558)
- Triggers: User dragging photo on canvas
- Responsibilities: 
  - Track pointer movement
  - Calculate pan offset
  - Re-render on each move

**Download:**
- Location: `.btn-dl` onclick handler (line 371)
- Triggers: User clicking download button
- Responsibilities: 
  - Convert canvas to PNG blob
  - Trigger browser download
  - Handle fallback for cross-origin restrictions

**Reset:**
- Location: `.btn-rst` onclick handler (line 370)
- Triggers: User clicking reset button
- Responsibilities: 
  - Clear all application state
  - Reset all UI elements to initial state
  - Clear file inputs

## Error Handling

**Strategy:** Graceful degradation with fallback mechanisms

**Patterns:**

1. **Image Load Errors:** Not explicitly handled. Images from `FRAME_CONFIG` are embedded as data URIs, so they always load. User photo is trusted input via file picker.

2. **Canvas Rendering Errors:** If canvas is not initialized or dimensions missing, `render()` returns early (line 466).

3. **Download Failures:** Try canvas.toBlob() first. If blob creation fails (cross-origin/security issue), fallback to `canvas.toDataURL()` and open in new tab (lines 506-520).

4. **Missing State:** All functions check for required state before proceeding:
   - `render()` checks `!frameImg || !photoImg` (line 466)
   - `pick()` checks `frameImg.complete` and `frameImg.naturalWidth` (line 426)
   - `onStart()` checks `!photoImg || !frameImg` (line 533)

## Cross-Cutting Concerns

**Logging:** Toast notifications (`.toast` element, lines 413-419)
- Used for: User feedback on actions (frame picked, photo uploaded, zoom changed, download complete, reset)
- Pattern: `toast(msg, ms)` function shows message in top-center for 2.2 seconds by default

**Validation:** 
- Frame selection: None needed (fixed config)
- Photo upload: File type validated in HTML with `accept="image/*"` (line 337)
- Zoom range: Clamped to 40-220% (line 459)
- Canvas dimensions: Defaulted to 1080px if missing (lines 469-470)

**Authentication:** Not applicable (no backend)

**Styling:**
- CSS Variables (`--maroon`, `--gold`, `--bg`, etc.) defined at root for theming (lines 11-24)
- Responsive design with media queries for device sizes (lines 238-313)
- Fixed bottom bar with safe-area inset for notched devices (line 23)
- Dark theme with gold accents throughout

---

*Architecture analysis: 2026-04-08*
