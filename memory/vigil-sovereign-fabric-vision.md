---
name: vigil-sovereign-fabric-vision
description: "Operator's north-star architecture for VIGIL — a sovereign cyber-defense fabric; Claude advises, the oracle+governance are the authority"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-21T10:32:18.302Z
---

Operator's strategic vision (2026-09-21) for where VIGIL evolves — the **Sovereign Cyber Defense Fabric**.
Treat this as the north star that later work is sequenced toward; it is a program-of-programs, NOT one build.

**LOAD-BEARING INVARIANT (already VIGIL's spine): Claude/CVP is NEVER the security authority.** The deterministic
ORACLE + the governance/authority layer are the authority; the LLM only advises/hypothesizes/plans. This already
maps to: oracle-sole-authority, `_ALL_ORACLES` frozen, FATAL-2 two-env boundary, the veracity firewall (re-execute,
demote-only), metacognition/multi-critic (advise not promote). Keep every extension inside this invariant.

**The layers the operator drew (mapped to CURRENT VIGIL):**
- National command / governance-authority (RBAC/ABAC/warrants/scope/approvals/expiry/dual-control/legal-basis/audit)
  → EXISTS partially: the authenticated governor, EngagementAuthority + charter scope, m-of-n destruction, per-action
  owner-signed approvals, RBAC/PoP/MFA/OIDC. Extend toward warrants/mandates + mission/expiry + dual control.
- VIGIL Intelligence (knowledge graph ← oracles ← evidence; Claude reasoning ↓ hypotheses→investigation→plans)
  → EXISTS: Kùzu/Bayesian world-model graph, oracles, signed evidence spine, SCE/entity-resolution, metacognition.
- Production defense ↔ validation/range planes → verification oracle → immutable evidence → EXISTS: AEGIS
  (defensive dual) + engage; the MERIDIAN range; the signed spine.
- Sentinels: extend the current agent mesh (ARCHIVIST/SENTINEL/STEWARD/ENVOY + phase agents) toward a DOMAIN
  hierarchy (RECON/IDENTITY/ENDPOINT/NETWORK/CLOUD/APP/CODE/DATA/EMAIL/WEB/DNS/PKI/OT/MOBILE/AI/DECEPTION/MALWARE/
  THREAT-INTEL/INSIDER/SUPPLY-CHAIN/RECOVERY/FORENSIC) under a CHIEF SENTINEL → INCIDENT COMMANDER → oracles+reasoner
  → policy engine → authorization gate → actuator plane.
- FORGE: EXISTS (FORGE RAMPART); extend toward twin-builder/threat-model/code-lab/detection-validation/ranges/
  purple-team/CVP-research-gateway/detection-generator/regression/benchmark. "Production VIGIL detects; FORGE proves
  the protections actually work" — keep that separation.

**GENUINELY NEW / high-leverage pieces the operator wants:**
1. **Sovereign AI Gateway** — a mandatory chokepoint between VIGIL and any external model (Anthropic/local): classify
   · minimize · secrets-filter · case-authorize · model-route · prompt+response provenance · token/rate · retention
   policy · output verification · full audit. Rationale: external-model data crosses a trust boundary (30-day
   retention/monitoring) so the provider is never in the institution's trust root. EXTENDS the existing LLM-egress
   gate + redaction + SCHOLAR SSRF gate; formalize as a first-class component. HIGH VALUE + buildable + reinforces
   "Claude ≠ authority". Likely the best NEXT strategic build.
2. **Separate-trust-domain OOB/recovery plane** — VIGIL's emergency-control env is a DISTINCT trust domain from the
   production network (one-way telemetry across a trust boundary; independent IAM/PAM/bastion; HSM/PKI signing keys;
   OOB mgmt network; recovery plane w/ golden images). Must keep functioning even if AD is compromised, SSO down,
   DNS poisoned, prod ransomware-encrypted, VPN creds leaked, cloud-admin identities compromised, SOC workstations
   compromised. This is the mature endpoint of the OOB work now in flight.
3. **Cyber Digital Twin** — a continuously-updated abstract twin (topology/identities/trust/apps/deps/vulns/controls/
   critical-services/propagation-paths/recovery-deps). Reason blast-radius BEFORE a consequential containment ("if I
   isolate this IdP, what gov services disappear?"). Maps onto the Bayesian world-model graph, extended. This is the
   RIGHT safety mechanism for production actuators + aligns with the ask-before-destruction doctrine.
4. **Continuous control verification** — replace "firewall enabled" with CONTROL-REQUIREMENT / EXPECTED-STATE /
   CONFIG / OBSERVATION / TELEMETRY / POLICY / LAST-VERIFIED / CRYPTOGRAPHIC-PROOF. Extends the Certificate-of-Non-
   Exploitability / posture-FACT work into continuous, signed control-verification.

**MAX-TARGET REFINEMENTS (2026-09-21, "highest credible" end-state):** 11 sovereign planes (Authority · OOB-trust ·
Sense · Twin · Intel/Detect · AI-Reasoning · Oracle · Decision/Authz · Response · Evidence · Recovery) wrapped by
VIGIL GUARDIAN (protects VIGIL itself). 26-module product family (CORE/SENSE/GRAPH/TWIN/INTEL/HUNT/ORACLE/AI/IDENTITY/
ENDPOINT/NETWORK/CLOUD/CODE/SUPPLY/DATA/AI-DEFENSE/OT/DECEPTION/RESPOND/FORENSICS/RECOVER/OOB/FORGE/FEDERATE/COMMAND/
GUARDIAN). Standards envelope: NIST CSF 2.0 (Govern/Identify/Protect/Detect/Respond/Recover), Zero-Trust (no implicit
network-location trust), OCSF (normalize telemetry but KEEP raw→normalized→enriched→derived as 4 provenance-distinct
objects), MITRE D3FEND (import as machine-readable countermeasure graph, not report labels), SSDF (supply-chain).
**SAFETY DOCTRINE (all correct, reinforce — these keep an autonomous national defender from being a liability):**
capability≠authorization (an op doesn't exist until an AUTHORITY OBJECT signs it — institution/legal-basis/mission/
permitted+prohibited-targets/time-window/max-impact/approval/rollback/sig); a deterministic SAFETY KERNEL with NO
generative AI sits between reasoning and actuators; narrowly-TYPED capabilities (isolate_endpoint/revoke_session/
disable_identity/... — scoped·signed·short-lived·revocable·rate-limited·impact-bounded), never run_command; TWIN
blast-radius simulation BEFORE any consequential containment; N-of-M independent-oracle-witness quorum for critical
facts; multi-model reasoning but model-agreement≠proof (oracle still decides); explicit uncertainty as first-class
states (confirmed/probable/possible/unverified/contradicted/unknown/telemetry-unavailable/...); autonomy levels
V0-observe·V1-advise·V2-prepare·V3-bounded-reversible·V4-emergency-predefined — **NEVER V5 (unrestricted)**; graceful
degradation modes (NORMAL→DEGRADED→ISOLATED→OOB-ONLY→EMERGENCY→RECOVERY); proof-carrying response (every action ships
a signed why/evidence/authority/impact/reversibility/result bundle); formal invariants (execute(A)⇒authorized∧scope∧
authn∧capability∧unexpired; AI_output≠evidence; cross-institution resource-disjointness unless federated). Also:
hardware-rooted trust (TPM/measured-boot/attestation/HSM), workload identity for every service/agent/oracle, crypto-
agility+PQC-readiness, confidential computing for sensitive workloads, privacy-preserving FEDERATION (share
indicators/TTPs/proofs, never centralize raw telemetry), clean-room reconstruction from a known trust root. Don't
recreate mature EDR/packet-capture/scanners/HSM-firmware — treat them as interchangeable sensors/actuators; VIGIL owns
authority·knowledge·verification·reasoning·orchestration·proof·sovereignty·federation·resilience.

**HONEST BOUNDARIES (don't let the vision outrun the deterministic layer):** national-scale + multi-tenant is a
large leap from single-operator-per-engagement; production actuators that isolate a national IdP are extreme-
consequence and REQUIRE the twin + the gates + dual-control (never Claude-authorized); OT/national-fabric pieces are
aspirational. Build each piece oracle-proven + red-penned + operator-gated; never claim the fabric before the
deterministic layer enforces it. See [[vigil-fact-coverage-program]] for the active OOB/FACT work this extends.
