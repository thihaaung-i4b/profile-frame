# Technology Stack

**Analysis Date:** 2026-04-08

## Languages

**Primary:**
- HTML5 - Document structure and semantic markup
- JavaScript (Vanilla ES5+) - Application logic, canvas manipulation, file handling
- CSS3 - Styling, responsive design, gradients, animations, transitions

**No external language compilers** - Pure browser-native code, no TypeScript, no preprocessors.

## Runtime

**Environment:**
- Browser (All modern browsers: Chrome, Firefox, Safari, Edge)
- No Node.js or backend runtime
- File:// protocol supported - Can run locally without server

**Platform:**
- Client-side only application
- Works on desktop and mobile browsers
- PWA-ready metadata present (see index.html head tags)

## Frameworks

**Core:**
- None - Vanilla JavaScript implementation
- No framework dependencies (no React, Vue, Angular, etc.)

**Canvas API:**
- Built-in browser Canvas 2D Context API
- Used for image compositing and rendering

**File APIs:**
- FileReader API - For loading user photos
- Blob API - For image export
- URL.createObjectURL() - For download handling

**Build/Dev:**
- None - Single HTML file serves as complete application

## Key Dependencies

**Critical:**
- None - Zero npm/package dependencies
- All functionality implemented with native browser APIs

**External Resources:**
- System fonts only (no web font imports)
- No CDN libraries
- No third-party scripts

## Configuration

**Environment:**
- No environment variables required
- Single configuration object `FRAME_CONFIG` embedded in HTML (line 381)
- Frame definitions hardcoded as base64-encoded SVG data URIs

**Build:**
- No build process
- No build config files present

## File Organization

**Primary Files:**
- `index.html` (585 lines) - Monolithic application file containing:
  - HTML structure
  - Embedded CSS styles
  - Embedded JavaScript logic
  - Embedded SVG frame graphics (as data URIs)

**Static Assets:**
- `frames/frame1.svg` - Reference SVG frame 1
- `frames/frame2.svg` - Reference SVG frame 2
- `frames/frame3.svg` - Reference SVG frame 3
- `frames/logo.svg` - Application logo

## Features by Technology

**Image Processing:**
- Canvas 2D API for compositing (line 467)
- Image masking with circular clipping (line 474-480)
- Zoom scaling with aspect ratio preservation (line 483-487)
- Pan/drag repositioning with touch and mouse events (line 524-559)

**File Handling:**
- FileReader for reading user-uploaded photos (line 435-455)
- Canvas.toBlob() for PNG export (line 506)
- Fallback to toDataURL() for compatibility (line 518)
- Download via dynamically created anchor tag (line 508-513)

**UI Interactions:**
- DOM manipulation with vanilla JavaScript
- Event listeners for clicks, touches, drag operations
- CSS transitions and transforms for visual feedback
- Toast notifications with auto-hide timers (line 413-419)

**Responsive Design:**
- CSS media queries for 5 breakpoints (lines 238-313)
- safe-area-inset support for notched devices
- Flexbox and CSS Grid for layout
- touch-action: none for canvas interactions

## Platform Requirements

**Development:**
- Text editor or IDE (no build tools needed)
- Git for version control (present: `.git/` directory)
- Browser for testing

**Production:**
- Static file hosting (any web server or file:// protocol)
- No server-side processing required
- No database requirements
- No authentication system

---

*Stack analysis: 2026-04-08*
