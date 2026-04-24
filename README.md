# pankkonen.dev

Personal CV and portfolio site for Joona Pankkonen — IT Engineering student focused on data analytics and AI, based in Jyväskylä, Finland.

The reference aesthetic is Revolut / Vercel / Linear: deep navy, editorial serif headings, spring-easing scroll reveals, one amber accent used sparingly. Full design contract lives in `../DESIGN_SPEC.md`.

## Stack

- **Framework:** Astro (static HTML at build time, component-based)
- **Styles:** Tailwind CSS + handwritten `global.css` for tokens and motion
- **Animation:** GSAP for the hero load timeline; `IntersectionObserver` + CSS keyframes for scroll reveals
- **Hosting:** Cloudflare Pages (auto-deploy on push to `main`)
- **Domain:** `pankkonen.dev` (purchase deferred; development URL `pankkonen.pages.dev`)

## Build progress

| Phase | Area | Status |
|-------|------|--------|
| 1 | Scaffolding — Astro + Tailwind + GSAP | Complete |
| 2 | Design system — tokens, fonts, layout primitives | Complete |
| 3 | Hero — name, tagline, photo, load stagger | Complete |
| 4 | Experience & Stack sections | Complete |
| 5 | Project cards | Complete |
| 5b | Divider redesign (oval amber aura + edge-faded line + shimmer) and IO flicker debounce | Complete |
| 6 | About & Beyond the Code | Pending |
| 7 | Contact | Pending |
| 8 | Polish & accessibility | Pending |
| 9 | Cloudflare Pages deploy | Pending |
| 10 | Final content & domain switch | Pending |

## Commands

```sh
npm install      # install dependencies
npm run dev      # local dev server on http://localhost:4321
npm run build    # production build to ./dist
npm run preview  # preview the production build
```

## Repo layout

```
src/
├── layouts/Layout.astro        # html shell, global CSS import
├── pages/index.astro           # page sections, scroll-reveal IO
├── components/                 # Experience, Stack, Projects
└── styles/global.css           # design tokens, motion, section divider
public/                         # static assets (photo, favicon)
docs/superpowers/               # locked specs and execution plans
```

## Design guarantees

- Dark only; no light-mode toggle in v1
- Motion uses spring easing `cubic-bezier(0.16, 1, 0.3, 1)` with 28px lift at 750ms
- Section dividers replay on every scroll pass (both directions)
- Reduced-motion users get revealed content with no transitions

Links: [GitHub](https://github.com/joonapankkone/pankkonen-dev)
