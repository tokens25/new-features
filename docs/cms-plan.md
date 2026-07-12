# Composable CMS — Content Model & Build Plan

A block-based content model for the DAZN "What's New" site, edited through **Sveltia CMS** (git-based, Decap-compatible) on GitHub + Vercel.

- **CMS:** Sveltia (git-based)
- **Backend:** GitHub · Vercel
- **Source of truth:** `content/features/*.md`
- **Status:** Schema & plan (proposal)

Interactive version: https://claude.ai/code/artifact/7388c9ae-0d3f-49a0-b657-49327c2fdfd7

---

## 01 · The model in one idea

Today `entries.js` gives every feature the same flat set of fields, and the polish lives in hand-tuned CSS. The composable model flips that:

- A small set of **shared identity fields** stays constant.
- The body becomes a **reorderable stack of blocks** the editor picks from.

Add a highlights rail when the story needs it; leave it out when it doesn't. The template renders whatever blocks are present, in order.

**Why this fits Sveltia:** Decap/Sveltia's `list` widget with `types` is a native variable-block editor — editors get an "Add section" menu, drag to reorder, and each block type shows only its own fields. The content model and the editing UI are the same object.

```
entry: my-clips
├─ name, label
├─ date
├─ market / devices / users
├─ theme: dark
└─ blocks:
     1. hero              (Hero)
     2. accordion         (New Features)
     3. stat_cards        (Closer Look)
     4. highlights        (Get the Highlights)
     5. gallery           (Made for Every Screen)
     6. how_it_works      (How It Works)
     7. statement         (Big Statement)
```

---

## 02 · Shared identity

Fields every feature carries, independent of which blocks it uses.

| Field | Purpose |
|---|---|
| `id` | URL slug — drives `feature.html?id=…` and the filename |
| `name` | Feature name, used in the hero eyebrow and landing grid |
| `label` | Update type — text ("New release") + kind (new / improvement / live / integration) for the pill colour |
| `date` | Release date shown in hero metadata |
| `market` · `devices` · `users` | Metadata chip lists — the audience vocabulary |
| `theme` | `light` or `dark` — recolours the whole page through the token set |
| `blocks` | The ordered section stack. Everything visual lives here. |

---

## 03 · The block library

Each block splits into **content** (what editors write) and **options** (how it presents). The presets currently buried in CSS become toggles.

### Feature-page blocks

| Block | Purpose | Content | Options |
|---|---|---|---|
| `hero` | Opening full-bleed band: product shot, headline, benefit line, metadata chips, CTA | headline, sub, cta, image, imageMobile, product | background (white/panel), focal point x/y, scrim (none/white/black), portrait-mobile |
| `accordion` | Expandable list of changes with a synced image that crossfades as items open | items[] (heading + body), images[], tip | crossfade tint (black/white), media aspect, show "Good to know" |
| `stat_cards` | Headline numbers that roll up like an odometer, each over a drawing arc gauge | cards[] (value + suffix + label), gauge % | odometer shuffle, gauge gradient |
| `highlights` | Horizontal, snap-scrolling cards — the most flexible block | cards[] (title + description), images[] | layout (overlay/below/inside), 3-up or 4-up, fill/contain + focal, square < 480px |
| `gallery` | 2×2 device grid that animates in from the four corners | caption, images[4] | per-tile fit (cover/contain), per-tile background |
| `gauges` | 270° arc meters that draw in and count up | gauges[] (target + % + label) | gradient, number format |
| `how_it_works` | Two side-by-side explainer cards: image over a short caption | cards[2] (title + body + image) | columns, media aspect |
| `statement` | Closing full-bleed image with one overlaid line — the sign-off | eyebrow, headline, lead, image | text position, scrim strength + tint |

### Landing-page blocks

| Block | Purpose | Content | Options |
|---|---|---|---|
| `feature_band` | Full-width stacked band on `index.html` — the top-billed updates | name, tagline, cta → feature, background, backgroundMobile | treatment (white photo / dark scrim), parallax, mobile = whole-card link |
| `tile` | Card in the 2-up landing grid, linking through to its feature page | tileName, tileTagline, image, link | card (light/dark), hover zoom |

---

## 04 · How it reads in Sveltia

The whole model is one collection with a variable-type `blocks` list.

```yaml
# public/admin/config.yml  (Sveltia reads the Decap config format)
backend:
  name: github
  repo: tokens25/new-features
  branch: second-main
media_folder: assets/img
public_folder: /assets/img

collections:
  - name: features
    folder: content/features
    create: true
    slug: '{{id}}'
    fields:
      - { name: id,    widget: string }
      - { name: name,  widget: string }
      - { name: theme, widget: select, options: [light, dark] }
      - { name: market, widget: list }
      # ---- the composable body ----
      - name: blocks
        label: Sections
        widget: list
        types:
          - name: hero
            widget: object
            fields:
              - { name: headline, widget: string }
              - { name: sub,      widget: text }
              - { name: image,    widget: image }
              - { name: imageMobile, widget: image, required: false }
              - { name: background,  widget: select, options: [white, panel] }
              - { name: scrim,       widget: select, options: [none, white, black] }
              - { name: focalY,      widget: number, required: false }
          - { name: accordion,    widget: object, fields: [ … ] }
          - { name: highlights,   widget: object, fields: [ … ] }
          - { name: gallery,      widget: object, fields: [ … ] }
          - { name: statement,    widget: object, fields: [ … ] }
```

---

## 05 · Migrating today's content

Nothing is thrown away — the current entries convert straight into block stacks.

A one-off script reads `window.DAZN_ENTRIES` and emits one `content/features/<id>.md` per feature. Today's flat fields map cleanly:

- `heroImage` → `hero`
- `sections[]` + `closerImages[]` → `accordion`
- `highlightItems[]` → `highlights`
- `galleryImages[]` → `gallery`
- `worksImages[]` → `how_it_works`
- `statementImage` → `statement`

The render layer barely changes: a tiny build step compiles the content files back into the array the templates already consume.

> **The real work isn't the CMS.** It's promoting the hand-tuned CSS — `object-position` crops, per-page scrims, aspect overrides — into block *options* the templates read from data. That's where composability actually pays off, and it's the bulk of the effort.

---

## 06 · Build plan

Four phases, front-loaded on making the templates fully data-driven.

| Phase | Work | Effort | Notes |
|---|---|---|---|
| **1 · Content migration** | Split `entries.js` into per-feature files + a build step that recompiles them. Site renders identically. | ~2–3 days | Pure refactor, low risk |
| **2 · Data-drive the polish** | Move focal points, scrims, aspect ratios and layout variants out of bespoke CSS into block option fields the templates consume. | ~1–2 weeks | **Core effort** |
| **3 · Sveltia config** | Define the collection and every block type; point media at `assets/img`; add the `/admin` route. | ~2–3 days | |
| **4 · Auth, preview & deploy** | GitHub sign-in (Sveltia handles OAuth directly), live preview on the real templates, editor live on Vercel. | ~1–2 days | |

**Total: roughly 3–4 weeks**, landing incrementally — after Phase 1 the site is file-per-feature; after Phase 3 editors can add and reorder blocks through the UI.

### Open decisions

- **Phase 2 scope:** everything editable, or the 80% that matters (leave the rarest CSS knobs hard-coded)?
- **Target branch:** commit content from the CMS to `second-main`, or a dedicated content branch with PR review?
