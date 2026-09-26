# copilotlabs.dev — maintainer notes

Hosted at **https://copilotlabs.dev/** (GitHub Pages, repo `Co-Pilot-Labs/co-pilot-labs.github.io`,
custom domain in `CNAME`). Push to `main` deploys. Plain HTML + one stylesheet — no build step,
no npm, no JavaScript framework.

This `_docs/` folder is **not published**: GitHub Pages' Jekyll (legacy) build skips folders that
start with `_`. Never add a `.nojekyll` file, or `_docs/` becomes public.

## Structure

```
index.html                 Co-Pilot Labs studio page (links to each app)
tarmaclog/index.html       Tarmac Log marketing page
support/ privacy-policy/ terms/ delete-account/   Legal + support (URLs are registered in
                           App Store Connect, Play Console and shipped app builds — never move them)
auth/callback/index.html   Email-verification deep-link bounce page (do not touch its script)
style.css                  The only stylesheet (tokens, @font-face, all components)
favicon.svg
robots.txt  sitemap.xml  llms.txt   SEO / AI-assistant files
assets/fonts/              IBM Plex Sans 400/600/700 + Plex Mono 500 (WOFF2 subsets) + OFL.txt
assets/img/                app-icon.png, apple-touch-icon.png, og-image.png
assets/img/screens/        01_offline.webp … 06_export.webp (store screenshots)
assets/img/badges/         app-store.svg, google-play.png (official store badges)
```

## Palette (Tarmac Log dark tokens, from `AppColors` in the app)

| variable | value | app token |
|---|---|---|
| `--bg` | `#08101A` | `bg` |
| `--surface` | `#151C25` | `surface` |
| `--surface-2` | `#252C35` | `surface2` |
| `--border` | `#313942` | `border` |
| `--text` | `#EDF2F9` | `ink` |
| `--text-muted` | `#A3ACB8` | `inkMuted` |
| `--text-faint` | `#676F7A` | `inkFaint` |
| `--accent` | `#F2A618` | `amber` — buttons, highlights |
| `--accent-ink` | `#190F03` | `amberInk` — text on amber |
| `--accent-hover` | `#F5B840` | lighter amber |
| `--link` | `#47C5D2` | `cyan` — links |
| `--danger` | `#FA6863` | `danger` — warning boxes |

Hex values live only in `:root` of `style.css`. Never hardcode colours in HTML or elsewhere in the
CSS. (`theme-color` meta tags carry `#08101A` because browsers need a literal there.)

## Nav / footer pattern (every page)

```html
<header>
  <nav aria-label="Main">
    <a href="/" class="nav-brand">Co-Pilot Labs</a>
    <ul class="nav-links">
      <li><a href="/tarmaclog/">Tarmac Log</a></li>
      <li><a href="/support/">Support</a></li>
      <li><a href="/privacy-policy/">Privacy</a></li>
      <li><a href="/terms/">Terms</a></li>
    </ul>
  </nav>
</header>
…
<footer>
  <p>© 2026 Co-Pilot Labs &nbsp;·&nbsp;
    <a href="/privacy-policy/">Privacy Policy</a> &nbsp;·&nbsp;
    <a href="/terms/">Terms of Use</a> &nbsp;·&nbsp;
    <a href="/support/">Support</a>
  </p>
</footer>
```

Mark the current page's link with `aria-current="page"`. Every public page also needs: a unique
`<title>`, `<meta name="description">`, a canonical on `https://copilotlabs.dev/<path>/`, Open
Graph + Twitter card tags, and exactly one `<h1>`. Add new public pages to `sitemap.xml`.

## Marketing pages — rules

- **Claims come only from the approved store copy** (`docs/marketing/app-store-copy.md` in the app
  repo). Never claim FAA/EASA compliance or certification, encryption at rest, iPad support, or
  anything the store listing doesn't say. No prices on the site — Pro is "an optional subscription,
  purchased in the app".
- **No third-party requests**: no analytics, trackers, cookies, Google Fonts or CDN scripts. Fonts,
  badges and images are all local files.
- **No personal information**: no individual names, personal emails, phone numbers, addresses or
  photos. The only contact address is the shared support group, and it appears on the support and
  legal pages only.
- Keep `FAQPage` JSON-LD in `tarmaclog/index.html` word-for-word in sync with the visible FAQ.

## Refreshing assets

**Store badges** — official artwork, never recoloured or altered:
- Apple: `curl -L -o assets/img/badges/app-store.svg https://tools.applemediaservices.com/api/badges/download-on-the-app-store/black/en-us`
- Google: `curl -L -o assets/img/badges/google-play.png https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png`

Both render at the same visible height: Apple SVG at 48px, the Google PNG at 70px (it has built-in
transparent padding, trimmed with a negative margin in `.badge-google`). The trademark line in the
`/tarmaclog/` footer must stay while the badges are shown. Store links:
`https://apps.apple.com/app/tarmac-log/id6756934482` and
`https://play.google.com/store/apps/details?id=com.hanrich.logbook`.

**Screenshots** — from the app repo's `docs/marketing/store-screenshots/play-phone/*.png`
(1080×1920), run from the app repo root:

```bash
for p in docs/marketing/store-screenshots/play-phone/*.png; do
  n=$(basename "$p" .png)
  magick "$p" -resize 540x /tmp/$n.png
  cwebp -q 80 /tmp/$n.png -o website/assets/img/screens/$n.webp
done
```

Keep each under ~80 KB, keep the file names, and update the `alt` text and captions in
`tarmaclog/index.html` if what a screen shows changes (captions mirror `slots.json`).

**Fonts** — WOFF2 subsets of the app's `assets/fonts/*.ttf`:

```bash
pyftsubset assets/fonts/IBMPlexSans-Regular.ttf --flavor=woff2 \
  --unicodes="U+0000-00FF,U+2013-2014,U+2018-201D,U+2022,U+2026,U+2192,U+2708" \
  --output-file=website/assets/fonts/IBMPlexSans-Regular.woff2
```

Characters outside that range fall back to the system font — extend `--unicodes` if new copy
needs them.

**Icons / OG image** — `app-icon.png` (512) and `apple-touch-icon.png` (180) come from the app's
`assets/images/app_icon_ios.png` (the opaque store icon). `og-image.png` is the Play feature graphic
scaled onto a 1200×630 `#08101A` canvas and quantised to 256 colours to stay small.
