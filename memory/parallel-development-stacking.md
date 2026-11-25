---
name: parallel-development-stacking
description: Owner directive 2026-11-19 — never idle on merges/approvals or the 5-hour limit; develop dependent slices in parallel on stacked branches, keep every merge gate
metadata:
  type: feedback
---

Owner, 2026-11-19: "there is no hour limit everything is okay proceed with parallel development" (after I said AEGIS
W1, legal-gov S2-S5 and invgraph S2/S4/S5 would wait for merges, approvals and the 5-hour reset).

**Why:** the owner wants maximal parallel throughput; serial waiting on merges, approvals and rate-limit windows is the
bottleneck they object to, not the review rigor.

**How to apply:**
- Start dependent slices NOW on stacked branches (worktree off the dependency's committed HEAD or a merge of several);
  merge the dependency's later fix commits in afterwards. Create all worktrees up front from the main loop (avoids
  concurrent `git worktree add` races), one per slice.
- Build on the plan's RECOMMENDED default for any open owner decision, as a single named constant, and surface it as
  "assumed — confirm at merge". Owner inputs (legal text, prices, tax, SARL identity) are still never fabricated:
  honest pending state + fail-closed enabling.
- Keep every gate: red-pen / multi-lens panel per slice (with fix loops), serial merges, and RECORDED owner approval
  for AEGIS merges ([[vigil-aegis-saas-program]]). "Everything is okay" is not a merge approval.
- The 5-hour subagent usage cap is REAL despite "no hour limit": at ~17:50 EST 2026-11-19 it killed ~45 in-flight agents
  across 10 workflows at once (reset 18:50). ~45 concurrent agents burn a window in under an hour. So: run merge-critical
  work (shepherds, conflict resolution, fix rounds, gates) FIRST, then the big build fleets; abort half-done merges of
  dead agents before re-running; never edit a cached call's prompt when resuming (positional cache — later calls re-run);
  new runs (0 cached) can be edited freely to describe the predecessor's partial state. State it plainly; don't argue.
Related: [[vigil-wave-cadence]], [[parallel-session-worktree-hazard]].
- **Checkpoint commits (lesson 2026-11-19):** the cap left ~17k lines of AEGIS work (AS-0 + 8 W1 slices) UNCOMMITTED in
  worktrees — safe on disk but invisible to the restarted agents, whose cached prompts still said "fresh off main". Every
  builder/fixer prompt must say "commit a WIP checkpoint after every coherent step (≥ every ~20 min)". On a restart:
  snapshot each dirty worktree (`git ls-files -m -o --exclude-standard -z | tar --null -czf <plans>/wip-snapshots-<date>/<wt>.tgz -T -`),
  and when no call of that run completed, launch an EDITED r2 script whose prompt says "review the predecessor's
  uncommitted work, keep what is sound, continue from it".
- **Durability (2026-11-22):** the machine rebooted twice (Nov 20 16:11, Nov 22 11:12) and each reboot WIPES /tmp, i.e. the
  session scratchpad (helper scripts, workflow .js files, probe dirs). Keep every tool + workflow script in
  `~/.claude/plans/vigil-tools/` (wf/*.js) and copy into the scratchpad only to launch (Workflow scriptPath must be
  readable there). Lost scratch files are recoverable from agent transcripts with `vigil-tools/recover_scratch.py`
  (replays Write/Edit/heredoc calls); restored set lives in `~/.claude/plans/scratch-recovered-2026-11-20/`.
  "Commit all work" = real WIP checkpoint commits in active slice worktrees + `vigil-tools/wip_snapshot.sh`
  (refs/wip/<branch>/<ts>, private-index snapshot, touches nothing) for stale/old worktrees. Credits ran out twice
  ("out of usage credits"): check with a 1-line test agent before launching fleets.
