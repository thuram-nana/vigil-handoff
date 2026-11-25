# AEGIS SaaS Production Plan — for approval

> Produced by a **read-only, specialist multi-agent workflow (31 agents, 0 errors)**: 12 evidence-cited domain audits →
> 4 SaaS lane architects → chief-architect synthesis → 3 independent critics (RED-PEN, FORGEMASTER, completeness;
> ~96 objections) → 10 parallel section finalizers → integrator. **Every objection is resolved in §16.** Nothing was
> built, run, installed or changed. Path aliases used throughout are defined at the top of the full plan.

## Executive summary

**What this is.** The end-to-end, **production-grade** implementation plan for **AEGIS — the defensive section of the
one VIGIL platform** operated by SIGIL SARL — meeting the 28-section AEGIS production specification under your scope
correction (SIGIL-operated SaaS; no on-prem/sovereign packaging) and your decisions **E1–E3** (full plan §0.1):
- **E1** — the full production system, not an MVP: every engine SIGIL operates, every supported path, isolation and
  fairness, operations, self-security and the production rollout procedure; each capability proven end to end on a
  staging test tenant first; production go-live only at an explicit owner gate.
- **E2** — ONE unified VIGIL platform on AWS (one control plane, tenant model, RBAC, billing, interface, deployment);
  isolation is internal, by zones (offensive sandboxes vs privileged defensive enforcement); no Kubernetes without
  measured need.
- **E3** — "rent the pipes, own the intelligence": CloudFront + Shield as edge/DDoS; existing-CDN (Cloudflare-first)
  integration brought forward; exploit-proven virtual patching and signed, offline-re-verifiable outcomes as the lead
  differentiators; a benchmark vs a generic-WAF baseline before any comparative claim.

**Where AEGIS stands today (full plan §1).** One partially built path — an inline gateway per (org, host) using VIGIL's
own deterministic oracles — **hosted nowhere and far below the production bar.** No third-party engine, no TLS, no
edge, no report transport, no health signal, no incident entity, no verification or drift handling; customer-facing
text overstates protection (grant-only "Monitoring · Respond" badge, "Respond also blocks them" while on hold, dead
gateway indistinguishable from a quiet one). It also carries real defects the program fixes **first**: a critical
shared-writable deployment directory exposing the trust root and gateway secret (N-A), replayable / undeliverable
revocation, actor identity by raw peer IP, a tenant-declared-origin confused-deputy risk, and a "defensive" image that
still pulls in offensive modules.

**What the program delivers.**
- **Connection paths (§6), production-grade wherever a real observation point exists:** 1a AEGIS-managed gateway;
  1b the customer's existing Cloudflare/CDN/WAF (brought forward); 2 cloud and identity via authorized provider
  integrations; 3 runtime (customer-side sensor or existing EDR integration, stated plainly); 4 network (only with a
  real observation and enforcement path).
- **Engines (§7)** deployed and operated by SIGIL behind typed adapters that declare what they can and cannot observe
  or enforce, each with a hosted-service licence record (service use vs distribution; engine, dependencies and rules
  reviewed separately).
- **Tenant isolation and capacity fairness (§8)** across events, policies, credentials, incidents, engine config and
  enforcement, including noisy-neighbour protection that degrades one tenant honestly instead of all silently.
- **Verified outcomes (§13):** every intervention tracked through requested → applied → outcome-verified; the attack
  outcome and the legitimate-workflow outcome checked independently; signed evidence an auditor can re-verify offline.
- **SIGIL operations and the production rollout procedure itself (§10).**

**How it is built (§9, §11).** **~72 reviewable slices (AS-0 … AS-34) in 24 dependency-derived waves.** Slices in one
wave never share a hot file set; shared surfaces (gateway, UI, spool, fleet, supply chain, infra, gate path, RBAC) are
strictly serialized chains. Every slice is default-OFF and byte-identical when off, passes independent red-pen and
mutation verification, needs a **recorded human merge approval (no agent self-merges)**, and is **DONE only when test
tenant TT-A uses it end to end through VIGIL** with TT-B as the cross-tenant control. Security fixes and truthful UI
states come first, not last.

**Rollout (§10).** R1 Observe on path 1a → R1b Observe on 1b → **R2 Defend per (org, host) under an owner-signed
release** (the first release covers only VIGIL-confirmed classes) → R3 slow path + virtual patching → R4 paths 2/3/4.
Each stage is an explicit owner go-ahead; production CD is owner-dispatched; no comparative claim without a signed
benchmark scorecard.

**Decisions.** E1–E3 are decided. §14.2 lists the remaining open decisions (D2r–D28: region and account layout, Shield
Advanced, quotas/HA/RTO, transport, licences and customer terms via counsel, the path-2 identity provider, the runtime
provider, verifier keys, AI-consent policy, …), each with a recommendation and a verification step. **None blocks
Wave 0.** External prerequisites (§14.3) are the only valid "blocked" states.

**What approving this plan means.** Execution starts with **Wave 0 (AS-0: record E1, the threat model, the protection
contract and the dependency-graph test)** and proceeds wave by wave under the gates above. **Every production stage
(R1–R4) still requires your explicit go-ahead.** If you want the plan recorded without starting execution yet, say so
when you respond.

**Paused work that resumes on approval (independent of AEGIS, previously authorized).**
- **Merge train:** K #1197, L #1196, J2 #1201, I #1200 are green and ready; G #1199 (flag-off cache leak in the fold
  cache) and F-ff #1202 (CLI guard order after the main merge) need fixes; B #1198 hit a possible lease-reclaim race
  in main's own code.
- **Fix rounds from the 3-lens panels:** C #1186 (Gate-5 census registration — done, uncommitted); legal/gov S1
  (a save that can wipe the customer profile); H #1190 (SCIM identity linking, SAML replay window, untested guards);
  D-ff (unenforced issue allowlist, non-atomic ledger); E (stale retest overriding a newer verdict); investigation
  S1/S3 (trust anchor inside the tenant tree, TOCTOU, unsigned refutes).
- **The CI guard catalogue's last two stages** (completeness critic and the runnable pre-push gate).

---

# Full plan — integrated and critique-resolved

*Plan only, 2026-11-19. This plan integrates ten section finalizers and three critiques; §16 resolves every critique objection. All analysis was static and read-only. Nothing was built, run, installed or modified.*

*"AEGIS" here means the defensive section of the one VIGIL platform. It is never a separate product, account or deployment.*

**Baseline**

- VIGIL is at `cb5d20b5`. origin/main is at `dd2d9281`, 13 files ahead. Those are session-folder UI files; the only lane path among them is `C/rbac.py` (+6).
- Run `gh pr list --state open` before AS-0 closes to refresh in-flight PR state.

**Path aliases**

| Alias | Path |
|---|---|
| V | /home/kali/vigil-wt-guardsweep |
| E | V/engine/crucible/framework/v2 |
| A | E/aegis |
| S | V/apps/sigil/sigil |
| C | V/packages/core/vigil_core/vigil_core |
| I | V/integration/vigil_integration |
| IT | V/integration/tests |
| P | /home/kali/Pictures/PENTEST-main |
| M | /home/kali/.claude/projects/-home-kali-Pictures-PENTEST-main/memory |
| Spec | …/scratchpad/aegis-doc/aegis.md |

**Evidence tags**

- Untagged: re-read by a finalizer or by the integrator.
- [L]: cited by a lane, not re-read.
- [W]: confirmed by a generic web query.
- [U]: unverified; the verification step is named.
- "Tested" means the test file and the CI step that runs it were both read. Nothing was executed.

The integrator re-read these in this pass:

- `docs/tests/test_claims_registry.py:42-43,197-237`
- `tools/verification_manifest.py:122,138,174`
- `release.yml:1-21`
- `required-status-checks.txt` (15 checks; `strict=true`; `enforce_admins=false`)
- `HANDOFF-SESSION.md:82-98`
- `E/common/egress_pin.py:28-46,153-193`
- `E/agents/egress_guard.py:275-290`
- `C/shard_assignment_record.py:62-77`
- `S/governor/shard_assignment.py:36-46`
- `ui-wiring-matrix.json` (`generated_from`)
- `test_gate5_customer_e2e.py:85-108`
- `custui.py:628-642,684`
- `provision.py:2862-2880`
- `ci.yml:69-84,306-325`
- `customer_app/index.html:36-40`

---

## 0. Context and intent

**Program**

- AEGIS becomes the defensive section of the single VIGIL platform. SIGIL deploys and operates it on AWS.
- Customers use it only through VIGIL's existing interface:
  - `#/monitoring` (`app.js:162,715`)
  - Assets (ownership proofs)
  - Billing (plans)
- SIGIL deploys, operates, updates and monitors every engine.
- The owner's plan-mode instruction supersedes the spec's "then implement".

**Repository rules**

- There is no `AGENTS.md` (`find` returned nothing). Rules come from `HANDOFF-SESSION.md` §0 and FORGE. `V/engine/crucible/FORGE.md` and `P/FORGE.md` are byte-identical (`cmp`).
- `P/framework/v2/aegis` is input to reconcile, never a target: 6 files differ, 0 exist only in P.

### 0.1 Owner decisions (DECIDED)

**E1 — Full production system, not an MVP.**

- **Scope.** Every engine SIGIL operates. Every supported path: 1a managed gateway, 1b existing CDN/WAF, 2 cloud/identity, 3 runtime, 4 network (only where a real observation path exists). Isolation and fairness. Operations: monitoring, upgrade, rollback, capacity, on-call. Self-security. **The production rollout procedure.**
- **Sequence.** Build to production grade → prove end to end on an isolated test tenant in staging (TT-STG) → go live only at an owner gate (spec §27).
- **DONE.**
  - TT-A uses the capability end to end through VIGIL: the authenticated customer API plus the browser. TT-B is the cross-tenant control.
  - Acceptance includes operations, isolation, fairness, runbooks and failure behaviour.
  - An installed engine or adapter is never DONE.
- **"Blocked"** is used only for an external prerequisite with a reproducible verification step. Nothing is left unbuilt as "demand-driven". Paths 2 and 3 have committed defaults.
- E1 supersedes the 2026-11-16 "no server/VPS work" directive (`HANDOFF:24-26,527-528`) **for this program only**. Contabo stays out of scope.

**E2 — One unified platform on AWS.**

- One control plane, tenant model, RBAC (`C/rbac.py`), billing (`S/governor/catalog.py`), customer interface and deployment.
- Isolation is internal, by zones (§4.1). Each zone has its own segment, IAM role, KMS key and egress.
- This carries the FATAL-2 two-plane boundary into hosting (`V/envs/README.md:3-9`, `IT/test_two_env_boundary.py`).
- The offensive sandbox zone (Z4) may also sit behind an account boundary. Payload-bearing probes originate only in Z4.
- No Kubernetes, streaming platform or graph DB without measured need (spec §20). Services are long-lived systemd units or containers.

**E3 — Rent the pipes, own the intelligence.**

- (a) AWS CloudFront + Shield are the edge and the volumetric-DDoS layer.
- (b) Path 1b comes forward, Cloudflare first. The customer keeps its CDN; AEGIS pushes verified, targeted rules.
- (c) Lead differentiators, for real tenants as well as TT:
  - exploit-proven virtual patching;
  - signed outcomes that re-verify offline.
- (d) A benchmark against a generic-WAF baseline. Any "rivals Cloudflare" claim must be earned by measurement. Own anycast is deferred.

**AWS facts the design relies on**

| Status | Fact |
|---|---|
| [W] | Multi-tenant distributions with "distribution tenants" |
| [W] | VPC origins (ALB/NLB/EC2), cross-account since 2025-11 |
| [W] | `CloudFront-Viewer-Address` via an origin request policy |
| [W] | The origin-facing prefix list covers **all** distributions, so it is not isolation |
| [W] | Origin mTLS is not supported with VPC origins |
| [W] | Shield Advanced costs US$3,000/month/org on a 1-year term |
| [U] | Shield Standard defaults, certificate non-exportability, quotas, per-tenant origin headers, managed-certificate validation, tenant failover — read the CloudFront/ACM/Shield docs before AS-6c |

### 0.2 How the scope correction reshapes the spec

| Change | Items |
|---|---|
| **Dropped** | §20 offline/sovereign profile; §20.5.1 and §20.5.5; §5.K.12; offline bundles; air-gap content age; RTBH; customer-installed gateway, middleware/SDK (`A/middleware.py`) and console `aegis_setup` (these would be distribution; the code stays byte-identical and is marked `not offered`) |
| **Re-mapped** | §20.5.2–20.5.4 update integrity and content age → AS-6a manifest digest refusal, AS-7b tampered-bundle refusal and stale-content handling |
| **Re-framed** | "Sovereign" = SIGIL's key-holding control-plane zone (FORGE reading is D29). Profiles = paths × zones. Fast path = local to the unit. T-23 = partition test. Licensing analysed as service use vs distribution |
| **Added** | Edge, fleet, transport, on-call, rollout procedure, quotas, DONE records. Threats: T-XO, compromised unit, co-host pivot, edge bypass, AWS-held TLS, undelivered revocation |

### 0.3 Reconciled or flagged

1. **E1 record.** AS-0 records E1 in HANDOFF §6 and the owner updates `M/vigil-owner-flip-delegation.md`. Until then no agent touches a server. The 2026-11-16 flip delegation does not cover AEGIS.
2. **Hosting memo.** `M/vigil-strix-shell-hosting-decision.md` puts the **control plane** on EKS (the draft misdescribed this). E2 supersedes that for this program. The memo's sandbox, egress and custody design stays the Z4 reference. `AWS-EC2-DEPLOY.md:12-15` is consistent with E2.
3. **Machine signatures are not owner authority.** The provisioner signs grants with the owner key unattended (`provision.py:2866-2873`), and the spine labels them A0/auto (`monitoring_authority.py:166-170`). Production enforcement therefore needs an owner-held release (AS-29, D30). UI and docs must say "provisioner-signed (owner key)".
4. **CD.** `deploy-ec2.yml:1-41` deploys every green `main`. For this program, CD goes to TT-STG only; production is owner-dispatched (D27).
5. **CodeQL.** `HANDOFF:1506` dismisses the gateway SSRF alert as "operator's chosen target". With tenant-declared origins that is false. The owner re-disposes it after AS-2d/2g (D37). Agents never dismiss alerts.
6. **In-flight programs** (§11.10). The X-CT gate applies, F-ff #1202 must land first, and D28 is widened.
7. **FORGE §1 (D29)** must be decided before AS-26, AS-1b, AS-2f, AS-2h and AS-6c. Proposed reading:
   - (a) Confirmation and PCF re-verification stay offline and keyless; KMS only wraps secrets at rest.
   - (b) The owner key is the root (m-of-n optional).
   - (c) Each new egress path gets a row in `SOVEREIGNTY-EGRESS-AUDIT.md`.
   - (d) A PROVER test enforces the boundary between proof paths and wall-clock paths.
   - (e) Byte-identity = the recorded `make gate` baseline.
   - (f) §26 governs order; FORGE §1/§3/§5 still bind.
8. **Governance.** There are no required reviews and `enforce_admins=false`; W12-3 is pending. The standing authorization at `HANDOFF:89` reads "merge once all required checks are green and the change is tested". `HANDOFF:83` means the public repo gets only fixed state.

### 0.4 Resolutions adopted

| # | Resolution |
|---|---|
| R1 | Edge path: Shield/CloudFront terminate viewer TLS → VPC origin → internal cell ALB → unit front (verifies the per-tenant edge secret) → unix socket → gateway → per-host egress broker → locked origin. No SIGIL SNI router, no ACME in units, no port-80 hop. AWS and the unit see plaintext (D11). |
| R2 | systemd units or containers on EC2 plus a reconciler. ECS/Fargate is measured as the alternative (H1, D7). The new orchestration code is stated, not hidden. |
| R3 | Transport is initiated from Z3 through a keyless relay identity (Z3R). |
| R4–R7 | One health schema; one `MonitoringAuthorityV2` with a `subject` discriminator; one coverage catalogue; one instance-id helper. |
| R8 | TT-A, TT-B, TT-CI (merge bar), TT-STG (DONE bar), TT-P (production canary). |
| R9 | Slices AS-0…AS-34; aliases in §9.6. |
| R10 | Origin binding uses a forward-path challenge **only**. Origin tuples are unique per org. A crypto lock is the only isolating lock; an IP allowlist is not isolation. |
| R11 | Enforcement renders only from an owner enforce release (AS-29, D30). |
| R12 | DONE evidence = the existing `vigil-verification-manifest/1` record + a signed binding, signed by the staging control plane. The claims registry is unchanged (§13.3). |
| R13 | Waves are derived mechanically (§9.5). OPS-1 (AS-26) precedes any TT-STG run. Break-glass (AS-27a) is the first gateway PR. |
| R14 | Merge at TT-CI with the flag off. "Demonstrated" or any R-gate needs a current DONE record. |
| R15 | Unit network: front ingress only from ALB subnets; gateway `PrivateNetwork=yes` behind a path unix socket that requires `SO_PEERCRED` = the front's uid; egress only via the broker; no DNS or IMDS; loopback is never edge identity. |
| R16 | No valid authority → uninspected pass-through, shown visibly. The Defend 503 policy applies only while a grant is active. |

---

## 1. Executive verdict

**AEGIS is one partially built path. It runs nowhere hosted and is far below the E1 bar.**

**What exists:**

- An inline gateway (`A/gateway.py`, 821 lines), one per (org, host). It re-authorizes every request against an owner-key grant (`A/monitoring_authority_check.py:113-169`).
- Deterministic oracles: 6 parse-proof classes plus a honeypot (`A/inspect.py:9-21`), response oracles, and belief-driven 429s.
- HMAC-sealed, metadata-only reports (`A/monitoring_report.py:56-80`), and a fail-closed drain (`S/inbound/monitoring_spool.py:82-124`).
- The `#/monitoring` screen and billing gates (`provision.py:2815-2835`).
- Unwired assets to reuse (§2.E): the Detection Mirror, `notify_channels`, `connected_repos`, the retest seam, offline certificate re-verification, and the Domain 7/8 oracles (offense tree; see AS-28).

**How it is proven:**

- Everything is default-OFF. The flags are on HOLD (`aegis-saas-flag-hold.md:3`).
- AE-7 (`IT/test_aegis_saas_acceptance.py`, 8 tests, `ci.yml:578`) runs both tenants in **one process and one uid** with the trust root monkeypatched (`:101`).

**What is missing:**

- **Third-party engines.** `content_gate.py:72` is a CrowdStrike `falcon` regex. `cdn_waf_detection.py:47-48` holds offensive WAF fingerprints. Falco appears only in a vendor README.
- **TLS** (`gateway.py:805`).
- **Hosted plumbing.** No edge, transport (scp at `onboarding.md:38`, `:59`), health signal, drain unit, incident entity, `legal_hold` (0 hits), asset groups, ledger, verification, drift handling or rollout procedure. Upgrades *stop* units (`RELEASE-UPGRADE-ROLLBACK.md:197-199`).
- **A defense-only build.** `inspect.py:31-33` imports `framework.v2.verify`, which eagerly imports offensive modules (`E/verify/__init__.py:32-139`). The image copies the whole tree unhashed (`A/Dockerfile:44-46`) and is never built in CI.

**Path status:** 1a partial (in-process); 1b, 2 (stubs `I/detection/telemetry.py:1-17`), 3 and 4 unsupported.

**E3 status:**

- No findings→defence link.
- A retest that hits SIGIL's own 403 could read FIXED (`provision.py:2733-2735`).
- SaaS keeps only `certificate_sha256`, while the 403 promises an offline re-run (`gateway.py:491-512`).
- No benchmark.

**Honesty gaps:**

| Gap | Where |
|---|---|
| Grant-only badge | `app.js:1834` |
| "Respond also blocks them." while on HOLD | `app.js:1814`; `guide.js:242,248` |
| A withheld response is labelled "Blocked" | `:1744` |
| Dead looks the same as quiet | `:1921` |
| Publish failure revokes the prior grant but the UI says "not changed" | `provision.py:2871-2878`; `app.js:1914` |
| "near-zero false positives" | `DEPLOYMENT.md:5-7` |
| "owner-signed" for a machine-minted grant | `provision.py:2840-2846` |

**Top defects:**

| # | Sev | Defect | Fix |
|---|---|---|---|
| 1 | Crit | **N-A.** Every unit runs as `User=aegis` with `ReadWritePaths=/var/lib/aegis-saas`, which holds the spools, publish dir and trust root. No `ProtectProc`, so `/proc/*/environ` leaks `AEGIS_GW_SECRET`. | AS-2j, 2h, 2e |
| 2 | High | Replay and undelivered revocation. v1 has no epoch; revoke is an unlink; window is 30 d (`config.py:275`), max ~1 y (`monitoring_authority.py:51`). | AS-2a/2c/2f, 3d |
| 3 | High | Actor collapse: actor = TCP peer (`gateway.py:370-373`; XFF reset at `:551`). | AS-5b |
| 4 | High→Crit | Origin SSRF and T-XO. Unsigned `--upstream`; only scheme and netloc checked (`:120-122`); no resolved-IP check (`:537-579`). | AS-2d, 2g |
| 5 | High | **N-D**: a kill-switch exception reads as "not tripped" (`:210-215`). **N-B**: compose is read-only with no `CRUCIBLE_ROOT`. | AS-27a |
| 6 | High | Exhaustion: unbounded threads; 10 MiB + 25 MiB joined (`:87-89,573-577`); no `MemoryMax`; `RestartSec=5`. | AS-2j, 5a |
| 7 | Med | Global fold poisoning (`monitoring_spool.py:247-251`); unvalidated fields (`:119-124`). | AS-3a |
| 8 | Med | **N-C**: raw IPs (`cli.py:114-133`), full verdict in journald (`:285-289`), `_DEMO_SECRET`. | AS-2e |
| 9 | High | Offense code in the defense runtime. | AS-28 |
| 10 | High | Machine-made authority. `A0/auto` contradicts A3 `grant` (`C/warden_tiers.py:41-55`). | AS-2f, 29 |
| 11 | Med | UI, doc and 403 overclaims. | AS-4a, 27a, 4b |

**Bottom line:**

- The authority wall, keyless verify, metadata seam, fail-closed drain and oracles are sound.
- The production system E1 requires does not exist.
- Isolation, revocation delivery, origin binding, attribution, break-glass, fail-open bounds and truthful state must land before any second tenant or any edge (spec §26).

---

## 2. Capability matrix

**Format.** `Row [spec line] | Status | Today → Change (slice · decision)`.

| Label | Meaning |
|---|---|
| IMP | Implemented and tested (none yet "demonstrated") |
| UNT | Implemented, untested |
| PART | Partial |
| STUB | Stubbed |
| DEAD | Present but unwired |
| INC | Security-inconsistent |
| MISS | Missing |
| UNS | Unsupported |
| DROP | Dropped by owner scope |
| DEC | Settled by E1–E3 |

**Publication.** Rows describing open gaps stay private (`HANDOFF:83`). AS-0 commits the public subset plus `aegis-requirement-trace.json` (spec line → row → slice → tests), which fails on any unmapped bullet.

| Row | St | Today → Change |
|---|---|---|
| **§1** 1.1 observe [L19] | PART | 1a only, in-process → hosted 1a (AS-2*/3*/5*/6*, 26); 1b (18a/b); 2 (17b/c); 3 (10a–c); 4 (20) |
| 1.2 complementary methods [L21] | PART | Oracles + belief; Mirror and SPRT unwired → Mirror (31), CRS leads (7b), Sigma (17a), runtime (10a/b) |
| 1.3 observation / policy violation / uncertain [L23] | PART | UI conflates (`app.js:1744`) → closed vocab (4b), declared policies (2k→31), `outcome_class` (5d) |
| 1.4 incidents [L25] | MISS | grep `Incident`/`incident_id`: none → AS-11a sole owner (D40) |
| 1.5–1.7 act / verify / independent outcome [L27-31] | PART/MISS/DEAD | Report precedes the 403 (`:731/743`); RAMPART test-only → receipts (5d), ledger (12), AS-14, RAMPART into the locker (11b) |
| 1.8 legit workflows [L33] | MISS | → declared workflows (2k) verified by AS-14 |
| 1.9 drift / missing telemetry [L35] | MISS | → AS-3b/3d, AS-15 |
| 1.10 explain coverage [L37] | INC | → AS-4a (unflagged, D13), AS-4b |
| 1.11 safe under load [L39] | PART | → AS-2j MemoryMax, AS-5a, AS-3a |
| 1.12 isolation + human authority [L41] | INC | → AS-2j, AS-29, labels (2f) · D7 (R1 gate) |
| 1.N do-not-claim [L43] | INC | → AS-4a lint, one negative per term; comparative claims need AS-23 (D36) |
| 1.C six aspects [L47-63] | MISS | → validator (AS-8/24); "demonstrated" needs an AS-25 record |
| **§2** 2.1 instructions [L71-99] | PART | AGENTS.md absent → AS-0 record; D29; D37 |
| 2.2 repo is truth [L101] | IMP | `diff -rq` 6/0 → AS-0 freeze |
| 2.5 classify [L107-121] | PART | STUB: telemetry stubs. DEAD: RAMPART, `assert_tenant_fleet_isolated` (`E/common/tenant.py:266`), `dismiss_token` [L], Mirror (SaaS) |
| 2.6 current work [L99,123] | MISS | → §11.10, X-CT, D28 |
| 2.7 migrate interfaces [L125] | PART | v1 `extra="forbid"` → v2 new classes; accept-sets before emitters |
| 2.8 no parallel systems [L127] | PART | Draft `C/egress_policy.py` and control-plane WM withdrawn (§11.13) |
| **2.E** reuse or not offer | — | See table below |

**2.E — existing functionality: reuse or explicitly not offer**

| Row | Status | Decision |
|---|---|---|
| E-1 Detection Mirror (`I/detection/*`, 1,874 lines; `IT/test_detection.py` 64 tests in P5) | DEAD for SaaS | Offered via AS-31. LEAD-only until certificate v2 with a defense signer (D20). Certificates go to the locker (11b) |
| E-2 credential-stuffing SPRT | UNT on path | Via declared login routes (2k→31) or IdP (17c) |
| E-3 LLM guard and canary | PART | Canary via engine-config (7b); LLM guard not in the first release |
| E-4 honeypot / OOB | PART | Honeypot offered (7b); OOB **not offered** |
| E-5 verdict webhook | PART | Not offered; replaced by AS-33 |
| E-6 `notify_channels` | — | Reused by AS-33 after D-ff |
| E-7 `connected_repos` | — | AS-15 release signal |
| E-8 `target_classification` | — | Not SaaS authority |
| E-9 Domain 7/8 oracles (`oracles.py:7070,8000`) | — | Path-2 posture after the AS-28 closure (17b/c) |
| E-10 CDN/WAF fingerprints | — | Promoted to vigil_core as a single source (AS-6d) |
| E-11 egress floor | — | Broker on `resolve_and_validate_all` with no lifts (2d). `_FloorPinnedBackend` is not reused: it lifts loopback and charter literals (`egress_guard.py:282-290`) |
| E-12 RAMPART | DEAD | Locker (11b) |
| E-13 telemetry stubs | STUB | Real only via 17b/c, 10a/b |
| E-14 middleware / SDK | DROP | — |

| Row | St | Today → Change |
|---|---|---|
| **§3** 3.1/3.2 boundaries, attackers | PART | → §4 |
| 3.3 privileges | INC | N-A → AS-2j + lint (the shipped unit is the negative control) |
| 3.4/3.5 dependencies, placement | PART/broken | → register (AS-0); trusted hop only per tenant, never the prefix list (5b) |
| 3.6/3.7 trust under compromise | MISS | → §4.4 |
| 3.8/3.C critical workflows, contract | MISS | → 2k declarations; §4.6 schema (2b/4b) |
| 3.9/3.S supported environments, honest state | PART/INC | → `not_observed`; no `protected` value (4b) |
| 3.T1 web attacks | PART | Benign corpus lacks attack shapes → same-shape corpus (1a), per-class FP (7b/23a), per-path opt-out (13b) |
| 3.T2–T6 | UNS | → 17b/c, 10a/b, 20 or "unsupported" |
| 3.T7 malicious dependencies | PART | → AS-28 |
| 3.T8/T10 authz changes, replay | PART | → intent row (2f), AS-29; leases + EpochStatement + acknowledgement (2a/2c/3d) |
| 3.T9/T11/T13 silence, defensive DoS, cross-tenant | MISS/INC | → 3b/3d; 5a/5b/6c; 2j/3a/2e/2d/2g |
| 3.T12 integrations | MISS | → broker (9a) |
| 3.T14 AI-aimed content | IMP | `contributing` → actor (`integrate.py:91-103`) → typed `rule_refs` (2b/3b) |
| SaaS-1 SSRF | MISS | → 2d + 2g |
| SaaS-2/3 DDoS, TLS custody | DEC | → 26/6c; D11 |
| SaaS-4 spine capacity | PART | → scoped append-rate measurement (3c) |
| SaaS-5 cross-uid hand-off | [U] | → relay (26) |
| SaaS-6 kill switch | INC | → 27a |
| SaaS-7 probes | INC | → 5c |
| SaaS-8 updates | MISS | → 6e |
| SaaS-9 separation | PART | → zones, AS-28 |
| SaaS-10 IMDS | hazard | → R15 |
| SaaS-11 T-XO | MISS | → §4.5 |
| SaaS-12 T-FO | MISS | → 2j/5a |
| SaaS-13 T-HOP | MISS | → R15 |
| SaaS-14 T-ISS | MISS | → 6c budget |
| SaaS-15 T-TR | MISS | → relay [U rsync advisories] |
| **§4** 4.0–4.15 assets | PART/MISS/UNS | Extend control-plane registries; anchors (9c); proof cascade (2l); **the control plane never imports the offense WM** (`S/reuse/__init__.py:47-56`); containers via 10a; route identity for 13b/14/15; one test per lifecycle case; graph-DB import guard |
| **§5** 5.0/5.EV evaluation | PART | → E1 ADR (§7.1) |
| 5.A.1 Coraza+CRS | MISS | → 7b; gated for tenants by 23a |
| 5.A.2 existing gateways | DEC | → 6c, 18a/b |
| 5.B Suricata | MISS | → 20 (D23) |
| 5.C Zeek | UNS | Reason in §7 |
| 5.D Tetragon/Falco | MISS | → 10a → 10b → 10c; enforcement UNS |
| 5.E Sigma | offense-only | Literal `*` (`sigma.py:252-266`) → 17a standalone |
| 5.F APIs | MISS | → defaults in §6 |
| 5.G files | UNS | D24 |
| 5.H DDoS | DEC | — |
| 5.V pins | MISS | → 7a/7b |
| 5.P modular | PART | — |
| 5.AD adapters | MISS | → 2b/4b |
| 5.OPS SIGIL operates every engine | MISS | → 6a, 30, 3d |
| **§6** 6.E1–E11 envelope | PART | → report v2 (2b/3b): `raw_ref`, `source{instance,component,build,boot,emit_seq}`, time quality, typed ids, `context`, `rule_refs`, `collection`, data/retention class |
| 6.C1–C13 ingestion | PART | → relay (26); receiver (9b); DRR, dedup, late label, reclaim, unit, accept-set, quarantine (3a/3c); async sinks (5a) |
| 6.H/6.M health, metrics | INC/MISS | → one schema with `component` (2b/3d); one emission test per metric |
| 6.P secrets | PART | → 2e, 2j |
| **§7** 7.1–7.5 methods | PART | Signatures (7b/17a/31); behavioural (31 + NAT/SSO corpora); **declared policy violations** (2k→31); invariants with same-shape corpus; anomaly = hypothesis |
| 7.6 IOC | UNS | Deferred |
| 7.9–7.13 content | PART/INC/MISS | → `detection-content.json` with required fields (8); `oracle_version` (D25); `rule_ref`; field maps + `not_deployable` (17a); pipeline + stale DEGRADED (7b) |
| **§8** 8.0 catalogue | MISS | 9 fields required; "supported" names its TT tests (8) |
| F1 | PART | 1a + 1b via DM-c |
| F2 | UNS | — |
| F3 | PART | 31/17c |
| F4/F12 | UNS→ | 17b/c |
| F5–F7/F11 | UNS→ | 10a/10c |
| F8/F9 | UNS | Flow logs / 20 |
| F10 | PART | Web withholding (5d) |
| F13 | UNS | Out of scope |
| F14 | PART | — |
| F15 | MISS | Edge metrics [U] |
| F16 | PART | 3d/27a/2j |
| **§9** correlation | MISS | Keys, basis labels (`same_observation`), duplicates ≠ corroboration, audited merge/split, ten answers, attribution negative `aegis:x:1:session:<victim>\|y`, single fold (11a) |
| **§10** 10.1/10.2 Observe | PART/MISS | Client IP forwarded (5b); **Observe creates proposals** (12/21) |
| 10.3 Defend | PART | 6th term `release_valid` (2c), 5d, 13a–c, 14 |
| 10.4 mode per tenant/group/capability | PART | Batch-issued signed grants (2k) |
| 10.5 switching | PART | Step-up + intent (2f); override (27b); release (29) |
| 10.6 existing protection | MISS | → 6d |
| 10.7 no silent removal | INC | → 2g, 13b/c |
| 10.8/10.9 six states, preview | MISS | → 4b |
| **§11** 11.1/11.2 fast path | PART/INC | Closure test (28); async sinks; partition continuity (3d) |
| 11.4/11.5 deadlines, latency | MISS | → 2b/12; 5a/23 |
| 11.6 outcome classes | INC | Five values (2b): `prevented_before_execution`, `interrupted_after_partial_execution`, `response_withheld_after_execution` (reported under spec "interrupted", qualified "app already processed it"), `contained_after_compromise`, `detected_without_enforcement` |
| 11.7 induced fail-open | INC | → 2j/5a |
| **§12** authority | PART/INC/MISS | WARDEN `classify` (2f); executor via `C/gate.py:63` (13a); reuse `Capability` (`C/capability.py:176-206,312,344,455`) + nonce ledger; one negative per denial; leases (2c); confused deputy §4.5; broker (9a); ImpactBudget (5e) |
| **§13** orchestrator | PART/MISS | Manifest = real adapters only; `_ContainmentBody` guard (12); 1b scope (13b) / provider rule (18b); 1c L7/IP-ASN only; 1d–f UNS; 1g–h (19); **one fault test per failure class** (12); eight states |
| **§14** verification | MISS | Z3 judge, distinct key, Z4 probes, origin-log Mirror, 11-field record (14); real tenants (32) |
| **§15/§16** recovery, revalidation | MISS/INC | `recovery_decide`; P1–P4 (R16); evidence snapshot first; `condition_binding` incl. release signal (consumer E / `connected_repos` / `POST monitoring/release`); one test per trigger (15) |
| **§17** findings→defence | PART/MISS | v2 negative certificate `mitigation_kind: edge_virtual_patch` (21); AS-21a guard (TB6); seven preconditions; feedback-loop guard |
| **§18** AI | PART/INC | Extend `assistant.py` after J2; tier gate; per-tenant consent (D26) (22) |
| **§19** self-security | PART/INC | Relay/receiver auth; isolation (2j); sealing (2h); override + `manage_defense` (27b); image in required A14 before first deploy (28); backups in **both** `I/backup.py:893` and `S/backup.py:182` (2i); per-engine health (3d); broker (2d) |
| **§20** profiles | PART/DEC | Six SaaS profiles + 10-field validator (24); 20.2f DROP; no-k8s DEC (H1) |
| **§21** evidence | MISS | Intervention record (11a/15); DEK locker, retention, legal hold, offline export (11b) · D12 |
| **§22** UI | INC/MISS | Gate-5 census for customer routes (`ui-wiring-matrix` covers only operator surfaces: its `generated_from` lists console, gated API, `uiproxy`, sigil server/actions, `vigil-ui`); lint on rendered strings (`provision.py:2834`; [U] `catalog.py:60,87` render?) |
| **§23** testing | PART | New-file rig, mandatory uids (1a); `V/tests/e2e` wired; T-01…T-24 map in §13.8 |
| **§24** benchmark | MISS | Arms A/B/C + **G1** (Coraza+CRS default PL1 blocking) + optional **G2** [U terms]; 14 metrics (23a/b) · D36 |
| **§25–§28** | PART/MISS | `use_mode` licence gate (7a); ADRs (0); 15 checks; release (29); D27; §12 |

---

## 3. Event trace today

1. **Authorize.** `POST /api/customer/monitoring/authorize` (`custui.py:1037,1194`) → `monitor_authorize` (`provision.py:2837-2884`). The chain is: flag, RBAC, step-up (no-op), 30-day window, plan gate, `grant()` (proof required; labelled A0), `_sign_monitoring` (v1: no epoch, no origin), publish, then audit.
   - **B1:** a failed publish revokes the prior grant (`:2873-2880`).
   - **B2:** nothing distributes the publish dir.
   - **B3:** no human acts.
2. **Secret.** Exported, then copied by manual scp.
   - **B4:** manual; shared uid; secret in env.
3. **Start.** Upstream comes from operator env; entitlement is read once.
   - **B5:** demo-secret fallback.
   - **B6:** unsigned origin.
   - **B7:** no TLS, no edge, offense tree.
4. **Request** (`:686-786`). Probes on the customer port → 411/413 → per-request authority (none = pass-through) → 5 terms → inspect 2 MiB → synchronous sinks → 403/429 keyed by peer → forward (XFF reset) → response withheld *after* the origin ran.
   - **B8:** actor collapse.
   - **B9:** unbounded resources.
   - **B10:** kill switch fails open.
   - **B11:** report sent before the 403.
   - **B12:** digest-only certificate, but the 403 promises offline proof; full verdict in journald.
5. **Report.**
   - **B13:** no transport.
6. **Drain** (`monitoring_spool.py:157-189`).
   - **B14:** no fairness, unit or reclaim; fold poisoning.
7. **Read.**
   - **B15:** dead looks the same as quiet; Respond overclaims while on HOLD.
8. **Revoke** = unlink.
   - **B16:** undelivered deletion is honoured until `not_after`; a re-plant is honoured; no "pending" state.
9. **Afterwards.**
   - **B17:** no incident, ledger, verification or drift handling.

Only AE-7 tests this trace, in-process.

**Hosted E1 trace.** viewer → Shield/CloudFront → VPC origin → ALB → front → gateway → broker → locked origin. Reports and grants move via the relay. B2, B6, B7, B8, B13, B16 and N-A close in AS-2c/2d/2g/2j, 5b, 26 and 3c before any TT-STG run counts.

---

## 4. Threat model and trust boundaries

**Governing rule.** The provisioner signs unattended (`provision.py:2866-2873`). Therefore the owner key may sign a tenant-influenced field **only** when it references a passed deterministic proof — the same pattern `grant()` uses (`monitoring_authority.py:159-171`). Today the origin has no provenance at all.

### 4.1 Zones (one VIGIL platform)

| Zone | Contents | Sees | Holds |
|---|---|---|---|
| Z1 AWS edge | Shield + CloudFront (distribution tenant per host, D3) | Routed plaintext | Viewer certs (AWS custody [U]); per-(org,host) edge secret [U] |
| **Z2E** enforcement | Unit = front + gateway (×2 across AZs for Defend); per-host **egress broker**; keyless payload-free **origin prover** | That tenant's plaintext | Report, pseudonym and origin credential, plus edge verifier, via `LoadCredential`. **No viewer keys, no cloud credentials.** |
| **Z2W** workers | Ingest receiver; connector pollers; detection jobs; Suricata | Telemetry | Short-lived broker credentials |
| **Z2X** actuation | Credential broker; defensive executor | Action specs | KMS-sealed provider credentials (Z2X role only) |
| Z3 control plane | custui, provisioner, spine, drains, folds, reconciler, placement, billing, RBAC; **verifier judge** (sends nothing) | All metadata | Owner key (vault + KMS KEK, `C/kek_backends.py:118`) |
| Z3R | Keyless relay | Files in transit | Nothing |
| Z4 offensive sandbox (may be its own account) | All payload-bearing probes | — | Offense credentials; egress **disjoint** from defense |
| Z5/Z6 | Customer; operators and CI | — | — |

Each Z2 segment, Z3 and Z4 has its own subnets/SGs, a permission-bounded role, a KMS key and its own egress. TT-STG uses the same layout in a separate staging account: environment separation, not a product split.

**Boundary rules** (each one is a test):

- **B-1** Z2 never initiates connections to Z3. Z3 pulls and pushes via Z3R with a pinned client, regular files only.
- **B-2** = R15.
- **B-3** Client IP is taken only from Z1, and only after the edge secret verifies. CDN chains follow §4.5.5.
- **B-4** No attack traffic outside Z4. Z2 and Z3 send payload-free requests only.
- **B-5** Z4 shares no host, image, role, key, subnet or egress with anything else (Reachability Analyzer, Access Analyzer, disjoint EIP sets).
- **B-6** Pushes land on the Z2W receiver, never on `I/uiproxy.py` (which serves the cockpit and billing webhooks, `:1-30,196-248`).
- **B-7** Parsing jobs never hold the connector principal.
- **B-8** The gateway honours only **signed** revocations. Release withdrawals and overrides are downgrade-only and honoured unsigned.
- **B-9** Only Z2X writes to provider APIs.
- **B-10** Each new path gets an egress-audit row.

**Attackers:** A1 internet · A2 hostile tenant · A3 RCE in a unit · A4 operator · A5 supply chain · A6 provider/connector · A7 customer sensor · A8 AI-aimed content · A9 compromised shared SIGIL component (broker, prover, receiver, relay, credential broker). AWS is a stated dependency, not an attacker.

### 4.2 Threat register

Each TT-STG negative is mandatory DONE acceptance for its slice. Aliases: T-CFB → T-PXY; T-RVD/REVP → T-RV; T-MEM/OOM → T-FO; T-RLY/XFER/RELAY → T-TR; T-LB → T-HOP; T-CB/BRK/CONN → T-CON; T-CERT → T-ISS; T-FLR → T-RP.

| Id | Threat | Control → slice | TT-STG negative |
|---|---|---|---|
| **T-XO** | A tenant names another tenant's or third party's origin and gets a relay from shared IPs | §4.5 binding (2g), broker (2d), crypto lock, catalogue | TT-A cannot declare TT-B's origin; TT-A's unit or uid gets no 2xx from TT-B's locked origin; forged client IP never arrives; raw CIDR/URL rejected by schema |
| T-SSRF | Metadata, rebinding, loops | Broker floor with no lifts; refuse SIGIL and edge-routed addresses; no redirects; broker refuses to start without `vigil_gateway.denylist` (`egress_pin.py:30-46`) (2d) · D37 | Every bad target refused with **zero bytes** sent |
| T-PXY | Forged attribution / foreign distribution | Edge secret + VPC origin (prefix list admits all distributions) (5b, 6c) | Forged header → `shared_actor` |
| T-XT | N-A | Per-unit uid, `ProtectProc=invisible`, read-only mounts (2j); N-A expected-RED (1b) | EACCES; `/proc` invisible |
| T-HOP | Co-hosted pivot | R15 (2j) | TT-B's uid cannot reach TT-A |
| T-RP | Replay; floor reset | Epoch; signed revocation; floor = max(local, EpochStatement) on boot, replacement and move; leases (2a/2c/6e) | Re-plant refused after replacement and after move |
| T-RV | Undelivered revocation | Leases (D35); acknowledged "pending" (3d) | Revoke during partition → *pending*; stops at lease expiry |
| T-REL | Machine-made enforcement | Release is the sole input + 6th term (29, 2c) | No release → no flag; provisioner-key release refused |
| T-FO | Induced fail-open | MemoryMax formula; Defend `fail_closed_503`; opt-in bypass only; crash-loop state (2j, 5a) | **Zero uninspected attack forwards**; *interrupted*; TT-B unaffected |
| T-FPB | Benign look-alikes blocked | Same-shape corpus; per-path opt-out; preview (1a, 13b, 4b) | Declared forum path: 0 blocks |
| T-DEF | Self-DoS | 429s off until 5b | 50 + 1 clients: only the attacker challenged |
| T-NN / T-ISS | Noisy neighbour; issuance starvation | §8.2; issuance budget (6c) | TT-B unaffected |
| T-INS | Operator abuse | UI-safe logs; audited downgrade-only override; edge drift paging; D8 (2e, 27b, 6c) | Manual edge change paged |
| T-HEC | Kernel escape | No unit network; **D7 kernel analysis is an R1 gate** | D7 evidence |
| T-FT / T-CON / T-ING / T-TR | Forged telemetry; shared principal; ingest flood; transfer client | MAC + tamper (3b/3c/9b); broker (9a); receiver (9b); relay (26) | Forged MAC alarms; ExternalId pivot fails; TT-B UI and billing webhook unaffected; symlink writes nothing |
| T-ATTR | `contributing` → actor | Typed `rule_refs` (2b/3b) | Crafted parameter attributes nothing |
| T-BYP / T-EXP / T-FRONT | Direct bypass; CDN silently removed; fronting | Lock checks (2d/6d); existing-edge check (6d); compiled routes + 421 (2c/6c) | "Origin reachable bypassing AEGIS"; test CDN not silently removed; SNI A + Host B blocked |
| T-SUP / T-BG / T-AI / T-DDOS / T-PRB / T-PUB | Supply chain; break-glass; injection; volumetric; wrong vantage; public weakness | 28/7a/7b; 27a before staging enforce; closed vocab; edge; B-4; §4.7 | Flipped byte refuses start; unreadable kill switch → no block; "unclassified"; volumetric not self-testable [U AWS policy] |

### 4.3 What stays trustworthy

| Compromised | Attacker gains | Still trustworthy |
|---|---|---|
| A unit | That tenant's secrets and plaintext | **Cannot choose egress or reach other units**; keyless authority |
| Edge misroute | — | Wrong edge secret → **fail closed** |
| Broker | Ciphertext only | Locked origins |
| Prover | Can forge an observation | Victim's lock and the uniqueness check |
| Relay | Can delay reports (shows STALE) | — |
| Host root | That host's tenants | Owner and release roots (D8) |
| Z3 | What the provisioner signs | Offline PCF; release root; m-of-n `TrustRoot` |
| Connector job | One tenant's short-lived credentials (broker only) | Other tenants |
| AI | Nothing | — |

### 4.4 Origin and edge-chain declarations

**Binding** (2g; patterns from `asset_ownership.py` and `ownership_fetch.py`):

1. **Declare** an HTTPS tuple; IP literals refused.
2. **Static refusals:** the floor, SIGIL/edge-routed addresses, and `hard_guardrail` (D39).
3. **Challenge** served at `/.well-known/vigil-origin-verification/<id>` **on the origin**. DNS-TXT, meta tags and "name under the proven domain" are all insufficient (`origin.attacker CNAME victim`).
4. **Prover fetch** through the broker using the exact forward tuple. Z3 never contacts tenant endpoints.
5. **Constant-time verdict.** A tuple already bound to another org is refused with a generic message. Overlapping **address sets** are allowed only with an exact (Host, SNI) proof, and are recorded for review (D39).
6. **Signed `aegis.origin_binding`** with high-water, expiring no later than the host proof.
7. **Re-proof** every D35 interval (24 h proposed), on any change, and on address drift. Three failures over ≥1 h → suspend → 502 "origin unverified" and an alert.
8. **v2 grant** carries `origin` + `origin_binding_sha256`. No binding → no signature. Gateway exits 2 on a mismatch.

**Broker** (2d): uid → bound (host, port) from the read-only signed grant; `resolve_and_validate_all(..., allowed_ips=frozenset(), loopback_allowed_if_scoped=False)`; SNI check; splice; TLS end to end. No `C/egress_policy.py` (memory: moving the denylist into vigil_core broke the gateway image). `IT/test_egress_backstop_mirror.py` is extended.

**Lock** (2d): mTLS client certificate (preferred), HMAC `X-Vigil-Origin-Auth` over (request id, ts, client-IP header), or a secret header over HTTPS. The client-IP header is trusted only on a credentialed request. An IP allowlist "does not isolate you from other SIGIL tenants". `lock.state` ∈ {none, ip_allowlist_only, crypto_verified, crypto_verified+ip}, checked by the prover and by a Z4 payload-free GET. **A release requires crypto_verified.**

**Edge chain:** the tenant picks a `provider_id` from a SIGIL catalogue signed by the release root (unattended updates may only shrink it), plus a per-tenant chain secret. Otherwise `shared_actor`.

**Who signs what:**

| Record | Signer | Unattended? |
|---|---|---|
| Host/origin proofs, v2 grant, lease, revocation | Owner key via provisioner ("provisioner-signed") | Yes |
| EpochStatement | Owner key via reconciler | Yes |
| Edge catalogue | Release root, m-of-n | No |
| **Enforce release** | **Owner act** under the release root (D30) | **No** |
| Slow-path token | Provisioner; high-impact also needs the release root | Partly |
| Reports | Per-(org,host) MAC | — |

### 4.5 Protection contract (`C/protection_contract.py`, read-only, never authority)

AS-2b ships the full schema. AS-4b evaluates it through a provider registry, so later slices add providers by appending. Fields:

- `asset{ref, connection_path, instance_id, cell, placement_state, isolation_tier, criticality(tenant_declared, authority:none)}`
- `edge{provider, ddos, viewer_tls{terminated_by:aws_cloudfront, key_custody:aws, cert_state}, edge_auth, chain, rate_limits}`
- `origin{tuple, binding{sha256, state, reprove_due}, lock{state, method}}`
- `required_sensors[]{kind, state incl. crash_looping}`
- `available_telemetry`, `not_observed[]` (direct-to-origin, cache hits, chunked, >2 MiB, multipart, websocket, non-HTTP)
- `active_detections`, `enforcement_capabilities{kind, outcome_class, effect_verification}`
- `authorized_actions{grant_sha256, signer_role, mode, epoch, capabilities, lease, revocation{state, acked_epoch}}`
- `enforcement_terms{grant_block, enforce_release{state}, flag, mode, entitlement, kill_switch, override, effective, reasons}`
- `failure_policy{gateway_down, overload, basis, tenant_opt_in}`
- `verification_checks`, `critical_workflows`, `customer_prerequisites` (CNAME, cert, origin bound, crypto lock, existing edge preserved or acknowledged, client-IP header)
- `coverage_limitations`, `versions`, `budgets`
- `trust_assumptions` (aws_edge_terminates_viewer_tls, sigil_unit_sees_plaintext, gateway_reported_not_independent, ip_allowlist_lock_does_not_isolate_tenants)
- `licensing_mode`, six `dimensions`
- `headline` ∈ unsupported | inactive | degraded | unverified | observing | enforcing

**Validators:**

- **V1** no `protected` value exists.
- **V2** an unproven tenant field caps the headline at `unverified`.
- **V3** `enforcing` needs a release, a crypto lock, a verified edge and applied receipts.
- **V4** an uninspected policy with block capability needs consent.
- **V5** an unverified chain → `shared_actor`.
- **V6** pending revocation, suspended binding, crash loop or expiring lease → `degraded`.
- **V7** vocabulary only; 1a reaches outcome classes 1, 3 and 5 only.

### 4.6 Trust statement (inputs to D11 and D29)

- AWS CloudFront terminates viewer TLS; keys are in AWS custody [U].
- An ALB (if used) and the tenant's unit see plaintext.
- SIGIL holds KMS-sealed origin, edge, report and pseudonym secrets.
- Confirmation stays offline and keyless.
- Roots: the owner root, plus a separate release root.

### 4.7 Publication

- **Public:** `aegis-threat-model.json` and `.md`, fixed state only.
- **Private:** open entries.
- A drift test checks that every id exists in one of the two.
- An entry may be marked `mitigated` only with a current AS-25 record.

---

## 5. Hosted architecture

**Request path:** Internet → Shield/CloudFront (distribution tenant per host; caching disabled by default; origin request policy forwards Host + `CloudFront-Viewer-Address`; per-tenant edge secret) → VPC origin → internal cell ALB (SG admits only the VPC-origin service; host rule → target group per `instance_id`, compiled from grants × placement) → front (ALB-only ingress; verifies the edge secret; origin-hop cert from the central keyless issuer; optional Coraza) → unix socket (`SO_PEERCRED`) → gateway (PrivateNetwork; 421 on a foreign Host; checks lease, floor and release) → broker → NAT (defense EIPs) → locked origin.

**Other flows:**

- Reports: Z3R pull → landing dirs → drain.
- Grants, leases, EpochStatements, releases and overrides: Z3R push → read-only per-tenant dirs.
- Sensor and provider pushes: `ingest.<sigil-zone>` (its own distribution) → Z2W receiver.
- Origin challenges: prover → broker.

**Fallbacks if CloudFront features are missing** (D3): per-host distributions; or a public ALB with the prefix list plus a per-tenant header.

**Authority distribution:**

- Leases: proposed 1 h, renewed every 10 min (D35). "≤1 s" refers to the unit-local verify cache only; end to end, the lease bounds it.
- EpochStatement accompanies every publish; floor = max(local, statement).
- Heartbeats acknowledge grant, revocation and release epochs.
- The reconciler renders `VIGIL_AEGIS_SAAS_ENFORCE` and the governed `AEGIS_RESPOND` **only** from a valid release, and the unit re-verifies it keylessly.
- Overrides and trips reach the unit via its control dir. The console `aegis_set_mode` (`E/console/actions.py:6209`) is not a SaaS channel.
- Lease expiry follows R16.

**Sizing and failure:**

- `MemoryMax ≥ pool × (max_req + 2 × max_resp) + base` (≈60 MiB per slot today, `:87-89,343-356,576-577`). AS-5a removes the join copy.
- Defend: HA pair; no edge bypass; 503 "Protection interrupted"; crash loop → `PROTECTION_INTERRUPTED`.
- Observe: overload sheds inspection (counted); unit down → error unless the tenant opted into failover [U].
- Updates use target-group swap with draining (6e).

**Host isolation** per R15, plus `ProtectProc=invisible`, `RestrictNamespaces`, `NoNewPrivileges`, `SystemCallFilter=@system-service`, `RestrictAddressFamilies` and `StartLimit*` in `[Unit]` (`vigil-command.service:52-56`).

**D7 (R1 gate, decided by kernel-isolation analysis):** T1 shared hardened kernel with a host cap and patch SLA; T2 ECS/Fargate task per unit [U]; T3 dedicated host. H1 records TT-STG start time, p99, cost and new code size.

**Orchestration and storage:**

| Choice | Reason |
|---|---|
| No k8s | Long-lived workloads; `infra/ha/k8s` is sovereign-HA only |
| EC2 + systemd | Reuses the unit file and OIDC/SSM pattern |
| ECS/Fargate | D7 alternative |
| CloudFront + ALB | Replace the router, ACME and route-flip code |
| Placement | New record class reusing the `shard_assignment.py` mechanics (sovereign, `assert_no_offense :39-41`). v1 `ShardAssignmentRecord` stays frozen (`extra="forbid"`, one per tenant, `:62-77`). H1 picks between a versioned discriminator and a separate kind |
| Data | File spool + one spine + per-tenant DEK locker; no streaming or graph DB |

**Delivery:** CloudFormation, following `infra/aws/github-oidc-deploy-role.cfn.yaml` (currently the only file there). Staging auto-deploys. Production is owner-dispatched in a protected `aegis-production` environment (today only `pypi` is protected, `release.yml:225`).

---

## 6. Customer connection paths

| Path | Customer-side requirement | Observes | Enforces | Slices |
|---|---|---|---|---|
| 1a | CNAME; serve the challenge; per-tenant lock | Routed L7 traffic | 403, withholding, 429s (after 5b), scopes | 2*, 3*, 4*, 5*, 6*, 7b, 8, 26–29 |
| **1b (forward)** | Scoped provider token; optional log push | Provider events (no bodies) | Verified provider rules | 18a, 18b |
| 2 | Read role with a SIGIL-issued ExternalId; IdP consent | Audit, sign-ins, posture | None by default (19 separate) | 9a–c, 17a–c, 19 |
| 3 | **Sensor always required**: provider agent (default), EDR, or SIGIL kit (later; distribution) | Runtime findings | None | 10a, 10b, 10c |
| 4 | VPC mirroring or flow findings | Packets (IDS) or flows | **None** | 20 |

Unbuilt paths read "Unsupported: needs <connection/sensor>".

**1a onboarding** (`#/monitoring/connect`, 6d):

1. Ownership proof, reusing consumer C [U one org per host].
2. Plan.
3. **Existing-protection check** using the promoted fingerprint table → offer 1b, a chain, or an acknowledgement (recorded with `preview_sha256`).
4. Declare the origin.
5. Bind it (§4.4).
6. CNAME (apex needs ALIAS [U]).
7. Lock the origin.
8. Configure the client-IP header.
9. Observe.

- **Not observed:** direct-to-origin, cache hits, WebSocket, chunked bodies until verified [U], over-cap bodies, multipart, non-HTTP.
- **Health checks:** CNAME, cert, binding, reach, lock, traffic seen (zero = "possible bypass"), targets, lease acknowledgement, build, acknowledgement.
- **Stated plainly:** AWS and the unit see plaintext; an unlocked origin is bypassable; unrouted origins are unprotected.

**1b:**

- **Order:** Cloudflare, then AWS WAF (D22).
- **Cloudflare:**
  - zone token [W]; over-broad tokens refused [U];
  - per-rule writes in `http_request_firewall_custom` [W]; customer rules never overwritten;
  - every expression includes `http.host in {proven}`; no raw string splicing;
  - observe via Logpush (generally Enterprise [W, conflicting]) or a sampled GraphQL pull (`sampled=true`) → AS-9b → Mirror (DM-c);
  - SIGIL withdraws rules at expiry; a rule persists if SIGIL is down (P3).

**Virtual patching (E3c; 1a via 21, 1b via 18b):**

1. Z4 FACT with PCF (never a LEAD).
2. `defense_proposal` with no authority.
3. Tenant step-up, plus a release in production.
4. Z2X applies a narrow, self-expiring scope or provider rule.
5. Read-back.
6. Z4 retest via `retest_reproof`: exploit refused through the edge; benign workflow passes; the direct path is **still vulnerable**.
7. The Z3 judge signs a `ResponseVerificationRecord` (PCF + v2 negative certificate `edge_virtual_patch`), exportable offline.
8. Status `STILL_VULNERABLE + mitigated_by_edge` until a direct retest clears it (21a).

**Path 2:**

- CloudTrail plus **one IdP named by the owner** before 17c. Default Okta; Entra is the alternative [U: P1/P2 needed for sign-in logs].
- Unique ExternalId per tenant; an account bound to another org is refused; health proves AssumeRole fails without the ExternalId; broker-issued credentials.
- Posture uses the Domain 7/8 oracles; events use Sigma + Mirror; latency in minutes.

**Path 3:** (p) GuardDuty Runtime Monitoring (EC2, ECS incl. Fargate, EKS [W]) via the path-2 role, with the AWS agent distributed by AWS → (e) EDR, TT on a SIGIL developer tenant or Blocked by D23 → (k) kit, disabled until D10/D23. Runtime enforcement is unsupported.

**Path 4:** IDS on mirroring (customer pays; D23) or flow findings. "Restrict a connection" exists only as a 1a scope, a 1b IP/ASN rule or an edge rate rule.

**Confused-deputy register** (each row is a TT-STG negative):

| Deputy | Control |
|---|---|
| Forward | §4.4 |
| Client IP | Edge secret, `SO_PEERCRED`, channel binding |
| Routing | Compiled routes + 421 |
| Prover / health | Bound targets only, payload-free |
| Reconciler | Release |
| Connector | ExternalId + broker |
| 1b push | Zone anchor, host-scoped expressions, SIGIL rule ids |
| Receiver | Per-connection HMAC + caps |
| Exports | `notify_channels` guard |

**Real tenants (32):** `outcome_verified` needs tenant-authorized non-destructive checks under existing signed authority (`IdentityAttestation` + `Capability`, `C/capability.py:110,280,455`) plus connected origin logs. Otherwise "Outcome not verifiable: needs …".

---

## 7. Engines and licensing

**Evaluation (E1 ADR):** capability, platforms, maturity, security history, footprint, updates, failure modes, overlap with 2.E, prerequisites, licences (engine, dependencies, rules), redistribution. Each [U] is closed by `git ls-remote --tags`, advisories, release date and a TT-STG run. An engine runs only per active connection. CRS is offered only when 23a shows B > A. The G1 baseline is CRS on Coraza; a managed-rules arm needs terms [U] (D36, D10).

| Component | Decision |
|---|---|
| AEGIS gateway | Selected (28 closure) |
| CloudFront + ACM + Shield (+Adv / WAF rate rules per D3) | **Selected (E3a)**; replaces the SNI router and ACME; WAF only for floods; Firewall Manager does not cover multi-tenant [W] |
| Cell ALB | Selected |
| Front (Caddy [U pin]) + Coraza v3 + connector (D21) | DetectionOnly; CRS hits are leads in `rule_refs` |
| CRS 4.x | Selected, gated for tenants; G1 arm |
| **Detection Mirror** | **Selected, evaluated before Sigma** |
| Sigma v2 + SigmaHQ | Standalone; pySigma not shipped |
| Domain 7/8 oracles | Selected after 28 |
| Virtual-patch compiler | Selected |
| GuardDuty findings | Path-3 default |
| EDR APIs | Preferred integration |
| Tetragon + forwarder | Conditional, observe-only |
| Falco | Ingest only |
| Suricata [L] + ET Open (reviewed) + Vectorscan (Hyperscan ≥5.5 is proprietary [W]) | Conditional (D23) |
| Zeek | **Not planned** (no packet vantage beyond mirroring, which EVE covers) |
| IOC | Deferred (no licensed feed) |
| File scanning | D24 |

**First-party features:**

| Feature | Status |
|---|---|
| Oracles and 429s | Offered |
| Honeypot and canary | Via engine-config (7b) |
| OOB | Not offered |
| SPRT | When login outcomes are declared |
| LLM guard | Later |
| Middleware / console setup | Not offered |
| Webhook | Replaced by 33 |

**Service vs distribution** (engineering analysis; counsel under D10):

- **Service use:** GPL, LGPL, Apache, BSD, MIT impose nothing. AGPL §13 applies only if modified. **Deny** SSPL, Elastic-2.0, Commons Clause, CC-BY-NC. BUSL per its grant. **DRL-1.1 requires attribution in match views** [W].
- **Distribution** happens only for the CFN template, the kit, the forwarder/SDK and rule content pushed to customers. Obligations: GPL source (denied for kits by default), LGPL relinking, Apache NOTICE, MIT/BSD text, DRL attribution. CRS- or Sigma-derived pushed rules carry their notices.
- `V/LICENSE` is PolyForm-NC + Gov-Use, or commercial. Distributed artefacts therefore need customer terms (D10).

**Matrix:**

| Component | Licence | Decision |
|---|---|---|
| First-party | PolyForm-NC / commercial (`LICENSING.md:5-12`) | Allow; terms D10 |
| Gateway Python deps | Permissive [U: trivy license scan] | Pending |
| Base image | Mixed [U] | Service only |
| AWS services | AWS terms; benchmark clauses [U] | D10 |
| Coraza, Caddy, CRS | Apache-2.0 [W/U pin] | Allow |
| HAProxy | GPL-2.0 [U] | Service only |
| Go module sets | [U: scan `go.sum`] | Gate |
| SigmaHQ | DRL [W] | Attribution |
| pySigma | LGPL [L] | Dev only |
| Suricata | GPL-2.0 [L] | Separate process |
| libhtp, Rust crates | [U] | Verify |
| Vectorscan | BSD | Allow |
| ET Open | Conflicting | Review |
| ET Pro / Snort subscriber | — | Deny |
| Tetragon | Apache + GPL BPF [U SPDX] | Review |
| GuardDuty agent | Customer terms | Not SIGIL distribution |
| Cloudflare API | Customer terms [U agent clauses] | Review |
| Elastic / SSPL / CC-BY-NC | — | Deny |
| TI feeds | None used | Gated |

**Licence gate (7a):** `license_policy.{json,py}`, `engine-pins.json`, `THIRD-PARTY-ENGINES.md`, `NOTICE`.

- Rules: `use_mode` tag on every artefact; GPL-3.0 fails **only** distribution; AGPL-mod/SSPL/Elastic/BUSL/CC-BY-NC fail service images; **unknown licences fail** unless ledgered; bundles carry licence digests.
- Negative controls: GPL-3 in the kit; Elastic rule in a service image; unknown licence; DRL without an author.
- Lands before the first third-party deploy. R1 requires a record for every deployed engine.

---

## 8. Isolation and fairness

**Rule for every shared resource:** a per-tenant bound, an over-limit state visible to that tenant through VIGIL, and a TT-STG test showing TT-B unaffected. All flagged off by default and byte-identical when off.

| Object | Design → slice |
|---|---|
| Edge | Distribution tenant per host; reconciler-only writes; host bound to another org refused (6c) |
| Origins | §4.4 (2d, 2g) |
| Events | Per-instance subtrees; strict domains; DRR with a reserved heartbeat budget; dedup; relay (3a, 3c, 26) |
| Policies | Read-only per-tenant dirs; EpochStatement and leases; acknowledgement; signed closed-vocabulary engine config, never raw SecLang (2c/2f/2g/7b) |
| Enforcement | Release (29); per-unit control dir with `CRUCIBLE_ROOT_STRICT=1`; fleet `aegis_enforce` latch reusing the customer-latch mechanism (`S/governor/capability.py:53`), rendered as `.halt`. `vigil panic` and `sigil warden kill` mean pass-through, **never stopping units** (27a/27b) |
| Credentials | Sealed (2h); `LoadCredential`; keyless certificate issuer; connector AAD `aegis-conn\0org\0conn` (`C/sealing.py:63`); broker (9a) |
| Compute | One slice per unit; admission control; pool derived from MemoryMax (P = ⌊(0.8·MemoryMax − base)/R⌋); no engine process parses two tenants' traffic (2j, 5a, 7b) |
| Incidents and evidence | Org-partitioned; another tenant's object answers like a nonexistent one; DEK per (org, epoch) (11a/11b) |
| Ingest | Dedicated receiver (9b) |
| Notifications | Coverage notices only, after D-ff (33) |
| Logs | HMAC ids; UI-safe projection (2e) |
| Operators | `manage_defense`: trip, downgrade, withdraw, relocate — **never** grant, release or read plaintext outside D8; SSM-only, recorded sessions (27b) |

**Fairness layers** (each degrades only the tenant over its bound):

| Layer | Bound | Over the bound | Slice |
|---|---|---|---|
| L0 edge | Shield; WAF rate rule per tenant (host+IP and host ceiling) [W] | 429, shown | 6c (volumetric not self-tested [U]) |
| L1 issuance | New-host budget; renewals first | "Certificate pending" | 6c |
| L2 ingress | Per-tenant concurrency; per-viewer sub-caps; explicit timeouts; NAT ranges alert-only | 503, counted | 6c/5b |
| L3 unit | cgroups, admission | §8.1 | 2j/5a/7b |
| L4 reporter | 600/min; heartbeat never dropped | `DEGRADED(dropped)` | 3b |
| L5 drain | DRR | "Reporting delayed" | 3c |
| L6 spine | Budget below the `sigil spine append-rate` ceiling; rollups | Aggregated | 3c/3d |
| L7 ingest | Pre-auth caps; buckets 429 + Retry-After | `DEGRADED(quota)` | 9b/17b |
| L8 slow path | Pure `fair_order`/`admit` (`fair_queue.py:104-165`) over the ledger; wiring only after #1202 | "Queued (n)" | 12/13a |

**Overload behaviour:**

| Event | Observe | Defend |
|---|---|---|
| Over capacity | `shed_inspection` (counted, disclosed) | `reject_503`; shed only on opt-in |
| Unit down | Error; failover on opt-in | HA pair; no failover |
| Lease lapse | Pass-through (R16) | Pass-through (R16) |

Others states are shown as: "Revocation pending", "Reporting delayed", "Enforcement paused by SIGIL", "Detection degraded: <engine>".

**Cells:** ≥2 AZs behind one ALB; shared L0–L2 fate is stated. ΣMemoryMax ≤ host RAM minus reserve; CPU overcommit for Observe only. Defend runs an active-active pair (D4). Until D7, TT-A and TT-B are co-located in staging only.

---

## 9. Implementation program

### 9.1 Rules

1. **Flags.** Strict default-OFF flags (pattern `_saas_enforce_flag`, `gateway.py:97-101`) with a flag-off golden; routes return 404 when off. Shipped unit, compose, `A/Dockerfile` and AE-7 stay untouched. Unflagged exceptions only for D13 copy and D9 crash fixes.
2. **DONE through VIGIL.**
   - Each slice names its GET ops (`_customer_read_relay`), POST verbs and `#/monitoring/*` screens.
   - Org comes from the session.
   - Customer routes go in the **Gate-5 census** (`_SCREENS`/`_CONTROLS`/`_ROUTE_STAGES`, `:85-110`). Operator routes go in `ui-wiring-matrix.json`.
   - **New-module convention:** new Provisioner delegate methods, appended `if path ==` lines (`custui.py:628-642`), and new screen scripts appended to `index.html` (5 at `:36-40`) and `ROUTES`. Only edits that *modify* existing monitoring code make a file hot.
3. **Tiers.**
   - **TT-CI** (merge): offense-leg list (`ci.yml:548-605`) under `VIGIL_REQUIRE_PLANES`, in the same PR; distinct uids mandatory.
   - **TT-STG** (DONE): authenticated TT-A/TT-B sessions over HTTP + Playwright; positive, benign, negative, cross-tenant and fault cases; an AS-25 record. A case that asserts a later slice's rendering waits for that slice.
4. **E1 grade:** metrics and alerts, runbook, rollback on TT-STG, capacity figure, R-gate line.
5. **Blocking** needs consent, release, tier, flag and kill-switch clear. TB5/TB6 also bind staging.
6. **Defensive only:** payloads come only from Z4 via `retest_reproof` and the `http_executor` chain under a signed charter; fixtures are inert; GATE-MARSHAL attests and adds egress-audit rows.
7. **Merge gate:** §11.5.
8. **Parallel** development in `/home/kali/vigil-wt-*` once dependencies have merged. Merges are serial in §9.5 order. Hot-file slices develop serially. Each PR carries its own registrations and manifest.
9. **X-CT** (`*`) for slices that modify the customer surface or depend on consumer modules. #1202 must land first. D28 is widened.
10. **Doctrine:** wire, don't rebuild; no AI on authority paths; no `protected` state.

### 9.2 Adaptations from the §26 order

| # | Adaptation | Reason |
|---|---|---|
| 1 | Evidence (25), rig (1a) and infra head (28 → 6a → 26 → 1b) join steps 1–2 | DONE needs a connected test tenant |
| 2 | Contracts (2a/2b) in step 2 | §26.2 |
| 3 | 27a, 3a, 4a in W1 | §26 forbids postponing failure handling and truthful UI; TB5 |
| 4 | 29 and 21a before any enforcing path, staging included | Owner authority; TB6 |
| 5 | Step 4 also covers 18a, 31, 23a, 34 | E3b, E3d, E1 |
| 6 | Ledger and slim image before any deploy | TB9 |
| 7 | Step 5 = 10a via the 17b role (CloudTrail moves from step 10); Sigma standalone | Path 2 never waits on runtime or distribution |
| 8 | 21 and 18b after 14 | E3c needs verification |
| 9 | Intervention record completes in 15 | It references ledger and verification records |
| 10 | Dependency-free steps run concurrently. **Requires D38**; fallback: step k starts after milestone k−1, with 1b still forward | Parallel agents |
| 11 | 2l pulled into W1 | Resurrection-proofing cannot wait for offboarding |

### 9.3 Slices

Columns: Slice | Scope | TT-STG DONE (TT-B always the control; plus TT-CI negatives named in §4.2/§2).

| Slice | Scope | TT-STG DONE |
|---|---|---|
| **AS-0** governance | ADRs: scope; **H1** zones/orchestration/placement/D7; **E1** engines; **E3** competitive; **EV** evidence. Threat model (§4.7); trace JSON; **slice-graph JSON + `test_aegis_slice_graph.py`**; `make gate`/`gate-zero-fp` baseline (`Makefile:28,35`; no workflow runs it) and corpus sha256 on the blackboard; 15 check names; N-A expected-RED; CODEOWNERS + `_CRITICAL_PREFIXES` (`test_codeowners_coverage.py:51-63`); charter template; FORGE domain map; current-work inventory; E1 record | Baseline shown in TT-A's `#/monitoring` |
| **AS-25** DONE records (D33) | Signed binding over `verification_manifest`; tag `aegis-tt-demonstration`; trust root `docs/evidence/aegis-tt/`; verifier `IT/test_aegis_tt_demonstrations.py` in P5; status projection | First AS-1b record verifies offline; a 1-byte flip fails |
| **AS-1a** TT-CI rig | New `IT/test_aegis_tt_rig.py` + fixtures; custui HTTP; distinct uids (`VIGIL_REQUIRE_TT_UIDS`, skip = fail [U useradd]); real test root; MERIDIAN; Playwright `aegis-tt-ci-browser` (advisory); same-shape corpus; `vigil tt-acceptance` | (TT-CI) Observe → event visible; benign baseline recorded; TT-B sees nothing |
| **AS-2a** authority contracts | `MonitoringAuthorityV2` (subject, capabilities incl. `virtual_patch`, origin + binding, `edge_chain`, epoch, **lease**); signed revocation; EpochStatement; EnforceRelease (separate root); unsigned withdrawal and override; instance-id + replica index; one publish-path helper; all domain tags | Via 2c/29 |
| **AS-7a** licence gate | §7 gate + advisory tracking | Deploy-time ledger check passes |
| **AS-27a** break-glass | Kill-switch exception ⇒ tripped; mode-file error ⇒ last-known-good, else observe; AEGIS-only atomic writer (`secure_write`, `paths.py:226-245`, untouched); writable control dir with `CRUCIBLE_ROOT_STRICT` (compose variant); SaaS 403 says "proof not retained" | Drill runs in AS-29 (TB5) |
| **AS-3a** drain strictness | Strict domains; quarantine; no global scan | Poisoned TT-B file: TT-A p95 holds; alert fires |
| **AS-4a\*** overclaims | Copy at `app.js:1814,1744,1921`, `guide.js:242,248`, `DEPLOYMENT.md:5-7`, `provision.py:2834`; pinned tests `:347,424`; enumerated lint | No blocking claim while held |
| **AS-2l\*** proof cascade | `revoke_org` cascades proofs and anchors [L `orgs.py:394-436`] | Re-created org has no proofs |
| **AS-2b** schemas | `aegis_event` v2, health v1, `asset_ref`, `adapter_capability`, protection contract + provider registry, response manifest (deadlines, five classes) | Via consumers |
| **AS-2i** backup | Exclusions in both backup paths; restore drill (after legal-gov S1) | Restored backup: TT-A stays Off; nothing inspected |
| **AS-2h** sealing | KMS + AAD; overlap; delivery via 6a | Rotation leaves no event gap; old secret refused; AAD mismatch fails |
| **AS-28** closure + image (D34) | AST closure + deny list; `Dockerfile.saas` hash-locked, built/scanned/SBOM/signed **inside required A14**; manifest entries; closure test (also with `anthropic` blocked) | Deployed digest verified by 6a |
| **AS-2j** isolated unit | R15 units (`--listen-unix`, `SO_PEERCRED`, PrivateNetwork, ALB-only front); `LoadCredential`; read-only mounts; MemoryMax formula; hardening; lint (shipped unit fails) | EACCES matrix; no route/DNS/IMDS from the gateway; N-A RED on the shipped unit |
| **AS-2f\*** sovereign v2 | v2 grant/revoke; EpochStatement; `classify` A3; high-water; seeded `_last_issued`; read-back; **intent row first; step-up for Respond and capability changes**; `monitor_renew`; labels; no-LLM test; **no tenant origins (2g)** | Step-up + activity line; TT-B POST → 403 |
| **AS-2c** gateway v2 | Dual-accept; floor; signed revocations; lease ⇒ pass-through (R16); 1 s cache; **`release_valid`**; **421 on foreign Host**; `MonotoneFloor.tla` extended | Re-plant produces no event (events API) |
| **AS-6a** placement + reconciler | §5.7 placement; `aegis_fleet.py`; worker-kind registry (units, broker, prover, receiver, jobs, Suricata, executor, judge); digest check; credentials; health; rollout; retire (snapshot first); SSM only; **never renders enforcement before 29**; only SIGIL TT origins before 2g | Observe → routed with no operator step; Off → retired; tampered image refuses |
| **AS-26** OPS-1 | CFN zones, keys, roles, disjoint EIPs; Route 53 test zone; **static staging CloudFront + Shield** (2 tenants [U]) → VPC origin → ALB; edge secrets; keyless cert issuer; brokers; **Z3R relay** (pinned rsync, `--no-links`, landing dirs); IMDSv2; SSM doc; test owner and **test release** keys; paging (D32); ledger check; drift detection; staging-only CD; `aegis-infra` advisory job | MFA sign-in; analyzers clean; relay EACCES on vault; hostile file names harmless; unauthenticated transport refused; page delivered; CloudFront bypass refused |
| **AS-1b** TT-STG runs | `aegis-test-tenant-e2e.yml` (dispatch + daily; `KNOWN_NONPR_ADVISORY` `:144`) under `verification_manifest run`; staging signs; record PR merged by owner; cumulative | First run = DONE for everything merged earlier: DNS → CloudFront → unit → MERIDIAN; SaaS-5; chunked |
| **AS-2d** broker + lock | §4.4 broker, forward, lock, startup refusal; backstop mirror extended | MERIDIAN lock; Z4 unauthenticated → 401; no 2xx cross-tenant; forged IP never arrives |
| **AS-2g\*** binding + publish | `origin_binding.py` + prover; per-instance publish; publish failure honest (B1); stray sweep (`monitoring_subscription.py:110-130`); "Declare origin" | Cross-tenant and CNAME tricks refused; honest removal; suspension → 502 |
| **AS-2e** hygiene | UI-safe stderr; HMAC snapshots; demo secret refused | No seeded secrets or raw IPs in journald or snapshots |
| **AS-2k\*** settings | Default mode; asset groups (batch-signed grants); failure opt-ins; workflows; criticality; login routes; policies; content surfaces — stored as requests | Group grants correct; still need step-up and release |
| **AS-21a\*** retest guard | SIGIL 403 ⇒ "inconclusive (edge enforcing)" (`:2733-2736`); flag-hold criterion; rebased on consumer E | With AS-29 |
| **AS-29\*** enforce release | `enforce_release.py`; `sigil enforce-release sign` offline/m-of-n (static test: key unreachable from sovereign paths); render rule; ceilings for 13b/18b/19/21; release state on the card | Test-key release → Z4 SQLi 403, no nonce; withdrawal alerts; TT-B unreleased stays observe; **kill-switch drill** (TB5); 21a cases |
| **AS-3c** drain hardening | Accept-set; DRR; reclaim from `billing_spool.py`; dedup; late label; alarm; append-rate budget; drain units | Flood isolation; killed drain loses nothing |
| **AS-3b** v2 emission | v2 + `rule_refs`; depth pre-scan; heartbeats; front sidecar; no UI | v2 fields via API; crafted-parameter negative |
| **AS-3d\*** health + ack | States incl. `CRASH_LOOP`, `EXPIRING`, `REVOCATION_PENDING`; fold cache on (`ci.yml:1036`); metrics with emission tests; alerts; `GET …/health` | Stale within 3 heartbeats + page; partition: delayed then backfilled, no gap; enforcement within lease; revoke pending → stops at expiry |
| **AS-4b\*** contract + preview | Evaluator + providers; manifest; profiles; six-state card; vocabulary incl. later fields; v2 rendering; preview + `preview_sha256` | Honest states; preview digest matches; injected lint term fails |
| **AS-27b\*** override + `manage_defense` | `sigil aegis-override` (downgrade-only, spined, rendered into the control dir); console audited; `C/rbac.py` role; SCIM ceiling test | "Stopped by SIGIL"; upgrade refused + audited |
| **AS-5a** capacity | Pool; admission; join copy removed; async sinks; §8.1 policy; crash-loop; eviction; TTL; histograms | Saturated Defend TT-A: **zero uninspected attack forwards**, "interrupted"; TT-B p99 intact |
| **AS-5c** probe socket | W6-1 claim updated | MERIDIAN `/healthz` reachable |
| **AS-5d** outcomes + receipts | Five classes; report after send | Prevented / withheld / unknown labelled correctly |
| **AS-5e** ImpactBudget (D19) | Graduated actions only | "Throttling paused" |
| **AS-6c** edge automation | Edge compiler worker kind replaces static tenants (re-attest); managed certs; ALB rules; secret rotation; WAF rate rules; L1 budget; Observe failover opt-in; drift paging | Cert with no human step; no cross-routing; missing secret refused; per-source flood contained; churn isolated |
| **AS-5b** attribution | Viewer address only after the secret verifies, passed over the socket; catalogue; NAT suppression | 50+1 clients; forged headers ignored |
| **AS-6d\*** onboarding | §6 wizard; promoted fingerprints (`make gate`); prover and Z4 lock checks | Checks honest; CDN never silently removed; bypass shown |
| **AS-6e** rolling | Target swap with drain; floor carried; moves bump epoch; rollback; H1 measurements | Zero drops; re-plant refused after replace or move |
| **AS-7b** Coraza + pipeline | Front Coraza with minimal audit; `coraza_audit.py` leads joined **in the fold** (no `gateway.py` edit); signed engine config; pipeline; detections screen | TT-A-only CRS lead; FP per class; tamper refused; exact-hash rollback; stale age shown |
| **AS-8** content + coverage (D25) | Both JSON files with per-field negatives; `oracle_version`; ten-question validator | F1 `covered_fact (fixtures)`; others "Unsupported" |
| **AS-30** job substrate (D17, #1202) | Primitives into vigil_core (`make gate`); `envs/defense-exec`; `_PROBE` extended; per-tenant runner | Killed job degrades TT-A only |
| **AS-31** Detection Mirror | DM-a access records (UTC CLF, exact-value HMACs); DM-b login surface; policy violations; anomalies; **DM-signer** (cert v2, defense delegation replacing wildcard `OFFENSE_SPINE_ROLE` ingest at `finding_receiver.py:180-218`, D20); NAT/SSO corpora | `forced_browsing` shown; sitemap crawl silent |
| **AS-23a** early benchmark (D36) | `aegis_bench.py` (lazy, off the gate); commit-reveal corpora; arms A/B/G1/(G2); signed scorecard; CRS and claims gates | Witnessed run; holdout tamper detected |
| **AS-9a\*** broker | `connector_secrets.py` on consumer C's registry; Z2X broker | ExternalId pivot fails; Disconnect stops jobs |
| **AS-9b** receiver | `apps/aegis-ingest/` behind its own distribution; caps; HMAC/mTLS; buckets; landing dirs | Forged → 401; TT-A flood leaves **TT-B UI + billing webhook** unaffected |
| **AS-18a** 1b observe | §6 1b observe + DM-c | Provider-reported traversal; revoked token → "Not assessed" |
| **AS-34** production + rollout (D27) | Prod stacks (merged, **not applied**); key ceremonies (D30); `aegis-production-deploy.yml` (owner-dispatched, protected); runbooks; R-checklists | Dress rehearsal on a fresh production-shaped stack |
| **AS-9c** anchors | `asset_anchor.py`; entity table; validity intervals; WM only in the defense zone; graph-DB import guard | Duplicate anchor gets identical refusal |
| **AS-17a** Sigma v2 | Wildcard/logsource fix; author and licence; field maps; `not_deployable`; v1 byte-identical | Rule, author and licence shown |
| **AS-17b** CloudTrail | Read-only role CFN (distributed, D10); broker; Domain 7 defense entry point | Key creation, no-MFA login, StopLogging detected; wrong ExternalId refused |
| **AS-10a** provider runtime | GuardDuty findings incl. containers; prerequisite stated; enforcement asserted unsupported | Test findings shown [U trigger]; disabled detector shown |
| **AS-10b** EDR | One vendor via broker | Or "Blocked by D23 (vendor sandbox access)" |
| **AS-10c** kit | Forwarder (no `framework`); inert fixtures F5–F8, F11; built but OFF until D10/D23 | BTF / non-BTF / container cases |
| **AS-11a** incidents | Sole fold (guard); keys, basis, audit; ten answers | One incident per chain; TT-B gets "nonexistent" |
| **AS-11b** locker (D12) | DEK locker with CertRef (via invgraph S1) + RAMPART; 403 says "retained"; `audit_package` extended; TTL purge; legal hold | Offline verify; flip fails; holds respected |
| **AS-12** ledger | Eight states; Observe proposals; advisory capped at `requested`; conflicts; deadlines; mock guard; one fault test per class | Partial → uncertain → corrected |
| **AS-13a** executor (D17, D18) | Z2X; `defensive_contract`; single-use token; release covers the kind; conjunctive gate; destruction gate; TOCTOU | One negative per denial reason |
| **AS-13b** scope | Narrow, expiring; opt-outs; digest in the 403 | via 13c |
| **AS-13c** actions UI | Propose/approve/cancel/withdraw; preview | In-scope 403, out-of-scope observed; replay/TT-B/advisory refused |
| **AS-14** verification (D20) | Z3 judge; Z4 probes; security + workflow checks; 11-field record | All negatives in §2 §14 |
| **AS-21** virtual patching | `defense_link` (7 conditions); §6 flow; feedback guard | Full MERIDIAN chain; hardened twin → FIXED |
| **AS-18b** 1b enforce + 1b patches | SIGIL-tagged expiring rules; IP/ASN rules; read-back; Z4 probe; release required | Patch end to end; no cross-zone action |
| **AS-32** real-tenant verification | §6 real tenants | No logs → "not verifiable" |
| **AS-15** recovery + drift | `recovery_decide`; P1–P4; triggers; quarantine snapshot; intervention record | One test per trigger |
| **AS-16** lifecycle | Revoke → proofs → unpublish → retire → snapshot (holds) → delete → purge → release placement | Offboard within SLA; no resurrection |
| **AS-17c / 19 / 20 / 33 / 22** | IdP (default Okta); containment (A3, read-back, Z4 verifier); mirror IDS (D23); `notify_channels` exports + signed SIEM webhook; AI adviser (D26) | Per §2 rows |
| **AS-23b / AS-24** | Full benchmark; profiles validator, regenerated matrix, HANDOFF, five-state report | Full regression with fresh records |

### 9.4 Graph and derived waves

**Hot sets** (modifications only):

| Set | Files |
|---|---|
| GW | `A/{gateway,cli,monitoring_authority_check,monitoring_report,boundary}` |
| UI | Existing monitoring code in `provision/custui/app.js/guide.js` |
| SPOOL | `monitoring_spool.py` |
| FLEET | `aegis_fleet.py` |
| MA | `monitoring_authority.py` (S) |
| SC | Supply chain |
| INFRA | `infra/aws/platform` |
| CON | Contract evaluator |
| RBAC | `C/rbac.py` |

**Tokens:** VERIFY (`verify/`, `aegis/registry.py`), GP (gate path; needs `make gate` evidence). **wave(n) = 1 + max wave(B).** ↺ = ordering edge from a hot set or token.

| W | Slice: merge deps (B) |
|---|---|
| 0 | 0 |
| 1 | 25:0 · 1a:0 · 2a:0 · 7a:0 (SC) · 27a:0 (GW) · 3a:0 (SPOOL) · 4a\*:0 (UI) · 2l\*:0 |
| 2 | 2b:2a · 2i:2a · 2h:2a · 28:7a↺ (SC, VERIFY, GP) · 2j:1a, 2a, 27a↺ (GW) · 2f\*:2a, 4a↺ (UI, MA) |
| 3 | 2c:2a, 2j↺ (GW) · 6a:2a, 2j, 28 (FLEET) |
| 4 | 26:7a, 28, 6a (INFRA) · 2d:2c↺, 2j (GW) · 30:6a, 28↺ (GP) |
| 5 | 1b:1a, 25, 26 · 2g\*:2f↺, 2d (UI, MA) · 2e:2d↺ (GW) · 3c:3a↺, 2b, 26 (SPOOL) · 17a:30↺ (GP) · 9a\*:30, 2a · 6c:6a, 26↺ (INFRA) |
| 6 | 2k\*:2g↺ (UI) · 3b:2e↺, 3c (GW) · 9c:9a, 2b · 9b:9a, 3c↺ (SPOOL) |
| 7 | 21a\*:2k↺ (UI) · 5a:3b↺ (GW) · 17b:17a, 9c, 9b · 10b:9b, 30 · 20:9b, 30, 7a |
| 8 | 29\*:2c, 2f, 6a, 21a↺, 27a (UI, FLEET) · 5c:5a↺ (GW) · 10a:17b · 17c:17b, 9c |
| 9 | 3d\*:3b, 2c, 29↺ (UI) · 5d:5c↺, 29 (GW) · 6e:6c, 2c, 29↺ (FLEET) |
| 10 | 4b\*:3d↺, 4a, 29 (UI, CON) · 5e:5d↺ (GW) |
| 11 | 27b\*:2a, 27a, 4b↺, 6e↺ (UI, FLEET, RBAC) · 5b:5e↺, 6c (GW) |
| 12 | 6d\*:6c, 4b, 2g, 27b↺, 17a↺ (UI, GP) · 7b:5b, 7a, 6e, 28↺ (SC) |
| 13 | 8:7b, 4b, 6d↺ (VERIFY, GP) · 31:30, 3b, 7b · 10c:17a, 9b, 7b↺ (SC) · 34:6d, 6e, 3d, 4b, 6c↺ (INFRA) |
| 14 | 23a:7b, 31, 8↺ (GP) · 18a:9b, 31, 4b · 11a:5b, 8, 31 |
| 15 | 11b:11a, 23a↺ (GP) · 12:11a, 29 · 33\*:11a, 3d · 22\*:11a |
| 16 | 13a:12, 30, 27b↺ (RBAC) |
| 17 | 13b:13a, 7b, 5b↺ (GW) |
| 18 | 13c:13b |
| 19 | 14:13c |
| 20 | 21:14, 11b, 21a · 32:14, 9b · 19:17c, 13a, 14 |
| 21 | 18b:21, 18a, 13a, 14 · 15:14, 21 |
| 22 | 16:15, 11b, 6e, 9c, 2l · 23b:15, 21, 18b, 23a |
| 23 | 24:all |

**Chains:**

| Chain | Order |
|---|---|
| GW | 27a → 2j → 2c → 2d → 2e → 3b → 5a → 5c → 5d → 5e → 5b → 13b |
| UI | 4a → 2f → 2g → 2k → 21a → 29 → 3d → 4b → 27b → 6d |
| SPOOL | 3a → 3c → 9b |
| FLEET | 6a → 29 → 6e → 27b |
| SC | 7a → 28 → 7b → 10c |
| INFRA | 26 → 6c → 34 |
| GP | 28 → 30 → 17a → 6d → 8 → 23a → 11b |
| VERIFY | 28 → 8 |
| RBAC | 27b → 13a |

Every wave has at most one holder per hot set and per token. The AS-0 graph test asserts acyclicity, levels, disjoint manifests, token uniqueness, chain/edge agreement and R-gate prerequisites. It uses negative controls for a same-wave dependency, a shared hot file and a second token holder.

**Critical path:** 0 → 27a → GW chain → 5b → 7b → 8/31 → 11a → 12 → 13a → 13b → 13c → 14 → 21 → 18b/15 → 16/23b → 24. That is 24 waves. AS-G0 (§11.12) would shorten the GW chain.

**Milestones:**

| Milestone | Slices |
|---|---|
| M-0 | First AS-1b run |
| **M-1a** | 2a–2l, 3a–3d, 4a/4b, 5a–5e, 6a, 6c–6e, 7a/7b, 8, 21a, 25–29 |
| M-1b | 18a (observe); 18b (enforce) |
| M-5 | 10a (or 10b) |
| M-6 | 11a/11b |
| M-7 | 12, 13a–c |
| M-8 | 14, 21, 32 |
| M-9 | 15, 16 |
| M-10 | 17a–c, 19, 20, 33, 22 |
| M-12 | 23a/b |
| M-13 | 24 |

### 9.5 Alias map

| Final | Earlier names |
|---|---|
| AS-25 | AS-1e, R12 attestation |
| AS-2b | AS-2x |
| AS-2c | AS-2b1 |
| AS-2d | AS-2b2, AS-2o egress part |
| AS-2e | AS-2b3 |
| AS-2f | AS-2c1 |
| AS-2g | AS-2c2, AS-2o, threat-contract "AS-2e" |
| AS-2h | AS-2c3 |
| AS-2i | AS-2c4 |
| AS-2j | draft/order "AS-2d" |
| AS-2l | AS-2c5 |
| AS-3b | order "AS-3d" |
| AS-3d | AS-3e |
| AS-6a | order "AS-6b", AS-WRK |
| AS-7a | AS-7L |
| AS-7b | AS-7c + AS-7e |
| AS-9b | order "AS-9c" |
| AS-9c | order "AS-9b" + AS-9d |
| AS-10a/b/c | AS-10p/e/k |
| AS-11b | AS-11b + 11c |
| AS-13a/b/c | order 13b/c/d |
| AS-14 | AS-14a |
| AS-17a | AS-SG, AS-SIG |
| AS-18b | AS-18b + AS-21b |
| AS-24 | AS-24z |
| AS-26 | OPS-1 |
| AS-27a | BG, AS-BG |
| AS-27b | override part of AS-27 |
| AS-28 | AS-6i, DEF-IMG, AS-CLOSURE |
| AS-29 | AS-2r, AS-ER |
| AS-30 | AS-DJ |
| AS-31 | AS-DM |
| AS-32 | AS-14b, AS-PV |
| AS-33 | AS-EX |
| AS-34 | OPS-2 + AS-24a |
| (not used) | AS-1c: the static staging edge in AS-26 replaces it |

---

## 10. SIGIL operations

**Authority.** E1 covers this scope, but the HANDOFF record (AS-0) must land first. **No AEGIS production act is delegated.** Agents prepare evidence packets; the owner deploys, flips, places production tenants, signs releases and transitions stages.

**Fleet** (each with a health contract mapped to a tenant state):

| Component | Deployed by |
|---|---|
| Edge (CloudFront, Shield, WAF) | IaC + reconciler |
| Ingress / ALB | IaC |
| Units | Reconciler, signed digests |
| Cert issuer | Reconciler |
| Broker / prover | Reconciler |
| Relay | IaC |
| Drains and folds (`I/unit_alerts.py`) | Platform |
| Receiver | Platform |
| Connectors + broker | Reconciler |
| Sigma / Mirror jobs | Reconciler |
| Suricata | Reconciler |
| Executor | Platform |
| Judge | Platform |
| Content pipeline | Pipeline |
| Reconciler | Platform |

Retirement order: snapshot → stop → destroy credentials → delete edge tenant → release placement.

**Environments:** TT-CI → TT-STG (AS-26) → TT-P (SIGIL-owned canary in production) → tenants. A `build_id` is promotable only with records for **that** build and topology. Preconditions: slim image, licence records, egress rows.

**Updates and rollback:**

- Accept-sets ship before emitters.
- Rolling replacement per AS-6e (or the EDGE-SWAP pattern).
- Waves: TT-STG → TT-P → ≤5% → 25% → 100% (bake 24/24/48 h proposed). Automatic halt on error, latency or would-block deltas, heartbeat gaps, crash loops or cross-tenant signals.
- Rollback classes: image; content (signed, replay refused); mode (withdraw release); edge (change-set); data (N-1).
- A rollback that removes protection lists affected tenants and is shown to them. On-call may roll back alone only when no protection is removed.
- Advisory watch per engine; nightly Trivy; non-VALID digests refused.

**Monitoring:**

- Reuse `C/metrics.py` (`plane="defense"`), `vigil-alerts.yml` and the `unit_alerts` notifier. Tenant identifiers are opaque hashes.
- Proposed SLOs: visibility p95 ≤120 s; staleness ≤3 heartbeats; revocation ≤5 min (bounded by the lease); renewal ≥14 days before expiry; convergence ≤2 sweeps.

| Alert | Severity | Action → tenant sees |
|---|---|---|
| Heartbeat stale | SEV2 (SEV1 Defend) | Reconcile → "Coverage degraded" |
| Crash loop | SEV1 Defend | Snapshot, roll back the wave → "interrupted" |
| Lease renewal failing | SEV2/SEV1 at 50% of lease | Partition runbook → "expiring" |
| Revocation pending past SLO | SEV2 | Force push or trip → "pending" |
| Forged MAC / cross-tenant signal | SEV1 security | Contain; owner |
| Non-VALID image | SEV1 | Halt |
| Kill switch unreadable | SEV1 | → "paused" |
| Cert <14 d | SEV2 | → "expiring" |
| Reconciler drift | SEV2 | Inspect |
| Drain lag | SEV3 | → "delayed" |
| Spine ≥80% | SEV2 | Aggregate |
| Edge 5xx / Shield event | SEV2 | Runbook |
| Engine down | SEV3 | Restart → "Detection degraded" |
| Canary anomaly | SEV2 | Halt |
| Failed verification of an active action | SEV2 | Keep restricted and escalate |

**On-call (D32).** A staffed rota and a page tested end to end are required before P1.

- On-call alone, with `manage_defense`: trips, latch, release withdrawal, relocation, protection-neutral rollback, pausing the pipeline, scaling within budget.
- Owner only: releases, re-enabling, new placements, stopping a Defend unit (unless SEV1 cross-tenant), quota raises, Shield Advanced.
- SSM-only access; D8 two-person rule for plaintext; post-incident review within 5 business days.

**Capacity:** ≥30% headroom; Service Quotas alarms at 70% (CloudFront tenants, certs, WAF, VPC origins, EIPs); scale through IaC plus epoch-bumped moves. ECS is used only when D7 or measurements justify it.

**Self-incidents:** snapshot evidence → relocate → rotate all four secrets → revoke and re-mint → notify. The owner decides whether to stop the unit.

**Backup/DR:** `vigil backup` plus exclusions; after a restore, re-derive EpochStatements, push them and run the stray sweep. Drills monthly on TT-STG and quarterly on TT-P. RTO/RPO per D4/D2.

**Offboarding:** AS-16 cascade; legal hold blocks purge; the tenant removes their CNAME; re-onboarding needs fresh proof.

**Drills** (from Z4, inert fixtures, GATE-MARSHAL): kill switch and latch, revoke (including during partition), restore, rollbacks, partition, crash/OOM under Defend, noisy neighbour, cert renewal, controller restarts, edge misconfiguration, transport authentication, compromise tabletop. Each drill records its build and run.

**Rollout stages (owner acts; evidence packet = build, records, checks, attestations, licences, drills, headroom, on-call, D11 statement):**

| Stage | Entry | Owner act | Abort / rollback |
|---|---|---|---|
| R0 staging | 26/27a/28 merged; D1-record, D2, D3, D5, D6, D29, D32 | Authorize staging | — |
| P0 production platform | AS-34 stacks; Shield; paging; D8; analyzers | Dispatch | Teardown |
| P1 TT-P canary | Observe, then enforce under a release; all drills in production | Placement + release | Bake ≥14 d / ≥7 d |
| R1 observe 1a | Six flag-hold criteria; M-1a observe records; **D7**; D11; existing-edge check; D37 re-disposed; licence records | Flag + selected placements | Cross-tenant, SEV1 or bypass → unset flag, withdraw |
| R1b 1b observe | 18a DONE | Per-provider enable | Revoke token |
| R2 Defend per (org, host) | Enforce criteria incl. **same-shape FP review**; drills on the live unit; 5b, 21a, 12, **14** on TT-P; fail-closed accepted; HA; lock = crypto | **Sign release** (first release covers only VIGIL-confirmed classes) | Withdraw release |
| R3 slow path + virtual patching | 13a–c, 14, 15, 21 DONE; offline-verifiable records; D18 | Release per kind | AS-15 decides |
| R4 paths 2/3/4 | M-10/M-5; 11b; 16 | Per-path enable | Disable |

**Claims gate (E3d):** no comparative claim without a signed AS-23 scorecard; never "protected"; the lint covers the public site.

---

## 11. Parallelization and merge strategy

**Lanes:**

| Lane | Scope | Specialist |
|---|---|---|
| C | vigil_core contracts | CRYPTO-NOTARY |
| G | Gateway | GATE-MARSHAL, PROVER |
| S | Control plane | CRYPTO-NOTARY, CHRONICLER |
| U | SPA | CHRONICLER |
| E | Gate-path engine files | VERACITY-WARD, GRAPH-KEEPER, PROVER |
| D | infra/CI/supply chain | GATE-MARSHAL |
| T | Tests (AE-7 byte-for-byte untouched) | PROVER |

**Registries (class R; pure appends union-resolved at serial merge, with the guard battery re-run):** `config.py` flags, `ci.yml` offense list, `_PROBE`, `inventory.json` (+ `gen_assurance_mapping.py`), claims registry (new entries), `KNOWN_*` ledgers, Gate-5 census, `ui-wiring-matrix.json`, `V2-LIMITATIONS.md`, `spine_domains.py`, `_GOVERNOR_EVENT_WRITERS`, CODEOWNERS, build-manifest artifacts, standards mirror, appended routes/methods/scripts. Registrations ship **in the PR they cover**, because validators require the covered artefact to exist (`test_claims_registry.py:214-237`). A modification such as W6-1 makes the file hot for that PR.

**Concurrency is allowed only when all hold:** merge dependencies merged; build gates met; disjoint manifests; no shared hot file; no shared token. FORGEMASTER checks this against the slice-graph JSON.

**Agents:**

- One builder per slice in an explicit `/home/kali/vigil-wt-aegis-<slice>` (never `isolation:'worktree'`).
- Builders commit locally; the orchestrator pushes.
- Prompts are resume-safe; plans go in `~/.claude/plans/`.
- Separate reviewers: RED-PEN, six-lens sweep, specialists.
- Decisions recorded on the FORGE blackboard.

**Per-PR merge gate:**

1. Signed stage-0 charter (defensive fact, proof, benign twin, sovereignty, non-goals, FORGE domain).
2. PROVER `make gate` / `gate-zero-fp` diff against the AS-0 baseline (manual; no CI job runs it); recall-core reproduction test (`test_recall_baseline.py:44`); `test_gate_byte_identical.py` / `test_gate_invariants.py`.
3. **15** required checks green on the exact head; each new test mapped to its collecting job; TT-CI cases; flag-off golden.
4. RED-PEN **plus** the full six-lens sweep on every PR, AS-1a/1b included (INCOMPLETE = BLOCK).
5. GATE-MARSHAL + egress row + FATAL-2 review for probe-bearing slices (1a, 1b, 6d, 14, 21, 23a/b, 32); CRYPTO-NOTARY, GRAPH-KEEPER, VERACITY-WARD as applicable.
6. CHRONICLER entries.
7. Recorded human approval.
8. Owner merge.

**Line-by-line review (highest weight):** 27a, 25, 2a, 2c, 2d, 2f, 2g, 2h, 2j, 29, 3b, 3c, 26 (relay), 6a, 6c, 6e, 28, 5a, 5b, 7b, 8, 9a, 9b, 11b, 13a, 13b, 14, 18b, 19, 21, 27b, 31. Others get diff+test review; documentation gets a spot check.

**DONE gate:** AS-25 record (§13.3). Expires when the build or topology changes.

**Merge mechanics (D31):**

- Live protection has no required reviews; CODEOWNERS "does not gate".
- **Agents do not merge AEGIS-program PRs.** This is a self-restriction the standing authorization (`HANDOFF:89`) permits; the owner merges with `gh pr merge N --merge --match-head-commit <verified sha>` after CLEAN and all checks are terminal green. Never `--auto` (it merged PR-J with a red check). Merges are serial.
- D31 asks the owner to confirm, or to allow agent merges for low-weight slices.
- Recommended: agents author under a bot identity, plus the W12-3 code-owner flip. Either form must be recorded before the first highest-weight merge.

**Hidden guards (pre-push checklist):**

- `test_ci_cross_plane_tests_run_with_both_planes.py` + offense list
- `test_required_checks_canonical.py` (counts say 15)
- `make gate` byte-identity tests
- fold-cache job (`ci.yml:1036`)
- `test_two_env_boundary.py` / `assert_no_offense`
- claims tests + `test_claim_vs_enforcement_audit.py`
- limitations and assurance-mapping tests
- `test_latch_writer_shape.py`
- Gate-5 census; `ui-wiring-matrix` drift; pinned strings
- `test_customer_kit_copies.py`
- `gen_standards_mirror.py`
- `test_codeowners_coverage.py`
- mutation critical set (`test_mutation_critical_set.py`)
- `MonotoneFloor` + `_broken` twin
- `test_aegis_dockerfile_wired.py` + a new test for `Dockerfile.saas`
- supply-chain and licence negative controls
- sigil-lint
- gitleaks (inline allow with reason only)
- `test_aegis_slice_graph.py`
- `IT/test_aegis_tt_demonstrations.py`

**Other programs:**

| Program | Rule |
|---|---|
| Consumer train (#1196–#1202, #1186, #1190) | X-CT: slices that modify the customer surface wait; others proceed |
| C (connect) | 2l, 6d, 9a reuse it |
| D (notifications) | 33 after D-ff |
| E (deploy-retest) | 21a, 15 consume it |
| F / F-ff #1202 | Fairness reuse and 30 wait for it |
| H (SSO/SCIM) | 27b SCIM ceiling; 17c does not duplicate it |
| I (proof badge) | 11b extends `audit_package` |
| J/J2 | 22 after J2 |
| Investigation graph S1/S5 | 11b resolves through S1; **11a is the sole incident owner**; one per-host view with S5 |
| Legal-gov S1/S2/S3/S12 | 2i serializes behind S1; shared consent and lifecycle model; S12 renders v2 grants |
| Front C S1 | 3c owns a scoped measurement |

**D28 widened** to aegis, sigil, vigil_core, infra, `integration/tests`, `.github`, the Gate-5 test and `ui-wiring-matrix`.

**A merge never deploys to production.** `workflow_run` never targets production. Staging may auto-deploy. Production is `workflow_dispatch` in a protected environment [U plan tier].

**Accelerators (owner opt-in, not assumed in the waves):**

- **AS-G0:** zero-logic-diff split of `AegisGatewayHandler` (`:302-792`) after 27a, AST-equivalence checked.
- **AS-4s:** move monitoring code into AEGIS-section modules.

**Refuse in review:**

- **Platform shape:** a separately deployed AEGIS, or any second control plane, tenant model, RBAC, billing or UI.
- **Authority and policy:** a second authorization record; enforce rendered from anything but a release; a placement or quota service beside the spine.
- **Ingest:** a second ingest bus; ingest on `uiproxy`.
- **Orchestration:** a second orchestrator; defensive connectors in the offensive runner.
- **Health and evidence:** a second health or metrics stack (incl. AGPL Grafana/Loki/MinIO on customer paths); a new evidence format other than AS-25; a second incident fold.
- **Edge and egress:** any egress predicate other than `common/egress_pin`; edge config not compiled from grants × placement; a second edge or own anycast.
- **Engines and content:** shared multi-tenant parsing processes; raw SecLang; pySigma; Falco responders.
- **Images:** non-allowlisted `framework` modules in the defense image or kits; ports from P.
- **Tests:** edits to AE-7; payloads from Z2/Z3.
- **Honesty:** mock enforcement success; any `protected` state; unearned comparative claims.
- **Process:** agent merges or `--auto`; a tenant-supplied CIDR or any declaration signed without a proof; loopback or peer address as identity; a production egress plane running on the stdlib backstop; Z3 contacting tenant endpoints.

---

## 12. Deliverables (spec §28)

| Item | Delivered by | Evidence |
|---|---|---|
| 1 Audit and matrix | §2 → AS-0 → AS-24 + trace JSON | Public subset + drift test |
| 2 Code and migrations | All slices; v1→v2 dual-accept | Records |
| 3 Engine rationale | E1 ADR (Mirror before Sigma; edge; 1b), 7b | Ledger |
| 4 Threat model | AS-0 (§4) | Split + drift |
| 5 Profiles (SaaS only) | AS-24 validator (ten §20 fields) | Doc test |
| 6 Coverage | AS-8 | Validator negatives |
| 7 Observe/Defend | 4b, 5a–e, 13a–c, 29, 2k | Records with UI legs |
| 8 Contracts | 2a, 9a, 12, 13a, 29 ("provisioner-signed") | Records |
| 9 Verification | 14 + **32** | Records |
| 10 Failure states | 3*, 5*, 15, 27a + fault tests | Records |
| 11 UI/API | Every slice; Gate-5 | `ui_legs` |
| 12 E2E | 1a/1b + **25** | Current records |
| 13 Benchmark | 23a/b vs **G1/G2** | Signed scorecard |
| 14 Licences | 7a before first deploy (TB9) | `use_mode` gate |
| 15 Runbooks | 26, 34, 24 incl. **the rollout procedure**, rehearsed | Rehearsal record |
| 16 Evidence export | 11b + retention / legal hold | Offline verify; flip fails |
| 17 Unsupported | §15, `LIMIT-aegis-*`, five-state projection | Drift |
| 18 PRs, handoff | All; 24; HANDOFF §6 (E1–E3) | 15 checks |

Dropped: offline bundles, operator keys, sovereign packaging, customer-installed gateway. §20.5.3 is re-mapped.

---

## 13. Final-acceptance standard

### 13.1 DONE

TT-A (with TT-B co-located) uses the capability end to end through VIGIL on TT-STG built from merged IaC, at production grade, proven by an AS-25 record. Go-live is a separate owner act.

### 13.2 Why the registry alone cannot hold DONE

`validate_claim` (`test_claims_registry.py:197-237`, re-read) requires `ci_job` to be a **required** PR check that collects the proving file (negative control at `:371-375`). TT-STG is a dispatch/schedule job, so a `proved_by` entry can never point at it. The registry **stays unchanged**.

### 13.3 The record (AS-25)

- **Run record:** `tools/verification_manifest.py run` → `vigil-verification-manifest/1` (`:138`). `check()` (`:174`) re-derives counts from the bound JUnit and rejects skips of covered terms. It is already tested via `IT/test_release_gate_manifest_binding.py` in P5.
- **Binding** `vigil-aegis-tt-demonstration-v1`:
  - slice, `claim_ids`, run, commit;
  - manifest and JUnit digests, suite digest;
  - `build{build_id, artifacts VALID}` from the signed build manifest;
  - `topology{class: iac, digest}`;
  - `subject{paths, digest}`;
  - tenants;
  - `ui_legs[{route, screen, session}]`;
  - `required_cases{positive, benign, benign_same_shape, negative, cross_tenant, failure_class}`;
  - `no_skip_terms` ⊇ {TT-B, cross-uid, browser, isolation, tenant}.
- **Signing:** domain-tagged Ed25519 via `verify_threshold` (`C/crypto.py:111,153`), tag `aegis-tt-demonstration`, by a **TT evidence key in the staging control plane** (KMS-wrapped). It is never a runner key: `release.yml:19-21` records why the repo avoids long-lived runner keys. An optional keyless cosign bundle may be added for third parties. Trust root at `docs/evidence/aegis-tt/trust-root.json`.
- **Verifier** `IT/test_aegis_tt_demonstrations.py`, collected wholesale by P5 (vigil_core installed at `ci.yml:325`). It checks:
  - signature under a non-production root;
  - digests match + `check()` passes;
  - claims are valid under the unchanged validator;
  - all case classes present, `ui_legs` non-empty, every route in the **Gate-5 census**;
  - artifacts VALID;
  - staleness by subject/topology digest (stale is a state, not a CI failure);
  - **CI fails if anything is labelled "demonstrated" without a current IaC record**.
  - Negatives: forged signature, production signer, flipped JUnit, dirty run, skipped TT-B, missing same-shape benign, empty legs, unknown claim, hand-configured topology, stale record.
- **Projection:** `docs/capability-matrix/aegis-saas-status.json` carries per-claim `demonstrated_by` and the five states, with a drift test.
- **Publication:** a PR containing only passing records, merged by the owner. Failed runs go to private ops (`HANDOFF:83`).
- **Limits:** this is SIGIL attesting its own staging run. It is not production behaviour and not third-party assurance.

### 13.4 Advancement and the E1 bar

**Advancement:**

1. Merge at TT-CI.
2. Dependents may build on merged slices.
3. TT-STG runs are cumulative.
4. Only IaC topology counts.
5. DONE requires current records for every claim; no R-gate without it.
6. "Blocked" only for external prerequisites (§14.3).
7. **Program DONE** = every slice DONE or externally blocked, **and** the rollout procedure rehearsed and recorded.

**Production bar per capability:** TT-STG positive/benign/negative through VIGIL; the §13.8 failure rows green; alerts, runbook and per-engine health; upgrade and rollback with zero loss; fairness under a noisy neighbour; licence records; the rollout step defined; honest UI.

### 13.5 Ten questions per claimed protection (validator in 8/24; plus the six §1 aspects)

Example: TT-A, path 1a, `sqli_attempt`.

| Question | Answered by |
|---|---|
| Observe? | Contract telemetry + checks + heartbeat |
| Detects? | Content entry + coverage row |
| Evidence? | Report v2 + locker CertRef (offline) |
| Authorized? | v2 epoch/capabilities + **release** + token |
| Enforce where? | Unit/stage placement (CloudFront/Shield is **not** an AEGIS enforcement point) |
| Took effect? | Receipt + `observed_applied` ("not independent") + Z4 probe |
| Prevented? | Security result "within tested coverage" + `outcome_class` |
| Legitimate operation OK? | Workflow result incl. same-shape corpus |
| Blind spots? | `coverage_limitations`, `trust_assumptions` |
| Re-check when? | `condition_binding`, lease, staleness |

### 13.6 Differentiators and claims

- **Virtual patching:** one record for the §6 chain (FACT → proposal → approved scope under release → in-scope block with benign pass → `STILL_VULNERABLE + mitigated_by_edge` → offline export, flip fails).
- **Verified outcomes:** an offline `ResponseVerificationRecord`.
- **Comparative claims** ("rivals Cloudflare", "better than <WAF>") only with a signed AS-23 scorecard against G1 on the same scenarios.
- **Forbidden terms (D13):** universal detection/prevention; zero FP/FN; near-zero false positives; complete MITRE coverage; permanent protection; fully protected; all threats blocked; `protected` as a state.

### 13.7 Five states (generated)

| State | Rule |
|---|---|
| Implemented and demonstrated | Valid claim **and** current IaC record |
| Implemented but not validated | No record, or record stale |
| Partially implemented | Some claims of the slice missing |
| Unsupported | Manifest says so, with reason |
| Blocked | `LIMIT-aegis-*` entry naming a §14.3 prerequisite and its check |

### 13.8 Test categories (TT-B always the control)

| T | Slices | T | Slices |
|---|---|---|---|
| 01 | 7b/8 | 13 | 12/13b |
| 02 | 8 | 14 | 12 |
| 03 | 5b/3b | 15 | 3c/12 |
| 04 | 1a/2j | 16 | 15 |
| 05 incl. partition | 2c/3d | 17 | 15 |
| 06 | 2g/13a | 18 | 7b/23a |
| 07 | 3c/9b | 19 | 5b/6c |
| 08 | 3a/3b | 20 | 22 |
| 09 | 3d | 21 (fail-open zero) | 5a/15 |
| 10 | 3c | 22 | 2c/7b/13b |
| 11 | 3c/5a/9b | 23 partition | 3d |
| 12 | 3c/12/13a | 24 | 2i |

**Baseline today:** 1a observe and enforce are partial (in-process; HOLD). 1b, 2, 3 and 4 are unsupported. Incidents, orchestration, verification, drift handling and DONE records do not exist.

---

## 14. Owner decisions and blockers

### 14.1 Decided

| Id | Settles | Recorded / verified by |
|---|---|---|
| **E1** | D1. Supersedes 2026-11-16 for this program; Contabo out; no demand-driven deferral; program DONE includes the rehearsed rollout | AS-0 HANDOFF §6 entry; memory note updated by the owner |
| **E2** | D2 core. Unified platform; EKS control plane superseded | No EKS clusters; analyzers clean; one key per zone; disjoint EIP sets; one custui |
| **E3** | D3 edge; D22 1b-first (Cloudflare). Replaces the SNI router | AS-0 record; distribution-tenant API; 23 scorecards; lint |

### 14.2 Open decisions

| Id | Decision | Recommendation / verification |
|---|---|---|
| D2r | Region; Z4 account; runtime form; Nitro [U] | Z4 behind its own account (AUP blast radius, K37); `list-accounts` |
| D3 | Shield Adv; distribution model; ALB vs direct VPC origin, HTTPS hop; origin-auth forms; gateway-down defaults; cert validation | Defend fails closed; Observe failover opt-in; edge secret + VPC origin; test that a direct hit is refused |
| D4 | Quotas, overload values, HA, RTO/RPO, memory caps | Owner (pricing); `catalog.py:84-90` diff; load test |
| D5 | Transport | Keyless relay; no inbound port on Z3 (`ss -ltn`) |
| D6 | Test resources, budget, **TT evidence key** | Provision via AS-26; fingerprints differ from production |
| D7 | Kernel isolation | Analysis; **R1 gate**; not an AS-2 dependency |
| D8 | Operator access policy | Before R1; SSM audit |
| D9 | Spine capacity; unflagged crash fixes | AS-3c measures |
| D10 | Licences and customer terms; benchmark terms; ET; Cloudflare/AWS clauses | Counsel; gate negatives (FORGE reading moved to D29) |
| D11 | Trust statement | AWS + unit plaintext; region; IP allowlist ≠ isolation |
| D12 | Custody, retention, legal hold | DEK; TTL and hold tests |
| D13 | Vocabulary; **unflagged overclaim removal** incl. 403 text; catalog wording | Approve; lint negatives |
| D14 | Mode legs; override channel | Sovereign, audited, downgrade-only |
| D15 | Staging enforcement via the test release key | Approve; flag-hold still HOLD |
| D16 | v2 subject; governed entitlement | Generalize; governed only via release |
| D17 | Executor env; primitives move | AS-30 |
| D18 | A3 approvers; `manage_defense`; quorum | Step-up + token + release; `manage_defense` never grants |
| D19 | Block without an oracle? | No; alert-only breaker |
| D20 | Verifier and **defensive-evidence signers (incl. Mirror delegation)**; probe vantage; real-tenant checks | Distinct keys; Z4-only payloads; existing signed authority |
| D21 | Coraza connector, pins | Maintainer evaluation; tags |
| D22 | Path-2 IdP (**owner names before 17c**; default Okta [U Entra P1/P2]); second 1b provider; containment | Connector records |
| D23 | Runtime (provider → EDR vendor → kit); mirroring; never IPS | Records or §14.3 entry |
| D24 | File scanning | Defer |
| D25 | `oracle_version` hole | Approve |
| D26 | AI consent | Consent-first; shared with legal-gov S2 |
| D27 | R-stages; **production CD owner-dispatched** | Protected-environment logs |
| D28 | Widened rebase guard; `make gate` baseline | `git diff --stat cb5d20b5..origin/main -- <paths>`; gate verdicts + corpus sha256 |
| **D29** | FORGE §1 reading for the SaaS (§0.3 item 7 a–f) | Before 26, 1b, 2f, 2h, 6c; PROVER boundary test |
| **D30** | Release-root custody and ceremony (also signs the edge catalogue) | Hardware / m-of-n; before 29 |
| **D31** | Merge authority (§11) + W12-3 + bot identity + CODEOWNERS | Before the first highest-weight merge; `gh api …/protection` |
| **D32** | On-call staffing and paging | Before 26 paging, P1 |
| **D33** | DONE-evidence schema extension and evidence key | Before 25 |
| **D34** | Closure mechanism (lazy `verify` + `make gate` vs inert allowlist) | Before 28 |
| **D35** | Lease (1 h) / renew (10 min); re-proof (24 h, 3 fails / 1 h); partition behaviour = R16 | Before 2a/2c/2g |
| **D36** | Benchmark arms; FP threshold; comparative-claim policy | Before 23a |
| **D37** | Re-dispose the SaaS SSRF alert | After 2d/2g; R1 item; owner only |
| **D38** | Acknowledge §26 concurrency (strict fallback otherwise) | Before concurrent step-5/10 nodes |
| **D39** | Shared-origin exception review; `hard_guardrail` for defensive forwarding | Before 2g |
| **D40** | Cross-program owners: incident entity (11a), consent store, data lifecycle | AS-0 record |

### 14.3 External prerequisites (the only valid "Blocked")

| Prerequisite | Verification |
|---|---|
| Counsel rulings (D10/D11) | Signed memo |
| AWS quotas [U] | `aws service-quotas list-service-quotas` |
| AWS AUP for offensive testing | Acknowledgement on file |
| Cloudflare test zone and token | API call succeeds |
| IdP / EDR developer tenants; AWS test accounts | Connector health |
| Tenant authorization and origin logs (real-tenant `outcome_verified`) | — |

### 14.4 Technical blockers

| Id | Blocker | Cleared by |
|---|---|---|
| TB1 | No edge 429s before attribution | 5b |
| TB2 | No second co-hosted tenant before isolation, binding, crypto lock and non-loopback hop | 2j/2d/2g |
| TB3 | No production block before replay fix, carried floor, leases and acknowledgement | 2a/2c/3d |
| TB4 | No customer hostname before probe socket and zero-downtime upgrade | 5c/6e |
| TB5 | No enforce drill (staging included) before break-glass | 27a |
| TB6 | No enforce with retests before the guard | 21a |
| TB7 | No Defend tenant before fail-closed, memory bound and crash-loop state | 5a/2j |
| TB8 | No production enforce without a release | 29/D30 |
| TB9 | No engine deploy before licence ledger and slim image | 7a/28 |
| TB10 | No "demonstrated" or R-gate without records | 25 |
| TB11 | No connector holding a shared principal | 9a |
| TB12 | No ingest on `uiproxy` | 9b |

---

## 15. Unsupported, out of scope, and risks

**Out of scope (owner):** on-prem, offline and sovereign packaging; customer-installed gateway or SDK; Contabo; own anycast (deferred); Kubernetes without a trigger.

**Unsupported:**

- Inline IPS.
- Volumetric DDoS protection off the SIGIL edge (routed hosts only, per Shield tier [U]; 1b keeps the provider's protection).
- Windows/macOS runtime except via EDR.
- On-prem networks without a sensor.
- File scanning (D24).
- Zeek.
- IOC matching.
- WebSocket, HTTP/3 to origin [U], chunked bodies until verified, over-cap bodies.
- F2, F13, F15 events.
- Runtime enforcement.
- Automatic finding→block.
- AI in any trust root.
- Unearned comparative claims.

Hosted engines cannot observe systems they are not connected to.

**Risks:**

| Id | Risk | Mitigation |
|---|---|---|
| K1/K36 | SIGIL and AWS become availability dependencies; rented-pipe terms change | Shield, D3, status visibility, rollback; 1b keeps the customer's provider |
| K2 | Kernel escape | D7 (R1) |
| K3 | Insider or AWS sees plaintext | D8, D11 |
| K4/K21/K25 | N-A, T-XO, co-host pivot | TB2 |
| K5/K23 | Replay; undelivered revocation | TB3 |
| K6 | Collapse / forged viewer header | 5b |
| K7 | SSRF | 2d, D37 |
| K8 | Spine saturation | 3c, D9 |
| K9 | Licence non-compliance | 7a, D10 |
| K10 | Coraza connector maintenance | D21 |
| K11 | Overclaiming | D13 + scorecard gate |
| K12/K35 | Out-of-scope server work; CD to production | E1; D27 |
| K13 | Offense coupling / closure pulls `verify/` | E2 zones, 28 |
| K14 | False FIXED | 21a |
| K15 | Evidence privacy | 11b |
| K16/K24 | Shedding evasion; induced fail-open | TB7 |
| K17 | Provider latency read as real time | Latency class shown |
| K18 | Test traffic to third parties | SIGIL-owned targets, Z4 |
| K19 | Parallel systems | §11 refusals |
| K20 | Porting from P | AS-0 freeze |
| K22 | Machine enforcement | TB8 |
| K26 | Caps used as DoS amplifier; issuance starvation | Sub-caps, budget |
| K27 | Shared connector principal | TB11 |
| K28 | Ingest flood | TB12 |
| K29 | Transfer client attacked | Relay |
| K30 | Misattribution | `rule_refs` |
| K31 | Same-shape benign blocked | Corpus + opt-outs + R2 review |
| K32 | Green-washed DONE | Signed records + staleness + owner merge |
| K33 | Train collision; long UI/GW chains; X-CT delay | D28, X-CT, AS-G0 |
| K34 | FORGE conflict left implicit | D29 |
| K37 | AUP enforcement hits the platform | Z4 own account |
| K38 | CDN silently removed | 6d |
| K39 | Unoperated workers | 6a/30 |
| K40 | Stateless `fair_queue` reused | #1202 |
| K41 | AWS DDoS simulation restrictions | Low-rate tests only |
| K42 | Release-key loss | D30 ceremony |
| K43 | Prover / broker compromise (A9) | Lock + uniqueness |
| K44 | Stale DONE after build/topology change | Projection |

**Sources (generic queries):**

- [CloudFront multi-tenant distributions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-config-options.html)
- [SaaS Manager](https://aws.amazon.com/blogs/aws/reduce-your-operational-overhead-today-with-amazon-cloudfront-saas-manager)
- [Tenant certificates](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/managed-cloudfront-certificates.html)
- [Tenant customization](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/tenant-customization.html)
- [VPC origins](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-vpc-origins.html)
- [Cross-account VPC origins](https://aws.amazon.com/about-aws/whats-new/2025/11/amazon-cloudfront-cross-account-vpc-origins)
- [Viewer-Address header](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/adding-cloudfront-headers.html)
- [Origin mTLS limits](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/origin-mtls-authentication.html)
- [Edge IP ranges](https://docs.amazonaws.cn/en_us/AmazonCloudFront/latest/DeveloperGuide/LocationsOfEdgeServers.html)
- [Shield pricing](https://aws.amazon.com/shield/pricing/)
- [WAF rate-based keys](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based-aggregation-options.html)
- [CloudFront quotas](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html)
- [GuardDuty Runtime Monitoring](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html)
- [Cloudflare custom rulesets](https://developers.cloudflare.com/waf/custom-rules/custom-rulesets)
- [Cloudflare Logpush datasets](https://developers.cloudflare.com/logs/logpush/logpush-job/datasets/)
- [DRL-1.1](https://github.com/SigmaHQ/Detection-Rule-License)
- [CRS engines](https://coreruleset.org/docs/deployment/engine_integration_options)
- [Vectorscan](https://github.com/ProBrian/vectorscan)
- [BPF licensing](https://docs.kernel.org/7.3/bpf/bpf_licensing.html)
- [ET licensing](https://community.emergingthreats.net/t/licensing-of-third-party-rules/2396)

---

## 16. Critique resolution log

### 16.1 Integrator choices where finalizers conflicted

C1–C27 were each resolved toward the safer or more honest option.

1. **Slice ids.** Adopted the per-slice bodies' ids. The graph rules came from the order finalizer. Waves were re-derived (§9.4) and aliases recorded (§9.5).
2. **Decision ids.** Consolidated to D29–D40. Former D-FORGE1/D-F1/D-FRG became D29; D-EV became D33; D-MRG became D31; D-BENCH became D36; D-SSRF became D37; D-CD was folded into D27; the Mirror signer moved into D20. A D29 that had meant on-call became D32.
3. **DONE evidence.** Reuses the existing manifest plus a binding. The signer is the staging control plane, not a runner key (`release.yml:19-21`). The registry is unchanged; its validator only checks required fields. The verifier runs in P5, not the briefing job, because vigil_core is installed in P5.
4. **Origin proof.** Forward-path challenge only. "Name under the proven domain" was rejected as sufficient because of the CNAME attack.
5. **Address overlap.** Allowed only with an exact (Host, SNI) proof plus review. A blanket refusal was rejected because it would block legitimate shared PaaS customers.
6. **Unit egress.** Chose the per-host broker over per-unit "any global :443", so a compromised unit cannot pick its destination. `_FloorPinnedBackend` was rejected because of its lifts at `:282-290`.
7. **Ingress.** VPC origin → ALB → ALB-only front → `PrivateNetwork` gateway over a `SO_PEERCRED` socket. This is the union of four designs.
8. **Staging edge.** Static CloudFront in AS-26 from the first run, instead of a private path plus a later edge leg (AS-1c dropped).
9. **Break-glass.** Minimal AS-27a in W1 instead of after the infra head; the override is split into AS-27b.
10. **Org proof cascade.** Early (AS-2l), not inside AS-16.
11. **Merge.** Agents self-restrict and the owner merges. D31 records this, without asserting that the owner's standing authorization is revoked.
12. **Lease lapse.** Pass-through (R16) instead of "apply the tenant recovery policy".
13. **Outcome classes.** Five, with an explicit mapping, instead of collapsing withholding into "partial execution".
14. **IdP.** The owner names it; default Okta. One finalizer's Entra default became the alternative.
15. **World model.** The control plane never imports the offense WM (a finalizer's rejection was adopted).
16. **Placement.** A new record class reusing the mechanics; v1 is frozen (`extra="forbid"`).
17. **Verifier judge** sits in Z3 (two finalizers) rather than Z2.
18. **ui-wiring-matrix scope** was corrected to operator surfaces only (`generated_from` re-read).
19. **Required checks** = 15 (re-read).
20. **Zones.** Z2E/W/X segments plus component names.
21. **Lease timing.** 1 h lease with 10 min renewal (the shorter of the two proposals).
22. **New-module convention** plus class R. AS-4s became optional, not planned.
23. **AS-7b** joins in the fold, with no `gateway.py` edit.
24. **Threat-id aliases** unified.
25. **D37** comes after the fix; it is not a start gate.
26. **`catalog.py`.** Lint covers rendered strings only; whether those lines render is [U].
27. **Observe overload.** Shedding is disclosed; unit-down gives no failover unless opted in.

### 16.2 RED-PEN (23 objections)

| # | Objection | Disposition |
|---|---|---|
| 1 | Cross-tenant relay | ACCEPTED. §4.4 binding, broker and lock; T-XO; negatives. "Name under domain" and "refuse overlapping addresses" partly REJECTED (C4, C5) |
| 2 | Blocking without owner authority | ACCEPTED. AS-29, D30, sixth term, labels |
| 3 | DONE vs registry | ACCEPTED. AS-25 (C3) |
| 4 | DONE through VIGIL | ACCEPTED. Routes and screens per slice; Gate-5 |
| 5 | Revocation latency | ACCEPTED. Leases, acknowledgement, partition negative |
| 6 | Induced fail-open | ACCEPTED. MemoryMax, Defend 503, opt-in, crash-loop |
| 7 | Co-host network | ACCEPTED. R15 |
| 8 | Noisy neighbour / ACME | ACCEPTED. L0–L2, NAT, issuance budget. Per-SNI router caps moot: no SIGIL router exists under E3a |
| 9 | Connector pivot | ACCEPTED. Broker |
| 10 | Ingest on uiproxy | ACCEPTED. AS-9b |
| 11 | Egress predicate | ACCEPTED. No `C/egress_policy.py` |
| 12 | Closure vs frozen gate | ACCEPTED. AS-28, D34 |
| 13 | Floor reset | ACCEPTED. EpochStatement |
| 14 | Open-ended paths | ACCEPTED. Defaults (§6) |
| 15 | Orchestration / D7 | ACCEPTED. Memo corrected; H1; D7 at R1 |
| 16 | Probe vantage / inert fixtures | ACCEPTED. Z4 only; inert, attested fixtures |
| 17 | Benign shape | ACCEPTED. Corpus, opt-out, preview |
| 18 | Incident / placement duplication | ACCEPTED. AS-11a sole owner; C16 |
| 19 | AE-7 / uid escape | ACCEPTED. New files; mandatory uids |
| 20 | Licence `use_mode` | ACCEPTED. AS-7a |
| 21 | Contradicted claims | ACCEPTED (15 checks; engine names). Port-80 moot under E3a |
| 22 | `contributing` | ACCEPTED. `rule_refs` |
| 23 | Rsync direction | ACCEPTED. Z3R |

### 16.3 FORGEMASTER (20 objections)

| # | Objection | Disposition |
|---|---|---|
| 1 | Waves vs deps | ACCEPTED. §9.4 mechanical + graph test |
| 2 | Shared files | ACCEPTED. Hot sets, tokens, class R. A separate registration PR per wave REJECTED: validators require the artefact to exist |
| 3 | Infra after DONE | ACCEPTED. AS-26/1b head; AS-6 split |
| 4 | Human approval / self-merge | PARTLY ACCEPTED. Owner merges as a self-restriction; D31 because of `HANDOFF:89` |
| 5 | Dual attestation | ACCEPTED. Every PR |
| 6 | `make gate` / 48 | ACCEPTED. Baseline, manual PROVER diff, 15 checks, `secure_write` untouched |
| 7 | DONE / advancement | ACCEPTED. §13.3–13.4 |
| 8 | Break-glass late | ACCEPTED. AS-27a in W1; TB5/TB6 bind staging; D15 |
| 9 | FORGE §1 | ACCEPTED. D29 gates |
| 10 | Defensive-only probes | ACCEPTED. Rule 6 |
| 11 | Slice sizing | ACCEPTED. Splits |
| 12 | FORGE recipe / domains | ACCEPTED. Charter; map; Domain 7/8 reuse. "Sigma only where no oracle exists" PARTLY REJECTED: posture oracles do not detect events |
| 13 | §26 adaptations | ACCEPTED. §9.2 + D38 |
| 14 | Licence / image late | ACCEPTED. W1/W2 |
| 15 | AS-10 bundling / substrate | ACCEPTED. 17a, AS-30 |
| 16 | Program collisions / D28 | ACCEPTED. X-CT, #1202, widened D28 |
| 17 | Unreconciled decisions | ACCEPTED. D37, D27, D7 |
| 18 | Truthful UI flag / 403 | ACCEPTED. D13 unflagged; 403 in 27a |
| 19 | Hidden guards / ci ordering | ACCEPTED. §11 list; same-PR registration |
| 20 | (verdict) | Resolved by 1–19 |

### 16.4 COMPLETENESS

Every item below was ACCEPTED, located as listed, except where noted.

- DONE evidence → AS-25.
- Existing functionality → §2.E, AS-31.
- In-flight programs → §11.10.
- Sigma coupling → 17a.
- Every worker operated → 6a/30/3d.
- §13 failure classes → AS-12.
- Real-tenant verification → AS-32.
- §10 existing protection → 6d.
- E2E for 18/19/20/23 → §9.3.
- EDR and 1a-before-1b → 10b; E3b.
- Retention and legal hold → 11b/16.
- Mode per tenant / group → 2k.
- Observe proposals, mode switch, preview → 12/13c/4b.
- T-map → §13.8.
- Mock guard → 12.
- Method rows and policy violation → §2 §7.
- Content and scenario fields → AS-8.
- F5–F12 tests → 10a/10c/20/5d.
- `not_deployable` and stale content → 17a/7b.
- Per-engine evaluation and CRS gating → §7, 23a.
- Zeek, IOC, Tetragon reasons → §7, §15.
- 13.1c → 13b/18b.
- Four outcome classes → five with mapping (C13).
- Denial tests → 13a.
- Step-up, rotation, `manage_defense`, override → 2f/2h/27b.
- Record fields → 14/15.
- §15 behaviours and evidence-before-cleanup → 15/6a/16.
- §16 triggers → 15.
- Correlation keys and ten answers → 11a.
- Workflows and criticality → 2k.
- §4 entities → 9c. "WM partitions on the control plane" REJECTED (C15).
- Envelope fields and metrics → 2b/3d.
- Fairness tests → §8.2.
- Engine health → 3d.
- Per-path onboarding → §6.
- Licence dependency sets and TI → §7.
- Z3 hosting → E2.
- Framework refusal → AS-28.
- Profile fields and export → 24/33.
- §20.5.3 → re-mapped.
- Public vs private → §4.7.
- Benchmark metrics → 23.
- §17 uses → 21/8.
- Containers → 10a/10c.
- Trace JSON → AS-0.
- AGENTS.md → recorded absent.
- STUB/DEAD → labels.
- 48→15 → corrected.
- Term list → §13.6.
- 19.14 → row.
- AI controls → 22.
- `catalog.py` byte-identity → PARTLY ACCEPTED: lint scoped to rendered strings, [U] whether they render (C26).
- Ten-question validator → 8/24.
- Advisory, transport and certificate tests → 7a/7b/26/6c.

### Critical Files for Implementation
- /home/kali/vigil-wt-guardsweep/engine/crucible/framework/v2/aegis/gateway.py
- /home/kali/vigil-wt-guardsweep/apps/sigil/sigil/ui/provision.py
- /home/kali/vigil-wt-guardsweep/packages/core/vigil_core/vigil_core/monitoring_authority.py
- /home/kali/vigil-wt-guardsweep/engine/crucible/framework/v2/common/egress_pin.py
- /home/kali/vigil-wt-guardsweep/tools/verification_manifest.py

---

## Appendix Z — prior plan stored in this file (APEX), preserved verbatim; NOT part of this approval

# APEX — the Autonomous Sovereign Pentester (3 parallel, production-grade PRs)

## Context

The operator wants to leapfrog Strix: not a bolted-on autonomous agent whose output is untrusted
leads, but a **native autonomous pentester whose every finding is oracle-confirmed, OOB-attested,
proof-carrying, and governed** — fully built and tested to production grade, delivered as **3 parallel,
end-to-end PRs** (no MVP). The winning move is the *combination* nobody else has (autonomy × signed OOB ×
FACT × governance), not out-autonomy-ing Strix on raw reach.

**Exploration finding that shapes everything:** most of this is already merged. This is **promote +
harden + wire + close specific gaps**, per the "reuse, don't reinvent" doctrine — NOT greenfield.
- OOB (VF-2b signed collector) is built end-to-end — canary→plant→capture→sign→FACT works today for
  `ssrf, blind_xxe, xxe, rce, command_injection, deserialization/JNDI, oob_sqli`, with the pinned-
  collector-key triple-lock and an owner-signed authority gate. There is even `test_oob_autonomous_feeder.py`.
- The autonomous OODA loop exists (`engage_autonomous.run_autonomous_cycle`) and the recon→test bridge
  (`intel/promote.py`, the merged "Prover→Discoverer" program) is wired — but the loop runs as a **post-scan
  advisory appendix with a throwaway executor**, not the engagement's **primary driver**.
- The governance/mint/meter/PCR invariants are fully mapped (below).

## The 5 invariants every PR must honor (guardrails — do NOT violate)

1. **Mint:** a FACT is minted ONLY via `oracle_adapter.certify_admitted → confirm_and_certify`
   (`integration/vigil_integration/oracle_adapter.py:113,178`) on a *fired, oracle-mapped,
   reproduced/live_redrive-provenance* verdict with ≥1 signer. LLM/RL/critics are demote-only
   (`verify/firewall.py:16-17`). Enforced by `scanner/tests/test_claim_discipline.py:385`.
2. **Gate:** every target-touching action flows through `conjunctive_gate.build_offense_gate →
   vigil_core/gate.py:conjunctive_decide` (kill-switch→window→scope→destructive→budget ∧ WARDEN ∧ m-of-n),
   first-failure-wins, error=DENY, egress floor never liftable (`agents/egress_guard.py`).
3. **Meter:** every model call flows through `ModelGateway`/`GrantMeter`
   (`apps/sigil/sigil/model_gateway.py:319,800,820`) — write-ahead metered, rate-card billed.
4. **Proof/PCR:** a clearance is a signed, offline-re-verifiable negative cert
   (`integration/vigil_integration/remediation/remediation_cert.py`), spawnable only from a FACT
   (`remediation/triage.py:347`).
5. **Flag-off byte-identical + FATAL-2:** each capability lands behind a default-OFF `VIGIL_*` flag read at
   call time (template `apps/sigil/sigil/config.py:289 redpen_enabled`); OFF ⇒ routes/actions absent (404)
   and every shared gate/evaluator byte-identical; offense imports lazy so the sovereign venv never
   co-loads `framework`/`strix` (proven by `integration/tests/test_two_env_boundary.py`).

Master flag `VIGIL_APEX_ENABLED` (default OFF) + per-PR sub-flags, each appended to `config.py` (append-only
⇒ low cross-PR conflict).

---

## PR-1 — APEX Autonomous Driver (promote the OODA loop to the primary driver)

**Goal:** make an authorized live web engagement run the select→act→observe→reorient loop *in-engagement*
over the long-lived gated executor (shared budget), choosing each next action from the live world-model,
instead of today's monolithic `WebScanCampaign` + post-return advisory appendix.

**Reuse (do not rebuild):**
- `engine/crucible/framework/v2/engage_autonomous.py` — `run_autonomous_cycle` (`:1223`, loop `:1489`),
  `_select` (`:361`), `_build_goal_tree`/`_endpoint_probe_targets` (`:273`/`:245`),
  `_drive_probe`/`_drive_reverify` (`:853`/`:825`). This OODA machine becomes the driver.
- `engine/crucible/framework/v2/agents/http_executor.py:HttpExecutor.gated_fetch` (`:840`) + `request_budget`
  (`:353`/`:689`) — the ONE long-lived gated executor to share across scan+act.
- `engine/crucible/framework/v2/intel/promote.py:promote_to_endpoints` (`:160`) — the recon→endpoint bridge.
- `agents/tools/invoker.py:invoke_tool/_gate` (`:195`/`:118`) — the ACT adjudication gate.

**Build/change:**
- `engine/crucible/framework/v2/engage.py` — behind `VIGIL_APEX_AUTONOMOUS` (default OFF): keep the executor
  from `_do_scan` (`:1699`) OPEN and drive `run_autonomous_cycle` in-engagement over it (replacing the
  post-return `_run_autonomous` appendix at `:2941` / throwaway executor at `:2231`). OFF ⇒ the exact current
  fixed pipeline, byte-identical.
- Widen ACT beyond the two built-in tools to the full governed probe arsenal via the goal-tree + planner;
  multi-agent decomposition = specialized surface selectors (recon/auth/injection/access-control/business-
  logic/SSRF-OOB/exploit-chain) over the existing planner.
- World-model memory = the signed spine (already the engine's store); session resume via the existing
  checkpoint path.

**Acceptance (end-to-end):** ON, against the MERIDIAN range, a seed with only a recon service →
promote→crawl→select→probe→oracle fires → **oracle-confirmed FACT**, nothing human-pointed, every request
through `gated_fetch` under one shared budget. OFF ⇒ byte-identical to today.

**Files:** `engage.py`, `engage_autonomous.py`, `agents/tools/invoker.py`+`builtin.py`, `agents/http_executor.py`,
`config.py` (flag), the `make gate` golden.

---

## PR-2 — APEX OOB-First Confirmation (harden + default-wire OOB; close the SSTI gap)

**Goal:** the loop confirms blind/async vulns as signed FACTs by default; close the one missing class
(blind SSTI); make collector provisioning production-turn-key. OOB is already built — this HARDENS and WIRES.

**Reuse (do not rebuild):**
- `engine/crucible/framework/v2/verify/oob.py` (token mint `:293`, receipt sign/verify `:94/:99`,
  `OOBReceiver`), `verify/oracles.py:oob_callback_oracle` (`:4516`), `verify/collaborator.py` + `dns_collector.py`,
  `console/oob_collaborator.py` (managed relay lifecycle, independent keypair `:367`).
- `integration/vigil_integration/live/runtime_redrive.py:oob_callback_redrive` (`:2242`),
  `_oob_probe_requests` (`:2203`), `OOB_REDRIVE_CLASSES` (`:364`); `live/wiring.py:_governed_oob_material`
  (`:1854`), `_with_trusted_oob_material` (`:1955`), the triple-lock; `authority/gate.py:authorize_oob_egress` (`:271`).
- `integration/tests/test_oob_autonomous_feeder.py` — the governed-autonomous-mint wiring to extend.

**Build/change (behind `VIGIL_APEX_OOB`, default OFF where new):**
- **Blind SSTI OOB** (the one real new class): add `scanner/library_entries/m3_blind_ssti_*.json` (callback-
  bearing templates per engine), add `ssti`→OOB routing in `runtime_redrive.py:OOB_REDRIVE_CLASSES`, a
  `oob_callback.ssti`-covered path, and an `evidence-branches.json` entry — all following the existing
  `oob_callback` pattern (no new OracleKind; reuse `OOB_CALLBACK`). Positive + same-shape negative control.
- **Default-wire into the loop:** PR-1's driver invokes OOB confirmation on every injection/SSRF/XXE/SSTI
  candidate via the existing `oob_callback_redrive`/governed-feeder seam (extend `test_oob_autonomous_feeder.py`).
- **Operationalize provisioning:** turn-key collector keypair mint + pubkey pin into the signed authority
  (reuse `ui/target_authorization.py:339`, `wiring.py:provision_authority`), managed relay lifecycle; DNS path
  operationalized for HTTP-egress-blocked targets.

**Acceptance (end-to-end):** ON + provisioned, a blind-SSRF and a blind-SSTI MERIDIAN target → loop plants a
unique canary → collector signs → `oob_callback_oracle` fires → **signed OOB FACT that re-verifies offline**;
a non-vulnerable same-shape twin → no callback → stays LEAD. OFF/un-provisioned ⇒ fail-closed LEAD,
byte-identical.

**Files:** `verify/oob.py`+`oracles.py`, `runtime_redrive.py`, `live/wiring.py`, `scanner/library_entries/*`,
`docs/capability-matrix/evidence-branches.json`, `console/oob_collaborator.py`, `config.py` (flag),
`test_oob_autonomous_feeder.py`.

---

## PR-3 — APEX Impact & Closure (chaining-to-impact + proof-carrying remediation + continuous retest)

**Goal:** turn confirmed FACTs into demonstrated impact, autonomous proof-carrying fixes, and continuous
prove-clean retesting. Operates OVER confirmed findings → decoupled from how PR-1/PR-2 produce them.

**Reuse (do not rebuild):**
- Chaining: `engage.py:_do_chaining` (`:1938`) + the attack-path/Bayesian scoring (`engage.py:793`).
- PCR: `remediation/remediation_cert.py` (`mint/verify_remediation_certificate`), `autopatch/loop.py:autopatch`
  (`:443`, FACT-gate `may_remediate` `:479`), `remediation/triage.py:may_remediate` (`:347`),
  `codescan.verify_finding_cleared`, the m-of-n `--open-pr` leg.
- Continuous retest: the merged live retest re-proof seam (#1175) + `jobs/engage_worker.py` + `jobs/launch_trigger.py`.

**Build/change (behind `VIGIL_APEX_IMPACT`, default OFF):**
- Extend chaining from "pure reasoning over the report" to pursue/demonstrate impact where authorized
  (data-access/privesc/ATO), each step oracle-confirmed or OOB-attested, all through the gate.
- Wire PCR as an autonomous close-the-loop step: confirmed FACT → propose fix → gated PR → signed negative
  cert (positive-control still fires, patched context silent).
- Continuous autonomous re-testing: a scheduler that re-fires each confirmed finding's `oracle_context` on a
  cadence (reuse the retest re-proof), proving fixes stay fixed and flagging regressions.

**Acceptance (end-to-end):** a confirmed MERIDIAN finding → chained impact path → fix PR → **signed negative
remediation cert that re-verifies offline** → scheduled retest proves-clean; a regression re-opens it. OFF ⇒
byte-identical.

**Files:** `engage.py` (chaining tail only), `remediation/*`, `autopatch/loop.py`, a continuous-retest
scheduler (reuse `retest_reproof` + `launch_trigger`), `config.py` (flag).

---

## Parallelism, coordination & merge order

- **File ownership is near-disjoint:** PR-1 = `engage.py` scan/loop section + `engage_autonomous.py` +
  executor/invoker; PR-2 = `verify/oob*` + `runtime_redrive.py` + `wiring.py` + library entries; PR-3 =
  `engage.py` chaining tail + `remediation/*` + `autopatch/*` + retest scheduler.
- **Coordination points (keep-both at merge):** (a) `engage.py` is touched by PR-1 (scan/loop, `_do_scan`
  region) and PR-3 (chaining tail, `_do_chaining` region) — different functions, section-scoped, merge keep-both;
  (b) `config.py` flags are append-only per PR; (c) `evidence-branches.json` only PR-2 edits (SSTI branch);
  (d) `test_two_env_boundary.py::_PROBE` + the CI offense run-list — one-line per PR, coordinate at merge.
- **Build in parallel** (3 worktrees off fresh origin/main), **merge sequentially** after each is red-penned.
  PR-1 is the integration spine; PR-2/PR-3 build against thin seams and rebase onto PR-1 at merge. Merge order
  PR-1 → PR-2 → PR-3 (or any order; the seams are additive + flag-gated).

## Multi-agent build workflow (how it gets built)

Per the standing cadence: for each PR, a builder team in its own worktree builds the slice end-to-end
(commit `-F`, no push); then an **adversarial red-pen** pass (construct a false FACT, prove the firewall
re-execution/OOB pin/byte-identical hold) before the orchestrator pushes + opens the PR. Highest-soundness
care: the OOB/oracle surfaces get a dedicated red-pen. Merge only on all required checks green + red-pen clean.

## Verification (per PR — "extreme", end-to-end)

Every PR must pass, before merge:
- **Unit + integration** for every new/changed symbol; **e2e against the MERIDIAN range** (`target up`
  governed range) demonstrating the acceptance capability above; the brain/scanner suites as regression.
- **Mutation-verify each guard** (break it → its named test reds; restore) incl. a *same-shape negative
  control* for every oracle/OOB path.
- **Adversarial red-pen** that tries to mint a false FACT (OOB forged-key refusal, producer-say-so, byte-
  identical-off).
- `integration/tests/test_two_env_boundary.py` (FATAL-2), `scanner/tests/test_claim_discipline.py`,
  `integration/tests/test_tool_manifest.py`, `make gate` byte-identical with the flag OFF, ruff, mypy ratchet,
  `tools/governance/money_hygiene.py`.

## Honest boundaries (named, not faked)

- **Not greenfield:** OOB + the OODA loop + recon→test bridge + PCR are already merged; this promotes/hardens/
  wires them. The genuinely *new* oracle work is narrow: blind-SSTI OOB coverage.
- **Owner-gated at go-live:** live *external* targets (vs MERIDIAN/loopback) need the signed charter + posture
  gate + `VIGIL_CUSTOMER_PROD_EXECUTION`; OOB needs collector provisioning + the pubkey pinned into the owner-
  signed authority. These stay operator inputs, not fabricated.
- **OOB is positive-only** (`clean_capable:false`) by design — it CONFIRMS a blind vuln, never CLEARS one;
  clearing is PR-3's PCR path. Kept honest.
- **"Fully built" ≠ "mints every class":** branches that cannot be soundly confirmed stay honest LEADs with a
  written rationale (the claim-discipline test forbids silent abandonment).

## Pre-work (before this program; already done, awaiting commit)

Two finished HexStrike-FACT oracle slices sit verified-but-uncommitted in their worktrees (plan mode blocked
the commit): `claude/oracle-strix-redrive` (3 promoted / 3 honest leads) and `claude/oracle-injection-replicate`
(3 `datastore_error` promoted via a parser-state-toggle oracle / 3 `sprt_inference` honest leads). On exiting
plan mode: run their remaining read-only suites + ruff/mypy/money-hygiene, `git commit -F`, push, and open both
PRs (they are independent of APEX and of each other).
