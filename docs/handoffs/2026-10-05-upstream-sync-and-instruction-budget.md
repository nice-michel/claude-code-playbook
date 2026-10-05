# Handoff — 2026-10-05: upstream sync, instruction-budget plan parked

**Repo state:** `main` = `origin/main`, clean, VERSION 0.1.25. Fork
`nice-michel/claude-code-playbook` fast-forwarded to upstream `ea9e934`;
tags `checkpoint/0.1.23`–`0.1.25` pushed to the fork. Installed `~/.claude/`
rulebook already reports 0.1.25.

**Done and verified:** sync (fast-forward only, no local divergence). Plan
written and committed: `docs/plans/2026-10-05-instruction-budget-plan.md`
(`8b1b398`); `PLAN.md` row (parked) and a `BACKLOG.md` line for the playbook's
own size budget.

**Not done:** the test suite was not run after the sync
(`sh tests/run.sh`). No task of the budget plan has started.

**Next, in order:**
1. Check the plan's start gate (5-min load ÷ `hw.ncpu` < 0.6; no lane editing
   mtls `CLAUDE.md`). At writing the load factor was 3.9.
2. Task 1 (split mtls `CLAUDE.md`) runs in `~/projects/mtls` under its rules.
3. Task 2 (quarantine `~/projects/CLAUDE.md`) and Task 3 (trim `LOCAL.md`).
4. Task 4 is a proposal; needs the owner's go.

**Gotcha:** an earlier reply suggested deduplicating mtls `AGENTS.md`; it is
imported by `CLAUDE.md` on purpose, so the plan leaves it alone.

**Left running:** nothing from this session.
