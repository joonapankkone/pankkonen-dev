# Phase 5 — Projects Section Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `Projects.astro` component — a 3-card grid with a full-width height-animated detail panel — and wire it into `index.astro` with an animated section divider.

**Architecture:** Single `Projects.astro` containing a typed `projects` data array, static card grid markup, `<template>` elements pre-rendered by Astro for each project's panel content, a shared `<div class="proj-panel">` height-animated container, and an inline `<script>` that manages open/close/swap via async JS + `requestAnimationFrame`. No new dependencies.

**Tech Stack:** Astro SSG, CSS custom properties, CSS Level 5 `translate` for hover (keeps hover independent of reveal `transform`), vanilla JS with `async/await` + `requestAnimationFrame` for panel height animation, existing IntersectionObserver in `index.astro`.

---

## File Map

| Action | Path | Responsibility |
|--------|------|----------------|
| **Create** | `src/components/Projects.astro` | Entire Projects section — data, markup, styles, interaction script |
| **Modify** | `src/pages/index.astro` | Import Projects, add animated divider before it, render `<Projects />` |

---

## Task 1: Component shell + data array

**Files:**
- Create: `src/components/Projects.astro`

- [ ] **Create the file with the full `projects` data array and outer section wrapper — no styles or script yet**

```astro
---
// Projects — Section 03
// Apple product-grid pattern: cards stay visible, full-width panel opens below.
// Ref: docs/superpowers/specs/2026-04-24-phase5-projects-design.md

type Stat = { value: string; label: string };
type Link = { href: string; label: string; style: 'primary' | 'ghost'; disabled?: boolean };

type Project = {
  id: string;
  badge: string;
  title: string;
  teaser: string;
  tags: string[];
  stats: Stat[];
  paragraphs: string[];
  links: Link[];
};

const projects: Project[] = [
  {
    id: 'used-car-ml',
    badge: 'Data & ML',
    title: 'Used Car Price Prediction',
    teaser:
      'Predicting used car prices across 2.6M listings with three ensemble models and honest segment-level results.',
    tags: ['Python', 'Polars', 'XGBoost', 'CatBoost', 'LightGBM', 'Optuna', 'scikit-learn'],
    stats: [
      { value: '2.6M', label: 'rows processed' },
      { value: '3',    label: 'models compared' },
      { value: 'K-fold + Optuna', label: 'hyperparameter search' },
    ],
    paragraphs: [
      'We built a full ML pipeline for used car price prediction — 2.6M rows processed with Polars, feature engineering that explicitly handled survivorship bias and target leakage, and a three-model comparison: XGBoost, CatBoost (ordered boosting), and LightGBM. Bayesian hyperparameter search via Optuna and K-fold cross-validation drove tuning.',
      'Evaluation went beyond headline metrics — MAE, RMSE, and R² broken down by price band and car type, because segment-level failure modes matter more than aggregate numbers. Limitations are documented honestly. A JAMK coursework project, delivered as a team.',
    ],
    links: [
      { href: '#', label: 'GitHub', style: 'primary', disabled: true },
    ],
  },
  {
    id: 'team-fearless',
    badge: 'Team Leadership',
    title: 'Future Factory — Team Fearless',
    teaser:
      'Led a 6-person cross-functional team delivering a PrestaShop e-commerce platform across six Agile sprints and four Gate reviews.',
    tags: ['PrestaShop', 'Docker', 'GitLab CI/CD', 'CSC Allas', 'Scrum'],
    stats: [
      { value: '6', label: 'team members' },
      { value: '6', label: 'Agile sprints' },
      { value: '4', label: 'Gate reviews' },
    ],
    paragraphs: [
      'Led a six-person cross-functional team building and deploying a PrestaShop e-commerce platform as a managed service — Docker containerisation, automated DB backup to CSC Allas, CI/CD on GitLab, payment integrations, and security hardening. Delivered across six Scrum sprints and four Gate reviews under the Essence Alpha framework.',
      'The dual Team Leader and Developer role meant sprint planning, task allocation, and Product Owner collaboration alongside writing actual code. Gate 1 passed cleanly. The team covered the full scope: OPS, security, testing, two more developers. Documentation is public.',
    ],
    links: [
      { href: '#', label: 'GitHub', style: 'primary', disabled: true },
      {
        href: 'https://core-c0ad9d.pages.labranet.jamk.fi/',
        label: 'Docs',
        style: 'ghost',
        disabled: false,
      },
    ],
  },
  {
    id: 'portfolio',
    badge: 'Frontend',
    title: 'pankkonen.dev',
    teaser:
      'A portfolio built from scratch — Astro, GSAP scroll animations, and a design system inspired by Revolut.',
    tags: ['Astro', 'Tailwind CSS', 'GSAP', 'Cloudflare Pages', 'TypeScript'],
    stats: [
      { value: 'Spring', label: 'scroll easing (0.16, 1, 0.3, 1)' },
      { value: '3',      label: 'custom Astro components' },
      { value: 'CF Pages', label: 'deployment' },
    ],
    paragraphs: [
      'Built from scratch with a clear design intent — something a recruiter would remember. Astro handles the build, scoped CSS components keep specificity manageable, and GSAP drives the hero entrance animation. The scroll reveal system uses IntersectionObserver with a double requestAnimationFrame trick to prevent skipped transitions.',
      'Every animation replays on scroll in both directions, spring easing (cubic-bezier 0.16, 1, 0.3, 1) throughout, and the expanding panel you\'re inside right now was designed before a line of CSS was written. The design system — tokens, typography scale, spacing — lives in global.css and is documented in DESIGN_SPEC.md.',
    ],
    links: [
      { href: '#', label: 'GitHub', style: 'primary', disabled: true },
    ],
  },
];
---

<section class="proj" id="projects" aria-labelledby="proj-heading">
  <div class="container proj-wrap">
    <!-- header, grid, templates, panel rendered in later tasks -->
  </div>
</section>

<style>
  .proj {
    padding-block: var(--space-13) var(--space-13);
    position: relative;
  }
  .proj-wrap {
    display: grid;
    grid-template-columns: minmax(0, 1fr);
    gap: var(--space-8);
  }
</style>
```

- [ ] **Verify: run dev server and confirm the section mounts without errors**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev" && npm run dev -- --port 4322 &
sleep 4 && curl -s http://localhost:4322 | grep -c "projects" && kill %1
```

Expected: at least `1` (the section id appears in HTML).

- [ ] **Commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add src/components/Projects.astro
git commit -m "feat(projects): scaffold component with data array"
```

---

## Task 2: Section header

**Files:**
- Modify: `src/components/Projects.astro` — replace the `<!-- header -->` comment

- [ ] **Replace the comment inside `.proj-wrap` with the section header markup**

Replace:
```html
    <!-- header, grid, templates, panel rendered in later tasks -->
```

With:
```html
    <header class="proj-header">
      <p class="section-index mono" data-reveal>03 · Projects</p>
      <h2 class="proj-heading" id="proj-heading" data-reveal data-reveal-delay="1">
        Work that ships.<br />
        <span class="proj-accent">Real data, agile teams, hard tradeoffs.</span>
      </h2>
      <p class="proj-lede" data-reveal data-reveal-delay="2">
        From 2.6M-row ML pipelines to a 6-person Agile team — each project here
        has a GitHub repo, a story, and something honest to say about what didn't
        go perfectly.
      </p>
    </header>

    <!-- card grid — Task 3 -->
    <!-- templates + panel — Task 4 -->
```

- [ ] **Add header CSS inside the `<style>` block**

```css
  /* ─── Header ──────────────────────────────────────────────── */
  .proj-header {
    display: grid;
    grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
    gap: var(--space-8);
    align-items: end;
    margin-bottom: var(--space-5);
  }

  .proj-header .section-index {
    grid-column: 1 / -1;
    margin-bottom: var(--space-3);
  }

  .proj-heading {
    grid-column: 1 / 2;
    font-size: var(--text-display-l);
    color: var(--text);
    line-height: 1.02;
  }

  .proj-accent {
    color: var(--primary-soft);
    font-style: italic;
    font-weight: 300;
  }

  .proj-lede {
    grid-column: 2 / 3;
    color: var(--text-muted);
    font-size: var(--text-body-l);
    line-height: 1.55;
    max-width: 48ch;
    justify-self: end;
    padding-bottom: 6px;
  }

  @media (max-width: 768px) {
    .proj-header {
      grid-template-columns: 1fr;
      gap: var(--space-3);
      align-items: start;
    }
    .proj-heading { grid-column: 1 / -1; font-size: 40px; }
    .proj-lede    { grid-column: 1 / -1; justify-self: start; }
  }
```

- [ ] **Commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add src/components/Projects.astro
git commit -m "feat(projects): add section header with stagger reveals"
```

---

## Task 3: Card grid markup + CSS

**Files:**
- Modify: `src/components/Projects.astro`

- [ ] **Replace the `<!-- card grid -->` comment with the card grid**

```html
    <div class="proj-grid">
      {projects.map((p, i) => (
        <article
          class="proj-card"
          data-id={p.id}
          data-active="false"
          data-reveal
          data-reveal-delay={String(i + 1)}
          role="button"
          tabindex="0"
          aria-expanded="false"
          aria-controls="proj-panel"
        >
          <span class="proj-badge mono">{p.badge}</span>
          <h3 class="proj-card-title">{p.title}</h3>
          <p class="proj-card-teaser">{p.teaser}</p>
          <ul class="proj-card-tags" role="list">
            {p.tags.slice(0, 4).map((tag) => (
              <li class="proj-tag mono">{tag}</li>
            ))}
          </ul>
        </article>
      ))}
    </div>
```

- [ ] **Add card CSS inside `<style>`**

```css
  /* ─── Card grid ───────────────────────────────────────────── */
  .proj-grid {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: var(--space-3);
  }

  .proj-card {
    position: relative;
    background: var(--surface);
    border: 1px solid var(--rule);
    border-radius: 16px;
    padding: var(--space-5);
    display: flex;
    flex-direction: column;
    gap: 16px;
    cursor: pointer;
    user-select: none;
    /* translate: hover lift | opacity+transform: data-reveal reveal */
    transition:
      background 240ms ease,
      border-color 240ms ease,
      translate 220ms cubic-bezier(0.16, 1, 0.3, 1),
      opacity 750ms cubic-bezier(0.16, 1, 0.3, 1),
      transform 750ms cubic-bezier(0.16, 1, 0.3, 1);
    overflow: visible;
  }

  .proj-card:hover {
    background: var(--surface-raised);
    border-color: rgba(96, 165, 250, 0.35);
    translate: 0 -2px;
  }

  .proj-card[data-active='true'] {
    border-color: rgba(245, 158, 11, 0.45);
    background: var(--surface-raised);
    translate: 0 0; /* cancel hover lift when active */
  }

  /* Amber bottom bar connecting active card to the panel below */
  .proj-card[data-active='true']::after {
    content: '';
    position: absolute;
    bottom: -1px;
    left: 50%;
    transform: translateX(-50%);
    width: 40px;
    height: 2px;
    background: var(--amber);
    border-radius: 2px 2px 0 0;
    box-shadow: 0 0 10px rgba(245, 158, 11, 0.5);
  }

  .proj-badge {
    display: inline-flex;
    align-items: center;
    align-self: flex-start;
    padding: 3px 10px;
    border: 1px solid rgba(245, 158, 11, 0.3);
    border-radius: 999px;
    background: rgba(245, 158, 11, 0.06);
    color: var(--amber);
    font-size: 11px;
    letter-spacing: 0.14em;
    text-transform: uppercase;
  }

  .proj-card-title {
    font-family: var(--font-display);
    font-size: clamp(22px, 2.2vw, 28px);
    font-weight: 400;
    color: var(--text);
    letter-spacing: -0.02em;
    line-height: 1.1;
    transition: color 200ms ease;
  }

  .proj-card:hover .proj-card-title,
  .proj-card[data-active='true'] .proj-card-title {
    color: var(--primary-soft);
  }

  .proj-card-teaser {
    color: var(--text-muted);
    font-size: var(--text-body-m);
    line-height: 1.55;
    flex-grow: 1;
  }

  .proj-card-tags {
    list-style: none;
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    padding-top: var(--space-1);
    border-top: 1px solid var(--rule);
    margin-top: auto;
  }

  .proj-tag {
    padding: 2px 8px;
    border: 1px solid var(--rule);
    border-radius: 4px;
    font-size: 11px;
    color: var(--text-muted);
    letter-spacing: 0.04em;
  }

  @media (max-width: 900px) {
    .proj-grid { grid-template-columns: 1fr; }
  }
```

- [ ] **Commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add src/components/Projects.astro
git commit -m "feat(projects): card grid with hover and active states"
```

---

## Task 4: Panel HTML structure + CSS

**Files:**
- Modify: `src/components/Projects.astro`

- [ ] **Replace the `<!-- templates + panel -->` comment with template elements and the panel container**

```html
    <!-- Pre-rendered panel content — inert until JS clones into panel -->
    {projects.map((p) => (
      <template id={`tpl-${p.id}`}>
        <div class="proj-panel-inner">

          <div class="proj-panel-left">
            <h3 class="proj-panel-title panel-reveal">{p.title}</h3>
            {p.paragraphs.map((para) => (
              <p class="proj-panel-para panel-reveal">{para}</p>
            ))}
            <div class="proj-panel-links panel-reveal">
              {p.links.map((link) =>
                link.disabled ? (
                  <span
                    class={`btn btn-${link.style} btn-disabled`}
                    aria-disabled="true"
                    title="GitHub repo coming soon"
                  >
                    {link.label}
                  </span>
                ) : (
                  <a
                    href={link.href}
                    class={`btn btn-${link.style}`}
                    target="_blank"
                    rel="noopener noreferrer"
                  >
                    {link.label}
                  </a>
                )
              )}
            </div>
          </div>

          <div class="proj-panel-right">
            <div class="proj-stats panel-reveal">
              {p.stats.map((stat) => (
                <div class="proj-stat">
                  <span class="proj-stat-value">{stat.value}</span>
                  <span class="proj-stat-label mono">{stat.label}</span>
                </div>
              ))}
            </div>
            <ul class="proj-panel-tags panel-reveal" role="list">
              {p.tags.map((tag) => (
                <li class="proj-tag mono">{tag}</li>
              ))}
            </ul>
          </div>

        </div>
      </template>
    ))}

    <!-- Panel container — height animated by JS -->
    <div class="proj-panel" id="proj-panel" aria-hidden="true" aria-live="polite"></div>
```

- [ ] **Add panel CSS inside `<style>`**

```css
  /* ─── Panel container ─────────────────────────────────────── */
  /* height: 0 → px animated by JS; do NOT use data-reveal here */
  .proj-panel {
    height: 0;
    overflow: hidden;
    transition: height 500ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  .proj-panel-inner {
    display: grid;
    grid-template-columns: minmax(0, 3fr) minmax(0, 2fr);
    gap: var(--space-8);
    padding: var(--space-5);
    border: 1px solid rgba(245, 158, 11, 0.2);
    border-radius: 16px;
    margin-top: var(--space-3);
    background: var(--surface);
  }

  /* ─── Panel left ──────────────────────────────────────────── */
  .proj-panel-left {
    display: flex;
    flex-direction: column;
    gap: var(--space-3);
  }

  .proj-panel-title {
    font-family: var(--font-display);
    font-size: clamp(28px, 3vw, 40px);
    font-weight: 400;
    color: var(--text);
    letter-spacing: -0.02em;
    line-height: 1.05;
  }

  .proj-panel-para {
    color: var(--text-muted);
    font-size: var(--text-body-m);
    line-height: 1.65;
    max-width: 60ch;
  }

  .proj-panel-links {
    display: flex;
    gap: 12px;
    flex-wrap: wrap;
    margin-top: var(--space-1);
  }

  /* Disabled link state for placeholder GitHub buttons */
  .btn-disabled {
    opacity: 0.4;
    cursor: not-allowed;
    pointer-events: none;
  }

  /* ─── Panel right ─────────────────────────────────────────── */
  .proj-panel-right {
    display: flex;
    flex-direction: column;
    gap: var(--space-5);
    padding-left: var(--space-5);
    border-left: 1px solid var(--rule);
  }

  .proj-stats {
    display: flex;
    flex-direction: column;
    gap: var(--space-3);
  }

  .proj-stat {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .proj-stat-value {
    font-family: var(--font-display);
    font-size: clamp(32px, 3.5vw, 48px);
    font-weight: 300;
    color: var(--primary-soft);
    line-height: 1;
    letter-spacing: -0.03em;
    font-style: italic;
  }

  .proj-stat-label {
    color: var(--text-muted);
    font-size: 12px;
    letter-spacing: 0.1em;
    line-height: 1.4;
  }

  .proj-panel-tags {
    list-style: none;
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
  }

  /* ─── Panel content reveal ────────────────────────────────── */
  /* JS drives .is-shown (not IntersectionObserver) */
  .panel-reveal {
    opacity: 0;
    transform: translateY(12px);
    transition:
      opacity 400ms cubic-bezier(0.16, 1, 0.3, 1),
      transform 400ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  .panel-reveal.is-shown {
    opacity: 1;
    transform: translateY(0);
  }

  /* ─── Panel mobile ────────────────────────────────────────── */
  @media (max-width: 768px) {
    .proj-panel-inner {
      grid-template-columns: 1fr;
      gap: var(--space-5);
    }
    .proj-panel-right {
      padding-left: 0;
      border-left: none;
      border-top: 1px solid var(--rule);
      padding-top: var(--space-5);
    }
    .proj-stats {
      flex-direction: row;
      flex-wrap: wrap;
      gap: var(--space-3);
    }
    .proj-stat { min-width: 100px; }
  }
```

- [ ] **Commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add src/components/Projects.astro
git commit -m "feat(projects): panel structure and CSS with reveal system"
```

---

## Task 5: JS — open and close panel

**Files:**
- Modify: `src/components/Projects.astro` — add `<script>` block after `</section>`

- [ ] **Add the script with open/close logic**

```html
<script>
  const cards = document.querySelectorAll<HTMLElement>('.proj-card');
  const panel = document.getElementById('proj-panel') as HTMLElement;
  let activeId: string | null = null;

  const sleep = (ms: number) => new Promise<void>((r) => setTimeout(r, ms));
  const raf   = () => new Promise<void>((r) => requestAnimationFrame(r));

  /** Clone a project's <template> content into the panel. */
  function injectContent(id: string) {
    const tpl = document.getElementById(`tpl-${id}`) as HTMLTemplateElement;
    panel.innerHTML = '';
    panel.appendChild(tpl.content.cloneNode(true));
  }

  /** Add .is-shown to .panel-reveal elements with stagger. */
  function staggerReveal() {
    const items = panel.querySelectorAll<HTMLElement>('.panel-reveal');
    items.forEach((el, i) => {
      setTimeout(() => el.classList.add('is-shown'), 80 + i * 80);
    });
  }

  /** Set data-active and aria-expanded on cards. */
  function setActive(id: string | null) {
    cards.forEach((c) => {
      const isActive = c.dataset.id === id;
      c.dataset.active = String(isActive);
      c.setAttribute('aria-expanded', String(isActive));
    });
  }

  async function openPanel(id: string) {
    injectContent(id);
    panel.setAttribute('aria-hidden', 'false');

    // Let browser paint the injected content before measuring
    await raf();
    const targetH = panel.scrollHeight;
    panel.style.height = targetH + 'px';

    panel.addEventListener(
      'transitionend',
      (e) => {
        if ((e as TransitionEvent).propertyName === 'height') {
          panel.style.height = 'auto';
        }
      },
      { once: true }
    );

    activeId = id;
    setActive(id);
    staggerReveal();
  }

  function closePanel() {
    // Convert height: auto → px so the CSS transition can run from px → 0
    panel.style.height = panel.scrollHeight + 'px';

    requestAnimationFrame(() => {
      panel.style.height = '0';
    });

    panel.addEventListener(
      'transitionend',
      (e) => {
        if ((e as TransitionEvent).propertyName === 'height') {
          panel.innerHTML = '';
          panel.setAttribute('aria-hidden', 'true');
        }
      },
      { once: true }
    );

    activeId = null;
    setActive(null);
  }

  // Wire up card clicks (keyboard Enter/Space handled via role="button" + tabindex)
  cards.forEach((card) => {
    card.addEventListener('click', () => {
      const id = card.dataset.id!;
      if (activeId === id) {
        closePanel();
      } else if (activeId) {
        openPanel(id); // swap handled in Task 6
      } else {
        openPanel(id);
      }
    });

    card.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        card.click();
      }
    });
  });
</script>
```

- [ ] **Visual verification — open dev server and test:**
  - Clicking card 1 opens a panel below the grid ✓
  - Panel height animates smoothly (500ms spring) ✓
  - Amber border + bottom indicator appears on active card ✓
  - Panel content fades up in stagger ✓
  - Clicking the same card again collapses the panel ✓
  - Panel content clears after collapse ✓

- [ ] **Commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add src/components/Projects.astro
git commit -m "feat(projects): panel open/close with height animation"
```

---

## Task 6: JS — swap between cards

**Files:**
- Modify: `src/components/Projects.astro` — update the `openPanel(id)` branch in the click handler and add `swapPanel`

- [ ] **Add `swapPanel` function and update the click handler branch**

Replace the click handler's `else if (activeId)` branch and add `swapPanel`:

Add `swapPanel` between `closePanel` and the card click wiring:

```typescript
  async function swapPanel(id: string) {
    // 1. Fade out current inner content
    const oldInner = panel.querySelector<HTMLElement>('.proj-panel-inner');
    if (oldInner) {
      oldInner.style.transition = 'opacity 150ms ease';
      oldInner.style.opacity = '0';
      await sleep(150);
    }

    // 2. Inject new content, start invisible
    injectContent(id);
    const newInner = panel.querySelector<HTMLElement>('.proj-panel-inner')!;
    newInner.style.opacity = '0';

    // 3. Measure new content height and transition
    await raf();
    const newH = panel.scrollHeight;
    // Force reflow at current pixel height so the transition has a from-value
    const currentH = panel.offsetHeight;
    panel.style.height = currentH + 'px';
    requestAnimationFrame(() => {
      panel.style.height = newH + 'px';
    });
    panel.addEventListener(
      'transitionend',
      (e) => {
        if ((e as TransitionEvent).propertyName === 'height') {
          panel.style.height = 'auto';
        }
      },
      { once: true }
    );

    // 4. Fade in new content
    requestAnimationFrame(() => {
      newInner.style.transition = 'opacity 300ms ease';
      newInner.style.opacity = '1';
    });

    activeId = id;
    setActive(id);
    staggerReveal();
  }
```

Update the click handler's `else if` branch:

```typescript
      } else if (activeId) {
        swapPanel(id);
      } else {
```

- [ ] **Visual verification:**
  - Click card 1, then card 2 — content crossfades (150ms out, 300ms in) ✓
  - Active card indicator moves to the newly clicked card ✓
  - Panel height adjusts smoothly if content is taller/shorter ✓
  - Stagger reveal plays on the new content ✓
  - Click card 3, then back to card 1 — works in both directions ✓

- [ ] **Commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add src/components/Projects.astro
git commit -m "feat(projects): crossfade swap between project cards"
```

---

## Task 7: Wire into index.astro

**Files:**
- Modify: `src/pages/index.astro`

- [ ] **Add the import at the top of the frontmatter**

In `src/pages/index.astro`, add to the existing imports:

```astro
---
import Layout from '../layouts/Layout.astro';
import Experience from '../components/Experience.astro';
import Stack from '../components/Stack.astro';
import Projects from '../components/Projects.astro';
---
```

- [ ] **Add the divider and Projects section after Stack**

After `<Stack />`, add:

```html
  <!-- Divider: stack → projects -->
  <div class="container section-divider" data-rule-reveal aria-hidden="true"></div>

  <!-- 03 · PROJECTS -->
  <Projects />
```

- [ ] **Visual verification — full page check:**
  - Animated divider appears between Stack and Projects ✓
  - Projects section header reveals on scroll ✓
  - Three cards reveal with stagger on scroll ✓
  - Divider replays on scroll in both directions ✓
  - Section numbering reads: 01 · Experience, 02 · Stack, 03 · Projects ✓

- [ ] **Commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add src/pages/index.astro
git commit -m "feat: wire Projects into index.astro with animated divider"
```

---

## Task 8: Final visual verification + polish commit

- [ ] **Run through this checklist in the browser:**

**Cards (closed state):**
- [ ] All three cards visible in a 3-column grid on desktop
- [ ] Hover: card lifts 2px, border turns blue-ish, title colour shifts
- [ ] Badges render with amber tint (Used Car ML: Data & ML, Team Fearless: Team Leadership, portfolio: Frontend)
- [ ] Tags truncate to 4 on card face

**Panel (open state):**
- [ ] Panel border is amber-tinted (distinguishes from blue card borders)
- [ ] Left column: title (larger), two paragraphs, link button(s)
- [ ] Right column: stats in large Fraunces italic numerals, full tag list
- [ ] Disabled GitHub buttons render at 40% opacity with not-allowed cursor
- [ ] Docs button on Team Fearless opens `https://core-c0ad9d.pages.labranet.jamk.fi/` in new tab

**Transitions:**
- [ ] Open: 500ms spring, no pop or snap
- [ ] Close: panel collapses cleanly, amber indicator disappears
- [ ] Swap: 150ms fade-out, 300ms fade-in, smooth height adjustment

**Mobile (resize to <768px):**
- [ ] Cards stack to single column
- [ ] Panel switches to single-column layout
- [ ] Stats display in a horizontal row

**Accessibility:**
- [ ] Tab key navigates between cards
- [ ] Enter/Space on a focused card opens/closes the panel
- [ ] Active card has `aria-expanded="true"`

- [ ] **Fix any issues found, then commit**

```bash
cd "/sessions/brave-happy-ride/mnt/Personal CV webpage/pankkonen-dev"
git add -A
git commit -m "feat(projects): Phase 5 complete — projects section with expandable panel"
```

---

## Self-Review Notes

**Spec coverage:**
- ✅ 03 · Projects section heading, tagline, lede
- ✅ 3-column card grid with badge, title, teaser, 4 tags
- ✅ Hover: lift, border shift (via CSS Level 5 `translate`)
- ✅ Active card: amber border + bottom indicator
- ✅ Scroll reveal on cards via existing `[data-reveal]` + IO
- ✅ Full-width panel with height animation (scrollHeight trick)
- ✅ Panel left: title, 2 paragraphs, link buttons
- ✅ Panel right: stat blocks + tags
- ✅ Crossfade swap on card switch (150ms/300ms)
- ✅ Content stagger via `panel-reveal` + JS setTimeout
- ✅ Disabled GitHub link styling
- ✅ Animated section divider before Projects
- ✅ Mobile: single column cards + single column panel
- ✅ Keyboard accessibility (Enter/Space on cards)
- ✅ `aria-expanded` on cards, `aria-hidden` on panel

**Known deferred (v2):**
- Screenshots/images inside panel
- Individual `/projects/[slug]` detail pages
- Real GitHub URLs (swap in before launch after GitLab migration)
