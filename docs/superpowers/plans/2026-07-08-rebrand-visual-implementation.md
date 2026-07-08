# Rebrand Visual (Fase 1) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the theme's cream/gold hardcoded color scheme with a white/black/brand-fuchsia token system, and add the missing About Us page.

**Architecture:** The theme already has a shared token snippet, `snippets/css-variables.liquid`, rendered in all three layouts (`theme.liquid`, `coming-soon.liquid`, `password.liquid`). Rather than adding a second, competing token snippet, this plan retires the old cream/gold values inside that existing snippet and adds the new brand tokens alongside them. Every section's inline `<style>` block, plus the global `assets/critical.css`, plus saved color values in template/group JSON files, are then edited to reference the new tokens instead of literal hex — using a fixed, deterministic hex→token mapping (below) so the change is mechanical and repeatable.

**Tech Stack:** Shopify Liquid theme, no build step, no JS framework, no existing test suite. Verification is `shopify theme check` (schema/lint, meaningful mainly for Liquid/JSON edits) plus a set of `grep` regression checks (the real gate for the CSS-only tasks) plus manual visual review via `shopify theme dev` (local dev theme, safe sandbox — never edit the published theme directly).

## Global Constraints

- Do not change any copy, text content, section structure, or functionality — this is a color-only + one new page change (per spec `docs/superpowers/specs/2026-07-08-rebrand-visual-design.md`).
- `about-banner.liquid` keeps its full background image + overlay treatment — do not flatten it to a solid color.
- No scroll-driven color-morph effect anywhere (explicitly out of scope per spec).
- All work happens in a Shopify dev theme (`shopify theme dev`), never pushed straight to the live theme.
- Logo replacement and favicon image are gated on assets the client has not sent yet — the plan implements everything around them (schema, markup, `<link>` tags) but the actual image swap is a follow-up task once the file arrives.
- **This plan was reviewed by a second model (Fable 5) against the actual theme source before execution.** That review found the theme has an *existing* token system and a global stylesheet that a naive per-section grep would miss entirely, silently leaving the site cream/gold after every task reported success. This plan's Task 1, Task 2, and the JSON-migration steps embedded in later tasks exist specifically to close those gaps — do not skip them or treat them as optional cleanup.

## Color Token Reference (used in every task below)

Added to `snippets/css-variables.liquid` in Task 1, referenced by name in every later task:

```
--color-bg: #FFFFFF
--color-fg: #111111
--color-fg-muted: #6B6B6B
--color-border: #E5E5E5
--color-accent: #CB6CE6
--color-accent-solid: #B125D9
--color-accent-hover: #BC41DF
--color-accent-active: #9920BC
--color-accent-a08: rgba(203, 108, 230, 0.08)
--color-accent-a10: rgba(203, 108, 230, 0.10)
--color-accent-a15: rgba(203, 108, 230, 0.15)
--color-accent-a20: rgba(203, 108, 230, 0.20)
--color-accent-a30: rgba(203, 108, 230, 0.30)
--color-accent-a40: rgba(203, 108, 230, 0.40)
```

The `-aNN` variants exist so no task ever needs to compute or use `color-mix()` at runtime — every alpha value the old gold (`rgba(107, 85, 14, N)`) was used at already has a matching static token. If a task finds an alpha value not listed here, add the token to this list in `css-variables.liquid` first, then use it — never invent an ad hoc `rgba()` inline.

## Master Hex/Var → Token Mapping Rule

Apply this exact rule to every literal hex color **and every reference to the old CSS variables** found in a file, unless the task below says otherwise for that file. "Old CSS variable" means any use of `--color-title-dark`, `--color-paragraph-dark`, `--bg-primary`, `--accent-color`, `--button-bg-dark`, or `--button-text-on-dark` (defined in `snippets/css-variables.liquid`), with or without a hex fallback.

| Literal hex(es) or old var found | Replace with | Rule |
|---|---|---|
| `#FDFBF5`, `#FAFAF8`, `#FDFCF8`, `#F0ECDF`, `#F5F5F0`, `#F0EDE6`, `#F5F5F3`, `#E0DCD1`, `#E5E0D6` (only when used as a `background`/`background-color` **section** fill) | `var(--color-bg)` | cream section backgrounds → white |
| `#F0ECDF`, `#E0DCD1`, `#e8e4d9` (only when used as a `background`/`background-color` on a small **component** — button, chip, image placeholder, disabled state — not a full section) | `var(--color-border)` on its own is too flat for anything meant to read as a button; see Task 4 Step 0 for the required per-component design decision before applying a token here | do not default this to `var(--color-bg)` — see Task 4 |
| `var(--bg-primary, #F0ECDF)` or bare `var(--bg-primary)` | `var(--color-bg)` | replace the **entire** `var(...)` expression, not just the hex fallback inside it |
| `#6B550E` or `var(--color-title-dark, #6B550E)` / bare `var(--color-title-dark)`, used as `background` or `background-color` | `var(--color-accent-solid)` | solid fill (button/hover fill with white text on top) |
| `#6B550E` or `var(--color-title-dark, #6B550E)` / bare `var(--color-title-dark)`, used as `color`, `border-color`, `border`, `outline`, `fill`, `stroke` | `var(--color-accent)` | text/border/icon/focus-ring uses |
| `var(--accent-color, #6B550E)` / bare `var(--accent-color)` | same split as `#6B550E` above, by CSS property | this is a second, separate old variable name for the same gold — do not skip it because it isn't literally `--color-title-dark` |
| `#5a480c` | `var(--color-accent-active)` | manually-darkened gold hover/active states |
| `rgba(107, 85, 14, 0.08)` | `var(--color-accent-a08)` | |
| `rgba(107, 85, 14, 0.1)` or `0.10` | `var(--color-accent-a10)` | |
| `rgba(107, 85, 14, 0.15)` | `var(--color-accent-a15)` | |
| `rgba(107, 85, 14, 0.2)` or `0.20` | `var(--color-accent-a20)` | |
| `rgba(107, 85, 14, 0.3)` or `0.30` | `var(--color-accent-a30)` | |
| `rgba(107, 85, 14, 0.4)` or `0.40` | `var(--color-accent-a40)` | |
| `#2C2C2C`, `#1a1a1a` | `var(--color-fg)` | dark text |
| `#6D6C6C`, `#666`, `#888`, `#888888`, `var(--color-paragraph-dark, #6D6C6C)` / bare `var(--color-paragraph-dark)` | `var(--color-fg-muted)` | secondary/muted text — replace the entire `var(...)` expression where present |
| `#d4d0c5`, `#EBEBE5`, `#eeeeec`, `#bbb`, `#f0f0f0` (only when used as `border`/`border-color`/divider) | `var(--color-border)` | borders, dividers, subtle gradients |
| `#999` | `var(--color-fg-muted)` | disabled/low-emphasis text |
| `#C9A961`, `#8B7355`, `#4A3A0A` | `var(--color-accent)` | old secondary-gold accents (ratings, decorative text) |
| `#fff`, `#FFFFFF`, `#000`, `#000000` | **leave unchanged** | already correct brand white/black (card fills, overlays, button text) — but see the ordering note in Task 3 Step 0 before doing any `replace_all` |
| `#c44`, `#c0392b`, `#2e7d32`, `#e8f5e9` | **leave unchanged** | functional error/success colors, out of scope |

This table was cross-checked against the real theme source (`sections/*.liquid`, `snippets/css-variables.liquid`, `assets/critical.css`) during two rounds of review. If a task's grep turns up a hex or `var(--...)` not listed here, stop and flag it — do not guess.

---

### Task 0: Git baseline checkpoint

**Files:** none (repo-wide)

**Interfaces:** none — this only exists to make every later task's commit clean and revertible.

The working tree currently has pre-existing uncommitted changes unrelated to this plan (from earlier exploration/setup). Committing straight into that dirty state would silently bundle unrelated changes into the first "Recolor…" commit.

- [ ] **Step 1: Check current state**

```bash
git status --short
```

- [ ] **Step 2: Commit whatever is currently pending, isolated from the rebrand work**

```bash
git add -A
git commit -m "Checkpoint: pre-rebrand working state"
```

If `git status --short` was already clean, skip this task entirely — there is nothing to checkpoint.

- [ ] **Step 3: Create a working branch for the rebrand**

```bash
git checkout -b rebrand-visual
```

All subsequent task commits in this plan happen on `rebrand-visual`, not `main`.

---

### Task 1: Add brand tokens to the existing `css-variables.liquid` snippet and retire the old gold/cream variables

**Files:**
- Modify: `snippets/css-variables.liquid`

**Interfaces:**
- Produces: CSS custom properties `--color-bg`, `--color-fg`, `--color-fg-muted`, `--color-border`, `--color-accent`, `--color-accent-solid`, `--color-accent-hover`, `--color-accent-active`, `--color-accent-a08` through `--color-accent-a40`, available globally on `:root` — because this file is already rendered by `layout/theme.liquid:4`, `layout/coming-soon.liquid:8`, and `layout/password.liquid:5`, every page type gets the tokens with zero additional wiring.

This theme already defines an old token system in this file: `--color-title-dark: #6B550E`, `--color-paragraph-dark: #6D6C6C`, `--bg-primary: #F0ECDF`, `--accent-color: #6B550E`, `--button-bg-dark`, `--button-text-on-dark`. Sections reference these via `var(--color-title-dark, #6B550E)` etc. **Do not just add new tokens next to the old ones and leave the old ones defined** — every section that consumes `var(--color-title-dark, #6B550E)` would keep resolving to gold, because the variable itself still exists and wins over its own fallback.

- [ ] **Step 1: Read the current file and confirm the old variable names**

```bash
grep -n "color-title-dark\|color-paragraph-dark\|bg-primary\|accent-color\|button-bg-dark\|button-text-on-dark" snippets/css-variables.liquid
```

Expected output includes these lines (confirmed during design review):
```
69:    --color-title-dark: #6B550E;
70:    --color-paragraph-dark: #6D6C6C;
81:    --bg-primary: #F0ECDF;
83:    --accent-color: #6B550E;
```
Note the exact line numbers may drift slightly — use them as a starting point, not gospel.

- [ ] **Step 2: Change the old variable *values* to the new brand colors, and add the new token names**

For each old variable, change its value (do not delete or rename the variable — later tasks will replace the `var(--old-name, ...)` call sites, but until every call site is migrated, the safest intermediate state is "old variable name, new value," so nothing regresses mid-plan):

```
--color-title-dark: #CB6CE6;
--color-paragraph-dark: #6B6B6B;
--bg-primary: #FFFFFF;
--accent-color: #CB6CE6;
```

Immediately below the existing block, add the new token names as the ones this plan's mapping table refers to by name:

```css
--color-bg: #FFFFFF;
--color-fg: #111111;
--color-fg-muted: #6B6B6B;
--color-border: #E5E5E5;
--color-accent: #CB6CE6;
--color-accent-solid: #B125D9;
--color-accent-hover: #BC41DF;
--color-accent-active: #9920BC;
--color-accent-a08: rgba(203, 108, 230, 0.08);
--color-accent-a10: rgba(203, 108, 230, 0.10);
--color-accent-a15: rgba(203, 108, 230, 0.15);
--color-accent-a20: rgba(203, 108, 230, 0.20);
--color-accent-a30: rgba(203, 108, 230, 0.30);
--color-accent-a40: rgba(203, 108, 230, 0.40);
```

Leave `--button-bg-dark` / `--button-text-on-dark` values as-is for now — Task 4 makes an explicit design decision about button fills and will update them there, not here.

- [ ] **Step 3: Start the dev theme and verify the tokens load**

Run: `shopify theme dev`

Open the printed preview URL, open browser DevTools → Elements → `<html>` → Computed styles on `:root`, and confirm `--color-accent` resolves to `rgb(203, 108, 230)` (i.e. `#CB6CE6`) **and** `--color-title-dark` also now resolves to `rgb(203, 108, 230)` (proof the old variable was repointed, not just shadowed).

Expected: the whole site should already look noticeably different (most gold text/borders should already have flipped to fuchsia, and the `--bg-primary`-driven cream fills should already be white), even though no section `.liquid` file has been touched yet — because dozens of sections already consume these variables. This is expected and correct; it is not "getting ahead" of later tasks, it's why Task 1 exists.

- [ ] **Step 4: Lint**

Run: `shopify theme check --path .`
Expected: no new errors introduced (pre-existing warnings unrelated to this change are fine).

- [ ] **Step 5: Commit**

```bash
git add snippets/css-variables.liquid
git commit -m "Repoint theme color variables to brand fuchsia/white and add new tokens"
```

---

### Task 2: Recolor the global stylesheet (`assets/critical.css`)

**Files:**
- Modify: `assets/critical.css`

**Interfaces:**
- Consumes: tokens from Task 1.

`critical.css` is loaded on every page (`theme.liquid:11`, and equivalent lines in the other two layouts) and is **not** a section file, so it is easy to miss with a `sections/*.liquid`-scoped search. It sets the page `<body>` background and several global, reused button/wishlist classes.

- [ ] **Step 1: Enumerate everything to change**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" assets/critical.css
```

Confirmed occurrences as of the design review: `background-color: #F0ECDF;` (line 19, the `<body>` background), and roughly ten instances of `#6B550E` styling `.btn--cta:hover`, `.btn--add-to-cart:hover`, `.btn--secondary` (color, border, hover fill), `.wishlist-btn.is-active`, `.product-card__wishlist.is-active`, plus `#f0f0f0`, `#1a1a1a`, `#e5e5e5`.

- [ ] **Step 2: Apply the mapping rule**

`#F0ECDF` (body background) → `var(--color-bg)`. `#6B550E` split by property per the master rule (`background`/`background-color` → `var(--color-accent-solid)`; `color`/`border-color` → `var(--color-accent)`). `#1a1a1a` → `var(--color-fg)`. `#e5e5e5` → `var(--color-border)`. `#f0f0f0` → `var(--color-border)`.

- [ ] **Step 3: Visual check**

Reload the dev preview. Confirm the `<body>` background (visible in any gap/margin area not covered by a section) is white, and that any product-card wishlist heart icon in its active/liked state, and any secondary button, show fuchsia instead of gold.

- [ ] **Step 4: Regression grep**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" assets/critical.css
```

Expected: only the intentionally-unchanged brand black/white and functional error/success colors remain (if any) — no `#F0ECDF`/`#6B550E`/`#f0f0f0` left.

- [ ] **Step 5: Commit**

```bash
git add assets/critical.css
git commit -m "Recolor global stylesheet to brand tokens"
```

---

### Task 3: Recolor header and footer, including saved footer background color

**Files:**
- Modify: `sections/header.liquid`
- Modify: `sections/footer.liquid`
- Modify: `sections/footer-group.json`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 0: Read this before running any `replace_all`**

Several hex strings are short and are substrings of longer ones in the same file (e.g. `#fff` is a substring of nothing here, but `#888` is a substring of `#888888` elsewhere in this plan — same risk applies generally). Before using `replace_all` on any hex string, first search for whether a longer hex sharing the same prefix also exists in the file; if so, replace the longer one first. For header/footer this is not currently an issue, but treat it as standing practice for every task in this plan.

- [ ] **Step 1: Replace hex values and old variable references in `sections/header.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/header.liquid
```

Apply the Master Mapping Rule to every result, including bare `var(--color-title-dark)` references with **no hex fallback** — these do not show up in a plain hex grep, which is why the command above also searches for the variable names directly. All uses in this file are `color`/`border-color`/hover-text, so they map to `var(--color-accent)`. `#000`/`#fff` stay unchanged.

- [ ] **Step 2: Replace hex values and old variable references in `sections/footer.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/footer.liquid
```

Apply the rule: `#2C2C2C` → `var(--color-fg)`, `#FDFCF8`/`#F0EDE6` → `var(--color-bg)`, `#E5E0D6` → `var(--color-border)`, `#6B550E`/`var(--color-title-dark, ...)` → `var(--color-accent)` (all uses in this file are `color`/hover text, not a fill), `#888` → `var(--color-fg-muted)`, `#FFFFFF` unchanged.

- [ ] **Step 3: Migrate the saved footer background color**

`sections/footer-group.json` stores a merchant-set value that overrides whatever default `footer.liquid` falls back to:

```bash
grep -n "background_color" sections/footer-group.json
```

Expected: `"background_color": "#fdfbf5"` (or similar cream hex) at approximately line 30. Change it to:

```json
"background_color": "#ffffff"
```

- [ ] **Step 4: Visual check**

With `shopify theme dev` still running, reload the preview. Confirm: header background is white/transparent with black text and fucsia hover/active link states; footer background is now genuinely white (not just "should be" — this is the step that catches the JSON override), footer text is black/gray, and the "SNC Designs" credit link still renders unchanged apart from recoloring if it inherited a token.

- [ ] **Step 5: Regression grep**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/header.liquid sections/footer.liquid
grep -n "background_color" sections/footer-group.json
```

Expected: no leftover gold/cream hex in the `.liquid` files; `footer-group.json` shows `#ffffff`.

- [ ] **Step 6: Lint**

Run: `shopify theme check --path .`
Expected: no new errors.

- [ ] **Step 7: Commit**

```bash
git add sections/header.liquid sections/footer.liquid sections/footer-group.json
git commit -m "Recolor header and footer, migrate saved footer background color"
```

---

### Task 4: Recolor hero-banner, product-carousel, featured-product, and decide the cream-button-fill treatment

**Files:**
- Modify: `sections/hero-banner.liquid`
- Modify: `sections/product-carousel.liquid`
- Modify: `sections/featured-product.liquid`
- Modify: `templates/index.json`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 0: Design decision — cream component fills (buttons, chips, placeholders) must not become invisible white-on-white**

Several components use `#F0ECDF`/`#E0DCD1`/`#e8e4d9` as a **component background** (an add-to-cart chip, a disabled-state pill, an image placeholder) sitting on top of a section that is *also* becoming white. Mapping these straight to `var(--color-bg)` (white) produces a borderless white button on a white background — invisible. Use this fixed sub-rule instead, in every task from here on wherever this pattern appears:

- If the component is an **interactive button/chip with text on it** (e.g. `product-carousel.liquid`'s `.product-card-carousel__add-btn`, background `#F0ECDF` with text `#6B550E`): background → `var(--color-bg)`, add `border: 1px solid var(--color-accent);`, text stays `var(--color-accent)`. This turns it into the spec's "secondary/outline" pill button treatment.
- If the component is a **disabled state** (e.g. `#E0DCD1`): background → `var(--color-border)`, text → `var(--color-fg-muted)`.
- If the component is a **neutral image placeholder** (e.g. `#e8e4d9` / the `#e8e4d9, #d4d0c5` gradient stops used for skeleton/placeholder blocks): background → `var(--color-border)` (a visible light-gray placeholder, not white-on-white).

- [ ] **Step 1: Recolor `sections/hero-banner.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/hero-banner.liquid
```

Apply the rule: `#1a1a1a` → `var(--color-fg)`, `#f0f0f0` → `var(--color-border)` (button hover-darken tone), `#000000`/`#fff` unchanged (video overlay + button text). Do **not** touch the `overlay_color`/`overlay_opacity` schema settings — those are merchant-editable and already default to black, which matches the brand.

- [ ] **Step 2: Recolor `sections/product-carousel.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)|rgba\(107, 85, 14[^)]*\)" sections/product-carousel.liquid
```

Apply the rule: cream section backgrounds (`#FDFBF5`, `#F0ECDF`/`var(--bg-primary, ...)`) → `var(--color-bg)`; `#6B550E` as `background` (arrow hover fill) → `var(--color-accent-solid)`; `#6B550E` as `color`/`border-color`/focus-outline → `var(--color-accent)`; any `rgba(107, 85, 14, 0.15)` → `var(--color-accent-a15)` (do not use `color-mix()` — the static token already exists); `#6D6C6C`/`var(--color-paragraph-dark, ...)` → `var(--color-fg-muted)`; the `.product-card-carousel__add-btn` fill (`#F0ECDF`) and its disabled variant (`#E0DCD1`) and the placeholder gradient (`#e8e4d9`/`#d4d0c5`) → apply Step 0's sub-rule, not a blanket `var(--color-bg)`; `#fff` unchanged.

- [ ] **Step 3: Recolor `sections/featured-product.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)|rgba\(107, 85, 14[^)]*\)" sections/featured-product.liquid
```

Apply the rule: `#FAFAF8`/`#F0ECDF` (section backgrounds) → `var(--color-bg)`; any `#F0ECDF` used as a badge/chip component fill → Step 0's sub-rule; `#6B550E` split by `background` vs `color`; `#6D6C6C` → `var(--color-fg-muted)`; any `rgba(107, 85, 14, ...)` → the matching `--color-accent-aNN` token; `#FFFFFF` unchanged.

- [ ] **Step 4: Migrate saved color/divider values in `templates/index.json`**

```bash
grep -n "background_color\|divider_color" templates/index.json
```

Expected (from design review): `"divider_color": "#fdfcf8"` (hero-banner block, ~line 23), `"background_color": "#fdfcf8"` (featured-product block, ~line 81), `"background_color": "#fdfbf5"` (testimonials-masonry block, ~line 167 — out of scope for *this* task but confirm it still reads correctly; Task 6 handles testimonials-masonry's `.liquid` file, this JSON value should be migrated at the same time as that task, see Task 6 Step 3).

For this task, update the hero-banner and featured-product values:

```json
"divider_color": "#ffffff",
```
```json
"background_color": "#ffffff",
```

- [ ] **Step 5: Visual check**

Reload the dev preview on the homepage. Confirm: hero unchanged visually except text/border tweaks; both product carousels ("Candles", "Reed Diffusers") show white background, fucsia titles/hover arrows, and an outlined (not invisible) add-to-cart chip; the "LOVE JONES" featured product block shows white background with fucsia badge/accent and black body text.

- [ ] **Step 6: Regression grep**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/hero-banner.liquid sections/product-carousel.liquid sections/featured-product.liquid
grep -n "background_color\|divider_color" templates/index.json
```

- [ ] **Step 7: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 8: Commit**

```bash
git add sections/hero-banner.liquid sections/product-carousel.liquid sections/featured-product.liquid templates/index.json
git commit -m "Recolor hero, product carousel, and featured product sections; migrate saved index.json colors"
```

---

### Task 5: Recolor PLP, list-collections, main-product

**Files:**
- Modify: `sections/plp.liquid`
- Modify: `sections/list-collections.liquid`
- Modify: `sections/main-product.liquid`
- Modify: `templates/product.json`

**Interfaces:**
- Consumes: tokens from Task 1; the component-fill sub-rule from Task 4 Step 0.

- [ ] **Step 1: Recolor `sections/plp.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)|rgba\(107, 85, 14[^)]*\)" sections/plp.liquid
```

Apply the rule: `#F0ECDF`/`#FDFBF5` (section backgrounds) → `var(--color-bg)`; any `#F0ECDF`/`#E0DCD1` used as a component fill (buttons/badges) → Task 4 Step 0's sub-rule; `#6B550E` split by `background` vs `color`/`border`; `#FFFFFF` unchanged; `#2C2C2C` → `var(--color-fg)`; `#6D6C6C` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` (placeholder gradient) → `var(--color-border)`; any `rgba(107, 85, 14, ...)` → matching `--color-accent-aNN`.

- [ ] **Step 2: Recolor `sections/list-collections.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/list-collections.liquid
```

Apply the rule: `#FDFBF5`/`#F0ECDF` → `var(--color-bg)`; `#6B550E` split by property as above; `#2C2C2C` → `var(--color-fg)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#6D6C6C` → `var(--color-fg-muted)`; `#fff` unchanged.

- [ ] **Step 3: Recolor `sections/main-product.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)|rgba\(107, 85, 14[^)]*\)" sections/main-product.liquid
```

Apply the rule: `#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property; `#5a480c` → `var(--color-accent-active)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#6D6C6C` → `var(--color-fg-muted)`; `#999` → `var(--color-fg-muted)`; any `rgba(107, 85, 14, ...)` → matching `--color-accent-aNN`; `#fff` unchanged.

- [ ] **Step 4: Migrate saved color value in `templates/product.json`**

```bash
grep -n "background_color" templates/product.json
```

Expected: `"background_color": "#f0ecdf"` on the disabled `testimonials_masonry_UJzxJn` block, approximately line 36. Change to:

```json
"background_color": "#ffffff",
```

- [ ] **Step 5: Visual check**

Reload dev preview: visit a collection page (list-collections), a category listing (plp), and a product page (main-product). Confirm white backgrounds, fucsia buttons/prices/accordion accents, black/gray text throughout, no leftover cream anywhere, and that any pill/chip components have a visible outline rather than blending into the white background.

- [ ] **Step 6: Regression grep**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/plp.liquid sections/list-collections.liquid sections/main-product.liquid
grep -n "background_color" templates/product.json
```

- [ ] **Step 7: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 8: Commit**

```bash
git add sections/plp.liquid sections/list-collections.liquid sections/main-product.liquid templates/product.json
git commit -m "Recolor PLP, collection list, and product page sections; migrate saved product.json color"
```

---

### Task 6: Recolor cart, contact-hero, contact-form, favoritos

**Files:**
- Modify: `sections/cart.liquid`
- Modify: `sections/contact-hero.liquid`
- Modify: `sections/contact-form.liquid`
- Modify: `sections/favoritos.liquid`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Recolor `sections/cart.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)|rgba\(107, 85, 14[^)]*\)" sections/cart.liquid
```

Apply the rule: `#FDFBF5`/`#F0ECDF` → `var(--color-bg)`; `#6B550E` split by property; `#5a480c` → `var(--color-accent-active)`; `#6D6C6C` → `var(--color-fg-muted)`; `#999` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; every `rgba(107, 85, 14, ...)` occurrence (there are several, including at least one `box-shadow`) → the matching `--color-accent-aNN` token, preserving which CSS property it's on; `#c44` unchanged (destructive/remove-item color, out of scope); `#fff` unchanged.

- [ ] **Step 2: Recolor `sections/contact-hero.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/contact-hero.liquid
```

Apply the rule: `#000000`/`#fff` unchanged (image overlay + heading text, already brand black/white).

- [ ] **Step 3: Recolor `sections/contact-form.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)|rgba\(107, 85, 14[^)]*\)" sections/contact-form.liquid
```

Apply the rule: `#FAFAF8` → `var(--color-bg)`; `#6B550E` split by property; `#5a480c` → `var(--color-accent-active)`; `#1a1a1a` → `var(--color-fg)`; `#888`/`#666` → `var(--color-fg-muted)`; `#F5F5F3`/`#eeeeec` → `var(--color-border)`; `#fff` unchanged; `#2e7d32`/`#e8f5e9` unchanged (success-message state, out of scope).

- [ ] **Step 4: Recolor `sections/favoritos.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)|rgba\(107, 85, 14[^)]*\)" sections/favoritos.liquid
```

Apply the rule: `#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property; `#5a480c` → `var(--color-accent-active)`; `#6D6C6C` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; every `rgba(107, 85, 14, ...)` → matching `--color-accent-aNN`; `#c44` unchanged; `#fff` unchanged.

- [ ] **Step 5: Visual check**

Reload dev preview: add a product to cart and open the cart page; visit `/pages/contact`; visit the favorites page. Confirm white backgrounds, fucsia CTAs/links, black/gray text, form inputs still functional (submitting the contact form still shows the existing success message green — untouched).

- [ ] **Step 6: Regression grep**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/cart.liquid sections/contact-hero.liquid sections/contact-form.liquid sections/favoritos.liquid
```

- [ ] **Step 7: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 8: Commit**

```bash
git add sections/cart.liquid sections/contact-hero.liquid sections/contact-form.liquid sections/favoritos.liquid
git commit -m "Recolor cart, contact, and favorites sections"
```

---

### Task 7: Recolor scrolling-gallery, testimonials-masonry, product-recommendations, comixub

**Files:**
- Modify: `sections/scrolling-gallery.liquid`
- Modify: `sections/testimonials-masonry.liquid`
- Modify: `sections/product-recommendations.liquid`
- Modify: `sections/comixub.liquid`
- Modify: `templates/index.json` (testimonials-masonry block)
- Modify: `templates/product.json` (already handled in Task 5 — do not repeat)

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Recolor `sections/scrolling-gallery.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/scrolling-gallery.liquid
```

Apply the rule: `#F0ECDF` / `var(--bg-primary, #F0ECDF)` (the section background — replace the **entire** `var(...)` expression, not just its fallback) → `var(--color-bg)`; `#6B550E`, `#8B7355`, `#4A3A0A` (all decorative gold text tones) → `var(--color-accent)`; `#6D6C6C` / `var(--color-paragraph-dark, ...)` → `var(--color-fg-muted)`. Per the spec, do **not** add any scroll-triggered color-morph — only replace the flat literal colors/variables with tokens. The section stays `disabled: true` on the homepage (no template change needed).

- [ ] **Step 2: Recolor `sections/testimonials-masonry.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/testimonials-masonry.liquid
```

Apply the rule: `#F0ECDF`/`#F5F5F0`/`#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property; `#C9A961` (rating stars/laurel) → `var(--color-accent)`; `#EBEBE5` → `var(--color-border)`; `#bbb` → `var(--color-fg-muted)` (this is the star "off" tone, i.e. text/icon color, not a border — do not send it to `var(--color-border)`); `#6D6C6C` → `var(--color-fg-muted)`; `#fff` unchanged.

- [ ] **Step 3: Migrate saved color value for testimonials-masonry**

```bash
grep -n "background_color" templates/index.json
```

Find the `testimonials_masonry_GyL6jr` block's `"background_color": "#fdfbf5"` (or similar) and change it to:

```json
"background_color": "#ffffff",
```

(`templates/product.json`'s testimonials-masonry block was already migrated in Task 5 Step 4 — do not duplicate that edit here.)

- [ ] **Step 4: Recolor `sections/product-recommendations.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/product-recommendations.liquid
```

Apply the rule: `#FDFBF5` → `var(--color-bg)`; `#6B550E` split by property; `#6D6C6C` → `var(--color-fg-muted)`; `#e8e4d9`/`#d4d0c5` → `var(--color-border)`; `#fff` unchanged.

- [ ] **Step 5: Recolor `sections/comixub.liquid`**

```bash
grep -noE "#[0-9a-fA-F]{3,8}|var\(--(color-title-dark|color-paragraph-dark|bg-primary|accent-color)[^)]*\)" sections/comixub.liquid
```

Apply the rule with two corrections specific to this file:

- `#FDFCF8` (page/section background) → `var(--color-bg)`.
- `#FFF3E8` is **not** a background tint — it is the default value of the `heading_color` schema setting (used for the "COMING SOON" heading text and the success-message text, both meant to sit on the page background). Map it to `var(--color-fg)` (black heading on white page), not `var(--color-bg)` — mapping it to the background token would render white text on a white page.
- `#6B550E` split by property as usual.
- `#1a1a1a`/`#2C2C2C` → `var(--color-fg)`.
- `#888`/`#888888` → `var(--color-fg-muted)` — **replace `#888888` before `#888`** if using `replace_all`, so the longer string isn't corrupted by a partial match on the shorter one.
- `#6D6C6C` → `var(--color-fg-muted)`.
- `#FFFFFF`/`#000000` unchanged.
- `#c0392b` unchanged (error state, out of scope).

Also update the **schema `"default"` values** for `heading_color` (~line 333, currently `"#FFF3E8"`) and any other color setting whose default is the old gold/cream (e.g. `#6B550E`, `#FDFCF8` around lines 368/468) to the new equivalents (`heading_color` default → `#111111`; gold defaults → `#CB6CE6`; cream defaults → `#FFFFFF`). This does not touch merchant-saved values (this page's template stores no color overrides, confirmed during review) — it only fixes what a merchant sees if they open the theme editor's color pickers for this section, which currently show swatches from the old palette.

- [ ] **Step 6: Visual check**

Reload dev preview: check the (disabled, so preview via theme editor "Add section" or a temporary dev-only template) scrolling-gallery and testimonials-masonry render white/fucsia; visit the coming-soon page and confirm white background, fucsia CTA button, **black** heading text (not white-on-white).

- [ ] **Step 7: Regression grep**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/scrolling-gallery.liquid sections/testimonials-masonry.liquid sections/product-recommendations.liquid sections/comixub.liquid
grep -n "background_color" templates/index.json
```

- [ ] **Step 8: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 9: Commit**

```bash
git add sections/scrolling-gallery.liquid sections/testimonials-masonry.liquid sections/product-recommendations.liquid sections/comixub.liquid templates/index.json
git commit -m "Recolor scrolling gallery, testimonials, recommendations, and coming-soon sections; fix comixub heading contrast"
```

---

### Task 8: Recolor about-banner (text/controls only — image stays) and reset the theme-wide background/foreground settings

**Files:**
- Modify: `sections/about-banner.liquid` (only if Step 1 finds something beyond the two expected colors)
- Modify: `config/settings_data.json`

**Interfaces:**
- Consumes: tokens from Task 1.

- [ ] **Step 1: Inspect current hex usage in about-banner**

```bash
grep -noE "#[0-9a-fA-F]{3,8}" sections/about-banner.liquid
```

Expected: only `#fff` and `#000000` (overlay color default + heading/body text color). Per the spec, **do not** change the section's background-image + overlay treatment — it stays a full-bleed photo. Leave both values unchanged; there is nothing to recolor here since it already uses brand black/white. If any other hex appears beyond these two, apply the Master Mapping Rule to it only, and commit that change under this task.

- [ ] **Step 2: Reset the theme-wide background/foreground settings**

`config/settings_schema.json` already defaults these to reasonable values (`background_color` → `#FFFFFF`, `foreground_color` → `#333333`), but the merchant-saved override in `config/settings_data.json` still holds the old cream value:

```bash
grep -n "background_color\|foreground_color" config/settings_data.json
```

Expected: `"background_color": "#fdfbf5"`, `"foreground_color": "#333333"`. Change to:

```json
"background_color": "#ffffff",
"foreground_color": "#111111",
```

This setting isn't consumed anywhere in the current custom sections (confirmed during review — every section hardcodes or uses the token system instead), so this step has no visible effect today. It's belt-and-braces: if any future base-theme component or Shopify default block reads these values, they won't resurrect the old cream.

- [ ] **Step 3: Commit**

```bash
git add config/settings_data.json
# only add sections/about-banner.liquid if Step 1 found something to change
git commit -m "Reset theme-wide background/foreground settings; confirm about-banner needs no change"
```

---

### Task 9: Create the About Us page and editorial section

**Files:**
- Create: `sections/about-editorial.liquid`
- Create: `templates/page.about.json`

**Interfaces:**
- Consumes: tokens from Task 1 (`var(--color-bg)`, `var(--color-fg)`, `var(--color-fg-muted)`, `var(--color-accent)`).
- Produces: a new page type `about-editorial` renderable via any `templates/page.*.json` that lists it in `order`.

- [ ] **Step 1: Create the section file**

Create `sections/about-editorial.liquid`:

```liquid
{% liquid
  assign alt_text = section.settings.image.alt | default: section.settings.title | escape
%}
<section class="about-editorial" data-section-id="{{ section.id }}">
  <div class="about-editorial__media">
    {% if section.settings.image != blank %}
      {{ section.settings.image | image_url: width: 1400 | image_tag:
        loading: 'lazy',
        class: 'about-editorial__image',
        alt: alt_text
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

Note the `{% liquid assign alt_text = ... %}` block at the top: `image_tag`'s `alt:` parameter must receive a plain string, not a filter chain ending in `| escape` inlined into the tag call — Liquid would apply `escape` to the whole `image_tag` output (HTML-escaping the entire `<img>` element into visible text) rather than to the alt value alone. Computing `alt_text` as its own variable first avoids that bug.

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

### Task 10: Add favicon plumbing (image swap pending client asset)

**Files:**
- Modify: `layout/theme.liquid`
- Modify: `layout/coming-soon.liquid`
- Modify: `layout/password.liquid`
- Modify: `config/settings_schema.json`

**Interfaces:**
- Consumes: a merchant-uploaded image via the new `favicon` setting.
- Produces: a `<link rel="icon">` (and `apple-touch-icon`) tag that resolves once the client uploads the emblem asset — no visual change until then, but the wiring is complete and testable independently of the missing file.

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

- [ ] **Step 2: Render the favicon link tags in all three layouts**

In `layout/theme.liquid`, `layout/coming-soon.liquid`, and `layout/password.liquid`, inside `<head>`, add (near the other `<link>` tags, but **outside** the `{% unless settings.type_primary_font.system? %}` block in `theme.liquid` — that block is conditional on font settings and unrelated to the favicon; nesting it there would make the icon disappear whenever a merchant picks a system font):

```liquid
      {% if settings.favicon != blank %}
        <link rel="icon" type="image/png" href="{{ settings.favicon | image_url: width: 32, height: 32 }}">
        <link rel="apple-touch-icon" href="{{ settings.favicon | image_url: width: 180, height: 180 }}">
      {% endif %}
```

- [ ] **Step 3: Verify with a placeholder image**

Since the client's asset hasn't arrived yet, temporarily upload any square PNG through **Theme editor → Theme settings → Favicon** on the dev theme to confirm the `<link>` tags render with a real URL (inspect page source on the homepage, the coming-soon page, and the password page — search for `rel="icon"`). Then remove the placeholder image from the setting so the live site doesn't ship a wrong icon.

Expected: both `<link>` tags appear in page source only when `settings.favicon` is set; absent otherwise (no broken icon request).

- [ ] **Step 4: Lint**

Run: `shopify theme check --path .`

- [ ] **Step 5: Commit**

```bash
git add layout/theme.liquid layout/coming-soon.liquid layout/password.liquid config/settings_schema.json
git commit -m "Add favicon setting and link tags across all layouts (pending client asset)"
```

- [ ] **Step 6 (follow-up, blocked on client asset): Upload the real favicon**

Once the client sends the "S." emblem file: crop/export it as a square PNG (recommend 512x512 master, Shopify will serve resized via `image_url`), upload it through **Theme editor → Theme settings → Favicon**, and visually confirm the browser tab icon updates on the dev preview before publishing.

---

## Final End-to-End Verification

- [ ] Run `shopify theme check --path .` one last time across the whole theme — expect zero new errors compared to the pre-rebrand baseline.
- [ ] Run this widened regression grep — it covers every file type this plan touches, not just `sections/*.liquid` (the gap that the pre-execution review found):

```bash
grep -rnoiE "#(FDFBF5|FAFAF8|FDFCF8|F0ECDF|F5F5F0|F0EDE6|F5F5F3|FFF3E8|E0DCD1|E5E0D6|6B550E|5a480c|C9A961|8B7355|4A3A0A)|rgba\(107, ?85, ?14" sections/ snippets/ assets/ layout/ templates/ config/
```

Expected: **no matches**. If anything matches inside `config/settings_data.json` or a `templates/*.json` file for a section/property not already covered by Tasks 3–8, add a step to the relevant task and re-run.

- [ ] Walk the dev preview end to end: home → collection list → PLP → PDP → cart → contact → favorites → about → coming-soon → password page (if password-protected). Confirm white background and fucsia/black/gray treatment everywhere except `hero-banner` (video/overlay) and `about-banner` (photo/overlay), which intentionally keep their image-based dark treatment.
- [ ] Confirm the header "About" link works and the new page has real content.
- [ ] Confirm favicon `<link>` tags render when a favicon is set (still pending client's actual image).
- [ ] Confirm no component (buttons, chips, placeholders) renders as invisible white-on-white — spot check the product-carousel add-to-cart chip, PLP filter pills, and any disabled-state buttons.
- [ ] Do **not** publish the dev theme yet — hand back to the client for visual review on the preview URL before promoting to live.
