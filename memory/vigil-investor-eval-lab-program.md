---
name: vigil-investor-eval-lab-program
description: "Design program for a 17-org investor evaluation lab of VIGIL — full capability, authorized internet targets behind a signed attestation, Anthropic model, per-org isolation; recommended substrate = orchestrated microVM sandboxes, not 17 hand-built VMs"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-04T12:20:05.255Z
---

Operator wants to let **17 prospective investor orgs** independently test VIGIL's FULL capability
(every AEGIS + every HexStrike + codescan + re-runnable PoC + LEAD/FACT), browser-only (no on-prem),
with HARD isolation (nothing mixes), fully tested, using the **ANTHROPIC API** as the reasoning model
(not local — a per-eval choice; the model-sovereignty ladder swaps to local for the real gov deploy).

Operator-confirmed decisions (this session, 2026-09-30):
- Internet targets ALLOWED (primary mode), gated by a **signed per-target attestation** ("this target is
  mine") — **signed-attestation-only** (NO domain-ownership proof, NO per-target operator approval; self-serve).
  Team monitors. (AskUserQuestion answers: guardrail=Signed attestation+audit; strictness=Signed attestation only.)
- FULL capability, no feature-gating — incl. host-code-exec features (local terminal / Strix shell /
  source-review-that-runs). Contained by per-org isolation.
- ACCESS MODEL (operator-confirmed 2026-10-01): **per-org OIDC/SSO at the operator-run TLS reverse proxy**
  (the ONLY public listener). Proxy does per-org OIDC → forwards to that org's loopback uiproxy (:8770)
  carrying the VM session token SERVER-SIDE; the investor browser NEVER holds/pastes a raw bearer. Revoke =
  disable that org's SSO (+ fleet kill-switch). NOT the per-user `sigil accounts create` bearer model, NOT a
  shared session token. Bearer tokens in VIGIL: `vigil up`/cockpit mints a fresh per-VM session token (rotates
  on restart, sent as X-SIGIL-Token, no longer embedded in page — Claim 6); `sigil accounts create <u> <role>`
  mints owner-signed, role-scoped, TTL'd, independently-revocable per-user bearers (the alternative we did NOT pick).

Non-negotiable I held (operator did not select "keep egress floor" but it enforces THEIR OWN "nothing
mixes up" + host safety): the egress floor stays — but see [[vigil-engine-egress-floor-residual]]: it is
CONFIRMED **not wired on the primary live ENGINE paths**, so the app floor CANNOT be relied on. ⇒ enforce
egress at the **sandbox/VM firewall** + **disable IMDS / require IMDSv2 hop-limit-1**; also fix the product.

Architecture (grounded via 2 workflows + red-pen, all 4 lenses WEAK on the naive VM plan):
- Isolation is FORCED by code: `common/tenant.py::assert_tenant_fleet_isolated` fail-closes unless a
  DEDICATED `CRUCIBLE_ROOT` per tenant; the shard only isolates spine+memory, not targets/evidence/authority/
  killswitch ⇒ **one root = one isolated environment per org** (single-tenant doctrine). A shared multi-tenant
  plane is OUT (doctrine + full-offense safety). `CRUCIBLE_FLEET` must stay OFF (else cross-deploy data pool).
- **RECOMMENDED SUBSTRATE (operator asked for better/faster than 17 VMs): one control plane orchestrating
  per-org microVM sandboxes** (Firecracker/Kata), ephemeral on-demand, from one golden rootfs image. Own
  KERNEL per org (full offense+shell ⇒ shared-kernel container = breakout to all). Sub-second boot, snapshot
  reset, per-sandbox netns+firewall = where the floor actually lives. Pick #1 bare-metal+Firecracker (eval);
  #2 K8s+Kata RuntimeClass (standing platform); #3 managed (E2B/Modal/Fly, not for gov).
- Anthropic: per-sandbox API key (metered/capped/revocable) + `api.anthropic.com` in the allowlist above the
  floor. Pitch: "AI advises, the deterministic ORACLE proves every FACT, re-runnable offline" — a cloud model
  cannot mint a false FACT. Demo the LEAD-vs-FACT tiering + offline re-verify as the moat.
- Two modes: Mode 1 = bundled MERIDIAN range, no attestation, instant FACT demo; Mode 2 = internet target
  behind signed spine-written attestation.

HONEST CAPABILITY CATALOG (grounded on wave-d/batch @ f6b04a34, red-penned; workflow wr07ry4u0):
- Tier-1 re-runnable FACT TODAY: web error/boolean SQLi + XSS (loopback-proven, tamper-rejected); AEGIS 4
  detection classes + inline Gateway (403+cert, block only once AEGIS_RESPOND provisioned) + 12-oracle
  Detection Mirror; **HexStrike EXACTLY 9 tool-FACTs** (6 port-scan=SERVICE_REACHABILITY, sslscan=TLS,
  katana/gobuster=ACHIEVED_STATE) + 3 VIGIL-direct families (SBOM/version, web_redrive, dom_redrive) —
  NOT the ~18 in [[vigil-hexstrike-completion-program]] (that was the PLAN ceiling; today 9, up from 2);
  FACT-mint DEFAULT-OFF (VIGIL_FAMILY_REDRIVE + --brain); codescan 2 static-rule tiers (broken-crypto +
  insecure-flag-literal) but BRIDGE-ONLY (static_facts.confirm_code_region not wired to scan CLI/console).
- Anthropic wiring VERIFIED SOUND (red-pen): first-class (ANTHROPIC_API_KEY, default claude-opus-4-8);
  LLM is PROPOSER-ONLY, never gets raw target RESPONSE bytes, wrong guess stays a LEAD; oracle re-drives
  fresh bytes = sole authority ⇒ a cloud model CANNOT mint a false FACT. Per-VM: CRUCIBLE_SOVEREIGNTY_TIER=
  PERMISSIVE (or TRUSTED_CLOUD+CRUCIBLE_ANTHROPIC_ZDR=1) — Anthropic REFUSED at higher tiers; own key per VM;
  MUST open api.anthropic.com:443 at the VM firewall (VIGIL does NOT auto-allowlist it). Cost = throttle NOT
  a hard cap (bound via the org's own key limits). Digest sends target URL+objective+operator instructions
  to Anthropic (not raw response bytes) — a data-residency note. Flip to AIR_GAPPED ⇒ egress refused pre-SDK.
- DO NOT tell investors: "every tool mints a FACT" (9 do); "codescan-FACT via CLI" (bridge-only); "model
  call is tested" (no real-Anthropic test); "firewall on out of the box" (enforce silently downgrades w/o
  AEGIS_RESPOND). Lead with tiers + one-command offline re-verify + tamper-fails-closed (all code-enforced).

Net-new to build: microVM golden image (VIGIL 2 venvs + Rust kernel + Strix Docker-in-microVM — the one
packaging spike to validate) + thin control plane (provision/reset/kill/monitor); Mode-2 live attestation →
spine (today the ceremony REFUSES environment=live, twin/staging only, file-seam); central read-only fleet
monitor (none ships — tail each spine via Blackboard.replay) + fleet kill; wire the rate limiter (test-only).
Honest capability catalog pass = workflow wr07ry4u0. [[vigil-live-external-program]] [[vigil-meridian-range-program]]

FULL-OFFENSE eval mode (operator CONFIRMED 2026-10-02, "Build it; test-identity boundary" — plan Part 11 /
Lane M-11): for the agency-supervised investor eval, the offense/credential tools (sqlmap/metasploit/hydra/
john) MAY run REAL weaponized payloads — the keyless-offense/no-cred-theft block is a POSTURE, not a soundness
law, liftable under signed authorization. Design: opt-in per-engagement owner-signed `EvalOffenseAttestation`
(extends WS2 ceremony, spine-written, HARD precondition — refuse to arm without it) + §0.3 per-action approval
for each weaponizing/credential step; VIGIL captures the achieved state through its gated channel and the
ORACLE re-fires → oracle-minted, offline-re-verifiable FACT (the tool's bytes are NEVER the FACT — that one
rule never relaxes). Non-negotiable boundaries held even in full mode: (1) oracle soundness; (2) credential
tools on operator-planted TEST accounts / operator-supplied sample hashes ONLY — real users' live secrets
never harvested; (3) destruction behind m-of-n; (4) never-liftable egress floor + third-parties-out; (5)
correlatable, not evasive. Pitch value: agency watches the full kill-chain AND watches VIGIL independently
re-prove every step offline — that IS the moat.

FULL-OFFENSE AUTHORIZATION UX — RESOLVED 2026-10-03 to **operator-arms-via-M-11.1** (NOT one-click).
The self-serve ONE-CLICK relaxation was earlier chosen but a grounding pass found it was **DECLINED BY
THE SECURITY GATE and that decline is recorded verbatim in the committed repo** (`docs/capability-
matrix/hexstrike.json` hydra note ~L426 + john note ~L462: "a one-click self-serve relaxation of that
arming was NOT built (declined by the security gate)"). Surfaced the conflict to the operator; operator
chose **"Operator arms via M-11.1"**: build the weaponized CAPABILITY (11.2) gated behind the already-
approved OWNER-SIGNED M-11.1 `EvalOffenseAttestation` (merged #892), and for the SUPERVISED eval the
SIGIL team (physically present) arms each engagement via the owner-signed attestation while investors
DRIVE everything else and WATCH the tool exploit + VIGIL independently re-prove. The one-click stays
**NOT built** — do NOT re-open it without an explicit operator override of the security-gate decision.
Rationale the operator accepted: for a supervised eval the operator is right there, so owner-signed
arming costs ~nothing vs one-click, keeps the stronger authorization the gate wants, and matches the
real government deploy. The SAFETY floors never relax regardless: per-org isolation, never-liftable
egress floor + per-VM firewall (only the attested target), test-identities-only (no real-user secrets),
destruction-m-of-n, third-parties-out, correlatable-not-evasive. [[vigil-engine-egress-floor-residual]]

MERGED STATE (2026-10-04, main @ c27700e1) — **M-11 is CLOSED** and the fabric frontier (Gate-B
consolidation) is landing. Merged this session: credential arm (#892/#893/#894), OOB-callback unit (#896),
fleet monitor + kill-switch (#895), **M4.1 Sovereign AI Gateway** (#897, one mandatory LLM-egress chokepoint
both paths route through), **#898** = M4.2 pre-action blast-radius veto (default-off advisory gate conjunct) +
M4.5 standing-twin refresh verb/systemd + **M-11.1 distinct-evaluator** (refuses owner self-attestation — the
11.4 BLOCK that closed M-11), and **#899** = the OOB autonomous collector-feeder (governed engage mints a
signed OOB FACT autonomously; red-pen BLOCKed a mis-keyed strip, fixed + re-red-pen PASS — see
[[weaponized-arm-corpus-provenance-triplelock]]). nuclei substring-FACT stays DEFERRED (unsound — see
[[vigil-wave1-remaining-gaps-lessons]]). IN FLIGHT: **Wave 4** (M4.3 typed defensive capabilities, M4.4
oracle-witness quorum gate [default-off, demote-only], M2.1 saturate worklist/delta perf) building under the
same build→red-pen→sequential-merge cadence. REMAINING for the eval-lab itself: the microVM golden-image
packaging spike + thin control plane + the Mode-2 live-attestation ceremony unblock (WS2) + per-VM firewall /
IMDS-off deploy (WS3) — all operator-infra, not yet built.
