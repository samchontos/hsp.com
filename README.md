# hendersonshapiro.com — Static Site

A fully self-contained, static rebuild of **hendersonshapiro.com**. The design,
content, links, images, and portfolio sorting are identical to the original
WordPress site, but there is **no WordPress, PHP, database, or plugins** — just
HTML, CSS, JavaScript, and asset files you can host anywhere, including
DigitalOcean.

Every image, font, stylesheet, and script the pages need is bundled inside the
package, so nothing loads back from the old WordPress server. (Two things still
talk to the internet at run time by design — see **Runtime notes** below.)

---

## What's included

```
hendersonshapiro-website/
├── public/                     ← the website (this whole folder is the web root)
│   ├── index.html              ← Home
│   ├── about/index.html        ← About
│   ├── team/index.html         ← Team
│   ├── portfolio-main/index.html  ← Our Work (portfolio + category filtering)
│   ├── contact/index.html      ← Contact (form + map)
│   ├── 404.html                ← Custom, on-brand "page not found"
│   ├── robots.txt
│   ├── favicon.jpg
│   ├── _vendor/fonts/          ← Bundled web fonts (Open Sans, Roboto Slab,
│   │                             Crete Round, Font Awesome 5)
│   ├── wp-content/             ← All images, the capabilities PDF, the background
│   │                             video, theme CSS/JS (kept as folder names only;
│   │                             nothing here runs WordPress)
│   └── wp-includes/            ← jQuery (used by the theme scripts)
├── deploy/
│   ├── nginx.conf              ← nginx site config for a DigitalOcean Droplet
│   ├── Dockerfile              ← Container image (nginx) for Droplet or App Platform
│   ├── nginx.docker.conf       ← nginx config used inside the container
│   └── do-app-platform.yaml    ← DigitalOcean App Platform static-site spec
└── README.md                   ← this file
```

The five pages and every navigation link match the original site's URL
structure (`/`, `/about/`, `/team/`, `/portfolio-main/`, `/contact/`).

---

## Preview it locally

All paths are relative, so you can just **double-click `public/index.html`** and
it opens correctly in your browser — styles, images, fonts, and the portfolio
filtering all work offline.

Two things need a web server to function fully (they can't work from a bare
file on disk): the contact-page **map tiles** and, if you switch to an inline
form service, the **contact form**. To preview those exactly as they'll behave
when hosted, run a tiny local server instead:

```bash
cd public
python3 -m http.server 8080
# then open http://localhost:8080
```

---

## Deploy to DigitalOcean

Three good options. **App Platform (static site)** is the simplest and cheapest.

### Option A — App Platform, static site (recommended)

1. Put this project in a GitHub repo (keep the `public/` folder as-is).
2. In the DigitalOcean control panel: **Create → Apps → pick your repo.**
   App Platform detects a static site automatically.
3. Set the **source/output directory** to `/public`
   (or import `deploy/do-app-platform.yaml`, editing the repo name first).
4. Add your domain under **Settings → Domains**; DigitalOcean issues a free
   HTTPS certificate and gives you the DNS records to point
   `hendersonshapiro.com` at the app.

Directory URLs like `/about/` resolve automatically because each section has an
`index.html`.

### Option B — Droplet + nginx

1. Create an Ubuntu Droplet and install nginx: `sudo apt update && sudo apt install nginx`.
2. Copy the site up:
   ```bash
   scp -r public/* root@YOUR_DROPLET_IP:/var/www/hendersonshapiro/
   ```
3. Copy `deploy/nginx.conf` to `/etc/nginx/sites-available/hendersonshapiro`, then:
   ```bash
   sudo ln -s /etc/nginx/sites-available/hendersonshapiro /etc/nginx/sites-enabled/
   sudo nginx -t && sudo systemctl reload nginx
   ```
4. Point DNS at the Droplet, then enable HTTPS:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d hendersonshapiro.com -d www.hendersonshapiro.com
   ```

### Option C — Docker (Droplet or App Platform container)

From the project root:

```bash
docker build -f deploy/Dockerfile -t hendersonshapiro .
docker run -p 8080:80 hendersonshapiro      # http://localhost:8080
```

---

## Portfolio sorting

The **Our Work** page keeps the original category filtering exactly as-is: the
"All / B2B / Construction / CPG / Education / Featured / Finance / Government /
Healthcare / Military / Non-Profit / Pharmaceutical / Publishing / Retail & B2C /
Technology" buttons filter the grid instantly in the browser (Isotope). All 36
projects and their category tags are baked into the page, so no server or
database is involved. This was verified — e.g. clicking **Government** narrows
the grid to its 9 matching projects.

---

## Runtime notes (two intentional exceptions)

Everything renders offline **except** two items that are inherently live services:

1. **Contact-page map.** The map background tiles stream from OpenStreetMap at
   run time — that's how every web map works; the whole world can't be bundled.
   The map, its marker, and the popup are otherwise fully local.

2. **Contact form.** A static site has no PHP to receive form posts, so the form
   is wired to open the visitor's email app pre-filled to
   `info@hendersonshapiro.com` (works immediately, no backend). To get a modern
   inline "message sent" experience instead, swap in a form service — see below.

### Optional: inline contact-form submission

Free drop-in services (no server needed): **Web3Forms** or **Formspree**.
Example using Web3Forms — edit `public/contact/index.html`:

1. Change the form tag's action and method:
   ```html
   <form id="hsp-contact-form" action="https://api.web3forms.com/submit" method="post" ...>
   ```
2. Add one hidden field inside the form (get a free key at web3forms.com):
   ```html
   <input type="hidden" name="access_key" value="YOUR-ACCESS-KEY">
   ```
3. Remove the small `<script>` block near the bottom of that page that starts
   with `HSP static contact-form handler` (that's the mailto fallback).

The existing field names (`your_name_required`, `your_email_required`,
`your_subject`, `your_message`) are sent along automatically.

---

## Notes

- Fonts are the genuine open-source families used by the original design
  (Open Sans, Roboto Slab, Crete Round) plus Font Awesome 5 icons, all bundled
  locally so nothing depends on Google Fonts or a CDN.
- The theme's asset folders are still named `wp-content` / `wp-includes`. That's
  cosmetic only — nothing WordPress runs. If you'd prefer non-WordPress-looking
  paths later, those folders can be renamed and references updated.
- The **pop-out menu** (hamburger icon, top-right) and **search** (magnifier icon)
  work as on the live site. The menu slides in from the right with the full nav;
  the search opens a panel and searches across all pages instantly in the browser
  (client-side — no server needed). Both the icons and all fonts are embedded, so
  they render in any browser even when the file is opened directly.
