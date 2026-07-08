# Andreu Enterprise Technology — Website

Marketing site for Andreu Enterprise Technology: services, about, case
studies, and a contact form. Built with [Astro](https://astro.build) and
Tailwind CSS for a fast, static, animation-free site. See `CLAUDE.md` for the
full stack rationale and design constraints.

## Project structure

```text
/
├── public/                  # static assets (favicon, etc.)
├── src/
│   ├── components/          # Nav, Footer, ServiceCard, CaseStudyCard, ContactForm,
│   │                        # Eyebrow, GridSection, BulletList, Button
│   ├── layouts/BaseLayout.astro
│   ├── pages/                # index, services, about, case-studies, contact
│   └── styles/global.css     # Tailwind entrypoint + navy/cobalt/cyan design tokens
└── package.json
```

## Commands

| Command | Action |
| :--- | :--- |
| `npm install` | Install dependencies |
| `npm run dev` | Start local dev server at `localhost:4321` |
| `npm run build` | Build production site to `./dist/` |
| `npm run preview` | Preview the production build locally |
| `npx astro check` | Type-check all `.astro` files |
| `npm run astro ...` | Run other Astro CLI commands (e.g. `astro add`) |

## Deployment

- Hosted on **Cloudflare Pages** (static output, no adapter needed), with
  Git integration: pushes to `main` deploy to production, other branches/PRs
  get automatic preview URLs.
- Production domain: `https://andreuenterprise.com`, set as `site` in
  `astro.config.mjs`. `BaseLayout.astro` builds canonical/OG/Twitter image
  URLs from `Astro.site`, so they stay correct if the domain ever changes.
- `src/components/ContactForm.astro` posts to a live Formspree endpoint and
  redirects to `/thank-you` on success via a `_next` hidden field. Includes a
  honeypot field (`_gotcha`) for basic spam protection.
- `public/_headers` sets baseline security headers (Cloudflare Pages reads
  this file automatically, no build step involved).
- `@astrojs/sitemap` generates `sitemap-index.xml` on every build;
  `public/robots.txt` points to it.
