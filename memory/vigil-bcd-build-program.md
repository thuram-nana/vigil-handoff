---
name: vigil-bcd-build-program
description: "Owner 2026-11-16 — build ALL of §15.11 B (money/metering) + C (customer-app screens) + D (Red-Pen R1–R5 with the Code screen) with multiple parallel agents, strictly tested; wave plan + worktrees + owner answers"
metadata:
  type: project
---

Owner (2026-11-16): "deploy multiple agents ... build the entire B and C end to end leaving nothing out, and test it
strictly ... 100 percent built and tested". Then answered: W5 disputes remove credits when the dispute OPENS (re-grant
when won); build D (Red-Pen R1–R5) together with C's Code screen (fake GitHub in tests; owner registers the real App).

Orchestration (no Workflow tool in that session → Agent tool, background builders, one vigil worktree each at
/home/kali/vigil-wt-<slice>, agents commit locally, orchestrator red-pens → pushes → PR → merges sequentially on the
15 required checks). Shared rules for builders: <scratchpad>/e0/agents/RULES.md (rewrite it if the scratchpad is gone:
hard rules = HANDOFF §0 + the takeover mandate). Wave 1: W2c-2, M1c-2b, Team, Account security, Monitoring+AEGIS
offers, Assessments launch. Wave 1b: W5, G1, Red-Pen planner. Later: W3 (after M1c-2b), W4, A1, W6/C5, L1, O1,
Retest (C1b), Overview (C7), R1–R5 + Code screen, wallet E2E last.

**Why:** owner wants everything; quality bar = default-OFF, mutation-verified, red-penned, CI-green.
**How to apply:** UI slices conflict in app.js/guide.js/custui.py/index.html — merge them one at a time and rebase the
rest; every new route must be covered by the onboarding guide census. Related: [[vigil-onboarding-guide]],
[[vigil-payment-provider-stripe]], [[vigil-wave-cadence]].

## Red-Pen (D) — planner findings + RESOLVED decisions (2026-11-16)
Planner result: the SecurityJob spine + webhook ingest/drain + reconciliation + gated api.github.com READ client +
gated comment/fix-PR writes + owner-signed repo-exact GitWriteGrant + diff-scoped `run_codescan` + PR-bot worker are
ALREADY BUILT (the merged "repo-connect" program). Red-Pen reuses them and adds: GitHub **App** auth (JWT→install
token + check run), the adversarial **reproduction engine**, per-repo settings/schedule/review-metering, Fix+Retest,
and the **Code screen** (C6; the merged onboarding guide already documents `#/code`, so C6 is what lets
`VIGIL_CUSTOMER_ONBOARDING` go on). Master flag `VIGIL_REDPEN_ENABLED` (default OFF). Full plan +
file/test lists: `<scratchpad>/e0/agents/REDPEN-PLAN.md` (re-derive from the planner's report in the transcript if the
scratchpad is gone). Slices: R1 (reproduction engine, offense) + R2 (App+registry+settings+metering, sovereign) are
the parallel long poles; R3 (PR-review worker + check run; sole-owns uiproxy.py) + R4 (System demand/schedule) + R5
(Fix+Retest) + C6 (Code screen; sole-owns custui/server/provision/app.js) queue behind them.

Orchestrator decisions (build to these; owner may revisit at go-live): **O-1** v1 posts only reproduced FACTs from the
OFFLINE/static oracle set (`static_oracle_confirmed` + request-side `_BREAKOUT` + sandbox crash/error/side-effect
signals); every other lens is a LEAD (never posted); never gate on `verified_by_oracle`. **O-2** per-repo limit =
calendar month, refuse-new-let-in-flight-finish. **O-3** the owner-signed connected-repo record is the authorization
of record (white-box owner-authorized repos = the APPROVED safety line, NOT the declined open-self-serve portal);
`reserve_redpen_review` reuses `_customer_slug(org,"github.com/org/repo")` + the W2 lease/settle path. **O-4** new
per-org `manage_code` perm in founder+operator, NOT org_admin (billing≠testing). **O-5** App private key sovereign-only
(Stripe discipline); mint short-lived per-repo install tokens; write the check run SOVEREIGN-side from an inert
outcome envelope (only FACT severities cross); only the comment/fix scoped install token reaches offense via a
nonce-bound 0600 seam. **O-6** a billed review needs `VIGIL_METERED_BILLING` (⇒ M1c-2b merged + owner flip); build/test
with fakes, live charge waits. Owner at go-live registers the real GitHub App (id+key+`checks:write`) — already in
HANDOFF §6.

## Wave 2 (2026-11-17 ~06:30) — ALL of B/C/D/E launched in parallel (owner: "all in parallel, nothing left out")
18 background agents, one worktree each at /home/kali/vigil-wt-<slice>, off origin/main f8e2979f (8 wave-1 slices
merged; W5 #1151 + Account-security #1147 finishing CI). Branches claude/<slice>. Each follows e0/agents/RULES.md
(trailer now **Claude Opus 4.8**); Red-Pen ones also e0/agents/REDPEN-PLAN.md. Agents commit locally, do NOT push;
orchestrator red-pens→pushes→PR→merges SEQUENTIALLY (UI slices conflict in app.js/custui/guide.js; money slices in
launch_gate/billing/config — merge one, rebase rest; same keep-both pattern used in wave 1).
- **B:** w3 (credit pause/resume; consumes M1c-2b paused_credit + W5 hold), w4 (worker+premium metering), w6
  (billing dashboard UI), a1 (auto top-up, Stripe off-session), o1 (owner money view — owner surface, margin NEVER in
  repo), l1 (Max depth: 1 per-engagement rate limit + governed tools only), stripe-default (flip unset default
  Flutterwave→Stripe).
- **C:** retest (C1b), overview (C7), aegis-sale (wire monitoring line into drain+checkout), code (C6 screen — the one
  that lets VIGIL_CUSTOMER_ONBOARDING turn on; builds vs R2 merged + R3/R4/R5 interfaces).
- **D:** r3 (PR-review worker+webhook+check run; sole-owns uiproxy), r4 (System demand+schedule), r5 (Fix+Retest). All
  behind VIGIL_REDPEN_ENABLED; build vs R1/R2 merged + each other's documented interfaces.
- **E:** front-c (in-repo scale foundation + simulated 100k/1M test; deploy-infra S2-S4 = owner), wallet-e2e (§11.12
  acceptance test — stages skip-pending-merge, un-skip as W3/W4/W5/A1 land), gate5 (E2E 0-dead-button validation +
  checklist; screens skip-pending-merge).
- Plus **enroll-harden** (the real finding: MFA enrol needs step-up + confirm-before-activate; mfa/verify rate limit).
**Dependency/merge order:** R3→R4/R5→Code; W3 after M1c-2b(merged)+W5; W6 shows A1/W5 fields; wallet-e2e+gate5 LAST
(un-skip as features land). Rate limit (5-hr) WILL stall some — resume via SendMessage after reset (wave-1 proof).
Interfaces the dependent agents assumed are in each agent's final report — reconcile at merge.
**2026-11-17 15:20 status:** wave 1 COMPLETE — Account-security #1147 (46fc767c) + W5 #1151 (6c574ecb) merged. W5's
P7 red was a latent gateway race (provider error relayed BEFORE the reservation freed → racy `pending==0` assert);
fixed in its own PR #1152 (`_free_before_reply` frees ticket + G1 lane before every pre-finally reply; 4/4 mutants).
Wave-2 agents hit the 5-hr cap twice (resets ~09:30, ~15:10 ET); 18 agents burn a window in ~25 min. Resume all via
SendMessage after each reset (trailer now Opus 5.5 — RULES.md updated). Stripe-default is the only one committed so far.

**2026-11-17 16:00 — merges + before-go-live gates:**
- MERGED to date (wave 1): #1147 Account-security, #1151 W5. Wave-2 landing: #1152 gateway free-before-relay
  (fixed its own mypy ratchet regression: `_g1_lane` declared on the class so method order can't fix its type),
  #1153 AEGIS plan sale (default OFF `VIGIL_CUSTOMER_MONITORING_ENABLED`; 918 tests green on rebased main; I
  spot-verified 3 money-critical mutants RED: drain-mints-own-line, flag-off byte-identical, guard-split-by-kind).
- **GATE (AEGIS, before flag flip):** monitoring renewal continuity — the drain mints the monitoring-subscription
  line from a verified charge, but `Provisioner.monitor_plan_changed` is never called, so the host-slot/respond
  config is not re-pushed on renewal and grants lapse at period end. Credit-plan + monitoring share ONE renewal
  mechanism (fresh owner-signed `subscribe`). Wire monitor_plan_changed into the drain (it raises when the flag is
  off in the drain env, fires after mint/invoice) + treat renewal invoices as mint events, before the monitoring
  flag goes live. W5 reversals act on top-ups only → AEGIS/credit-plan dispute handling also a gate.
- **FIX sent to Stripe-default agent:** checkout treats unset provider as Stripe but `billing-drain`/`ops-sweep`
  `--provider` still default to flutterwave and the Flutterwave systemd unit runs the bare drain → two sources of
  truth. One shared resolver for checkout+drain+sweep; pin the unit's --provider; test all three agree.

**2026-11-18 00:10 — Retest (C1b) reported + a THIRD before-go-live gate:**
Retest built default-OFF (`VIGIL_CUSTOMER_RETEST_UI`), 57/57 mutants, IDOR-tested, byte-identical off, commit
a5da9190 (not pushed; base f8e2979f — conflicts with Overview+Account-security in app.js/custui/provision, rebase
after those merge). Identity = slug+finding_ref+oracle_context_digest. **GATE: `provision._retest_reverify` is a
fail-closed `"unavailable"` stub — the LIVE re-proof is NOT wired (FATAL-2: sovereign plane can't import the offense
engine). Completing slice = a cross-plane seam running `remediation.prove_remediation` under a LaunchGate lease
(gateway-metered), mapping only decisive REMEDIATED→cleared. Keep VIGIL_CUSTOMER_RETEST_UI (and onboarding) OFF
until it lands.** Pattern now recurring: UI+authz+seam scaffold ships default-OFF; the live engine (offense re-proof
/ AEGIS renewal push / live payment) is the completing work before each flag flips. Build these completing slices
(cross-plane re-proof; AEGIS monitor_plan_changed renewal) after the current wave merges.

## Wave 2 — ALL 21 slices built & PR'd (2026-11-18 ~02:00 ET). MERGE TRAIN in progress.
Every B/C/D/E slice is merged or in CI; only R4 (System reviews) still building. Orchestrator (me) resolved every
rebase conflict against the moving base. Merge order: independent first, then overlap clusters rebased in turn.
- **MERGED (9):** #1147 Account-sec · #1151 W5 · #1152 gateway-free-before-relay · #1153 AEGIS-sale · #1154 Overview ·
  #1155 enroll-harden · #1159 wallet-e2e · #1156 L1 Max-depth · #1158 Gate5. main ~ef827f1b.
- **IN CI (11):** #1157 W4(worker-meter) · #1160 O1(owner-money) · #1161 R3(pr-worker) · #1162 R5(fix/retest) ·
  #1163 W6(billing-dash) · #1164 A1(auto-topup) · #1165 FrontC(scale) · #1166 Stripe-default · #1167 W3(pause/resume) ·
  #1168 Retest · #1169 Code(C6→enables onboarding flag). Watcher b1oqinjz0 on 1160/1162/1165/1161.
- **Conflict clusters remaining to merge-order-carefully:** model_gateway.py{W4,O1,W3} · billing.py{W4,A1,W3,Stripe} ·
  app.js/custui{W6,Retest,Code} · launch_gate{W3}.
- **Fixes I folded during integration:** W5↔A1 billing.py 6-way merge (event-dispatch guard = A1 widened set + W5
  reversal branch; checkout-mint A1-capture-inside-if + W5-repay-outer); latch allowlist += auto_topup.py(writers),
  provision.py(dynamic, RETEST_KIND); W4 ci.yml offense-leg += test_engage_worker_heartbeat.py; R3 docs CLI-REFERENCE
  + wiring-status.json (redpen-review/redpen-webhook-drain verbs); Code limitation-inventory non-debt site
  (installation_verifier = owner go-live, fail-closed); enroll provision.py keep-both + debt-phrase reword.
- **3 before-go-live GATES (flags stay OFF until):** Retest live re-proof (cross-plane seam, metered) · AEGIS
  monitor_plan_changed renewal · (R3/R5 live reviewer + metered reserve + installation_verifier = owner go-live).
- **TODO after all merge:** verify+PR R4; final HANDOFF §15; delete merged claude/* branches; retest/aegis completing slices.

## Completing-engine wave + main-red hotfix (2026-11-18 ~morning)
**MAIN WENT RED** after W3 merged: wallet-E2E's W3 stages (5b/6b) keyed `_merged_w3()` on a `reason` param of
`record_lease_pause` — but W3 added SEPARATE accessors (`record_credit_pause`/`credit_pause_reason`/
`credit_paused_launches`) and left `record_lease_pause` as the plain halt marker. So the stages took the pending
branch and asserted the opposite of reality, reddening the whole-suite jobs (P7 + fold-cache-ON) on main AND on
W6/A1/Code/R4. **Hotfix PR #1171** (test-only): `_merged_w3()` → `hasattr(clp,"credit_pause_reason")`; stage 5b reads
`credit_pause_reason` not the spec's `pause_path`. wallet-E2E = 18 pass / 1 W5 xfail / 1 A1-skip. Watcher bbsjhaxy2.
**LESSON:** a skip-pending-merge stage that auto-un-skips on a `_merged_X()` probe MUST be reconciled to the real
slice's symbols the moment X merges, or it reddens main. Check every `_merged_*` probe when its slice lands.
**Stripe sandbox payment e2e PROVEN GREEN** (~264 tests): checkout→webhook verify+re-fetch→mint→meter→exact settle→
statement reconcile→disputes/hold/repay→egress, sovereign + integration legs. Fake-Stripe (no real money). Real
api.stripe.com test-mode run still needs owner sk_test_ keys + public webhook endpoint.
**COMPLETING ENGINES building (owner: WIRE/IMPLEMENT ALL fully):**
- Retest live re-proof (agent aedb670390483947b, wt retest-reproof): replace `_retest_reverify` stub w/ real
  sovereign→offense seam + worker re-firing the finding's oracle_context under a metered lease; fixed ONLY on
  decisive cleared (reuse R5 refire_reproduction soundness). Behind VIGIL_CUSTOMER_RETEST_UI.
- AEGIS renewal (agent a73e3aae3d4d7ffba, wt aegis-renewal): wire monitor_plan_changed into the drain so a verified
  renewal re-pushes slots+respond (resolve the 3 blockers AEGIS-sale named: raise-when-off, re-drive-wedge, needs
  provisioner+publish-dir). Behind VIGIL_CUSTOMER_MONITORING_ENABLED.
- Red-Pen live wiring (live gateway reviewer + metered reserve/settle + Code's installation_verifier): SPAWN once
  R4 #1170 + Code #1169 merge (needs their code on base to wire+test). NOT skipped — queued-on-merge.
**FLAG-FLIP STANCE (told owner):** do NOT flip committed default-OFF (it's the money-safety/fail-closed constitution
+ engines not all wired + real-money/AI risk). Instead: build engines → local sandbox bring-up w/ flags ON + Stripe
test mode to prove live; committed defaults stay OFF. Front C S2-S4 real-cluster run needs owner to RE-OPEN server
work (2026-11-16 no-server rule); S2/S3 CODE is buildable offline now.

## Completing-engine wave COMPLETE (2026-11-18) — all 6 PR'd + go-live artifact
Owner "WIRE/IMPLEMENT everything left" + "outline + implement EVERYTHING". Items 1-10 of the left-to-code list:
all 6 buildable ones DONE + PR'd, 3 queued-on-merge, 1 (stage-9) self/defer.
- #1175 Retest live re-proof (cross-plane seam, metered lease, 13/13 mutants; adds 2nd flag VIGIL_CUSTOMER_RETEST_REPROOF).
- #1172 AEGIS renewal (monitor_plan_changed hook in drain; 7/7).
- #1173 doctor preflight (checkout⇒provider provisioned; 5/5).
- #1174 O1 arrears column (W5 owed beside at-risk; 4/4).
- #1176 monitoring fold-index (last O(N) customer read → O(returned); 6/6).
- #1177 Front C S2/S3 (ShardAssignment + routing resolver + crash-safe migration, VIGIL_MULTI_SHARD default-OFF; 24/24;
  cross-host copy/dispatch + S4 cluster = owner deploy, LIMIT-frontc-shard-cross-host-deploy).
QUEUED (spawn on merge): Red-Pen live wiring (R4+Code), GitHub connect nonce (Code), wallet-E2E stage-9 (A1 merged;
  stage-9 is an unconditional skip → doesn't break main; wire to assert A1 auto-topup).
Main cascade: W6+A1 merged (+hotfix #1171 for the wallet-E2E _merged_w3 main-red). R4/Code re-pushed clean, re-running.
Recurring cross-slice fix pattern at merge: new page-flag → add to Gate5 _WALK_FLAGS + security-UI node stubs;
new governor writer → latch _GOVERNOR_EVENT_WRITERS; new vigil verb → wiring-status.json + CLI-REFERENCE; new
framework-importorskip integration test → ci.yml offense leg; _merged_X() wallet-E2E probe → reconcile to real symbols.
## GO-LIVE ARTIFACT delivered: https://claude.ai/artifact/R9R2gXJ162frBYw7yNbLio (owner runbook — accounts/keys/3 origins/
Stripe/flags/Gate5; grounded in docs/runbooks/GO-LIVE-quickstart.md). Private to owner.
