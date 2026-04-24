# Divider Redesign + Flickering Fix — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the section divider animation (static line + dot) with an oval amber aura + amber line + shimmer sweep, and fix Stack card flickering via debounced IO removal.

**Architecture:** Two isolated edits — one CSS block replacement in `global.css`, one IO callback replacement in `index.astro`. No new files, no new dependencies, no DOM structure changes.

**Tech Stack:** Astro, CSS animations (keyframes), vanilla TypeScript/JS IntersectionObserver

---

## Files

| File | Change |
|------|--------|
| `src/styles/global.css` | Replace `[data-rule-reveal]` block (lines ~193–237) |
| `src/pages/index.astro` | Replace IO callback with debounced WeakMap version |

---

## Task 1 — Replace `[data-rule-reveal]` CSS in `global.css`

**File:** `src/styles/global.css`

- [ ] **Step 1: Delete the existing `[data-rule-reveal]` block**

Find and remove everything from `/* ─── Section rule` comment down to and including `[data-rule-reveal].is-visible::after { ... }` — approximately lines 193–237.

- [ ] **Step 2: Insert the new block in its place**

```css
/* ─── Section dividers — oval amber aura + amber line + shimmer ── */
/* No dot. Line and aura both animate on scroll-enter.              */
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
    rgba(245, 158, 11, 0.42)   0%,
    rgba(245, 158, 11, 0.19)  38%,
    rgba(245, 158, 11, 0.063) 65%,
    transparent               100%
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
    transparent               0%,
    rgba(255, 255, 255, 0.75) 50%,
    transparent               100%
  );
  border-radius: 2px;
  opacity: 0;
  pointer-events: none;
}

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

- [ ] **Step 3: Verify dev server renders correctly**

Check `http://localhost:4321` — scroll past each section divider. Expect:
- Oval amber glow rises up around the line
- Amber line fades in inside it
- White shimmer travels left → right after ~350ms
- No dot anywhere
- On scroll-out, everything resets; on scroll back in, it replays

---

## Task 2 — Replace IO callback in `index.astro` (flickering fix)

**File:** `src/pages/index.astro`

- [ ] **Step 1: Replace the existing IO callback**

Find this block (inside the `else {` branch, after the `prefersReduced` check):

```javascript
const io = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        requestAnimationFrame(() => {
          requestAnimationFrame(() => {
            entry.target.classList.add('is-visible');
          });
        });
      } else {
        entry.target.classList.remove('is-visible');
      }
    });
  },
  {
    rootMargin: '-8% 0px -6% 0px',
    threshold: 0.06,
  }
);

document.querySelectorAll('[data-reveal], [data-rule-reveal]').forEach((el) => io.observe(el));
```

Replace it entirely with:

```javascript
// WeakMap debounce: prevents flutter when an element sits exactly
// on the threshold boundary (rapid enter/exit events).
const removeTimers = new WeakMap<Element, ReturnType<typeof setTimeout>>();

const io = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        // Cancel any pending removal — element came back into view
        const pending = removeTimers.get(entry.target);
        if (pending !== undefined) {
          clearTimeout(pending);
          removeTimers.delete(entry.target);
        }
        // Double rAF: ensures opacity:0 is painted before transition starts
        requestAnimationFrame(() => {
          requestAnimationFrame(() => {
            entry.target.classList.add('is-visible');
          });
        });
      } else {
        // Delay removal by 250ms — if element re-enters within that window
        // (flutter), the removal is cancelled and is-visible stays on.
        const timer = setTimeout(() => {
          entry.target.classList.remove('is-visible');
          removeTimers.delete(entry.target);
        }, 250);
        removeTimers.set(entry.target, timer);
      }
    });
  },
  {
    rootMargin: '-8% 0px -6% 0px',
    threshold: 0.06,
  }
);

document.querySelectorAll('[data-reveal], [data-rule-reveal]').forEach((el) => io.observe(el));
```

- [ ] **Step 2: Verify flickering is gone**

Scroll to the boundary between the Stack section and the Projects section. Scroll slowly up and down so the Stack cards sit right at the viewport edge. Cards should hold their state — no flickering.

- [ ] **Step 3: Commit both changes**

```bash
cd /path/to/pankkonen-dev
git add src/styles/global.css src/pages/index.astro
git commit -m "feat: divider oval amber aura + shimmer, fix IO flutter debounce"
```

---

## Self-review

- Spec requirement: oval aura, 55px height, 0.42 brightness → Task 1 ✅
- Spec requirement: amber line, 0.50 opacity → Task 1 ✅
- Spec requirement: shimmer 850ms, 350ms delay → Task 1 ✅
- Spec requirement: oval rise 950ms → Task 1 ✅
- Spec requirement: debounce removal, 250ms, WeakMap → Task 2 ✅
- No dot anywhere → confirmed, `::after` is now shimmer only, no dot CSS ✅
- Reset on scroll-out → confirmed, `forwards` fill + class removal resets all three ✅
