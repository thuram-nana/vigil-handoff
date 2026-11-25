---
name: fake-fixture-pattern
description: "SEVEN fixtures found in one session that SIMULATED a vulnerability's signature instead of having the property — including the benchmark gate corpus itself. Check the fixture HAS the property, not that it matches the detector."
metadata:
  node_type: memory
  type: project
---

Found across 2026-09-26/27 in VIGIL. Seven distinct fixtures, in product tests, in the benchmark gate,
and in a verification harness. This is the most repeated finding of the programme.
See [[runtime-redrive-live-false-fact]], [[clean-verdict-soundness]], [[demotion-merge-hazards]].

## THE PATTERN
A fixture is written to make a detector fire. If the detector is unsound, the fixture ends up
SIMULATING the bug's SIGNATURE rather than HAVING the property. Then:
- the unsound detector passes,
- the SOUND detector correctly REFUSES,
- and the refusal looks like a regression, so the pressure is to "fix" the detector.
**Every one of these was discovered by a SOUND detector failing against it.** That is the tell: when
hardening an oracle makes a long-green positive fixture go red, suspect the fixture first.

## THE SEVEN
1. **The benchmark GATE corpus.** `boolean_sqli @ /users?name` string-concatenated (a real injection) but
   "evaluated" with TWO REGEXES understanding only literal equality. Of 7 syntactically-varied clause
   families only 1 partitioned; the SPRT correctly refuted. **The gate had been certifying that an
   unsound oracle still fires.** Every "1069 reqs byte-identical" attestation partly rested on it.
2. **`test_engine_runtime_redrive_seam::_VulnApp./download`** — the suite's own "vulnerable traversal"
   origin was `any(t in f for t in ("..","%2e","/etc/","passwd"))`: a payload-correlated substring
   matcher, not a read.
3. **`test_release_gate_deceptive_tool::_TLSServer`** — named as "a STRONG endpoint", but Python's default
   `PROTOCOL_TLS_SERVER` on this OpenSSL has `minimum_version=SSLv3` and enables ANONYMOUS key exchange
   (ADH/AECDH, no server authentication). The probes minted two TRUE FACTs against it.
4. **`test_campaign`'s `/search`** — keyed on the payload SHAPE (`'1'='1` in q), i.e. exactly what a WAF
   tautology rule in front of a static page does.
5. **`scanner/tests/test_m2_sqli_library.py:268-282 _SqliVulnApp`** — the same shape-matching interposer
   with no SQL engine, and it is where the 14 dropped dialect entries' end-to-end coverage now LIVES. So
   that coverage is "covered, against a fixture of the kind we just condemned".
6. **`eval/tests/test_validation.py::_VulnApp`** — `_looks_sqli` matched `"'='"` or `" or "`, which EVERY
   clause a sound detector sends contains: measured **7/7 TRUE and 7/7 FALSE both matched**, so no
   truth-value partition existed and even a correctly wired SPRT would refuse it.
7. **A VERIFICATION HARNESS, not product code.** A recovered `dump_consumers` harness reported success
   while every consumer errored on a signature mismatch and the derivation fired **0 rules**. A harness
   that reports success while measuring nothing is indistinguishable from one that works.
   (Related, same review: `best_paths(w, src, dst, k=3)` silently returned `[]` because arg 3 is
   `objective_kinds`, not a destination — so that leg of an EARLIER red-pen was vacuous too.)

## THE RULES
- **Check the fixture HAS the property, not that it matches the detector.** A planted SQLi must run a real
  engine; a planted file-read must read a real file; a "strong TLS endpoint" must actually be strong.
- **Non-vacuity assertions belong in the fixture's own test.** Assert the fixture produced a discriminating
  observation — e.g. that all 7 TRUE clauses agree, all 7 FALSE agree, and the two clusters are disjoint —
  BEFORE asserting the verdict. The gate-corpus slice's `_assert_detector_really_ran` is the model: it
  requires non-empty SPRT rounds carrying the full clause sets plus their determinism repeats, asserts the
  control is still SHAPE-READABLE, and carries a positive control so the file cannot pass by refusing
  everything.
- **A corpus that does not DISCRIMINATE sound from unsound is only half a gate.** Both detectors agreeing
  on both fixtures means a revert to the unsound one passes cleanly. The fix is a fixture the sound
  detector REFUSES and the unsound one MINTS — a WAF-tautology page and a strict-input-validation page,
  both static, both with no database.
- **When a sound detector refuses a long-green fixture, that is EVIDENCE ABOUT THE FIXTURE.**

## THE COMPANION TRAP: DEMOTION WITHOUT A REPLACEMENT PRODUCER
Removing `DIFFERENTIAL_RESPONSE` from `boolean_sqli`/`auth_bypass`/`nosqli` closed a rate-1.0 false FACT —
but `DEFAULT_CHECKS` carries a bare `DifferentialCheck` and **no `BooleanInferenceCheck`**, and the library
entries for those classes are all `differential`. So the default scan path was left with **no producer of a
confirming oracle context for those classes at all**: the sound SPRT is wired only into the gate profile.
A false-FACT closure that removes the true positive on the path operators actually run is a **pure
capability removal**. **Rule: when demoting a class, name the sound producer that replaces it ON EVERY
PATH — and if one class genuinely has none (here `auth_bypass`, which routes to `ACHIEVED_STATE` only),
register that as a named capability loss rather than inventing a producer.**
