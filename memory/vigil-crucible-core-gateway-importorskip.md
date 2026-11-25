---
name: vigil-crucible-core-gateway-importorskip
description: A console test importing vigil_integration.live.* that needs vigil_gateway fails in the CRUCIBLE-core CI job (no gateway) — importorskip it
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-25T15:13:21.699Z
---

In VIGIL (repo thuram-nana/vigil-sovereign), a test under `engine/crucible/framework/v2/console/tests/`
that imports (even lazily-inside-a-helper) `vigil_integration.live.executor` / `.wiring` / `codefix_runner`
/ `sandbox_exec` will `ModuleNotFoundError: No module named 'vigil_gateway'` in the **`CRUCIBLE core on
vigil_core`** CI job — that job installs framework + vigil_core but NOT `vigil_gateway`. The same test
passes locally (offense venv has gateway) and in the **`integration two-env boundary (P5)`** job (installs
integration + gateway). Hit while landing D1 (`test_chat_clone.py` mocked `subprocess_runner` via
`from vigil_integration.live import executor`).

**Why:** the CRUCIBLE-core job also runs `framework/v2/console/tests`, but the offense `vigil_integration.live`
modules pull `vigil_gateway` (SSRF denylist etc.), which isn't on that job's path.

**How to apply:** put `pytest.importorskip("vigil_integration.live.executor")` (or the specific offense
module the test needs) at module top — the file RUNS in P5 (deps present) and SKIPS cleanly in
CRUCIBLE-core; coverage is preserved because P5 runs it. The production code should already lazy-import
these guarded (returns "toolchain unavailable"), so only the TEST's direct import needs the guard. Apply
this proactively to the D2/D3 chat-orchestrator tests (they mock codefix_runner/sandbox_exec). Related:
[[vigil-live-external-fact]] (CRUCIBLE_ROOT / env gotchas), the P5 job also bundles the egress-guard
seccomp flake (see repo `knowledge/kb/two-env-boundary.md` §5).

**INVERSE direction — the P5 BOUNDARY leg has framework ABSENT (gold-standard Lanes S/O/P, 2026-10).**
Two recurring traps there, BOTH masked locally because `.venv-offense` has framework editable-installed
(so a local run NEVER hits the ImportError — the venv-masking trap; only the clean-venv P5 leg is faithful):
(1) TEST: a NEW `integration/tests/*` test that imports `framework` (even in a fixture/helper) ERRORs in the
P5 boundary leg (`PYTHONPATH=integration:gateway`, no engine/crucible). Fix = module-level
`pytest.importorskip("framework.v2")` (skips boundary leg) AND add the file to the ci.yml OFFENSE leg list
(`PYTHONPATH=integration:engine/crucible:gateway`) so it still RUNS+proves. Classifier blocks a `--ignore`
edit as CI-bypass; the importorskip + additive offense-leg add is the accepted, equivalent route. Bit Lane S
(test_lane_s_mint_choke) and Lane O (test_conjunctive_gate_blast_veto).
(2) PRODUCTION CODE: a new gate/seam that LAZILY imports `framework` and FAILS-CLOSED on ImportError will
DENY on the integration-only / boundary plane where framework is legitimately absent — breaking existing
boundary-leg tests. A CRUCIBLE-side defense-in-depth control must DISARM (behave byte-identically to
pre-change) when its half is simply absent, reserving fail-closed for present-but-corrupt. Bit Lane O's
blast-veto (`conjunctive_gate._effective_blast_ceiling` imported `framework.v2.worldmodel.impact`
unconditionally → fixed with `try/except ImportError: return None`). Verify such a fix FAITHFULLY with a
`sys.meta_path` finder that raises ImportError for the framework module (NOT a plain local run). Also: any
change to `predicate_oracle` FIRING must be mirrored in BOTH zero-VIGIL kernels — see
[[achieved-state-sampling-unsound]].

**SHARPENING (G2 #919, cost TWO CI cycles 2026-10-08 — a known lesson under-applied):**
(a) The offense-leg requirement is ENFORCED by a meta-guard, `integration/tests/test_ci_framework_tests_run_in_offense_leg.py` — it greps every `integration/tests/test_*.py` for `importorskip("framework…")` (and every sovereign `--ignore`d file) and FAILS if the file is not in the ci.yml offense-leg positional run-list (~line 518+). So "add to the offense leg" is not optional hygiene; omitting it REDS P5 at ~15min. RUN this guard locally after adding any such test.
(b) `importorskip` the dep that is ACTUALLY ABSENT in the pass you must skip. The P5 SOVEREIGN pass has `vigil_gateway` PRESENT but `framework` ABSENT — so a test needing both must `importorskip("framework")` FIRST (a bare `importorskip("vigil_gateway")` does NOT fire there, and the hard `from framework…` then ERRORs collection at ~43s). A test needing both deps importorskips BOTH, framework first.
(c) FAITHFUL local check = a `sys.meta_path` finder raising `ModuleNotFoundError` for `framework*` (gateway left importable) → collect-only must report SKIPPED, not ERROR; `.venv-offense` masks it. [[vigil-engine-egress-floor-residual]]

**(d) LAZY framework imports STILL need the module-level importorskip (repo-connect Slice 1 PR #954, 2026-10-25).** `test_jobs_securityjob_slice1.py` has NO hard `from framework import` — every framework touch is inside a test body or a lazy in-method import (the jobs spine imports EngagementScheduler/SlugLease/authority lazily). So it COLLECTS clean on the sovereign leg (no collection ERROR) but its framework-dependent tests ERROR at RUN there. Fix is identical: module-level `pytest.importorskip("framework.v2.engagement_scheduler")` (skips the whole file on the sovereign leg) + add to the offense-leg run-list. Don't be lulled by "no top-level framework import" — if ANY test exercises framework (incl. via a lazy prod import like `queue.enqueue`→EngagementScheduler), the file needs the guard. The guard test flags it only once it HAS an `importorskip("framework…")`; the pre-guard failure is the sovereign leg erroring, not the meta-guard.
