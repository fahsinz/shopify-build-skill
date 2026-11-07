# Shopify CLI command reference (verified against the installed CLI)

Verified by running `--help` against the actually-installed CLI
(`shopify version` → **4.9.3**) on 2026-10-13, not just read off shopify.dev —
docs pages can lag or drift from what's actually installed. If commands start
erroring, re-run `shopify <command> --help` yourself and trust that output
over this file; note the new version here if you update it.

**The single most important correction this file exists to encode:**
**`--unpublished` belongs to `theme push`, not `theme pull`.** `theme pull`
has no `--unpublished` flag at all — it only takes `--development`, `--live`,
or `--theme=<id>` to pick a source theme. Don't write `theme pull
--unpublished`; it will error.

## `shopify auth login` / `shopify auth logout`
No flags beyond the universal ones (`--auth-alias`, etc.). No `whoami`
subcommand exists under `auth` or anywhere else in this CLI version — to
confirm an active session and which store it's pointed at, use
`shopify theme info` instead (below).

## `shopify theme info`
Shows the current store and, with `-t/--theme=<id>`, info about a specific
theme. This is the auth/store confirmation command — use it in place of a
`whoami` that doesn't exist.
- `-t, --theme=<value>` — theme ID or name.
- `-d, --development` — info for the development theme.
- `-j, --json` — machine-readable output.

## `shopify theme list`
- `--role=<live|unpublished|development>` — filter by role. Useful for a
  reconciliation check like "what's actually live right now" without
  scrolling a full list.
- `--id=<value>` / `--name=<value>` — filter to one theme.
- `-j, --json` — machine-readable output.
No store-write side effects; safe to run freely.

## `shopify theme pull`
Downloads remote theme files locally. Theme selection flags (pick exactly
one in non-interactive use): `-d/--development`, `-l/--live`,
`-t/--theme=<value>`. File-scoping flags: `-o/--only=<value>` (repeatable),
`-x/--ignore=<value>` (repeatable), `-n/--nodelete` (keep local files that
don't exist remotely — otherwise `pull` deletes them to match remote).
**No `--unpublished` flag.** No store-write side effects (read-only against
Shopify); it does overwrite local files on disk, though, wherever it's
pointed — never run it directly against the project working directory just
to inspect what's changed remotely.

**`--path=<value>`** targets the whole operation (including where files
land) at a given directory instead of the current working directory. This
is the mechanism `references/reconciliation.md` uses for pre-upload
reconciliation: pull into a throwaway temp directory
(`shopify theme pull --theme=<id> --path=<temp-dir>`) to get a clean,
isolated snapshot of what's on Shopify right now, then diff that against
the real working directory and the last synced baseline — rather than
pulling over the working directory itself.

## `shopify theme push`
Uploads local files to a remote theme, by default overwriting the remote
version. Theme selection (pick exactly one in non-interactive use):
`-d/--development`, `-l/--live`, `-t/--theme=<value>`, `-u/--unpublished`
(creates a new unpublished theme and pushes to it — when combined with
`-t/--theme` and *without* `-d`, `--theme` supplies the **name** for that new
theme, not an existing numeric ID to target). `-a/--allow-live` is
**required** to push to a theme whose role is live, in a non-interactive
environment — pushing to live without it will be refused by the CLI itself
(separately from the Claude Code tool-use classifier that can also block an
`--allow-live` push outright — see troubleshooting.md). `-p/--publish`
publishes the pushed theme as live immediately after uploading — never pass
this flag as part of routine Phase F work; publication is Phase I's job and
always a separate, explicitly confirmed action. `--strict` fails the push if
Theme Check finds any error-level issue. `-o/--only`, `-x/--ignore`,
`-n/--nodelete` scope which files are touched, same as `pull`.

## `shopify theme publish`
Sets an already-pushed remote theme as the live theme — this *is* the
"go live" action. `-t/--theme=<value>` (required in non-interactive use),
`-f/--force` skips the confirmation prompt (required in non-interactive
use). Never run this as part of routine phase work; it belongs to Phase I,
after explicit user confirmation in that moment, regardless of what
`STORE-BRIEF.md` authorizes in general.

## `shopify theme check`
Runs Theme Check (lint/schema validation). `--fail-level` controls what
severity fails the command (default `error`). Passing this command proves
the theme's syntax/schema is valid — it proves nothing about visual or
functional correctness (see troubleshooting.md).
