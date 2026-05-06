# pankkonen.dev

Personal CV and portfolio site for Joona Pankkonen — IT Engineering student focused on data analytics and AI, based in Jyväskylä, Finland.

The reference aesthetic is a blend of modern fintech and editorial sites: deep navy, editorial serif headings, spring-easing scroll reveals, one amber accent used sparingly. Full design contract lives in `../DESIGN_SPEC.md`.

## Stack

- **Framework:** Astro (static HTML at build time, component-based)
- **Styles:** Tailwind CSS + handwritten `global.css` for tokens and motion
- **Animation:** CSS-only keyframes for the hero load stagger (`heroStaggerReveal` + `heroImageReveal`); `IntersectionObserver` + CSS keyframes for scroll reveals — no animation library at runtime
- **Hosting:** Cloudflare Workers Assets (auto-deploy on push to `main`)
- **Domain:** `pankkonen.dev` — live

## Build progress

| Phase | Area | Status |
|-------|------|--------|
| 1 | Scaffolding — Astro + Tailwind + CSS animations | Complete |
| 2 | Design system — tokens, fonts, layout primitives | Complete |
| 3 | Hero — name, tagline, photo, load stagger | Complete |
| 4 | Experience & Stack sections (4 categories incl. AI Workflow) | Complete |
| 5 | Project cards | Complete |
| 5b | Divider redesign (oval amber aura + edge-faded line + shimmer) and IO flicker debounce | Complete |
| 6 | About & Beyond the Code (initial: 50/50 grid + portrait clip) | Complete |
| 6b | Beyond redesign — full-bleed 16:9 video stage, copy refresh, EAWrc clip swap | Complete (2026-05-01) |
| 6c | Polish — chapter rail, divider v3 (StackDivider + ProjectsDivider), Projects card count-up, About-konsolidointi | Complete (2026-05-02) |
| 7 | Contact | Complete |
| 8 | Polish & accessibility (Lighthouse 100/99) | Complete (2026-05-05) |
| 9 | Cloudflare Workers Assets deploy (wrangler.toml, auto-deploy on push) | Complete (2026-05-06) |
| 10 | Domain switch — `pankkonen.dev` live, v1.0 published | Complete (2026-05-06) |

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
├── layouts/Layout.astro        # html shell, global CSS import, fonts, meta
├── pages/index.astro           # page composition, hero, scroll-reveal IO
├── components/
│   ├── ChapterMarker.astro     # right-rail wayfinder (00–05)
│   ├── Experience.astro        # two-track work | study timeline
│   ├── Stack.astro             # 4-column tech index
│   ├── Projects.astro          # 3 cards + expanding panel + stat count-up
│   ├── About.astro             # legacy filename — renders Beyond the Code only
│   ├── Contact.astro           # email, GitHub, LinkedIn
│   └── *Divider.astro          # 5 section transitions (sweep + editorial)
└── styles/global.css           # design tokens, motion, section divider
public/                         # joona.{avif,webp,jpeg}, btc-sim.mp4 + poster
docs/superpowers/               # locked specs and execution plans
wrangler.toml                   # Cloudflare Workers Assets config (assets.directory = dist)
.node-version                   # pins Node 22 for CF build environment
```

## Design guarantees

- Dark only; no light-mode toggle in v1
- Scroll-reveal motion uses spring easing `cubic-bezier(0.16, 1, 0.3, 1)` with 28px lift at 750ms; hover and panel transitions use the same curve
- Hero entrance animations are CSS-only (`@keyframes heroStaggerReveal` + `heroImageReveal`); no animation library at runtime. Hero uses `cubic-bezier(0.215, 0.61, 0.355, 1)` (power2.out) at 600–700ms — intentionally snappier than the scroll-reveal spring
- Section dividers replay on every scroll pass (both directions)
- Reduced-motion users get revealed content with no transitions; ambient background videos pause and show their poster frame
- Beyond the Code video plays only while the section is in the viewport (IO-gated); muted, looped, no controls — pure ambient element
- Beyond stage caps width at `min(100vw, 85vh × 16/9)` so the 16:9 sim racing clip is always shown in full at every viewport size, with the page background showing on the sides on screens wider than the cap (Linear / Vercel hero pattern)

## Performance

Lighthouse scores from `npm run preview` build (Chrome DevTools, 2026-05-05):

| Category | Desktop | Mobile |
|----------|---------|--------|
| Performance | **100** | **99** |
| Accessibility | 100 | 100 |
| Best Practices | 100 | 100 |
| SEO | 100 | 100 |

Desktop core web vitals: FCP 0.2s · LCP 0.5s · TBT 0ms · CLS 0.
Mobile core web vitals: FCP 0.8s · LCP 2.1s · TBT 0ms · CLS 0.

Notable optimisations:
- Hero photo served as AVIF (29 KB) with WebP and JPEG fallbacks via `<picture>`
- AVIF preload hint in `<head>` shaves ~0.4s off mobile LCP
- Google Fonts CSS loaded non-blocking (`media="print" onload="this.media='all'"`)
- No JavaScript animation library — hero stagger is CSS keyframes only

Re-run with: `npm run build && npm run preview`, then Chrome DevTools → Lighthouse. Keep the tab in the foreground throughout the audit.

Links: [Live site](https://pankkonen.dev) · [GitHub](https://github.com/joonapankkone/pankkonen-dev)
