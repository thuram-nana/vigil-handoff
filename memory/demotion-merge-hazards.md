---
name: demotion-merge-hazards
description: "Two traps when demoting an unsound class: the demotion must reach EVERY render path (or it becomes a silent gap), and a sibling slice editing the same registry object can silently re-arm it on merge"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-28T22:04:30.848Z
---

Both found 2026-09-27 red-penning VIGIL's `fix/audit-a-campaign`. See [[clean-verdict-soundness]],
[[runtime-redrive-live-false-fact]], [[claim-discipline-meet-up]].

## TRAP 1 — DEMOTION IS NOT DISAPPEARANCE
Demoting `request_smuggling` from FACT to UNCONFIRMED LEAD reached exactly ONE of three render paths.
Measured against a benign stock-404 origin producing 2 leads and 2 probe records:

| surface | mentions the class | carries `blocking_work` |
|---|---|---|
| `to_json` / `build_report["arsenal_leads"]` | yes | yes |
| **`to_sarif`** | **no — 0 results** | no |
| **`to_html`** ("A self-contained human report") | **no** | no |
| `coverage_verdict["classes_inconclusive"]` | **no** | — |
| console API / `packages/vigil-ui/app.js` / spine event | **no** | no |

An operator rendering HTML or SARIF saw NOTHING — not the lead, not the blocking work, not even that the
class had been probed. **That converts a false FACT into a SILENT GAP**, which is the same failure this
programme keeps rediscovering (the `.env` leak dropped with a fabricated note; the arm that was "still
driven" but sent zero requests).
**RULE: after demoting a class, enumerate every render/export path and confirm the operator still SEES the
class, its verdict and its `blocking_work`. A demotion that is invisible is a regression, not a fix.**
(Corollary from the same review: SARIF must carry it at `note`/`warning`, never `error`.)

## TRAP 2 — A SIBLING SLICE CAN SILENTLY RE-ARM THE DEMOTION ON MERGE
A whole demotion can live in ONE JSON object. `fix/audit-a-campaign` set
`request_smuggling.differential_desync` to `fact_capable:false` in
`docs/capability-matrix/evidence-branches.json`. **`strix/s2-transport-intake` edits the SAME object and
keeps `fact_capable: true` with the OLD four-precondition list.** If S2 lands second and its side of that
conflict wins, the class becomes FACT-capable again and the untouched `live/smuggling_redrive.py` mints
immediately. A tripwire exists (`test_admission_refuses_the_single_connection_differential`), but the
sequence must be deliberate, not left to `git`.
**RULE: before merging parallel slices, diff every shared REGISTRY OBJECT — not just file paths. A
capability flag is a single token whose loss is invisible in a conflict resolution.**
Related, from the same wave: S1 rewrites the `alias_keys` disjointness hunk as a hand-written 9-arm dict
omitting S2's two arms, so **resolving S1-side silently deletes the S2 pin**. Correct resolution recorded
as: S2-side plus S1's three `class_sets`/`cwe_maps` rows.

## WHAT GOOD LOOKS LIKE: INDEPENDENT DEFENCES
`fix/audit-b-oracle-verifier` refuses the same class at a DIFFERENT layer — the oracle discriminator
(`_DIFFERENTIAL_BASELINE_REFUSED = frozenset({"request_smuggling"})`) — and does not touch the registry
file at all. Two defences, neither depending on the other, so a bad conflict resolution on one still
leaves the class refused. **Prefer two independent refusals at different layers over one authoritative
flag** — and note the codebase already states this pattern at `verify/verifier.py:52-58` ("this absence is
the SECOND, independent refusal").

## A THIRD, SMALLER ONE — "BYTE-IDENTICAL" NEEDS A SCOPE
A slice claimed "every existing certificate/export is byte-identical" after adding two fields defaulting
to `""`. Measured: the `oracle_context` digest, `to_sarif` and `to_html` were identical, but
`ScanReport.model_dump_json` — which IS `--reverifiable-out`, the re-verification INPUT — moved, because
every finding row gained `"title":"", "severity":""`. The precedent sat six lines above the writer
(`scanner/cli.py:203-207`, a new default-`None` field explicitly `exclude`d for exactly this reason).
**RULE: "byte-identical" must name WHICH artifact. A moved re-verification input is a moved proof.**

---

## TRAP 1, EXTENDED (2026-09-27): THE IMMUTABLE EVENT STREAM IS A RENDER PATH TOO
The demotion audit found three render paths missing the class (`to_sarif`, `to_html`, the coverage
verdict). Checking the rest turned up a FOURTH, and the worst of them:
`engage._spine_finding_payload` (`engage.py:618-619`) hardcoded
`"title": f"{bug_class} confirmed at {param}"` and `"severity": "High"`, so a producer's explicit
"…UNCONFIRMED", Medium verdict was written to the **append-only spine** as a CONFIRMED HIGH.
**A false label on an immutable stream is strictly worse than one in a report**: the report can be
regenerated, the spine is supersede-only and the wrong record stays forever as the primary audit
artifact. When enumerating render paths after a demotion, include the spine/blackboard writers,
the console API AND its fallback doc, and the UI — and grep those writers for literal `"confirmed"`,
`"High"`, or an f-string composing a verdict, because the same relabelling has now been found in FOUR
places (`_arsenal_finding`, `report.py`, the spine payload, and the report re-labeller).

## A SERIALISER RULE THAT CAME OUT OF THE SAME ROUND
Adding two fields defaulting to `""` moved `--reverifiable-out` (the re-verification INPUT), because
every finding row gained `"title":"", "severity":""`. The clean fix is a `model_serializer` that
**omits a producer claim that was never made** (the precedent is `scanner/cli.py:203-207`, where a
new default-`None` field was explicitly `exclude`d for exactly this). That restores byte-identity
while a producer that DID hedge still carries its hedge.
**But the rendered report is a different case and should NOT hide an empty key:** `to_json` keeps
`arsenal_leads` even when empty, deliberately, because a key that vanishes when empty re-creates the
very ambiguity the demotion work exists to kill (absent vs none-found). Correct the CLAIM, not the
behaviour. **Rule: omit-when-unset for re-verification inputs; always-emit for operator-facing
reports.**

## TRAP 4 (2026-09-28): A SOUND DEMOTION LEAVES A TEST-DEBT CASCADE THE GATE DOES NOT CATCH
Landing `fix/gate-corpus-sound-boolean` (#855): demoting `boolean_sqli`/`nosqli`/`ldap`/`xpath`'s bare
differential to LEAD-only was correct, but the WHOLE report/veracity/evidence/console test corpus built its
canonical "confirmed FACT" fixture from exactly that now-demoted path (`report/tests/conftest.py::firing_ctx`,
`_oracle_ctx`, stored JSON). ~95 tests went red at once. **`make gate` PASSED throughout** — it runs a
different corpus (benchmark_app), so it never sees unit-fixture debt. Fix = migrate the canonical fixture to a
class that STILL soundly confirms (boolean_sqli→error_based_sqli, keeping the differential context so metadata
is byte-equal; nosqli→lfi via SIDE_EFFECT for the --library-adjudicates test), NEVER weaken the assertion.
**RULE: after demoting a class, grep the ENTIRE test tree for it as a confirmed-FACT exemplar, not just the
scanner tests; the gate's green is not coverage of the unit corpus.**

## TRAP 5 — A CLASS-SPECIFIC FORGERY TEST MUST SELECT THAT CLASS'S PROBE
`test_posture_reexecutable::test_forged_negative_is_refused` forged open_redirect fields
(location_host/canary_host/status) but picked "the first clean re-executable probe of ANY class". A roster
reorder (or a new re-executable class like graphql_introspection at "/") floats a different class first, the
open_redirect forge no longer fires that probe's predicate, and the tamper SILENTLY PASSES ("DID NOT RAISE").
Deterministic, not flaky. **RULE: bind a forgery/tamper test to the probe class its forge targets.**

## TRAP 6 — A SOUND-BUT-EXPENSIVE DEFAULT CHECK CAN STARVE A DOWNSTREAM PASS
Putting the sound SPRT `BOOLEAN_SQLI_SPRT` (~64 req/point) in DEFAULT_CHECKS exhausted `engage`'s per-engagement
request budget (200) on the audit ALONE, so the additive arsenal/discovery pass never ran (0 discovered paths).
Fix = raise the fail-closed cap (200→1000); it is monotonically SAFE for false-clean (a false clean comes from
too FEW requests, red-pen confirmed `active_findings` identical at 200 vs 1000). But it is a ~5x default
on-the-wire increase — an OPSEC-relevant default worth surfacing to the operator. Also: the runtime_redrive
path_traversal arm regressed to INCONCLUSIVE because ContentSignatureCheck's NEW internal benign control read a
bare-404 body that the arm folded into the TREATMENT's readability window — fix was to send the treatment
directly (`from_side_effect`) so `treatment_unreadable` measures the treatment alone. When one slice gives a
shared check a new internal control, audit every OTHER consumer that measures readability around it.

## TWO SMALL THINGS WORTH REMEMBERING
- **`"٥".isdigit()` is True and `int("٥")` is 5.** A `Content-Length` validated with `str.isdigit()`
  accepts Unicode digits. Match `[0-9]+` when parsing a protocol field.
- **A limitation you deliberately will not fix needs a POSITIVE test.** The reader could not separate
  a genuinely pipelined response from an under-declared `Content-Length`'s leftovers, and the rule
  that would separate them (a sent-request watermark) would silently discard the very origin the lead
  exists to cover. So the boundary is pinned by a test that asserts the limitation holds — it fails if
  the limit is silently FIXED or silently WIDENED, so the docstring cannot drift back into claiming
  more. Better than a comment.
