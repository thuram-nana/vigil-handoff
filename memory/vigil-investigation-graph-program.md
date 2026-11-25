---
name: vigil-investigation-graph-program
description: "VIGIL persistent investigation + coverage graph program (owner task 2026-11-19) — audit-first, extend existing stores, one slice per PR; gap matrix PRIVATE (public repo); ARTEX zip forbidden"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T12:03:10.871Z
---

**Task (owner, pasted 2026-11-19 ~05:10):** audit then implement an evidence-linked investigation graph — authorized assets -> goals -> planned actions -> results -> leads -> independent verification -> findings -> remediation -> retests — plus explicit coverage tracking (assets x check families; 8 states: not-assessed/queued/running/completed/blocked/failed/inconclusive/n-a; coverage SEPARATE from verdict; versioned denominator). Reliable execution: atomic claims/leases/idempotency, safe resume, auth revalidated before + after resume, revocation stops work, uncertain side-effect outcomes never blindly retried. 10 acceptance tests (claims, restart, out-of-scope asset, revocation, cross-tenant, unsupported claims stay LEAD/INCONCLUSIVE, coverage not inflated, lineage, uncertain outcomes, UI/export = persisted state). NO offensive expansion; preserve every safety gate; FACT only via the existing independent verification path.

**Binding constraints:**
- EXTEND existing models/services (spine, blackboard, world model, registries, ScanReport.coverage, retest_reproof...) — no parallel subsystem, no new graph DB unless measured need.
- HANDOFF-SESSION.md §0.2: one slice per PR off fresh main, default-OFF + byte-identical when off, mutation-verify each guard, adversarial red-pen, merge only on green.
- §0.3.8 PUBLIC REPO: the gap matrix (unfixed weaknesses) stays PRIVATE — chat + `~/.claude/plans/`, never committed; repo docs describe only fixed state + honest limitations.
- **Never open `/home/kali/Downloads/ARTEX-main.zip`** — owner forbids copying ARTEX code/prompts/assets. The repo has NO ARTEX code (grep hits for "artex" are all "cleartext").

**State:** Phase 1 read-only audit = workflow `wf_e9531e8a-5b3` (7 area auditors -> synthesizer: gap matrix + slice plan), launched in parallel with the A-J workflow. Phase 2 = implement slices via build -> mutation-verify -> red-pen, sequenced AFTER overlapping in-flight A-J branches merge (scheduler [F-fixfwd], retest [E], reports [G], customer app.js [B/G/I]).

**How to apply:** resume from the audit journal; show the gap matrix to the owner; save it to `~/.claude/plans/investigation-graph-audit.md`. Related: [[vigil-consumer-al-program]], [[vigil-consumer-cascade-lessons]], [[clean-verdict-soundness]].

**Status 2026-11-19:** audit DONE (wf_440cc8ea → ~/.claude/plans/investigation-graph-gap-matrix.json, supersedes wf_e9531e8a). 5 slices, NO new graph DB (extend F4 graph + evidence-cert layer): S1 evidence-by-ID resolver (VIGIL_EVIDENCE_RESOLVER), S3 charter Goal node (VIGIL_GRAPH_OBJECTIVES) — building in wf_f15c11fa; S2 join-verified CONFIRMED + S4 remediation/retest nodes after S1; S5 coverage view last.

**3-lens panel 2026-11-19: S1 (835e4796) + S3 (b99a3c7d) BOTH BLOCKED.** S1: default governance trust root sits INSIDE
the tenant tree (CRUCIBLE_ENTITLEMENT_DIR unset -> self-auth via planted anchor), TOCTOU symlink swap -> cross-tenant
resolve, no aggregate memory bound, RecursionError blinds index, vacuous import guard. S3: unsigned refute retires a
CONFIRMED REACHES edge / erases objectives; objective_id secret filter admits real creds + second id channel via
_project_reaches; explicit enable_objectives='false' turns it ON (bool(str)); 39/90 mutants survive.

**2026-11-19 parallel build:** S2 (join) + S4 (lifecycle) stacked on S1 @ 835e4796, then S5 on S2+S4+S3
(wf_b985beff-6c2; vigil-wt-invgraph-s2-join / -s4-lifecycle / -s5-coverage). S1/S3 fixes in wf_b5d3e043-e60 merge in later.
