# housetohomes.net

The marketing site for **House to Home Solutions**, a Sioux Falls–based handyman business run by Charlie Tibbetts.

Built with [Hugo](https://gohugo.io/) + [Tailwind CSS v4](https://tailwindcss.com/), deployed via [Cloudflare Pages](https://pages.cloudflare.com/). Estimate-request form is handled by [Web3Forms](https://web3forms.com/). Zero JavaScript shipped to the browser.

## Predecessor

This repo is a complete rewrite of the previous PHP site that lived at [`mcculleytech/HouseToHomes_Website`](https://github.com/mcculleytech/HouseToHomes_Website). It shares no git history with that repo — the new site is a clean rebuild with different architecture, content baked in, and a static-first deployment model. The old site is preserved at the original URL during cutover.

## Stack

- **Hugo** — static site generator
- **Tailwind CSS v4** — utility-first CSS, built via `@tailwindcss/cli`
- **Web3Forms** — form-to-email relay for the estimate request page
- **Cloudflare Pages** — git-connected build + global CDN + free TLS

No client-side JavaScript. Native HTML elements (`<details>`, `<fieldset>`) cover the few interactive bits.

## Local development

```bash
npm install
npm run dev
```

This runs `tailwindcss --watch` and `hugo server`. Open <http://localhost:1313>.

To produce a production build:

```bash
npm run build
```

Output lands in `public/`.

## Structure

```
content/         page-level frontmatter (each .md selects a layout)
data/            home page services + testimonials (YAML)
layouts/
  _default/      page templates (about, contact, estimate, thanks, single)
  partials/      head, nav, banner, footer
  index.html     homepage template
assets/css/      Tailwind input.css (theme tokens + custom utilities)
static/          images + compiled CSS (output.css)
hugo.toml        site config + business params (phone, email, social, Web3Forms key)
```

To edit copy, edit the markdown frontmatter or YAML data files — no template changes needed.

## Deployment (Cloudflare Pages)

Build configuration:

| Setting | Value |
| --- | --- |
| Build command | `npm install && npm run build` |
| Build output | `public` |
| Production branch | `main` |

Pinned build environment (Cloudflare → Variables and secrets, Production **and** Preview):

| Variable | Value |
| --- | --- |
| `HUGO_VERSION` | `0.163.2` |
| `NODE_VERSION` | `20` |
| `HUGO_BASEURL` | `https://dev.housetohomes-net.pages.dev/` — **Preview only** |

`baseURL` in `hugo.toml` is the production value (`https://housetohomes.net/`). The
`dev` branch deploys as a Preview, where `HUGO_BASEURL` overrides it so staging
links resolve to the dev alias. This keeps `main` and `dev` identical in git (no
per-branch config divergence). Local matches via `.nvmrc` (Node) + Homebrew Hugo.

Custom domains: `test.housetohomes.net` (pre-prod, retired after launch) and
`housetohomes.net` (production). The `dev` branch's staging URL is the Pages
branch alias `dev.housetohomes-net.pages.dev`.

## Business contact info

Centralized in `[params]` in `hugo.toml`:

- Phone: (605) 400-4440
- Email: housetohomes.solutions@gmail.com
- Facebook, Instagram, Google reviews — all linked

## Form submissions

Estimate requests POST to Web3Forms (key in `hugo.toml`) and deliver to the configured destination email. The form includes a honeypot field (`botcheck`) for spam filtering. To rotate the key, regenerate at [web3forms.com](https://web3forms.com/) and update `hugo.toml`.
