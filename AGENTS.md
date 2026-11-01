# Project notes

## What this app is

A **single static HTML file** (`index.html`) — an academic performance dashboard
("Dashboard de Desempeño Académico") in Spanish, for I.E. Luis Enrique Figueroa Rey.
There is no backend, no build step, no package manager, and no database.

- All dashboard data is **encrypted inside the HTML** (`ENC` object: AES-GCM,
  key derived from the user's password with PBKDF2-SHA256, 150k iterations).
  The page therefore opens on an "Acceso restringido" password screen; entering
  the school's password decrypts and renders the data in the browser.
  The password is **not stored in the repo** — only the data's owner knows it.
- External dependencies are CDN-only: Chart.js 4.4.0 (jsDelivr) and IBM Plex
  fonts (Google Fonts). No API keys or credentials are required.
- Logos are inline base64 data URIs.

## Running it in the sandbox

```bash
docker compose -f docker-compose.base44.yml up -d --build
```

`web` serves the repo root (bind-mounted read-only) with nginx on host port 3000.
Because nothing is compiled, **edits to `index.html` appear on reload** — no
rebuild or restart needed.

Sandbox-specific details (see `.base44/nginx.conf`):

- nginx workers run as `root` because the sandbox checkout is root-owned (mode 700);
  the default unprivileged `nginx` user gets 403.
- Hidden paths (`.git`, `.base44`) are denied because the whole repo is mounted.
- Caching is disabled so edits show immediately.

## Deployment (GitHub Pages)

The site is published with GitHub Pages from `.github/workflows/pages.yml`
(publishes the repo root on every push to `main`). One-time setup in the repo:
**Settings → Pages → Source: "GitHub Actions"**. The public URL is
`https://oscarsuarez13.github.io/lefr./`.

## Verifying it works

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/   # expect 200, ~64 KB
docker compose -f docker-compose.base44.yml ps                    # expect web (healthy)
```

The password screen, wrong-password error, and Chart.js/font loading can be
checked without the password. The decrypted dashboard can only be verified by
entering the school's password in the page.
