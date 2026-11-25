---
name: vigil-ceremony-verify-workflow
description: "The reusable 6-lens adversarial verify workflow for VIGIL sensitive ceremony slices, and the parallel-mutation race that MUST be worktree-isolated"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-08-29T12:11:17.335Z
---

For VIGIL sensitive owner-key ceremony slices (Waves 9-11 of [[vigil-full-ui-control-program]]) I run a
**6-lens adversarial verify** as a Workflow before each merge — stronger than a single `red-pen` agent.
Script: `scratchpad/ceremony_verify.js` (per session). Lenses (each an independent `red-pen` skeptic told to
REFUTE one property): **key-leak · rbac-bypass · argv-injection · fail-closed · crypto-reuse · test-substance**.
Args: `{slice, branch, files[], invariants}`. Returns `{verdict: PASS|PASS_WITH_NITS|BLOCK, blocks[], lows[], all[]}`.

**LOAD-BEARING GOTCHA (cost a false-positive BLOCK on Wave 10b):** the lenses run in `parallel()`, and several
lenses MUTATION-TEST (edit a guard in the source → confirm the test goes red → restore). If they share ONE
working tree, agent A can READ a file at the instant agent B has a mutation written — and misread it as an
"external writer planting a backdoor" (I saw a lens report a phantom `--leak-secret` flag + a fail-open `_run`
that existed in NO file/git-object/reflog — they were a concurrent lens's transient, since-restored edit).
**FIX: give each lens `isolation: 'worktree'`** so it gets its own exclusive repo copy — no cross-agent file
race. This REQUIRES the slice COMMITTED first (a worktree is created from HEAD, so uncommitted changes are
invisible to it): commit the slice → (rebase onto fresh main) → run the verify on the committed branch. Also
tell the lenses in-prompt: "you are in an isolated worktree; mutation-testing is safe; if a file looks
modified it is your own edit; NEVER attribute file state to an external writer," and give the worktree-relative
test command (`PYTHONPATH=$PWD/apps/sigil:$PWD/packages/core/vigil_core:$PWD/integration SIGIL_HOME=$(mktemp -d)
/home/kali/vigil/.venv-sovereign/bin/python -m pytest …` — the venv is shared, outside the worktree).

**Reading a BLOCK:** don't merge a SENSITIVE slice on ANY block, but DO check whether it is real — a genuine
BLOCK cites a defect in the AUTHORED bytes (grep confirms it in the committed file); a false positive cites
state that is absent from the file/git NOW and is contradicted by the same or another lens's own clean-state
proof. When the other lenses independently clear the code (key-leak NONE, rbac NONE) and the block is a
tree-race artifact, fix the workflow (isolation) + any REAL lows the run surfaced, then RE-RUN for a clean
record. Real lows the 10b run correctly caught: a missing CSRF/origin NEGATIVE CONTROL (the `_post` test helper
always sent a valid Origin, so neutering `_origin_host_ok` stayed green) → add a cross-origin-POST-with-valid-
owner-token→403 test.
