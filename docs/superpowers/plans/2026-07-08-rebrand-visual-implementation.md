# Rebrand Visual (Fase 1) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the theme's cream/gold hardcoded color scheme with a white/black/brand-fuchsia token system, and add the missing About Us page.

**Architecture:** A single Liquid snippet defines all color values as CSS custom properties, rendered once in `theme.liquid`. Every section's inline `<style>` block is edited to reference `var(--token-name)` instead of literal hex values, using a fixed, deterministic hex→token mapping (below) so the change is mechanical and repeatable. A new section + template add the About Us page.

**Tech Stack:** Shopify Liquid theme, no build step, no JS framework, no existing test suite. Verification is `shopify theme check` (schema/lint) plus manual visual review via `shopify theme dev` (local dev theme, safe sandbox — never edit the published theme directly).

## Global Constraints

- Do not change any copy, text content, section structure, or functionality — this is a color-only + one new page change (per spec `docs/superpowers/specs/2026-07-08-rebrand-visual-design.md`).
- `about-banner.liquid` keeps its full background image + overlay treatment — do not flatten it to a solid color.
- No scroll-driven color-morph effect anywhere (explicitly out of scope per spec).
- All work happens in a Shopify dev theme (`shopify theme dev`), never pushed straight to the live theme.
- Logo replacement and favicon image are gated on assets the client has not sent yet — the plan implements everything around them (schema, markup, `<link>` tags) but the actual image swap is a follow-up task once the file arrives.

## Color Token Reference (used in every task below)

Defined once in Task 1, referenced by name in every later task:

```
--color-bg: #FFFFFF
--color-fg: #111111
--color-fg-muted: #6B6B6B
--color-border: #E5E5E5
--color-accent: #CB6CE6
--color-accent-solid: #B125D9
--color-accent-hover: #BC41DF
--color-accent-active: #9920BC
```

## Master Hex → Token Mapping Rule

Apply this exact rule to every literal hex color found in a section's `<style>` block, unless the task below says otherwise for that file:

| Literal hex(es) found | Replace with | Rule |
|---|---|---|
| `#FDFBF5`, `#FAFAF8`, `#FDFCF8`, `#F0ECDF`, `#F5F5F0`, `#F0EDE6`, `#F5F5F3`, `#FFF3E8`, `#E0DCD1` (only when used as a `background`/`background-color` value) | `var(--color-bg)` | cream section backgrounds → white |
| `#6B550E` used as `background` or `background-color` | `var(--color-accent-solid)` | solid fill (button/hover fill with white text on top) |
| `#6B550E` used as `color`, `border-color`, `border`, `box-shadow`, `outline`, or inside `rgba(107, 85, 14, ...)` | `var(--color-accent)` | text/border/icon/focus-ring uses |
| `#5a480c` | `var(--color-accent-active)` | manually-darkened gold hover/active states |
| `#2C2C2C`, `#1a1a1a` | `var(--color-fg)` | dark text |
| `#6D6C6C`, `#666`, `#888`, `#888888` | `var(--color-fg-muted)` | secondary/muted text |
| `#e8e4d9`, `#d4d0c5`, `#EBEBE5`, `#eeeeec`, `#E0DCD1` (only when used as `border` / `border-color` / gradient stop, not background) | `var(--color-border)` | borders, dividers, subtle gradients |
| `#C9A961`, `#8B7355`, `#4A3A0A` | `var(--color-accent)` | old secondary-gold accents (ratings, decorative text) |
| `#999`, `#bbb` | `var(--color-fg-muted)` | disabled/low-emphasis text |
| `#fff`, `#FFFFFF`, `#000`, `#000000` | **leave unchanged** | already correct brand white/black (card fills, overlays, button text) |
| `#c44`, `#c0392b`, `#2e7d32`, `#e8f5e9` | **leave unchanged** | functional error/success colors, out of scope |

This table is exhaustive for every hex value that appears in the theme today (confirmed by `grep -noE "#[0-9a-fA-F]{3,8}" sections/*.liquid` during the design review). If a task's file grep turns up a hex not listed here, stop and flag it — do not guess.

---

### Task 1: Create the shared color token snippet and wire it into the theme

**Files:**
- Create: `snippets/theme-tokens.liquid`
- Modify: `layout/theme.liquid` (add render call inside `<head>`)

**Interfaces:**
- Produces: CSS custom properties `--color-bg`, `--color-fg`, `--color-fg-muted`, `--color-border`, `--color-accent`, `--color-accent-solid`, `--color-accent-hover`, `--color-accent-active`, available globally on `:root` to every section's inline `<style>` block for the rest of this plan.

- [ ] **Step 1: Create the token snippet**

Create `snippets/theme-tokens.liquid`:

```liquid
<style>
  :root {
    --color-bg: #FFFFFF;
    --color-fg: #111111;
    --color-fg-muted: #6B6B6B;
    --color-border: #E5E5E5;
    --color-accent: #CB6CE6;
    --color-accent-solid: #B125D9;
    --color-accent-hover: #BC41DF;
    --color-accent-active: #9920BC;
  }
</style>
```

- [ ] **Step 2: Render it in the theme layout**

In `layout/theme.liquid`, find the line:

```liquid
      <link rel="preconnect" href="https://fonts.shopifycdn.com" crossorigin>
```

Add immediately before it:

```liquid
      {% render 'theme-tokens' %}
```

- [ ] **Step 3: Start the dev theme and verify the tokens load**

Run: `shopify theme dev`

Open the printed preview URL, open browser DevTools → Elements → `<html>` → Computed styles on `:root`, and confirm `--color-accent` resolves to `rgb(203, 108, 230)` (i.e. `#CB6CE6`).

Expected: the custom properties are present; page still looks unchanged (cream/gold) since no section references them yet.

- [ ] **Step 4: Lint**

Run: `shopify theme check --path .`
Expected: no new errors introduced (pre-existing warnings unrelated to this change are fine).

- [ ] **Step 5: Commit**

```bash
git add snippets/theme-tokens.liquid layout/theme.liquid
git commit -m "Add shared color token snippet for rebrand"
```

---

### Task 2: Recolor header and footer

**Files:**
- Modify: `sections/header.liquid`
- Modify: `sections/footer.liquid`

**Interfaces:**
- Consumes: tokens from Task 1 (`var(--color-*)`).

- [ ] **Step 1: Replace hex values in `sections/header.liquid`**

Apply the Master Hex → Token Mapping Rule to every occurrence found by:

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/header.liquid
```

Known occurrences to replace (per the rule table): `#6B550E` → `var(--color-accent)` everywhere it appears as `color`/`border-color` in this file (links, hover state, focus). `#000`/`#fff` stay unchanged.

Use the Edit tool with `replace_all: true` for each hex string, scoped to this file.

- [ ] **Step 2: Replace hex values in `sections/footer.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/footer.liquid
```

Apply the rule table: `#2C2C2C` → `var(--color-fg)`, `#FDFCF8`/`#F0EDE6` → `var(--color-bg)`, `#E5E0D6` → `var(--color-border)`, `#6B550E` → `var(--color-accent)` (all uses in this file are `color`/hover text, not a fill), `#888` → `var(--color-fg-muted)`, `#FFFFFF` unchanged.

- [ ] **Step 3: Visual check**

With `shopify theme dev` still running, reload the preview. Confirm: header background is white/transparent with black text and fucsia hover/active link states; footer background is white, footer text is black/gray, and the "SNC Designs" credit link still renders (unchanged, just recolored if it inherited a token).

- [ ] **Step 4: Lint**

Run: `shopify theme check --path .`
Expected: no new errors.

- [ ] **Step 5: Commit**

```bash
git add sections/header.liquid sections/footer.liquid
git commit -m "Recolor header and footer to brand tokens"
```

---

### Task 3: Recolor hero-banner, product-carousel, featured-product

**Files:**
- Modify: `sections/hero-banner.liquid`
- Modify: `sections/product-carousel.liquid`
- Modify: `sections/featured-product.liquid`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Recolor `sections/hero-banner.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/hero-banner.liquid
```

Apply the rule: `#1a1a1a` → `var(--color-fg)`, `#f0f0f0` → `var(--color-border)` (used as a light divider/border tone), `#000000`/`#fff` unchanged (video overlay + button text). Do **not** touch the `overlay_color`/`overlay_opacity` schema settings — those are merchant-editable and already default to black, which matches the brand.

- [ ] **Step 2: Recolor `sections/product-carousel.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/product-carousel.liquid
```

Apply the rule: cream backgrounds (`#FDFBF5`, `#F0ECDF`) → `var(--color-bg)`; `#6B550E` as `background` (arrow hover fill) → `var(--color-accent-solid)`; `#6B550E` as `color`/`border-color` (title, arrow icon, focus outline, and inside `rgba(107, 85, 14, 0.15)` — replace the whole `rgba(...)` with `var(--color-accent)` at 15% opacity via `color-mix(in srgb, var(--color-accent) 15%, transparent)`) → `var(--color-accent)`; `#6D6C6C` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` (gradient/border) → `var(--color-border)`; `#fff` unchanged.

- [ ] **Step 3: Recolor `sections/featured-product.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/featured-product.liquid
```

Apply the rule: `#FAFAF8`/`#F0ECDF` → `var(--color-bg)`; `#6B550E` per the `background` vs `color` split above; `#6D6C6C` → `var(--color-fg-muted)`; `#FFFFFF` unchanged.

- [ ] **Step 4: Visual check**

Reload the dev preview on the homepage. Confirm: hero unchanged visually except any text/border tweaks; both product carousels ("Candles", "Reed Diffusers") show white background, fucsia titles/hover arrows; the "LOVE JONES" featured product block shows white background with fucsia badge/accent and black body text.

- [ ] **Step 5: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 6: Commit**

```bash
git add sections/hero-banner.liquid sections/product-carousel.liquid sections/featured-product.liquid
git commit -m "Recolor hero, product carousel, and featured product sections"
```

---

### Task 4: Recolor PLP, list-collections, main-product

**Files:**
- Modify: `sections/plp.liquid`
- Modify: `sections/list-collections.liquid`
- Modify: `sections/main-product.liquid`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Recolor `sections/plp.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/plp.liquid
```

Apply the rule: `#F0ECDF`/`#FDFBF5` → `var(--color-bg)`; `#6B550E` split by `background` vs `color`/`border`; `#FFFFFF` unchanged; `#E0DCD1` → `var(--color-border)` when it's a `border`, `var(--color-bg)` when it's a `background`; `#2C2C2C` → `var(--color-fg)`; `#6D6C6C` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`.

- [ ] **Step 2: Recolor `sections/list-collections.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/list-collections.liquid
```

Apply the rule: `#FDFBF5`/`#F0ECDF` → `var(--color-bg)`; `#6B550E` split by property as above; `#2C2C2C` → `var(--color-fg)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#6D6C6C` → `var(--color-fg-muted)`; `#fff` unchanged.

- [ ] **Step 3: Recolor `sections/main-product.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/main-product.liquid
```

Apply the rule: `#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property (note: `#5a480c` also appears here → `var(--color-accent-active)`); `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#6D6C6C` → `var(--color-fg-muted)`; `#999` → `var(--color-fg-muted)`; `#fff` unchanged.

- [ ] **Step 4: Visual check**

Reload dev preview: visit a collection page (list-collections), a category listing (plp), and a product page (main-product). Confirm white backgrounds, fucsia buttons/prices/accordion accents, black/gray text throughout, no leftover cream anywhere.

- [ ] **Step 5: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 6: Commit**

```bash
git add sections/plp.liquid sections/list-collections.liquid sections/main-product.liquid
git commit -m "Recolor PLP, collection list, and product page sections"
```

---

### Task 5: Recolor cart, contact-hero, contact-form, favoritos

**Files:**
- Modify: `sections/cart.liquid`
- Modify: `sections/contact-hero.liquid`
- Modify: `sections/contact-form.liquid`
- Modify: `sections/favoritos.liquid`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Recolor `sections/cart.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/cart.liquid
```

Apply the rule: `#FDFBF5`/`#F0ECDF` → `var(--color-bg)`; `#6B550E` split by property; `#5a480c` → `var(--color-accent-active)`; `#6D6C6C` → `var(--color-fg-muted)`; `#999` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#c44` unchanged (destructive/remove-item color, out of scope); `#fff` unchanged.

- [ ] **Step 2: Recolor `sections/contact-hero.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/contact-hero.liquid
```

Apply the rule: `#000000`/`#fff` unchanged (image overlay + heading text, already brand black/white).

- [ ] **Step 3: Recolor `sections/contact-form.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/contact-form.liquid
```

Apply the rule: `#FAFAF8` → `var(--color-bg)`; `#6B550E` split by property; `#5a480c` → `var(--color-accent-active)`; `#1a1a1a` → `var(--color-fg)`; `#888`/`#666` → `var(--color-fg-muted)`; `#F5F5F3`/`#eeeeec` → `var(--color-border)`; `#fff` unchanged; `#2e7d32`/`#e8f5e9` unchanged (success-message state, out of scope).

- [ ] **Step 4: Recolor `sections/favoritos.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/favoritos.liquid
```

Apply the rule: `#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property; `#5a480c` → `var(--color-accent-active)`; `#6D6C6C` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#c44` unchanged; `#fff` unchanged.

- [ ] **Step 5: Visual check**

Reload dev preview: add a product to cart and open the cart page; visit `/pages/contact`; visit `/pages/favoritos` (or wherever `favoritos` is templated). Confirm white backgrounds, fucsia CTAs/links, black/gray text, form inputs still functional (submitting the contact form still shows the existing success message green — untouched).

- [ ] **Step 6: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 7: Commit**

```bash
git add sections/cart.liquid sections/contact-hero.liquid sections/contact-form.liquid sections/favoritos.liquid
git commit -m "Recolor cart, contact, and favorites sections"
```

---

### Task 6: Recolor scrolling-gallery, testimonials-masonry, product-recommendations, comixub

**Files:**
- Modify: `sections/scrolling-gallery.liquid`
- Modify: `sections/testimonials-masonry.liquid`
- Modify: `sections/product-recommendations.liquid`
- Modify: `sections/comixub.liquid`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Recolor `sections/scrolling-gallery.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/scrolling-gallery.liquid
```

Apply the rule: `#F0ECDF` (the `background: var(--bg-primary, #F0ECDF);` fallback) → `var(--color-bg)`; `#6B550E`, `#8B7355`, `#4A3A0A` (all decorative gold text tones) → `var(--color-accent)`; `#6D6C6C` → `var(--color-fg-muted)`. Per the spec, do **not** add any scroll-triggered color-morph — only replace the flat literal colors with tokens. The section stays `disabled: true` on the homepage (no template change needed).

- [ ] **Step 2: Recolor `sections/testimonials-masonry.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/testimonials-masonry.liquid
```

Apply the rule: `#F0ECDF`/`#F5F5F0`/`#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property; `#C9A961` (rating stars/laurel) → `var(--color-accent)`; `#EBEBE5`/`#bbb` → `var(--color-border)`; `#6D6C6C` → `var(--color-fg-muted)`; `#fff` unchanged.

- [ ] **Step 3: Recolor `sections/product-recommendations.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/product-recommendations.liquid
```

Apply the rule: `#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property; `#6D6C6C` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#fff` unchanged.

- [ ] **Step 4: Recolor `sections/comixub.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/comixub.liquid
```

Apply the rule: `#FDFCF8` → `var(--color-bg)`; `#6B550E` split by property; `#1a1a1a`/`#2C2C2C` → `var(--color-fg)`; `#888`/`#888888` → `var(--color-fg-muted)`; `#6D6C6C` → `var(--color-fg-muted)`; `#FFF3E8` → `var(--color-bg)` (light tint background used for a notice box — flatten to white per "no cream anywhere" rule); `#FFFFFF`/`#000000` unchanged; `#c0392b` unchanged (error state, out of scope). Note: `comixub.liquid` already exposes several `"type": "color"` schema settings (lines ~330–481) — leave those schema field *definitions* alone; only change the hardcoded CSS defaults/fallbacks that duplicate the old palette.

- [ ] **Step 5: Visual check**

Reload dev preview: check the (disabled, so preview via theme editor "Add section" or temporarily enable in a duplicate dev-only template) scrolling-gallery and testimonials-masonry render white/fucsia; visit the coming-soon page (`page.comixub.json`) and confirm white background, fucsia CTA button, black heading text.

- [ ] **Step 6: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 7: Commit**

```bash
git add sections/scrolling-gallery.liquid sections/testimonials-masonry.liquid sections/product-recommendations.liquid sections/comixub.liquid
git commit -m "Recolor scrolling gallery, testimonials, recommendations, and coming-soon sections"
```

---

### Task 7: Recolor about-banner (text/controls only — image stays)

**Files:**
- Modify: `sections/about-banner.liquid`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Inspect current hex usage**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/about-banner.liquid
```

Expected: only `#fff` and `#000000` (overlay color default + heading/body text color). Per the spec, **do not** change the section's background-image + overlay treatment — it stays a full-bleed photo. Leave both values unchanged; there is nothing to recolor here since it already uses brand black/white. If any other hex appears beyond these two, apply the Master Mapping Rule to it only.

- [ ] **Step 2: Confirm no changes needed / commit if any were made**

If Step 1 found only `#fff`/`#000000`, skip to Task 8 — no commit needed for this task. Otherwise:

```bash
git add sections/about-banner.liquid
git commit -m "Recolor about-banner text controls (image treatment unchanged)"
```

---

### Task 8: Create the About Us page and editorial section

**Files:**
- Create: `sections/about-editorial.liquid`
- Create: `templates/page.about.json`

**Interfaces:**
- Consumes: tokens from Task 1 (`var(--color-bg)`, `var(--color-fg)`, `var(--color-fg-muted)`, `var(--color-accent)`).
- Produces: a new page type `about-editorial` renderable via any `templates/page.*.json` that lists it in `order`.

- [ ] **Step 1: Create the section file**

Create `sections/about-editorial.liquid`:

```liquid
<section class="about-editorial" data-section-id="{{ section.id }}">
  <div class="about-editorial__media">
    {% if section.settings.image != blank %}
      {{ section.settings.image | image_url: width: 1400 | image_tag:
        loading: 'lazy',
        class: 'about-editorial__image',
        alt: section.settings.image.alt | escape
      }}
    {% endif %}
  </div>
  <div class="about-editorial__content">
    {% if section.settings.eyebrow != blank %}
      <p class="about-editorial__eyebrow">{{ section.settings.eyebrow }}</p>
    {% endif %}
    <h1 class="about-editorial__title">{{ section.settings.title | default: 'Our Story' }}</h1>
    <div class="about-editorial__body">
      {{ section.settings.body }}
    </div>
  </div>
</section>

<style>
  .about-editorial {
    background: var(--color-bg);
    display: grid;
    grid-template-columns: 1fr;
    gap: 0;
  }

  .about-editorial__media {
    width: 100%;
  }

  .about-editorial__image {
    width: 100%;
    height: auto;
    display: block;
  }

  .about-editorial__content {
    max-width: 680px;
    margin: 0 auto;
    padding: 64px 24px 96px;
  }

  .about-editorial__eyebrow {
    font-size: 13px;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: var(--color-accent);
    font-weight: 600;
    margin: 0 0 16px;
  }

  .about-editorial__title {
    font-family: var(--font-helvetica-world), Arial, sans-serif;
    font-size: 40px;
    color: var(--color-fg);
    margin: 0 0 32px;
    text-transform: uppercase;
    letter-spacing: 2px;
  }

  .about-editorial__body {
    font-size: 17px;
    line-height: 1.8;
    color: var(--color-fg-muted);
  }

  .about-editorial__body p {
    margin: 0 0 24px;
  }

  @media (min-width: 900px) {
    .about-editorial {
      grid-template-columns: 1.1fr 0.9fr;
      align-items: center;
    }

    .about-editorial__media,
    .about-editorial__content {
      height: 100%;
    }

    .about-editorial__content {
      padding: 96px 64px;
      margin: 0;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }

    .about-editorial__image {
      height: 100%;
      object-fit: cover;
    }
  }
</style>

{% schema %}
{
  "name": "About Editorial",
  "settings": [
    {
      "type": "image_picker",
      "id": "image",
      "label": "Image"
    },
    {
      "type": "text",
      "id": "eyebrow",
      "label": "Eyebrow text",
      "default": "Our Story"
    },
    {
      "type": "text",
      "id": "title",
      "label": "Title",
      "default": "Our Story"
    },
    {
      "type": "richtext",
      "id": "body",
      "label": "Body text",
      "default": "<p>Serenity Candle Haus began as a passion project fueled by a desire to stay creative. Our candles are inspired by intentional moments and thoughtfully crafted to bring warmth, comfort, and a sense of serenity into every space.</p><p>Hand-poured with a premium soy-coconut wax blend and phthalate-free fragrances, each candle is designed to help you slow down, unwind, and create a vibe that feels uniquely yours.</p><p>Light the candle. Set the mood. Create the vibe.</p>"
    }
  ],
  "presets": [
    {
      "name": "About Editorial"
    }
  ]
}
{% endschema %}
```

- [ ] **Step 2: Create the page template**

Create `templates/page.about.json`:

```json
{
  "sections": {
    "about_editorial": {
      "type": "about-editorial",
      "settings": {
        "eyebrow": "Our Story",
        "title": "About Us"
      }
    }
  },
  "order": [
    "about_editorial"
  ]
}
```

Note: `body` and `image` are intentionally left out of this settings block so the section's schema defaults populate them (the reused copy). The client can swap the image and expand the text later from the theme editor without touching code.

- [ ] **Step 3: Verify the page resolves**

In the Shopify admin (or via the dev theme preview), go to **Online Store → Pages** and confirm a page exists with the handle `about` (create one if it doesn't exist yet, assigning it the `page.about` template — a `templates/page.about.json` file alone does not create the page record, it only makes the template available to assign). Then visit `/pages/about` on the dev preview URL.

Expected: white background, eyebrow "OUR STORY" in fucsia, "ABOUT US" heading in black, body paragraphs in muted gray, image on one side (or on top on mobile).

- [ ] **Step 4: Confirm the header link now resolves**

Reload any page, open the mobile menu, click "About" (`sections/header.liquid:123`, `href="/pages/about"`).

Expected: navigates to the new page instead of a 404 or empty default page.

- [ ] **Step 5: Lint**

Run: `shopify theme check --path .`
Expected: no schema errors on the new section.

- [ ] **Step 6: Commit**

```bash
git add sections/about-editorial.liquid templates/page.about.json
git commit -m "Add About Us editorial page and section"
```

---

### Task 9: Add favicon plumbing (image swap pending client asset)

**Files:**
- Modify: `layout/theme.liquid`
- Modify: `config/settings_schema.json`

**Interfaces:**
- Consumes: a merchant-uploaded image via the new `favicon` setting.
- Produces: a `<link rel="icon">` tag that resolves once the client uploads the emblem asset — no visual change until then, but the wiring is complete and testable independently of the missing file.

- [ ] **Step 1: Add a favicon setting to the theme settings schema**

Open `config/settings_schema.json` and add a new setting group (as its own object in the top-level array):

```json
{
  "name": "Favicon",
  "settings": [
    {
      "type": "image_picker",
      "id": "favicon",
      "label": "Favicon image",
      "info": "Square image, at least 32x32px. Recommended: the \"S.\" emblem mark."
    }
  ]
}
```

- [ ] **Step 2: Render the favicon link in the theme layout**

In `layout/theme.liquid`, inside `<head>`, add (near the other `<link>` tags):

```liquid
      {% if settings.favicon %}
        <link rel="icon" type="image/png" href="{{ settings.favicon | image_url: width: 32 }}">
      {% endif %}
```

- [ ] **Step 3: Verify with a placeholder image**

Since the client's asset hasn't arrived yet, temporarily upload any square PNG through **Theme editor → Theme settings → Favicon** on the dev theme to confirm the `<link>` tag renders with a real URL (inspect page source, search for `rel="icon"`). Then remove the placeholder image from the setting so the live site doesn't ship a wrong icon.

Expected: `<link rel="icon" ...>` appears in page source only when `settings.favicon` is set; absent otherwise (no broken icon request).

- [ ] **Step 4: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 5: Commit**

```bash
git add layout/theme.liquid config/settings_schema.json
git commit -m "Add favicon setting and link tag (pending client asset)"
```

- [ ] **Step 6 (follow-up, blocked on client asset): Upload the real favicon**

Once the client sends the "S." emblem file: crop/export it as a square PNG (recommend 512x512 master, Shopify will serve resized via `image_url`), upload it through **Theme editor → Theme settings → Favicon**, and visually confirm the browser tab icon updates on the dev preview before publishing.

---

## Final End-to-End Verification

- [ ] Run `shopify theme check --path .` one last time across the whole theme — expect zero new errors compared to the pre-rebrand baseline.
- [ ] Run `grep -rnoE "#(FDFBF5|FAFAF8|FDFCF8|F0ECDF|F5F5F0|F0EDE6|F5F5F3|FFF3E8|6B550E|5a480c|C9A961|8B7355|4A3A0A)" sections/*.liquid` — expect **no matches** (every old brand color literal has been replaced with a token).
- [ ] Walk the dev preview end to end: home → collection list → PLP → PDP → cart → contact → favorites → about → coming-soon. Confirm white background and fucsia/black/gray treatment everywhere except `hero-banner` (video/overlay) and `about-banner` (photo/overlay), which intentionally keep their image-based dark treatment.
- [ ] Confirm the header "About" link works and the new page has real content.
- [ ] Confirm favicon `<link>` renders when a favicon is set (still pending client's actual image).
- [ ] Do **not** publish the dev theme yet — hand back to the client for visual review on the preview URL before promoting to live.
