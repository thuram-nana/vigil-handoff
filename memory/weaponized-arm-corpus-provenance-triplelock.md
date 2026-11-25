---
name: weaponized-arm-corpus-provenance-triplelock
description: "The reusable soundness pattern that lets a weaponized/credential re-drive arm mint a FACT without letting a model-derived value manufacture one — a triple-lock on corpus/attestation provenance. Apply to every M-11 offense arm (credential done; exploit unit next)."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-04T12:17:06.541Z
---

When a VIGIL offense re-drive arm (M-11) consumes operator-planted material — a credential corpus, an
exploit attestation, a login spec — to mint a FACT, the load-bearing soundness risk is the
[[runtime-redrive-live-false-fact]] class: a model/LLM-derived value sourcing that material and reaching a
fired signed record. The CREDENTIAL arm (M-11.2, PR #894, `credential_acceptance_redrive` in
integration/vigil_integration/live/runtime_redrive.py) closed it with a **TRIPLE-LOCK**, red-pen-verified
PASS. Reuse this exact shape for the exploit unit (sqlmap/metasploit) and any future weaponized arm:

1. **Router emits NO sensitive material.** `engine._redrive_spec` is a `@staticmethod` over LLM-derived
   inputs; for the offense class it returns ONLY `{kind, url, bug_class}` — never the corpus/attestation/
   login_spec. Nothing downstream mutates the spec.
2. **Seam REQUIRES a trusted channel.** The `_live_<class>_redrive_fact` seam returns LEAD (None) unless the
   trusted operator material (corpus + policy + attestation + spec) is ALL present — which the governed-engage/
   LLM path never carries, so it is inert fail-closed there. The material reaches the arm ONLY via explicit
   params or a trusted planted-corpus spec.
3. **LEAD-fallback OMITS the class.** `_redrive_branch_for("<class>")` returns None, so even a crafted LLM
   `oracle_context` carrying a *firing* record cannot reach `admit()`; and `provenance="llm"` demotes anyway
   (`certify_admitted` never yields is_fact for llm provenance).

Net: the only target-controlled values in a fired record are the response legs; the operator-attested
discriminator (e.g. `authenticated_signal`, ≥12 chars/≥2.5 bits-per-char, present only in the authenticated
view) cannot appear in the weak leg yet be absent from the no-session baseline without GENUINE achieved
access. Plus: gate stays byte-identical by using a DEDICATED branch/class set (`CREDENTIAL_REDRIVE_CLASSES`/
`_CREDENTIAL_BRANCH`), NOT the single-capture `_BRANCH_FOR`; `_ALL_ORACLES` frozen at 15; secret lives ONLY in
the transient login body (never in the retained record/context/certificate/ids); FATAL-2 (framework imports
function-local). Arming = owner-signed M-11.1 attestation + mandatory planted TEST corpus (operator-arms, NOT
the gate-declined one-click — see [[vigil-investor-eval-lab-program]]). [[claim-discipline-meet-up]]

OOB AUTONOMOUS FEEDER (M-11, PR #899, MERGED) reused this triple-lock for autonomous OOB-callback FACT
minting (ssrf/xxe/rce/cmdi/deser/oob_sqli) via `_with_trusted_oob_material` + `_live_oob_redrive_fact` in
wiring.py. The Wave-3 red-pen BLOCKed it on a **mis-keyed Lock #1**, a new same-shape instance of
[[runtime-redrive-live-false-fact]]: the trusted-material STRIP keyed on `redrive.kind == "oob"` while the
CONSUMER gated on `bug_class ∈ OOB_REDRIVE_CLASSES` — so a `{kind:"web", bug_class:"ssrf", oob_material:
<self-consistent attacker bundle>}` spec bypassed the strip and could mint from producer material (Lock #3's
`_oob_material_verifier` catches only a FORGED collector keypair≠pin, never a self-consistent bundle → the
triple-lock collapsed to the single router lock). RULE: **a sanitize/strip predicate MUST key on the SAME
surface the consumer reads** — here, make the strip UNCONDITIONAL (every spec) and scope the trusted
re-injection to the consumer's exact predicate (a shared `_spec_is_oob_consumable` mirroring the consumer's
`redrive.bug_class OR info.bug_class` resolution). Latent (engine._redrive_spec never attaches oob_material, so
not model-reachable) but still a BLOCK: a defense-in-depth collapse + a docstring overclaim in an autonomous-
minting capability is non-negotiable. Fixed + re-red-pen PASS.

BUILD GOTCHA (tripped #894 on merge, not caught by build or red-pen): an offense-arm's honest
docstrings naturally use the debt phrase "not wired" (e.g. "hydra/john stay EXTERNAL LEAD PROPOSERS
(not wired here)"). The honesty-ledger CI guard `docs/tests/test_limitation_inventory.py::
test_no_unregistered_honest_limit_prose` (run in the "the briefing explains every agent and capability"
check) REFUSES any `_DEBT_PHRASES` hit (does not yet / not\s+wired / is a stub / deploy-gated / …) in
non-test product code unless it's a `VIGIL-LIMIT:` marker OR a registered `honest_limit_sites` entry in
`docs/limitations/inventory.json`. Fix: register each as kind `non-debt` with a `snippet` (case-insensitive
substring of the line) + a `note` saying why it's deliberate doctrine, not debt. RUN THIS GUARD LOCALLY
for the exploit unit (sqlmap/metasploit) build — the gate tests + two-env + claim-discipline do NOT cover it.
