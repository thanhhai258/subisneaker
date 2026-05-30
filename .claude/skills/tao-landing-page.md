---
name: tao-landing-page
description: >
  Tạo landing page bán giày COD tiếng Việt cho SUBI STORE. Dùng skill này khi
  người dùng muốn tạo landing page mới cho sản phẩm giày mới, thay ảnh/video
  sản phẩm, hoặc clone trang hiện tại sang dòng giày khác. Trigger khi người
  dùng nói: "tạo landing page", "làm trang mới cho giày", "có sản phẩm mới",
  "thay ảnh giày", "tạo trang cho [tên sản phẩm]", hoặc upload ảnh/video giày
  mới và muốn ra ngay một trang bán hàng.
---

# Skill: Tạo Landing Page Bán Giày SUBI STORE

Skill này giúp tạo ra một landing page bán giày COD hoàn chỉnh bằng cách clone
template hiện tại và thay thế nội dung theo sản phẩm mới.

## Template gốc

Template đang dùng nằm tại:
```
/Users/tunbui/Downloads/SUBI-STORE/Ladipage/subisneaker/
├── index.html       ← toàn bộ HTML/CSS/JS
├── logo.png         ← logo SUBI STORE (dùng lại, không đổi)
├── img1.jpg–img9.jpg
└── video.mp4
```

---

## Bước 1 — Thu thập thông tin sản phẩm

Hỏi người dùng các thông tin sau (có thể hỏi gộp một lần):

| Thông tin | Ví dụ | Ghi chú |
|---|---|---|
| Tên sản phẩm | `CloudWalk Pro 2026` | Dùng cho title, h1, alt ảnh |
| Giá bán | `290.000đ` | Giá sau giảm |
| Giá gốc | `420.000đ` | Giá gạch ngang |
| % giảm | `-30%` | Badge giảm giá |
| Ảnh sản phẩm | đường dẫn folder hoặc từng file | Chấp nhận .jpg/.png |
| Video sản phẩm | đường dẫn file .mp4 | Có thể bỏ qua nếu chưa có |
| Thư mục dự án mới | `cloudwalk-pro` | Tên folder output |
| Mô tả vật liệu | upper, đế, size range | Điền vào info-box |
| 4 tính năng nổi bật | icon emoji + tiêu đề + mô tả | Features 2×2 grid |

Nếu người dùng đã cung cấp ảnh/video trong lúc chat, không cần hỏi lại.

---

## Bước 2 — Xử lý ảnh sản phẩm

Dùng Python + Pillow để chuẩn hoá ảnh. Chạy script này (thay đường dẫn phù hợp):

```python
from PIL import Image
import os

# Danh sách ảnh nguồn (sắp xếp theo thứ tự muốn hiển thị trên carousel)
SOURCE_IMAGES = [
    "path/to/image1.jpg",
    "path/to/image2.png",
    # ...
]

OUTPUT_DIR = "/path/to/new-project/"
os.makedirs(OUTPUT_DIR, exist_ok=True)

for i, src in enumerate(SOURCE_IMAGES, start=1):
    img = Image.open(src).convert("RGB")
    w, h = img.size

    # Center-crop ảnh dọc (portrait) thành hình vuông
    if h > w:
        top = (h - w) // 2
        img = img.crop((0, top, w, top + w))
    elif w > h:
        left = (w - h) // 2
        img = img.crop((left, 0, left + h, h))

    # Resize về 600×600
    img = img.resize((600, 600), Image.LANCZOS)
    img.save(os.path.join(OUTPUT_DIR, f"img{i}.jpg"),
             format="JPEG", quality=72, optimize=True)
    print(f"✓ img{i}.jpg")

print(f"Xong! {len(SOURCE_IMAGES)} ảnh đã được xử lý.")
```

Ảnh nguồn nằm ở `/Users/tunbui/Downloads/SUBI-STORE/SP1/` hoặc subfolder `web/`.

---

## Bước 3 — Copy file và tạo dự án mới

```bash
# Tạo thư mục dự án mới
NEW_PROJECT="/Users/tunbui/Downloads/SUBI-STORE/Ladipage/<tên-dự-án>"
mkdir -p "$NEW_PROJECT"

# Copy logo (dùng lại)
cp /Users/tunbui/Downloads/SUBI-STORE/Ladipage/subisneaker/logo.png "$NEW_PROJECT/"

# Copy video (nếu có)
cp /path/to/video.mp4 "$NEW_PROJECT/video.mp4"

# Copy .claude/launch.json để chạy preview ngay
mkdir -p "$NEW_PROJECT/.claude"
cp /Users/tunbui/Downloads/SUBI-STORE/Ladipage/subisneaker/.claude/launch.json "$NEW_PROJECT/.claude/"
```

---

## Bước 4 — Sinh file index.html

Đọc toàn bộ file template:
```
/Users/tunbui/Downloads/SUBI-STORE/Ladipage/subisneaker/index.html
```

Sau đó thay thế các vị trí sau (dùng str.replace hoặc regex, **không** dùng sed):

### 4a. Các chuỗi thay thế cố định

| Tìm | Thay bằng |
|---|---|
| `Ultra Run Pro 2026` | `{TEN_SAN_PHAM}` |
| `ULTRA RUN PRO 2026` | `{TEN_SAN_PHAM_UPPER}` |
| `250.000đ` (giá bán) | `{GIA_BAN}` |
| `360.000đ` (giá gốc) | `{GIA_GOC}` |
| `-30%` | `{PHAN_TRAM_GIAM}` |
| `san_pham:'Ultra Run Pro 2026 - 250.000đ'` | `san_pham:'{TEN_SAN_PHAM} - {GIA_BAN}'` |

### 4b. IMAGES array

Thay block cũ:
```js
var IMAGES = [
  "img1.jpg",
  ...
  "img9.jpg"
];
```
Bằng array mới với đúng số ảnh đã xử lý ở Bước 2.

### 4c. Features section

Thay tiêu đề và 4 feat-card theo tính năng của sản phẩm mới:
```html
<div class="section-title">Tại sao chọn {TEN_SAN_PHAM}?</div>
<div class="feat-card">
  <div class="feat-icon">{EMOJI}</div>
  <div class="feat-title">{TIÊU ĐỀ}</div>
  <div class="feat-desc">{MÔ TẢ NGẮN}</div>
</div>
```

### 4d. Info box

Thay nội dung mô tả vật liệu/thông số sản phẩm trong `.info-box`:
```html
<p>+ Chất liệu: {UPPER}<br>
   + Đế: {DE}<br>
   + Full size: {SIZE_RANGE}<br>
   + Phù hợp: {MUC_DICH_SU_DUNG}</p>
```

### 4e. Google Apps Script URL

Giữ nguyên `SCRIPT_URL` của template gốc — không thay đổi, đơn hàng vẫn đổ về
cùng một Google Sheet. Nếu người dùng muốn tách riêng Sheet cho sản phẩm mới,
hỏi họ URL Apps Script mới.

### 4f. Sale banner

Cập nhật nếu % giảm khác template gốc:
```html
<div class="sale-banner">ƯU ĐÃI LÊN TỚI {XX}% — CHỈ HÔM NAY</div>
```

---

## Bước 5 — Kiểm tra và chạy thử

Sau khi sinh xong `index.html`:

1. Cập nhật `.claude/launch.json` trong thư mục mới — đổi `"name"` thành tên
   dự án mới (port giữ nguyên `8765` trừ khi đang chạy song song):
   ```json
   { "version":"0.0.1", "configurations":[{
     "name": "<tên-dự-án>",
     "runtimeExecutable": "/usr/local/bin/python3",
     "runtimeArgs": ["-m","http.server","8765"],
     "port": 8765
   }]}
   ```

2. Dùng `preview_start` với name mới để khởi chạy server.

3. Chụp screenshot kiểm tra: header, carousel, price block, video, footer.

4. Báo người dùng danh sách file cần deploy lên Netlify:
   - `index.html`
   - `logo.png`
   - `img1.jpg` … `imgN.jpg`
   - `video.mp4` (nếu có)

---

## Checklist trước khi giao

- [ ] Tên sản phẩm đúng ở: `<title>`, `<h1>`, `alt` ảnh, `san_pham` param
- [ ] Giá / % giảm đúng
- [ ] IMAGES array khớp số ảnh thực tế trong thư mục
- [ ] Video poster = `img1.jpg`
- [ ] Features 2×2 đã cập nhật
- [ ] Info box đã cập nhật
- [ ] `SCRIPT_URL` giữ nguyên (hoặc đã thay theo yêu cầu)
- [ ] Nút "Nhắn tin" vẫn trỏ đúng Facebook Page

---

## Lưu ý quan trọng

- **Font**: Không dùng `Bebas Neue` cho text có dấu tiếng Việt. Dùng `Barlow`
  weight 700–800 + `text-transform: uppercase`.
- **Ảnh**: Luôn crop vuông 1:1 trước khi resize, tránh letterbox trắng.
- **Lazy load**: Chỉ `img1.jpg` có `src`, các ảnh còn lại dùng `data-src` —
  đừng thay đổi pattern này.
- **Màu chủ đạo**: `#f07820` (cam). Không thay đổi trừ khi được yêu cầu.
