# hendersonshapiro.com — Static Site

A fully self-contained, static rebuild of **hendersonshapiro.com** — no WordPress,
PHP, database, or plugins. Just HTML, CSS, JavaScript, and asset files you can host
anywhere (GitHub Pages, DigitalOcean, any static host).

**Important:** the HTML pages are *not* self-sufficient on their own. The design
(all CSS) and behavior (menu, search, portfolio filtering) live in the
`wp-content`, `wp-includes`, and `_vendor` folders. You must upload **all** of these
files together, keeping the folder structure exactly as-is, or the site will appear
unstyled. Images and fonts are embedded inside the HTML, but CSS and JS are separate
files.

---

## Folder structure (upload everything, at the repo root)

```
index.html                 ← Home
404.html                   ← Not-found page
about/index.html           ← About
team/index.html            ← Team
portfolio-main/index.html  ← Our Work (portfolio + filtering)
contact/index.html         ← Contact (form + map)
robots.txt
favicon.jpg
.nojekyll                   ← tells GitHub Pages to serve every folder (see below)
wp-content/                ← ALL the CSS + JavaScript live here (the design!)
wp-includes/               ← jQuery
_vendor/                   ← font files (used by the 404 page)
deploy/                    ← optional hosting configs (nginx, Docker, DigitalOcean)
README.md                  ← this file
```

The pages reference assets by relative path (e.g. `wp-content/uploads/…css`), so
`wp-content` has to sit right next to `index.html` at the root of whatever the host
serves.

---

## Deploy to GitHub Pages

1. Put **all** the files above at the root of your repository (not just the `.html`
   files — include `wp-content`, `wp-includes`, `_vendor`, and the `.nojekyll` file).
   Using GitHub Desktop or `git` from the command line is far easier than the web
   uploader for this many files:
   ```bash
   git add .
   git commit -m "Add static site"
   git push
   ```
2. In your repo: **Settings → Pages**. Set **Source** to "Deploy from a branch",
   pick your branch (usually `main`) and the folder **`/ (root)`**. Save.
3. Wait a minute, then open the URL GitHub shows you.

The `.nojekyll` file is required: without it, GitHub Pages runs Jekyll, which ignores
folders that start with an underscore (like `_vendor/`). The file is already
included — just make sure it gets committed.

To use your custom domain (hendersonshapiro.com), add it under **Settings → Pages →
Custom domain** and follow the DNS instructions there.

---

## Preview locally

Every image and font is embedded, so you can **double-click `index.html`** and it
opens correctly offline. Two things need a web server to work fully (the contact-page
map tiles and, if you switch to an inline form service, the contact form) — to
preview those, run a tiny local server from this folder:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

---

## Deploy to DigitalOcean (alternative)

The `deploy/` folder has ready configs:

- **App Platform (static site):** point a new App at your GitHub repo and set the
  source directory to `/` (see `deploy/do-app-platform.yaml`).
- **Droplet + nginx:** copy all the site files to `/var/www/hendersonshapiro` and use
  `deploy/nginx.conf`.
- **Docker:** `docker build -f deploy/Dockerfile -t hendersonshapiro .`

---

## What works

- **Design/layout** — full Avada design, fonts and images embedded.
- **Pop-out menu** — the hamburger icon opens the right-side slide-in menu.
- **Search** — the magnifier icon opens a panel and searches all pages in the browser
  (client-side, no server needed).
- **Portfolio filtering** — the category buttons on Our Work filter instantly.
- **Contact form** — opens the visitor's email app pre-filled to
  `info@hendersonshapiro.com`. To get an inline "message sent" form instead, wire it
  to a free service like Web3Forms or Formspree (edit `contact/index.html`; the form
  field names are already set up).

## Runtime notes

- The **contact-page map tiles** stream live from OpenStreetMap — that's inherent to
  any web map and needs internet; the marker and everything else are embedded.
- The asset folders are still named `wp-content` / `wp-includes` — cosmetic only,
  nothing WordPress runs. They can be renamed later if you rewrite the references.
