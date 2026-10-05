# Instruction budget — get the otel-mtls sessions under Claude Code's 150k limit

- **Status:** parked 2026-10-05. The owner agreed to the approach and asked for this
  plan to run **when the laptop is not under load** (see "Start gate").
- **Written:** 2026-10-05 · **Last updated:** 2026-10-05
- **Dev mode:** this repository: unchanged. otel-mtls: `production` (its own
  `CLAUDE.md`, owner's ruling 2026-10-04). Nothing here touches code, so no code-review ladder applies; the
  mtls task closes out under mtls's own rules (version bump, checkpoint tag, push).
- **Execution:** tasks run back to back once started. Task 1 and Task 2 are
  independent; Task 3 depends on nothing; Task 4 is a separate plan proposal.

## The problem

Claude Code warns, in otel-mtls sessions (the main checkout and its worktrees):

> 13 instruction files add up to 224.3k chars, over the 150.0k-char total limit ·
> largest: CLAUDE.md (115.7k), ~/.claude/rules/AUTHORITY.md (19.9k), AGENTS.md (19.8k)

We have not established what Claude Code does beyond the limit (cuts files off, drops
some, or only warns). Either way the instructions that load are not the ones
written, so this is treated as a defect, not a cosmetic warning.

## Measurements (2026-10-05, byte counts via `wc -c`)

| File | Bytes | Loads where |
|---|---|---|
| `~/projects/mtls/CLAUDE.md` | 127,945 (116–128k across worktrees) | every mtls session |
| — of which `### ⚠️ Open issues` | 67,933 | 26 numbered entries, 9 headed `[FIXED …]`/`[CLOSED …]`, most carrying full history and addenda |
| — of which `### Module layout` | 43,894 | single tree lines up to several KB each (e.g. the `enrich.rs` line narrates ADR history) |
| — everything else | ~16,100 | resources index, the rustls decision, code flow, repo map, config, gotchas, pointers |
| `~/projects/mtls/AGENTS.md` | 22,242 | imported by `CLAUDE.md` line 9 (`@AGENTS.md`) on purpose; 17,903 of it is the testing/trap section shared with Codex |
| `~/projects/CLAUDE.md` | 19,861 | **every project under `~/projects`** — an old copy of the global rules from before the playbook |
| Playbook, always loaded (`~/.claude/CLAUDE.md` + 8 rule files + `platform/MACOS.md`) | ~69,700 | every session everywhere |
| — of which `rules/LOCAL.md` | 9,008 | ~8k is template how-to and examples; the only real entry is the git identity Fill |

Sum today in mtls ≈ 128k + 22k + 20k + 70k ≈ 240k. Target after this plan: **≤ 110k**,
which leaves headroom for mtls's `CLAUDE.md` to grow again.

## Threat sketch (rule 14.8)

- Who: nobody external; these are local instruction files.
- Through what: a lossy move could drop an invariant ("never restore the four
  throws", "never reintroduce connection sampling") that today keeps an agent from re-breaking a fixed bug.
- What they get: a regression shipped to the fleet by an agent that no longer sees the trap.
- Mitigation: the move is **verbatim, nothing deleted**; every "never" line stays in
  the loaded file as a one-liner pointing to the full entry; a diff check proves no bytes were lost (Task 1, step 5).
- The stale `~/projects/CLAUDE.md` is itself a risk: it outranks the global rules and
  contradicts them (port registry path, `ss -tlnp` on macOS, "only two moments need OK").

## Start gate — "the laptop is not humming"

Measured, not felt. Start only when all hold:

1. `sysctl -n vm.loadavg` 5-minute figure ÷ `sysctl -n hw.ncpu` **< 0.6**
   (at writing: 39.07 ÷ 10 = **3.9** — heavy).
2. No other agent session is working in otel-mtls `main` (check `pgrep -lf claude`
   and the mtls `HANDOFF.md`/`PLAN.md` for a running lane); Task 1 edits a file
   ~30 worktrees branch from, so it must not race a lane that is editing the open-issues list.
3. `git -C ~/projects/mtls status` shows no edits to `CLAUDE.md`.

## Task 1 — split `mtls/CLAUDE.md` (saves ~100k; alone it clears the warning)

Owner: one session in `~/projects/mtls` (Standard tier is enough: mechanical moves; the
judgement step — choosing what stays loaded — is done by the coordinator).
File boundary: `CLAUDE.md`, `ARCHITECTURE.md`, two new docs, `CHANGELOG.md`, `PROGRESS.md`, `VERSION`.

1. Move `### ⚠️ Open issues` **verbatim** to
   `docs/reports/2026-10-05-open-issues-ledger.md` (dated per the repo's `docs/` convention).
2. In its place keep an index: one line per entry, still-open first —
   number, status, one-sentence title, pointer to the ledger anchor. Every
   `**Never** …` / `**NEVER** …` sentence is copied into the index line of its entry, word for word.
3. Move `### Module layout` **verbatim** to `docs/guides/module-reference.md`; keep in
   `CLAUDE.md` a bare tree (file name + ≤ 1 line of purpose each) and the pointer.
   Check `ARCHITECTURE.md` (16.5k) first so the guide does not duplicate it.
4. Add a line to `CLAUDE.md`: entries are added to the ledger, and only a one-liner goes in the index — so it does not grow back.
5. Evidence: `wc -c CLAUDE.md` ≤ 20,000; a script proves every line removed from
   `CLAUDE.md` appears in one of the two new files (no lost bytes); `grep -c 'NEVER\|Never'`
   on the index ≥ the count in the old section; a fresh mtls session shows no budget warning (quote its line).
6. Close out under mtls's rules (bump, commit, `checkpoint/` tag, push). Worktrees pick
   it up on rebase; a branch that edited an open-issue entry re-targets the ledger file
   in its rebase — note this in mtls `HANDOFF.md`.

## Task 2 — quarantine `~/projects/CLAUDE.md` (saves 20k, removes a contradiction)

Owner: this session or any session; outside a repository, so no version bump.

1. Read `~/.claude/rules/QUARANTINE.md` by path first.
2. Confirm no project relies on it: `grep -l` across `~/projects/*/CLAUDE.md` for
   rules that exist only there (the `~/.config/fleet/ports/` registry is the one to check).
3. Move it to `~/.quarantine/` with its manifest (same volume — atomic `mv`).
4. Evidence: a fresh session in another `~/projects` repo no longer lists it.
   Final deletion from quarantine is a separate owner decision.

## Task 3 — trim `~/.claude/rules/LOCAL.md` (saves ~8k)

1. Back up to `~/.claude/rules-backups/LOCAL.md.2026-10-05`; verify with `shasum -a 256`.
2. Keep the header paragraph, the section headings, and the git identity Fill; drop
   the how-to text and the worked examples (they live in the repo's `templates/`).
3. Evidence: `scripts/check-local.sh` against the installed rules exits 0; byte count before/after.

## Task 4 — playbook: a size budget (proposal; needs its own go)

The playbook alone costs ~70k, nearly half the limit, before any project's file.
Proposed for a separate plan, recorded in `BACKLOG.md`:

- a test that fails when the always-loaded set exceeds a budget (≈ 40k proposed);
- ship `LOCAL.md` as entries-only, move the how-to into `INSTALL.md`;
- re-measure `AUTHORITY.md`'s summary tables against what the detailed files already carry (ADR 0011 did this
  once at 75.7k; the 0.1.23 dev-modes section pushed it back up).

## Expected result

| | Before | After |
|---|---|---|
| mtls `CLAUDE.md` | ~128k | ≤ 20k |
| `~/projects/CLAUDE.md` | 19.9k | 0 (quarantined) |
| `LOCAL.md` | 9.0k | ~1k |
| Total in an mtls session | ~240k | **≈ 105k** (AGENTS.md 22k + playbook ~62k + mtls ≤ 20k) |
