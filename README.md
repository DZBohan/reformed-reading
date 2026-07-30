# reformed-reading

A self-hosted static site publishing **Chinese translations of classic Reformed works**, all
translated from **public-domain originals**. Text-first: readable, searchable, no reader accounts,
no paywall, no JavaScript required to read a chapter.

> **Note on language:** the site content is in **Chinese**. This README is in English so the
> tooling and licensing are legible to anyone who lands here; the books themselves are not
> translated into English.

🔗 **Live: https://reformedreadingchinese.com** (also https://reformed-reading.bohanzh6.workers.dev)
📦 Source: https://github.com/DZBohan/reformed-reading

## Current contents

**Herman Bavinck, *Magnalia Dei*** (Dutch original, 1909) — **chapters 1–10 of 24 published**:
the highest good · knowing God · general revelation · the value of general revelation ·
special revelation (mode) · special revelation (content) · Scripture · Scripture and the creeds ·
the being of God · the triune God. The rest are in progress.

Scripture references follow the **Chinese Union Version (和合本)** numbering throughout. Where
that differs from the original's Dutch Statenvertaling numbering — or where the original appears
to misquote — the difference is recorded in an **end-of-chapter footnote** (see `CONVENTIONS` in
the `reformed-translation` project).

## Stack

- **MkDocs + Material theme** — chapter-tree navigation, full-text search (Chinese and English),
  light/dark mode, mobile layout
- **Hosting: Cloudflare Workers** (static assets) — `wrangler.toml` sets
  `[assets] directory = "./site"`; global CDN, automatic HTTPS
- **Source: GitHub** — Cloudflare builds and deploys straight from the repo
- **Domain**: reformedreadingchinese.com (Cloudflare Registrar)

## Layout

```
reformed-reading/
├── README.md            # this file
├── logs/                # engineering notes / change history
├── mkdocs.yml           # site config (name / nav / theme / search / site_url)
├── wrangler.toml        # Cloudflare Workers config (serves ./site as static assets)
├── requirements.txt     # build dependency (mkdocs-material)
├── .gitignore           # ignores site/ and .venv/
└── docs/                # site content (markdown)
    ├── index.md         # home
    ├── copyright.md     # copyright / licensing
    ├── stylesheets/
    │   └── extra.css    # footnote typography overrides (see warning below)
    └── books/
        └── magnalia-dei/
            ├── index.md # book home (introduction + 24-chapter table of contents)
            └── NN.md    # one file per chapter, filled from reformed-translation
```

## Content pipeline (fully automated after push)

```
reformed-translation/books/<book>/zh/NN.md     translated chapter
        │  copied into this repo
        ▼
reformed-reading/docs/books/<book>/NN.md       (+ CC footer)
        │  update the TOC status table in index.md and the nav in mkdocs.yml
        │  git push → GitHub
        ▼
Cloudflare builds (pip install + mkdocs build) → npx wrangler deploy
        ▼
live within minutes at https://reformedreadingchinese.com
```

Every chapter page is Chinese body text plus a footer naming the source edition and the license.

Translation itself happens in the separate `reformed-translation` project; this repo is the
publishing layer. A push every few chapters is enough — Cloudflare handles the rest.

## Local preview and build

Dependencies live in `.venv/` (a plain `python3 -m venv`, not uv).

```bash
.venv/bin/mkdocs serve      # preview at http://127.0.0.1:8000
.venv/bin/mkdocs build      # build the static site → ./site
```

## Cloudflare deployment settings (Workers · already configured)

Recorded here for reference and in case the repo ever needs reconnecting:

| Setting | Value |
|---------|-------|
| Production branch | `main` |
| Build command | `pip install -r requirements.txt && mkdocs build` |
| Deploy command | `npx wrangler deploy` |
| Build variable | `PYTHON_VERSION = 3.11` |

`wrangler.toml` designates `./site` as the static asset directory. Pushing to `main` triggers
build and deploy automatically.

## Custom stylesheet — please read before editing

`docs/stylesheets/extra.css` (wired in via `extra_css` in `mkdocs.yml`) fixes footnote
typography: it hides the invisible placeholder back-arrow, widens the marker column so
two-digit footnote numbers are not clipped, and normalises item line-height and spacing.

> ⚠️ **Do not delete it, and do not override `margin-left` on footnote `li`** — that margin is
> what reserves room for the number. Removing it clips footnotes 10+ on narrow screens.
> Background and the failed attempts that preceded the fix:
> `logs/2026-07-27-footnote-typesetting-fixes.md`.

## Copyright

The **source texts are public domain** (Bavinck's Dutch original, 1909). Modern English
translations remain under copyright and are **not** used here — everything is translated directly
from the public-domain original.

The **Chinese translations are original work**, released under
**[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)**. See
`docs/copyright.md` for details.
