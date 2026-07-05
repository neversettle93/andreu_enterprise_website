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

## Before going live

- `src/components/ContactForm.astro` posts to a placeholder Formspree
  endpoint (`https://formspree.io/f/YOUR_FORM_ID`). Replace it with a real
  Formspree or Web3Forms endpoint before launch. The form already includes a
  honeypot field (`_gotcha`) for basic spam protection; once the real
  endpoint is set, consider also adding a `_next` hidden input pointing to a
  thank-you page on the live domain so submitters stay on-site.
- No `site` is set in `astro.config.mjs` yet, so Open Graph tags render
  without a canonical URL. Add `site: "https://yourdomain.com"` once the
  domain is final.
- Deploy target is Cloudflare Pages (static output, no adapter needed).
