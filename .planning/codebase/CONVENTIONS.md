# Coding Conventions

**Analysis Date:** 2026-04-08

## Overview

This is a vanilla HTML/CSS/JavaScript single-page application. Code is contained entirely within `index.html` with no external frameworks, build tools, or type checking. Conventions reflect a pragmatic approach optimized for simplicity and direct browser execution.

## Naming Patterns

**Files:**
- Single-file application: `index.html`
- All assets (frames) embedded as data URIs or in `frames/` directory
- No separate JS/CSS/HTML file splitting

**Functions:**
- camelCase for all function names: `loadPhoto()`, `zoom()`, `render()`, `download()`
- Verbs used as function prefixes: `load`, `pick`, `zoom`, `render`, `download`, `reset`, `toast`
- Private functions wrapped in IIFE: `(function setupDrag() { ... })()`
- Single-letter function parameters acceptable in closures: `e`, `ev`, `f`, `i`

**Variables:**
- camelCase for global state variables: `selIdx`, `frameImg`, `photoImg`, `zoomPct`, `panX`, `panY`, `dragging`
- Abbreviated names for common properties: `dw` (draw width), `dh` (draw height), `sc` (scale), `pa` (photo aspect ratio), `cv` (canvas)
- Abbreviated event handler parameters: `e`, `ev`, `r` (FileReader), `t` (touch event)

**HTML IDs and Classes:**
- HTML IDs use abbreviations prefixed with type intent: `frameGrid`, `upArea`, `photoIn`, `dlBtn`, `uicon`, `utitle`, `udone`, `zval`, `dragHint`, `ph`
- HTML IDs for dynamic frame cards: `fc0`, `fc1`, `fc2` (frame-card-index)
- CSS class names kebab-case: `.frame-card`, `.frame-grid`, `.upload-area`, `.zoom-wrap`, `.btn-dl`, `.btn-rst`, `.preview-card`, `.drag-hint`

**Constants:**
- Uppercase for configuration objects: `FRAME_CONFIG`

## Code Style

**Formatting:**
- No external formatter (no Prettier, ESLint, etc.)
- Indentation: 2 spaces for HTML/CSS, 4 spaces for JavaScript
- Semicolons: Required; statements must end with semicolons
- Line length: No enforced limit, but generally compact

**CSS Organization:**
- CSS Variables (custom properties) for theming at `:root`: `--maroon`, `--gold`, `--bg`, `--text`, `--muted`, `--radius`
- Color palette defined at top of CSS: dark maroon background, gold accents, cream text
- Media queries organized at end of CSS by breakpoint: small phones (≤380px), mobile (381-480px), tablet (481-768px), desktop (769px+), landscape orientation

**JavaScript Style:**
- Comments use `// ──` delimiter for section headers within the code
- Prefer inline arrow functions for event handlers: `onclick="zoom(-10)"` or `.onclick = () => { ... }`
- Favor direct DOM manipulation with `document.getElementById()` and `.classList`
- No destructuring; parameters used as-is
- Ternary operators for simple conditionals; avoid verbose if-else in render logic

## Comment Patterns

**When to Comment:**
- Section headers marked with: `// ── [Purpose] ──`
- Explain "why" not "what": Used for configuration notes and data flow explanation
- HTML comments for major sections: `<!-- Step 1: Frame -->`, `<!-- Preview -->`
- Frame SVG data comments explain structure: `<!-- Ring -->`, `<!-- Text -->`, `<!-- Stars -->`

**No JSDoc/TSDoc:**
- No formal documentation generation
- Parameter types implicit from usage context
- Function intent conveyed through verb-based naming

## Import Organization

**Not Applicable:**
- No module system (no ES6 imports)
- No dependencies or external libraries
- All code in single `<script>` block within HTML

## Error Handling

**Patterns:**
- Try-catch only in critical operations: `download()` wraps `toBlob()` with fallback to `toDataURL()`
- Fallback behavior preferred to error throwing: Download failure falls back to opening data URL in new window
- Silent validation: Check state before operations (e.g., `if (!file) return;` in `loadPhoto()`)
- FileReader error handling implicit: Relies on `.onload` callback, no explicit error handler

## Logging

**Framework:** No logging library; uses `console` implicitly available.

**Patterns:**
- Toast notifications used for user feedback, not console logs
- No debug logging in production code
- Toast messages localized in Burmese: `toast('✓ Frame ရွေးပြီး')`

## Module Design

**Patterns:**
- IIFE for encapsulation: `(function setupDrag() { ... })()` for drag handling
- Direct function definitions for event handlers: `function loadPhoto(e)`, `function zoom(d)`
- Shared state via global scope: `let selIdx`, `let frameImg`, etc. (no class/object encapsulation)
- Configuration via constant object: `FRAME_CONFIG` with frame metadata

## Event Handling

**Patterns:**
- Inline `onclick` attributes in HTML for simple handlers: `onclick="zoom(-10)"`
- Event listener attachment for complex interactions: `.addEventListener('mousedown', handler)`
- Passive event listeners explicitly disabled for drag handlers: `{ passive: false }`
- Event delegation avoided; direct element targeting
- `e.preventDefault()` used liberally for drag/touch interactions

## Function Design

**Size Guidelines:**
- Prefer compact functions under 15 lines
- `render()` at 30 lines is the most complex operation
- Helper functions used to avoid nesting: `onStart()`, `onMove()`, `onEnd()` inside drag IIFE

**Parameters:**
- Functions accept 0-2 parameters maximum
- Single event parameter for handlers: `function(e)`
- Index parameter for array operations: `function(idx)`
- Delta parameter for incremental changes: `function(d)` in `zoom(d)`

**Return Values:**
- Most functions have side effects (DOM mutation) instead of returning values
- `render()`, `zoom()`, `loadPhoto()`, `download()` all return undefined
- FileReader callbacks use nested function closures, not returns

## Canvas Operations

**Patterns:**
- State saved/restored: `ctx.save()` and `ctx.restore()` around clip region
- Clipping used to constrain photo to circular boundary
- Translate-then-draw pattern: `ctx.translate()` moves origin before `drawImage()`
- Aspect ratio calculated: `pa = photoImg.naturalWidth / photoImg.naturalHeight`

## Responsive Design

**Approach:**
- Mobile-first CSS with media queries for larger screens
- CSS variables adjust for safe areas: `--safe-bottom: env(safe-area-inset-bottom)`
- Container max-width caps layout at 600px on desktop
- No JavaScript-based responsive behavior; pure CSS

---

*Convention analysis: 2026-04-08*
