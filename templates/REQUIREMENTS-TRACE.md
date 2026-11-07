# Requirements trace: <STORE NAME>

Every requirement from `STORE-BRIEF.md` (and anything added from reference
inspection in Phase D) gets a row here, created in Phase D and filled in as
later phases do the work. By Phase I, every row must have an implementation
and a verification result — a row with "displayed on the page" as its only
verification is not done; it needs a verification method that actually
exercises the backend (cart total, admin record, checkout behavior), not
just the presence of text.

| # | Requirement | Source | Content mode | Implementation | Verification method | Verified result | Status |
|---|---|---|---|---|---|---|---|
| | | brief / reference / invented-flagged | migrate-as-is / adapted / new-creation | file(s), section(s), or resource ID/handle that implements it | which check in `verification-checklist.md`, or a specific manual check | pass / fail / n/a-with-reason | not started / in progress / blocked / done |

**Column notes**
- **Source**: where the requirement came from. `invented-flagged` means it
  isn't in the brief or a reference and must be confirmed with the user
  before being treated as real — never leave a row at this source without
  following up.
- **Content mode** (only applies to copy/imagery/design-pattern rows, per
  `STORE-BRIEF.md`'s References section): whether this specific item is a
  literal migration from a reference site, an adaptation of it, or new
  content created for this brand. A row claiming "migrate-as-is" for
  something `STORE-BRIEF.md` only authorized as "adapted" is a defect, not
  a style choice.
- **Verification method**: must name something that was actually checked —
  a checklist item, a DOM read, an admin screenshot showing a real record,
  a cart/checkout total. "Looks right on the page" is not a verification
  method.
- **Status**: same values as `BUILD-STATUS.md`'s task log. Never mark a row
  done while its verification result is blank or marked fail.

## Theme-complete vs. launch-ready summary

Fill this in once every row above is in a terminal state (done or
n/a-with-reason):

- **Theme complete** (Phase F finished: every section/page/interaction in
  this table that belongs to the theme itself is built and passes its
  verification): yes / no — if no, list what's outstanding.
- **Store launch-ready** (Phase F *and* G *and* H all complete, plus every
  out-of-scope user action from `BUILD-STATUS.md`'s blockers list is
  actually resolved — payment activation, paid apps, business/legal/tax
  decisions): yes / no — if no, list what's outstanding. A "yes" here is
  never claimed solely because the theme looks finished.
