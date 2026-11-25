---
name: vigil-legal-governance-program
description: "Legal/Compliance/Billing/Authorization/Governance UI + backend-enforcement program for VIGIL (SIGIL SARL) — audit-first, wire don't rebuild"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T15:10:04.656Z
---

Owner spec (2026-11-19): build the complete **Legal, Compliance, Billing, Authorization, Governance** UI + backend enforcement for VIGIL. Real persistence, backend validation, audit logging — NO mockups/placeholders. Structure: a 7-tab `/settings/*` surface (General, Team&Seats, Billing/Subscriptions/Credits, Authorizations/RoE, AI&HITL, Privacy/Data-lifecycle/Cookies, Legal agreements/Policies), 3 in-app flows (signup consent, pre-run confirmation modal, active-run dashboard with emergency stop + action-specific approval + out-of-scope banner), and 8 non-negotiable backend acceptance rules.

**Ground facts (load-bearing):**
- VIGIL is owned by **SIGIL SARL** — the seller/contracting entity named on checkout + invoices. Seat = **USD 25/seat/month + taxes**. Payment provider = **Stripe** (see [[vigil-payment-provider-stripe]]).
- **WIRE, don't rebuild.** Most of the 8 backend rules already have real enforcement: scope/exclusions/expired-auth-block/out-of-scope-refusal/kill-switch/egress-floor → the conjunctive gate chain + signed charter/authority (`ui/target_authorization.py`, `authority/gate.py`, `vigil_core/gate.py`); AI-can't-authorize → `verify/firewall.py` demote-only + SIGIL owner-Ed25519 governor + per-action A3 + m-of-n; idempotent billing/no-double-charge → GrantMeter + Stripe-webhook fulfilment (W2-W6, A1, G, J2). The build wires the new UI to these; a weaker parallel enforcement path would VIOLATE the spec ("never rely solely on frontend checks").
- **Overlaps the in-flight consumer A–L wave** ([[vigil-consumer-al-program]]): billing↔G/W-series/J2, seats/SSO↔H, authorizations↔authority. Billing/Authorizations/Seats slices are scheduled AFTER that wave merges to avoid worktree collisions. Non-overlapping slices (settings shell+General, Privacy/Cookies/Data-rights, Legal archive+policy pages, AI/HITL) build first.
- **Honest boundary:** authoritative legal TEXT (Terms, DPA, privacy, subprocessors, AUP, retention, AI policy) + SIGIL SARL registration/tax details are **owner/counsel input**, not to be fabricated. Build ships the capture + immutable versioned archive + enforcement + page scaffolding; policy text is owner_input.

**Status:** audit-first. Audit workflow **wf_2810edfc-c4a** (7 domain maps → Opus synthesis) produces the private gap matrix + ordered slice plan (each slice = 1 PR, default-OFF VIGIL_* flag, ~10 acceptance tests mapped to the 8 rules, build-now vs after-consumer-wave). Every route-adding slice must honor [[vigil-ui-wiring-matrix-guard]]. Not yet built.

**S1 status 2026-11-19:** fix (guide coverage w/ enforced when-gating, Gate-5 census, edge-POST flag-off test) done and
verified (40 files / 1042 passed) but UNCOMMITTED in vigil-wt-legalgov-s1 (plan mode). Panel found MEDIUM data-integrity
defects to fix next: write_prefs caps by code points but JSON-escaping inflates bytes and reads >8KB as DEFAULTS ->
an accepted save can wipe the profile; a non-401 read error renders an empty form whose Save blanks stored fields;
backup excludes the customer-entered profile under a "regenerates" rationale.

**2026-11-19 parallel build:** S2-S5 stacked on S1 @ 86cb9a09 (wf_7a4713b7-668; vigil-wt-legalgov-s2..s5). S6-S12
(wf_ad7c1018-0b0): S6 off main, S9 on prJ2, S10 on S1, S11 on prG + S1, S12 on prC + S1, S7/S8 on S6 (S8 + prH).
4-lens Opus panels (attack/integrity/regression/mutation), ≤2 fix rounds; S1 defect classes baked into every prompt.
Owner inputs stay EMPTY + fail-closed enabling. S1's own fixes land via fix-rounds wf_b5d3e043-e60, then merge into
the stacked branches.
