# Build phases

Work these in order, A → I. Each phase lists: prerequisites, tasks, concrete
operations, acceptance criteria (what you must actually observe, not infer),
failure recovery, whether it touches shared/live store data, and the evidence
to log in `BUILD-STATUS.md` before moving on.

"Shared/live data" means: visible or effective on the real store regardless of
whether the theme itself is published. Products, discounts, collections,
shipping zones, markets, policies, and domain settings are all shared — a
customer or the live theme can be affected by them even while you're only
editing a draft theme.

---

## A. Intake and audit of existing work

**Prerequisites:** `STORE-BRIEF.md` has at least the store domain and a note on
whether prior theme/store work already exists.

**Tasks**
1. If the brief says prior work exists, inspect it before writing anything —
   `shopify theme list`, pull/read the existing theme, browser-check the
   current draft and live states. Don't assume a blank slate just because the
   brief is thin on detail.
2. **Identify the theme itself** by reading its actual files — theme name
   and version (check `config/settings_schema.json`'s theme info, `layout/theme.liquid`
   header comments, and/or `locales/en.default.json` for a theme-name
   string), not by assuming it matches a prior project. This determines
   whether `references/shrine-theme-notes.md` applies: only load/trust it
   once this task has confirmed the theme is actually Shrine or a close
   fork — for any other theme, note that here and plan to derive its
   structure independently in Phase F by reading its own section/snippet
   files when you get there.
3. Compare what exists against what the brief (and Phase D's reference
   inspection, once available) will require. Produce a written inventory:
   what's done, what's missing, what's wrong.

**Operations:** `shopify theme list`; `shopify theme pull` into a scratch
read; browser screenshots of admin + storefront; reading theme files to
identify the theme (no CLI command names the theme for you).

**Acceptance criteria:** a written inventory exists in `BUILD-STATUS.md`
listing current state per area (theme, products, pages, shipping, etc.),
each tagged done / missing / wrong; the theme's identity (name, and Shrine
or not) is explicitly recorded, not assumed.

**Failure recovery:** if you can't reach the store or theme at all, this
phase can't complete — go establish access in Phase B first, then come back.

**Shared data touched:** none (read-only phase).

**Evidence to log:** the inventory itself; CLI/screenshot outputs it's based on.

---

## B. Tooling, Shopify authentication, browser connection, access verification

**Prerequisites:** none — this is the first real setup phase.

**Tasks**
1. Confirm the Shopify CLI is installed and authenticated: run
   `shopify auth login` (it's a no-op if already logged in) and
   `shopify theme info`, which echoes back the current store and confirms a
   working session — there is no `whoami` subcommand in the current CLI.
   This is necessary but **not sufficient** — CLI auth and browser access
   are separate failure points and both must be checked independently. See
   [troubleshooting: CLI authenticated but browser has no access](troubleshooting.md#cli-authenticated-but-browser-has-no-access).
2. Confirm browser automation is loaded and actually reaches the real admin
   (`https://admin.shopify.com/store/<handle>`) as an authenticated staff
   session showing real data — not just that the extension responds to a ping.
   Load every browser tool you expect to need in one batched call, not one at
   a time.
3. Confirm, out loud, which store and which theme ID(s) are the actual
   target — read this back from the CLI/admin UI, never from memory or
   assumption, and state each theme's current role (draft / unpublished /
   live) before doing anything to it. This single check is the highest-
   leverage one in the whole workflow: theme roles can change between
   sessions without anyone telling you (see
   [troubleshooting: draft theme turns out to be live](troubleshooting.md#the-draft-theme-turns-out-to-be-live)).
4. If the storefront is password-protected, confirm the authenticated staff
   browser session bypasses the password screen (it normally does, via the
   theme preview bar) rather than trying to use the storefront password as a
   workaround.

**Operations:** `shopify auth login`; `shopify theme info`; browser
navigation to admin + storefront; `shopify theme list` with role column
read back explicitly.

**Acceptance criteria:** `shopify theme info` returns the expected store;
browser shows the real admin dashboard; target theme ID and its current
role are explicitly stated and confirmed before Phase C begins.

**Failure recovery:** if CLI auth works but the browser can't reach the
store, don't try to infer store state from `WebFetch` instead — it can't
substitute for visually inspecting a password-protected or JS-rendered page.
Fix the browser connection, or ask the user for a direct screenshot.

**Shared data touched:** none (read-only phase).

**Evidence to log:** `shopify theme info` output; admin dashboard
screenshot; the explicit theme ID + role statement.

---

## C. Local repository, backups, draft theme, target verification

**Prerequisites:** Phase B complete.

**Tasks**
1. Create the project folder under the agreed parent location
   (`~/shopify-projects/<store-name>/`), not inside an unrelated existing
   repo or the home directory root. Immediately after `git init`, run
   `git rev-parse --show-toplevel` and confirm it equals the intended project
   folder — see
   [troubleshooting: git repo covers more than the project folder](troubleshooting.md#git-repo-covers-more-than-the-project-folder).
2. **If `STORE-BRIEF.md` names no existing theme ID** (a genuinely fresh
   store, or one still on its default theme), the target theme doesn't
   exist in the store's theme library yet — there's nothing for `theme
   pull` to pull. Figure out how it gets there before proceeding: a
   Theme Store purchase/install (done by the user in the admin UI — this
   is a purchasing/licensing decision, not something to do silently even
   if it's a free theme), or pushing a theme codebase the user already has
   (`shopify theme push --unpublished`, which creates the new unpublished
   theme from local files instead of pulling one down). Confirm which
   applies with the user rather than assuming; once the theme exists in
   the store, continue with task 3 below as normal.
3. Pull the target theme down (`shopify theme pull --theme=<id>`, see
   `cli-commands.md` for the full flag reference) into that folder, *then*
   `git init` and make a first commit
   ("Checkpoint: initial draft theme as downloaded") before any edits. This
   commit is the rollback point for everything that follows.
4. Re-confirm the theme's role again, right here, even though Phase B just
   checked it — it can change between the two phases if the user (or anyone
   else) publishes something independently.
5. Local git only: commit as a checkpoint after each meaningful unit of work
   (a phase, or a risky edit within one). No GitHub remote for this workflow.
6. This first commit is also the initial "last synced baseline" — record
   its hash in `BUILD-STATUS.md`. From this point on, every future push in
   Phase F follows `reconciliation.md`'s pre-upload steps (temp-dir pull
   via `--path`, three-way compare against baseline, merge deliberately,
   re-confirm store+theme ID, push, verify, only then advance the
   baseline) — not just this first pull. Note that here so it isn't
   treated as a one-time Phase C formality.

**Operations:** `shopify theme pull`; `git init`; `git rev-parse --show-toplevel`;
`git add` + `git commit`; `shopify theme list` (role re-check).

**Acceptance criteria:** `git rev-parse --show-toplevel` equals the project
folder; the first commit exists and matches a fresh pull; theme ID + role
reconfirmed and logged.

**Failure recovery:** if the git toplevel is wrong, do not run destructive
git commands against it — re-`git init` inside the correct folder, or move
the misscoped `.git` aside and start over there.

**Shared data touched:** none (local-only phase).

**Evidence to log:** `git rev-parse --show-toplevel` output; `git log -1`;
theme role reconfirmation.

---

## D. Reference inspection and requirements inventory

**Prerequisites:** `STORE-BRIEF.md` has reference site(s) listed; Phase C
complete.

**Tasks**
1. Visually inspect every reference site via the browser — screenshots and
   DOM/text reads — not `WebFetch` alone. `WebFetch` can't substitute for
   visual inspection, especially behind a password or heavy client-side
   rendering.
2. Extract what's actually there: copy, imagery, reviews/ratings if present,
   stated benefits, FAQs, guarantees, bundle structure, prices, discounts,
   promo messaging, interaction patterns (gallery behavior, upsell
   placement, sticky elements, etc.). Separate what's adaptable to the new
   brand from what must never be invented — fabricated reviews, specs,
   guarantees, or shipping promises that aren't in `STORE-BRIEF.md`.
3. For every content/copy/imagery item pulled from a reference, apply the
   migration mode `STORE-BRIEF.md`'s References section specified for that
   reference — **migrate-as-is**, **adapted**, or **new-creation** (replace
   entirely, reference used only as a structural example). Don't default to
   "adapt" when the brief didn't actually say that; if a reference or
   content area has no migration mode specified, that's a gap — ask, don't
   assume. This distinction matters for rights/authorization, not just
   style: migrating content as-is is a different, more consequential action
   than writing new copy inspired by it, and the brief's answer on this is
   what authorizes which one happens.
4. Where the reference site and `STORE-BRIEF.md` disagree (e.g. a bundle
   discount shown on the reference but not specified in the brief), the
   brief's actual data always wins. Note the discrepancy rather than silently
   picking one.
5. Produce a written, section-by-section requirements inventory in
   `BUILD-STATUS.md` that becomes the task backlog for Phases E–G, **and**
   create the corresponding rows in `templates/REQUIREMENTS-TRACE.md` (now
   copied into the project as `REQUIREMENTS-TRACE.md`) — one row per
   requirement, with its source and, for content items, its migration mode
   filled in now even though implementation/verification won't be filled in
   until later phases.

**Operations:** browser screenshots + DOM/text reads of every reference URL
and every existing page of the target store.

**Acceptance criteria:** the inventory covers every `STORE-BRIEF.md`
requirement category (products, bundles, discounts, shipping, pages,
policies) with a source noted for each; `REQUIREMENTS-TRACE.md` has a row
for each one with source and migration mode (where applicable) filled in.

**Failure recovery:** if a reference site needs access the user hasn't
supplied (login-walled, etc.), ask rather than guessing its content.

**Shared data touched:** none (read-only phase).

**Evidence to log:** the inventory; the screenshots it's based on.

---

## E. Store-data configuration (products, variants, bundles, discounts, inventory, collections, metafields)

**Prerequisites:** Phase D inventory exists.

**Tasks**
1. Before creating anything, search for an existing resource by handle,
   title, or SKU and reuse/update it if found. Reruns of this phase must
   never create duplicate products, variants, discounts, or collections.
2. Create/update products with variants, SKUs, prices, and currencies exactly
   as `STORE-BRIEF.md` specifies — never invent a price, SKU, or spec that
   isn't in the brief or sourced from the reference and flagged as such.
3. Build bundles as **real, working mechanisms**, not display text. For
   quantity-tier pricing (e.g. "buy 3, save 15%"), Shopify's automatic
   "Amount off products" discounts scoped to the specific variants work well
   and were used successfully on Phantom Tag; native Shopify Bundles may fit
   better depending on what the brief actually needs — pick based on the
   brief's inventory model, not habit.
4. Record every created/updated resource's ID or handle, and its previous
   value if this was an update, in `BUILD-STATUS.md`. This is required both
   for idempotency on reruns and for rollback.
5. Set new products to draft/unpublished status until Phase I, unless
   `STORE-BRIEF.md` explicitly authorizes otherwise — products are live
   store data and visible immediately once published, independent of the
   theme's own draft status.

**Operations:** Shopify admin product/discount/collection creation (CLI,
Admin API, or browser — whichever the session has working); admin search
before create.

**Acceptance criteria:** every product/bundle/discount named in
`STORE-BRIEF.md` exists in the store, with concrete, checked facts logged
against each — not just "it exists": variant IDs/SKUs match what the brief
specified, inventory quantities/tracking are set correctly, and each
discount's eligibility rule (quantity threshold, scoped variants, stacking
behavior) matches the brief exactly, verified by reading the discount's
actual configured conditions in admin, not by re-reading the brief's
intent. A quick add-to-cart sanity check shows correct cart math for at
least one combination, including a boundary case (one unit below a
discount's quantity threshold does *not* get the discount). Displayed offer
text on a page is not evidence by itself — full checkout-total verification
happens in Phase H, but this phase's own acceptance already requires more
than "the product page looks right."

**Failure recovery:** if a duplicate is found from a prior partial run,
update it in place — do not create a second copy.

**Shared data touched:** yes — products, discounts, collections, and
inventory are live store data. Record what was created/changed.

**Evidence to log:** resource IDs/handles created or updated; screenshots of
the product/discount admin pages.

---

## F. Storefront implementation (theme settings, sections, pages, navigation, product selection, cart, responsive design)

**Prerequisites:** Phase E resources exist, so sections can reference real
products.

**Tasks**
1. Work only on the draft/target theme ID confirmed in Phases B/C. Before
   every push, run `reconciliation.md`'s pre-upload steps in full
   (temp-dir pull via `--path`, three-way compare against the last synced
   baseline, merge deliberately, re-confirm store+theme ID+role, push,
   verify, advance the baseline) — this applies to every push in this
   phase, not just the first one back in Phase C. Never pull directly into
   this working directory just to check for drift.
2. When building navigation/menus, check for an existing menu with the same
   handle first and update it rather than creating a duplicate — same
   idempotency rule as Phase E's resources, applies here too since menus are
   store data, not theme files.
3. Don't satisfy a design requirement with a default/placeholder section —
   match the reference's actual structure (layout, section order,
   interaction behavior) with real section/snippet work. See
   [troubleshooting: default sections don't satisfy the design](troubleshooting.md#default-sections-dont-satisfy-the-design).
4. When reusing or porting a snippet (e.g. a media gallery) via `render`,
   remember it creates an isolated Liquid variable scope — a locally
   assigned variable like `product` does not automatically reach the
   rendered snippet; pass it explicitly as a render parameter. See
   [troubleshooting: render snippet can't see a variable you just assigned](troubleshooting.md#render-snippet-cant-see-a-variable-you-just-assigned).
5. If a feature looks broken after what seems like correct Liquid/schema
   work (a slider not switching, an element not showing), check the theme's
   global stylesheet(s) for a stale, unrelated rule before assuming the new
   code is wrong. See
   [troubleshooting: a feature looks broken but the new code is fine](troubleshooting.md#a-feature-looks-broken-but-the-new-code-is-fine).
6. After making a change directly in the theme editor (not in local files —
   e.g. toggling a block setting that only the editor's native picker can
   set correctly), pull it back into the local repo before making further
   local edits. See
   [troubleshooting: theme editor changes and local files diverge](troubleshooting.md#theme-editor-changes-and-local-files-diverge).
7. Do an explicit mobile/responsive pass — not just desktop.
8. Once every task above and every theme-related row in
   `REQUIREMENTS-TRACE.md` is done or n/a, log the theme as **complete** in
   `BUILD-STATUS.md` explicitly — and just as explicitly note that this is
   not the same as the store being launch-ready (see SKILL.md §2). Phases
   G and H still gate real launch.

**Operations:** `shopify theme push` (to the confirmed draft theme only);
theme editor (browser) for anything that needs its native pickers;
`reconciliation.md`'s temp-dir pull-and-adopt procedure to resync after
editor changes; `shopify theme check`. See `cli-commands.md` for exact
flags.

**Acceptance criteria:** every section in the Phase D inventory renders with
real content on both desktop and mobile, and interactive elements (gallery
switching, arrows, upsell toggles, variant pickers) actually function —
verified live in a browser by performing the interaction, not inferred from
`theme check` passing or a push succeeding. Neither proves visual or
functional correctness.

**Failure recovery:** if a direct-to-live push gets hard-blocked by Claude
Code's own automation classifier (a tooling behavior, not a real Shopify
error — happens on `--allow-live`-style pushes), don't retry the identical
blocked call. Fall back to making that specific change in the theme editor
browser UI, save there, then resync locally via `reconciliation.md`'s
temp-dir pull-and-adopt procedure — not a direct pull over the working
directory, even though in this specific case you know exactly which one
change you're adopting. If the theme's role changed to live since Phase C,
stop and reconfirm before proceeding — don't assume last check still holds.

**Shared data touched:** none if genuinely scoped to the draft theme; flag
immediately if the role has changed.

**Evidence to log:** before/after screenshots per major section, desktop and
mobile, with interaction proof (e.g. a screenshot sequence or DOM read
showing a thumbnail click actually changed the active image).

---

## G. Operational setup (shipping, markets/currency, policies, domain, notifications, analytics, integrations)

**Prerequisites:** Phase E/F substantially underway (brief's shipping/markets
data can be configured in parallel once known).

**Tasks**
1. Treat **Markets** (Settings → Markets) and **Shipping zones/rates**
   (Settings → Shipping and delivery) as two separate systems that must both
   be configured — a country being "Active" in a Market does not make it
   deliverable at checkout. Always check for a Shopify-surfaced warning like
   "Shipping rates aren't set up for N regions" rather than assuming Markets
   alone is sufficient. See
   [troubleshooting: checkout only offers one country despite markets being set up](troubleshooting.md#checkout-only-offers-one-country-despite-markets-being-set-up).
2. Before changing any shared/global setting, read and record its current
   value in `BUILD-STATUS.md` so it can be restored if needed.
3. Configure domain, policies, notifications, and analytics per
   `STORE-BRIEF.md`; anything the brief leaves blank gets skipped with a
   logged reason, never silently.
4. Payment provider activation, tax/business verification, and any paid app
   purchase are **out of scope** for this skill to perform, even if asked —
   they require real financial/business credentials and account
   verification only the user can provide. Surface them as required user
   actions in `BUILD-STATUS.md`'s blockers list.
5. For any free/already-installed app this phase needs to configure (e.g. an
   analytics pixel app already on the store), check whether it's already
   installed and configured before touching it — same idempotency rule as
   products/discounts/menus. This does not relax the rule above: installing
   a *new* app, free or paid, is still never done without asking first.

**Operations:** Shopify admin Settings pages (Markets, Shipping and
delivery, Policies, Domains, Notifications); browser DOM/JS reads to verify
dropdown contents rather than trusting a screenshot of a native `<select>`
(its options don't render visually until opened, and automated screenshots
often can't capture a native OS dropdown at all — read the DOM instead).

**Acceptance criteria:** checkout is reachable end-to-end for every shipping
destination `STORE-BRIEF.md` requires, verified by actually opening checkout
and reading the country/shipping-method list via a DOM/JS read.

**Failure recovery:** if a shared setting change breaks something unrelated,
restore it from the recorded previous value.

**Shared data touched:** yes, entirely — log every change made in this
phase for the final handoff report regardless of theme publish status.

**Evidence to log:** DOM/JS-read proof of checkout dropdown contents;
shipping zone list; before/after values of any changed shared setting.

---

## H. End-to-end verification and correction

**Prerequisites:** F and G complete.

**Tasks:** work the full checklist in `verification-checklist.md` — desktop
and mobile passes over navigation, PDP/variants, add-to-cart, cart
updates/upsells, discount combinations (displayed price matches cart matches
checkout), shipping availability per destination, and checkout using a
non-paying test method:
- If no real payment provider is active yet, Shopify's Bogus Gateway test
  mode can simulate a full paid order safely — use it.
- If a real payment provider is already active, do not attempt any real
  charge. Verify everything up to (but not including) submitting real
  payment, and log in `BUILD-STATUS.md` that final live-payment checkout
  remains the user's own action.

**Operations:** browser end-to-end flows; Bogus Gateway test orders where
available; DOM/JS reads for anything a screenshot can't confirm (native
dropdowns, computed CSS, cart totals).

**Acceptance criteria:** every checklist item is explicitly pass / fail /
not-applicable-with-a-reason in `BUILD-STATUS.md`, backed by the concrete
evidence the checklist itself demands for that item — a cart/checkout total
actually read off the page, an admin record actually opened, a DOM read of
actual dropdown contents. An item marked pass on the strength of "the offer
text displays correctly" without that backing evidence doesn't count as
verified; mark it fail or go get the real evidence. A failure routes back to
the phase it belongs to (data issue → E, display issue → F, shipping/market
issue → G) for a real fix, then rechecks here — it doesn't get patched ad
hoc inside Phase H. Once this phase and `REQUIREMENTS-TRACE.md` are both
fully green, that's what "store launch-ready" (modulo the blockers list)
actually means — not Phase F finishing alone.

**Failure recovery:** see above — route back, fix at the source, recheck.

**Shared data touched:** testing may create cart/checkout sessions; no new
persistent shared-data writes expected beyond what E/G already made.

**Evidence to log:** the completed checklist; screenshots for each major
flow.

---

## I. Reviewable handoff and separately authorized publication

**Prerequisites:** Phase H checklist fully passed, or every remaining
failure explicitly accepted/deferred by the user.

**Tasks**
1. Finish `REQUIREMENTS-TRACE.md`: every row done or n/a, and its
   theme-complete / store-launch-ready summary filled in honestly — a "no"
   with a clear list of what's outstanding is a normal, correct outcome
   here, not a failure to report.
2. Compile a handoff report: what was built, the draft preview link,
   screenshots, every shared/live setting touched across C/E/G (consolidated,
   not just listed per-phase), everything marked blocked/not-applicable and
   why, the explicit list of actions still requiring the user (payment
   activation, paid apps, business/legal/tax decisions, final publish), and
   the requirements-to-evidence table from `REQUIREMENTS-TRACE.md` itself —
   point to it or inline it, don't summarize it away.
3. State plainly, as two separate lines, whether the theme is complete and
   whether the store is launch-ready — never let the first imply the
   second.
4. Do not publish the theme, activate payments, or purchase apps as part of
   "finishing" the build — publication is always a separate action, asked
   for and confirmed in its own turn.

**Operations:** none beyond compiling the report from `BUILD-STATUS.md` and
`REQUIREMENTS-TRACE.md`.

**Acceptance criteria:** `REQUIREMENTS-TRACE.md` is fully resolved (every
row terminal); the user has the report, including the theme-complete /
launch-ready distinction and the requirements-to-evidence table, and has
confirmed or deferred publication themselves.

**Failure recovery:** not applicable — terminal phase.

**Shared data touched:** nothing new; the report must enumerate everything
shared-data-related from earlier phases one more time, consolidated.

**Evidence to log:** the handoff report; `BUILD-STATUS.md` fully updated to
its terminal state.
