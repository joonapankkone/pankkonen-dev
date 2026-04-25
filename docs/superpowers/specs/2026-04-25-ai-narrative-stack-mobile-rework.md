# AI Narrative + Stack 4th Card + Project Copy + Mobile Card Expand — Design Spec

**Date:** 2026-04-25
**Status:** LOCKED (approved by Joona, ready for implementation plan)

---

## Overview

Four-thread rework of the existing pankkonen.dev portfolio site:

1. **AI-narrative angle** — a "B + per-project D" prominence: hero stays locked, Stack gains a 4th `AI Workflow` card, project copy is reworked per-project at three different volume levels (pankkonen.dev = full, Used Car ML = mid, Future Factory = light).
2. **Stack section structure** — moves from 3-column grid to 2×2 grid to accommodate the new AI Workflow card without cramping.
3. **Project card copy + tags + stats rework** — incorporates AI tooling honestly and accurately per project.
4. **Mobile project-card expand UX** — replaces "panel below all cards" with "panel slides in directly below the clicked card", solving the painpoint where new content currently appears 1.5 screens below the user's tap.

The hero section (`Data. Code. Systems. — and a 2× world title on the side.`) is **NOT touched** — it remains locked per Q3 of the original brainstorming.

---

## 1. AI-Narrative Direction (Meta-Decision)

**Locked stance:** "I architect and direct, AI types." Joona orchestrates AI tools rather than being orchestrated by them. Every design and engineering decision stays his; AI handles execution and acts as a sparring partner.

**Per-project volume curve (decided 2026-04-25):**
- **pankkonen.dev** — full volume; AI angle leads the project narrative as the primary differentiator
- **Used Car ML** — mid volume; one sentence acknowledging AI as team-sparring third voice, team-decisioning primacy preserved
- **Future Factory** — light volume; tag-only mention of ChatGPT, no paragraph change

**Honesty constraints (non-negotiable):**
- Used Car ML stays framed as "team effort", never "I + AI", because that is the truth (per `project_used_car_ml.md`)
- pankkonen.dev paragraph mentions **Cowork mode** specifically, not "Claude Code", because Claude Code was NOT used for this build (Cowork mode in the Claude desktop app is what actually drove file edits)
- Claude Code remains in the Stack `AI Workflow` card because Joona uses it elsewhere (school work / general workflow), but it does NOT appear in the pankkonen.dev tag list
- Future Factory mentions ChatGPT only because that is the AI tool actually used in that project

---

## 2. Stack Section Rework

### 2.1 Layout change

**From:** `grid-template-columns: repeat(3, minmax(0, 1fr))` (3-column grid)
**To:** `grid-template-columns: repeat(2, minmax(0, 1fr))` (2×2 grid on desktop)

Mobile (`@media (max-width: 900px)`) stays at single column — no change.

A new mid breakpoint may be added later if a tablet view shows awkwardness, but is not part of the v1 implementation.

### 2.2 New card — IV. AI Workflow

```
IV. AI Workflow
   "The pair across the desk."

01  Claude          pair on big builds
02  Claude Code     agent at the keyboard
03  ChatGPT         code + writing polish
04  GitHub Copilot  the office pair
```

**Rendered inside the existing `categories` array in `Stack.astro`:**

```ts
{
  index: 'IV',
  title: 'AI Workflow',
  subtitle: 'The pair across the desk.',
  items: [
    { name: 'Claude',         note: 'pair on big builds'    },
    { name: 'Claude Code',    note: 'agent at the keyboard' },
    { name: 'ChatGPT',        note: 'code + writing polish' },
    { name: 'GitHub Copilot', note: 'the office pair'       },
  ],
},
```

**Order rationale (don't re-debate):** Claude family first (primary pair), then ChatGPT (everyday workhorse), then Copilot (work-context day-job tool). Reads as a small story top-to-bottom.

### 2.3 Other Stack cards — unchanged

Cards I, II, III (Data & ML / Web & Programming / Infrastructure) keep current content, items, notes. The only change at the existing-card level is grid position (now part of 2×2 layout instead of 3-col row).

### 2.4 CSS adjustments

`.stack-grid` rule changes:
```css
.stack-grid {
  grid-template-columns: repeat(2, minmax(0, 1fr));  /* was repeat(3, ...) */
  /* gap, margin-top unchanged */
}

@media (max-width: 900px) {
  .stack-grid { grid-template-columns: 1fr; }   /* unchanged */
}
```

No other Stack styles need to change. The 2×2 layout gives each card ~600px width on desktop (well above the cramping threshold), keeping the right-aligned `cat-item-note` legible.

---

## 3. Project Card Copy Rework

### 3.1 pankkonen.dev (full volume)

**Tags (final list — 6 tags, card glance shows first 4):**
```
Astro · Claude · Tailwind CSS · GSAP · TypeScript · Cloudflare Pages
```
- Removed: `Claude Code` (was not used for this build)
- Added: `Claude` (in slot 2, between `Astro` and `Tailwind CSS`)
- "Hybrid b" arrangement: technical foundation + AI pair both visible at glance

**Stats (panel, replace one):**
| Slot | Value | Label |
|------|-------|-------|
| 1    | `Spring`        | `scroll easing (0.16, 1, 0.3, 1)` |
| 2    | `Claude Cowork` | `pair-build environment`           | ← REPLACES `3` / `custom Astro components` |
| 3    | `CF Pages`      | `deployment`                       |

**Paragraphs (V3-kuiva, Cowork-corrected — first paragraph is NEW, then existing two follow as paragraphs 2 and 3):**

Paragraph 1 (NEW, opener):
> Built with a clear division of labour: I architected and directed, Claude typed. The whole project ran in Cowork mode — Claude reading and writing files in the workspace directly, while I made the design and direction calls. Every decision — palette, animation parameters, the expanding panel you just clicked open — stayed mine.

Paragraph 2 (existing, unchanged):
> Built from scratch with a clear design intent — something a recruiter would remember. Astro handles the build, scoped CSS components keep specificity manageable, and GSAP drives the hero entrance animation. The scroll reveal system uses IntersectionObserver with a double requestAnimationFrame trick to prevent skipped transitions.

Paragraph 3 (existing, unchanged):
> Every animation replays on scroll in both directions, spring easing (cubic-bezier 0.16, 1, 0.3, 1) throughout, and the expanding panel you're inside right now was designed before a line of CSS was written. The design system — tokens, typography scale, spacing — lives in global.css and is documented in DESIGN_SPEC.md.

### 3.2 Used Car ML (mid volume)

**Tags (final list — 9 tags, card glance shows first 4 = pure technical):**
```
Python · Polars · XGBoost · CatBoost · LightGBM · Optuna · scikit-learn · Claude · ChatGPT
```
- Added at end: `Claude`, `ChatGPT`
- Card glance unchanged (still leads with Python · Polars · XGBoost · CatBoost)
- AI tags appear only in panel-view full list

**Stats:** UNCHANGED (`2.6M` · `3` · `K-fold + Optuna`).

**Paragraphs (V-B — one sentence added to paragraph 2):**

Paragraph 1 (existing, unchanged):
> We built a full ML pipeline for used car price prediction — 2.6M rows processed with Polars, feature engineering that explicitly handled survivorship bias and target leakage, and a three-model comparison: XGBoost, CatBoost (ordered boosting), and LightGBM. Bayesian hyperparameter search via Optuna and K-fold cross-validation drove tuning.

Paragraph 2 (REVISED — one sentence appended):
> Evaluation went beyond headline metrics — MAE, RMSE, and R² broken down by price band and car type, because segment-level failure modes matter more than aggregate numbers. Limitations are documented honestly. A JAMK coursework project, delivered as a team — with Claude and ChatGPT sparring across feature engineering, code review, and the writeup.

### 3.3 Future Factory — Team Fearless (light volume)

**Tags (final list — 6 tags, card glance shows first 4 = pure tech/devops):**
```
PrestaShop · Docker · GitLab CI/CD · CSC Allas · Scrum · ChatGPT
```
- Added at end: `ChatGPT`
- Card glance unchanged
- ChatGPT appears only in panel-view full list

**Stats:** UNCHANGED (`6` · `6` · `4`).

**Paragraphs:** UNCHANGED. No paragraph rework. Tag-only mention is the entire AI footprint for this card.

---

## 4. Mobile Project-Card Expand UX

### 4.1 Painpoint (current state)

On mobile (`@media (max-width: 900px)` for grid → 1 column), all three cards stack vertically. The single shared `.proj-panel` div sits as a sibling AFTER the `.proj-grid`, so when the user taps card 1, the panel opens 1.5+ screens below the tap. The user's eye stays at the tapped card; new content materialises off-screen.

### 4.2 Pattern B — inline-after-clicked-card (DECIDED)

**Architecture:** Each card gets its own dedicated mobile-only panel slot rendered immediately after it. JS routes `injectContent(id)` into the right slot based on viewport. Desktop keeps the existing single panel below the grid; CSS toggles which is visible.

### 4.3 HTML changes (`Projects.astro`)

Inside the `.proj-grid` `projects.map(...)`, each card gets a sibling panel slot:

```astro
---
import { Fragment } from 'astro/jsx-runtime';
---
<div class="proj-grid">
  {projects.map((p, i) => (
    <Fragment>
      <article class="proj-card" data-id={p.id} ...>
        ...existing card markup...
      </article>
      <div
        class="proj-panel-mobile"
        data-panel-for={p.id}
        aria-hidden="true"
      ></div>
    </Fragment>
  ))}
</div>
```

The existing `<div class="proj-panel" id="proj-panel">` after the grid is **kept** (desktop drawer behaviour stays).

**Grid-layout note:** Both the cards and the new `.proj-panel-mobile` divs are children of `.proj-grid`. On desktop the mobile slots are `display: none` (which removes them from grid layout entirely — no impact on the 3-column cell positioning). On mobile (`grid-template-columns: 1fr`), each card and each mobile slot occupies its own row, naturally alternating.

### 4.4 CSS changes

```css
/* Mobile panel slots are hidden by default (desktop) */
.proj-panel-mobile {
  display: none;
  height: 0;
  overflow: hidden;
  transition: height 500ms cubic-bezier(0.16, 1, 0.3, 1);
}

@media (max-width: 768px) {
  /* Hide the desktop drawer */
  #proj-panel { display: none; }

  /* Show mobile slots — they sit inside .proj-grid between cards */
  .proj-panel-mobile { display: block; }
}
```

`.proj-panel-inner` styles (border, padding, grid layout) already responsively collapse to single-column on mobile via the existing `@media (max-width: 768px)` rules — they apply equally inside the mobile slot.

### 4.5 JS changes (`Projects.astro` script)

Replace the single `panel` reference with viewport-aware target selection:

```ts
const cards = document.querySelectorAll<HTMLElement>('.proj-card');
const desktopPanel = document.getElementById('proj-panel') as HTMLElement;
const mobilePanels = document.querySelectorAll<HTMLElement>('.proj-panel-mobile');
const mobileMQ = window.matchMedia('(max-width: 768px)');

let activeId: string | null = null;

function getPanelFor(id: string): HTMLElement {
  if (mobileMQ.matches) {
    return document.querySelector<HTMLElement>(
      `.proj-panel-mobile[data-panel-for="${id}"]`
    )!;
  }
  return desktopPanel;
}

// All existing functions (injectContent, openPanel, closePanel, swapPanel,
// staggerReveal, setActive) are refactored to take `id` and resolve the
// correct panel via getPanelFor(id).
```

**Swap behaviour on mobile:** Because each card has its own panel slot, swap is two operations — close the previous card's mobile panel, then open the new one. This replaces the existing `swapPanel` crossfade (which was specific to a shared single-panel desktop architecture). On mobile the natural flow is sequential: close → open.

**Suggested swap timing on mobile:** 250ms close animation, then 50ms gap, then open. Total feels like ~800ms of motion — acceptable per Revolut-grade feel.

### 4.6 Scroll polish

After `openPanel(id)` resolves height transition on mobile, run:
```ts
if (mobileMQ.matches) {
  const card = document.querySelector<HTMLElement>(`.proj-card[data-id="${id}"]`)!;
  card.scrollIntoView({ behavior: 'smooth', block: 'start' });
}
```

This anchors the active card's title at the top of the viewport so the user sees both the card header and the start of the panel content. Without this, the user's tap leaves them mid-panel and they lose context.

### 4.7 Amber underline tab — semantics preserved

The `.proj-card[data-active='true']::after` rule (amber 40px wide tab pointing down from the card's bottom) keeps its meaning on mobile: the panel sits directly below, so the tab still visually connects card → panel. No CSS change needed.

### 4.8 Resize handling (edge case)

If the user opens a card on mobile, then rotates / resizes to desktop width:
- The mobile panel slot collapses (CSS hides it)
- The desktop panel is empty (we never injected into it)
- The card's `data-active='true'` state remains

**Resolution:** add a `mobileMQ.addEventListener('change', ...)` listener that, if a card is currently active, re-routes content to the now-visible panel via re-running `openPanel(activeId)`. Keep this implementation simple — measurement re-runs on the new target.

---

## 5. Implementation Order (recommended)

1. **Stack rework** (smallest risk, no JS changes)
   - Edit `Stack.astro` — add IV. AI Workflow card to `categories` array
   - Change `.stack-grid` to 2-column
2. **Project copy rework** (data-only changes, no architecture)
   - Edit `Projects.astro` — update `projects` array (tags, stats, paragraphs) per § 3
3. **Mobile UX rework** (architecture change, most testing)
   - Add mobile panel slots in template
   - Add CSS visibility toggles
   - Refactor JS to be viewport-aware
   - Test resize handling, swap behaviour, scroll polish

Ship sequentially as three commits:
- `feat(stack): add IV. AI Workflow card and switch to 2x2 grid`
- `feat(projects): rework copy/tags/stats to surface AI orchestration`
- `feat(projects): inline-after-clicked-card panel pattern on mobile`

---

## 6. Acceptance Criteria

### Stack
- [ ] 4 cards visible in 2×2 grid on desktop ≥901px
- [ ] 1-column stack on viewports ≤900px
- [ ] AI Workflow card content matches § 2.2 exactly
- [ ] All existing animation behaviour (data-reveal stagger, hover lift) works on the 4th card identically to others

### Projects copy
- [ ] pankkonen.dev card has 6 tags in the order specified, replaced stat, and 3 paragraphs in the order specified
- [ ] Used Car ML card has 9 tags in the order specified and the revised paragraph 2
- [ ] Future Factory card has 6 tags in the order specified, no paragraph changes
- [ ] No mention of "Claude Code" in pankkonen.dev card body or tags
- [ ] Used Car ML preserves "team" framing

### Mobile UX
- [ ] On viewports ≤768px, tapping any card opens its panel directly below it
- [ ] On viewports ≥769px, behaviour is identical to current (single panel below grid)
- [ ] Active card's title scrolls into view at the top after open
- [ ] Tap a different card while one is open → previous closes smoothly, new one opens
- [ ] Tap active card again → closes
- [ ] Resize from mobile to desktop with a card open does not orphan the panel content
- [ ] Amber underline tab on active card still rendered

---

## 7. Out of Scope

- Hero tagline changes (locked)
- Adding `Cowork` to Stack section (too obscure for tag-glance — lives in pankkonen.dev paragraph only)
- Bottom-sheet modal pattern for mobile (rejected — breaks scroll-storytelling rhythm)
- Replacing other Used Car ML or Future Factory stats with AI-velocity stats (existing stats are stronger)
- About / Beyond the Code / Contact sections (Phase 6+ — separate spec)

---

## 8. References

- Original portfolio brainstorming Q1–Q9: memory file `project_portfolio_design_decisions.md`
- Used Car ML team-effort framing rule: memory file `project_used_car_ml.md`
- Future Factory project context: memory file `project_future_factory_team_fearless.md`
- Animation philosophy (spring easing, IO double-rAF): memory file `feedback_animation_design.md`
- Phase 5 Projects original spec: `docs/superpowers/specs/2026-04-24-phase5-projects-design.md`
- Section divider redesign: `docs/superpowers/specs/2026-04-24-divider-redesign.md`
