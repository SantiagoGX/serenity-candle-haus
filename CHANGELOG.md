# Serenity Candle Haus — Rebrand Changelog

All work below lives on the `rebrand-visual` branch (never pushed anywhere,
`main` untouched except for the initial "pre-rebrand checkpoint" commit).
Nothing here has touched the live/published theme directly — everything was
built and tested through the dev theme first.

## 1. Brand color system

The theme used a gold/cream palette that didn't match the real brand. Replaced
it end-to-end with white background, black text, gray for secondary text, and
the brand fuchsia (`#CB6CE6`, pulled from the client's logo files) reserved
strictly as a punctual accent — buttons, prices, badges — never as a
background fill.

- `snippets/css-variables.liquid`: repointed the existing color tokens
  (`--color-title-dark`, `--bg-primary`, `--accent-color`, etc.) to the new
  values instead of renaming them, since ~70+ places in the theme already
  referenced those exact variable names. Added a clean new token set
  (`--color-bg`, `--color-fg`, `--color-fg-muted`, `--color-border`,
  `--color-accent` + hover/active/alpha variants) for new work going forward.
- `assets/critical.css` (global stylesheet): recolored shared heading/button
  rules; removed a `h1.avigea-font` selector that was silently overriding
  section-level color rules due to CSS specificity (this was the cause of the
  "LOVE JONES" title staying fuchsia after the rest of the page changed).
- Recolored every custom section: header, footer, hero banner, product
  carousel, scrolling gallery, testimonials, product recommendations,
  coming-soon, comixub. Prices and CTAs use the accent color; titles and body
  copy use black/gray.
- Fixed a pre-existing (not introduced by this work) bug where
  `.product-title { font-weight: 400 }` was silently cancelling the intended
  bold weight — corrected to 700.

## 2. Header

- Icon and text colors on scroll now stay black instead of turning fuchsia —
  fuchsia is reserved for the logo only.
- "PRODUCTS" submenu label renamed to "Candles."
- Fixed the mobile sidebar menu, which was hardcoded to "Shop / About /
  Contact" regardless of what menu the merchant picked in the editor. The
  `Mobile Menu` setting now falls back to the `Menu` setting before falling
  back to the hardcoded defaults, so picking a menu in the editor genuinely
  changes what renders in the sidebar.
- Added favicon support (didn't exist before): a `Favicon` setting under
  Theme Settings, wired through `snippets/meta-tags.liquid` (including the
  Apple touch icon), rendered once across all layouts.

## 3. Footer

- Newsletter form: added editable heading/placeholder/button/success-message
  text; fixed the signup so it actually opts the customer into email
  marketing (`contact[accepts_marketing]`), not just a tag.
- Added an Instagram icon link (enlarged per request, centered on mobile).
- Added an optional "S." brand emblem image that can sit opposite the
  newsletter column, with a show/hide toggle in the editor.
- Added a size slider for the footer brand-name text.
- Fixed the "Crafted with love by SNC Designs" agency credit so it can
  actually be cleared to blank from Visual Edits — it was stuck showing a
  fallback because of Liquid's `| default:` filter firing on blank strings;
  switched to `{% if x != blank %}` checks.
- Fixed Instagram icon alignment on mobile (was left-aligned, now centered).

## 4. Hero banner

- Added a divider on/off toggle and a content alignment control
  (left/center/right), matching the header's page-width margin.
- Fixed a layout bug where turning the divider off caused the whole content
  block to jump upward (re-centering); it now anchors to the bottom, so
  removing the divider just moves the heading down.
- Rebuilt the background video handling so it plays immediately by default on
  both mobile and desktop, with a genuine last-resort fallback (poster image)
  only if autoplay is confirmed blocked by the browser — never a "tap to
  play" icon as the default state.

## 5. Product carousel

- Replaced the old JS index/`translateX` carousel with native scroll +
  drag (mouse) + swipe (touch), using CSS scroll-snap.
- Arrows repositioned as circles flanking the sides instead of centered
  above; solid fuchsia by default.
- Unified all badges/buttons as pills, icon-only buttons as circles.
- Fixed the "ADD +" button wrapping into two lines / becoming circular on
  narrow product cards (missing `white-space: nowrap`) — same fix applied to
  the favorites page and product recommendations section.

## 6. About page

Rebuilt twice. Final version is a Louis Vuitton/De Levillé-style editorial
layout (researched with the ui-ux-pro-max design skill): eyebrow, title,
subtitle, header image, then flexible blocks (text / image / quote) the
merchant can add and reorder. Fixed mobile top padding so the header (fixed,
70px) no longer overlaps the "Our Story" title.

## 7. Reviews / testimonials

- Added a per-review star rating field (previously hardcoded to always show
  5 stars).
- Wrote 6 realistic customer reviews with natural voice, mixed 4- and
  5-star ratings, real product names.
- Fixed mobile behavior: shows 3 reviews initially with a "View More" button
  that reveals 3 more per click (desktop unchanged at 6 initially).
- Per later request: changed the one overly-negative 4-star review to 5
  stars with softer copy, and bumped the displayed average rating from 4.8
  to 5.0 everywhere it appears.
- Fixed a Liquid type error (`Integer` vs `String` comparison) in the star
  rendering logic.

## 8. SEO / GEO

- Added Organization, WebSite (with SearchAction), and Product structured
  data (JSON-LD) so search engines and AI assistants (ChatGPT, Claude, etc.)
  can better identify and surface the store's products.
- Fixed a duplicate `<h1>` on the homepage (hero banner and featured product
  both rendered one — downgraded the featured product's to `<h2>`).
- De-duplicated favicon `<link>` tags that had been accidentally added in
  multiple places.

## 9. Mobile bug fixes

From screenshots: squished/illegible "Your cart is empty" and "No favorites
yet" headings (a global `letter-spacing: -4px` rule was breaking on short
headings — fixed with scoped overrides); About page title too close to the
fixed header; product page content starting underneath the fixed header;
"ADD +" button text wrapping on narrow cards.

## 10. 404 page

Fully custom design: large "404" digits with the middle zero replaced by an
animated fuchsia flame (continuous flicker), candle-themed copy, a pill CTA
back to the shop, and mouse/touch parallax motion (respects
`prefers-reduced-motion`). Also fixed a missing `templates/404.json` file,
which was the actual reason the Theme Editor showed no sections available on
that page.

## 11. Page loader

Added a once-per-session loading animation: white screen with the small
brand mark spinning briefly, gated by `sessionStorage` so it only shows once
per browsing session, and skipped entirely if the visitor has
`prefers-reduced-motion` on.

## 12. Customer review submissions

The client wanted customers to be able to submit reviews. A theme has no
backend/database of its own, so a public storefront form structurally cannot
write into another section's saved settings (Shopify's security boundary,
not a workaround-able limitation) — this was explained directly rather than
building something fragile.

Landed on the simpler, robust option instead: a dedicated `/pages/review`
page (`sections/review-form.liquid` + `templates/page.review.json`) with a
form for name, email, product, a 5-star rating, and the review text. It uses
Shopify's native `{% form 'contact' %}`, which emails the merchant directly
with all the field values — the client can then manually add the good ones
to the Testimonials section, same as always. Confirmed working end-to-end.

## 13. Product ingredients

The "Ingredients" accordion on the product page was hardcoded as a section
setting on the shared `main-product` template, so every product showed the
same text regardless of what it actually contained. Created a `custom.ingredients`
(rich text) product metafield definition in the store and repointed
`sections/main-product.liquid` (accordion block, ~line 129) to read
`product.metafields.custom.ingredients.value` instead of
`section.settings.ingredients_content`. The accordion now only renders when
that metafield is filled in for the given product. Removed the now-unused
`ingredients_content` setting from the section schema and from
`templates/product.json`; kept `ingredients_title` as a section setting since
the label is the same across all products.

Client fills in the per-product ingredient list from Admin → Products →
[product] → Metafields → Ingredients.

## Known manual steps for the client

- Confirm the favicon image is uploaded in Theme Settings if not already
  done.
- The `/pages/about` and `/pages/review` pages both need to be created and
  set active in the Shopify admin (client confirmed both are done).
- If automated review moderation is wanted later (approve/reject from a
  dashboard rather than manual copy-paste), that would require a third-party
  app like Judge.me/Loox — flagged as a future option, not built here.
