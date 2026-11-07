# Shrine theme notes

Structural knowledge of the "Shrine" Shopify theme, captured from the Phantom
Tag build.

**Load this only after Phase A/B has already confirmed, by reading the
target theme's own files, that it's Shrine or a close fork of it.** Don't
open this file on the assumption that a new store "probably" uses Shrine
because a past one did — check first. For any other theme, this file does
not apply; derive the equivalent structure by reading that theme's own
files (same method, different file names).

Even once Shrine is confirmed, everything below is a **hypothesis to verify
against this store's actual files, not ground truth**. Shrine has shipped
multiple versions, and a theme vendor can rename settings or restructure
files between them. If something here doesn't match what you find in this
store's copy, trust the files in front of you — that's a version mismatch
with this doc, not a bug in the store.

## Global color system

Colors are **not** per-section `color_schemes` (Shrine doesn't have that key).
There's a single global palette, set in `config/settings_data.json` under the
top-level `current` settings object:

- `colors_accent_1`, `colors_accent_2`
- `colors_background_1`, `colors_background_2`
- `colors_text`
- `colors_solid_button_labels`, `colors_outline_button_labels`

These get converted to CSS custom properties in `layout/theme.liquid`, and
consumed throughout `assets/base.css` via classes like `.color-background-1`,
`.color-accent-1`, `.color-inverse`. To re-theme the whole store, change
these values once in `settings_data.json` — don't hunt for per-section color
settings that don't exist.

Individual sections still carry a `color_scheme` setting (e.g.
`"color_scheme": "background-1"` vs `"inverse"` vs `"accent-1"`) that picks
*which* of the above roles that section uses — check what each section
currently has set if a palette swap makes something unexpectedly bright or
unreadable (e.g. a footer set to `"accent-1"` will render in the new accent
color, which may not suit a footer).

## Product media gallery

`snippets/product-media-gallery.liquid` is the canonical, full-featured
gallery (thumbnail slider, arrows, dots, mobile pagination) used by
`sections/main-product.liquid`. It needs:

1. The render call, with `product` passed explicitly (render creates an
   isolated scope — see troubleshooting.md):
   `{% render 'product-media-gallery', product: product, variant_images: variant_images %}`
2. A matching set of schema settings on the *calling* section — if you're
   porting this gallery into a different section (e.g. `featured-product`
   for a homepage buy-box), copy the full settings block from
   `main-product.liquid`'s schema, not just the render call. At minimum:
   `gallery_layout`, `mobile_thumbnails`, `desktop_thumbnails_count`,
   `mobile_thumbnails_count`, `display_variant_image_first`, `hide_arrows`,
   `dots_color_scheme`, `mobile_pagination`, `mobile_preview_media`,
   `mobile_spacing`, `trust_badge`, `trust_badge_position`, `trust_badge_size`.
3. CSS custom properties tied to the thumbnail-count settings
   (`--desktop-thumbnails` / `--mobile-thumbnails`), and the product wrapper
   div needs classes like `product--{{ section.settings.gallery_layout }}
   product--mobile-{{ section.settings.mobile_thumbnails }}` — `base.css`'s
   `is-active` visibility rules depend on these classes being present.
4. Watch for a stale rule in `assets/base.css` scoped to the *old*,
   simpler single-image gallery markup (something like
   `.featured-product .product__media-item:not(:first-child) { display: none; }`)
   that can silently override the new gallery's slide-switching logic by
   selector specificity, not by anything wrong in the new code. If slides
   render but don't switch on click, check for this before touching the
   Liquid again — grep `base.css` for the section's class name.

## Cart-drawer cross-sell upsells

`sections/cart-drawer.liquid` (or its group config) supports a
`product_upsells` block type for pre-checkout cross-sells. Two things that
aren't obvious from the schema alone:

- `product_list` expects **product handles** (e.g.
  `["shipping-protection", "skip-the-line"]`), not GIDs. Setting GIDs
  directly resolves as "not available" in the editor.
- The per-product "preselected" setting IDs have a typo baked into the
  schema: `prdouct_1_preselected`, `prdouct_2_preselected` (not
  `product_1_preselected`). Setting the correctly-spelled key silently does
  nothing.

The most reliable way to get the right values: use the theme editor's native
product picker and preselect toggles in the browser, save, then follow
`reconciliation.md`'s single-file pull-and-adopt procedure (pull
`config/settings_data.json` into a temp directory, then deliberately copy
it into the working directory) to bring the correct values back into the
local repo — rather than guessing the JSON shape by hand, and rather than
pulling straight over the working directory.

## Header/footer group sections

`sections/header-group.json` and `sections/footer-group.json` are
editor-managed group configs (see troubleshooting.md). Notes:

- The header's horizontal-ticker and any announcement-bar section both carry
  their own `color_scheme` setting independent of the global palette swap —
  check these explicitly after a re-theme.
- `footer-group.json`'s footer section has a `blocks` object *and* a
  `block_order` array that must be kept in sync — removing a block (e.g. a
  "Quick links" `link_list` block) means deleting its entry from *both*
  places, not just one, or the theme will error or show a phantom gap.

## Theme push to live

See troubleshooting.md's entry on the Claude Code classifier blocking
`--allow-live` pushes — applies to Shrine the same as any theme, included
here as a pointer since it comes up often when iterating on a published
Shrine theme.
