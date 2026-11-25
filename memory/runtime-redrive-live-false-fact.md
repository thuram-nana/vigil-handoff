---
name: runtime-redrive-live-false-fact
description: "CRITICAL (found 2026-09-26): path_traversal + exposure mint FALSE signed FACTs at rate 1.0 through runtime_redrive on MAIN, reachable from the governed engage path; plus a FABRICATED note in the audit trail"
metadata:
  node_type: memory
  type: project
---

Adversarial red-pen of Strix S1 (`strix/s1-runtime-classes` @ `ad5e0464`) constructed **working false
FACTs** — signed certificates persisted to `proofs/reverifiable.json` that pass the offline
`python3 -m framework.v2 verify` CLI with rc 0 — at **measured rate 1.0**. Fix branch:
`fix/runtime-redrive-soundness`. See [[vigil-strix-fact-breadth-program]], [[anti-hallucination]],
[[claim-discipline-meet-up]].

**NOT confined to the unmerged slice — LIVE ON MAIN.** `RUNTIME_FACT_CLASSES = ("path_traversal","xss",
"exposure","ssi")` (`integration/vigil_integration/live/runtime_redrive.py:44`) is already shipped, and
`wiring.py::_live_redrive_fact` (~:1936-1945) routes any governed-engage finding of those classes into
`runtime_redrive`. S1 only added a SECOND door (the Strix proof-sink rail) to an already-unsound room.

**D1 — `path_traversal` fires on payload-CORRELATED benign content (no file read).** The control at
`runtime_redrive.py:282-296` refuses only a page that prints the signature REGARDLESS OF INPUT
(constant emission); the oracle is a bare `haystack.find("root:x:0:0:")` (`verify/oracles.py:1232`
`side_effect_oracle`). A read-only docs/KB **search endpoint** whose corpus contains an /etc/passwd
article: control `?q=vigilctl<nonce>.txt` -> "0 results", treatment `?q=../../../../etc/passwd` -> the
article quoting `root:x:0:0:` => **12 signed FACTs**. The control value is not PATH-SHAPED, so it
retrieves nothing and the differential is vacuous.

**D2 — the `exposure` "random soft-404 control" is a STATUS check, not a signature differential.**
`PathProbeCheck.probe` (`scanner/checks.py:1332-1348`) returns a `FindingContext` for ANY status not in
`(0,404)`; the signature predicate rides INSIDE the returned context and is never evaluated by the check.
**The worst consequence is a FABRICATED AUDIT-TRAIL STATEMENT, not a false positive:** behind an SPA that
200s unknown paths, a REAL `DB_PASSWORD=` leak is REFUSED with the persisted note *"signature also served
at a random path (soft-404/catch-all)"* when the control body contained no such string — and `/.env` was
never fetched at all. So the arm is also silently DEAD against any origin that does not 404 unknown paths
(SPA/custom-404/403/500 = most of the modern web). For a product whose value IS the trustworthiness of the
record, an untrue sentence in the proof record outranks a false positive.

**D3 — "value-shaped signature" does not exist**: the predicate is a bare
`{"icontains":[{"var":"body"},"DB_PASSWORD="]}`. Minted on docs prose, a CI template
`DB_PASSWORD=${{ secrets.DB_PASSWORD }}`, and a JS bundle `cfg.env="DB_PASSWORD="+process.env.DB_PASSWORD`
— the last is VERBATIM the case a docstring claimed was refused.

**D4 — 12 duplicate certificates for ONE bug**: the traversal and ssi arms lack the first-FACT `break` that
`ssti_redrive`/`boolean_redrive`/`timing_redrive` have (`runtime_redrive.py:568-572,695-699,764-768`).

**Fix belongs in `runtime_redrive.py`, NOT `scanner/checks.py`** — that module is BY DESIGN where the sound
guards live that the core checks lack (benign control, dose-response, tag-only acceptance). Required:
path-shaped control; a SECOND semantically-different traversal encoding must reproduce the signature;
evaluate the signature predicate on the CONTROL BODY in the runner; never emit an unobserved note; a real
value shape; first-FACT break.

**Lessons that generalise:**
- A differential control is only sound if the CONTROL VALUE IS THE SAME SHAPE as the treatment. A control
  that cannot plausibly retrieve anything proves nothing. Extends [[vigil-fact-coverage-program]]'s
  same-shape principle from the target to the PAYLOAD.
- **Payload-correlated output defeats constant-emission controls.** "Refuses a page that always prints the
  signature" is much weaker than it sounds — the realistic FP is content correlated with what the payload
  MEANS.
- **A negative control that only asserts `result is None or not result.is_fact` is greenwash**: 4 of 6
  shipped controls still PASSED with every arm stubbed to `return None`. Assert positively that the arm RAN
  and the oracle REFUSED.
- **Never write a note you did not observe.** A fabricated justification in a persisted proof record is a
  worse defect than the false verdict it accompanies.
- What HELD and is worth keeping: gate byte-identical 11/0/0 at 1069 reqs across 3 runs; no cert can claim a
  class the runner did not literally pass (singleton classifier codomains); cross-class laundering and
  tamper rejected; the `ssi` arm survived every attack because its per-probe UNGUESSABLE PRODUCT plus
  raw-directive-absent requirement is a genuine CAUSATION proof, not a signature match. Signature-matching
  arms are the rot; causation-proving arms held.

---

## SECOND LIVE FALSE-FACT ON MAIN (found 2026-09-26, independent mechanism): the UNPINNED TIMING arm

`verify/verifier.py:1208-1215` reads `alpha=float(ctx.get("timing_alpha", 0.01))`, plus `timing_injected_ms`
and `timing_dose`, **straight from the UNTRUSTED report context**. Measured on `time_based_sqli` /
`time_based` / `time_based_command_injection`:

| ctx | verdict |
|---|---|
| samples `[100..104]` vs `[105..109]` (5 ms shift), defaults | not confirmed — "median shift 5ms < floor 250ms" |
| **same samples + `timing_injected_ms: 1.0`** | **CONFIRMED** — floor becomes 0 ms |
| overlapping samples, defaults | not confirmed — "rank-sum p=0.07184 >= alpha=0.01" |
| **same samples + `timing_alpha: 0.5`** | **CONFIRMED** — p=0.0718 accepted |

Ordinary network jitter mints a `time_based_sqli` FACT. The **boolean** arm (`verifier.py:1217-1232`)
already pins its parameters and documents why; the timing arm simply was not given the same treatment —
while being cited as "the dedicated statistically-sound oracle" that trust is routed TO. Fix = pin
`timing_alpha`/`timing_injected_ms`/`timing_dose` as protocol constants, mirroring the boolean arm.

## THIRD: the F8 protocol-discriminator pin is bypassable THROUGH THE FIELD IT KEYS ON
`verifier.py:81-94` keys the pinned discriminator on `normalize_bug_class(ctx["bug_class"])` — which the
producer of the untrusted report **also writes**. Two BYTE-IDENTICAL bodies + a fabricated
`latency_ms: 9999.0` mint a confirmed `request_smuggling` FACT: no canary, no desync, no second
connection. Four aliases reach the row (`http_request_smuggling`, `http_desync`, `request_desync`,
`response_smuggling`).
**A producer-side demotion does NOT close it** — `framework.v2 verify` re-executes whatever ctx a
hand-edited report carries, so the pin TABLE must lose the `request_smuggling` row and the DIFFERENTIAL
arm must refuse `baseline`/`mutated` for that class outright.

## GENERALISED LESSONS (these are the durable ones)
- **A pin keyed on a producer-writable field is not a pin.** Any "the AUTHORITY chooses this, not the
  producer" claim must be checked against *every* ctx field the decision reads — including the class
  selector itself.
- **Runner-side guards do not fix the OFFLINE path.** `framework.v2 verify` re-executes the retained ctx,
  so a precondition enforced only in the live runner leaves the certificate's re-verification weaker than
  the mint-time decision. This is why `path_traversal`'s new controls (branch
  `fix/runtime-redrive-soundness`) are a runner-side residual: `BUG_CLASS_ORACLES["path_traversal"] =
  (SIDE_EFFECT,)` takes ONE `(marker, sink)` pair and cannot express a negation or a conjunction over
  several captures. `exposure` does NOT share the gap — it compiles the whole value-shape decision into
  the JSON predicate AST the certificate carries, so it genuinely re-executes. **Prefer compiling the
  decision into the predicate AST over enforcing it in the runner.**
- **A hand-rolled lexer that decides whether a FACT mints is a false-FACT generator.** fx-b's JS lexer
  claimed "every misparse biases toward NOT-code, never a manufactured FACT" — refuted: with no
  regex-literal state, an apostrophe in a regex opens a phantom string that the NEXT real string's opening
  quote closes, leaving real string contents at "code position". A page using the canonical Django
  `json_script` / Rails `json_escape` defence plus one benign `/(\w+)'s\b/g` minted 3 High xss FACTs at
  conf 0.90. **Remedy pattern: FAIL CLOSED ON PARSER AMBIGUITY** (downgrade to advisory when a
  regex-literal candidate or `<!--`/`-->` precedes the marker) rather than trying to out-parse the
  grammar.
- **A prefix test is not a closed vocabulary.** `a.startswith("on")` fired on `on-click`, `once`,
  `online`. Use the closed set of HTML event-handler content attributes. Same family of bug as the bare
  `icontains` "value-shaped signature".
- **`side_effect_oracle` still fires at 0.9 on bare substring presence and is the SOLE confirmer for
  `path_traversal` and `lfi`.** Routing `xss` away from it was already true on base — not something any
  slice achieved. Needs its OWN slice (payload-correlation / content-provenance guard in the oracle, with
  path_traversal/lfi negative controls); folding it into an unrelated slice invites unreviewable blast
  radius.

---

## THE DESIGN PRINCIPLE THIS WHOLE EPISODE TAUGHT (2026-09-26, most important entry here)

**You cannot prove a side effect by varying the SHAPE OF THE INPUT. You must test a relationship
between the RESPONSE and the thing you claim was touched.**

The `path_traversal` fix added two input-shape guards — a path-shaped sibling control and a
second-encoding corroboration. A red-pen defeated BOTH at rate 1.0 with 8 benign origins that read no
file, including three with no normaliser at all (`v.endswith("passwd")`, an anchored WAF regex
returning a 403 that quotes a sample line, `v.rsplit("/",1)[-1] == "passwd"`).

Two reasons the guards were structurally unfixable, both worth remembering:
1. **The path-shaped control APPENDS a suffix** (`/etc/passwd.vigilctl<nonce>`), which mutates exactly
   the token real matchers key on — the basename. Changing `in` to `endswith` in the suite's OWN
   negative fixture — three characters — flips it from refusing to minting. That was the entire margin
   the fix bought.
2. **Both alternate encodings canonicalise to the same path**, so `posixpath.normpath`-based
   endpoints — i.e. the TEXTBOOK-CORRECT defence — corroborate. The fix systematically minted false
   CRITICALs against *correctly hardened* applications.

**The decisive observation:** the suite's true-positive fixture `_LfiApp` and the worst false positive
are THE SAME PROGRAM with a different dict value (an advisory *about* /etc/passwd instead of its
contents). No input-shape guard can separate them, because neither looks at the response↔file relation.

**What sound looks like — copy the `ssi` arm.** `ssi` survived every attack across two red-pens because
its evidence is a **per-probe UNGUESSABLE PRODUCT the server must COMPUTE**, plus a requirement that the
raw directive not survive. That is a CAUSATION proof. Every arm that rotted was a SIGNATURE MATCH.
Candidate sound shapes for a file read: multi-file corroboration (a second system file whose format
VIGIL validates independently — a canonical-path KB has an article for `passwd` but cannot produce a
well-formed second file), a size/entropy relation, or a per-probe unpredictable target whose content
VIGIL already knows.

**Precedent the project had already set and then contradicted:** `verify/verifier.py:384` and `:401`
dropped SIDE_EFFECT for `postmessage_exploited` and `csp_bypass` — *"a bare marker-in-sink is not a
sound proof of execution and let a crafted finding mint a FACT with no execution."* `path_traversal`
was left on that same predicate. **Check for an existing in-repo adjudication of a predicate before
building on it.**

RESOLUTION: `path_traversal` removed from `RUNTIME_FACT_CLASSES` → LEAD-only, with both prerequisites
named as blocking_work. `exposure`/`xss`/`ssi` keep minting. This is NOT softening a provable claim —
it is declining to call an input-shape observation a read. See [[claim-discipline-meet-up]].

### Two smaller rules with teeth
- **A negative control chosen to avoid the failure mode is worse than none.** The encoding-corroboration
  guard was pinned by exactly ONE fixture, whose normaliser (`_strip_leading_dots`) was the only one in
  the corpus weaker than `posixpath.normpath`. Mutation matrix also showed the constant-emission control
  was pinned by NOTHING (removing it failed zero tests) while costing 12 extra live requests.
- **A truncating extractor must never mint.** The value-shape extractor cut on `& # ;`, so
  `DB_PASSWORD=django-insecure-8k#l2@v...` minted a certificate carrying `secret_value =
  "django-insecure-8k"` — a TRUNCATED secret asserted as signed evidence — and
  `DB_PASSWORD=abc;defghij` was refused with "its right-hand side is only 3 characters long" when it is
  11: the runner's own cut created the fact it then complained about. Measured recall over 22 realistic
  credential shapes: **50%**; bcrypt/argon2/sha512-crypt **0%** (all contain `$`); base64 51.8% (the `+`).
  **Disclose a recall cost as a NUMBER, never as prose.**

---

## 2026-09-27 — THE DEMOTION BINDS ONE ROUTE ONLY. Measured per-route table.

`path_traversal` was demoted in `live/runtime_redrive.py` (branch fix/runtime-redrive-soundness). A red-pen
measured every other route:

| route | still mints a path_traversal FACT? |
|---|---|
| `python3 -m framework.v2 scan` | **YES — 11/11 benign origins, `grounding:"fact"`, High, conf 0.90** |
| `python3 -m framework.v2 verify` (offline) | **YES — and a HAND-WRITTEN report re-verifies `ok=True`** |
| **`aegis/inspect.py:547`** | **YES — `decision='confirmed'`, `action='block'`** |
| `scanner/library.py:404` (`kind=="content"` → ContentSignatureCheck) | YES, same uncontrolled path |
| `runtime_redrive` / `wiring.py` governed engage / Strix proof rail | NO — closed |

**The AEGIS row is the worst of these and deserves its own slice:** the DEFENSIVE product issues a false
**block** on legitimate traffic — a benign hardening advisory that quotes a sample passwd line gets
`action='block'`. An offensive false positive wastes an analyst's time; a defensive one takes a customer's
page down.

**Why `fact_capable:false` did not close it.** `verdict.admit` (`live/verdict.py:153`) is the ONLY reader of
`evidence-branches.json`, and the core engine never imports the integration layer (zero `vigil_integration`
imports under `scanner/` or `verify/`). The codebase's own precedent (`verify/verifier.py:52-58`) calls the
registry flag "the SECOND, independent refusal" — but that only holds for paths that go through `admit`.
The FIRST refusal must be the `BUG_CLASS_ORACLES` row: `["path_traversal"]`/`["lfi"] = (SIDE_EFFECT,)`
(`verifier.py:139-140`) are what keep the core route open.
**Lesson: a demotion expressed only in the capability registry binds only the layer that READS the
registry. Always produce a per-route table before calling a class demoted.**

**Corollary the slice created:** the row names `scanner/checks.py:ContentSignatureCheck` in
`implementation_refs` while declaring `fact_capable:false`, and that exact implementation mints at rate 1.0.
Before: registry true / scan mints = consistent and unsound. After: registry false / scan mints =
INCONSISTENT and still unsound. A partial demotion can make the honesty ledger worse.

## THE UNICODE OFF-BY-N (new class, worth remembering)
`runtime_redrive._secret_assignment` did `i = low_hay.find(low_needle)` — an index into the LOWERCASED
body — then sliced `rest = hay[j:]` out of the ORIGINAL body. **`str.lower()` is not length-preserving.**
Exactly ONE codepoint in all of Unicode does this: **U+0130 `İ` (Turkish capital dotted I)** — `'İ'.lower()`
is 2 characters (verified by enumerating all 0x110000 codepoints). Any `İ` before the signature
desynchronises the offset and the extractor returns an arbitrary window.
A benign `.env.example` containing NO credential —
```
#İİİİİİİİ dagitim notlari
DB_PASSWORD=
notes:DB_PASSWORD=documented_here
```
— minted a **signed** `exposure` FACT with `secret_value='B_PASSWORD=documented_here'` that
**re-verified offline, rc=0, 1/1**. Also, on the ACCEPT branch, a real secret behind an `İ` came back
TRUNCATED (`'r0dKey-Real-Value-9x'`).
**Rules: never index one string and slice another; and a JSON predicate AST that checks only the
SELF-CONSISTENCY of (assignment, value, body) does NOT re-derive the extraction — so a mis-offset window
that happens to contain the signature satisfies every clause.** The docstring claiming the AST "is what
makes a substituted value fail the conjunction" was false.

## FAIL-CLOSED SETS MUST BE DEFINED BY WHAT WAS OBSERVED, NOT BY A STATUS ALLOWLIST
`_NON_ANSWER_STATUSES = {0,408,425,429} + >=500` routed everything else to REFUTED. Measured gaps against
a GENUINE LFI: **403 — the canonical WAF block — was not in the set** (nor 401/302/451/404/empty-200), and
an **undecodable body** (brotli, or two concatenated gzip members) also read as "observed, signature
absent". The correct rule is the inverse: anything that is not a 2xx **with a decoded body** is
could-not-observe. Note the only negative control sat on 429 — the one status inside the set.
