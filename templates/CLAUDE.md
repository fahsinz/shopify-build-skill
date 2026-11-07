# <STORE NAME>

Shopify store build. The full build workflow lives in the `shopify-build`
personal skill (`~/.claude/skills/shopify-build/`) — invoke it with
`/shopify-build` to start or resume. Don't duplicate its procedures here;
this file is just the per-project pointer and the non-negotiable defaults.

## Project facts
- Store domain (`*.myshopify.com`): <PLACEHOLDER>
- Live custom domain (if any): <PLACEHOLDER>
- Target theme ID(s): <PLACEHOLDER> — role (draft/live) must be reconfirmed
  live each session, never assumed from here.
- Brief: see `STORE-BRIEF.md`
- Status / next action / reconciliation log: see `BUILD-STATUS.md`
- Per-requirement implementation + verification trace: see
  `REQUIREMENTS-TRACE.md`. "Theme complete" and "store launch-ready" are
  tracked separately there — never assume one from the other.

## Standing defaults for this project
(Mirrors `shopify-build`'s safety rules — repeated here so they're visible
without opening the skill.)
- Never publish a theme, activate payments, install a paid app, or place a
  real order without asking in that moment, even if authorized generally.
- Never touch a theme other than the one named above without explicit
  confirmation that it's now the real target.
- Record the previous value of any shared/live setting before changing it.
- No credentials or tokens in this file, the brief, the status file, or git.
