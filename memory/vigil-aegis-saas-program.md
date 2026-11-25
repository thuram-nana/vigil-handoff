---
name: vigil-aegis-saas-program
description: AEGIS SaaS production program — APPROVED 2026-11-19; owner decisions E1-E3; plan location; ~72 slices / 24 waves; owner-gated rollout R1-R4
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T21:38:06.749Z
---

**APPROVED 2026-11-19** (owner exited plan mode on it). Plan: `~/.claude/plans/parsed-popping-lemur.md` = 6.5K executive
summary + the full 118K integrated plan (§0–§16, §16 = resolution of ~96 critic objections) + an APEX appendix (NOT part
of the approval). Source spec: owner's AEGIS_FULL_PRODUCTION_IMPROVEMENT.docx (28 sections + 10-question acceptance
standard). Produced by read-only workflow wf_44cb94ec-454 (journal holds the 12 audits, 4 SaaS lanes, critiques).

**Owner decisions (DECIDED):**
- **E1** full end-to-end PRODUCTION system (not MVP): every engine SIGIL operates, every supported path, isolation +
  fairness, ops, self-security, the production rollout procedure itself. Prove each capability on a staging TEST
  TENANT first (TT-A; TT-B = cross-tenant control); production go-live only at explicit owner gates (R1–R4).
  **Supersedes the 2026-11-16 "no server/VPS work" directive FOR THIS PROGRAM ONLY** (Contabo VPS still out).
- **E2** ONE unified VIGIL platform on AWS (one control plane/tenant model/RBAC/billing/UI/deployment); AEGIS = VIGIL's
  defensive SECTION, never a separate product/account/deployment. Isolation is INTERNAL by zones (offensive sandbox
  zone Z4 vs privileged defensive enforcement); no Kubernetes without measured need.
- **E3** "rent the pipes, own the intelligence": CloudFront + Shield edge/DDoS; Path 1b (customer's existing
  Cloudflare/CDN/WAF) brought forward; lead with exploit-proven virtual patching + signed offline-verifiable outcomes;
  benchmark vs generic-WAF before any "rivals Cloudflare" claim; own anycast deferred.
- Engines run on SIGIL infra = service use; licence review separates service use vs distribution (per engine, deps, rules).

**Ground truth at approval:** AEGIS = one partially built inline gateway per (org, host), hosted nowhere; no third-party
engine, no TLS, no edge/transport/health/incident/verification; UI overclaims. Top defects to fix FIRST: N-A (shared
writable /var/lib/aegis-saas holding trust root; /proc environ leaks AEGIS_GW_SECRET), replayable/undeliverable
revocation (LWW, no epoch), IP-as-actor, tenant-declared-origin confused deputy (owner key must NEVER sign a raw
tenant-declared value), defensive image still imports offensive modules.

**Program:** ~72 slices AS-0…AS-34 in 24 dependency-derived waves (W0–W23); hot-set chains (GW, UI, SPOOL, FLEET, SC,
INFRA, GP, VERIFY, RBAC) serialize shared surfaces. Every AEGIS merge needs a RECORDED HUMAN APPROVAL — agents never
self-merge AEGIS slices. Open decisions D2r–D28 in plan §14.2 (none blocks Wave 0). Wave 0 = AS-0 (record E1 in
HANDOFF §6, threat model, protection contract, dependency-graph test). Related: [[vigil-consumer-al-program]],
[[vigil-investigation-graph-program]], [[vigil-ui-wiring-matrix-guard]], [[vigil-customer-kit-mirror-guard]].

**Execution 2026-11-19:** AS-0 building (wf_6ae5cc01-70a, vigil-wt-aegis-as0). Wave 1 (AS-25, 1a, 2a, 7a, 27a, 3a,
4a*, 2l*) building in parallel off main b6d65028 (wf_dd3ab762-a06; worktrees vigil-wt-aegis-<id>), each through
RED-PEN + six lenses (+crypto for 25/2a, +gate-marshal for 1a), ≤2 fix rounds. Built on ASSUMED plan defaults pending
owner confirmation: D13 (forbidden-term list, unflagged copy fixes), D33 (TT binding over verification_manifest, staging
TT evidence key, empty committed trust root), D35 (lease 1 h / renew 10 min / re-proof 24 h, 3 fails ≥1 h, R16),
D16 (one v2 record w/ subject; entitlement only via release), D30 (separate release root; custody owner's). Merge order:
AS-0 first (owner approval), X-CT 4a/2l after #1202.
