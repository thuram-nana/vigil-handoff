---
name: vigil-multilang-codescan-program
description: Multi-language codescan PR (LanguagePlugin registry + tree-sitter + 6 FACT-grade languages) — plan location, phase state, cut
metadata:
  type: project
---

Plan: `~/.claude/plans/multilang-codebase-plan.md` (recovered 2026-11-19 from the planner agent's transcript after the
scratchpad e0/ dir was wiped — scratchpad is NOT durable; keep plans in ~/.claude/plans). Worktree
/home/kali/vigil-wt-multilang, branch claude/multilang-codescan, flag VIGIL_MULTILANG_CODESCAN (default OFF).

**Cut (one PR):** registry + tree-sitter re-parse substrate + Python ref (byte-identical) + JS/TS, Go, Java, Ruby, PHP at
FULL FACT grade (broken-crypto-invocation + insecure-flag-literal, same-shape benign controls); Rust + C/C++ at DAA-fact +
fix-apply grade only (static FACT tier refused — crate spread / preprocessor). NO new egress: per-language dependency
fetchers + real-suite toolchain images are owner-gated follow-on (§4.2).

**Gotchas:** tree-sitter is NOT in /home/kali/vigil/.venv-offense; never install into the shared venvs — local installs go
to `pip --target <scratchpad>/ml-site` + PYTHONPATH; CI must install the pinned grammars and the harness must FAIL (not
skip) in CI without them. Langplugins live in integration/vigil_integration (both venvs) so every tree_sitter/framework
import must be function-local (FATAL-2). Function-local imports do not enter the oracle_version closure.

**Status 2026-11-19:** Phase 1 had been stranded uncommitted (rate limit, 11-18). Workflow wf_a43a90a6-50b: finish
Phase 1 → 6 parallel language builders (vigil-wt-ml-<lang>) → integrate → 3-lens Opus panel. Related:
[[vigil-pcr-program]], [[oracle-version-class-hole]], [[fake-fixture-pattern]].
