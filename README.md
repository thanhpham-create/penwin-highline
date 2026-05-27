# HighLine landing page — deploy guide

Static HTML landing page cho `highline.penwin.vn`. Single file, no build step.

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

### Option A — Cloudflare Pages (RECOMMEND, free)

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
- Page Rules → URL: `highline.penwin.vn/*` → Forwarding URL 301 → `https://highlinefc.substack.com/$1`

**Trade-off:** Không có landing page, anh mất chance build brand. Khuyến nghị Option A.

## Update content sau này

Sau khi anh có 5-10 bài Substack, có thể thêm "Latest posts" section vào index.html bằng cách:

1. Substack có RSS feed: `https://highlinefc.substack.com/feed`
2. JavaScript fetch RSS → parse → render 3-5 latest posts
3. Hoặc static — copy paste 3 latest links thủ công weekly

Để turn sau khi cần.
