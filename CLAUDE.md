# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Vietnamese COD (cash-on-delivery) sneaker landing page for SUBI-STORE. Sells "Ultra Run Pro 2026" men's sneakers at 250.000đ (sale từ 360.000đ, -30%). No build system — open `index.html` directly in a browser or deploy to Netlify.

## Running locally

```bash
python3 -m http.server 8765
# then open http://localhost:8765/index.html
```

The `.claude/launch.json` is already configured — use `preview_start` with name `"subisneaker"` to launch via the preview tool.

## Architecture

**File structure**:
```
index.html          ← all HTML/CSS/JS
logo.png            ← SUBI STORE logo
img1.jpg–img9.jpg   ← carousel product images (600×600px)
color-xanhreu.jpg   ← color swatch thumbnail - Xanh rêu (300×300px)
color-xanhthan.jpg  ← color swatch thumbnail - Xanh than (300×300px)
video.mp4           ← product demo video (loop enabled)
```

**Page sections (top → bottom)**:
1. Header — orange gradient, `logo.png` absolute left + "SUBI SNEAKER" centered + tagline
2. Carousel — 9 product images (`IMAGES[]` array), thumbnails, arrows, dots, swipe, auto-play 4s
3. Product info — title, countdown timer, price (250.000đ / 360.000đ / -30%), trust badges, info box
4. Video section — `video.mp4` with custom play button overlay, `loop` enabled
5. Sale banner
6. Features grid (2×2)
7. Size selector (39–44)
8. Order form (`#order-form`) — includes size dropdown + color picker
9. Guarantee box
10. Reviews (5 cards)
11. Shop footer — contact info block (orange bg) + sales policy block
12. Sticky bottom bar — Nhắn tin (Facebook Page) + MUA NGAY
13. Social proof popup — 15 entries in `socialData[]`, appears 3s, rotates 8s, visible 3.5s
14. Success modal

**Order flow**:
1. Customer fills form: name, phone, address, size, **màu sắc (color)**
2. `submitOrder()` validates all fields including color selection
3. Sends GET request to Google Apps Script URL — `mode: 'no-cors'` is intentional
4. Params include: `ho_ten`, `so_dien_thoai`, `size_giay`, `mau_sac`, `dia_chi`, `san_pham`
5. On success: fires `fbq('track', 'Purchase', ...)` then shows success modal

**Key JS functions**:
- `selectSize(el, size)` — syncs `.size-btn` to `#inputSize` and `selectedSize` var
- `selectColor(el, color)` — syncs `.color-option` selection to `selectedColor` var
- `submitOrder()` — validates (including color), sends to Google Sheet, fires Pixel, shows modal
- `showSuccess()` — clears form, removes size/color selection, shows modal
- `playVideo()` — hides overlay, plays `#productVideo`
- Countdown timer — IIFE, counts to 23:59:59 of current day
- Social proof — cycles through `socialData[]`

**Google Apps Script URL**: Lives in `submitOrder()` as `SCRIPT_URL`. Do not change — breaking it stops order collection.

**Carousel**: Built in JS at runtime. `IMAGES[]` array (line ~396) holds 9 filenames. `goTo(idx)` updates `transform: translateX`, dots, thumbnails, lazy-loads images. Touch swipe threshold 40px. Thumbnail scroll uses manual `scrollLeft` (not `scrollIntoView`) to avoid page scroll side-effects.

**Lazy loading**: Only `img1.jpg` loads on open. Images 2–9 use `data-src`, loaded on demand in `goTo()` — current + next slide together. Same for thumbnails.

## Design System

- **Colors**: Background `#f5f5f5` (page), `#fff` (cards), accent `#f07820` (orange), text `#222`
- **Fonts**:
  - `Bebas Neue` — numbers and Latin-only labels (prices, "MUA NGAY", countdown digits)
  - `Barlow` — all Vietnamese text, body copy, buttons with diacritics, headings
  - **Rule**: Never use `Bebas Neue` for Vietnamese diacritics — use `Barlow` 700–800 + `text-transform: uppercase`
- **Layout**: Mobile-first, `max-width: 480px`, centered, `padding-bottom: 72px`
- **Shop footer contact block**: `.shop-info-block-contact` — orange bg, white text + white SVG icons
- **Color picker**: `.color-option.sel` highlights with orange border + shadow. Selected state tracked in `selectedColor` JS var.

## Product images

Source images: `/Users/tunbui/Downloads/SUBI-STORE/SP1/web/` (carousel) and `/Users/tunbui/Downloads/SUBI-STORE/SP1/` (color swatches: `17.png` = Xanh rêu, `18.png` = Xanh than).

**Carousel images** — resize to 600×600px, JPEG quality 72:
```python
from PIL import Image
img = Image.open("path/to/image.jpg").convert("RGB")
w, h = img.size
if h > w:
    top = (h - w) // 2
    img = img.crop((0, top, w, top + w))
img = img.resize((600, 600), Image.LANCZOS)
img.save("img10.jpg", format="JPEG", quality=72, optimize=True)
```

**Color swatch images** — resize to 300×300px, JPEG quality 80:
```python
img = Image.open("path/to/color.png").convert("RGB")
img = img.resize((300, 300), Image.LANCZOS)
img.save("color-newcolor.jpg", format="JPEG", quality=80, optimize=True)
```

## Facebook Pixel

Pixel ID: `3209318139254957` — code lives in `<head>` of `index.html`.

Two events tracked:
- `PageView` — fires on page load (in Pixel base code)
- `Purchase` — fires in `submitOrder()` after successful order send

Domain `subisneaker.netlify.app` is verified via:
```html
<meta name="facebook-domain-verification" content="ssr88ye94n9uzueqeg232t3l7tnk51" />
```

## External links

- **Facebook Page**: `https://www.facebook.com/profile.php?id=61570114722211`
- **Google Sheet webhook**: `SCRIPT_URL` inside `submitOrder()` — do not change
- **GitHub repo**: `https://github.com/thanhhai258/subisneaker.git` (branch `main`)
- **Live site**: `https://subisneaker.netlify.app`

## Deployment

**Files required**: `index.html` + `logo.png` + `img1.jpg`–`img9.jpg` + `color-xanhreu.jpg` + `color-xanhthan.jpg` + `video.mp4`

- **Manual (Netlify drag & drop)**: Drag the entire project folder onto Netlify Deploys tab — never drag only `index.html`
- **Auto-deploy**: Repo linked to Netlify via GitHub — every push to `main` triggers deploy
- **Alternative (GitHub Pages)**: Enable via repo Settings → Pages → branch `main` → `/root` → live at `https://thanhhai258.github.io/subisneaker/`
