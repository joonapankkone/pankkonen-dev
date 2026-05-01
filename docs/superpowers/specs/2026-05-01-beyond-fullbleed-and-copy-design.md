# Beyond the Code — Full-Bleed Video Treatment + About/Beyond Copy Refresh — Design Spec

**Date:** 2026-05-01
**Status:** LOCKED (approved by Joona, ready for implementation plan)

---

## Overview

Two-thread rework of the existing About + Beyond the Code area in `src/components/About.astro`:

1. **Beyond the Code section** — replace the current 50/50 text + vertical 9:16 video grid with a **full-bleed background video treatment** (treatment 3A: bottom-up dark scrim, text at bottom). New 16:9 highlight clip from the EAWrc knockout simulator race replaces the existing `btc-sim` clip.
2. **About + Beyond copy refresh** — tighten About (drop the third "I build systems" paragraph entirely, lighten paragraphs 1–2, tighten the Harvia mention and add a hand-in-larger-IT-projects note), and replace the Beyond copy entirely (drop Witcher 3, drop the "pattern I notice" closing, reframe sim racing as competitive history rather than active hobby).

Hero, Projects, Stack, Experience and Contact sections are **NOT touched**.

---

## 1. Beyond the Code — Locked Design (Treatment 3A)

### 1.1 Visual concept

The 16:9 sim racing highlight clip plays full-bleed as the section's background. A bottom-up dark gradient scrim overlays the video so that the upper portion shows the clip strongly and the lower portion is dark enough for text to sit comfortably on top. Section content (label, heading, copy) is anchored to the bottom of the section, sitting inside the scrim.

Reference aesthetic: Apple keynote chapter break + the dark-amber palette already used throughout pankkonen.dev.

### 1.2 Layout — desktop (`min-width: 769px`)

- Section is **full viewport width** (`100vw`), overriding `.container`'s 1280px cap with `width: 100vw; max-width: 100vw; margin-inline: calc(50% - 50vw); padding-inline: 0;` (same neutralisation pattern already used by `.section-divider`).
- Section height: `min(80vh, 720px)`. Below that lower bound (rare laptops), it gracefully shrinks; above that upper bound (large monitors), it caps so it does not eat a 1440p screen whole.
- Inner content (label + heading + copy) sits inside an inner `.container` (1280px max, `padding-inline: 24px`) anchored to the bottom of the section with `padding-block-end: var(--space-10)`.
- Video is `position: absolute; inset: 0; object-fit: cover; object-position: center` so the 16:9 clip fills any aspect ratio without distortion.

### 1.3 Scrim

```css
background: linear-gradient(
  180deg,
  rgba(8, 8, 10, 0.35) 0%,
  rgba(8, 8, 10, 0.55) 40%,
  rgba(8, 8, 10, 0.92) 85%,
  rgba(8, 8, 10, 0.96) 100%
);
```

Top 35% reveals the clip clearly; bottom is dark enough for body text contrast. Tuneable; treat the listed stops as starting values to adjust during implementation.

### 1.4 Layout — mobile (`max-width: 768px`)

Full-bleed is **abandoned on mobile**. A 16:9 clip cropped into a portrait viewport via `object-fit: cover` would show only a narrow horizontal strip of the original frame — most of the visual content disappears.

Mobile layout reverts to a **stacked letterbox** (treatment 2 from brainstorming):

- Label `05 · Beyond the Code` at top
- Heading
- 16:9 video laid out as a contained card, full-width, with the existing rounded-corner / soft-shadow treatment from the current `.beyond-media` style preserved
- Copy below the video

No scrim on mobile — video sits as its own card on the dark page background.

### 1.5 Playback behavior (both desktop and mobile)

- **Loop, no controls, muted, ambient.** Pure background element. No hover-pause, no audio toggle, no fullscreen affordance.
- **Attributes:** `autoplay loop muted playsinline preload="metadata"`. (`muted` + `playsinline` are required for iOS Safari autoplay.)
- **IntersectionObserver gating:** when the section enters the viewport, call `video.play()`. When it leaves, call `video.pause()`. This saves CPU and battery — do not let the clip play while it's offscreen.
- IO config: `rootMargin: '10% 0px 10% 0px'` (start playing slightly before the section is fully visible so it's already mid-frame when the user scrolls in).
- Reveal animations on label / heading / copy use the existing spring pattern (`cubic-bezier(0.16, 1, 0.3, 1)`, 750ms, 28px translateY, double-rAF, debounced exit) — no change to global `[data-reveal]` rules.

### 1.6 Reduced motion

Users with `prefers-reduced-motion: reduce` see a **poster image** instead of the playing video:

- `<video>` is rendered with `poster="/btc-sim-poster.jpg"` always, so the poster is the SSR-visible state.
- Inside a `@media (prefers-reduced-motion: reduce)` block, video playback is suppressed by JS (`video.removeAttribute('autoplay')` + skip the IO play call). The poster remains visible.
- Reveal animations for the surrounding text continue to honour the global `prefers-reduced-motion` handling already in place site-wide.

### 1.7 Accessibility

- Video carries `aria-hidden="true"` since it is purely decorative — content is conveyed by the visible heading and copy.
- Section retains `<section id="beyond">` so the existing in-page nav and skip patterns continue to work.
- Heading remains an `<h2>`; reading order is `label → heading → copy → (video implicit, hidden from AT)`.

---

## 2. Video file — encoding and sources

### 2.1 Source file

New highlight clip (provided by Joona):
`EAWrc_knockoutv2.mov`, 16:9, original-quality master.

### 2.2 Encoded outputs

The `.mov` master is **not** served directly — `.mov` (QuickTime) plays unreliably across browsers. Two web-targeted re-encodes are produced **without quality or framerate degradation**:

| File | Codec / container | Target | Notes |
|------|-------------------|--------|-------|
| `public/btc-sim.webm` | VP9 in WebM | Modern browsers (Chrome, Firefox, Edge, modern Safari) | Higher visual quality per byte than H.264 |
| `public/btc-sim.mp4` | H.264 High Profile in MP4 | Safari fallback, older browsers | Universal compatibility |
| `public/btc-sim-mobile.webm` | VP9, 720p, lower bitrate | Mobile (`media="(max-width: 768px)"`) | Reduces data + decode load on phones |
| `public/btc-sim-poster.jpg` | JPEG, frame ~1s in | Poster + reduced-motion fallback | Used as `poster=` attribute |

Encoding parameters (ffmpeg):

- WebM (desktop): `-c:v libvpx-vp9 -crf 28 -b:v 0 -row-mt 1 -an` — preserves source fps, no audio track
- WebM (mobile): same, plus `-vf scale=-2:720` and `-crf 32`
- MP4: `-c:v libx264 -profile:v high -preset slow -crf 20 -pix_fmt yuv420p -movflags +faststart -an`
- Poster: `ffmpeg -ss 1 -i source.mov -frames:v 1 -q:v 3 btc-sim-poster.jpg`

`-an` strips audio entirely from all video files (the clip is muted in DOM anyway; stripping the track avoids shipping unused bytes).

### 2.3 `<video>` element

```html
<video
  class="beyond-video"
  autoplay
  loop
  muted
  playsinline
  preload="metadata"
  poster="/btc-sim-poster.jpg"
  aria-hidden="true"
>
  <source src="/btc-sim-mobile.webm" type="video/webm" media="(max-width: 768px)" />
  <source src="/btc-sim.webm" type="video/webm" />
  <source src="/btc-sim.mp4" type="video/mp4" />
</video>
```

### 2.4 Old files — replacement strategy

The old `public/btc-sim.webm` and `public/btc-sim.mp4` are overwritten in place with the new 16:9 encodes — same canonical names, new content. No rename, no separate delete step needed; the writes from Section 2.2 produce the final state. Two new files are added alongside (`btc-sim-mobile.webm`, `btc-sim-poster.jpg`).

---

## 3. Copy changes

### 3.1 About — locked

Drop paragraph 3 entirely. Polish paragraphs 1 and 2.

**FROM (3 paragraphs):**

> I'm studying IT Engineering at JAMK in Jyväskylä, specialising in AI and data analytics. Alongside that, I'm doing an IT Trainee placement at Harvia — device management, Intune, help desk work. It's a deliberately different layer of tech from the coursework.
>
> The academic track goes deep into data: large-scale ML pipelines, multi-model comparisons with Bayesian tuning, and evaluation that breaks results down by segment instead of stopping at a headline number. The direction I'm pointing at is a junior data or AI role — somewhere the outputs have to be right, not just plausible.
>
> I tend to build systems rather than features when I have the choice. Whether that's a 2.6-million-row ML pipeline or this portfolio — proper design tokens, a bespoke animation layer, a spec written before a line of CSS was touched. The extra time upfront is usually worth it.

**TO (2 paragraphs):**

> I'm studying IT Engineering at JAMK in Jyväskylä, specialising in AI and data analytics. Alongside studies, I'm an IT Trainee at Harvia — Intune, device management, and help desk on the day-to-day, plus a hand in larger IT projects across the company.
>
> Academically the focus is data — large-scale ML pipelines, multi-model comparisons with Bayesian tuning, evaluation that breaks results down by segment rather than stopping at a headline number. The direction I'm pointing at is a junior data or AI role — somewhere outputs have to be right, not just plausible.

The `.about-body` layout currently uses `grid-template-columns: repeat(3, minmax(0, 1fr))`. With two paragraphs, switch to `repeat(2, minmax(0, 1fr))` so the prose still fills the row evenly without an empty third column.

### 3.2 Beyond — locked

**FROM:**
> Outside engineering I compete in sim racing — two world championships in the DiRT Rally series — train at the gym four times a week, and am probably replaying The Witcher 3 as you read this. The pattern I notice: I gravitate toward systems complex enough to reward patience.

**TO (two short paragraphs):**

> Two-time DiRT Rally world champion with a competitive sim racing background.
>
> These days, I keep busy with gym training, outdoor activities, open water swimming, and Latin dance.

The Beyond heading (`Not just at the keyboard.`) stays. The copy renders as two `<p>` elements inside `.beyond-text`, each with its own `data-reveal` and staggered delay.

---

## 4. Markup overhaul — Beyond section

The current `.beyond-grid` (1fr + 280px columns) is replaced with a single full-bleed container holding the absolutely-positioned video, the scrim, and the bottom-anchored content `.container`.

```html
<section class="beyond" id="beyond">
  <video class="beyond-video" ...>...</video>
  <div class="beyond-scrim" aria-hidden="true"></div>
  <div class="container beyond-inner">
    <p class="section-index mono" data-reveal>05 · Beyond the Code</p>
    <h2 class="beyond-heading" data-reveal data-reveal-delay="1">
      Not just<br />
      <span class="beyond-accent">at the keyboard</span>.
    </h2>
    <p class="beyond-para" data-reveal data-reveal-delay="2">
      Two-time DiRT Rally world champion with a competitive sim racing background.
    </p>
    <p class="beyond-para" data-reveal data-reveal-delay="3">
      These days, I keep busy with gym training, outdoor activities, open water swimming, and Latin dance.
    </p>
  </div>
</section>
```

Mobile branch (handled in CSS, not separate markup): on `(max-width: 768px)` the section reverts to natural document flow — `.beyond` loses its fixed height and full-bleed override; `.beyond-video` gets `position: static; width: 100%; aspect-ratio: 16 / 9; border-radius: 14px; box-shadow: 0 0 0 1px var(--rule);` so it presents as a contained letterbox card; `.beyond-scrim` is hidden (`display: none`); `.beyond-inner` becomes a normal-flow column with the video appearing between heading and copy via `display: contents` plus DOM-order tweak (or by giving `.beyond-video` `order: 1` inside a flex `.beyond-inner`). Single source of truth in markup, layout swap via media query only.

---

## 5. Files affected

| File | Change |
|------|--------|
| `src/components/About.astro` | Beyond markup overhaul (full-bleed video + scrim + bottom-anchored container), About copy edit (drop para 3, polish 1–2), Beyond copy replaced, CSS updates for both sections |
| `public/btc-sim.webm` | Replaced (new 16:9 source from `EAWrc_knockoutv2.mov`) |
| `public/btc-sim.mp4` | Replaced (same new source) |
| `public/btc-sim-mobile.webm` | New — mobile-bitrate variant |
| `public/btc-sim-poster.jpg` | New — poster + reduced-motion fallback |

No other component files touched. Global styles (`src/styles/global.css`) untouched — the spring easing tokens, design tokens and `[data-reveal]` rules already exist and are reused as-is.

---

## 6. Performance & accessibility checklist

- [ ] Video play/pause gated by IntersectionObserver — confirmed via DevTools (verify `video.paused` toggles when section enters/leaves viewport)
- [ ] Mobile source served only on mobile (verified via DevTools network tab on a throttled connection)
- [ ] `preload="metadata"` confirmed (no full clip download until play)
- [ ] Poster image visible during initial load and reduced-motion mode
- [ ] Lighthouse: Performance score does not drop below the current main score by more than 3 points
- [ ] No CLS contribution from the video (size reserved via aspect-ratio or fixed section height)
- [ ] Keyboard tab order skips the decorative video (achieved via `aria-hidden="true"` and absence of focusable affordances)

---

## 7. Out of scope

- Adding a Beyond CTA / link
- Adding multiple clips with crossfade
- Server-side adaptive streaming (HLS/DASH) — overkill for a 5–15s ambient loop
- Restoring or replacing the deleted About paragraph 3 (deletion is intentional and locked)
- Touching any other section (Hero, Projects, Stack, Experience, Contact)

---

## 8. Open implementation questions (for the writing-plans phase)

These are not design questions — they are details to settle when producing the step-by-step plan:

1. Does Joona have ffmpeg installed locally, or should the encoding step be scripted into `package.json` so it can be re-run against future source clips?
2. Does the existing `dist.stale.*` cleanup workflow need a tweak, or are those folders ignored by git already?
3. Should the IO that gates video playback be a separate small script, or merged into the existing reveal IO loader?

These are tractable from the codebase and don't require further user input.
