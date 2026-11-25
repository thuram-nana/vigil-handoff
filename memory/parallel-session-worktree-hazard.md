---
name: parallel-session-worktree-hazard
description: "Two Claude processes of the SAME session id can run at once on this machine and edit the same worktrees; check ListAgents/ps before editing, split ownership, and never `git stash` in a worktree (the stash list is shared)"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-16T21:51:30.166Z
---

On 2026-11-16 a second process of the same session (same scratchpad path) worked the same local worktrees as the
first: one merged #1136 while the other committed W2c-1; uncommitted edits appeared mid-turn. Coordination via
SendMessage worked (split: one owned W2c-1/NYX, the other HANDOFF/M1c-2). Also: `git stash` on a clean worktree
saves nothing and the following `git stash pop` applied the main checkout's OLD stash
(`salvage-main-dirty-327behind-2026-09-30`, still kept) into the W2c-1 worktree.

**Why:** shared worktrees + a shared scratchpad (slice symlink, venv copies) make concurrent sessions clobber each
other silently.

**How to apply:** at session start run `ListAgents` and `ps -ef | grep claude`; give each session its own worktree and
its own scratch subdir (e.g. `<scratchpad>/e0/`); never re-point another session's `slice` symlink; compare against the
base with `git show origin/main:<path>` or a second worktree, never `git stash`. Related: [[crucible-testing-and-gotchas]].

**Disk / cleanup (2026-11-19):** 117 worktrees filled / to 98%. Safe cleanup = `git worktree remove` WITHOUT --force on
worktrees whose branch is an ancestor of origin/main AND that are clean (freed ~12 GB, 75 removed). Trap: a slice
branch freshly cut from main with a builder still reading (no commit yet) ALSO looks "merged + clean" — exclude every
worktree a running agent owns (it was vigil-wt-aegis-as0) before removing.
