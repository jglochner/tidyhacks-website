# TidyHacks Website — Handoff Notes
*Last updated: June 2026 — site is LIVE at https://tidyhacks.co*

---

## What this project is

A custom **link-in-bio / affiliate product showcase** for the TidyHacks brand
(`tidyhacks.co`). It replaces Beacons.AI (which has no search and no product grid)
and serves as the "working website" needed to apply for **Amazon Associates** approval.

It was designed to look like `c8ke.com/samfindz` (see `samInspo.PNG` in this folder).

---

## The two repos that matter

```
/Users/jonaslochner/ClaudeCode/
├── tidyHacks/          ← THE ENGINE  (source of truth, Studio, AI generator)
└── tidyHacksWebsite/   ← THIS REPO   (the public website you're looking at)
```

**Never edit `tidyHacksWebsite/products.json` or `images/` by hand** — they are
generated output. Always edit via the engine and rebuild.

---

## File structure

```
tidyHacksWebsite/
├── index.html               ← The entire website (vanilla HTML/CSS/JS, no framework)
├── products.json            ← Generated — do NOT edit manually
├── images/
│   ├── avatar.png           ← TidyHacks logo (downloaded from Beacons CDN)
│   └── products/
│       ├── B00E4NMQ6U.jpg   ← Foaming Drain Cleaner
│       ├── B0D8PSDMYH.jpg   ← Silicone Ice Cube Tray
│       ├── B0BHSXFTGH.jpg   ← Vegetable Chopper
│       ├── B0D54PM77N.jpg   ← Olive Oil Sprayer/Mister
│       └── B0F886H7GZ.jpg   ← Diatomite Bath Mat
├── netlify.toml             ← Netlify config: PostHog reverse-proxy rewrites
├── samInspo.PNG             ← Reference design (c8ke.com/samfindz screenshot)
└── HANDOFF.md               ← This file
```

---

## How to run locally

```bash
# From the tidyHacksWebsite folder:
python3 -m http.server 8777

# Then open: http://127.0.0.1:8777/
```

---

## Live site

- **Production URL:** https://tidyhacks.co
- **Netlify URL:** https://tidyhacks-website.netlify.app
- **GitHub repo:** https://github.com/jglochner/tidyhacks-website
- **Hosting:** Netlify free tier (connected to GitHub — every `git push` to `main` auto-deploys)
- **DNS:** Porkbun — ALIAS `tidyhacks.co` → `apex-loadbalancer.netlify.com`, CNAME `www` → `tidyhacks-website.netlify.app`
- **HTTPS:** Provisioned automatically by Netlify (Let's Encrypt)

---

## How to rebuild the site data (after adding/editing products)

The engine lives in `../tidyHacks/`. After editing products there:

```bash
cd /Users/jonaslochner/ClaudeCode/tidyHacks
python3 scripts/build_site.py
```

This script:
1. Reads `../tidyHacks/data/products.json` (the single source of truth)
2. Follows each affiliate short link → resolves the Amazon ASIN
3. Scrapes the main product image from the Amazon page
4. Downloads the image to `tidyHacksWebsite/images/products/<ASIN>.jpg`
5. Rewrites `tidyHacksWebsite/products.json` with local image paths

After running, commit and push:
```bash
git add -A && git commit -m "Update products" && git push
```
Netlify deploys automatically within ~30 seconds.

---

## Product data

**Source of truth:** `../tidyHacks/data/products.json`

Currently 7 products:

| # | Product | Status | US Link | UK Link | Image |
|---|---------|--------|---------|---------|-------|
| 1 | Foaming Drain Cleaner | posted | ✅ | ✅ | ✅ downloaded |
| 2 | Silicone Ice Cube Tray | posted | ✅ | ✅ | ✅ downloaded |
| 3 | Vegetable Chopper (6-in-1) | posted | ✅ | ✅ | ✅ downloaded |
| 4 | Olive Oil Sprayer/Mister | posted | ✅ | ✅ | ✅ downloaded |
| 5 | Diatomite Bath Mat | posted | ✅ | ✅ | ✅ downloaded |
| 6 | Handheld Vacuum Sealer | posted | ✅ | ✅ | ✅ downloaded |
| 7 | Cordless Mini Handheld Vacuum | prompt-ready | ❌ missing | ❌ missing | ⏳ needs links first |

**6 products are now live on the site.** Product #7 is excluded until affiliate links are added.

---

## What the website does

- Loads `products.json` via `fetch()` on page load
- Renders a **responsive 2-column card grid** (1 column on narrow phones)
- Each card: product image (square thumbnail, **clickable**), name, niche tag, "View on Amazon →" button
- Both the image AND the button link to the Amazon affiliate URL
- **Live search** filters by product name and niche as you type
- Clear (✕) button resets search
- Social icons: Instagram `instagram.com/tidyhacks_`, TikTok `tiktok.com/@tidyhacks_`
- Amazon Associate disclosure in footer
- Links: **US only** for v1. UK links are stored in data but not surfaced (deferred to genius links)

---

## Analytics — PostHog

- **Platform:** PostHog EU Cloud (GDPR-friendly)
- **Account:** `jonaslochner@hotmail.com`
- **Dashboard:** https://eu.posthog.com/project/193147
- **Project name:** TidyHacks.Co (was "Default project")
- **Organisation:** Tidy Hacks
- **Project token:** `phc_wkWTdKNX3a9aJbsvNacDvH87dTotVpnsdGGeePWp3W7F`
- **Features enabled:** Pageview tracking, session recordings (all sessions), console log capture
- **Free tier limits:** 1M events/month, 5K session recordings/month — very unlikely to hit these

**Custom `amazon_click` event** fires on every "View on Amazon" button click with these properties:
```
product_name, niche, asin, click_target
```
Filter or break down by these in the PostHog dashboard to see which products drive the most clicks.

**Ad-blocker reverse proxy:** PostHog requests are routed through `tidyhacks.co/ingest`
rather than directly to PostHog's servers, so ad blockers don't silently drop events. This
is configured as a rewrite rule in `netlify.toml` (in the repo root):
```toml
[[redirects]]
  from = "/ingest/static/*"
  to = "https://eu-assets.i.posthog.com/static/:splat"
  status = 200

[[redirects]]
  from = "/ingest/*"
  to = "https://eu.i.posthog.com/:splat"
  status = 200
```
Do not remove these — without them, a significant portion of analytics events will be lost.

The PostHog snippet is in `index.html` just after the `<title>` tag. To verify tracking is working, visit `tidyhacks.co` and check the Live Events feed in PostHog.

---

## What's NOT done yet (immediate next steps)

### 1. ⚡ Amazon Associates application — DO THIS FIRST

The site is live at `tidyhacks.co` with HTTPS. You are now ready to apply.

- Go to: https://affiliate-program.amazon.co.uk (UK) or https://affiliate-program.amazon.com (US)
- Requirements met: ✅ live website, ✅ original content, ✅ affiliate-style layout
- After approval: make **3 qualifying sales within 180 days** to keep the account active
- Once approved: replace the existing short links in `../tidyHacks/data/products.json` with your official Associates tracking links and rebuild

### 2. Add affiliate links for product #7

Jonas needs to find US/UK Amazon affiliate links for:
- Cordless Mini Handheld Vacuum (#7 — the only one still missing)

Add them via the Studio (`python3 ../tidyHacks/scripts/server.py` → http://localhost:8765), then run `build_site.py`, commit, and push.

### 3. Genius links (geo-routing) — future

Currently US links only. When Jonas sets up Geniuslink/Geni.us:
- Each product gets one geo-routing URL instead of separate US/UK
- `build_site.py` will need a `genius_link` field in the schema
- The button label changes from "View on Amazon →" to something region-neutral

---

## Branding

- **Handle:** `@tidyhacks_` on Instagram and TikTok
- **Avatar:** purple "TH" monogram logo (`images/avatar.png`)
- **Colour:** `#8b6ff0` (brand purple)
- **Current live Beacons page:** https://beacons.ai/tidyhacks (the old link-in-bio being replaced)

---

## The engine (sibling project)

The `../tidyHacks/` project is the content production engine:

- `data/products.json` — source of truth, edit here or via the Studio
- `scripts/server.py` — local Studio UI (browse/edit products, AI-generate prompts)
- `scripts/build_site.py` — generates this website's data + downloads images
- `scripts/build.py` — generates CSV + markdown for Google Sheets and Seedance

---

## Tech decisions made (and why)

| Decision | What | Why |
|----------|------|-----|
| No framework | Vanilla HTML/CSS/JS | Zero build step; deploys as flat files |
| Local images | Downloaded to `images/products/` | No Amazon CDN dependency; self-contained |
| US links only | Single "View on Amazon" button | Simplicity for v1; UK retained for genius links later |
| Static site | No server, no DB | Free hosting; nothing to maintain; fast everywhere |
| Netlify + GitHub | CI/CD deploy | Every `git push` auto-deploys; no manual steps |
| PostHog EU | Analytics | Free tier generous; EU cloud = GDPR-compliant; session replay included |
| Clickable images | `<a>` wraps `.thumb` div | Larger click target; better UX; same affiliate link as button |
