---
name: crucible-engine-tests-honor-crucible-root
description: "Running the CRUCIBLE/vigil engine test suite honors CRUCIBLE_ROOT (and CRUCIBLE_AGENT/TEST_PREFIX/OPSEC_POSTURE) from the shell env — if set, the suite writes test runtime state into THAT checkout, not the worktree. Scrub CRUCIBLE_* (or use a hermetic env) for every engine-test run, especially a delegated agent's."
metadata:
  node_type: memory
  type: reference
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-02T18:37:37.395Z
---

This shell exports `CRUCIBLE_ROOT=/home/kali/Music/PENTEST/crucible` (+ `CRUCIBLE_AGENT`,
`CRUCIBLE_TEST_PREFIX`, `CRUCIBLE_OPSEC_POSTURE`). The engine resolves its runtime root via
`crucible_root()`, which READS those env vars. So `pytest` on `engine/crucible/framework/v2/**`
run in a bare shell writes its test state (`.blackboard/store.sqlite`, `.memory/`, `.authority/`,
`.authority-root/`, `.entitlement/`, `.evidence-keys/`, `.console/runs/`, `.dryrun/`,
`.vigil-live/`, and untracked `targets/<slug>/*` artifacts) into the CRUCIBLE_ROOT checkout — NOT
the worktree under test. It also PROVISIONS a test trust root there, which then causes ~57 spurious
`EntitlementMissing`/gateway failures on the next run in a tree that already has one.

Observed 2026-10-02: a delegated agent's first (un-scrubbed) run polluted
`/home/kali/Music/PENTEST/crucible` this way (untracked runtime state only — no git-tracked source
harmed; the acme target artifacts it overwrote are untracked demo output). Reading file CONTENTS
there is blocked by the credential-exploration classifier (keys live under `.evidence-keys/`).

**RULE for every engine-test run (and in EVERY delegated agent prompt that runs engine pytest):**
run hermetically — either `env -i HOME=<scratch> TMPDIR=<scratch> PATH=... PYTHONSAFEPATH=1
PYTHONPATH=engine/crucible:packages/core/vigil_core:integration:gateway <venv>/python3 -m pytest …`
(CI style), or at minimum `env -u CRUCIBLE_ROOT -u CRUCIBLE_AGENT -u CRUCIBLE_TEST_PREFIX
-u CRUCIBLE_OPSEC_POSTURE TMPDIR=/tmp …`. Then `crucible_root()` falls back to the worktree and no
foreign checkout is touched. The vigil build worktrees (/home/kali/vigil-wt-*) are unaffected
regardless — the pollution only hits whatever CRUCIBLE_ROOT names. [[crucible-testing-and-gotchas]]
