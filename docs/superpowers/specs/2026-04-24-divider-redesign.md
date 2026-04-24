# Section Divider Redesign Spec
**Date:** 2026-04-24
**Status:** LOCKED

---

## Overview

Two changes in one:

1. **Divider visual rework** — Replace the static line + amber dot animation with an oval amber aura + amber line + shimmer sweep. No dot.
2. **Flickering fix** — Eliminate Stack card flutter at section boundaries by debouncing `is-visible` removal in the IntersectionObserver.

---

## Part 1 — Divider Animation

### Concept

Full-width oval amber aura that rises into view as you scroll past a section boundary. An amber line sits at the centre of the oval. A shimmer sweep travels across the line after the oval settles.

No dot. No expanding line. No scale pop.

### Locked values (user-tuned 2026-04-24)

| Parameter | Value |
|-----------|-------|
| Aura height | `55px` |
| Aura centre opacity | `0.42` |
| Aura mid stop (×0.45) | `0.19` |
| Aura outer stop (×0.15) | `0.063` |
| Line amber opacity | `0.50` |
| Oval rise duration | `950ms` |
| Oval easing | `cubic-bezier(0.16, 1, 0.3, 1)` |
| Line fade duration | `570ms` (60% of rise) |
| Line fade delay | `80ms` |
| Shimmer delay | `350ms` |
| Shimmer duration | `850ms` |
| Shimmer easing | `cubic-bezier(0.3, 0, 0.2, 1)` |

### CSS implementation

Replace the entire `[data-rule-reveal]` block in `src/styles/global.css`:

```css
/* === Section dividers === */
[data-rule-reveal] {
  position: relative;
  width: 100%;
  height: 1px;
  overflow: visible;
  background: rgba(245, 158, 11, 0); /* amber line — starts transparent */
}

/* Oval amber aura */
[data-rule-reveal]::before {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  height: 55px;
  top: 50%;
  border-radius: 50%;
  background: radial-gradient(
    ellipse 75% 55% at 50% 50%,
    rgba(245, 158, 11, 0.42) 0%,
    rgba(245, 158, 11, 0.19) 38%,
    rgba(245, 158, 11, 0.063) 65%,
    transparent 100%
  );
  transform: translateY(-50%) scaleY(0.25);
  opacity: 0;
  pointer-events: none;
}

/* Shimmer sweep */
[data-rule-reveal]::after {
  content: "";
  position: absolute;
  top: -1px;
  height: 3px;
  width: 140px;
  left: -140px;
  background: linear-gradient(
    to right,
    transparent 0%,
    rgba(255, 255, 255, 0.75) 50%,
    transparent 100%
  );
  border-radius: 2px;
  opacity: 0;
  pointer-events: none;
}

/* Keyframes */
@keyframes dividerOval {
  from { transform: translateY(-50%) scaleY(0.25); opacity: 0; }
  to   { transform: translateY(-50%) scaleY(1);    opacity: 1; }
}
@keyframes dividerLine {
  from { background: rgba(245, 158, 11, 0); }
  to   { background: rgba(245, 158, 11, 0.50); }
}
@keyframes dividerShimmer {
  from { left: -140px; opacity: 1; }
  to   { left: calc(100% + 140px); opacity: 1; }
}

/* Triggered by is-visible */
[data-rule-reveal].is-visible {
  animation: dividerLine 570ms ease forwards;
  animation-delay: 80ms;
}
[data-rule-reveal].is-visible::before {
  animation: dividerOval 950ms cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
[data-rule-reveal].is-visible::after {
  animation: dividerShimmer 850ms cubic-bezier(0.3, 0, 0.2, 1) forwards;
  animation-delay: 350ms;
}
```

### Reset behaviour

When `is-visible` is removed on scroll-out:
- Element `background` reverts to `rgba(245,158,11,0)` — line disappears
- `::before` reverts to `scaleY(0.25); opacity:0` — oval collapses
- `::after` reverts to `left: -140px; opacity:0` — shimmer resets

No explicit reset CSS needed. The animation runs fresh on every scroll-in.

---

## Part 2 — Flickering Fix

### Problem

Stack cards at the bottom of the viewport flicker (gain and lose `is-visible` rapidly) when the page is scrolled to a position where the IO threshold sits on the card's edge.

### Root cause

`rootMargin: '-8% 0px -6% 0px'` reduced the flutter zone but did not eliminate it. The IO fires alternating enter/exit events when an element sits exactly at the threshold boundary.

### Fix: debounced removal via WeakMap

Replace the IO callback in `src/pages/index.astro`:

```javascript
const removeTimers = new WeakMap();

const io = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      // Cancel any pending removal first
      clearTimeout(removeTimers.get(entry.target));
      removeTimers.delete(entry.target);
      requestAnimationFrame(() => requestAnimationFrame(() => {
        entry.target.classList.add('is-visible');
      }));
    } else {
      // Delay removal — prevents flutter when element sits on threshold boundary
      const timer = setTimeout(() => {
        entry.target.classList.remove('is-visible');
        removeTimers.delete(entry.target);
      }, 250);
      removeTimers.set(entry.target, timer);
    }
  });
}, {
  rootMargin: '-8% 0px -6% 0px',
  threshold: 0.06,
});

document.querySelectorAll('[data-reveal], [data-rule-reveal]').forEach((el) => io.observe(el));
```

The 250ms debounce means: if the element exits and re-enters within 250ms (flutter), the removal is cancelled — `is-visible` stays on. If the element genuinely exits (stays out >250ms), removal fires normally.

---

## Out of Scope

- Changing the divider DOM structure (stays as `<div class="container section-divider" data-rule-reveal aria-hidden="true">`)
- Any change to `[data-reveal]` card or element animations
- Mobile-specific overrides (divider already works at all widths — 100% width, height 1px)

---

## Files to Edit

| File | Change |
|------|--------|
| `src/styles/global.css` | Replace `[data-rule-reveal]` block |
| `src/pages/index.astro` | Replace IO callback with debounced version |
