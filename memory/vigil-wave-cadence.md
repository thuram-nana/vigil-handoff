---
name: vigil-wave-cadence
description: "Operator's confirmed working cadence for the VIGIL FACT-coverage program — safe wave-by-wave, red-pen non-negotiable, autonomous with boundary reports"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-24T17:24:25.929Z
---

Operator confirmed (2026-09) the preferred professional cadence for building out the VIGIL FACT-coverage program ([[vigil-fact-coverage-program]]): **continue the safe wave-by-wave multi-agent method, autonomously, reporting at wave boundaries.** "Continue as before, it is more professional."

**Why:** the red-pen step is the product's whole value (near-zero false FACTs); trading it for speed would ship false FACTs — the one failure this tool cannot have. Thoroughness is the intended binding constraint.

**How to apply:**
- Core loop per wave: parallel slice builds (isolated worktrees) → adversarial red-pen per slice → sequential additive merges (rebase each onto current main, resolve shared-registry conflicts by union, re-verify the byte-identical gate + OracleKind counts on the merged tree). NEVER skip the red-pen.
- One wave at a time; all slices within a wave in parallel. Only overlap waves that touch DISJOINT code (e.g. Wave 5 SAST bridge). Do NOT parallelize waves that rewrite the same core files (e.g. Wave 7 perf).
- Bake each wave's lessons into the next wave's prompts — this compounds (Wave 4 was 3/5 first-try vs Wave 3's 0/4).
- Harden background runs: strict `timeout` on every command, idempotent reconcile-from-origin, resume-after-restart. Biggest time-sinks were stalls / re-rebases / session restarts, not the real work — let background runs finish; don't interrupt mid-run.
- Default to continuing autonomously to completion (finish current wave → next wave → … → Part 7 Gate A), surfacing only genuine hard-stops and a summary at each wave boundary. Operator interrupts only to redirect.
