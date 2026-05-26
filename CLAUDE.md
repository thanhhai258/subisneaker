# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Vietnamese COD (cash-on-delivery) sneaker landing page for SUBI-STORE. Single static HTML file selling "Ultra Run Pro 2025" men's sneakers at 790,000đ. No build system — open `index.html` directly in a browser or deploy to Netlify.

## Running locally

```bash
python3 -m http.server 8765
# then open http://localhost:8765/index.html
```

The `.claude/launch.json` is already configured — use `preview_start` with name `"subisneaker"` to launch via the preview tool.

## Architecture

**Single-file structure**: All HTML, CSS (inline `<style>`), and JS (inline `<script>`) live in `index.html`. No external JS files, no frameworks, no dependencies beyond Google Fonts. Product images are embedded as base64 data URIs — the file is intentionally large (~475KB).

**Page sections (top → bottom)**:
1. Header — brand bar (orange gradient, SUBI STORE logo)
2. Carousel — 6 product images with thumbnails, arrows, dots, swipe, auto-play
3. Product info — title, countdown timer, price, trust badges, info box
4. Sale banner
5. Features grid (2×2)
6. Size selector
7. Order form (`#order-form`)
8. Guarantee box
9. Reviews
10. Sticky bottom bar (Messenger + MUA NGAY)
11. Social proof popup (rotates every 8s, appears after 3s delay)
12. Success modal

**Order flow**:
1. Customer fills form (name, phone, address, size)
2. `submitOrder()` sends a GET request to the Google Apps Script URL with URL params — `mode: 'no-cors'` is intentional to bypass CORS
3. Order data lands in a Google Sheet via Apps Script
4. Success modal always shows after fetch (even on network error — `no-cors` response is opaque, try/catch swallows errors)

**Key JS functions**:
- `selectSize(el, size)` — syncs `.size-btn` selection to `#inputSize` and `selectedSize` var
- `submitOrder()` — validates, sends to Google Sheet, shows success modal
- `showSuccess()` — clears form fields, removes size selection, shows modal
- Countdown timer — IIFE, counts down to 23:59:59 of current day
- Social proof — cycles through `socialData[]` array, show 3.5s / hide 4.5s

**Google Apps Script URL**: Lives in `submitOrder()` as `SCRIPT_URL`. This is the live webhook — changing it breaks order collection.

**Carousel**: Built in JS at runtime. `IMAGES[]` array holds 6 base64 data URIs. `goTo(idx)` updates `transform: translateX`, dots, and thumbnails. Touch swipe threshold is 40px.

## Design System

- **Colors**: Background `#f5f5f5` (page), `#fff` (cards), accent `#f07820` (orange), text `#222`
- **Fonts**:
  - `Bebas Neue` — numbers, Latin-only labels (prices, "MUA NGAY", "SUBI STORE", countdown digits)
  - `Barlow` — all Vietnamese text, body copy, buttons with diacritics, headings
  - **Rule**: Never use `Bebas Neue` for text containing Vietnamese diacritics — fallback glyphs render at inconsistent sizes. Use `Barlow` weight 700–800 + `text-transform: uppercase` instead.
- **Layout**: Mobile-first, `max-width: 480px`, centered, `padding-bottom: 72px` (sticky bar clearance)

## Product images

Source images live in `/Users/tunbui/Downloads/SUBI-STORE/SP1/`. When adding or replacing carousel images:
1. Use Python + Pillow to resize to max 600px width and save as JPEG quality 72
2. Base64-encode and inject into the `IMAGES[]` array in `index.html`
3. The array is near the top of the `<script>` block

```python
from PIL import Image
import base64, io

img = Image.open("path/to/image.png").convert("RGB")
img = img.resize((600, int(img.height * 600 / img.width)), Image.LANCZOS)
buf = io.BytesIO()
img.save(buf, format="JPEG", quality=72, optimize=True)
b64 = "data:image/jpeg;base64," + base64.b64encode(buf.getvalue()).decode()
```

## Deployment

Static file — deploy by uploading `index.html` to Netlify, GitHub Pages, or any static host. No build step. Remote: `https://github.com/thanhhai258/subisneaker.git`
