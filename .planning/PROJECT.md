# PRF Profile Frame Generator

## What This Is

A single-page web app that lets PRF supporters create profile pictures by combining their photo (or a pre-made avatar) with a circular frame overlay. Users select a frame, upload a photo or choose an avatar, adjust positioning, and download the result. Burmese-language UI with a maroon & gold theme.

## Core Value

Users can quickly generate a framed profile picture — either from their own photo or a stylized PRF avatar — and download it instantly.

## Requirements

### Validated

- ✓ Frame selection (circular overlays with Burmese text) — existing
- ✓ Photo upload with zoom/drag repositioning — existing
- ✓ Canvas-based preview and PNG download — existing
- ✓ Fully responsive design (mobile, tablet, desktop) — existing
- ✓ Base64 inline data URIs for cross-origin safe downloads — existing

### Active

- [ ] Replace current 3 SVG frames with 3 new PNG frames from Photo_Frame_Apr_2026 (PF_1_Thin, PF_2_Thin, PF_3_Thin)
- [ ] Add avatar selection — 8 pre-made avatars (4 styles x Male/Female) displayed in a grid
- [ ] Avatar grid appears alongside photo upload area — user can upload a photo OR pick an avatar, last choice wins
- [ ] Selected avatar works with same zoom/drag controls as uploaded photos
- [ ] Avatar + frame combination renders on canvas and downloads as PNG

### Out of Scope

- Social sharing / link sharing — not requested, keep it simple download-only
- Photo editing (filters, crop, brightness) — not needed
- Multi-page app / routing — stays as single HTML file
- User accounts / saving — stateless tool

## Context

- Single `index.html` file, no build tools, no dependencies
- Avatar assets in `Photo_Frame_Apr_2026/` folder: Avatar 1-4 with _M and _F variants (PNG)
- New frame assets in same folder: PF_1_Thin.png, PF_2_Thin.png, PF_3_Thin.png
- Current frames are inline SVG base64 data URIs — new PNG frames will need same treatment for download safety
- Existing codebase map at `.planning/codebase/`

## Constraints

- **Tech stack**: Single HTML file, vanilla JS, no frameworks — match existing approach
- **Assets**: Must embed frames/avatars as base64 data URIs for file:// and cross-origin download support
- **Language**: UI text in Burmese (Myanmar)
- **Responsiveness**: Must maintain current responsive breakpoints

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Replace SVG frames with PNG frames | User has new designed PNG frames, old SVGs no longer needed | — Pending |
| Show all 8 avatars in grid | Simpler UX, no gender toggle step needed | — Pending |
| All-in-one flow (avatar alongside upload) | Less friction, last choice wins | — Pending |
| Avatars use same zoom/drag as photos | Consistent UX, users may want to reposition avatar in frame | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-08 after initialization*
