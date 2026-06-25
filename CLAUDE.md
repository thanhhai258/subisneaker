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
index.html            ← all HTML/CSS/JS
index.html.bak        ← backup of index.html — do not delete
logo.png              ← SUBI STORE logo
img1.jpg–img9.jpg     ← carousel product images (600×600px)
color-xanhreu.jpg     ← color swatch - Xanh rêu (300×300px)
color-xanhthan.jpg    ← color swatch - Xanh than (300×300px)
video.mp4             ← product demo video (loop enabled)
```

**Page sections (top → bottom)**:
1. Header — orange gradient, `logo.png` left + "SUBI SNEAKER" centered + tagline
2. Carousel — 9 product images (`IMAGES[]`), thumbnails (xám `#e8e8e8` placeholder), arrows, dots, swipe, auto-play 4s
3. Product info — title, countdown timer (flash sale kết thúc 22:00), price block, sold count badge, trust badges, info box
4. Video section — `video.mp4` with custom play button overlay, `loop` enabled
5. Sale banner
6. Features grid (2×2)
7. Size selector (39–44) — stock badge động theo size (`stockBySize` object), bảng quy đổi size HTML
8. Order form (`#order-form`) — name, phone (auto-format), address, size dropdown, color picker, ghi chú (optional)
9. Guarantee box — 4 dòng với icon ✓ tròn xanh lá
10. Reviews (5 cards) — avatar màu đa dạng, badge "Đã mua hàng"
11. Shop footer — contact block (orange bg) + chính sách bán hàng
12. Sticky bottom bar — Nhắn tin (Facebook Page) + MUA NGAY (pulse animation)
13. Social proof popup — 15 entries in `socialData[]`, appears 3s, rotates 8s, visible 3.5s
14. Success modal

**Order flow**:
1. Customer fills form: name, phone, address, size, màu sắc, ghi chú (optional)
2. `submitOrder()` validates inline (no alert) — shows `.field-error` under each invalid field
3. Sends GET request to Google Apps Script URL — `mode: 'no-cors'` is intentional
4. Params: `ho_ten`, `so_dien_thoai`, `size_giay`, `mau_sac`, `dia_chi`, `san_pham`, `ghi_chu`
5. On success: fires `fbq('track', 'Purchase', ...)` then shows success modal

**Key JS functions**:
- `selectSize(el, size)` — syncs `.size-btn` to `#inputSize`, `selectedSize` var, và cập nhật `stockBySize` badge
- `selectColor(el, color)` — syncs `.color-option.sel` and `selectedColor` var
- `formatPhone(input)` — auto-formats phone thành `0987 654 321` khi gõ
- `setError(id, errId, show)` — toggle inline error state (`.error` class + `.field-error.show`)
- `submitOrder()` — validates all fields inline, sends to Google Sheet, fires Pixel, shows modal
- `showSuccess()` — clears form (bao gồm `inputNote`), removes size/color selection, shows modal
- `playVideo()` — hides overlay, plays `#productVideo`
- Countdown timer — IIFE, counts to `23:59:59` of current day. **Note**: the HTML label displays "22:00" but the JS target (`end.setHours(23,59,59,0)`) is 23:59:59 — an intentional inconsistency; change both together if updating
- Social proof — cycles through `socialData[]`

**Stock by size** (`stockBySize` object in JS):
```js
var stockBySize = { '39':17, '40':8, '41':11, '42':5, '43':14, '44':9 };
```
Badge đổi màu đỏ khi ≤ 8 đôi. Cập nhật thủ công khi cần.

**Google Apps Script URL**: Lives in `submitOrder()` as `SCRIPT_URL`. Do not change.

**Carousel**: Built in JS at runtime. `IMAGES[]` array holds 9 filenames. `goTo(idx)` updates `transform: translateX`, dots, thumbnails, lazy-loads images. Touch swipe threshold 40px. Thumbnail scroll uses manual `scrollLeft`. Thumbnails have `background: #e8e8e8` placeholder while lazy-loading.

**Lazy loading**: Only `img1.jpg` loads on open. Images 2–9 use `data-src`, loaded on demand in `goTo()` — current + next slide together.

## Design System

- **Colors**: Background `#f5f5f5` (page), `#fff` (cards), accent `#f07820` (orange), text `#222`
- **Fonts**:
  - `Bebas Neue` — numbers and Latin-only labels (prices, "MUA NGAY", countdown digits)
  - `Barlow` — all Vietnamese text, body copy, buttons with diacritics, headings
  - **Rule**: Never use `Bebas Neue` for Vietnamese diacritics — use `Barlow` 700–800 + `text-transform: uppercase`
- **Layout**: Mobile-first, `max-width: 480px`, centered, `padding-bottom: 72px`
- **Color picker**: `.color-option.sel` highlights with orange border. Selected state tracked in `selectedColor` JS var.
- **Guarantee icon**: `.g-check` — hình tròn xanh lá `#27ae60`, 22×22px, chứa ✓ trắng
- **Review avatar**: 40×40px, màu đa dạng theo người, badge `.rev-verified` xanh lá

## Facebook Pixel

Pixel ID: `3209318139254957` — code lives in `<head>` of `index.html`.

Events tracked:
- `PageView` — fires on page load
- `Purchase` — fires in `submitOrder()` after successful order send
- `TimeOnPage_30_seconds` — custom event, fires after 30s on page
- `ScrollDepth_50_percent` — custom event, fires once when user scrolls past 50% page height

Domain verified via:
```html
<meta name="facebook-domain-verification" content="ssr88ye94n9uzueqeg232t3l7tnk51" />
```

## Product images

Source images: `/Users/tunbui/Downloads/SUBI-STORE/SP1/web/` (carousel) and `/Users/tunbui/Downloads/SUBI-STORE/SP1/` (color swatches: `17.png` = Xanh rêu, `18.png` = Xanh than).

**Carousel images** — center-crop vuông 1:1, resize 600×600px, JPEG quality 72:
```python
from PIL import Image
img = Image.open("path/to/image.jpg").convert("RGB")
w, h = img.size
if h > w:
    top = (h - w) // 2
    img = img.crop((0, top, w, top + w))
elif w > h:
    left = (w - h) // 2
    img = img.crop((left, 0, left + h, h))
img = img.resize((600, 600), Image.LANCZOS)
img.save("img10.jpg", format="JPEG", quality=72, optimize=True)
```

**Color swatch images** — resize 300×300px, JPEG quality 80:
```python
img = Image.open("path/to/color.png").convert("RGB")
img = img.resize((300, 300), Image.LANCZOS)
img.save("color-newcolor.jpg", format="JPEG", quality=80, optimize=True)
```

## Git

```bash
# Push lần đầu hoặc khi bị lỗi HTTP 400 (file lớn):
git config http.postBuffer 524288000
git push origin main
```

## External links

- **Facebook Page**: `https://www.facebook.com/profile.php?id=61570114722211`
- **Google Sheet webhook**: `SCRIPT_URL` inside `submitOrder()` — do not change
- **GitHub repo**: `https://github.com/thanhhai258/subisneaker.git` (branch `main`)
- **Live site**: `https://subisneaker.netlify.app`

## Deployment

**Files required**: `index.html` + `logo.png` + `img1.jpg`–`img9.jpg` + `color-xanhreu.jpg` + `color-xanhthan.jpg` + `video.mp4`

- **Auto-deploy**: Repo linked to Netlify — every push to `main` triggers deploy
- **Manual (Netlify drag & drop)**: Drag toàn bộ folder — never drag only `index.html`
- **GitHub Pages**: Settings → Pages → branch `main` → `/root` → `https://thanhhai258.github.io/subisneaker/`
