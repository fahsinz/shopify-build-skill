# Reconciliation: don't trust the log, check reality

`BUILD-STATUS.md` is a record of what was true the last time someone checked.
It is not a guarantee of what's true now — a theme can be edited in the
browser editor, a product can be changed or deleted, or the draft theme can
get published, all without anyone updating the file or telling you. Two
separate moments require reconciliation, below. Both are mandatory, not
optional "if it seems necessary."

## On every resume (start of session, before doing any new work)

1. **Git vs. the log.** Run `git log -1` and `git status` in the project
   folder. Compare the last commit against what `BUILD-STATUS.md`'s task log
   claims is the latest completed work. If there's a mismatch (uncommitted
   changes present, or the log references work with no matching commit),
   stop and reconcile before proceeding — don't just note it and continue as
   if the log were still accurate.
2. **Remote theme vs. the log.** Run `shopify theme list --role=live` and
   `shopify theme list --role=unpublished` (see `cli-commands.md`) and
   compare against what `BUILD-STATUS.md`'s access-confirmation section
   says the target theme's role is. If the role changed, this is the
   [draft-theme-turns-out-to-be-live](troubleshooting.md#the-draft-theme-turns-out-to-be-live)
   situation — stop and get explicit confirmation before continuing, even
   though a past session already covered this once.
3. **Resources vs. the log.** For every row in `BUILD-STATUS.md`'s
   "Resources created/updated" table, spot-check that the resource still
   exists with the recorded handle/ID and still matches what was logged (a
   product's price, a discount's status) before relying on it as "already
   done." Don't re-verify every field of every resource every session — but
   don't skip this for resources the current session's work actually
   depends on.
4. **Browser/CLI access.** Re-run Phase B's access checks (CLI session,
   browser reaching admin) regardless of what a previous session recorded —
   access can be lost between sessions (extension disconnected, auth
   expired) and a stale "yes" in the log is actively misleading if access is
   actually broken right now. If the browser is unavailable this session,
   do not fall back to inferring storefront/theme-editor state from
   `WebFetch` or from memory of a previous session's screenshots — say so,
   and either stick to CLI/git-only tasks that don't need it, or stop and
   ask how to proceed.

Log the outcome of this reconciliation pass in `BUILD-STATUS.md` (a short
"reconciled on `<date>`: `<findings>`" note) before moving on to new work.

## Before every upload (any `theme push`, any theme-editor save)

A local edit you're about to push and a remote edit someone made in the
theme editor since your last sync are both real work. Blindly pushing can
silently discard the remote one; pulling directly over your working
directory just to "see what's different" can just as easily discard yours
(or at minimum, makes diffing confusing — you'd be diffing against a
working tree that's no longer in the state either side actually produced).
**Never pull into the working directory merely to inspect differences.**
Inspect in an isolated location instead, using three distinct states:

- **Baseline (B):** the git commit that matches what's actually live on
  Shopify as of the last confirmed-successful sync. Track its commit hash
  in `BUILD-STATUS.md` (a "Last synced baseline commit" field). This is
  *not* necessarily `HEAD` — it only moves forward when a sync is verified,
  per the last step below.
- **Local (L):** the project's actual working directory, exactly as it
  stands right now — committed history plus any uncommitted edits. Never
  touched by this procedure until you deliberately merge something into it.
- **Remote (R):** a fresh, read-only snapshot of what's on Shopify right
  now, pulled into a throwaway directory that is *not* the project folder:
  `shopify theme pull --theme=<id> --path=<temp-dir>` (`--path` points the
  whole operation, including where files land, at `<temp-dir>` instead of
  the current directory — see `cli-commands.md`). Use a fresh empty temp
  directory each time; delete it when done.

With all three available, reconcile deliberately rather than pushing or
pulling blind:

1. **Diff B against R** (`git show <baseline-hash>:<path> ` vs. the file in
   `<temp-dir>`, or check out B into another throwaway worktree and run
   `diff -r` against `<temp-dir>`) — this is what changed **remotely**
   since the last verified sync. If empty, remote hasn't moved; skip to
   step 3.
2. **Diff B against L** for the same files — this is what changed
   **locally** since the last verified sync (ordinary `git diff
   <baseline-hash>`). For any file where the remote diff (step 1) is
   non-empty:
   - If the local diff for that file is empty (you haven't touched it),
     adopt the remote version: copy that file from `<temp-dir>` into the
     working directory. This is a deliberate, read file-by-file action, not
     a bulk overwrite.
   - If both diffs are non-empty, that's a real conflict — read both diffs
     and merge the remote's actual change into your locally-edited version
     by hand. Never resolve this by force-overwriting one side without
     reading what's actually different, and never resolve it by blindly
     taking "whichever is newer."
   - Editor-managed files (`config/settings_data.json`, the `*-group.json`
     section-group files — see troubleshooting.md) are the ones most
     likely to show up here; treat a diff in one of these as routine, not
     alarming, but still read and merge it deliberately.
3. **Commit the merged result** in the real working directory before
   pushing — this commit is what you're about to make remote match.
4. **Re-confirm the explicit target** immediately before the push call
   itself: the store (`shopify theme info` or the `-s/--store` you're
   passing) and the exact theme ID (`-t/--theme=<id>`), read back out loud
   from command output, not recalled from earlier in the session or from
   `BUILD-STATUS.md`. This is deliberately redundant with Phase B/C's
   earlier checks — a long work session is exactly when "I already checked
   this" quietly stops being true.
5. **Push**, then verify the push actually landed as intended (re-run
   `shopify theme info`/`theme list`, or re-open the theme editor preview)
   before declaring success.
6. **Only now update the baseline** in `BUILD-STATUS.md` to the commit you
   just pushed, and only because step 5 actually verified it. A baseline
   updated before a verified push is worse than no baseline — it makes a
   future reconciliation pass trust a sync that never actually completed.

This isn't a one-time Phase C check — repeat it before every push for the
rest of the build, every session. It replaces any earlier instinct to
"just pull and see" directly in the project folder.
