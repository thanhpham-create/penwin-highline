# HighLine landing page — deploy guide

Static HTML landing page cho **penwin.vn (root)** + optionally `highline.penwin.vn` (subdomain). Single file, no build step.

## Deploy target options

**Recommended (v2)**: Deploy to **penwin.vn root** — anh đã quyết định pivot toàn bộ penwin.vn sang football content.

3 deploy paths:
- **Path 1 — Cloudflare Pages on penwin.vn root** (recommended, see Section A below)
- **Path 2 — Keep subdomain, redirect root → subdomain** (current state, lower-effort but split brand)
- **Path 3 — Same content both root + subdomain** (max coverage)

## Cấu trúc

```
web/
├── index.html              # Landing page chính
├── assets/                 # Brand assets (PNG)
│   ├── favicon_64.png
│   ├── logo_wordmark_dark.png
│   └── ...
└── README.md
```

## Local preview

```bash
cd web/
python -m http.server 8000
# Open http://localhost:8000
```

## Deploy — 3 options

## Section A — Deploy to penwin.vn ROOT (Cloudflare Pages) — RECOMMENDED v2

**Strategy**: 2 separate Cloudflare Pages projects.
- **Existing `highline-web`** (GitHub: `thanhpham-create/highline-web`) → keeps serving `highline.penwin.vn` subdomain với content cũ (v1). **KHÔNG push updates ở đây.**
- **NEW `penwin-highline`** (GitHub repo mới) → serves `penwin.vn` root với content v2 ("Data analytics for everyone").

→ Subdomain stays as-is. Root gets new content. Anh có thể diverge 2 brands theo thời gian.

### Bước 1 — KHÔNG động vào project cũ highline-web

Anh **không commit + push** web/ folder updates vào `thanhpham-create/highline-web` origin. Subdomain `highline.penwin.vn` continues serving last-deployed v1 content.

### Bước 2 — Setup NEW repo `penwin-highline`

```bash
cd /Users/thanhpham/Documents/Claude/Projects/football/highline/web

# Add NEW remote (không thay origin hiện tại)
git remote add penwin git@github.com:thanhpham-create/penwin-highline.git

# Commit current changes (v2 content)
git add .
git commit -m "HighLine v2 — Data analytics for everyone, penwin.vn root launch"

# Push lên repo MỚI (penwin), KHÔNG push origin (highline-web)
git push -u penwin main
```

**Verify**:
```bash
git remote -v
# Expected output:
# origin   git@github.com:thanhpham-create/highline-web.git (fetch)
# origin   git@github.com:thanhpham-create/highline-web.git (push)
# penwin   git@github.com:thanhpham-create/penwin-highline.git (fetch)
# penwin   git@github.com:thanhpham-create/penwin-highline.git (push)
```

### Bước 3 — Cloudflare Pages tạo project mới `penwin-highline`

### Bước 1 — Backup project cũ trên penwin.vn

**TRƯỚC KHI XOÁ**: Anh check Cloudflare Pages hiện tại cho penwin.vn:
```
Dashboard cloudflare.com → Account → Pages → tìm project hiện đang serve penwin.vn
```
- Note tên Pages project + GitHub repo connected
- Download backup project cũ (Settings → Source → View repo → clone về local làm backup)
- KHÔNG xoá Cloudflare Pages project cũ ngay — chỉ disconnect custom domain

### Bước 2 — Push web/ folder vào GitHub repo riêng

```bash
cd /Users/thanhpham/Documents/Claude/Projects/football/highline/web

# Khởi tạo Git nếu chưa
git init
git add .
git commit -m "HighLine penwin.vn relaunch — football content v2"

# Tạo repo GitHub mới riêng cho landing page (tách khỏi main highline repo để Cloudflare dễ deploy)
# Tạo trên GitHub UI: thanhpham-create/penwin-highline (hoặc tên anh muốn)
git remote add origin git@github.com:thanhpham-create/penwin-highline.git
git branch -M main
git push -u origin main
```

### Bước 3 — Cloudflare Pages tạo project mới

1. Dashboard cloudflare.com → Pages → Create application → Connect to Git
2. Select `thanhpham-create/penwin-highline`
3. Build settings:
   - Framework preset: **None**
   - Build command: **(để trống)**
   - Build output directory: **/** (root)
4. Click Deploy → đợi 1-2 phút
5. Verify default URL: `penwin-highline.pages.dev`

### Bước 4 — Add custom domain `penwin.vn` vào project mới

1. Dashboard → Pages → `penwin-highline` project → Custom domains
2. Set up custom domain → Enter: **`penwin.vn`** (apex/root)
3. Cloudflare auto-create A record với proxied flag (orange cloud)
4. **Repeat for `www.penwin.vn`** — Cloudflare tự CNAME → apex
5. Wait 1-5 phút for SSL provision

**KHÔNG add `highline.penwin.vn` ở đây** — subdomain đó vẫn được serve bởi project cũ `highline-web`.

### Bước 5 — Verify cả 2 deployments work parallel

```bash
# Root (NEW v2 content)
curl -sI https://penwin.vn | grep -E "HTTP|content-type"
# Expected: HTTP/2 200

# www redirect to apex
curl -sI https://www.penwin.vn | grep -E "HTTP|location"
# Expected: HTTP/2 301 → https://penwin.vn

# Subdomain (OLD v1 content, KHÔNG được đổi)
curl -sI https://highline.penwin.vn | grep -E "HTTP|content-type"
# Expected: HTTP/2 200, content vẫn là v1
```

Mở browser test:
- `https://penwin.vn` → **NEW v2**: "Data analytics for everyone"
- `https://www.penwin.vn` → redirect to penwin.vn
- `https://highline.penwin.vn` → **OLD v1**: "Data analytics for the leagues everyone else ignores"

### Bước 6 — Future maintenance

Khi anh muốn update penwin.vn root:
```bash
cd web/
# Edit index.html
git add . && git commit -m "Update penwin.vn"
git push penwin main          # ← chỉ push lên penwin, KHÔNG origin
```

Khi anh muốn update highline.penwin.vn subdomain:
```bash
git push origin main          # ← push lên origin (highline-web)
```

→ 2 remotes độc lập. Anh kiểm soát rõ ràng URL nào update.

---

## Section B — Legacy: Subdomain only (highline.penwin.vn)

Anh đã có Cloudflare account quản lý penwin.vn DNS.

**Bước 1 — Push web/ vào GitHub repo mới:**

```bash
cd /Users/thanhpham/Documents/Claude/Projects/football/highline/web

git init
git add .
git commit -m "Initial HighLine landing page"

# Tạo repo trên GitHub: thanhpham-create/highline-web
gh repo create thanhpham-create/highline-web --public --source=. --push

# Hoặc thủ công:
# - Vào github.com/new → tạo repo "highline-web"
# - git remote add origin git@github.com:thanhpham-create/highline-web.git
# - git push -u origin main
```

**Bước 2 — Connect Cloudflare Pages:**

1. Vào dashboard.cloudflare.com → Pages → Create application → Connect to Git
2. Select `thanhpham-create/highline-web`
3. Build settings:
   - Framework preset: None
   - Build command: (để trống)
   - Build output: `/`
4. Deploy → đợi 1-2 phút

**Bước 3 — Custom domain:**

1. Trong Cloudflare Pages project → Custom domains → Set up custom domain
2. Enter: `highline.penwin.vn`
3. Cloudflare tự tạo CNAME record `highline.penwin.vn → <project>.pages.dev`
4. Wait 1-5 phút for SSL provision

**Bước 4 — Verify:**

```bash
curl -I https://highline.penwin.vn
# Expected: HTTP/2 200
```

### Option B — GitHub Pages (free, anh đã setup sẵn cho penwin.vn)

Tận dụng `thanhpham-create.github.io` setup hiện có:

**Bước 1 — Tạo branch riêng:**

```bash
cd thanhpham-create.github.io  # repo của anh
git checkout -b highline
# Copy web/ files vào subdirectory
mkdir -p highline
cp -r /Users/thanhpham/Documents/Claude/Projects/football/highline/web/* highline/
git add . && git commit -m "Add HighLine landing"
git push origin highline
```

Sau đó URL = `https://thanhpham-create.github.io/highline/` (subdirectory, không custom domain).

**Bước 2 — Custom domain CNAME:**

Tạo file `highline/CNAME` chứa `highline.penwin.vn`.

Cloudflare DNS → Add CNAME `highline → thanhpham-create.github.io`.

### Option C — Cloudflare Page Rule (chỉ redirect, không landing page)

Skip web/ entirely. Trong Cloudflare:
- DNS → A record `highline → 192.0.2.1` (dummy, proxied)
- Page Rules → URL: `highline.penwin.vn/*` → Forwarding URL 301 → `https://highlineasia.substack.com/$1`

**Trade-off:** Không có landing page, anh mất chance build brand. Khuyến nghị Option A.

## Update content sau này

Sau khi anh có 5-10 bài Substack, có thể thêm "Latest posts" section vào index.html bằng cách:

1. Substack có RSS feed: `https://highlineasia.substack.com/feed`
2. JavaScript fetch RSS → parse → render 3-5 latest posts
3. Hoặc static — copy paste 3 latest links thủ công weekly

Để turn sau khi cần.
