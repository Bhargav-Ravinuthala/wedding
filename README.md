# missingpieceinvites.com/demos/meenaya — scraped copy

Source: https://www.missingpieceinvites.com/demos/meenaya
Scraped: 2026-05-30

## Files
- `page.html` — original unmodified HTML from the server
- `index.html` — same HTML, asset URLs rewritten to local paths
- `images/` — 33 images (PNG/SVG, ~21 MB total)
- `fonts/` — 44 web fonts (woff2)
- `js/` — Framer JS bundles + CSS (the site is a Framer SPA, so most layout/copy lives in these `.mjs` files, not the HTML)
- `asset_urls.txt`, `images_to_dl.txt`, `js_css_urls.txt` — URL lists from the scrape

## Viewing locally
```
cd ~/missingpiece-scrape
python3 -m http.server 8000
# open http://localhost:8000/index.html
```
A file:// open will likely break — the Framer JS bundles use ES module imports
that browsers refuse to load from disk. Serve over HTTP.

## Caveats
- The site is built with Framer. The HTML you see is mostly a shell; the real
  layout, copy and animations are generated at runtime from the `.mjs` bundles
  in `js/`. Editing the HTML alone won't change much — the JS rerenders the DOM.
- 2 `framerusercontent.com` references remain in `index.html` (likely inside JS
  string blobs that the rewrite regex deliberately skipped).
- A couple of third-party analytics scripts (pixelflow, contentsquare) are also
  in `js/` — strip those if you reuse anything.

## Copyright
Images, fonts (where licensed per-site), and copy are © Missing Piece Invites.
This is fine for reference / studying the layout. **Do not republish the images
or copy on a live site without permission** — that's infringement. The HTML/CSS
structure itself is generally fair to learn from.
