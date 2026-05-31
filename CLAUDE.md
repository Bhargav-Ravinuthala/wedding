# CLAUDE.md — Bhargav & Bhargavi Wedding Invite

Context for any AI/human working on this project. Read this first.

## What this is

A **static, single-page wedding invitation** website, hosted on GitHub Pages.

- **Live URL:** https://bhargav-ravinuthala.github.io/wedding/
- **Repo:** https://github.com/Bhargav-Ravinuthala/wedding
- **Local dir:** `~/missingpiece-scrape/`

### Wedding details (used throughout the page)

| | |
|---|---|
| **Groom** | Bhargav, son of Sri Ravinuthala Nageswara Rao & Smt. Janaki (Nellore) |
| **Bride** | Bhargavi, daughter of Sri Rayavarapu Ravikumar & Smt. Pushpalatha (Payakaraopet, Anakapalle Dist.) |
| **Wedding date** | Sunday, 23 August 2026 |
| **Wedding time** | 11:21 AM (Mula Nakshatra, Tula Lagna) |
| **Venue** | Sri Lakshmi Grand Convention, Bypass Road, Rayavaravupeta, Anakapalle District |
| **Source card** | `~/Downloads/BHARGAV WEDS BHARGAVI.pdf` (Telugu invitation, translated to English) |

## How this project came to exist (critical context)

This is **not normal code**. It was created by:

1. Scraping the Framer-built template page at `https://www.missingpieceinvites.com/demos/meenaya`
2. Downloading all assets (images, fonts, JS bundles, CSS) locally
3. Rewriting all `framerusercontent.com` URLs to relative paths so it works offline / on any host
4. Patching the JS bundles + HTML to swap the template's names/dates/photos for Bhargav & Bhargavi's
5. Stripping all trackers, analytics, payment CTAs, and "Made in Framer" branding
6. Adding original CSS for motion graphics, dark text, gold-gradient name highlights, location pin button

**Implication:** the page is a Framer SPA that hydrates from compiled JS bundles. The HTML is just the SSR shell. **Most editable content lives in the JS bundle, not the HTML.** See "Pitfalls" below.

## File structure

```
~/missingpiece-scrape/
├── CLAUDE.md                  ← this file
├── index.html                 ← main page, ~411 KB. Has all SSR markup + injected <style id="strip-branding"> + <script id="force-music-autoplay"> + #venue-map-btn anchor
├── page.html                  ← original unmodified scrape (kept for reference; not deployed)
├── images/                    ← 30+ PNG/SVG files (couple photos, temple backdrops, decorations, icons)
├── fonts/                     ← 44 woff2 files (Framer template fonts)
├── gfonts/                    ← 44 woff2 files (Google Fonts, downloaded locally to avoid 3rd-party fetch)
├── js/                        ← Framer ESM bundles (.mjs). Main bundle: -XZpmxfF1qm1oVTXMUajuqutYRzxhY2USEO22jmpgfI.vzzpaPs1.mjs
├── media/                     ← q8x7MYVF61gOLfkcxdG7dlXjxw.mp3 (background music, ~1.4 MB)
├── .gitignore                 ← ignores .DS_Store, *.bak
└── (junk artifacts kept for reference: asset_urls.txt, gfont_urls.txt, etc.)
```

### Couple photo files

The page renders 6 carousel "Wedding shoot" slots backed by 4 unique image files. All are currently the user's actual photos (replaced from template placeholders):

| File | Used as | Source |
|---|---|---|
| `images/HbWyQm5QqRuXQY7LxMf5kx10akU.png` | Wedding shoot 1 + "Meet the bride and groom" | `photo1.png` (Bhargav & Bhargavi standing, flower backdrop) |
| `images/gXWpfu1hEOSPMvZeJtINB07pfJU.png` | Wedding shoots 2 & 6 | `photo2.png` (couple looking at each other) |
| `images/VgBpHrwsihOjP7HT4E3SSrxEs0.png` | Wedding shoot 3 | `photo3.png` (bride seated, parents blessing) |
| `images/4moRDGJdHuGozf1nEo8Y2t4pdw.png` | Wedding shoots 4 & 5 | `photo1.png` (rotated back) |

If user gives more photos: copy/convert to PNG via `sips`, then overwrite one or more of these four files.

## Critical pitfalls (where I burned hours)

### 1. The HTML is a lie. The JS is the source of truth.

Framer's React hydration runs after page load and **overwrites the DOM** with content rendered from the JS bundle. Editing HTML alone is invisible.

For **text replacements** (names, dates, venue, button labels, etc.), you must edit BOTH:
- `index.html` (so SSR/first paint matches)
- `js/-XZpmxfF1qm1oVTXMUajuqutYRzxhY2USEO22jmpgfI.vzzpaPs1.mjs` (so hydrated DOM matches)

For **image URLs**, the JS bundle hardcoded `framerusercontent.com/images/*.png` URLs. These were rewritten to `images/*.png` (relative). If you swap a couple photo, the JS already points at the local path — just overwrite the file on disk.

### 2. Browser caching is brutal

When changes don't appear, it's almost always cache. Two reliable bypasses:
- Bump the local dev server to a **new port** (browser has no cache for that origin yet)
- Hard-refresh: `Cmd+Shift+R` on Mac

### 3. CSS `!important` is required to beat Framer Motion

Framer Motion sets inline `style="transform: ..."` on wrappers continuously. Any CSS animation targeting `transform` on those elements needs `!important` to override. Prefer targeting **inner elements** (e.g., the `<h3>` inside `.framer-t7d3wo`) rather than the wrapper itself — those are untouched by Framer Motion.

### 4. Audio autoplay is blocked by browsers

Modern browsers (Chrome, Safari, Firefox, all mobile) **refuse to play audio with sound** until the user interacts with the page. No JS trick bypasses this.

The current `<script id="force-music-autoplay">` listens for ANY interaction (click, scroll, mousemove, touchmove, pointermove, keydown, wheel) and starts the music on the first one. That's the best possible UX — the user usually moves the mouse or scrolls within the first second.

### 5. The site is built with ES Modules

`<script type="module">` + `<link rel="modulepreload">` require an HTTP origin. `file://` won't work. Always use a local server for testing (`python3 -m http.server PORT`).

## Common edit recipes

### Replace a couple photo
```bash
# Convert JPEG → PNG (preserves quality)
sips -s format png ~/Downloads/new.jpeg --out images/new.png

# Overwrite one of the 4 wedding-shoot slots:
cp images/new.png images/HbWyQm5QqRuXQY7LxMf5kx10akU.png   # Shoot 1
# (or gXWpfu1hEOSPMvZeJtINB07pfJU.png for 2&6, VgBpHrwsihOjP7HT4E3SSrxEs0.png for 3, 4moRDGJdHuGozf1nEo8Y2t4pdw.png for 4&5)
```

### Replace text content (e.g., change venue or date)
```python
# In Python, edit BOTH HTML and the main JS bundle:
import re
for fpath in ['index.html', 'js/-XZpmxfF1qm1oVTXMUajuqutYRzxhY2USEO22jmpgfI.vzzpaPs1.mjs']:
    with open(fpath) as f: c = f.read()
    c = c.replace('OLD TEXT', 'NEW TEXT')
    with open(fpath, 'w') as f: f.write(c)
```

### Hide a UI element you can find in the rendered page
1. Inspect the element in DevTools → note a stable class (e.g., `framer-abc123` or `[data-framer-name="..."]`)
2. Add a `display: none !important` rule to the `<style id="strip-branding">` block in `index.html`

### Change name/heading animations
The motion CSS lives in `<style id="strip-branding">` in `index.html`. Look for `@keyframes` rules (`heroGlowPulse`, `heroNameFloat`, `wedsBreath`, `goldShimmer`, `pinPulse`). Adjust values for intensity. Selectors target:
- `.framer-t7d3wo h3` → "Bhargav" (hero, left)
- `.framer-11b7scn h3` → "Bhargavi" (hero, right)
- `.framer-1p5h4t7 h2` → "WEDS" (hero, center)
- `.framer-11ag0sw p`, `.framer-13k4cyv p` → mid-page stacked names with gold gradient
- `.framer-rtadh1` → mid-page "&" ampersand

### Move the location-pin button
Edit `#venue-map-btn` rules in the style block + the `<script id="position-venue-pin">` block before `</body>`. The script finds the audio button and places the pin directly below it. Change the `gap` value (currently `12`) for spacing.

## Deploy workflow

```bash
cd ~/missingpiece-scrape

# After making changes:
git add .
git -c user.name='Bhargav Ravinuthala' -c user.email='rnsbhargav@gmail.com' \
  commit -m "describe change"
git push
```

GitHub Pages auto-rebuilds in ~30 seconds. SSH key for `Bhargav-Ravinuthala` is already in keychain (`~/.ssh/id_ed25519`).

## Local dev server

```bash
cd ~/missingpiece-scrape
python3 -m http.server 9000     # or any port; bump it when browser cache misbehaves
# open http://localhost:9000/index.html
```

## Things that were intentionally removed

- ❌ "Made in Framer" / "Published" comments
- ❌ Google Analytics (`G-VCMPQ02N6E`), Contentsquare, Pixelflow, Framer events
- ❌ `Missing Piece` logo, "© Missing Piece 2025" footer, promo CTAs linking to missingpieceinvites.com
- ❌ JSON-LD schema (named the source site)
- ❌ Razorpay payment button (`rzp.io/rzp/meenaaya`) — `INR 3999` / `Buy Now`
- ❌ "Find more templates on missingpieceinvites.com" / `missingpiecedesign.com` strings in JS
- ❌ Template's 5 non-wedding event cards (Mehendi, Haldi, Sangeet, Engagement, Reception)
- ❌ RSVP section ("Please rsvp" + WhatsApp button) — hidden via CSS in latest commit
- ❌ `og:url` pointing to template's URL (replaced with deployed URL)
- ❌ All `fonts.gstatic.com` Google Fonts URLs (downloaded locally to `gfonts/`)

## Things to consider next

- **WhatsApp share preview** uses `og:image` = `https://bhargav-ravinuthala.github.io/wedding/images/photo1.png`. If `photo1.png` is replaced, the preview updates. WhatsApp aggressively caches — use https://developers.facebook.com/tools/debug/ to force refresh.
- **Mobile load time:** 42 MB total. The two biggest are `images/uQ5GdDW5jaPT8LYNBYabSn2bj0.png` (6.9 MB, hero temple) and `images/cyIPT3IPm257uiP1lpxhaxTt8E.png` (4.5 MB). Converting to WebP would cut load by ~75%.
- **Privacy:** GitHub Pages is public. If you don't want guests' future Google searches to surface this URL, add `<meta name="robots" content="noindex,nofollow">` to `<head>`.
- **Custom domain:** Point a domain's CNAME → `bhargav-ravinuthala.github.io`, then add a `CNAME` file in the repo root with the domain name.

## Template licensing

This is a scraped copy of the **Meenaya** template (₹3999) from Missing Piece Invites. For personal use, the practical risk is near-zero. If anyone redistributes the template files as their own work, that's infringement. For a fully-legit version, buy the template at missingpieceinvites.com and edit it inside Framer.
