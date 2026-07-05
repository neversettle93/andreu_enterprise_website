# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Marketing/portfolio website for **Andreu Enterprise Technology** — a RevOps and AI-automation consultancy for B2B companies. The site is informational (services, about, case studies, contact form), with no authenticated app surface. Built with [Astro](https://astro.build) chosen specifically for near-zero client-side JavaScript, since the site must be fast, modern-looking, and free of animations/heavy client JS. Styling uses Tailwind CSS. See `.claude/plans/` history for the full stack rationale (Astro over Next.js, Cloudflare Pages over Vercel/Netlify, Formspree/Web3Forms for the contact form instead of a custom backend).

## Commands

| Command | Action |
| :--- | :--- |
| `npm install` | Install dependencies |
| `npm run dev` | Start local dev server at `localhost:4321` |
| `npm run build` | Build production site to `./dist/` |
| `npm run preview` | Preview the production build locally |
| `npx astro check` | Type-check all `.astro` files (0 errors expected) |
| `npm run astro ...` | Run other Astro CLI commands (e.g. `astro add`) |

When starting the dev server for testing, prefer background mode: `astro dev --background` (manage with `astro dev stop`, `astro dev status`, `astro dev logs`).

## Architecture

- `src/pages/*.astro` — one file per route (file-based routing): `index`, `services`, `about`, `case-studies`, `contact`.
- `src/components/` — reusable `.astro` components:
  - `Nav`, `Footer` — shared chrome.
  - `ServiceCard`, `CaseStudyCard` — content cards for the services/case-studies grids.
  - `ContactForm` — the contact form (posts to Formspree, see "Before going live" in README).
  - `Eyebrow`, `GridSection`, `BulletList`, `Button` — small layout primitives extracted to avoid duplicating the same Tailwind class strings across the 5 pages. `GridSection` wraps the light/dark grid-textured section pattern (`variant`, `border`, `maxWidth` props) used at the top of every page and in most dark bands — reuse it for any new section with that look instead of hand-rolling the `bg-grid-light`/`bg-grid-dark` overlay again.
- `src/layouts/BaseLayout.astro` — shared `<head>` (incl. OG/Twitter meta)/nav/footer wrapper used by every page.
- `src/styles/global.css` — Tailwind entrypoint + design tokens (navy/cobalt/cyan palette, `--font-mono`, `.bg-grid-light`/`.bg-grid-dark` textures). The three brand colors (`navy-900` `#0a1628`, `cobalt-600` `#1b4fd8`, `cyan-400` `#00c2d4`) are pinned to the exact hex values in `brand-assets/` — don't drift them without checking the kit.
- `public/` — static assets served as-is: favicon set + `site.webmanifest`, `og-image.png` (1200×630 social share image), `brand/icon-color.svg` (square mark used next to the wordmark in `Nav`/`Footer`).
- `brand-assets/` — source brand kit (favicon exports, PNG icon/lockup/social variants, SVG originals) the client provided; `public/` only contains the subset actually referenced by the site. Pull additional variants from here (e.g. `AET-lockup-horizontal-*.svg`, dark/white/reversed icon variants) if a new section needs one — the full horizontal lockup's tagline becomes illegible below ~90px tall, so prefer the icon-only mark for anything nav/footer-sized.

## Design constraints (do not violate without asking)

- **No animations or transition libraries.** This was an explicit client requirement to keep the site light. Don't add `client:*` Astro islands, React/Vue components, or JS animation libs unless the user asks — the zero-JS-by-default behavior of Astro is intentional here, not an oversight.
- **Case studies are anonymized.** Do not name specific client companies (e.g. the founder's employer) in copy — describe them by industry/type instead (see `src/pages/case-studies.astro` for the established pattern).
- **Never invent a specific case-study outcome (numbers, incidents, anecdotes).** The original "Data Integrity" case study card described a specific royalty-calculation incident that turned out not to have happened that way — it was replaced with a generic, unquantified capability description ("Validating the numbers before they reach leadership") instead of a fabricated replacement incident. If a case study needs a concrete stat/outcome, it must come from the client directly; when in doubt, write it as a capability statement with no invented numbers rather than guessing.
- Keep the palette to navy/cobalt/cyan and use color for emphasis (CTAs, accents) rather than decoration, consistent with the Linear/Stripe/Vercel-inspired bold-typography direction (large type scale, dense grid textures, mono numerals for stats/indices — see `src/pages/index.astro` stats band and `ServiceCard`/`CaseStudyCard` for the established pattern) layered on the original Apple.com-inspired minimalism.
- **Text on white/light backgrounds must be `text-navy-600` or darker** (`navy-400` fails WCAG AA contrast at 4.21:1 on white — confirmed by code review). `navy-400` is only safe on dark (`bg-navy-950`) backgrounds.
- New pages/sections should keep heading levels sequential (H1 → H2 → H3, no skipping) even when a section's H2 is purely visual scaffolding — use `class="sr-only"` on it rather than omitting it (see `case-studies.astro`).
- **"Tools in daily use" (`about.astro`) is about the founder's own personal daily workflow, not the services the company offers.** These can diverge: e.g. n8n is offered to clients under "AI & Automation Consulting" (`services.astro`) but isn't a tool the founder personally uses day to day, so it was removed from the About page's tools list (replaced with Claude Code) while staying in Services. Don't assume a tool mentioned in one context belongs in the other without checking.
- **`about.astro`'s "Career path" timeline is ordered most-recent-first** (current role at the top, oldest at the bottom) — this was an explicit request, the opposite of the usual chronological-forward resume convention. Keep new entries prepended, not appended.

## Astro documentation references

Consult before working on related tasks:

- [Routing, dynamic routes, middleware](https://docs.astro.build/en/guides/routing/)
- [Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Framework components (React/Vue/Svelte)](https://docs.astro.build/en/guides/framework-components/) — not currently used; only add if a genuinely interactive feature is requested
- [Content collections](https://docs.astro.build/en/guides/content-collections/)
- [Styling / Tailwind](https://docs.astro.build/en/guides/styling/)

## Custom subagents

Configured in `.claude/agents/`:

- **`code-reviewer`** — Zero-context code reviewer. Invoke for explicit review requests ("revisa este código", "review this diff"). Reviews fresh against the actual code/git diff, reports blocking/important/minor findings, never edits files.
- **`research-agent`** — Research and fact-verification agent. Invoke proactively before decisions that need current external information. Returns a structured report, never edits files.
- **`qa-agent`** — QA engineer with persistent memory (`qa/known-issues.md`). Invoke to test code end-to-end and hunt for bugs, not just review it statically. Maintains a defect log across sessions and writes regression tests; only writes to the QA log and test files, never to application code.
- **`github-devops-agent`** — Owns the Git/GitHub lifecycle (init, `.gitignore` hygiene, secret scanning, commits, push). Invoke for "sube esto a GitHub", "haz commit y push", "conecta este proyecto a GitHub". Never pushes (and never force-pushes) without a separate, explicit confirmation for each; never creates the remote repo itself (the user creates it manually and gives the URL); the only code file it may write is `.gitignore`.

All four are report-only or narrowly-scoped writers — none of them edit application source code themselves.

**Environment note:** in this VSCode-extension/FleetView-based session, `Agent` calls with `subagent_type` set to one of these four custom names fail with "Agent type not found" — only `claude`, `claude-code-guide`, `Explore`, `general-purpose`, `Plan`, `statusline-setup` are recognized here, regardless of what's defined in `.claude/agents/`. This differs from the standard Claude Code CLI, which auto-discovers project-level agents. The working workaround used in this project so far: read the target `.md` file's full contents and pass it as the prompt to a `general-purpose` agent (or, for the GitHub workflow specifically, follow it directly in the main conversation instead of delegating, since it requires pausing for real user confirmation before `push`/`force-push` — something a one-shot subagent call can't reliably do).

## Git / GitHub status

As of the last update to this file, this project directory is **not yet a git repository** (no `.git`, no commits, no remote). Setting it up is expected to follow the `github-devops-agent` workflow above: `git init` → `.gitignore` check → the user provides an already-created GitHub repo URL (the agent doesn't create the remote) → secret scan → commit → push only after explicit confirmation.
