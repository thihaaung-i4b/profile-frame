# Codebase Concerns

**Analysis Date:** 2026-04-08

## Tech Debt

**Monolithic Single-File Architecture:**
- Issue: All HTML, CSS, and JavaScript are contained in a single `index.html` file (585 lines), making maintenance and testing difficult
- Files: `/Users/thihaaung/profile-frame/index.html`
- Impact: Hard to isolate logic, no code reuse across projects, difficult to version control and review specific features
- Fix approach: Separate into modular structure with separate HTML, CSS (or CSS-in-JS), and JS files. Consider a build tool (Vite, Webpack) to bundle assets

**Inline Frame Configurations:**
- Issue: Frame SVG overlays are embedded as base64-encoded data URIs directly in the JavaScript configuration, making them unreadable and hard to update
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 381-385: `FRAME_CONFIG` array)
- Impact: Adding or modifying frames requires base64 encoding/decoding; version control diffs are unreadable; team collaboration is hindered
- Fix approach: Store frames as separate SVG files and load them via fetch/import at runtime, or maintain a decode script to view/edit them

**Global State Variables:**
- Issue: Application state stored in global variables (`selIdx`, `frameImg`, `photoImg`, `zoomPct`, `panX`, `panY`, `dragging`, etc.)
- Files: `/Users/thihaaung/profile-frame/index.html` (line 388-390)
- Impact: No state isolation; potential race conditions; difficult to add features like undo/redo; harder to debug state mutations
- Fix approach: Consolidate into single state object or use state management library (minimal option for single-file app: use a `state = {}` object)

**Mixed Concerns in Functions:**
- Issue: Functions like `render()` and `download()` handle rendering, DOM updates, and UI state simultaneously
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 465-499 for `render()`, 502-521 for `download()`)
- Impact: Difficult to test independently; hard to reuse rendering logic; changes to UI may affect canvas logic
- Fix approach: Separate canvas rendering logic from DOM manipulation; consider canvas rendering as pure function accepting state

## Known Bugs

**Missing Image Load Error Handling:**
- Symptoms: If a user selects an invalid image or the file read fails, no error message is shown; UI states become inconsistent
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 435-455: `loadPhoto()` function)
- Trigger: Select corrupted image file, or file read operation fails (e.g., browser permission denied)
- Workaround: Currently none; user experience silently fails
- Fix: Add `img.onerror` handler and `r.onerror` handler for FileReader to display user-friendly error messages

**Frame Loading Race Condition:**
- Symptoms: If user rapidly clicks multiple frames before previous frame loads, state may become inconsistent
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 422-432: `pick()` function)
- Trigger: Click frame 1, quickly click frame 2 before frame 1 finishes loading and fires onload callback
- Workaround: None; user must wait for toast notification
- Fix: Add loading state; prevent frame clicks while loading; cancel previous load operations

**Canvas Taint Not Mitigated for Cross-Origin Frames:**
- Symptoms: If frame images are loaded from external URLs (cross-origin), canvas becomes tainted and `toDataURL()` in download will fail
- Files: `/Users/thihaaung/profile-frame/index.html` (line 379-380: comment acknowledges this with inline data URIs, and line 518: fallback to `window.open()`)
- Trigger: User serves app with cross-origin frame images instead of data URIs
- Workaround: Fallback uses `window.open(cv.toDataURL())` which requires user to manually save, but fails silently in some browsers
- Fix: Implement CORS headers on frame server; document requirement for data URIs or CORS-enabled endpoints

**Zoom/Pan Limits Not Enforced:**
- Symptoms: Users can pan photo far outside the visible area (no bounds checking on `panX` and `panY`)
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 540-546: `onMove()` in drag handler)
- Trigger: Drag photo far to the edge multiple times
- Workaround: User can reset with Reset button
- Fix: Clamp `panX` and `panY` to reasonable bounds based on canvas size and image dimensions

## Security Considerations

**No Input Validation on File Upload:**
- Risk: No file size limit, MIME type validation, or content inspection; users could attempt to upload very large files causing memory exhaustion
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 435-455: `loadPhoto()` function)
- Current mitigation: Browser's `accept="image/*"` attribute provides basic filtering, but can be bypassed
- Recommendations:
  1. Add explicit file size check before reading (e.g., reject files > 10MB)
  2. Validate MIME type via magic bytes or Content-Type
  3. Add timeout to FileReader to prevent hanging
  4. Display user-friendly error for invalid files

**No CORS or CSP Headers:**
- Risk: If served over HTTP, vulnerable to MITM attacks that modify frame images or inject scripts
- Files: Server configuration (not in codebase)
- Current mitigation: Frames embedded as data URIs (good), but no CSP enforced
- Recommendations:
  1. Serve over HTTPS
  2. Add `Content-Security-Policy` headers to prevent inline script injection
  3. Set frame options and origin restrictions if deployed

**Base64-Encoded Images Vulnerable to Size-Based Attacks:**
- Risk: Large base64 strings in JavaScript may slow down parsing; no validation of embedded content
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 381-385: `FRAME_CONFIG` base64 values)
- Current mitigation: None
- Recommendations: Monitor payload size; consider lazy-loading frames instead of preloading all base64 data

## Performance Bottlenecks

**Full Canvas Redraw on Every Interaction:**
- Problem: `render()` redraws entire canvas (photo + frame) on every zoom increment (+10%), pan movement, or interaction; no optimization for unchanged regions
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 465-499: `render()` called from `zoom()`, `onMove()`, and after photo load)
- Cause: `ctx.clearRect()` and full redraw are unoptimized; no dirty rect tracking
- Improvement path:
  1. Implement requestAnimationFrame batching to coalesce multiple render calls
  2. Consider separating static frame rendering from dynamic photo rendering
  3. Use canvas transform/save-restore to minimize redraw scope
  4. On desktop, redraws happen frequently during drag; throttle or debounce

**Base64 Data URIs Loaded on Every Page Load:**
- Problem: Large base64-encoded SVGs preloaded in memory on page load; no lazy loading or compression
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 381-385, 393: `FRAME_CONFIG` and `preloaded` array)
- Cause: All 3 frames preloaded upfront; no deferred loading
- Improvement path:
  1. Use fetch to load only selected frame on demand
  2. Compress SVG source before base64 encoding (use SVGZ or minification)
  3. Consider WebP or optimized image format instead of SVG if possible

**FileReader.readAsDataURL() Can Stall on Large Files:**
- Problem: Reading large image files as data URL blocks main thread; no progress indication
- Files: `/Users/thihaaung/profile-frame/index.html` (line 454: `r.readAsDataURL(file)`)
- Cause: Synchronous in terms of main thread blocking; no streaming or chunked reading
- Improvement path:
  1. Add progress event listener to FileReader (`progress` event)
  2. Consider using Blob.slice() for streaming read (though less common for images)
  3. Add loading spinner while FileReader processes

## Fragile Areas

**DOM Query Selector Brittle to HTML Changes:**
- Files: `/Users/thihaaung/profile-frame/index.html` (throughout script: `document.getElementById()` calls)
- Why fragile: Hardcoded element IDs scattered throughout script; renaming any ID breaks multiple functions
- Safe modification: Use a centralized object for element references (e.g., `const EL = { canvas: document.getElementById('canvas'), ... }`)
- Test coverage: No tests; manual regression testing required after any HTML changes

**Toast Function Uses Global Timeout Handle:**
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 413-419: `toast()` function, line 417: `el._t` timeout)
- Why fragile: Attaches `_t` property to DOM element; if multiple toasts called rapidly, timeouts may conflict
- Safe modification: Use a Map or WeakMap to track timeouts per element, or implement proper toast queue
- Test coverage: No tests; edge cases like rapid successive toasts untested

**Click Handler Binding in IIFE:**
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 396-410: buildGrid IIFE, line 402: `onclick` handlers)
- Why fragile: Frame grid built with inline onclick handlers; no event delegation; if grid rebuilt, handlers may orphan
- Safe modification: Use event delegation on parent element instead of individual onclick attributes
- Test coverage: No tests; grid rebuilding (if needed in future) could break selection

**Hard-Coded Canvas Dimensions:**
- Files: `/Users/thihaaung/profile-frame/index.html` (line 469: `cv.width = S` where `S = frameImg.naturalWidth || 1080`)
- Why fragile: Fallback to 1080px assumes frame is always 1080x1080; if frame sizing changes, all calculations break
- Safe modification: Explicitly define frame dimensions in config, assert on load, validate before render
- Test coverage: No validation of frame dimensions at load time

## Test Coverage Gaps

**No Unit Tests:**
- What's not tested: Zoom calculation (`zoomPct` clamping), pan calculation (no bounds), canvas rendering logic, photo aspect ratio handling, image load race conditions
- Files: `/Users/thihaaung/profile-frame/index.html` (entire file)
- Risk: Regressions in zoom/pan math could produce incorrect frame compositions; no safety net for refactors
- Priority: High - core rendering logic is untested

**No Integration Tests:**
- What's not tested: Frame selection → photo upload → zoom → download flow; error scenarios (missing image, invalid file, etc.)
- Files: `/Users/thihaaung/profile-frame/index.html`
- Risk: End-to-end flow breaks silently; user-facing errors not caught
- Priority: High - user workflow should be verified

**No Browser/Device Testing:**
- What's not tested: Responsive behavior on various screen sizes, touch vs. mouse interactions on actual devices, canvas rendering on different browser engines
- Files: `/Users/thihaaung/profile-frame/index.html` (CSS media queries, touch event handlers)
- Risk: Media queries have multiple breakpoints but no validation that they render correctly; touch drag may be broken on specific browsers
- Priority: Medium - many users will hit responsive code paths

**No Image Processing Tests:**
- What's not tested: Aspect ratio calculations for photos (landscape, portrait, square), clipping to circular mask, pan bounds, zoom range limits
- Files: `/Users/thihaaung/profile-frame/index.html` (lines 482-490: aspect ratio and pan logic in `render()`)
- Risk: Certain image aspect ratios may clip incorrectly; pan may go out of bounds; zoom may not work for very tall/wide photos
- Priority: High - core feature; many photos will exercise these paths

## Scaling Limits

**Single-Page Limitation:**
- Current capacity: Single HTML file with 3 hardcoded frames; can support hundreds of users (client-side only)
- Limit: Cannot track user history, store generated images, or serve multiple frame packs without modifying code
- Scaling path: Move to client-server architecture to support frame management, image history, sharing, and analytics

**Memory Usage for Large Photos:**
- Current capacity: Typically handles 5-10MB JPEG in browser comfortably
- Limit: Very large photos (> 20MB) or rapid successive uploads may exhaust memory; no cleanup of previous images
- Scaling path: Add image compression before processing, implement memory pooling for canvas, add upper size limit with user feedback

**No Persistence:**
- Current capacity: Single session; generated frames lost on page refresh
- Limit: Cannot save drafts or provide frame templates
- Scaling path: Add localStorage for draft state, cloud storage for final images, user accounts for history

## Missing Critical Features

**No Error Recovery:**
- Problem: Invalid inputs or failures (bad image, missing frame) leave UI in inconsistent state; no way to recover except full reset
- Blocks: Users cannot troubleshoot without clearing everything; support burden increases

**No Accessibility (a11y):**
- Problem: No alt text on dynamically generated images, no keyboard navigation, no focus management, emojis used for icons (screen readers read "pile of poo" for upload icon)
- Blocks: Visually impaired users cannot use app; keyboard-only navigation unsupported

**No Undo/Redo:**
- Problem: Once user adjusts zoom or pan, no way to go back to previous state except reset and start over
- Blocks: Advanced users cannot explore variations easily

**No Mobile App:**
- Problem: Web app works on mobile but not installable or optimizable for mobile workflows
- Blocks: Cannot offer app store distribution or optimized offline experience

---

*Concerns audit: 2026-04-08*
