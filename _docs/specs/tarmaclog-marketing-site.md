# Feature Spec: Tarmac Log Marketing Site

**Created**: 2026-09-26
**Branch**: `feature/tarmaclog-marketing-site` — in the **website repo** (`website/` submodule,
`Co-Pilot-Labs/co-pilot-labs.github.io`), not the root repo
**Spec location**: `website/_docs/specs/` (website repo; `_` folders are not published)
**Status**: approved

---

## 1. User Story

As a pilot searching for a logbook app (on Google, or by asking an AI assistant), I want a clear,
fast page about Tarmac Log that tells me what it does and where to download it, so that I can
install it from my phone's app store.

**Acceptance Criteria:**
- [ ] `https://copilotlabs.dev/tarmaclog/` is the Tarmac Log marketing page (`website/tarmaclog/index.html`).
- [ ] It states plainly that Tarmac Log is available on the **App Store** and **Google Play**, with
      official store badges linking to:
      - App Store: `https://apps.apple.com/app/tarmac-log/id6756934482`
      - Google Play: `https://play.google.com/store/apps/details?id=com.hanrich.logbook`
        (developer's decision: link it live now, even though the Play page is not public yet)
- [ ] Badges appear in the hero **and** again in a closing call-to-action section.
- [ ] Shows the 6 store screenshots (offline, logbook, entry, aircraft, totals, export) as
      compressed WebP with `alt` text describing each screen.
- [ ] Shows Free vs Pro **without prices**: Free = up to 3 flights plus aircraft, crew and airports
      setup; Pro = unlimited flights and SACAA-format PDF logbook export.
- [ ] Has a short FAQ (5–6 questions) visible on the page and mirrored in `FAQPage` JSON-LD.
- [ ] Root `https://copilotlabs.dev/` becomes a simple **Co-Pilot Labs** studio page: one short
      paragraph about the studio and one card linking to `/tarmaclog/`. No personal info.
- [ ] SEO/AI files exist at the site root: `robots.txt`, `sitemap.xml`, `llms.txt`.
- [ ] Every public page has: unique `<title>`, `<meta name="description">`, canonical on
      `https://copilotlabs.dev/…`, Open Graph + Twitter card tags, exactly one `<h1>`.
- [ ] The whole site uses the Tarmac Log brand palette (dark) and IBM Plex, self-hosted.
- [ ] **No personal information** anywhere: no individual names, personal emails, phone numbers,
      addresses, or photos. The only contact is the existing shared
      `tarmac-log-support@googlegroups.com` (support page only — do not add it to new pages).
- [ ] **No third-party requests**: no analytics, trackers, cookies, Google Fonts, or CDN scripts.
      Store badges are local files.
- [ ] Works at 360px wide with no horizontal scroll; readable at desktop widths.
- [ ] Existing URLs keep working unchanged: `/privacy-policy`, `/terms`, `/support`,
      `/delete-account`, `/auth/callback`. The Flutter app is **not** modified.

## 2. Architecture Decisions

- **Scope**: website repo only. Plain HTML + CSS, no build step, no npm, no JS framework (per
  `.cursor/rules/website/RULE.md`). No JavaScript is needed on the new pages.
- Subscription tier / tables / edge functions / PowerSync: **N/A** — static site. There is no
  Flutter code, so no Dart tests, AppLogger, Riverpod, etc. Their absence is intended.
- **Path**: `/tarmaclog/` (folder with `index.html`), so future apps can live at `/<app>/`.
- **Root page**: studio page, not a redirect. Keeps `copilotlabs.dev` meaningful as the brand.
- **Legal/support pages stay at the root** — their URLs are registered in App Store Connect, Play
  Console, and shipped app builds (`lib/src/core/config/legal_urls.dart`). Do not move them, do
  not change their body text (legal). Allowed changes to them: shared nav/footer markup, the new
  palette/fonts via `style.css`, and adding OG/Twitter meta tags.
- **Palette**: replace the GitHub-style values in `style.css` with the app's dark brand tokens.
  Keep the existing variable names so current pages keep working, and add new ones:

  | variable | new value | source token |
  |---|---|---|
  | `--bg` | `#08101A` | `bg` |
  | `--surface` | `#151C25` | `surface` |
  | `--surface-2` (new) | `#252C35` | `surface2` |
  | `--border` | `#313942` | `border` |
  | `--text` | `#EDF2F9` | `ink` |
  | `--text-muted` | `#A3ACB8` | `inkMuted` |
  | `--text-faint` (new) | `#676F7A` | `inkFaint` |
  | `--accent` | `#F2A618` | `amber` (primary — buttons, highlights) |
  | `--accent-ink` (new) | `#190F03` | `amberInk` (text on amber) |
  | `--accent-hover` | `#F5B840` | lighter amber for hover |
  | `--link` (new) | `#47C5D2` | `cyan` (links) |

  Links use `--link`; buttons/highlights use `--accent`. Never hardcode colours in HTML.
- **Fonts**: self-host IBM Plex Sans (400, 600, 700) and IBM Plex Mono (500) as `.woff2` in
  `website/assets/fonts/`, converted from `assets/fonts/*.ttf` in the root repo
  (`pyftsubset <ttf> --flavor=woff2 --unicodes="U+0000-00FF,U+2013-2014,U+2018-201D,U+2022,U+2026,U+2192,U+2708" --output-file=…`;
  if fonttools is not installed use `pip3 install --user fonttools brotli`). `@font-face` with
  `font-display: swap`. Body uses Plex Sans; numbers/figures (e.g. "3 flights") may use Plex Mono.
  IBM Plex is SIL OFL — include `website/assets/fonts/OFL.txt`.
- **Images** (all under `website/assets/`):
  - `img/app-icon.png` — 512×512 from `assets/images/app_icon.png` (also used as OG image fallback
    and `apple-touch-icon` at 180×180 → `img/apple-touch-icon.png`).
  - `img/screens/0N_<name>.webp` — from `docs/marketing/store-screenshots/play-phone/*.png`
    (1080×1920), resized to 540 wide, `cwebp -q 80`. Target < 80 KB each. Include `width`/`height`
    attributes, `loading="lazy"` on all but the first, `decoding="async"`.
  - `img/og-image.png` — 1200×630: use `docs/marketing/store-screenshots/play-feature-graphic/feature-graphic-1024x500.png`
    scaled/padded onto `#08101A` with `magick`.
  - Store badges: official artwork, saved locally:
    - `img/badges/app-store.svg` — Apple "Download on the App Store" black badge (US English) from
      Apple's marketing tools (`https://tools.applemarketingtools.com/api/badges/download-on-the-app-store/black/en-us`).
    - `img/badges/google-play.png` — Google Play "Get it on Google Play" badge (English) from
      `https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png`.
    - Do not recolour or alter badges. Render both at the same visual height (~48px); the
      Google PNG has built-in padding, so size it taller (~70px) to match. Add the required legal
      line in the footer of `/tarmaclog/`: "App Store and the Apple logo are trademarks of Apple Inc.
      Google Play and the Google Play logo are trademarks of Google LLC."
- **Nav** (all pages): brand `Co-Pilot Labs` → `/`; links: `Tarmac Log` (`/tarmaclog/`), `Support`,
  `Privacy`, `Terms`. On `/tarmaclog/`, the brand area may show the app icon + "Tarmac Log".
- **Favicon**: keep `favicon.svg`.

## 3. Content — `/tarmaclog/` (copy source: `docs/marketing/app-store-copy.md`)

Only use claims already in the approved store copy. **Never** claim encryption at rest, FAA/EASA
"compliance/certification", iPad support, or anything not in the store copy. The current root
page's "FAA and EASA compliant" claim must not be carried over.

Sections, in order (semantic HTML: `<header>`, `<main>`, `<section aria-labelledby>`, `<footer>`):
1. **Hero** — `<h1>Tarmac Log — the pilot logbook that works without a signal</h1>`, one-sentence
   lead from the Promotional Text, the two store badges, app icon or first screenshot.
2. **Features** (`<h2>`), 4–6 cards, each an `<h3>` + 1–2 sentences, from the description:
   offline-first; fast flight entry; fleet with low-poly aircraft icons; SACAA-format PDF export;
   totals and currency; syncs privately to your account across devices.
3. **Screenshots** (`<h2>See it in action</h2>`) — horizontal scroll-snap row on mobile, grid on
   desktop; `<figure>` + `<figcaption>` per screen.
4. **Free and Pro** (`<h2>`) — two cards, no prices, note "Pro is an optional subscription,
   purchased in the app."
5. **FAQ** (`<h2>`) — `<details><summary>` items. Suggested questions: Does it work offline? /
   Which phones does it run on? (iPhone and Android) / Can I export my logbook? (SACAA-format A4
   PDF, Pro) / Is it free? / Where is my data stored? (on your device first, synced privately to
   your account) / How do I delete my account? (link `/delete-account`).
6. **Closing CTA** — "Download Tarmac Log" `<h2>` + both badges.
7. **Footer** — standard footer + trademark line.

## 4. SEO and AI files

- **`<head>` of `/tarmaclog/`**: title `Tarmac Log — Offline Pilot Logbook for iPhone and Android`;
  description (≤160 chars) based on the promo text; canonical `https://copilotlabs.dev/tarmaclog/`;
  `og:type=website`, `og:title`, `og:description`, `og:url`, `og:image=https://copilotlabs.dev/assets/img/og-image.png`,
  `og:site_name=Co-Pilot Labs`; `twitter:card=summary_large_image`;
  `<meta name="apple-itunes-app" content="app-id=6756934482">` (Safari smart banner);
  `theme-color #08101A`.
- **JSON-LD** (`<script type="application/ld+json">`) on `/tarmaclog/`:
  - `MobileApplication`: name, description, `operatingSystem: "iOS, Android"`,
    `applicationCategory: "TravelApplication"`, `url`, `image`, `screenshot` (absolute URLs),
    `offers: {"@type":"Offer","price":"0","priceCurrency":"USD"}` (free download),
    `downloadUrl`/`installUrl` = both store URLs, `publisher: {"@type":"Organization","name":"Co-Pilot Labs","url":"https://copilotlabs.dev/"}`.
    **No** `aggregateRating` (we have no verified rating data).
  - `FAQPage` mirroring the visible FAQ text exactly.
- **JSON-LD on `/`**: `Organization` (name, url, logo) and `WebSite`.
- **`robots.txt`**: allow all (including AI crawlers), `Disallow: /auth/`, `Sitemap: https://copilotlabs.dev/sitemap.xml`.
- **`sitemap.xml`**: `/`, `/tarmaclog/`, `/support/`, `/privacy-policy/`, `/terms/`,
  `/delete-account/` with `<lastmod>2026-09-26</lastmod>`. Exclude `/auth/callback/`.
- **`llms.txt`** (llmstxt.org format): `# Co-Pilot Labs`, one-line blockquote summary, then a
  `## Tarmac Log` section: plain-text summary (what it is, platforms, offline-first, Free vs Pro
  without prices, SACAA PDF export), store URLs, and links to the marketing, support, privacy and
  terms pages.
- Add `<meta name="robots" content="noindex">` to `auth/callback/index.html` **only if** it has no
  such tag — this is the single permitted change to that file. Do not touch its script.

## 5. Files

**Create** (website repo):
- `tarmaclog/index.html`
- `robots.txt`, `sitemap.xml`, `llms.txt`
- `assets/fonts/*.woff2`, `assets/fonts/OFL.txt`
- `assets/img/app-icon.png`, `assets/img/apple-touch-icon.png`, `assets/img/og-image.png`
- `assets/img/screens/01_offline.webp` … `06_export.webp`
- `assets/img/badges/app-store.svg`, `assets/img/badges/google-play.png`

**Modify** (website repo):
- `index.html` — rewrite as the Co-Pilot Labs studio page.
- `style.css` — brand tokens, `@font-face`, new components (hero, badge row, screenshot strip,
  plan cards, FAQ). Keep existing classes working for the legal/support pages.
- `support/`, `privacy-policy/`, `terms/`, `delete-account/` `index.html` — nav/footer update +
  OG/Twitter meta only. Body text unchanged.

- `_docs/README.md` (create) — short maintainer notes for the site: hosted at
  `https://copilotlabs.dev/`, structure, palette table, nav/footer pattern, and "Marketing pages"
  rules: claims must come from the approved store copy; no third-party requests; no personal info;
  how to switch/refresh store badges and screenshots.

**Everything lives in the website repo.** Do **not** modify any file in the root LogBook repo —
not the Flutter app, not `.cursor/rules/`, not `docs/`. Root-repo files (`assets/`,
`docs/marketing/`) are **read-only sources** to copy/convert from.

**`_docs/` is not published**: the site is built by GitHub Pages' Jekyll (legacy) build, which
skips `_`-prefixed folders. Do **not** add a `.nojekyll` file, or `_docs/` would go public.

**Do not touch**: `CNAME`, the `auth/callback` script.

## 6. Verification (replaces Flutter TDD for this static feature)

No test framework exists for the website and none should be added. DEV must instead:
1. Write a throwaway Python check script **in the scratchpad, not the repo** that parses every
   public `index.html` and asserts: one `<h1>`; title + description + canonical (on
   `copilotlabs.dev`) + `og:image` present; every `<img>` has `alt` and width/height; all internal
   `href`/`src` targets exist on disk; every JSON-LD block parses as JSON; no `http(s)://` resource
   (`src`, stylesheet `href`, `@import`, `url()`) points off-site; no hardcoded hex colour outside
   `:root` in `style.css`; no personal-info patterns (email other than the support group, phone
   numbers). Run it — all pass.
2. Validate `sitemap.xml` parses as XML; every URL in it maps to an existing file.
3. Serve locally (`python3 -m http.server` from `website/`) and `curl` each sitemap URL → 200.
4. Report total transfer weight of `/tarmaclog/` (HTML + CSS + fonts + images) — target < 1.2 MB.
5. If a headless browser is available (e.g. Python Playwright), capture `/tarmaclog/` at 390px and
   1280px wide into the scratchpad and check there's no horizontal overflow. If not available, say so.

## 7. Deploy (orchestrator, after developer approval)

Commit in `website/` on the feature branch → PR in `Co-Pilot-Labs/co-pilot-labs.github.io` →
merge deploys. The root repo's submodule pointer is **not** bumped as part of this feature
(developer: keep all work in the website repo); it can be bumped later with an app change.

After deploy, verify `_docs/` is **not** served: `curl -I https://copilotlabs.dev/_docs/specs/tarmaclog-marketing-site.md` → 404.

## 8. Rule files to read first
- `.cursor/rules/website/RULE.md`
- `.cursor/rules/design-system/RULE.md` (palette source)
- `docs/marketing/app-store-copy.md` (approved claims)
