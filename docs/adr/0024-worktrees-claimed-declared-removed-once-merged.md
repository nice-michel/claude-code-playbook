# 0024 — Every worktree is claimed, declared, and removed once its work is merged

- **Status:** accepted, 2026-10-26 (the owner: "please update and fix the hygiene rules in the playbook", after "keeping 100 worktrees, that's a bad hygiene maintenance that I fight against non stop"; then, on the first draft: "what if that lane dies and I have left-overs", "when task is done, commit and push and check if possible and declare if you are ready for merge to the orchestrator", "we should not remove a worktree that is paused ... or uncommitted or not ready", "merged branch should continue with worktree removal after validations ... and that it's really all merged").
- **Scope:** new rule 13.7; the header and rules 13.3, 13.4 and 13.5 (`rules/HYGIENE.md`); the section 13 trigger and summary (`rules/AUTHORITY.md`); the cleanup trigger in rule 10.2 (`rules/DESTRUCTIVE.md`); the hygiene checkpoint in rule 6.1 (`rules/WORKFLOW.md`); the section table in `CLAUDE.md`.

## Context

On 2026-10-25 the owner's largest project held 100 git worktrees. Every one of
them passed rule 13.3's three checks, or failed them only for build output, and
almost every tip was already on the remote. Nothing was at risk, yet most of
them were stale: 22 finished lanes whose branches were already merged into
their integration branch, 33 detached review snapshots in `/tmp`, and a dozen
lanes paused weeks earlier, each keeping a worktree and a build cache. The
owner had been asking, session after session, "list all worktrees, tell me
which are stale, why do we keep them". The free space on the disk was below the
floor, and two passes freed 19 GB.

Section 13 already said "the lane that creates a worktree owns it ... and
unlocks and removes it at close-out", in the middle of rule 13.3's removal
procedure. It failed for four reasons:

1. **The duty was read too late.** Section 13 loads only on a cleanup trigger,
   and its `paths:` scope matches `.worktrees/` inside the project. A lane that
   *creates* a worktree, usually as a sibling folder or a snapshot in `/tmp`,
   never matches the scope and has no trigger that sends it to the file.
2. **Nothing made the hand-back carry it.** A lane ends its turn by returning
   its close-out; nothing asked that close-out to account for the worktrees it
   made, so a coordinator had nothing to check.
3. **State was invisible.** A paused lane's worktree said nothing about who
   paused it or why, a finished lane never declared whether its work was ready
   to merge, and a lane whose branch was merged into an integration branch
   (not the default branch) was never called finished by anyone. A lane that
   died left its worktree with no one to account for it.
4. **The report had no inventory.** Rule 13.5's report listed removals and
   candidates, but not the worktrees themselves, so the owner's recurring
   question had no standard answer.

## Decision

New rule 13.7, **every worktree is claimed, declared, and removed once its work
is merged**:

1. The creator claims it at creation with `git worktree lock --reason`, naming
   the owner, task, date and state, and re-locks it when the state changes.
   The lock reason is the worktree's marker.
2. When the task is done, the lane commits, pushes, runs its checks and
   declares to the coordinator *ready to merge* (branch, head, evidence) or
   *not ready* and why. It does not remove the worktree: the work is not merged
   yet, and a fix round may need it.
3. Whoever merges it removes it, after validating that all of it is merged:
   its HEAD is an ancestor of the target line, the status shows only build
   output, no stash came from it, nothing runs in it, and rule 13.3's checks
   pass. A merged worktree is never left behind.
4. A paused, not-ready or uncommitted worktree stays, re-locked with its state
   and reason, and the handoff carries the `git worktree add` line.
5. A worktree whose lane died is adopted by the coordinator, never swept.
   Uncommitted or unpushed work stays and is listed for the owner; nobody
   commits or discards a dead lane's work on its behalf. Clean and merged goes
   by step 3; clean and unmerged is listed as unfinished work.
6. Every hygiene report lists all worktrees with their lock reasons: running,
   ready to merge, paused or not ready, merged, unclaimed or orphaned.

Section 13's trigger now also names creating a worktree and pausing a lane, in
the file's header, in section 0's summary, in rule 10.2 and in the section
table. The header says the `paths:` scope cannot fire for a worktree outside
the project folder.

## Alternatives rejected

- **A `.hygiene.json` marker inside the worktree** (rule 13.4's existing
  marker). An untracked file in a worktree makes `git worktree remove` refuse,
  so the marker would block the removal it exists to enable. It was observed:
  five such markers sat untracked in paused lanes on 2026-10-25. The lock
  reason is git's own place for this, travels with `git worktree list
  --porcelain`, and needs no file.
- **A periodic sweep by a coordinator or a scheduled job.** It moves the cost to
  someone who did not create the worktree and cannot prove it is theirs (rule
  13.2), which is the failure we are fixing. The inventory in 13.5 still
  surfaces what a creator left behind.
- **A cap on the number of worktrees.** The owner wants parallel lanes; the
  defect is the leftover, not the count. A cap would throttle real work and
  still leave stale ones under the cap.
- **Removing a worktree at the lane's close-out, before the merge** (the first
  draft of this rule). The work is not merged yet, a fix round may need the
  same worktree, and the lane cannot prove its work landed. The owner moved
  removal to the merge.
- **Removing a paused lane's worktree** (also the first draft). A pause is the
  owner's decision to resume later, often for reasons outside the work, such
  as a budget; the worktree may hold state the lane has not committed, and
  removing it turns a pause into a teardown. A paused worktree stays,
  labelled with why.
- **Sweeping a dead lane's leftovers by age.** Age says nothing about whether
  the work is safe. Adoption and classification do: merged and clean goes,
  anything else comes to the owner.

## Consequences

- A lane's close-out gains a declaration and one line; a merge gains a
  validation and a removal; a pause gains a re-lock and a handoff line.
- A worktree with no lock is unclaimed by definition, so worktrees made before
  this rule show up as "unclaimed" in the first inventory and come to the owner
  once, with what removing each would lose.
- Another lane's lock still stops removal (rule 13.3). A stale lock is listed,
  never broken.
- Any local-layer Override bound to section 13 goes stale, because the section's
  digest changed; it is re-anchored at install, as designed.
