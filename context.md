# Project Context — Ultimate Tool Directory

## What this is

A static, cyberpunk-styled, searchable directory of AI tools, developer resources,
design apps, learning platforms, and useful web tools. **471 tools** across
**26 categories**, maintained as a personal curated collection by
[tejas-singh-0212](https://github.com/tejas-singh-0212).

- **No build step, no backend, no frameworks** — plain HTML + CSS + vanilla JS.
- Deployed via **GitHub Pages** (push to `main` auto-deploys) at a custom domain
  (see `CNAME`; README references `tejastools.qd.je`).
- Repo: https://github.com/tejas-singh-0212/tool-directory (public).

## Repository structure

```
tool-directory/
├── index.html                          # Single-page UI shell (111 lines)
├── style.css                           # All styling — cyberpunk theme, dark/light modes
├── script.js                           # All app logic — fetch, filter, render (~400 lines)
├── tools.json                          # THE data file — 471 tool records (~2,800 lines)
├── CNAME                               # GitHub Pages custom domain
├── README.md                           # User-facing docs (setup, validation, categories)
├── scripts/
│   ├── validate-tools-json.py          # CI validator for tools.json
│   └── check-links.py                  # Link checker for all tool URLs
└── .github/workflows/
    ├── validate-tools-json.yml         # CI: validate on push/PR touching tools.json
    └── check-links.yml                 # Scheduled monthly link check + manual dispatch
```

`.playwright-mcp/` and `.kilo/` are local tooling artifacts (gitignored).

## The data model (`tools.json`)

Top-level JSON array of tool records. One record:

```json
{"name": "Unique Name", "url": "https://...", "purpose": "One-line description", "category": "One of 26 categories"}
```

Rules enforced by the validator (`scripts/validate-tools-json.py`):

- All four fields (`name`, `url`, `purpose`, `category`) are **required, non-empty strings**.
- `name` must be **unique** (case-insensitive); `url` must be **unique** (normalized:
  trimmed, lowercased, trailing slash stripped).
- `url` must be a valid `http(s)` URL.
- `category` must be one of the 26 allowed: AI Agents, AI Chat, AI Writing, AI Coding,
  Web Builder, Design, Image Editing, Image Generation, Video, Audio & Music,
  Presentation, Research, Learning, Coding Practice, Developer Tools, Productivity,
  Automation, Jobs & Career, Travel, 3D & CAD, Games, Utilities, Cybersecurity,
  Hardware & Simulation, Marketplaces, Other.

Current category distribution (largest first): Developer Tools 71, Design 48,
Learning 37, Utilities 27, AI Coding 26, AI Writing 26, Video 23, Web Builder 22,
Coding Practice 20, Image Editing 19, Jobs & Career 19, Image Generation 16,
Productivity 14, Research 13, Automation 11, AI Chat 11, 3D & CAD 10,
Audio & Music 10, Games 10, Presentation 9, AI Agents 8, Cybersecurity 8,
Other 5, Travel 4, Marketplaces 3, Hardware & Simulation 1.

## Frontend behavior

`index.html` is a thin shell; everything dynamic lives in `script.js`:

- **Data flow:** on `DOMContentLoaded`, fetches `tools.json` with `cache: 'no-store'`,
  sorts tools alphabetically by name, populates the category `<select>` with counts,
  and renders rows into `#toolTableBody`. Shows a loading row first and a detailed
  error row (with "run a local server" hint) if fetch/parse fails.
- **Search:** live text filter (`#toolSearch`) matching name, purpose, url, and
  category (case-insensitive substring).
- **Category filter:** `<select>` with per-category counts; "All categories (N)" default.
- **Clear button:** appears only when a search term or non-"all" category is active.
- **Tool counter badge:** shows `TOOLS: N` normally, `SHOWING: x / N` while filtering.
- **Empty state:** contextual message explaining which search term / category matched nothing.
- **Copy-link buttons:** per-row button using Clipboard API with
  `document.execCommand('copy')` fallback; icon swaps to a checkmark + toast for 2s.
- **Theme toggle:** dark (default) / light via `body.light-mode`; persisted in
  `localStorage` under key `theme` (try/catch-guarded).
- **XSS safety:** rows are built with `createElement`/`textContent`, never HTML injection.
- Asset URLs are cache-busted with `?v=3` (`style.css?v=3`, `script.js?v=3`) — bump
  these when editing CSS/JS.

`style.css` provides the cyberpunk aesthetic: Orbitron display font (Google Fonts),
neon-on-dark palette, light-mode override class, responsive/mobile-friendly layout,
and the floating GitHub link / counter / theme-toggle controls.

## Validation & CI

Run locally from repo root:

```bash
python3 -m json.tool tools.json > /tmp/tools.valid.json   # syntax check
python3 scripts/validate-tools-json.py                    # schema/duplicate/category check
python3 scripts/check-links.py [--strict]                 # live HTTP check of every URL
python3 -m http.server 8000                               # local dev server (NOT file://)
```

- `check-links.py`: pure-stdlib crawler, HEAD-then-GET per URL, 0.25s delay between
  requests, 10s timeout. Treats 2xx/3xx as OK, 401/403/429 as ALLOWED (auth-gated but
  alive), reports redirects. `--strict` exits non-zero on broken links.
- **CI (`.github/workflows/`):**
  - `validate-tools-json.yml` — runs both JSON checks on every push to `main` and
    every PR touching `tools.json`, the validator, or the workflow itself.
  - `check-links.yml` — monthly cron (1st of month, 00:00 UTC) plus manual dispatch
    with optional strict mode.

## Contributing a tool (the typical change)

Append/edit records in `tools.json`, then:

1. `python3 scripts/validate-tools-json.py` must pass.
2. Push to `main` — CI validates, Pages redeploys.
3. Keep names unique and pick an existing category (don't invent new ones without
   updating both the README list and `ALLOWED_CATEGORIES` in the validator).

## Gotchas

- **Don't open `index.html` via `file://`** — the `fetch('tools.json')` will fail
  under CORS; always serve over HTTP locally.
- Duplicate names/URLs (even differing by case or trailing `/`) fail CI.
- README's "Non-deployable" section lists piracy-adjacent links kept only as README
  references, intentionally excluded from deployment consideration.
- The worktree should stay clean after pushes; stray formatting edits to `tools.json`
  (indentation) have happened before — validate before committing.
