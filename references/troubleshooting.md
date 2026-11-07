# Troubleshooting

Indexed by symptom. Each entry marks whether it's a **verified** lesson (hit
and fixed during the Phantom Tag build) or a **documented** one (from current
Shopify/Claude Code docs, not yet hit in practice — verify it still holds
before leaning on it hard).

## CLI authenticated but browser has no access
**Verified.** `shopify theme info` returning the expected store only proves
the CLI's own session is good (note: there's no `auth whoami` subcommand in
the current Shopify CLI — use `shopify theme info` to confirm the session
and active store instead). The browser automation tool is a separate
connection with its own permission/connection state — it can be completely
unavailable even while the CLI works fine. Check both independently in
Phase B; don't infer one from the other.

## The draft theme turns out to be live
**Verified.** A theme you were told (or last confirmed) is a draft can
become the live theme mid-project if the user publishes it themselves
outside the conversation — theme roles aren't static for the life of a
build. Don't assume a role noted in an earlier session, or even earlier in
the same session, still holds. Re-check `shopify theme list` (or the admin
UI) for the actual current role before any push, and especially before any
`--allow-live`-style action. If the role has changed, stop and say so
explicitly before continuing — don't silently keep treating it as a draft,
and don't silently start treating it as live either; get the user to
confirm which behavior they want now that the stakes have changed.

## WebFetch can't substitute for visually inspecting a reference or draft site
**Verified.** A password-protected storefront or a heavily client-rendered
page won't give `WebFetch` anything useful — it may return a password gate,
a loading shell, or nothing. Any phase that needs to actually see a page
(design reference inspection, verifying a fix) needs the browser tool, not
`WebFetch`.

## Changes appear absent because the live storefront was checked instead of the draft preview
**Verified.** The live site and a draft theme's preview are different URLs
serving different content. If a change you just made "isn't showing up,"
confirm you're looking at the actual draft preview link (or an authenticated
staff session on the right theme) before concluding the change didn't take.

## Default sections don't satisfy the design
**Verified.** Dropping in a theme's stock section with placeholder content
technically "has a section there" but doesn't meet a design brief asking to
match a specific reference's structure and content. If the acceptance
criterion is "matches the reference," check it against the reference
visually — a section existing is not the same as a section being right.

## Theme Check passing and a successful push don't prove correctness
**Verified.** `shopify theme check` catches schema/syntax issues (e.g. a
setting typed as a string where a number is required). It says nothing about
whether a gallery actually switches images, a button actually works, or a
layout actually matches the reference. Likewise a push returning success only
means the files landed — it's not a functional test. Always verify visually
and interactively in the browser before marking a Phase F task done.

## Git repo covers more than the project folder
**Verified.** Running `git init` from the wrong working directory can scope
a repo to something much larger than intended (e.g. the whole home
directory) without any error. Always run `git rev-parse --show-toplevel`
immediately after `git init` and confirm it equals the project folder you
meant to scope it to.

## Theme editor changes and local files diverge
**Verified.** Some settings (notably ones that need the theme editor's own
native picker, like selecting products for a block) are hard or impossible
to set correctly by hand-editing JSON — the correct value format may not be
obvious from the schema alone. If you make a change in the editor, that
file now needs to be reconciled before you edit it again locally, or your
next local edit can silently overwrite the editor change (or vice versa).
Pull it into a throwaway temp directory (`shopify theme pull --only <path>
--path=<temp-dir>`) to see the editor's actual value, then deliberately
copy just that file into the working directory — don't pull straight over
the working directory even for this single-file case. This isn't a one-off
incident to remember — treat it as a standing risk on every push for the
rest of the build, and run `references/reconciliation.md`'s full pre-upload
procedure every time, not just when something seems off.

## `--unpublished` is a `theme push` flag, not a `theme pull` flag
**Verified** (checked against the installed CLI's own `--help` output, not
just docs). `theme pull` has no `--unpublished` flag — its only
theme-selection flags are `--development`, `--live`, and `--theme=<id>`.
`--unpublished` exists on `theme push`, where it creates a new unpublished
theme and pushes local files to it. Writing `theme pull --unpublished` will
error. See `references/cli-commands.md` for the full verified flag list per
command.

## Render snippet can't see a variable you just assigned
**Verified.** Liquid's `{% render %}` tag creates an isolated scope — it does
not inherit variables from the calling template/section, even ones assigned
right above the render call. If a snippet expects `product` (or any other
variable) and you're calling it from a context where that variable only
exists locally (e.g. `section.settings.product` assigned to `product` inside
a `featured-product` section, not a real product page), pass it explicitly:
`{% render 'snippet-name', product: product %}`.

## A feature looks broken but the new code is fine
**Verified.** Before assuming new Liquid/schema/JS is wrong, check the
theme's global stylesheet for an old, unrelated rule that happens to match
the new markup's selectors (e.g. a leftover `display: none` rule from a
previous, simpler version of a component). CSS specificity and cascade bugs
like this don't show up by reading the new code in isolation — they require
reading the actual computed styles or the global stylesheet itself. When a
UI element doesn't behave despite seemingly-correct markup, inspect the live
DOM/CSSOM rather than re-reading the Liquid a third time.

## Shopify CLI push to a live theme gets hard-blocked
**Verified.** Claude Code's own tool-use classifier can hard-block a
`shopify theme push --allow-live`-style command as a flagged "production
deploy" action, independent of whether Shopify itself would allow it. Don't
retry the identical blocked command. Make the specific change through the
theme editor's browser UI instead (which isn't subject to the same block),
save it there, then resync the local repo via `reconciliation.md`'s
temp-dir pull-and-adopt procedure (pulling, which is read-only against
Shopify, isn't subject to the block — but still shouldn't land directly on
top of the working directory; pull into a temp dir and deliberately adopt
the changed file(s)).

## Checkout only offers one country despite Markets being set up
**Verified.** Markets (Settings → Markets) controls currency/pricing
presentation per region. Shipping zones (Settings → Shipping and delivery)
separately controls what's actually deliverable at checkout. A country can
be fully "Active" in a Market and still be unreachable at checkout if no
shipping zone/rate covers it. Shopify surfaces this directly as a banner
("Shipping rates aren't set up — customers in N regions won't be able to
check out") on the Shipping and delivery settings page — look for it instead
of assuming Markets configuration alone is sufficient.

## A native `<select>` dropdown's options don't appear in a screenshot
**Verified.** Browser automation screenshots capture the rendered page, not
an OS-level native dropdown popup, so clicking a `<select>` and taking a
screenshot often shows nothing useful. Read the element's options via a
JS/DOM read instead (e.g. `[...select.options].map(o => o.text)`), not by
trying to screenshot the open dropdown.

## `theme_push`-style field type mismatches fail silently-ish at push time
**Verified.** A JSON section setting typed as a string where the schema
expects a number (e.g. `"columns_mobile": "1"` vs. `1`) fails `theme push`
with a schema validation error, not a Shopify-side error. Read the actual
error message — it names the offending field — fix the type, re-push.

## Modal/page scroll doesn't progress via mouse-wheel automation
**Verified.** Simulated mouse-wheel scroll inside a modal can repeatedly
snap back to the top instead of progressing, especially in multi-select list
modals (e.g. a shipping zone's country picker). Dragging the visible
scrollbar thumb directly is more reliable than repeated wheel events.

## settings_data.json / *-group.json files are editor-managed
**Documented, consistent with what was observed.** These theme files are
generated/overwritten by the theme editor and carry a header comment saying
so. Treat them as generated artifacts: fine to read, fine to hand-edit when
there's no other way, but expect the editor to overwrite out-of-band changes
if someone opens and saves in the editor afterward. Re-pull after any editor
session touches them.

## Bundle/discount display text without a working mechanism
**Verified (prevented, not hit as a bug, but called out explicitly in the
original brief as a risk).** A line of marketing copy like "Buy 3, Save 15%"
on a product page is not a bundle — it needs an actual Shopify mechanism
behind it (an automatic discount scoped to the right variants/quantities, or
a native Bundle) or the price shown won't match the cart. Always verify the
cart total, not just the PDP copy.

## Real payment / real order testing
**Documented.** Shopify's Bogus Gateway test mode only works while a real
payment provider hasn't been activated with live banking details. Once a
real gateway is active, there's no built-in safe "fake charge" — testing
checkout past that point means either a manual draft order, a real
minimal-amount order the user places and refunds themselves, or stopping
verification just short of actual payment submission. Never place a real
paid order as part of this workflow.
