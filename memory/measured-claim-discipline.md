---
name: measured-claim-discipline
description: "A number written beside code is a claim. Three VIGIL slices were BLOCKED purely on unreproducible measurements — no code defect. Fix: commit the harness or withdraw the number."
metadata:
  node_type: memory
  type: feedback
---

2026-09-26/27. Three slices were blocked with **nothing wrong with the implementation** — only with the
numbers written down beside it. This is now the second-most-common block after the fake-fixture pattern.
See [[fake-fixture-pattern]], [[claim-discipline-meet-up]], [[oracle-version-class-hole]].

## THE RULE
**Every number in a docstring, a registry row or a commit message is a CLAIM and must be re-derivable
from a committed harness or from a command printed beside it.** If you cannot commit the harness,
withdraw the number. A figure from an uncommitted scratch script is unauditable and rots silently.

## THE THREE BLOCKS
- **W7-graph** — four figures did not reproduce, including the worst shape: a docstring **withdrew two
  unreproducible speedups and justified the withdrawal with a third unreproducible number, in the same
  sentence.** Root cause when diagnosed: the 3.5-4.5x came from a hand-rolled loop calling
  `nodes_of_kind` per node, not from `saturate`, the consumer it named. Real answer: **1.01-1.06x, no
  measurable win.** Also: "read‖write 16/30 at the default" was **0/30, three times** — it came from an
  uncommitted scratch harness.
- **rr-fix** — "five of twelve origins moved LEAD→INCONCLUSIVE" was **three**; the author had read the
  after-column alone instead of measuring the DIFFERENCE against an extracted pre-change tree. And "zero
  `vigil_integration` imports under `scanner/` or `verify/`" was **five**, all in `scanner/tests/`.
- **fx-b** — "eight oracle kinds change version" was **five**; the measurement was contaminated by three
  kinds that were never deterministic at all.

## FIVE SUB-RULES THAT EACH CAUGHT SOMETHING REAL
1. **A difference must be measured as a difference.** Extract the pre-change tree and run the same
   harness against both. Reading the after-column and attributing everything in it to your change is how
   "five" happened.
2. **Measure in SUBPROCESSES when anything could be process-local.** `PYTHONHASHSEED`, heap addresses and
   `@lru_cache` all hide non-determinism from an in-process comparison. A test literally named
   `..._is_deterministic` compared two in-process calls and proved nothing.
3. **Check your counterfactual is valid before trusting it.** W7-graph isolated a lock by swapping in a
   no-op context manager, measured ~1.00x, and nearly "refuted" a correct finding — a Python
   `__enter__`/`__exit__` costs about what a C `RLock.acquire` does, so it measures nothing. The valid
   method was comparing against the unlocked REVISION.
4. **A threshold measured on one host is not a constant.** "Fires at ~7,500 dirty buckets" reproduced on
   one machine and not another. The real rule was "whether the dirty loop outruns the 5 ms GIL quantum"
   (6.63 ms at 7,500 here, 4.84 ms there). **Publish the RULE plus both observations, not the constant.**
   Neither party was wrong; the portable constant was.
5. **Publish YOUR measurement, not the reviewer's.** When W7-graph could not reproduce the reviewer's
   5.7% lock-construction share it published its own ~3-4% and flagged the disagreement, rather than
   adopting a number it had not seen. Correct instinct.

## WHAT "FIXED" LOOKS LIKE
Not a corrected number — a **committed, revision-aware harness**: `bench_graph.py --compare-rev <sha>`
(extracts the old module out of git and measures against it), `--order desc`, `--mode operators`, plus a
`race_graph.py --rev` so a rate quoted for a now-superseded revision stays derivable. Every sweep also
prints its own fixture profile (e.g. the dirty-bucket counts), so a favourable fixture cannot pass for a
general one. That removes the CONDITIONS that produce bad numbers instead of correcting one instance.

## AND: RECORD THE CAUSE, NOT JUST THE CORRECTION
rr-fix wrote the mechanism of its own error into the registry row — *"An earlier revision of this
sentence said FIVE, having read the after-column alone and attributed every INCONCLUSIVE in it to this
change; that is the mistake this paragraph exists to prevent."* More likely to prevent a recurrence than
a corrected figure alone.

---

## COROLLARY (2026-09-27): A STATIC CONFIG IS INERT IF A DOWNSTREAM COMPONENT RE-DERIVES IT
Restoring sound `boolean_sqli` detection put a 64-req `BooleanInferenceCheck` into `DEFAULT_CHECKS`,
which under a bounded audit budget starved cheaper checks (soak at cap=160 found 0 of a planted XSS).
The obvious fix — reorder `DEFAULT_CHECKS` cost-ascending — was **measured to do nothing**: `AuditEngine`
re-sorts checks at every insertion point by a `ContextualBandit`, and `ContextualBandit.rank` broke ties
(all posteriors equal on a uniform bandit) by **arm id = alphabetically by bug_class**, floating
`boolean_sqli` (b…) to the front and `xss` (x…) to the back on EVERY real scan. The real fix was the
engine tie-break (order by learned posterior, ties by cost-ascending position, not alphabetical id).
**Rule: before "fixing" an ordering by changing a static list, check whether anything downstream re-sorts
it. Measure the fix end-to-end through the real caller (the campaign, with its bandit), not through the
standalone component — the standalone `AuditEngine` found the XSS at cap 156 while the campaign did not,
and that gap WAS the bug.** An alphabetical tie-break is a silent, data-independent reordering that no one
wrote as policy; audit tie-breaks in any ranker for accidental key-order dependence.
Also: this change lands in the RL RANKING path, so it needed its own red-pen for the invariant "learning
may re-rank/defer but never promote, oracle is sole authority" — a tie-break is allowed to move ORDER but
must not move WHAT IS FOUND (verified only under the unbounded default, where every check runs regardless
of order; under a cap, order is coverage).
