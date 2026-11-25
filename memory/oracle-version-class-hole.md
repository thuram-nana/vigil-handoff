---
name: oracle-version-class-hole
description: "VIGIL's oracle_version closure followed only types.FunctionType, so for any class-based oracle the version tracked the PROSE and missed the PROCEDURE — certificates silently survived behaviour changes"
metadata:
  node_type: memory
  type: project
---

Found 2026-09-27 while fixing audit-B (branch fix/audit-b-oracle-verifier). The finder found it itself,
on the way past. See [[anti-hallucination]], [[clean-verdict-soundness]], [[demotion-merge-hazards]].

## THE DEFECT
`verify/oracle_version.py::_reachable_source_blob` exists so that "a certificate minted under the old
oracle body is DETECTABLY STALE" — `evidence/pcf.py:216` fails loudly on a version mismatch. Its docstring
promised the closure covers every reachable helper "so the entry function alone no longer hides a changed
procedure". **The walk followed only `types.FunctionType`, so it stopped dead at any class.**

Measured at `9bc3e632`: mutating `_js_scan`, `_ReflectionScanner.handle_data`,
`_ReflectionScanner.handle_starttag`, `_EVENT_HANDLER_ATTRIBUTES` or `_INERT_TEXT_CONTAINERS` left
`oracle_version(reflection_context)` **BYTE-IDENTICAL** — while editing one prose string in the entry
function changed it. **The version tracked the prose and missed the procedure.** Every fix in the very
commit that found it would have shipped invisible to the staleness check.

FIX: visit a module-owned class and continue the walk through its methods; all 9 decision sites then move
the version. **Places to check for the same hole in any such closure:** nested classes,
`classmethod`/`staticmethod`/`property` bodies, a callable stored in a module-level dict or tuple, a
`functools.partial`, a closure captured in a default argument, a dataclass `__post_init__`, a callable
imported from a sibling module, a C-implemented builtin, and a method inherited from a base class the
module does not own.

## THE COMPANION DEFECT: THE VERSION WAS NOT EVEN DETERMINISTIC
Measured over 47 kinds in REAL SUBPROCESSES at PYTHONHASHSEED 0/1/2/7/12345, at three separate revisions:
- `_CMD_ALT = "|".join(sorted(set(_SHELL_CMDS), key=len, reverse=True))` and `_NOSQL_OP_ALT` baked **set
  iteration order** into a compiled pattern string that `_canon` hashes via `repr` → 5 distinct hashes
  across 5 seeds. Fix: key `(-len, s)`, a total order preserving longest-first.
- `_SECRET_RECOGNIZERS` holds **lambdas**; `_canon` recursed to `repr(<lambda at 0x…>)` and hashed a
  **heap address** → a different version in EVERY process. So every `secret_credential_validity`
  certificate was permanently "oracle_version CHANGED since mint" and could never verify fully-sound.
  Fix: `_canon` names a callable stably, carrying its source for in-module functions so a behaviour change
  is still detected.
Result after the fix: all 47 kinds byte-identical across 15 processes at 5 seeds.
**The in-repo test named `..._is_deterministic` could not catch any of this — it compared two IN-PROCESS
calls behind `@lru_cache`. A determinism test must spawn SUBPROCESSES and vary `PYTHONHASHSEED`.**

## WHY THIS MATTERS BEYOND VIGIL
A content-addressed "version" over source is only as good as its reachability walk, and every gap is
silent by construction: nothing fails, the version simply does not move. **Test a staleness mechanism by
MUTATING THE DECISION PROCEDURE AND ASSERTING THE VERSION MOVES** — for each kind of callable the
language allows — rather than by reading the walk.

## A GROUND-TRUTH TECHNIQUE WORTH REUSING
When browser behaviour is the ground truth, make the page COMPUTE the sentinel at runtime — e.g.
`document.documentElement.setAttribute('data-gt', ['OBSIDIAN','EXEC'].join('-'))` — so a `--dump-dom`
grep cannot be fooled by the page's own source bytes. The same idea applies anywhere an oracle greps
output that the input could have contained verbatim.
