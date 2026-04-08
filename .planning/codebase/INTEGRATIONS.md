# External Integrations

**Analysis Date:** 2026-04-08

## APIs & External Services

**None** - No external API integrations present. Application is fully self-contained.

## Data Storage

**Databases:**
- Not applicable - No database integration

**File Storage:**
- Local filesystem only
- User photos loaded via FileReader API (`index.html` line 435-455)
- Download output directly to user device via blob export (`index.html` line 506-516)
- No cloud storage integration

**Caching:**
- Browser native cache
- Image preloading of frame assets (line 393)
- No external caching service

## Authentication & Identity

**Auth Provider:**
- None - No authentication system
- No user accounts required
- Fully anonymous, client-side application

## Monitoring & Observability

**Error Tracking:**
- Not configured - No error monitoring service
- Try/catch wrapper around canvas export (line 505-520) with fallback

**Logs:**
- None - No logging infrastructure
- Toast notifications for user feedback (line 413-419, 428, 450, 515, 520, 580)

## CI/CD & Deployment

**Hosting:**
- Not specified - Can be deployed to:
  - Static file hosting (GitHub Pages, Netlify, Vercel, etc.)
  - Web server (Apache, Nginx, etc.)
  - File:// protocol (runs locally without server)
  - Any CDN or content delivery service

**CI Pipeline:**
- Not configured - No CI/CD setup detected
- Git repository present (`.git/` directory)
- No workflow files (.github/workflows/, .gitlab-ci.yml, etc.)

## Environment Configuration

**Required env vars:**
- None - No environment variables needed

**Secrets location:**
- Not applicable - No secrets or credentials

## Webhooks & Callbacks

**Incoming:**
- None

**Outgoing:**
- None

## CORS & Cross-Origin

**Status:**
- Not applicable for file:// protocol
- If deployed to domain: Single-origin application
- No cross-origin resource requests (all resources embedded or local)

## Performance & Delivery

**Content Delivery:**
- Single HTML file (25.2 KB) contains everything:
  - HTML structure
  - CSS styles
  - JavaScript code
  - SVG frames (base64-encoded)
  
**Zero External Dependencies:**
- No npm packages
- No CDN libraries
- No web fonts
- No remote scripts
- No API calls

**Asset Sizes:**
- Frame SVGs embedded as base64 data URIs (lines 382-384)
- Logo embedded as base64 data URI (line 321)
- Reduces HTTP requests to 1 (the HTML file itself)

---

*Integration audit: 2026-04-08*
