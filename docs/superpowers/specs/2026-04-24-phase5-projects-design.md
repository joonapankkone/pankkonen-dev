# Phase 5 — Projects Section Design Spec
**Date:** 2026-04-24
**Status:** LOCKED

---

## Overview

A Projects section (`03 · Projects`) featuring three cards in a horizontal grid. Clicking a card opens a full-width detail panel below the grid — the Apple product grid pattern. Cards stay visible while the panel is open; clicking a different card swaps the panel content with a crossfade. Clicking the active card again collapses the panel.

---

## Section Header

**Index label:** `03 · Projects` (follows 01 · Experience, 02 · Stack)

**Heading:**
```
Work that ships.
```

**Tagline (italic accent span):**
```
Real data, agile teams, hard tradeoffs.
```

**Lede paragraph:**
```
From 2.6M-row ML pipelines to a 6-person Agile team — each project here
has a GitHub repo, a story, and something honest to say about what didn't
go perfectly.
```

Header follows the established stagger pattern: section-index → data-reveal, heading → data-reveal-delay="1", lede → data-reveal-delay="2".

---

## Projects — Content

### Card 1 — Used Car Price Prediction (headline)
- **Category badge:** `Data & ML`
- **Title:** Used Car Price Prediction
- **Teaser:** Predicting used car prices across 2.6M listings with three ensemble models and honest segment-level results.
- **Tags:** Python · Polars · XGBoost · CatBoost · LightGBM · Optuna · scikit-learn
- **Stat highlights (panel):**
  - `2.6M` / rows processed
  - `3` / models compared (XGBoost, CatBoost, LightGBM)
  - `Optuna` / Bayesian hyperparameter search + K-fold CV
- **Panel description:** Two paragraphs covering: what the project set out to do, pipeline highlights (Polars for scale, three-model comparison with rigorous eval, Optuna + K-fold), honest limitations and segment-level breakdown. Team effort framing throughout — no solo credit.
- **Links:** GitHub (primary button) — placeholder until repo migrated from JAMK GitLab
- **Framing note:** Team project. Credit JAMK coursework and dataset origin honestly.

### Card 2 — Future Factory / Team Fearless
- **Category badge:** `Team Leadership`
- **Title:** Future Factory — Team Fearless
- **Teaser:** Led a 6-person cross-functional team delivering a PrestaShop e-commerce platform across six Agile sprints and four Gate reviews.
- **Tags:** PrestaShop · Docker · GitLab CI/CD · CSC Allas · Scrum
- **Stat highlights (panel):**
  - `6` / team members (TL + DEV + SEC + OPS + Test)
  - `6` / Agile sprints
  - `4` / Gate reviews (GATE0–GATE4)
- **Panel description:** Two paragraphs covering team composition and Joona's dual TL+DEV role, technical deliverables (Docker, CSC Allas backup, CI/CD, payment integrations, security), Scrum + Essence Alpha methodology.
- **Links:** GitHub (primary, placeholder) + Docs (ghost → https://core-c0ad9d.pages.labranet.jamk.fi/)
- **Framing note:** Emphasise leadership + delivery as much as tech stack.

### Card 3 — pankkonen.dev (this site)
- **Category badge:** `Frontend`
- **Title:** pankkonen.dev
- **Teaser:** A portfolio built from scratch — Astro, GSAP scroll animations, and a design system inspired by Revolut.
- **Tags:** Astro · Tailwind CSS · GSAP · Cloudflare Pages · TypeScript
- **Stat highlights (panel):**
  - `Spring` / scroll easing (cubic-bezier 0.16, 1, 0.3, 1)
  - `3` / custom Astro components
  - `Cloudflare` / Pages deployment
- **Panel description:** Two paragraphs on design intent (Revolut-inspired dark premium aesthetic) and technical highlights (Astro SSG, IntersectionObserver with double-rAF replay, GSAP hero stagger, design token system).
- **Links:** GitHub (primary, placeholder until repo is public)
- **Framing note:** Solo project — slightly lighter, more personal tone in copy.

---

## Card Component (collapsed state)

```
[ Category badge ]          ← mono pill, amber-tinted border + bg
Project Title               ← Fraunces display, ~28–32px
One-sentence teaser.        ← text-muted, body-m
─────────────────────
[ Tag ] [ Tag ] [ Tag ]     ← small mono pills, rule border
```

**Hover:** `translate: 0 -2px`, border shifts to blue, soft glow — same pattern as Stack cards.

**Active/selected:** amber border (`rgba(245,158,11,0.45)`), small amber bottom-edge indicator connecting visually to the panel below.

**Scroll reveal:** each card gets `data-reveal` + `data-reveal-delay` (1, 2, 3).

---

## Detail Panel (expanded state)

Full-width panel immediately below the card grid. Two-column layout:

**Left (~60%):**
- Project title (larger than card)
- 2-paragraph description
- Primary button (GitHub) + optional ghost button (docs/live)

**Right (~40%):**
- 2–3 stat highlight blocks: large Fraunces numeral + small label
- Full tech stack tag row below stats

**Open animation:** height 0 → auto, spring easing `(0.16, 1, 0.3, 1)`, 500ms. Content fades up with stagger once panel is open.

**Swap (card switch):** content crossfades — fade out 150ms, fade in 300ms. Panel height adjusts smoothly.

**Close:** clicking active card again → height → 0, spring, 400ms.

**Mobile:** single column, stats become 2-col grid or horizontal scroll.

---

## Component Architecture

Single `Projects.astro` component:
- Section header (static markup)
- Card grid (rendered from typed `projects` array in frontmatter)
- Detail panel (single DOM node, content swapped via JS)
- Inline `<script>` for all interaction (no new dependencies)

---

## GitHub Link Strategy

Placeholder links (`#`) render as visually disabled (reduced opacity, cursor: not-allowed) until repos are migrated. Swap to real URLs before launch.

- Used Car ML → migrate from JAMK GitLab, strip course internals, new README
- Team Fearless → same migration process
- Portfolio site → https://github.com/joonapankkone/pankkonen-dev

---

## Out of Scope (v1)

- Individual `/projects/[slug]` detail pages
- Screenshots or images inside the panel
- Card filtering/sorting
- Light mode

---

## Design System Consistency

- Section top padding: `var(--space-13)` (104px) — matches Experience and Stack
- Animated divider (`data-rule-reveal`) before this section
- Spring easing `cubic-bezier(0.16, 1, 0.3, 1)` throughout
- `translate` (CSS Level 5) for hover, `transform` for reveal — no conflict
- All transitions declared explicitly in component CSS (Astro specificity rule)

---

## Implementation Notes (for writing-plans)

**Height animation:** CSS cannot transition `height: 0` to `height: auto`. Implementation must:
1. Measure the panel's `scrollHeight` in JS before opening
2. Set `height` to that pixel value and start the CSS transition
3. After `transitionend`, set `height: auto` so the panel reflows correctly if content changes

**Panel content stagger:** Panel content is injected dynamically so the IntersectionObserver cannot drive its reveal. Instead, after injecting new content, JS adds `is-visible` to each stagger child with `setTimeout` offsets (0ms, 80ms, 160ms, 240ms) — matching the established stagger timing.

**Transition property conflict:** The panel element needs `transition: height 500ms cubic-bezier(0.16, 1, 0.3, 1)` declared explicitly. It must NOT be a `[data-reveal]` element (that would add `opacity`/`transform` transitions that conflict with the height animation). Scroll reveal does not apply to the panel itself — only to the cards above it.
