# server.golf

A fast, scannable index of ad-hoc HTTP static-server one-liners. Find your
language, hit copy, you're serving the current directory at `:8000`.

It renders [willurd's "Big list of http static server one-liners"][gist] gist as
a scannable list — a fixed left column of language names (with a family-colored
stripe so you can run your eye straight down), commands with one-click copy on
the right, a row of language icons that filter the list (with number hotkeys
`1`–`8`, and `0`/Esc to reset), and a live port field that rewrites `8000` everywhere.

[gist]: https://gist.github.com/willurd/5720255

## How it stays up to date (no build step)

`index.html` is a single self-contained file. On every visit it `fetch()`es the
raw gist (`https://gist.githubusercontent.com/willurd/5720255/raw`) client-side
and re-renders. GitHub serves that raw URL with `access-control-allow-origin: *`,
so the browser is allowed to read it directly.

- A **baked-in copy** of the gist markdown lives in a `<script id="seed">` tag.
  It paints instantly, works offline, and puts the command text in the HTML
  source so crawlers can read it (SEO).
- The live fetch then refreshes from the gist and caches the result in
  `localStorage`. Edit the gist → the page follows. Nothing to rebuild.

The freshness indicator in the footer shows `live from the gist` /
`offline · showing cached`.

## Why static (and not Ruby)

The content is one small Markdown file fetched at runtime, parsed in ~80 lines of
vanilla JS. There's no per-request work a server would do, so a static file is
faster (cacheable at the edge, no cold starts), cheaper, and can't fall over.
SEO is covered by the inlined seed + meta/OG/JSON-LD tags. A server-side render
would add a moving part without buying anything here.

If you ever want to dogfood the very list this site publishes, you can serve it
locally with the Ruby one-liner from the page itself:

```shell
ruby -run -ehttpd . -p8000
```

## Deploy

It's just one file — host it anywhere. To put it on `server.golf`:

**Cloudflare Pages** (recommended — free, edge-cached, easy custom domain)
1. Push this folder to a Git repo (or use `wrangler pages deploy .`).
2. In Cloudflare Pages, create a project from the repo, no build command, output
   dir = `/`.
3. Add `server.golf` as a custom domain; Cloudflare wires the DNS for you.

**GitHub Pages**
1. Push to a repo, enable Pages on the `main` branch (root).
2. Add a `CNAME` file containing `server.golf`, and point your DNS:
   `ALIAS/ANAME @ → <user>.github.io` (or the four A records GitHub lists).

**Netlify**: drag the folder into the dashboard, then add the custom domain.

**Any host / nginx / S3**: upload `index.html` (and `og.png`, `og-square.png`). Done.

> Deploy the `og*.png` files alongside `index.html` — the social/OpenGraph tags
> point at `https://server.golf/og.png` (1.91:1) and `…/og-square.png` (1:1).

## Files

- `index.html` — the whole app (HTML + CSS + JS + seed data).
- `og.png` — 1200×630 social-share image (wide / Twitter `summary_large_image`).
- `og-square.png` — 1200×1200 square share image (chat apps that crop square).
- `og.html` / `og-square.html` — sources for the images. To regenerate after editing,
  screenshot each at its size with any headless Chrome/Chromium:
  ```shell
  chromium --headless=new --no-sandbox --hide-scrollbars \
    --force-device-scale-factor=1 --window-size=1200,630 \
    --virtual-time-budget=8000 --screenshot=og.png "file://$PWD/og.html"

  chromium --headless=new --no-sandbox --hide-scrollbars \
    --force-device-scale-factor=1 --window-size=1200,1200 \
    --virtual-time-budget=8000 --screenshot=og-square.png "file://$PWD/og-square.html"
  ```
- `README.md` — this file.

## Credits

- Commands & all credit: [willurd's gist][gist] and its many contributors.
- Built by [rwrrll](https://rwrrll.com) · [@rwrrll](https://x.com/rwrrll).
