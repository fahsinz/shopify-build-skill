---
name: shopify-build
description: Run a full Shopify store build (theme + store data + operations) from a filled-in STORE-BRIEF.md, in sequential verified phases, resuming across sessions via BUILD-STATUS.md. User-invoked only — covers everything from "finish this theme" to "stand up this whole store" for a new Shopify project.
disable-model-invocation: true
---

# Shopify Build

Sequential, resumable, multi-session workflow for building a Shopify store (theme +
store data + operations) from a brief. Built from lessons learned shipping the
Phantom Tag store — see `references/troubleshooting.md` for the incidents this
workflow exists to prevent.

*CLI commands/flags referenced throughout this skill were checked against the
**actually-installed** Shopify CLI (`shopify version` → 4.9.3) by running
`--help` on every command, on 2026-10-13 — not just read off shopify.dev,
since docs can drift from what's installed. See `references/cli-commands.md`
for the verified flag reference. One correction worth repeating here since
it's an easy mix-up: **`--unpublished` belongs to `theme push`, not
`theme pull`** — `pull` has no `--unpublished` flag. There is also no
`shopify auth whoami`; use `shopify theme info` to confirm the active
session/store. Re-verify with `shopify <command> --help` yourself if a
command starts erroring — CLI syntax changes across versions.*

## Starting a build (do this in a terminal, before opening Claude)

This skill only runs inside a Claude Code session, and it needs browser
access to do most of its work (Phases D, F, G, H all require the browser
tools). Get both right before invoking it:

1. In a terminal, `cd` into the project folder (a new empty folder under
   `~/shopify-projects/<store-name>/` for a new store, or the existing
   project folder to resume one). Don't run Shopify CLI commands or
   `/shopify-build` directly in PowerShell/bash outside Claude — the whole
   point of this skill is Claude driving the CLI and browser together with
   the safety rules below applied; running steps manually in the shell
   first just means redoing them inside Claude anyway, and risks a push or
   publish happening without the phase's acceptance checks.
2. Start Claude Code with the browser integration explicitly enabled:
   `claude --chrome`. This is a real, opt-in CLI flag ("Enable Claude in
   Chrome integration") — it is not on by default, so don't assume plain
   `claude` already has it just because a past session did. If you've
   independently verified it's enabled by default in your current install
   (the browser tools show up unprompted at session start), plain `claude`
   is fine — but verify that each time rather than assuming it still holds.
3. Once Claude starts, **verify the connection before doing anything
   else**: confirm the browser tools are actually available and can reach
   a real page (not just that the flag was accepted) before invoking this
   skill. Phase B's access check does this formally, but catching a broken
   connection here is faster than discovering it mid-phase.
4. Inside the Claude Code session (not the terminal), type `/shopify-build`.

## 0. Orient: new project or resume?

Check the current directory for `STORE-BRIEF.md` and `BUILD-STATUS.md`.

**Resuming** (both exist): read `BUILD-STATUS.md` first for the current
phase, what's done/blocked, and the next action — then run the full
reconciliation pass in `references/reconciliation.md` before doing any new
work. The log tells you what was true last session; reconciliation tells you
what's true now. Do not treat anything in `BUILD-STATUS.md` as still
accurate until reconciliation confirms it, including the theme's role, which
resources still exist, and whether local git and the remote theme have
diverged.

**New project**: confirm the project folder is a sibling under the agreed parent
location (e.g. `~/shopify-projects/<store-name>/`), not inside an unrelated
existing repo. Copy the four templates from `templates/` into this folder:
`CLAUDE.md`, `STORE-BRIEF.md`, `BUILD-STATUS.md`, `REQUIREMENTS-TRACE.md`
(this last one starts empty — Phase D populates it). Fill in `STORE-BRIEF.md`
together with the user — ask for whatever it needs that isn't already in the
conversation. Do not start Phase A until the brief has at minimum: store domain,
reference site(s), and whatever products/bundles exist at launch. Everything
else in the brief can be filled in incrementally as later phases need it, but
flag gaps rather than inventing values.

## 1. Standing safety rules (apply regardless of what STORE-BRIEF authorizes)

These override any "authorized actions" in the brief unless the brief's language
is unambiguous about that *specific* action:

- Never publish a theme live, activate a payment provider, install/purchase a
  paid app, change billing, or place a real paid order without asking in that
  moment — a general "you're authorized to publish when ready" in the brief
  still gets a final confirmation at the actual go-live moment.
- Never touch an existing live theme unless STORE-BRIEF explicitly names it as
  the target *and* says so — when in doubt which theme is "the" theme, stop and
  ask, don't infer from which one looks most complete.
- Before changing any setting that isn't scoped to a single draft theme
  (shipping zones, markets, policies, notification templates, payment
  settings pages even just to look) — record its current value in
  `BUILD-STATUS.md` before changing it. Store data (products, discounts,
  collections, menus) and operational settings affect the live storefront
  immediately regardless of theme draft status.
- Credentials, tokens, API keys never go in `CLAUDE.md`, `STORE-BRIEF.md`,
  `BUILD-STATUS.md`, or any git commit. Rely on the Shopify CLI's own
  authenticated session and an authenticated staff browser session.

## 2. Phase execution

Work the phases in `references/phases.md` (A through I) in order. Each phase
lists its own numbered tasks. Give **every numbered task its own row** in
`BUILD-STATUS.md`'s task log, with its own status and its own evidence —
don't track completion at the phase level only. A phase's acceptance
criteria pass when every one of its required tasks is `done` (with evidence)
or `n/a` (with a reason); if even one required task is `blocked` or
unverified, the phase is **not** complete, and any later phase that depends
on it does not start. Partial progress is fine and expected across sessions
— marking a phase complete when it isn't is not.

For each task: read the prerequisites, do the work, check the acceptance
criteria against real evidence (a screenshot, a CLI output, a DOM read — not
"the code looks right" or "theme check passed"), log it, and move on
automatically. Don't stop to ask permission for routine, in-scope tasks the
brief already authorizes.

Stop and ask only for: information `STORE-BRIEF.md` doesn't have and the
conversation hasn't supplied, a business/legal/financial decision, or an
action `references/phases.md` or the safety rules above mark as out of scope.

Record every task as done, blocked, or not-applicable-with-a-reason in
`BUILD-STATUS.md`. Never mark blocked work as done. Before creating any store
resource — product, variant, discount, menu, page, shipping zone, collection,
or app configuration — check whether it already exists first; reruns must
not create duplicates. Log what you found in the "Resources created/updated"
table either way (reused vs. newly created), so the next session's
reconciliation pass (`references/reconciliation.md`) has something to verify
against.

**"Theme complete" is not "store ready to launch."** Phase F finishing means
the theme itself — design, sections, interactions — is built and verified.
It does not mean the store can take real orders: Phase G (operational
setup), Phase H (end-to-end verification), and the out-of-scope user actions
in the blockers list (payment activation, paid apps, business/legal/tax
decisions, final publish) all still gate real launch-readiness. State these
two as separate, explicit statuses — never let "the theme looks done" imply
the store is ready to go live. `templates/REQUIREMENTS-TRACE.md` tracks both
explicitly in its closing summary.

## 3. Reference files

- `references/phases.md` — the detailed phase-by-phase procedure: prerequisites,
  tasks, concrete operations, acceptance criteria, failure recovery, which
  settings are shared/live, and required evidence. Read the current phase's
  section before starting it.
- `references/cli-commands.md` — verified Shopify CLI flag reference (checked
  against the installed CLI, not just docs). Check a command here before
  running it if you're not certain of a flag.
- `references/reconciliation.md` — required reconciliation steps: on every
  session resume, and before every theme upload. Don't skip these thinking
  `BUILD-STATUS.md` already covers it — the log is a record, not a guarantee.
- `references/troubleshooting.md` — known failure modes and their fixes, indexed
  by symptom. Check here first when something doesn't behave as expected.
- `references/shrine-theme-notes.md` — structural knowledge of the Shrine theme
  (color system, gallery snippet, cart-drawer upsell blocks, known stale-CSS
  trap). **Only load this after Phase A/B has actually identified the target
  theme as Shrine or a close fork of it** by reading the theme's own files —
  don't load it on the assumption that it's probably Shrine just because
  past stores used it. For any other theme, skip this file and derive the
  equivalent structure by reading that theme's own files first (same
  method, different file names). Even when Shrine is confirmed, treat every
  specific claim in that file as something to verify against *this* store's
  actual files before relying on it — Shrine has shipped multiple versions,
  and a mismatch there means the file is stale for this version, not that
  the store is wrong.
- `references/verification-checklist.md` — the concrete checklist for Phase H.
- `templates/` — `CLAUDE.md`, `STORE-BRIEF.md`, `BUILD-STATUS.md`,
  `REQUIREMENTS-TRACE.md` starting points for a new project.
