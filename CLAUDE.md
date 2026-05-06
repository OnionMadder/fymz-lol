# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`fymz.lol` (Find Your Madder Zone) — a static HTML/CSS/JS site of free, ad-free kids' games hosted on **GitHub Pages**. The custom domain (`CNAME`) and `404.html` at the repo root are GitHub-Pages-specific.

There is **no build system, no package manager, no test suite, no linter**. The repo has no `package.json`. Pushing to the configured Pages branch is the deploy.

## Running / previewing locally

Since most pages reference assets via absolute production URLs (e.g. `https://fymz.lol/cookie-cache/style.css` and `<script src="https://fymz.lol/...">`), opening files via `file://` will load the live site's assets, not your local edits.

To preview local edits:

```bash
# from repo root
python3 -m http.server 8000
# then open http://localhost:8000/<game-folder>/
```

But the cross-references inside each game's `index.html` will still pull from `https://fymz.lol/...`. When iterating on a game locally, temporarily switch its `<link>` / `<script>` URLs to relative paths, or accept that you're testing against the deployed CSS/JS. Note: a few newer pages (`bala-draws/`, `krazy-kritters/`, `krazy-kritters-flat/`) already use relative paths and preview correctly.

## Repo layout

```
/                   landing hub (index.html) + GitHub Pages files (CNAME, 404.html)
/assets/            site-wide CSS + favicons (used by the hub and 404 page)
/<game-name>/       one self-contained game per directory
/zextra-assets/     source / working art assets — NOT served, kept out of links
```

Each game folder is its own mini-app: `index.html` + `style.css` + `game.js` (or a `js/` directory), plus its `assets/` for that game. Games do not share JS code with each other or the hub.

### Per-game entry shapes

- **Single-file JS** (`cookie-cache`, `friend-picker`, `georges-jump`, `krazy-kritters`, `krazy-kritters-flat`): one `game.js` (~900–1500 lines) loaded as a regular script at the bottom of `index.html`.
- **Split modules, no bundler** (`gazonionaire`): `js/data.js` → `js/game.js` (pure state/rules, no DOM) → `js/ui.js` (DOM bindings). They share state via a window-level `Game` IIFE; `data.js` must load first.
- **ES modules** (`giggle-gears`): `<script type="module" src=".../main.js">`. Module graph handles load order — do not reorder script tags. Modules: `main.js`, `screens.js`, `audio.js`, `state.js`, `race.js`, `tricks.js`, `environment.js`, `sprites.js` (generated), plus a Node codegen step (see below).
- **Inline-styled single page** (`bala-draws`): all CSS inside `<style>` in `index.html`; no external `style.css`.

## Common conventions

- **Screen flow**: every game uses sibling `<div class="screen">` blocks (menu, game, end, etc.) and toggles a single `.active` class to switch screens. Don't re-architect; match the existing pattern.
- **Neon brand palette** (kept consistent across hub + games): `--accent-cyan: #00ffcc`, `--accent-pink: #ff00ff`, `--theme-color: #00ffcc`, dark `#050505` background with a faint cyan grid. Font is Inter.
- **Analytics**: every page includes the GoatCounter snippet pointing at `fymz.goatcounter.com`. Keep this on new pages.
- **SEO/meta block**: each game's `<head>` repeats the same OpenGraph/Twitter card pattern with `https://fymz.lol/...` absolute URLs, canonical link set to its own subpath, and shared FYMZ description/keywords. Mirror this for new games.
- **Year footer**: pages with footers set `document.getElementById("year").textContent = new Date().getFullYear();` — keep this.
- **Mute control**: games with audio expose a `#btn-mute` (or similar) and unlock the audio context on the first user gesture. Browser audio autoplay policy requires this — don't try to play sound before a tap/click.

## Sprite-sheet patterns (three different approaches — don't unify)

- **Cookie Cache** hard-codes `SHEETS` and `FRAMES` lookup tables in `cookie-cache/game.js` and applies them via an `applySprite(el, sheet, frame, w, h)` helper that sets `background-image` + `background-size` + `background-position`.
- **Krazy Kritters** ships per-sheet JSON metadata alongside PNGs (`alien.json`, `bio.json`, `mech.json`); `game.js` loads and consumes them at runtime.
- **Giggle Gears** has a Node codegen step that reads sprite-sheet JSON and writes both CSS and JS:

  ```bash
  cd giggle-gears/assets/js
  node build-sprites-css.js
  ```

  This regenerates `assets/css/sprites.css` and `assets/js/sprites.js`. **Both are auto-generated — do not hand-edit.** When sheets change, edit the JSON metadata and rerun. The script hardcodes `SHEET_BASE_JS = 'https://fymz.lol/giggle-gears/assets/sprite-sheets/'`; this absolute path is intentional (history of folder-rename bugs noted in the file's comments).

## Adding a new game

1. Create `/<game-name>/` with `index.html`, `style.css`, `game.js`, and `assets/`.
2. Copy the `<head>` meta block from an existing game and update title, canonical URL, and any game-specific OG image.
3. Add a `.game-card` entry to the root `index.html` game grid linking to `https://fymz.lol/<game-name>/`.
4. Use absolute `https://fymz.lol/...` URLs for shared favicons/manifest if you want to match existing games, or relative paths if you want local previewability — both patterns exist in the repo.

## Deploy

Pushing to the GitHub Pages branch deploys to `fymz.lol` automatically. The custom domain and 404 routing depend on `/CNAME` and `/404.html` at the repo root — don't move or rename them.

## Things to avoid

- Don't add a build system, bundler, framework, or `package.json` for the site as a whole. The whole point is "open file in browser." The Giggle Gears Node script is an exception scoped to that game's sprite generation.
- Don't introduce shared JS/CSS dependencies between games. Each game is intentionally self-contained so one can break without affecting others.
- Don't link to `zextra-assets/` from any served page. It's a workspace for source art.
- Don't strip the `goatcounter` snippet or the canonical/OG meta tags — they're load-bearing for analytics and link previews.
