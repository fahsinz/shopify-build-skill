# Build status: <STORE NAME>

Updated after every task. Read this first when resuming a session — then run
the full reconciliation pass in the `shopify-build` skill's
`references/reconciliation.md` before trusting anything below as still
accurate. This file is a record of what was true last time someone checked,
not a guarantee of what's true now.

## Current phase
<A | B | C | D | E | F | G | H | I> — <one-line description of where things
stand>

## Theme complete vs. store launch-ready (two separate questions — see SKILL.md §2)
- Theme complete (Phase F done, every theme row in `REQUIREMENTS-TRACE.md`
  terminal): <yes/no> — if no: <what's outstanding>
- Store launch-ready (F + G + H all done, every blocker below resolved):
  <yes/no> — if no: <what's outstanding>

## Next action
<the single concrete next step>

## Access confirmation (re-verify every session, and again before every upload — see reconciliation.md)
- CLI authenticated as: <PLACEHOLDER> (last checked: <date>)
- Browser confirmed reaching admin: <yes/no> (last checked: <date>)
- Target theme ID: <PLACEHOLDER> — role: <draft/unpublished/live>
  (last checked: <date>)
- Last synced baseline commit: <PLACEHOLDER — git hash> (verified synced on:
  <date>). Only move this forward after a push has been verified to have
  landed — see `reconciliation.md`'s pre-upload steps. Do not advance this
  on a push that wasn't confirmed, or on a plain local commit that was
  never pushed.

## Reconciliation log
| Date | Type (resume / pre-upload) | Findings | Action taken |
|---|---|---|---|
| | | | |

## Shared/live settings changed (carry forward into the Phase I report)
| Setting | Previous value | New value | Phase | Date |
|---|---|---|---|---|
| | | | | |

## Resources created/updated (idempotency log — check before recreating)
| Type | Handle/ID | Phase | Status | Notes |
|---|---|---|---|---|
| | | | | |

(Type includes products, variants, discounts, collections, menus, and any
app configuration touched — not just products.)

## Task log
Every numbered task from `references/phases.md`'s current phase gets its own
row here — not just one row per phase. A phase is not complete while any of
its required tasks are blocked or unverified.

| ID | Phase | Task | Dependencies | Status | Evidence | Notes |
|---|---|---|---|---|---|---|
| | | | | not started / in progress / blocked / done / n/a | | |

(Status values: **not started**, **in progress**, **blocked** — with a
reason — **done** — with evidence logged — or **n/a** — with a reason. Never
mark blocked work as done.)

## Blockers
- <anything needing the user's input: business/legal/financial decisions,
  missing brief data, actions outside authorized scope — payment activation,
  paid apps, and final publish always belong here until the user resolves
  them themselves>

## Verification checklist status (Phase H)
See `references/verification-checklist.md` in the `shopify-build` skill.
Summarize pass/fail/n/a counts here once run; link to the full filled
checklist if kept separately. Also see `REQUIREMENTS-TRACE.md` for the
per-requirement implementation/verification trace.
