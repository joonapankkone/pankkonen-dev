# Beyond the Code — Full-Bleed Video + About/Beyond Copy Refresh — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the existing About/Beyond pair in `src/components/About.astro` with a refreshed two-paragraph About and a full-bleed background-video Beyond section (treatment 3A) using a new 16:9 EAWrc highlight clip, with mobile letterbox fallback and IO-gated playback.

**Architecture:** Single-component change. All markup, CSS and the video play/pause script live inside `src/components/About.astro` (Astro pattern). Four new video assets land in `public/`. No new component files, no global CSS changes — the existing reveal IO, design tokens and section-divider work unchanged.

**Tech Stack:** Astro 6, scoped Astro `<style>` and `<script>`, IntersectionObserver, ffmpeg (VP9/H.264 encoding).

**Spec:** `docs/superpowers/specs/2026-05-01-beyond-fullbleed-and-copy-design.md`

**Source clip:** `EAWrc_knockoutv2.mov` — 1920×1080, 60 fps, ~16 s, ~19 Mbps (verified via `ffprobe`). Currently in the session's `uploads/` folder; the user will copy it to a working location at the start of Task 1.

---

## File map

| Path | Change |
|------|--------|
| `src/components/About.astro` | Modify — markup overhaul of Beyond, copy edits in About + Beyond, CSS rework, new `<script>` for video IO |
| `public/btc-sim.webm` | Overwrite — new 1080p VP9 from EAWrc source |
| `public/btc-sim.mp4` | Overwrite — new 1080p H.264 from EAWrc source |
| `public/btc-sim-mobile.webm` | Create — 720p VP9 mobile variant |
| `public/btc-sim-poster.jpg` | Create — poster + reduced-motion still |
| `public/btc-sim.gif` | Delete — legacy file no longer used |

---

## Task 1: Encode the new video assets

**Files:**
- Source: `EAWrc_knockoutv2.mov` (provided by user, ~30 MB)
- Create: `public/btc-sim.webm`, `public/btc-sim.mp4`, `public/btc-sim-mobile.webm`, `public/btc-sim-poster.jpg`

- [ ] **Step 1: Place the source clip in a known working location**

Copy `EAWrc_knockoutv2.mov` (currently in the session uploads folder) to a stable working path:

```bash
SRC="/tmp/EAWrc_knockoutv2.mov"
cp "<uploads-path-of-the-mov-file>" "$SRC"
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,r_frame_rate,duration -of default=noprint_wrappers=1 "$SRC"
```

Expected output:
```
width=1920
height=1080
r_frame_rate=60/1
duration=16.033333
```

- [ ] **Step 2: Encode desktop WebM (VP9, 1080p, source fps preserved)**

```bash
SRC="/tmp/EAWrc_knockoutv2.mov"
DST="<repo-root>/pankkonen-dev/public/btc-sim.webm"
ffmpeg -y -i "$SRC" \
  -c:v libvpx-vp9 -crf 28 -b:v 0 -row-mt 1 -tile-columns 2 \
  -an -pix_fmt yuv420p \
  "$DST"
```

Verify:
```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,r_frame_rate -of default=noprint_wrappers=1 "$DST"
```
Expected: `width=1920`, `height=1080`, `r_frame_rate=60/1`. Resulting file should be ~3–6 MB.

- [ ] **Step 3: Encode desktop MP4 (H.264 High, 1080p, source fps preserved)**

```bash
SRC="/tmp/EAWrc_knockoutv2.mov"
DST="<repo-root>/pankkonen-dev/public/btc-sim.mp4"
ffmpeg -y -i "$SRC" \
  -c:v libx264 -profile:v high -preset slow -crf 20 \
  -pix_fmt yuv420p -movflags +faststart \
  -an \
  "$DST"
```

Verify dimensions and fps as in Step 2. Expected: ~5–10 MB.

- [ ] **Step 4: Encode mobile WebM (VP9, 720p, slightly higher CRF)**

```bash
SRC="/tmp/EAWrc_knockoutv2.mov"
DST="<repo-root>/pankkonen-dev/public/btc-sim-mobile.webm"
ffmpeg -y -i "$SRC" \
  -c:v libvpx-vp9 -crf 32 -b:v 0 -row-mt 1 \
  -vf scale=-2:720 \
  -an -pix_fmt yuv420p \
  "$DST"
```

Verify:
```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of default=noprint_wrappers=1 "$DST"
```
Expected: `width=1280`, `height=720`. Resulting file should be ~1.5–3 MB.

- [ ] **Step 5: Generate the poster JPEG (~1 second into the clip)**

```bash
SRC="/tmp/EAWrc_knockoutv2.mov"
DST="<repo-root>/pankkonen-dev/public/btc-sim-poster.jpg"
ffmpeg -y -ss 1 -i "$SRC" -frames:v 1 -q:v 3 "$DST"
```

Open the file and confirm it shows a clear, representative frame (no motion blur, recognisable as a sim racing scene). If frame 1s isn't a great still, try `-ss 2` or `-ss 0.5`.

- [ ] **Step 6: Delete the legacy `btc-sim.gif`**

```bash
cd "<repo-root>/pankkonen-dev"
rm public/btc-sim.gif
```

(The current About.astro doesn't reference it — confirmed via grep — so this is dead weight only.)

- [ ] **Step 7: Commit assets**

```bash
cd "<repo-root>/pankkonen-dev"
git add public/btc-sim.webm public/btc-sim.mp4 public/btc-sim-mobile.webm public/btc-sim-poster.jpg
git rm public/btc-sim.gif
git commit -m "assets: replace btc-sim with EAWrc 16:9 highlight clip + mobile + poster"
```

---

## Task 2: About — drop P3, polish P1–P2, switch grid to 2 columns

**Files:**
- Modify: `src/components/About.astro:21-41` (the `<div class="about-body">` block + paragraphs)
- Modify: `src/components/About.astro:133-137` (the `.about-body` grid CSS)

- [ ] **Step 1: Replace the three `<p class="about-para">` elements with two**

In `src/components/About.astro`, replace the entire `<div class="about-body">…</div>` block (lines 21–41) with:

```astro
    <div class="about-body">
      <p class="about-para" data-reveal data-reveal-delay="2">
        I'm studying IT Engineering at JAMK in Jyväskylä, specialising in AI and
        data analytics. Alongside studies, I'm an IT Trainee at Harvia — Intune,
        device management, and help desk on the day-to-day, plus a hand in
        larger IT projects across the company.
      </p>
      <p class="about-para" data-reveal data-reveal-delay="3">
        Academically the focus is data — large-scale ML pipelines, multi-model
        comparisons with Bayesian tuning, evaluation that breaks results down
        by segment rather than stopping at a headline number. The direction
        I'm pointing at is a junior data or AI role — somewhere outputs have
        to be right, not just plausible.
      </p>
    </div>
```

The third paragraph (`I tend to build systems rather than features…`) is removed entirely — this is intentional, locked in spec section 3.1.

- [ ] **Step 2: Switch `.about-body` grid from 3 columns to 2 columns**

In the same file, find the `.about-body` rule (around line 133):

```css
  .about-body {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: var(--space-6);
  }
```

Replace with:

```css
  .about-body {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: var(--space-6);
  }
```

- [ ] **Step 3: Run dev server and visually verify About**

```bash
cd "<repo-root>/pankkonen-dev"
npm run dev
```

Open the local URL Astro prints (typically `http://localhost:4321/`), scroll to section 04 (About). Verify:
- Two paragraphs, side-by-side on desktop
- Both fade in with stagger (P1 first, P2 ~80ms after)
- Mobile (Chrome DevTools, narrow viewport): two paragraphs stack to single column (the existing `@media (max-width: 768px) .about-body { grid-template-columns: 1fr }` rule handles this — no change needed)

- [ ] **Step 4: Commit About changes**

```bash
cd "<repo-root>/pankkonen-dev"
git add src/components/About.astro
git commit -m "content(about): tighten copy, drop closing paragraph, switch to 2-col grid"
```

---

## Task 3: Beyond — markup overhaul + new copy

**Files:**
- Modify: `src/components/About.astro:50-92` (the entire `<section class="beyond">` block)

- [ ] **Step 1: Replace the Beyond section markup**

Find the existing Beyond section (lines 50–92). Replace the whole `<section class="beyond" id="beyond">…</section>` block with the following:

```astro
<section class="beyond" id="beyond">
  <video
    class="beyond-video"
    autoplay
    loop
    muted
    playsinline
    preload="metadata"
    poster="/btc-sim-poster.jpg"
    aria-hidden="true"
    data-beyond-video
  >
    <source src="/btc-sim-mobile.webm" type="video/webm" media="(max-width: 768px)" />
    <source src="/btc-sim.webm" type="video/webm" />
    <source src="/btc-sim.mp4" type="video/mp4" />
  </video>
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
      These days, I keep busy with gym training, outdoor activities, open water
      swimming, and Latin dance.
    </p>
  </div>
</section>
```

Notes:
- The `data-beyond-video` attribute is the JS hook the IO script in Task 6 will look for.
- `aria-hidden="true"` on the video and scrim makes them invisible to screen readers — the heading and copy convey all meaning.
- The old `.beyond-grid`, `.beyond-text`, `.beyond-media`, `<div class="beyond-grid">` wrapper, and the old single-paragraph copy are gone in this replacement.

- [ ] **Step 2: Run dev server and confirm markup parses**

```bash
cd "<repo-root>/pankkonen-dev"
npm run dev
```

Astro should hot-reload without errors. The Beyond section will look broken visually right now (no styles applied yet — Tasks 4 + 5 fix this). What to verify in this step is **no compile errors in the Astro output**.

- [ ] **Step 3: Commit markup change**

```bash
git add src/components/About.astro
git commit -m "feat(beyond): replace 50/50 grid with full-bleed structure + new copy"
```

---

## Task 4: Beyond — desktop full-bleed CSS

**Files:**
- Modify: `src/components/About.astro` `<style>` block — replace the existing `.beyond*` rules (currently lines 145–202) and the existing mobile branch's `.beyond*` rules

- [ ] **Step 1: Remove the old Beyond CSS**

In the `<style>` block, delete the entire current Beyond block (`/* ─── 05 · Beyond the Code layout ───────────────────────── */` down through `.beyond-gif { … }`) and the inner `.beyond-grid`, `.beyond-media` rules inside the `@media (max-width: 768px)` block at the bottom.

What remains in the `<style>` block after this step: the About rules + the global `[data-reveal]` + the About mobile rules. The Beyond rules are blank for now.

- [ ] **Step 2: Add the new desktop full-bleed Beyond CSS**

Insert this block where the old Beyond rules used to live (right after the About rules, before `[data-reveal]`):

```css
  /* ─── 05 · Beyond the Code — full-bleed video stage ─────── */
  .beyond {
    position: relative;
    width: 100vw;
    max-width: 100vw;
    margin-inline: calc(50% - 50vw);
    padding-inline: 0;
    height: min(80vh, 720px);
    overflow: hidden;
    isolation: isolate;
  }

  .beyond-video {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: center;
    z-index: 0;
    display: block;
  }

  .beyond-scrim {
    position: absolute;
    inset: 0;
    background: linear-gradient(
      180deg,
      rgba(8, 8, 10, 0.35) 0%,
      rgba(8, 8, 10, 0.55) 40%,
      rgba(8, 8, 10, 0.92) 85%,
      rgba(8, 8, 10, 0.96) 100%
    );
    z-index: 1;
    pointer-events: none;
  }

  .beyond-inner {
    position: relative;
    z-index: 2;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    padding-block-end: var(--space-10);
    gap: var(--space-3);
  }

  .beyond-inner .section-index {
    margin: 0;
  }

  .beyond-heading {
    font-size: var(--text-display-l);
    color: var(--text);
    line-height: 1.02;
    margin: var(--space-2) 0 var(--space-5);
  }

  .beyond-accent {
    color: var(--primary-soft);
    font-style: italic;
    font-weight: 300;
  }

  .beyond-para {
    color: var(--text-muted);
    font-size: var(--text-body-l);
    line-height: 1.65;
    max-width: 56ch;
    margin: 0;
  }
```

- [ ] **Step 3: Visual verification on desktop**

Run `npm run dev`. Open the page, scroll to section 05. Verify:
- The Beyond section spans the full viewport width (no centred 1280px column visible)
- Section height feels like ~80% of viewport on a typical laptop (~600 px on a 800-px-tall window)
- The video plays in the background (will be confirmed working in Task 6 — for now you should at least see the poster image)
- The bottom-up gradient darkens the lower ~60% of the section
- Heading + paragraphs sit in the bottom-left, inside the 1280px content column (`.beyond-inner` uses `.container`)
- Reveal animations on label / heading / paragraphs trigger as you scroll the section into view

If the heading bleeds into the very bottom edge, `padding-block-end: var(--space-10)` on `.beyond-inner` is your knob — increase to `var(--space-12)`.

- [ ] **Step 4: Commit desktop CSS**

```bash
git add src/components/About.astro
git commit -m "style(beyond): desktop full-bleed video stage with bottom-up scrim"
```

---

## Task 5: Beyond — mobile letterbox CSS

**Files:**
- Modify: `src/components/About.astro` `<style>` block — extend the existing `@media (max-width: 768px)` block

- [ ] **Step 1: Add mobile overrides inside the existing media query**

Inside the existing `@media (max-width: 768px) { … }` block (currently around lines 224–256), append the following. Keep the existing About rules in that block — they stay as-is.

```css
    /* Beyond — revert to natural document flow on mobile */
    .beyond {
      width: 100%;
      max-width: 100%;
      margin-inline: 0;
      padding-inline: var(--space-5);
      padding-block: var(--space-8);
      height: auto;
      overflow: visible;
    }

    .beyond-video {
      position: static;
      width: 100%;
      height: auto;
      aspect-ratio: 16 / 9;
      border-radius: 14px;
      box-shadow: 0 0 0 1px var(--rule);
      order: 2;
      object-fit: cover;
    }

    .beyond-scrim {
      display: none;
    }

    .beyond-inner {
      padding-block-end: 0;
      display: flex;
      flex-direction: column;
      gap: var(--space-4);
    }

    .beyond-inner .section-index { order: 0; }
    .beyond-heading { order: 1; margin-block: 0; }
    .beyond-inner .beyond-para:nth-of-type(1) { order: 3; }
    .beyond-inner .beyond-para:nth-of-type(2) { order: 4; }
```

The `order` values place the video as a 16:9 card between the heading and the first paragraph: `index → heading → video → para1 → para2`.

- [ ] **Step 2: Visual verification on mobile width**

In Chrome DevTools, switch to a mobile preset (iPhone 14 Pro / 390 × 844). Reload, scroll to section 05. Verify:
- Section is no longer full-bleed; it sits inside the normal horizontal padding
- Order top-to-bottom: `05 · Beyond the Code` label → heading → 16:9 video card → both paragraphs
- Video card has rounded corners and a thin border (the box-shadow ring)
- No dark scrim overlay visible
- Reveal animations on heading + paragraphs still trigger

- [ ] **Step 3: Commit mobile CSS**

```bash
git add src/components/About.astro
git commit -m "style(beyond): mobile letterbox card under 768px"
```

---

## Task 6: Video play/pause IntersectionObserver + reduced-motion handling

**Files:**
- Modify: `src/components/About.astro` — add a new scoped `<script>` block at the bottom of the file (after the existing `<style>` block)

- [ ] **Step 1: Add the scoped script**

After the closing `</style>` tag, append this block:

```astro
<script>
  // Gates the Beyond background video on viewport entry/exit:
  //   - Plays when the section is (about to be) visible
  //   - Pauses when fully off-screen (saves CPU + battery)
  //   - Respects prefers-reduced-motion: never starts the clip,
  //     leaves the poster image as the visible state
  const initBeyondVideo = () => {
    const video = document.querySelector<HTMLVideoElement>('[data-beyond-video]');
    if (!video) return;

    const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    if (reduceMotion) {
      // Strip autoplay so the browser doesn't begin playback even briefly,
      // and explicitly pause in case some attribute hot-path already kicked in.
      video.removeAttribute('autoplay');
      try { video.pause(); } catch {}
      return;
    }

    const section = video.closest('section');
    if (!section) return;

    const io = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            // play() returns a Promise that may reject under iOS autoplay
            // policy — swallow it; the user will still see the poster.
            const p = video.play();
            if (p && typeof p.catch === 'function') p.catch(() => {});
          } else {
            video.pause();
          }
        });
      },
      {
        rootMargin: '10% 0px 10% 0px',
        threshold: 0,
      }
    );

    io.observe(section);
  };

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', initBeyondVideo, { once: true });
  } else {
    initBeyondVideo();
  }
</script>
```

- [ ] **Step 2: Verify play/pause behaviour in DevTools**

Run `npm run dev`. Open the page in a fresh tab.

In DevTools console, with the page loaded:

```javascript
const v = document.querySelector('[data-beyond-video]');
v.paused; // before scrolling Beyond into view: should report true
```

Scroll until the Beyond section is on screen, then re-check:

```javascript
v.paused; // should now be false (video is playing)
```

Scroll back to Hero (Beyond fully off-screen), then:

```javascript
v.paused; // should report true again
```

If `v.paused` never flips to false: check the browser console for autoplay-policy errors. The most common cause on Chrome is the user not having interacted with the page yet — click anywhere once and re-test.

- [ ] **Step 3: Verify reduced-motion behaviour**

In DevTools, open Command Palette (Cmd-Shift-P / Ctrl-Shift-P) → "Show Rendering". In the Rendering panel, set "Emulate CSS media feature `prefers-reduced-motion`" to `reduce`. Reload the page.

Scroll to Beyond. Verify:
- Poster image (`btc-sim-poster.jpg`) is visible in the section background
- Console: `document.querySelector('[data-beyond-video]').paused` should report `true`
- The video does not play even when the section is on screen

Switch the rendering panel back to `no-preference` and reload — playback should resume.

- [ ] **Step 4: Verify mobile network strategy**

In DevTools, switch to a mobile preset (≤768 px viewport) and the Network tab. Filter by "Media". Reload the page and scroll into Beyond.

Verify only `btc-sim-mobile.webm` is requested (not `btc-sim.webm` or `btc-sim.mp4`). Switch back to a desktop viewport and reload — desktop request should hit `btc-sim.webm` first.

If both are downloaded on mobile: confirm the `media="(max-width: 768px)"` attribute is present on the first `<source>` element in Task 3's markup.

- [ ] **Step 5: Commit IO + reduced-motion script**

```bash
git add src/components/About.astro
git commit -m "feat(beyond): IO-gated video playback + reduced-motion poster fallback"
```

---

## Task 7: Build verification + final visual pass

**Files:**
- Read-only verification (no edits expected)

- [ ] **Step 1: Run a production build**

```bash
cd "<repo-root>/pankkonen-dev"
npm run build
```

Expected: build completes without errors. Astro should print the list of generated routes including `/`. No warnings about missing video files or invalid HTML.

- [ ] **Step 2: Preview the production build**

```bash
npm run preview
```

Open the printed URL. This serves the static build (different code path from `npm run dev`). Walk through the page top to bottom:

- Hero loads
- Section dividers animate as before
- About: 2 paragraphs, 2-column grid on desktop, single column on mobile width
- Beyond on desktop: full-bleed video plays, scrim darkens lower portion, text legible at bottom
- Beyond on mobile (DevTools mobile preset): letterbox card, all elements ordered correctly
- Reveal animations trigger in both directions (scroll down → up → down again, the existing replay policy)

- [ ] **Step 3: Lighthouse spot-check (optional, recommended)**

In DevTools → Lighthouse, run a Performance + Accessibility audit on the previewed page. Note the Performance score. The target from the spec is "no drop > 3 points vs the current main score." If it dropped more, the most likely cause is the desktop WebM being too large — re-encode with `-crf 30` (slightly lower quality) and re-check.

- [ ] **Step 4: Final commit if any tweaks were needed during verification**

If you adjusted CRF, scrim opacity, padding-block-end on `.beyond-inner`, etc.:

```bash
git add -A
git commit -m "polish(beyond): tune <whatever-you-adjusted>"
```

If nothing needed adjusting, this step is a no-op.

---

## Spec coverage check

| Spec section | Implemented in |
|--------------|----------------|
| 1.1 Visual concept (3A scrim treatment) | Task 4 |
| 1.2 Desktop layout (100vw, min(80vh, 720px), bottom-anchored) | Task 4 |
| 1.3 Scrim gradient stops | Task 4 |
| 1.4 Mobile letterbox layout | Task 5 |
| 1.5 Loop/muted/IO play/pause | Task 3 (attrs) + Task 6 (script) |
| 1.6 Reduced-motion poster | Task 6 |
| 1.7 ARIA + reading order | Task 3 (`aria-hidden`, heading hierarchy) |
| 2.1 Source clip | Task 1 step 1 |
| 2.2 Four encoded outputs | Task 1 steps 2–5 |
| 2.3 `<video>` element shape | Task 3 |
| 2.4 Old files overwritten in place | Task 1 steps 2–3 |
| 3.1 About copy + 2-col grid | Task 2 |
| 3.2 Beyond copy | Task 3 |
| 4 Markup overhaul | Task 3 |
| 5 Files affected | Task 1 (assets) + Tasks 2–6 (component) |
| 6 Performance/a11y checklist | Task 7 |
| 7 Out of scope | Honoured (no other components touched) |

---

## Out of scope for this plan

- Producing a CI script that re-encodes future video sources automatically (mentioned as an open question in spec §8 but explicitly deferred)
- Refactoring the global `[data-reveal]` IO into a shared utility (existing IO is fine; the video IO is a distinct concern with different config)
- Touching `dist.stale.*` cleanup (orthogonal)
