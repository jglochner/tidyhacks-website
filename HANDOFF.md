# TidyHacks Website — Handoff Notes
*Last updated: June 2026 — pick up from here*

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

Or use the Claude Code Launch preview (`.claude/launch.json` is configured on :8778).

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
3. Scrapes the main product image from the Amazon page (browser headers required — bare requests get blocked)
4. **Downloads the image** to `tidyHacksWebsite/images/products/<ASIN>.jpg` (local, no Amazon CDN dependency after first run)
5. Rewrites `tidyHacksWebsite/products.json` with local image paths

**Important notes on image scraping:**
- Amazon rate-limits rapid batch requests. The script has 5s gaps + a 20s-cooldown second pass to handle this.
- Already-downloaded images are skipped on subsequent runs (cached by ASIN filename).
- If a product gets no image after retries, it falls back to the remote CDN URL gracefully.
- The script is at `../tidyHacks/scripts/build_site.py`.

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
| 6 | Handheld Vacuum Sealer | prompt-ready | ❌ missing | ❌ missing | ⏳ needs links first |
| 7 | Cordless Mini Handheld Vacuum | prompt-ready | ❌ missing | ❌ missing | ⏳ needs links first |

**Products #6 & #7 are excluded from the live site** until affiliate links are added.
To add them: edit `../tidyHacks/data/products.json` (or use the Studio at
`python3 ../tidyHacks/scripts/server.py` → http://localhost:8765), then rerun
`build_site.py`.

---

## What the website does

- Loads `products.json` via `fetch()` on page load
- Renders a **responsive 2-column card grid** (1 column on narrow phones)
- Each card: product image (square thumbnail), name, niche tag, "View on Amazon →" button
- **Live search** filters by product name and niche as you type
- Clear (✕) button resets search
- Social icons: Instagram `instagram.com/tidyhacks_`, TikTok `tiktok.com/@tidyhacks_`
- Amazon Associate disclosure in footer
- Links: **US only** for v1 (`links.us` field). UK links are stored in the data but not surfaced. Deferred to **genius links** (geo-routing US→a.co, UK→amzn.eu) in a future pass.

---

## What's NOT done yet (immediate next steps)

### 1. Deploy to Netlify + point `tidyhacks.co` DNS

The site is **fully built and tested locally** but not yet live. Jonas is logged into
both Netlify and Porkbun in Chrome — drive the browser to deploy:

- Drag-and-drop the `tidyHacksWebsite/` folder into Netlify (or connect the GitHub repo)
- In Porkbun DNS, point `tidyhacks.co` CNAME/A record at the Netlify domain
- Netlify free tier is sufficient — 5 product images (~200KB total per page load) will
  never come close to the 100GB/month free bandwidth limit

### 2. Add affiliate links for products #6 & #7

Jonas needs to find and add US/UK Amazon affiliate links for:
- Handheld Vacuum Sealer
- Cordless Mini Handheld Vacuum

Add them via the Studio (`python3 ../tidyHacks/scripts/server.py`), then run
`build_site.py` to scrape images and update the site.

### 3. Genius links (geo-routing) — future

Currently US links only. When Jonas sets up Geniuslink/Geni.us:
- Each product gets one geo-routing URL instead of separate US/UK
- `build_site.py` will need a `genius_link` field in the schema
- The button label changes from "View on Amazon →" to something region-neutral

### 4. Amazon Associates application

The site is built specifically to support the application. Requirements:
- ✅ Working website you own (`tidyhacks.co`)
- ✅ Original content (your product selections + captions from the engine)
- ✅ Affiliate-style content clearly visible
- ⏳ Site must be **live on the domain** before applying
- ⏳ After approval, make **3 qualifying sales within 180 days** to keep the account

---

## Branding

- **Handle:** `@tidyhacks_` on Instagram and TikTok
- **Avatar:** purple "TH" monogram logo (`images/avatar.png`) — downloaded from Beacons CDN
- **Colour:** `#8b6ff0` (brand purple), matches existing Beacons page
- **Current live Beacons page:** https://beacons.ai/tidyhacks (the old link-in-bio being replaced)

---

## The engine (sibling project)

The `../tidyHacks/` project is the content production engine. It's a separate concern
but feeds this website. Key things to know:

- `data/products.json` — source of truth, edit here or via the Studio
- `scripts/server.py` — local Studio UI (browse/edit products, AI-generate prompts)
- `scripts/build_site.py` — the script that generates this website's data
- `scripts/build.py` — generates CSV + markdown for Google Sheets and Seedance
- The engine has its own `README.md` with full documentation

---

## Tech decisions made (and why)

| Decision | What | Why |
|----------|------|-----|
| No framework | Vanilla HTML/CSS/JS | Matches engine's stdlib-only ethos; zero build step; deploys as flat files |
| Local images | Downloaded to `images/products/` | No Amazon CDN dependency; self-contained Netlify deploy; free |
| US links only | Single "View on Amazon" button | Simplicity for v1; UK retained in data for genius links later |
| Static site | No server, no DB | Free hosting; nothing to maintain; fast everywhere |
| `products.json` copy | Website has its own generated data file | Decouples deploy from the engine; engine stays the source of truth |
