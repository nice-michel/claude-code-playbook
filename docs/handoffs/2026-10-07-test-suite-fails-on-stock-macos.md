# Handoff — 2026-10-07: the install-preflight suite fails on a stock Mac

**Repo state:** branch `main`, in step with `origin/main` (fork
`nice-michel/claude-code-playbook`), VERSION 0.1.25. This handoff is the only
change since `1b49542`. No source was edited.

**Verified done:**
- Fork fast-forwarded to upstream 0.1.25 (2026-10-05).
- Instruction-budget plan written and parked:
  `docs/plans/2026-10-05-instruction-budget-plan.md` (start gate: 5-min load ÷
  10 cores < 0.6; it was 3.9–4.4 on every check).
- Suite results on this Mac (macOS, Darwin 25.6.0):
  `check_local_test` pass · `dead_words_vectors_test` pass · `rules_text_test`
  pass · `install_preflight_test` **FAILED 14 of 291** · `mutation_test`
  47 checks all ok, then stopped (too slow under load) ·
  `rules_text_mutation_test` not run alone. The full `tests/run.sh` reported
  2 failing suites; the second is **unconfirmed**.

**Root cause of the 14 (confirmed):** `tests/lib.sh` `make_tmpdir` uses
`mktemp -d`, which on macOS creates under `/var/folders/...` and ignores
`TMPDIR` (measured). `/var` is a symlink to `/private/var`; INSTALL.md's
destination guard refuses any linked ancestor by design (ADR 0010), so every
"allowed" case exits 2 with `Destination blocked: linked or non-directory
ancestor: /var`. With the temp root on a real path (`/private/tmp/...`) the same
suite gave `# passed 291`. Real installs are unaffected (`~/.claude` has no
linked ancestor). It is a test-portability defect, not a rules defect.

**In progress / awaiting the owner:** where the fix lands. Recommended:
upstream `michelabboud/claude-code-playbook`, then fast-forward this fork, so
the fork stays a mirror. Fix: `make_tmpdir` returns the physical path
(`cd "$d" && pwd -P`), with a regression check that fails first.

**Next, in order:**
1. Owner picks "upstream" or "fork" for the fix; make it, red then green.
2. Run `sh tests/run.sh` on an idle host; confirm the second failing suite
   is the same cause.
3. When the load gate holds, start the instruction-budget plan, Task 1.

**Gotchas:**
- `tests/run.sh | tail` hides the exit status; run suites separately or
  check `$?` without a pipe.
- The mutation suites re-run whole suites per mutation; under load (~40 on
  10 cores) they take hours.

**Left running:** nothing. All background test runs were stopped or finished;
no subagents. Scratch output (not evidence for the repo) is in this session's
scratchpad under `tests/`.

## Budget pause — 2026-10-07

Paused on the owner's word (relayed by mtls-49). Everything above is still exact: no
work was in progress, nothing half-done, nothing running. Open decision unchanged:
fix the temp-folder test defect upstream (recommended) or on the fork.
