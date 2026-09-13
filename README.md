# techvalueinvestor.com

The **Tech Value Investor** publication: daily market analysis and a weekly
letter on technology, capital, and what it costs.

**Live site:** [techvalueinvestor.com](https://www.techvalueinvestor.com)

Sibling of [prabhukasinathan.com](https://github.com/pkasinathan/prabhukasinathan.com),
which is the personal AI and career site. Same operational model, different
brand, different content track, deliberately separate repos.

---

## Most of this repo is generated

`blog/`, `newsletter/`, `index.html`, `404.html`, `disclaimer/`, `rss.xml` and
the marked block inside `sitemap.xml` are all written by the
[`tvi-brandsite`](https://github.com/pkasinathan/tvi/tree/main/tvi-brandsite)
CLI from source markdown in `tvi-media`. **Editing them by hand is pointless:**
the next publish rebuilds the whole site and overwrites the lot.

There are exactly three things a human edits here.

| File | What it controls |
| --- | --- |
| `content/site.json` | Every word that is not a generated post: hero, section headings, subscribe block, footer links, the disclaimer |
| `assets/css/main.css` | The entire design system. One hand-written stylesheet, no framework, no build step |
| `assets/images/` | The brand mark and the social card |

That split is the point. Copy changes and colour changes are a commit here, not
a release of the publisher.

---

## Tech stack

- Static HTML5, one hand-written CSS file, and **zero JavaScript**
- Inter from Google Fonts, system monospace for figures and tickers
- Cloudflare Workers static assets, no build step
- GitHub for source control and as the deploy trigger

**No JavaScript is a requirement, not a flex.** X cards, WhatsApp previews,
Buttondown, and Google's crawler all read raw markup and none of them execute
scripts, so every page ships as a finished document with its metadata already
in the HTML. The only `<script>` tags on the site are `application/ld+json`
structured data, which is markup rather than code.

---

## Structure

```
.
├── index.html                  # GENERATED: hero + latest analysis + latest issues
├── 404.html                    # GENERATED
├── wrangler.jsonc              # Cloudflare Workers static-assets config
├── manifest.json               # PWA manifest
├── robots.txt
├── sitemap.xml                 # hand-maintained, content URLs injected between markers
├── rss.xml                     # GENERATED: the daily analysis feed
├── blog/                       # GENERATED: daily analysis
│   ├── index.html
│   └── YYYY-MM-DD-<slug>/index.html
├── newsletter/                 # GENERATED: the weekly letter
│   ├── index.html
│   ├── feed.xml                # full-content feed, so an email provider can mail it
│   └── YYYY-MM-DD-<slug>/index.html
├── disclaimer/index.html       # GENERATED from content/site.json
├── content/
│   └── site.json               # HAND-EDITED: all editorial copy
└── assets/
    ├── css/main.css            # HAND-EDITED: the whole design system
    └── images/
        ├── icons/favicon.svg
        └── og/
            ├── og-image.svg        # source of truth for the social card
            └── og-image-1200.png   # rendered from it; no platform renders an SVG card
```

Regenerate the social card after editing the SVG:

```bash
rsvg-convert -w 1200 -h 630 assets/images/og/og-image.svg \
  -o assets/images/og/og-image-1200.png
```

---

## Publishing

From the publisher, not from here:

```bash
cd ~/workspace/tvi/tvi-brandsite

./.venv/bin/tvi-brandsite doctor              # is everything wired up
./.venv/bin/tvi-brandsite publish --dry-run   # writes nothing
./.venv/bin/tvi-brandsite publish --no-commit # renders, leaves it uncommitted
./.venv/bin/tvi-brandsite publish             # renders, commits, pushes, deploys
```

**Commit equals deploy.** Cloudflare ships every push to `main` in roughly 30
to 60 seconds, so the last command above is live-affecting. Rehearse with
`--dry-run` or `--no-commit` first.

---

## Local preview

There is no build step, but the pages use absolute asset paths, so `file://`
will not load the stylesheet. Serve it:

```bash
python3 -m http.server 8911 --bind 127.0.0.1
```

Then open `http://127.0.0.1:8911/`. The `--bind 127.0.0.1` is not optional: this
laptop travels, and a static server on `0.0.0.0` is reachable by everyone on the
same coffee-shop Wi-Fi.

---

## Deployment (Cloudflare)

| Setting | Value |
| --- | --- |
| Project name | `techvalueinvestor-com` |
| Build command | *(leave empty)* |
| Deploy command | `npx wrangler deploy` |
| Branch | `main` |
| Path | `/` |

The domain is registered at Cloudflare, so DNS needs no transfer. Attach
`techvalueinvestor.com` and `www.techvalueinvestor.com` as custom domains on the
Worker after the first deploy.

---

## Nothing private belongs here

Every push to `main` publishes. `.gitignore` blocks `.env`, keys, and certs as a
safety boundary rather than for tidiness, and the generator only ever reads the
STOCK content track, so nothing personal, financial, or family-related has a
path into this repo by design.

---

**Status:** Live · Generated nightly
