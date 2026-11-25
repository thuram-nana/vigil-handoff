---
name: timing-concurrency-phase-lock
description: "Concurrent benign/treatment ALTERNATION phase-locks across workers and manufactures timing FACTs; preserving the within-point interleave is what CREATES the cross-point defect"
metadata:
  node_type: memory
  type: project
---

Found 2026-09-26 red-penning VIGIL's `perf/w7-timing-interleave`. Generalises well beyond VIGIL —
applies to any statistical A/B timing probe run concurrently. See [[runtime-redrive-live-false-fact]],
[[vigil-fact-coverage-program]].

## THE MECHANISM
A timing oracle alternates `base, treat, base, treat …` within a point so ambient load hits both arms
equally. Run N points CONCURRENTLY and the arms are still NOT concurrent with each other — at any instant
**every worker is in the SAME half-cycle**. The N workers PHASE-LOCK: the origin serves N cheap requests
during the baseline window and N expensive ones during the treatment window. Any per-request
service-time asymmetry — a longer/quoted parameter costing more to parse, regex-scan, log or error on —
is amplified **≈N-fold by self-inflicted queueing**.

**So preserving the within-point interleave is exactly what creates the cross-point phase-lock.** The
property that looks like the soundness core is the mechanism of the defect. I initially treated
"interleave preserved" as proof of soundness; that was wrong.

Compounding it: the oracle's decision rule is **ONE-SIDED**, so a zero-mean perturbation still raises the
FP rate. "Ambient load biases both arms equally" does NOT imply "cannot manufacture a positive" — the
measured shift distribution was BIMODAL (`min=-600ms`, `max=+720ms`) and only the positive mode can ever
be recorded.

## MEASURED (origin with NO sleep anywhere; service time 10ms + 10ms x len(value) => 60ms asymmetry)
| workers | confirmed | shift p90 | shift max |
|---|---|---|---|
| 1 | 0/36 | 60.0 ms | 60.1 ms |
| 12 | 2/36 | 474.5 ms | 720.0 ms |
| 24 | 6/36 | 593.8 ms | 720.4 ms |
A confirmed `time_based_sqli`, conf 0.97, **p=4.7e-4**, 717 ms Hodges-Lehmann shift, near-zero within-arm
variance — firing on the SAME points every rep (a stable phase-locked steady state, NOT a stochastic
tail). Amplification ~10-12x at 12-24 in flight. Shipped entries use `injected_ms=5000` (floor 2500 ms),
so a ~250 ms payload-dependent asymmetry at 12 in flight suffices — ordinary for an endpoint with an
error path that logs a stack trace, a `LIKE` query, or a WAF regex.
A STRICTLY payload-independent origin is safe even CPU-saturated (0/60 fired, shift bounded to +/-12 ms).

## THE FIX PATTERN: CONCURRENCY FOR SEARCH, SERIAL FOR PROOF
Use the concurrent sweep to find CANDIDATES fast; **re-measure every candidate SERIALLY before anything
mints**. Only the serial re-measurement may confirm. Keeps the wall-clock win (most points are negative
and discarded cheaply) while every FACT rests on a phase-lock-free measurement. Same shape as the rest of
the product: the fast path yields a LEAD, a controlled re-execution yields the FACT.
Weaker alternatives: de-phased/randomised per-point arm ordering PLUS a concurrency-aware effect-size
floor. NOT acceptable: keep the claim and add a caveat.

## MY OWN WRONG CALL, RECORDED SO IT IS NOT REPEATED
I advised capping in-flight at 8 because of an observed recall wobble at 12, and said "lost recall is a
quality issue, not a soundness one". Both halves were wrong:
- the recall loss was a FIXTURE ARTEFACT (a single `ThreadingHTTPServer` accept loop). Against a
  genuinely concurrent origin (asyncio, backlog=512): **60/60 confirmed at workers=1/4/8/12/24, zero lost**;
- the real exposure was PRECISION, the opposite direction.
**Lesson: when a concurrency experiment shows degradation, suspect the FIXTURE's concurrency before the
code's** — and check the other error direction before concluding which one you are exposed to.

## TWO COMPANION FINDINGS FROM THE SAME REVIEW (both about evidence that proves nothing)
- **The feature was INERT from every product entry point.** `campaign._resolve_bandit()` ALWAYS returns a
  `ContextualBandit`, and the staged plan refuses whenever a bandit exists, so the concurrent sweep ran 0
  times through `WebScanCampaign` — while the docstring told the operator "the operator caps blast radius
  here". Always verify a new knob is REACHABLE from a product entry point, not just from a test.
- **The headline gate evidence was VACUOUS.** `eval/benchmark_run.py:159` strips every timing entry
  (`if e.oracle.kind != "timing"`), so forcing `timing_workers=4/8` and getting a green gate exercised
  ZERO lines of the new code (`timing_checks_seen=0`, `sweep_staged=0`). **Instrument that the code under
  test actually RAN before citing a green suite as equivalence evidence.** This is the second
  "the gate does not cover what we assumed" finding of the session.

---

## 2026-09-27 — MECHANISM SHARPENED, AND THE FIX'S OWN COST

**The phase-lock needs QUEUEING, not merely concurrency.** My original note said N workers phase-lock and
any payload-dependent asymmetry is amplified ~N-fold. True, but incomplete: the amplification requires a
CAPACITY-LIMITED resource — one worker, one DB connection, a global lock. Measured: against a pure
`await asyncio.sleep` origin (no queue) the defect fires **0/60 at every width**; behind a capacity-1 pool
it fires **10/60 at 12 and at 24 workers, on the same points every rep**. So the reproduction fixture must
include the queue, and a test that omits it will pass while proving nothing.

**FIX AS LANDED — "concurrency for SEARCH, serial for PROOF", implemented as a stage 2b.** The concurrent
pass yields CANDIDATES only. A flagged point has its concurrent outcomes **discarded entirely** and its
whole slow sequence re-run by the literal serial loop, one point at a time, **after the ThreadPoolExecutor
has drained**, starting from a fresh `local_seen` so short-circuited siblings also re-run. Only the serial
run reaches `seen`/`_emit`. Measured after: **0 confirmed at 1/12/24 workers**; recall 60/60 at 1/4/8/12/24.

**THE COST, which must be stated with the fix:** a flagged point spends its sequence TWICE. On an
all-vulnerable surface the entire win is given back (11.8 s serial vs 12.8 s at 12 workers, 192→384
requests). The win survives only where most points are negative — the field condition. The in-tree perf
test was MOVED onto a slow-everywhere surface so it stops asserting a win on a surface that has none.

**THE RESIDUAL THE FIX CREATES — precision bought with recall.** The shift distribution is BIMODAL; a true
positive landing in ANTIPHASE gets a large NEGATIVE shift, is therefore never flagged, and **a point that
is not flagged is never re-measured**. So the fix converts a measured precision failure into an unmeasured
recall failure. Candidate remedies, cheapest first: re-measure any point whose concurrent shift is strongly
NEGATIVE as well as positive (directly targets the bimodality); bound the worker count; or re-measure every
point serially (destroys the win). **Rule: when a fix converts one error direction into the other, measure
BOTH before calling it done.**

## THREE REUSABLE TECHNIQUES FROM THE SAME ROUND
- **Pinning a lock deterministically:** hold the guard's OWN lock from outside and assert the workers block.
  Under the GIL a counter race is not reliably reproducible, so `test_counters_are_exact_under_concurrency`
  proved nothing; the lock-holding test fails as `assert 32 == 0` when the lock is removed.
- **A structural guard beats an ordering-lucky one.** The reservation-leak invariant (no gate may follow the
  budget reservation) was made structural by putting everything after the reservation inside a `finally` and
  placing `admitted = True` and `return None` on adjacent lines, so a new gate has nowhere to go but above
  the protected region.
- **A counter defeats silent fallback.** An equivalence test that compares a staged run to a serial run
  passes trivially if staging never engaged. Assert `staged_entered == 1` (and `== 0` for the control).

## AND THE HONEST OUTCOME: THE KNOB WAS REMOVED, NOT SHIPPED INERT
`timing_workers` was found unreachable from every product entry point (`_resolve_bandit()` always returns a
bandit; the staged plan refuses whenever one exists). Making it reachable would require dropping the
no-bandit precondition — impossible to do soundly, because the bandit re-ranks each point's roster from
rewards earned at EARLIER points, so staging changes request order, certificate emission order AND the
bandit's persisted end state. So the parameter was DELETED from `WebScanCampaign` (2 touch points → 0) and
the slice's own headline downgraded to: an engine-level capability with tests, not a product feature; the
shipped operator-reachable value is the `HttpExecutor` hardening. **Preferred over shipping a knob whose
docstring promises blast-radius control it cannot deliver.**

---

## 2026-09-27, FINAL: THE FEATURE WAS DELETED. The fix could not close the other error direction.

The serial-proof fix WORKED for precision — 0 false positives over 264 point-runs with 79 serial
re-measurements actually exercised, pool drain proved structurally (`ThreadPoolExecutor.__exit__` is
`shutdown(wait=True)`) and by server-side timeline, discarded concurrent samples proved unable to reach
the retained evidence, cost arithmetically exact at `baseline + flagged x 2 x samples`. **It still had to
be deleted**, for reasons that generalise well past VIGIL:

**1. The recall claim had been measured on the one fixture that CANNOT show the defect.** The FP fixture
was correctly rebuilt with a capacity-1 pool, because queueing is the cause. The RECALL number was then
taken on asyncio/backlog-512 — no capacity bound, therefore no queueing, therefore 60/60 recall
guaranteed by construction. **When you fix one error direction, re-measure the OTHER direction on the
fixture that can actually exhibit it.**

**2. Measured properly, on a capacity-bounded origin where every point carried a REAL delay:**
`timing_workers=2` against 12 points — the SMALLEST possible opt-in — lost **50%**, and was the WORST
row; 33-67% across 2/3/4/8/12/24; **50% at production scale** (3 s delay, `injected_ms=3000`). Shifts
quantised to integer multiples of ~2x delay (±624 ms on a true +300 ms channel, range −3120 to +3745):
**the bimodality is the entire distribution, not a tail.**

**3. THE DECIDING ARGUMENT — the win and the defect are mutually exclusive.** At capacity 1 the concurrent
sweep was **33-66% SLOWER** than serial (the origin serialises anyway, then flagged points pay a second
sequence); the only shape where recall survived was still 17% slower; the ~10x win existed only on the
UNBOUNDED origin, which is exactly where the defect cannot occur. **Queueing is simultaneously the
precondition for the amplification and the condition under which the speed-up does not exist.** A real
web application is capacity-bounded by definition (worker pool, DB pool, PHP-FPM children, a GIL). So the
feature helped exactly where it was harmless and harmed exactly where it was useless.

**4. The losses were false DECISIVE NEGATIVES, not honest silence.** A non-flagged point's concurrent
outcome became a `ProbeRecord` with verdict `clean` — defined in-code as "an applicable oracle
CONCLUSIVELY adjudicated the negative … exercised-and-provably-clean". 26 such records were observed on
surfaces carrying a real conditional SQLi delay. **Guarding the FACT branch left the REFUSAL branch
minting fact-strength negatives from a measurement the code's own comments called untrustworthy.**
See [[clean-verdict-soundness]].

**Both cheap mitigations were measured and failed:** two-sided `|shift| >= floor` catches 22/26 but
re-measures **92% of all points** (no win left); negative-only catches 20/26.

## THE RULES
- **When a fix converts one error direction into the other, measure BOTH on a fixture that can exhibit
  each.** A recall number from a fixture without the causal ingredient is not a measurement.
- **If a feature's benefit and its defect have mutually exclusive preconditions, the feature has no
  operating regime.** Check that before optimising it.
- **Guarding the positive branch is half a fix.** Ask what the same untrusted measurement does to the
  NEGATIVE branch — a false `clean` on a vulnerable surface is worse than a missed finding.
- **Delete, do not park.** Dead unsound code is the `dns_pin.resolve_and_validate` footgun: one import
  away from the next author. The `HttpExecutor` hardening (locks, atomic budget reserve, monotonic rate
  anchor, client warm-up above the spacing window, dry-run exemption, action-id nonce) was the slice's
  real value — 8 of 11 tests fail against base — and it survives the deletion.
