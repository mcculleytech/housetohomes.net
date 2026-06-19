# CLAUDE.md

Guidance for working in this repo. Read before making changes.

## What this is

Marketing site for **House to Home Solutions**, a handyman business in Sioux
Falls, SD. Built with **Hugo** (static site generator) + **Tailwind CSS v4**.
**No client-side JavaScript** — keep it that way unless there's a strong reason.

## Local development

```bash
npm install
npm run dev      # builds CSS, then runs `hugo server` with live reload
npm run build    # production build: css:build + `hugo --minify` → ./public
```

- Tailwind input: `assets/css/input.css` → output: `static/css/output.css`.
- **`static/css/output.css` is committed.** Cloudflare rebuilds it on deploy
  (`npm run build`), but rebuild and commit it locally too if you change styles
  so local previews and the repo stay in sync.
- Custom theme tokens (colors `purple`/`tan`/`ink`, fonts Fraunces + Inter) live
  in `assets/css/input.css`.

## Project structure

```
content/            Markdown pages: _index (home), about, contact, estimate, thanks
data/               services.yaml, testimonials.yaml — pulled into the homepage
layouts/
  _default/         Page templates: baseof, about, contact, estimate, thanks, single
  index.html        Homepage template
  404.html
  partials/         head, nav, banner (top contact bar), footer
static/             images/ and the built css/output.css
hugo.toml           Site config, params, baseURL
```

Business details (phone, email, social links, service area, Web3Forms key, OG
image) are all params in `hugo.toml` — **reference `site.Params.*` in templates,
don't hardcode** these values.

## Dev vs. Production (Cloudflare Pages)

Single Cloudflare Pages project: **`housetohomes-net`**. Auto-deploys on push.

| Branch | CF environment | URL | Effective baseURL |
|--------|---------------|-----|-------------------|
| `main` | Production | `test.housetohomes.net` (+ `housetohomes-net.pages.dev`); `housetohomes.net` at launch | `https://housetohomes.net/` (from `hugo.toml`) |
| `dev`  | Preview | `https://dev.housetohomes-net.pages.dev` | `https://dev.housetohomes-net.pages.dev/` (from `HUGO_BASEURL` Preview var) |

**`baseURL` strategy:** `hugo.toml` holds the production value on *both* branches.
The Preview environment sets `HUGO_BASEURL=https://dev.housetohomes-net.pages.dev/`,
which Hugo maps onto `baseURL` — so only preview builds use the dev URL, and
`main`/`dev` stay identical in git (no per-branch config to merge). Pinned build
vars (`HUGO_VERSION=0.163.2`, `NODE_VERSION=20`) are set on both environments;
`.nvmrc` pins Node locally.

- **Production branch must stay `main`** in CF Pages settings. (If a `dev` push
  ever shows up tagged "Production" on `test.housetohomes.net`, the production
  branch was switched to `dev` by mistake — flip it back to `main`.)
- Cloudflare Pages can only bind a custom domain to the *production* branch, so
  the `dev` branch uses its **branch alias** `dev.housetohomes-net.pages.dev` as
  its stable staging URL. Don't try to attach a custom subdomain to a preview
  branch (it 522s).
- Build settings: command `npm install && npm run build`, output `public`,
  production branch `main`. Build env pinned via the Cloudflare vars above.
- `test.housetohomes.net` is the current pre-prod domain; it will be retired
  after the `housetohomes.net` cutover.

### DEV banner

`layouts/_default/baseof.html` renders an amber "DEV ENVIRONMENT" bar under the
contact banner. It's gated on the hostname:

```go-html-template
{{- if ne (urls.Parse site.BaseURL).Hostname "housetohomes.net" -}} ... {{- end -}}
```

So it shows on dev/staging and `localhost`, but **never** on the live
`housetohomes.net` domain — even if `dev` is merged to `main`.

### Branch workflow

- `dev` is for staging/testing; `main` is what ships to production.
- Both branches are protected on GitHub (no force-push, no deletion); direct
  pushes are allowed.
- `dev` and `main` hold identical config (`baseURL` is the prod value on both;
  the dev URL comes from the `HUGO_BASEURL` Preview var), so `dev` → `main` merges
  cleanly with no baseURL conflict.

## Forms (Web3Forms)

The estimate form (`layouts/_default/estimate.html`) submits to
**Web3Forms** — a serverless form-to-email service. No backend of our own.

- Action: `POST https://api.web3forms.com/submit`
- Access key: `site.Params.web3FormsKey` (in `hugo.toml`).
- Submissions are emailed to **housetohomes.solutions@gmail.com** (configured in
  the Web3Forms dashboard, keyed to the access key — verify there if recipients
  change).
- After submit, redirects to `/thanks/` (`content/thanks.md`, which is
  `robotsNoIndex`).
- Spam protection: hidden `botcheck` honeypot field. No CAPTCHA, no JS.

The access key is a public client-side key (safe to commit). To rotate it,
update `hugo.toml` and redeploy.

## Analytics

**Cloudflare Web Analytics**, enabled via automatic edge injection in the
Cloudflare dashboard for the `housetohomes.net` host only — no snippet in the
repo, and dev/staging traffic is not tracked.

## SEO

`layouts/partials/head.html` emits canonical URL, Open Graph, and Twitter card
tags (OG image: `site.Params.ogImage`). `enableRobotsTXT` is on and Hugo
generates `sitemap.xml` — both depend on a correct `baseURL`.

`layouts/partials/schema.html` (included from `head.html`) emits a
`HomeAndConstructionBusiness` JSON-LD block built from `site.Params`. Note: it
pipes through **`safeJS`** — inside a `<script>` tag Go's `html/template` applies
JS-context escaping and will re-encode `jsonify` output as a quoted string
without it. To add fields, edit the `dict` in that partial (don't hardcode).
