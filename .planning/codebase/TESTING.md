# Testing Patterns

**Analysis Date:** 2026-04-08

## Test Framework

**Status:** No automated testing framework detected.

**Runner:** Not applicable

**Assertion Library:** Not applicable

**Configuration:** None

**Current Coverage:** 0% — No test files exist in the codebase.

## Testing Approach

This is a single-file HTML application without any test infrastructure. All validation occurs through manual testing in a browser environment.

## Manual Testing Surface Area

**Frame Selection:**
- User clicks frame card → `pick(idx)` executes
- Frame card DOM updates with `.selected` class
- Frame image preloaded state triggers `render()` or defers to `.onload`
- Toast notification confirms selection

**Photo Upload:**
- File input triggers `loadPhoto(e)`
- FileReader reads file as data URL
- Image loads asynchronously via `.onload` callback
- DOM elements toggle visibility (`#udone`, `#usub`)
- Upload area gains `.done` class styling

**Zoom Controls:**
- Plus/minus buttons call `zoom(+10)` or `zoom(-10)`
- Zoom percentage clamped: `Math.max(40, Math.min(220, zoomPct + d))`
- DOM displays updated zoom value in `#zval`
- Canvas re-renders with new scale

**Drag-to-Reposition:**
- Canvas accepts mouse and touch events
- `setupDrag()` IIFE manages state: `dragging`, `dragStartX`, `dragStartY`, `panStartX`, `panStartY`
- Drag delta converted to canvas-space pixels via `getCanvasScale()`
- Pan values constrain photo position inside circular mask
- Visual feedback: cursor changes to `grab` / `grabbing`

**Canvas Rendering:**
- `render()` checks if frame and photo exist before drawing
- Canvas dimensions set to frame SVG natural width (1080px)
- Photo clipped to circular mask using `ctx.arc()` and `ctx.clip()`
- Photo scaled by `zoomPct / 100`
- Aspect ratio preserved via conditional logic:
  ```javascript
  if (pa > 1) { dh = S * sc; dw = dh * pa; }
  else { dw = S * sc; dh = dw / pa; }
  ```
- Frame overlay drawn on top without transformation

**Download:**
- Canvas converted to blob via `toBlob()`
- Blob URL created and auto-clicked for download
- Fallback: If `toBlob()` throws, `toDataURL()` opens in new window
- Success toast displays after operation

**Reset:**
- All state variables reset to initial values
- DOM elements restored to default state
- Frame card `.selected` class removed
- Photo file input cleared
- Canvas hidden, placeholder shown

## Browser APIs Tested

**Canvas API:**
- `getContext('2d')`
- `drawImage()` with positioning
- `beginPath()`, `arc()`, `clip()`
- `save()` and `restore()` for context state
- `toBlob()` with error fallback to `toDataURL()`
- `translate()` for coordinate transformation

**File API:**
- `FileReader.readAsDataURL()`
- Image element `.onload` callback
- File input `.files` array access

**DOM API:**
- `document.getElementById()`
- `document.querySelectorAll()` with `.forEach()`
- `.classList.add()`, `.classList.remove()`, `.classList.toggle()`
- `.style.display` direct manipulation
- `.textContent` and `.innerHTML` assignment

**Event Handling:**
- `onclick` inline attributes
- `addEventListener()` for mouse and touch events
- `e.preventDefault()` for drag interactions
- Touch event object `.touches[0]` for multi-touch filtering

**Browser Compatibility Concerns:**
- `URL.createObjectURL()` and `URL.revokeObjectURL()` for blob handling
- CSS custom properties (`:root` vars) for theming
- CSS Grid for frame layout
- Touch events may need vendor prefixes or polyfills on older browsers
- SVG data URIs inline in JS must be base64-encoded to avoid CORS

## Testing Gaps

**Not Tested:**
- Cross-browser canvas rendering differences (color accuracy, anti-aliasing)
- Touch event handling on various devices (iPad, Android)
- File size limits or memory constraints with large images
- SVG frame rendering on browsers without full SVG support
- Download fallback path (never triggered in modern browsers)
- Safe-area insets on notched devices (CSS-only, not testable in code)
- Mobile viewport meta tags and scaling behavior
- Landscape orientation constraints on small screens
- Responsive grid layout at exact breakpoint thresholds

## Manual Test Checklist

**Basic Flow:**
- [ ] Page loads without errors in browser console
- [ ] Frame grid displays all 3 frames correctly
- [ ] Click frame card → selection state visible
- [ ] Upload photo file (JPG, PNG, WEBP)
- [ ] Photo upload state shows checkmark and "done" message
- [ ] Canvas preview displays photo inside frame ring
- [ ] Download button becomes enabled after frame + photo selected

**Zoom Interaction:**
- [ ] Zoom minus button decreases percentage (minimum 40%)
- [ ] Zoom plus button increases percentage (maximum 220%)
- [ ] Canvas re-renders at new zoom level
- [ ] Photo scales smoothly without distortion

**Drag Interaction:**
- [ ] Click and drag canvas to reposition photo
- [ ] Pan state persists across zoom changes
- [ ] Photo stays within circular mask boundary
- [ ] Cursor shows grab/grabbing feedback
- [ ] Touch drag works on mobile devices

**Download:**
- [ ] Click download button → file saves as `PRF-frame.png`
- [ ] Downloaded image is 1080x1080 pixels
- [ ] Photo position matches preview
- [ ] Frame overlay visible with correct colors
- [ ] Success toast appears after download

**Reset:**
- [ ] Click reset button → all state clears
- [ ] Frame selection deselected
- [ ] Upload area shows camera icon again
- [ ] Canvas hidden, placeholder shown
- [ ] Download button disabled
- [ ] Zoom reset to 100%

**Responsive Testing:**
- [ ] Layout adjusts for small phones (320px)
- [ ] Layout adjusts for mobile (380px+)
- [ ] Layout adjusts for tablet (480px+)
- [ ] Layout adjusts for desktop (768px+)
- [ ] Landscape orientation: frames scroll horizontally
- [ ] Text remains readable on all sizes
- [ ] Touch targets (buttons) are ≥44px on mobile

**Accessibility:**
- [ ] Keyboard navigation works (if applicable)
- [ ] Color contrast meets WCAG standards
- [ ] Title and alt text present on images
- [ ] Toast messages readable with screen readers

## No Testing Infrastructure

**Why:**
- Single-file application designed for immediate browser deployment
- No build step or dependencies requiring test harness
- Direct file:// or served directly without bundling
- Target users are non-technical (frame selection, photo upload)

**If Testing Were Added:**
- Framework: Vitest or Jest for unit tests
- E2E: Playwright or Cypress for browser interactions
- Mock Canvas: sinon stubs for `drawImage()`, `toBlob()`
- Mock FileReader: Stub `readAsDataURL()` with test data
- Test structure: One test file per major function (`test/render.test.js`, `test/drag.test.js`)

---

*Testing analysis: 2026-04-08*
