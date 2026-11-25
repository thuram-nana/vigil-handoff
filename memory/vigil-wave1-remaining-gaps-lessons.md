---
name: vigil-wave1-remaining-gaps-lessons
description: "Load-bearing lessons from the VIGIL remaining-gaps Wave 1 (OOB unit / fleet / M4.1 gateway / nuclei): the .venv-offense two-env masking trap, and why nuclei substring-matching cannot soundly mint a FACT."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-04T07:34:56.254Z
---

Two hard-won lessons from the remaining-gaps Wave 1 (building OOB-callback unit, fleet monitor, M4.1 AI-gateway, nuclei corpus against main 45f1f72b).

## 1. The `.venv-offense` two-env MASKING trap (cost real time — reuse this)
`/home/kali/vigil/.venv-offense` has the offense `framework` package IMPORTABLE (editable-installed at engine/crucible/framework). The CI **two-env P5 keyless leg** runs `integration/tests` with `PYTHONPATH=integration:gateway` and framework NOT on the path / NOT installed. So a LOCAL pytest of the P5 leg is UNFAITHFUL: it (a) PASSES tests that CI fails with `ModuleNotFoundError: No module named 'framework'` (a test/handler importing framework function-locally runs fine locally, fails keyless in CI), and (b) FAILS the SIGIL-sovereignty tests (test_finding_spool/test_floor_witness/test_destruction_gate/test_scitt/test_agent_react) that CI PASSES — because those assert `sys.modules` has NO framework, and locally framework IS importable so some prior test loads it. Confirmed: a local keyless repro of MAIN itself showed the same 6 sovereignty "failures" that CI main passes. FAITHFUL keyless repro = block framework with a meta_path finder:
```
import sys
class B:
    def find_spec(self,n,p=None,t=None):
        if n=='framework' or n.startswith('framework.'): raise ModuleNotFoundError("No module named 'framework'")
        return None
sys.meta_path.insert(0,B()); import pytest; raise SystemExit(pytest.main(['-q', <targets>]))
```
Or just trust CI's P5 job. REAL P5 failures this wave: OOB = proof-sink mint-gate over-relaxed (a non-re-drivable no-capture report reached the mint — fixed by requiring class AND re-drive-target, not a pure class check); fleet = 5 sign-revocation/fleet-kill tests call framework-importing handlers → split into an `importorskip("framework.v2")` offense-leg file (keep aggregator tests keyless). PATTERN for a new offense-arm/CLI-verb slice: any test that exercises a handler importing framework MUST be in an importorskip offense-leg file (or P5 reds); verify with the meta_path blocker, never the bare venv. [[crucible-engine-tests-honor-crucible-root]]

## 2. nuclei substring-matching CANNOT soundly mint a FACT (M1.4 DEFERRED after 3 red-pen BLOCKs)
The nuclei-template oracle re-fires `status==200 && contains(body,L1) && contains(body,L2)` (plain substring). Three independent red-pens each found a deeper hole: #1 weak literals + #2 a real COMPILER status-drop (a `word`+`status:and` matcher silently dropped the status matcher → fired at 403/500; FIXED in nuclei_compile.py so word+status compiles to a status-enforcing predicate, + NucleiTemplateCheck 0/404 guard — both sound, salvageable) + #3 the KILLER: even with status==200 + two "exclusive" literals, a benign 200 **docs/tutorial page quoting the exposed-page format** mints a CONCLUSIVE ACHIEVED_STATE FACT (nginx.org docs contain "server accepts handled requests"; every .git/config write-up contains "[core]"/"repositoryformatversion"; phpinfo how-tos contain "Configuration File (php.ini) Path"). Substring co-occurrence is NOT exclusivity — a page that QUOTES the resource is indistinguishable from the resource SERVED. Mitigant only: the check is path-bound (one fetch at the probe path), but soft-404/catch-all/CMS-slug/knowledge-base hosts defeat that. SOUND fix = STRUCTURAL parse-as-resource validation (body must parse AS INI with a [core] section for git-config, AS a JSON object with memstats for go-expvar, AS the actual status table) — cleanly achievable for the structured-format classes, fundamentally hard for the HTML-page classes (apache/nginx/phpinfo/tomcat/laravel). DECISION 2026-10-04: DEFER nuclei (branch prod/m1-nuclei-corpus local-only, NOT merged) — honest LEAD beats a false FACT; shipping a FACT-source that fires on documentation would be the corner-cut. Revisit as either LEAD-only breadth or structural-validation for the 2 structured classes. [[vigil-fact-coverage-program]] [[claim-discipline-meet-up]]

**Sovereign-venv import faithfulness (verified 2026-11-19):** `/home/kali/vigil/.venv-sovereign` has `sigil` (and vigil_core/vigil_integration) EDITABLE-installed from `/home/kali/vigil`, which is checked out on `meridian-s5-expansion`, NOT main. setuptools' `__editable___sigil_*_finder` is APPENDED to sys.meta_path, so PYTHONPATH wins only when it contains `$WT/apps/sigil` AND `$WT/packages/core/vigil_core` (+ `$WT/integration`) — then imports resolve to the worktree (checked from both the worktree root and apps/sigil). Omit them and tests silently import the stale meridian checkout. A test that strips PYTHONPATH for a subprocess has the same hole. Always use the full 3-path sovereign leg.
