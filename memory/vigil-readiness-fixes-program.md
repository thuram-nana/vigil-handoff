---
name: vigil-readiness-fixes-program
description: "VIGIL audit-remediation batch — verify-then-fix 4 real defects (correctness + reproducibility), red-penned + CI-green"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-08-16T20:18:56.373Z
---

Operator handed an external audit list (6 claims + "does one command install everything?"). Instruction:
**verify each claim against real code BEFORE touching anything.** Three parallel Explore agents verified; then
scope = **"correctness + reproducibility"** (gov-guard exception design = owner-signed exact-scope). Repo:
`/home/kali/vigil` (thuram-nana/vigil-sovereign). Per-slice cadence: build → red-pen → fix → CI-green → merge.

**Verification verdicts:** C1 cloud-unpacking REAL · C2 Home-crash REAL · C3 bootstrap PARTIAL (bypass-locks TRUE
& strong; stated *cause* "uv sync fails" WRONG — script used `uv pip install -r`) · C4 mypy REAL · C5 gov-guard
REAL (zero prod call sites; neither route wires it) · C6 ops PARTIAL (governance-replay already FIXED; cloud/K8s
verifiers already REACHABLE via `engage --fuse-only`/console `launch_cloud`; single-key/no-RBAC + backup +
no-HA hold).

**MERGED (all red-penned; each sharp slice caught a real defect):**
- **#358 slice 1** — `_oracle_signal` returns 3-tuple, 2 prod sites in `cloud_live_posture.py` unpacked 2 →
  ValueError. Fix: unpack 3 (`rules` discarded — future per-rule attribution, readiness-audit-2.7) + docstring.
  **CI silent-skip gap was BROADER than the 2 named files**: 7 framework-dependent integration suites
  (cloud_live_posture, cloud_benchmark, cloud_scope, iac_posture, k8s_posture, live_executor, mesh_cicd_posture)
  `importorskip`'d in the sovereign CI leg + omitted from the offense leg → never ran (183 tests). Wired all 7 +
  added `test_ci_framework_tests_run_in_offense_leg.py` (2 robust invariants: importorskip-framework files ∈
  offense list; sov-`--ignore`d files ∈ offense list — no brittle import parsing).
- **#359 slice 2** — `dashboard.snapshot()` returns `recent_decisions` as a Counter **dict**; `app.js` called
  `.slice()` → TypeError on every HEALTHY-backend Home load. Fix: emit `recent_events` **list** + `Array.isArray`
  guard + contract test. Red-pen BLOCK: test wasn't wired into CI (dead guard) → wired into `sigil-governor`.
- **#360 slice 3** — a `#   type:  int|...` PROSE comment in `settings.py` parsed as a PEP 484 type comment →
  mypy aborts (exit 2) while ruff stays green. Fix: reword to `# type field:`. New `sigil-lint` CI job: ruff
  BLOCKING + mypy **can-complete gate** (fails only on exit 2, NOT the ~60 pre-existing type errors = tracked debt).
- **#361 slice 4** — no `[tool.uv.sources]` (uv errors on `vigil-core` member) + `build_envs.sh` bypassed the
  committed `--hash`-pinned deploy locks. Fix: `[tool.uv.sources]` (all 6 members `workspace=true`); rewrite
  build_envs.sh → `--require-hashes -r <lock>` + members `--no-deps` (strix is the exception: its live-scan deps
  aren't in the framework lock → installed WITH deps, ZERO churn of locked pkgs) + **`pip check` completeness
  gate** + py3.13 pin; `make all` one-command (bootstrap→`vigil up`). VERIFIED end-to-end with real throwaway
  builds (both planes). Red-pen BLOCK: shallow smoke (`import sigil`/0-byte `framework/__init__`) did NOT catch an
  incomplete lock while 3 places claimed it did → built the capability UP with `pip check` (metadata-based).

**Key facts:** sovereign env FULLY hash-locked; offense framework-closure hash-locked, strix extras fresh. Locks
are pure third-party (pip-compile closures); the "missing" from a --no-deps coverage check is only first-party
`vigil-core`. `uv pip check` catches an omitted member dep (rc=1). Sandbox HAS network+cargo (use scratchpad, not
/tmp — `rm -rf /tmp/...` denied). CI legs: sovereign `pytest integration/tests` (framework absent → importorskip)
vs offense explicit hand-list (framework present). mypy exit 2 = can't-complete, 1 = type errors, 0 = clean.

**Documented follow-ons (NOT built — larger/needs operator):** C5 wire the government hard-guard into every
target-touching entry point + owner-signed exact-scope agency exception (safety-critical, gate-marshal+crypto-notary).
C6 real gaps: multi-user RBAC, offense-state backup/retention automation, HA/cluster profile, independent audit,
ANTIC/MINPOSTEL pilot. **The `sigil-governor` job hand-lists only 33 of 85 `apps/sigil/tests` → ~52 sovereign tests
never run in CI (incl. `test_governance_replay_guard.py` — the standout risk — + 12 `test_snapshot_fold_*`)** =
same silent-skip class as slice 1, larger; a dedicated slice. Also: a strix lock (full offense reproducibility);
burn down the ~60 mypy type errors; pin CI linter versions.

See [[claim-discipline-meet-up]] (build up, never soften — applied to slice-4 pip-check), [[vigil-crucible-core-gateway-importorskip]]
(the importorskip class), [[vigil-lww-governance-replay-gap]] (now fixed; guard dark in CI), [[vigil-authorship-contributors]] (no Co-Authored-By).
