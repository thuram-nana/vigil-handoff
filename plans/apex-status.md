# APEX — Status Report (PRIVATE)

> Private working note for the owner. Do not commit. Plain English.
> Written 2026-11-24 from four independent area audits. Every HIGH and
> MEDIUM gap below was re-checked by reading the actual code on `main`
> (read-only worktree `/home/kali/vigil-wt-apex-audit`, HEAD `dceea03a`).
> Nothing was changed. "OBSERVED" = I read it. "INFERRED" = I reasoned it
> from what I read without running it.

---

## 1. What APEX is

APEX is the program that turns VIGIL from a scanner into an **autonomous
pentester**: it drives its own attack loop (pick a surface, act, watch,
re-think), confirms every finding with an oracle (not a guess), proves
blind bugs out-of-band, chains bugs into real impact, opens a fix PR with
a signed "it's fixed" certificate, and keeps re-testing on a schedule.

**Relation to the customer app:** the customer-facing surfaces (Fix-first
view, Attack-path view, a "Confirmed out-of-band" badge, a Retest button)
are the consumer presentation layer built *on top of* APEX. They only show
real content when the APEX engine capabilities underneath them are turned
on. Today both layers ship off.

---

## 2. What was delivered (all MERGED into `thuram-nana/vigil-sovereign`)

| PR | Name | Merged | What landed |
|----|------|--------|-------------|
| #1181 | oracle-strix-redrive (pre-work) | 2026-11-18 | Promotes Strix DOM/transport branches to FACT. |
| #1182 | oracle-injection-replicate (pre-work) | 2026-11-18 | Promotes NoSQL/LDAP/XPath datastore-error to FACT via a parser-state-toggle oracle. |
| #1185 | APEX PR-1 Autonomous Driver | 2026-11-18 | The in-engagement OODA loop becomes the primary driver when the flag is on; all 7 surface selectors; one shared request budget; default-OFF, byte-identical-OFF. |
| #1184 | APEX PR-2 OOB-First Confirmation (+ blind SSTI) | 2026-11-18 | New blind-SSTI OOB class (8 engine templates), fail-closed when off, signed OOB FACT end-to-end. |
| #1183 | APEX PR-3 Impact & Closure | 2026-11-19 | Chaining-to-impact (wired), autonomous PCR fix-PR + negative cert (library), continuous retest scheduler (library). |

All three source branches are fully merged with no orphan commits. The
soundness and honesty discipline is strong — in particular, PR-2 proves
*out-of-band execution of the callback URL*, not template evaluation, and
leaves the SSTI attribution as the operator's lead. That is correct.

**The important nuance:** PR-1 and PR-2 are *operating* capabilities (they
run inside an engagement). Two of PR-3's three pieces are *verified
libraries with no production caller* — proven by one hermetic test each,
but nothing in production assembles and runs them yet.

---

## 3. What is switched ON in the shipped/hosted config today

**Honest answer: nothing. APEX is entirely off and entirely undocumented
for deployment.**

- OBSERVED: `grep VIGIL_APEX` across every `*.env.example`, every systemd
  unit, and every runbook returns **zero hits**.
- Every APEX flag defaults OFF and reads the environment at call time.
- With the flags off, the engine runs the old fixed pipeline and every
  APEX customer surface renders nothing — byte-identical to a build
  without APEX.

| Flag | Default | In any deploy env? |
|------|---------|--------------------|
| `VIGIL_APEX_AUTONOMOUS` | OFF | No |
| `VIGIL_APEX_OOB` | OFF | No |
| `VIGIL_APEX_IMPACT` | OFF | No |
| `VIGIL_APEX_PRESENTATION` | OFF | No |
| `VIGIL_CUSTOMER_POSTURE / ATTACK_PATH / BLIND_VULN / RETEST_UI / RETEST_REPROOF` | OFF | No |
| `VIGIL_APEX_ENABLED` (planned master) | — | Does not exist |

So turning APEX on is not one switch. It is a scoped rollout with
provisioning and owner inputs (Section 5).

---

## 4. Gaps and loose ends (deduped, ranked, verified)

Severity key: **HIGH** = blocks a real go-live claim. **MEDIUM** = a real
hole, safe only because APEX is off. **LOW** = polish / docs.

Every HIGH and MEDIUM row below was confirmed by reading the cited code.
**None were dropped** — all checked out.

### HIGH

| # | Gap | Evidence (file:line) | Fix | Owner input |
|---|-----|----------------------|-----|-------------|
| H1 | **No supported way to enable APEX.** Not one APEX flag is in any env.example, systemd unit, or runbook. The owner would have to enable it ad-hoc, risking inconsistent config. | OBSERVED: grep `VIGIL_APEX` over `apps/sigil/deploy/*.env.example`, `infra/systemd/*`, `docs/runbooks/*` = 0 hits. | Add an APEX section to `customer.env.example` + `cockpit.env.example`, `OWNER-SWITCHES.md`, and `GO-LIVE-quickstart.md`. | Which APEX layers + consumer surfaces launch. |
| H2 | **PCR close-the-loop has no production driver.** `close_remediation_loop` (fix-PR + signed negative cert) is called only by tests. Flipping `VIGIL_APEX_IMPACT` will NOT run it. | OBSERVED: grep — only its own `def` + `__all__` in `integration/vigil_integration/remediation/apex_pcr.py:61,184`; no caller in cli/jobs/ui. | Build a CLI/worker that injects signers + GitHub App + quorum and calls it. | Target repo/branch, governance quorum, approval to open real fix PRs. |
| H3 | **Continuous retest has no production ticker.** `run_cadence_tick` is never fired on a cadence and nothing registers findings to watch. The existing `vigil-reprove.timer` is a different, older path. | OBSERVED: grep `run_cadence_tick` = 0 non-test callers; lives only in `jobs/apex_retest_scheduler.py`. | Add a systemd timer + worker that loads a registry and ticks it. | Cadence + registry location; approve enabling the timer. |
| H4 | **No public OOB collector host.** The managed relay binds `127.0.0.1` only; a remote arm is guide-only; the DNS path needs an owner-owned delegated domain on `:53`. So blind-SSRF/SSTI confirmation against a real external target cannot happen out of the box, and nothing about it is in deploy env or runbooks. | OBSERVED: `console/oob_collaborator.py:14-26` ("127.0.0.1 ONLY", remote arm GUIDE-ONLY); `verify/dns_collector.py:1-12` (needs an operator-owned nameserver); no deploy/runbook coverage. | Stand up a public TLS proxy + delegated DNS, set the LiveConfig OOB fields + `CRUCIBLE_OOB_RELAY_SECRET`, run the signed provision ceremony, write a runbook. | The collector host/domain/DNS, open ports, and the owner-signed authority ceremony. The plan already names this as an owner input. |

### MEDIUM

| # | Gap | Evidence (file:line) | Fix | Owner input |
|---|-----|----------------------|-----|-------------|
| M1 | **`VIGIL_APEX_IMPACT` has two different truthy sets.** The sovereign reader accepts the extra value `enabled`; the engine + integration readers do not. So `VIGIL_APEX_IMPACT=enabled` reads ON on one plane and OFF on the other. **Latent today** (the sovereign reader has no live caller), but a live split the moment it gains one. | OBSERVED: `config.py:1248` `_OIDC_AFFIRMATIVE` = 5 values incl. `enabled`; `apex_flags.py:15` and `apex_impact.py:37` `_TRUTHY` = 4 values, no `enabled`; `config.apex_impact_enabled` has no non-test caller on main. | Make `config.apex_impact_enabled` delegate to the one integration reader (the consumer-E fix). | Land the consumer-E fix, or patch on main first. |
| M2 | **Resume loses the driver's findings.** On `--resume` the driver is skipped (no open executor), and the report is persisted at scan-complete *before* the driver folds its discoveries, with no re-persist after. A resumed run neither re-runs the driver nor recovers its oracle-confirmed findings. | OBSERVED: `engage.py:1994-1997` (skip on resume), `engage.py:1965` (persist at scan-complete only). Loss is INFERRED from the code, not re-executed. | Re-persist after the driver folds, or re-run on resume, or document non-resumable. | Choose the resume semantics. |
| M3 | **The hosted/customer path never runs the driver.** The launch worker reaches `run_engagement`, but the flag is wired into no deploy env / UI / launch gate, and the separate `live/engine.py` OODA engine never runs the APEX driver. | OBSERVED: `jobs/engage_worker.py:456-460` -> `run_engagement`; `live/engine.py:258-272` is a separate engine; no `VIGIL_APEX` wiring in deploy/custui. | Add owner-gated deploy-env + launch-gate wiring + a test, or document CLI-only. | Confirm operator/CLI-only vs hosted. |
| M4 | **Autonomous discovery arsenal is only 5 checks** (XSS / open-redirect / path-traversal / SSTI x2). SQLi excluded (FP), OOB deferred. So injection/ssrf/auth/access-control/business-logic/exploit-chain surfaces only get exercised on *reverify* of scan findings, not fresh discovery. | OBSERVED: `agents/tools/builtin.py:213-236` (`curated_probe_checks` returns 5; SQLi + OOB deliberately excluded, documented). | Add SPRT-gated SQLi discovery + OOB discovery (after H4). | Confirm the intended division of labor. |
| M5 | **In-engagement impact never exercises the live arm.** `engage` calls `pursue_impact` with no gate and no live_prover, so in production it only re-fires oracles offline; the live data-access/privesc/ATO demo runs only in tests. | OBSERVED: `engage.py:2049-2053` (no gate=/live_prover=); `apex_impact.py:170-172` (live arm needs both injected). | Add an authorized prod caller that injects the gate + a live_prover, or state offline-only. | Prod-execution authority + signed charter before any live pursuit. |
| M6 | **Gate-5 sign-off diverges from the shipped config.** The owner's E2E walk turns the presentation surfaces ON; `customer.env.example` ships them OFF. A green Gate-5 does not reflect what the launch serves. | OBSERVED: `test_gate5_customer_e2e.py:167,177` enables posture/attackPath/blindVuln/`VIGIL_APEX_PRESENTATION`; `customer.env.example` has none. | Ship the surfaces on and re-run Gate-5 against the real env, or scope Gate-5 to the off config. | Confirm whether Fix-first/Attack-path/Retest are part of launch. |

### LOW

| # | Gap | Evidence | Fix |
|---|-----|----------|-----|
| L1 | Master `VIGIL_APEX_ENABLED` flag (planned) never implemented — no single off-switch. | OBSERVED: grep = 0 hits. | Implement it, or record the "three independent sub-flags" decision in the plan. |
| L2 | `V2-LIMITATIONS.md` section 31 is stale — still calls NoSQL/LDAP/XPath datastore-error "LEAD-only", but #1182 promoted them to FACT. | OBSERVED: `V2-LIMITATIONS.md:1407-1442` (LEAD-only) vs `evidence-branches.json:2261/2319/2377` (`fact_capable: true`). | Rewrite section 31; add an APEX residuals section. |
| L3 | `impact_report` is computed but surfaced in no report or UI. | Area audit grep (producer only, no reader). | Render it in the technical/engagement report and/or a UI panel. |
| L4 | PR-1 e2e acceptance runs on loopback, not the named MERIDIAN range (capability still demonstrated end-to-end). | Area audit (test header "Loopback pytest_httpserver only"). | Add a range-backed e2e, or amend the plan. |
| L5 | In-app Guide and the public marketing site point to / claim capabilities that ship off. | Area audit (guide.js steps lack when-gates; public_site copy). | Gate the guide steps + soften copy, or enable the flags. |

---

## 5. Ordered go-live checklist for APEX

Nothing here bypasses the oracle, the offense gates, or the signatures.

1. **Decide scope.** Which engine layers (autonomous / OOB / impact) and
   which consumer surfaces launch. *(owner)*
2. **Provision OOB (H4).** Public relay host + advertise URL; collector
   keypair (private half in the deployment keystore); pin the collector
   pubkey into the owner-signed authority via the provision ceremony; set
   `CRUCIBLE_OOB_RELAY_SECRET` + the LiveConfig OOB fields; delegate a DNS
   base domain if using the DNS collector. *(owner + build)*
3. **Provision GitHub for PCR fix PRs (H2).** Register the GitHub App
   (`VIGIL_GITHUB_APP_SLUG`), store its key, confirm the m-of-n quorum +
   governance signers. *(owner)*
4. **Set an API budget.** Size `CRUCIBLE_CLAUDE_CODE_BUDGET` for OODA
   volume; enable the hard token-budget throttle so a runaway loop stops.
5. **Turn on the live-offense master gate.** `VIGIL_CUSTOMER_PROD_EXECUTION=1`
   + per-host ownership proof + signed authority. Without it no live
   assessment or retest fires, whatever the APEX flags say. *(owner)*
6. **Build the two missing PR-3 drivers (H2, H3)** before claiming
   close-the-loop / continuous retest. *(build)*
7. **Set engine flags** on the offense/integration plane:
   `VIGIL_APEX_OOB=1`, `VIGIL_APEX_AUTONOMOUS=1` (+ tune
   `VIGIL_APEX_MAX_CYCLES`), `VIGIL_APEX_IMPACT=1`.
8. **Set consumer flags consistently in BOTH `customer.env` and
   `cockpit.env`.** Presentation/posture/attack-path/blind-vuln/retest-ui;
   turn `RETEST_REPROOF` on only after steps 2 + 5 (it sends live traffic).
   Run `vigil doctor` to catch edge/cockpit mismatches.
9. **Reconcile customer copy (L5).** Gate the guide steps; review the
   marketing claims against the shipped flags.
10. **Validate + document.** Run Gate-5 against the real go-live env (M6),
    confirm each surface renders and each off-path 404s, then record every
    APEX flag in the env.examples, `OWNER-SWITCHES.md`, and the runbook
    (H1) so the state is reproducible.

---

## 6. Proposed master-plan work items

Full machine-readable copy in `/home/kali/.claude/plans/apex-items.json`.

| ID | Name | Effort | Depends on | Already covered? |
|----|------|--------|-----------|------------------|
| AP01 | APEX enablement path in deploy env + runbook (H1) | S | — | No |
| AP02 | Production driver for PCR close-the-loop (H2) | L | AP01 | No |
| AP03 | Production ticker for continuous retest (H3) | M | AP01 | No |
| AP04 | Provision a public OOB collector host (H4) | M | — | No |
| AP05 | Unify the `VIGIL_APEX_IMPACT` flag reader (M1) | S | — | **Yes — consumer-E fix round (commit e093e163)** |
| AP06 | Driver resume durability (M2) | M | — | No |
| AP07 | Owner-gated customer-path driver wiring (M3) | M | AP01 | No |
| AP08 | Widen the autonomous discovery arsenal (M4) | L | AP04 | No |
| AP09 | Wire the live-impact arm for authorized prod (M5) | M | — | No |
| AP10 | Reconcile Gate-5 with shipped config (M6) | S | AP01 | No |
| AP11 | Master `VIGIL_APEX_ENABLED` kill-switch or record decision (L1) | S | — | No |
| AP12 | Refresh V2-LIMITATIONS + APEX residuals section (L2) | S | — | No |
| AP13 | Render impact_report in reports/UI (L3) | S | — | No |
| AP14 | Range-backed MERIDIAN e2e acceptance (L4) | S | — | No |
| AP15 | Reconcile customer-facing copy with shipped flags (L5) | S | — | No |

**Already covered by running work:** only **AP05** — the flag-reader fix is
already written on an unmerged commit in the **consumer-E fix round**;
landing that round closes it. The **investigation graph** and **AEGIS**
programs do not cover any APEX item.

---

## 7. Verification note

Method: read `main` at the read-only worktree; PR/commit history via
`git` and `gh`. All four source audits ran no test suites (machine loaded);
I ran only read-only `grep`/`sed`/file reads plus one `python3 -I` JSON
read. Every HIGH and MEDIUM gap above I re-confirmed at the cited
`file:line`. The only items left as INFERRED (not re-executed) are the
*runtime consequences* of the resume path (M2) and whether an operator
sets the env var in a real deployment (H1/M3) — both are outside what a
static read can prove.

---

## 8. Added by the completeness check (2026-11-24, second pass)

> What the four area audits + the synthesis never examined. Read-only on the
> same worktree (`/home/kali/vigil-wt-apex-audit`, HEAD `dceea03a`); PR file
> lists via `gh`. Nothing changed. **No HIGH/MEDIUM finding above is overturned**
> — these refine, correct, or close an examination gap the first pass left open.
> Dimensions checked: every PR file-list item, every APEX env flag, the full
> APEX test set, the runbooks, and the customer screens.

### 8.1 PR file-list items the first pass never opened (now read — capability is real, OBSERVED)

The #1181/#1182 "pre-work" rows were taken on trust; their producer files were
never read. They are real:

- **Strix→FACT re-drive rail (#1181):** `integration/vigil_integration/live/dom_redrive.py:1-14`
  (DOM-XSS + client-side prototype-pollution via the `dom_execution` oracle in VIGIL's own
  egress-gated CDP harness), `live/transport_redrive.py:1-16` (a Strix host:port/TLS report
  routed into VIGIL's OWN gated handshake capture → ADMITTED to the FACT-capable
  `service_reachability.tcp_handshake`/TLS branch), `proof/run.py:1-12` (Strix report + executor
  capture → persisted proof record for Proof Studio). RUNNER-owned, re-gated, offline-reverifiable.
- **Injection→FACT parser-state-toggle oracle (#1182):** `engine/crucible/framework/v2/verify/oracles.py:4484-4518`
  (BREAK-vs-REPAIR: an unbalanced metacharacter payload provokes a genuine datastore parser error
  while a balanced control does not — a real query-syntax break, not a static error banner) +
  `verify/adapter.py:555-566,2748-2749` (the `parser_state_toggle` ctx key, appended so the frozen
  gate stays byte-identical). This is exactly the oracle **L2** says `V2-LIMITATIONS §31` still
  mislabels "LEAD-only" — confirms L2 and supplies the rewrite content.
- **7-surface decomposition (#1185):** `engine/crucible/framework/v2/engage_surface_selectors.py:1-182`
  (a stateful round-robin `_select` drop-in sweeping RECON/INJECTION/… so the OODA loop is not
  myopic) over `planner/goal_tree.py` (best-first `score = prior*value/(requests+1)`). Backs the
  status's "all 7 surface selectors" one-liner.

### 8.2 Flags the §3 table omitted (correction)

- **`VIGIL_APEX_MAX_CYCLES`** — read at `engage.py:120`, default `64` (`_APEX_MAX_CYCLES_DEFAULT`,
  engage.py:104), operator-tunable OODA-cycle bound. Mentioned only in checklist step 7; missing
  from the "every APEX flag" table. AP01 must list it in the env so the runaway-loop bound is reproducible.
- **`VIGIL_CUSTOMER_PROOF_BADGE` (APEX PR-I)** and **`VIGIL_CUSTOMER_FIX_IT`** — two APEX
  presentation/closure consumer surfaces absent from the §3 consumer-flag row. PROOF_BADGE renders
  the "Independently verifiable" per-finding badge + proof-bundle download (`__VIGIL_PROOF_BADGE__`
  custui.py:276; route `/api/customer/assessments/proof` gated at server.py:726). FIX_IT (config.py:301)
  is the one-click **gated fix-PR + signed NEGATIVE remediation cert** button — the consumer trigger
  for close-the-loop; its offense side re-checks `VIGIL_REDPEN_ENABLED` + the owner-signed `"pr"`
  grant, mints no fact, implies no clearance. Both default OFF. Add to the AP01/AP10/AP15 surface list.

### 8.3 Correction to §3 "in any deploy env?"

PROOF_BADGE **is** in both env examples (commented, OFF): `customer.env.example:77`,
`cockpit.env.example:136` ("the SAME value as the cockpit, which also needs the delegation path").
So "no APEX presentation flag is in any deploy env" is not exact — one consumer flag is documented
(commented). **H1 still stands:** its literal claim is 0 `VIGIL_APEX_` hits over the env examples,
and PROOF_BADGE is a `VIGIL_CUSTOMER_` flag. Every engine `VIGIL_APEX_*` flag and the other
presentation surfaces (POSTURE/ATTACK_PATH/BLIND_VULN/RETEST_UI/RETEST_REPROOF) remain absent.

### 8.4 Refinement to M1/AP05 — divergence is isolated AND the engine reader is the outlier

Verified all four APEX flags across both planes (the first pass checked only IMPACT):

| Flag | Sovereign `config.py` | Engine/integration | Match |
|------|----------------------|--------------------|-------|
| AUTONOMOUS | `_OIDC_AFFIRMATIVE` 5-val (:1248) | `_APEX_AFFIRMATIVE` 5-val (engage.py:103) | yes |
| OOB | 5-val (:1816) | inline 5-val (runtime_redrive.py:391) | yes |
| PRESENTATION | 5-val (:337) | `_PRESENTATION_AFFIRMATIVE` 5-val (certify.py:191) | yes |
| IMPACT | 5-val (:470) | `_TRUTHY` **4-val** (apex_flags.py:15, apex_impact.py:37) | **no** |

M1 is correctly and completely scoped to IMPACT. The cleaner read: **IMPACT's engine/integration
reader is the family outlier** — three of four engine readers deliberately use the 5-value set so
both planes agree. So the lowest-risk AP05 fix is to bring IMPACT's `_TRUTHY` UP to 5 values (add
`"enabled"`), matching the family, rather than down-scoping `config`. `VIGIL_APEX_IMPACT=enabled`
today reads ON on the sovereign plane, OFF on the offense/integration plane.

### 8.5 Test→capability map (the first pass built none) + correction to §2

CI runs these by directory (`pytest framework/v2`, `pytest integration/tests`, `apps/sigil/tests`)
under two coverage guards (`framework/v2/tests/test_ci_crucible_core_covers_all_suites.py`,
`apps/sigil/tests/test_ci_suite_legs.py`); none sits in a silent `--ignore` (injection_parity +
oob_autonomous_feeder are ignore-then-run in a dedicated leg). So "merged + green" is CI-backed —
never checked by the first pass.

| Capability | Test(s) | Proves |
|-----------|---------|--------|
| Byte-identical OFF (structural) | `integration/tests/test_two_env_boundary.py` | sovereign venv can't import framework/strix; APEX seams lazy-import; negative control fires |
| Flag default-OFF + FATAL-2 | `apps/sigil/tests/test_apex_flag_off_golden.py` | default OFF, affirmative-set parity, accessor loads no offense module |
| PR-1 driver runs when ON | `agents/tests/test_engage_apex_driver.py` | in-engagement OODA over the still-open executor |
| 7-surface decomposition | `tests/test_engage_surface_selectors.py` | round-robin domain sweep |
| PR-2 blind-SSTI OOB | `integration/tests/test_apex_oob_ssti.py`, `test_oob_autonomous_feeder.py` | OOB FACT when armed, fail-closed when OFF |
| #1182 oracle | `verify/tests/test_parser_state_toggle_oracle.py`, `integration/tests/test_injection_parity.py` | BREAK-vs-REPAIR soundness; scan/engage parity |
| #1181 redrive | `integration/tests/test_strix_dom_redrive.py`, `test_strix_transport_redrive.py`, `scanner/tests/test_claim_discipline.py` | Strix branch → FACT discipline |
| PR-3 chaining | `agents/tests/test_apex_impact_chain.py`, `integration/tests/test_apex_impact_e2e.py` | chaining-to-impact |
| PR-3 PCR | `integration/tests/test_apex_pcr.py` (**9** tests) | cert mint+offline-reverify, LEAD-refused, flag-off-inert, patched-build-not-certified, quorum-blocks, request-side-not-provable |
| PR-3 retest | `integration/tests/test_apex_retest_scheduler.py` (**8** tests) | regression-reopen, no-false-clear, inconclusive-never-clears, due-cadence, flag-off-inert, registry-persists |

**Correction to §2:** the PCR and retest libraries are NOT "proven by one hermetic test each" —
they carry **9** and **8** hermetic tests with real negative controls. The "no production caller"
finding (H2/H3) is unchanged and remains the real gap; the libraries are better-proven than §2 states.

### 8.6 Customer-screen pointer correction (M3/L5)

The APEX consumer screens are **server-rendered in `custui.py`** via `__VIGIL_*__` placeholder
substitution (`__VIGIL_POSTURE__`/`__VIGIL_RETEST__`/`__VIGIL_ATTACK_PATH__`/`__VIGIL_BLIND_VULN__`/
`__VIGIL_PROOF_BADGE__`, custui.py:267-278) + route gates in `server.py` (`/api/customer/assessments/posture`
@1818, `/proof` @726). **`customer_app/app.js` carries zero APEX content** — citing it as the customer
artifact (M3/L5) points at the wrong file. The conclusion (surfaces default-OFF, 404 when off) is
unchanged; the surface to review for L5 is custui.py's placeholder/route gating.

### 8.7 OOB machinery is code-complete and test-proven (strengthens H4)

The blind-SSRF/SSTI confirmation *path* is thoroughly tested — `verify/tests/test_oob.py` (15),
`test_oob_signed_window.py` (6), `test_oob_offline_reverify.py` (8), `scanner/tests/test_oob_producer.py`
(4), `authority/tests/test_oob_egress.py` (4), `aegis/tests/test_oob_belief_elevation.py` (14),
`integration/tests/test_oob_redrive_arm.py` (15). H4 is a **deployment-provisioning** gap (public TLS
host + delegated DNS + owner-signed collector pin), not a code gap. The "out-of-band" hits in the
runbooks (`GO-LIVE-quickstart.md:5,65-66`, `AWS-DEPLOY.md:118`, `anchor-trust-root-out-of-band.md`)
are the trust-root GENESIS ceremony / governance-pubkey distribution, NOT the APEX collector — so
H4's "no runbook coverage" is confirmed, not contradicted.
