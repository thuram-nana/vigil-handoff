---
name: machine-work-stays-local
description: Owner 2026-11-24 — all half-finished work STAYS on the Kali machine and is organised here (not pushed out); master completion plan audit + local WIP snapshots
metadata:
  type: feedback
---

Owner, 2026-11-24: asked to "push all machine work to my local machine"; clarified = **keep it here, just organise it** —
nothing leaves this machine; commit loose work into its branches, line everything up with the master plan, continue
development here.

**Why:** development is faster on this machine; the owner wants every half-finished work item included and completed.

**How to apply:**
- Do NOT push unfinished branches to GitHub just to back them up (repos are PUBLIC); push only as part of the normal PR flow.
- Local safety net: `~/.claude/plans/vigil-tools/wip_snapshot.sh` (refs/wip/<branch>/<ts>, private index, touches nothing).
  2026-11-24 18:50 snapshots taken for every dirty VIGIL worktree + /home/kali/vigil, /home/kali/sigil, PENTEST-main,
  vigil-apex, COMPANY.
- The machine-wide audit (wf_8c9cd1a6-e56) writes the PRIVATE master plan to ~/.claude/plans/MASTER-COMPLETION-PLAN.md +
  master-completion-items.json; every half-done item goes into it and gets completed in its waves.
Related: [[workflow-resume-state-2026-11-22]], [[parallel-session-worktree-hazard]].

**2026-11-24 23:00 — snapshot-script secret leak (local only), FIXED forward:** wip_snapshot.sh had been capturing ~/sx-s2's untracked
offense-governance.key / offense-spine.key / operator.key (+ planreq.spine, usage-ledger.jsonl) into 6 LOCAL refs
refs/wip/strix/s2-transport-intake/{20261122-1123,20261124-1850,-2134,-2208,-2248,-2254}. Verified 0 refs/wip on GitHub (ls-remote).
Script now skips ~/sx-s2 and drops UNTRACKED *.key/*.pem/*.p12/*.pfx/*.jks/*.kdbx/*.secret/.env*/id_*/*.spine/usage-ledger.jsonl from the
private index (backup at wip_snapshot.sh.bak-20261124; self-tested: code files like token_broker.py are kept). Deleting those 6 refs
needs owner OK (originals still on disk in ~/sx-s2, so nothing is lost); after deletion, objects stay until a quiet-time `git gc`.
NEVER push refs/wip/* (no --mirror, no refs/* refspec).
- 2026-11-24 23:10 (owner: "clear the backups that are no longer needed"): deleted 97 of 121 refs/wip — 6 sx-s2 key snapshots, 25
  mutation/review-rig snapshots (detached-*), 66 whose every changed path is byte-identical in the branch tip / origin/main / a newer
  snapshot. Recovery list (ref->sha) ~/.claude/plans/vigil-tools/deleted-wip-refs-20261124.tsv (valid until gc). KEPT 24 (latest per
  worktree with unsaved work + 4 older ones holding content found nowhere else). KEPT (not provably merged): ~/vigil-builder-backups
  (uncommitted parts differ from main; prH/prE unmerged), ~/.claude/plans/wip-snapshots-2026-11-19 (AEGIS pre-build, until AEGIS merges),
  ~/vigil-backups (Oct config backups), all salvage/* branches. TODO when NO workflow is running: `git -C /home/kali/vigil gc --prune=now`
  to physically drop the deleted refs' objects (incl. the sx-s2 key blobs) — never while agents commit.
