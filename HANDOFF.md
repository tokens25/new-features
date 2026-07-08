# DAZN "What's New" — Design & Build Handoff

A two-page, Apple-style product-updates site built with the DAZN design language.
Vanilla HTML/CSS/JS, no build step. This doc is a handoff for a designer picking the work up.

- **Repo:** https://github.com/tokens25/new-features.git (branch `main`)
- **Run locally:** any static server from the project root, e.g. `npx serve` or `python3 -m http.server`, then open `index.html`. (`serve.json` is included.)
- **Last updated:** 2026-07-07

---

## 1. The two pages

| File | Role | Theme |
|------|------|-------|
| `index.html` | Landing / "What's New" — video splash hero, three stacked full-width feature **bands**, then a 2-up grid of **tiles**, footer. | **Dark** (DAZN brand yellow) |
| `feature.html` | Apple-style feature **detail** template, data-driven per entry (`?id=<entry-id>`). | **Light** |
| `entries.js` | Single shared data source (`window.DAZN_ENTRIES`) that drives both pages. | — |

Both pages read the same `entries.js`, so **content and ordering are edited in one place**.

---

## 2. Content model (`entries.js`)

Each entry is one product update. Key fields:

```js
{
  id, name, tagline, ctaText,        // headline + benefit line + CTA label
  labelClass, labelText,             // update type, e.g. "New release", "Improvement"
  date,                              // "August 4"
  market, devices, users,            // arrays → hero metadata labels (feature page)
  title, desc, tip,                  // longer copy for the detail page
  sections: [{heading, text}, …],    // accordion + centered blocks
  product,                           // "DAZN app · Smart TV, Web"
  color, tileBg, tileBorder, icon    // per-entry accents
}
```

- **First 3 entries** render as the full-width **bands** on the landing page.
- **Entries 4–9** render as the **tiles** in the 2-up grid.
- **Reordering entries reorders the page.** (e.g. Watch Party ⇄ Offline Downloads was a simple array swap.)

**Metadata vocabularies** (feature-page hero labels, comma-joined per type):
- Date — the release date.
- Market — United Kingdom, Germany, Italy, Japan, Canada, United States, France.
- Devices — Mobile, Tablet, Desktop, TV.
- Users — Guest, Anonymous, Subscriber, Free User.

---

## 3. Design system

Follows the **DAZN design system** (skill/tokens). Never hardcode—use the token that matches.

**Landing page (dark)**
- Surfaces: `--surface-default #080E12` → `--surface-1 #0C1216` … `--surface-5 #2A3238` (lightest).
- Brand: `--brand-primary #F7FF1A` (yellow). Text: `--text-primary #FAFBFC`.
- `--grid-margin 64px`, `--max-width 1200px`.

**Feature page (light)**
- `--bg #FFFFFF`, `--panel #F5F5F7`, `--ink #1D1D1F`, `--ink-2 #6E6E73`.
- `--max 1120px`, page gutter `--gm 40px`.

**Buttons** — DAZN button form: 48px height, radius 8px, 16/16 weight 700, `'DAZN Oscine'`.
`.btn--solid` = white bg / black border → **yellow on hover**.

**Labels** — DAZN label spec (height 24px, radius 2px, uppercase Oscine Bold). The feature-page
hero metadata uses a softened variant: **regular case, `#ececec` bg, `#0C1216` text, 0 8px padding**.

---

## 4. Landing page (`index.html`)

### Splash hero
- Fixed **800px** tall, full-bleed looping muted video (`assets/video/…`), dark scrim, centered logo + "What's New".
- **On scroll (mirrors the feature hero):** narrows from full-bleed toward the wrap width (floored at **1440px** wide) and rounds corners 0→28px over the first 340px (`--fp` variable). Content drifts up + fades; video drifts down. rAF-throttled, respects `prefers-reduced-motion`.

### Feature bands (first 3 entries)
Min-height 640px, content bottom-aligned, **capped at 1800px and centered**; corners round to 28px once the viewport is wide enough that side margins appear (`@media (min-width: 1840px)`). Each has its own treatment:

| Band | Treatment |
|------|-----------|
| **Multi-View** | White band, row-of-phones image (`multiview-desktop.png`) at natural scale, `20%` from top. Subtle photo parallax (`--mv-shift`). |
| **Faster start-up** | Dark photo backdrop (`faster-startup.png`) with a transparent→`#0C161C` scrim so text stays legible. Background + text drift apart on scroll. |
| **Win Probability** | Dark product-shot backdrop (`win-probability.png`) with a bottom scrim, light text. *(Earlier this band had a decorative floating-scorecards parallax, §6 — its CSS/JS is still in the file but no longer rendered.)* |

### Highlight tiles (2-up grid, entries 4–9)
The grid section (`#highlights`) is capped at **1440px** and centered. Uniform cards, min-height 520px, radius 20px. Two variants:
- **Light tiles** (`card--light`): white bg, dark text — used where the source image is on white.
- **Dark tiles** (default `--surface-5 #2A3238`): light text — used where the image is dark.

Each tile carries a **background image** on a `.card::before` layer (each id gets a `card--<id>` hook), with only the base color on the card itself:

| Tile | Image | Layout |
|------|-------|--------|
| Live Key Moments | `live-key-moments.png` (dark) | cover + bottom scrim |
| Seamless Casting | `seamless-casting.png` (white) | image top, text bottom |
| 4K HDR | `4k-hdr.png` (white) | image top (76%), text bottom |
| My Clips *(id `watch-party-beta`)* | `watch-party.png` (dark) | cover + bottom scrim |
| Match Chat | `match-chat.png` (dark) | cover + bottom scrim (dark tile) |
| Offline Downloads | `offline-downloads.png` (white) | image top, text bottom |

- **Clickable:** each tile is an `<a href="feature.html?id=…">` (whole tile links to its feature page). No "Learn more" buttons.
- **Hover:** the `::before` image layer zooms (`scale(1.08)`) and the text (`.card__body`) lifts (`translateY(-12px)`); both disabled under reduced-motion.
- **Reveal:** cards slide up + fade in, staggered left→right per row, when scrolled into view (gated on `.reveal` so they're visible without JS).

### Footer
Simplified to copyright + Terms / Privacy / Cookie links, white background.

---

## 5. Feature detail page (`feature.html`)

Section order: **Hero → "New Features" accordion → "Closer look" stat cards → Gallery → "Get the highlights" carousel → "By the numbers" gauges → "How {name} works" → Big statement.**

**Motion (all scroll-driven, rAF-throttled, reduced-motion safe):**
- **Hero** narrows to the 1120px wrap + rounds corners on scroll (`--fp`). Layered content parallax: text (−0.42×) and price pill (−0.12×) translate up and **fade — held at full opacity for the first quarter of the hero, then eased out** so content lingers (doesn't disappear too early).
- **Hero metadata:** the update type ("New release", etc.) is a clickable link back to the landing page, followed by Date / Market / Devices / Users labels.
- **Stat cards** slide in (left / bottom / right) and count up on enter, reverse on leave.
- **Gallery** tiles arrive from the four corners; reversible.
- **Gauges** (§7) — arc draw + count-up, triggered at viewport center.

---

## 6. Win Probability floating scorecards (landing) — *currently disabled*

> The Win Probability band now uses a product-shot image background (§4). The scorecards below are **no longer rendered** (the injection was removed) but the CSS/JS remains in `index.html` — restore by re-adding the `scoreCluster` injection in the band template, or delete it if the image is permanent.

A decorative depth backdrop that previously sat behind the Win Probability band.
- 9 live-style **scorecards** (flags, scores, minute, goal/card events) layered with opacity + blur for depth. Classes namespaced `wp-*` so they never collide with the real tiles.
- Cards sit on a **fixed 1280px canvas**, centered — so they keep their pixel size and simply **crop like an image** on smaller viewports instead of shrinking.
- **Scroll parallax = depth:** each card drifts by `norm × data-depth × 620px`. Crisp front cards barely move (~27–48px of travel); faint blurred back cards drift far (~130–150px). Plus a small depth-keyed rotation drift.

---

## 7. Gauges ("By the numbers", feature page)

- 270° SVG arc (`stroke-dasharray: 245 82`, rotated 135°), gradient `#8923A0 → #F1A322 → #F7FF1A`.
- Numbers count up (formatted `de-DE`, e.g. `66.000`).
- **Trigger:** IntersectionObserver with `rootMargin: -40% 0 -40%` — the arc + count-up run **only when a gauge reaches the middle of the viewport** (central ~20% band), once each.

---

## 8. Assets

```
assets/
  img/    multiview-desktop.png, multiview-mobile.png, hero-multiview.png,
          faster-startup.png, win-probability.png, live-key-moments.png,
          seamless-casting.png, 4k-hdr.png, watch-party.png (My Clips),
          match-chat.png, offline-downloads.png
  video/  video-hero-720.mp4 (desktop splash), video-hero-vertical.mp4 (mobile splash)
```
Tile/band images are AI-generated match mockups on either white (→ light tile) or dark (→ dark tile + scrim) backgrounds. Swap an image by dropping a same-named file in `assets/img/` and tuning `background-size`/`-position` on that tile's `.card--<id>` rule.

---

## 9. Conventions & gotchas

- **Robust-by-default motion:** hidden/animated states are gated behind a `.reveal` class and guarded by `prefers-reduced-motion`; content is fully visible without JS.
- **No horizontal scroll:** transform-based effects are contained with `overflow: hidden`/`clip`.
- **Namespacing:** the scorecards use `wp-*` classes to avoid clashing with the global `.card` tiles.
- **One source of truth:** copy, ordering, dates, and per-entry metadata all live in `entries.js`.
- **Design-language reference:** DAZN tokens (surfaces, brand yellow, button/label specs) — match existing usage rather than introducing new values.

---

*Questions on any specific interaction or token choice — the git history is granular (one commit per change) and messages describe the intent.*
