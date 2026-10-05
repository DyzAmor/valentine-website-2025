# AGENTS.md

## What this project is
A **fully static website** — hand-written HTML/CSS/JS with **no build step, no bundler, no package manager, no package.json, no backend and no database**. Files served as-is: `index.html`, `config.js`, `theme.js`, `script.js`, `styles.css`.

## Running it (Base44 sandbox)
```bash
docker compose -f docker-compose.base44.yml up -d   # serves repo root on :3000
```
`web` is a `python:3.12-slim` container running `python -m http.server 3000 --bind 0.0.0.0 --directory /app` with the repo bind-mounted at `/app`. It serves the cloned source directly (no prebuilt image), so edits to any file appear on the next request.

## Non-obvious quirks
- **No live reload / no file watcher.** `http.server` re-reads files from disk per request, but nothing pushes a refresh to the browser. After any edit, call `reload_preview` so the change shows in the preview.
- **Config-driven UI.** All copy, colors, animation timings and the music URL live in `config.js` (`window.VALENTINE_CONFIG`); `theme.js` maps `CONFIG.colors`/`CONFIG.animations` onto CSS custom properties applied at `DOMContentLoaded`, and `script.js` fills in every text node. Editing `config.js` is the normal way to change the site.
- **Load order matters** in `index.html`: `config.js` → `theme.js` → `script.js`. `script.js` runs top-level `getElementById('loveMeter')` before `DOMContentLoaded`, so those elements must stay above the script tags in the body.
- **No secrets and no env vars.** The background-music URL is the project's own committed Cloudinary link in `config.js`; nothing is read from the environment, so `docker-compose.base44.yml` deliberately has no `env_file` and `.base44/environment.json` lists no secrets. Adding a real credential would require `set_secrets`.
- **No repository Dockerfile / compose is used.** The repo ships none; `docker-compose.base44.yml` is the only compose file and is the runbook.

## How to verify it works
```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/     # expect 200
curl -s http://localhost:3000/config.js | head -3                   # expect live source, not a bundle
```
Interactive checks: the three question flows (`moveButton` for the playful "No" buttons, `showNextQuestion`, the secret answer → question 2 → question 3), the love meter slider updating `#loveValue` and `#extraLove`, and `celebrate()` revealing `#celebration`.
