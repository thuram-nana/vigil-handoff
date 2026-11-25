---
name: vigil-claim5-6-chat-gaps-program
description: VIGIL audit Claims 5 (gov guard) & 6 (RBAC/backup/HA/artifacts) end-to-end + chat rename/delete + the two chat-orchestrator gaps
metadata: 
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-08-17T15:41:19.395Z
---

Follows [[vigil-readiness-fixes-program]] (batch 1-4). Operator: implement audit **Claim 5** (gov guard) with a
UI on/off toggle, and **Claim 6** (RBAC foundation + all ops code + audit/pilot artifacts) end-to-end; then
chat rename/delete; then close the two chat-orchestrator gaps. Repo `/home/kali/vigil` (thuram-nana/vigil-sovereign).
Cadence: 5-agent parallel DESIGN workflow → per-slice build-agent → adversarial red-pen (caught a REAL defect on
nearly every slice) → CI-green → merge. Build agents run targeted tests only (a broad `pytest <dir>` STALLS them);
salvage a stalled agent's uncommitted work + validate yourself. **Operator decisions:** guard default = ON
(safe); guard OFF = works end-to-end (global toggle, NOT a per-host exception — docs must say so); RBAC = real
enforced foundation at key gates, honestly scoped.

**MERGED:**
- **#362 Claim 5** — the `hard_guardrail` (blocks .gov/.gov.cm/.mil/.edu/.int + ~200 IGOs) had ZERO prod call
  sites; relocated to `vigil_core/hard_guardrail.py` (FATAL-2 shareable), wired into the no-gap covering set
  {A1 ethics.require_in_scope, A3 egress_guard, C1 intake front door} + {B1 executor, B2 external_tool, B3
  live_transport, B4 codefix}, each toggle-gated by `protected_guard_enabled()` (env `VIGIL_ALLOW_PROTECTED_DOMAINS`,
  fail-safe ALLOW polarity: unset⇒protected; only owner-signed "1" ⇒ off; OFF never relaxes charter scope).
  Owner-signed + spine-logged; UI toggle + danger warning. Red-pen BLOCK: the **intake** subcommand egressed via
  raw httpx bypassing the guard → hooked; planner/watchdog swallowed the new HardBlockError → catch explicitly.
- **#363 Claim 6 RBAC** — admission gate IN FRONT of the owner-signing funnel (owner key stays sole signer);
  `governor/accounts.py` (owner-signed spine grants, per-username anti-replay LWW in BOTH resolve()+accounts(),
  default-deny); per-user bearer on the existing X-SIGIL-Token; `set_config VIGIL_ALLOW_PROTECTED_DOMAINS` is
  owner-only (`toggle_protected_guard`). Red-pen BLOCK: the `vigil up` command UI **embeds the owner token**, so
  the login gate there is NOT an isolation boundary → honest-scoped (enforcement is real for BEARER-ONLY clients:
  CLI/API; command UI is OWNER-ONLY until per-user proxy auth — the named next slice).
- **#364 chat rename/delete** — `console/chat.py` rename (append-only `meta` title, latest-wins, not-a-turn) +
  delete (transcript + `<id>.attachments/` + registry entry, idempotent, `_safe_chat_id` path-safe); routes on
  the same same-origin+token POST gate; UI pencil/trash icons. Red-pen PASS.
- **#365 Gap: end-to-end model-sovereignty** — the per-session chat model pick was honored only for the chat's OWN
  turn; SPAWNED work (agentic engage, fireteam members, codebase edit) fell back to tier-default cloud
  (`claude-opus-5`). Now threaded end-to-end: `--model`/`--backend` on `vigil engage`, EngineConfig, think_seam,
  member_runner, dev_edit. LOCAL pick → `local_backend_or_refusal` (kernel provider, loopback-enforced, NO cloud
  failover, classified before any anthropic client) — run-local or REFUSE, never silent cloud. Red-pen BLOCK: the
  Ollama constructor PROBES its host in `__init__` (httpx GET) → an off-host probe for a remote CRUCIBLE_OLLAMA_HOST
  BEFORE the loopback check → fixed by pre-validating the CONFIGURED endpoint (env) BEFORE construction, in BOTH
  think_claude AND chat._reason_local. + `--backend` unrecognized ⇒ REFUSE (drift hardening).
- **#366 Gap: durable fireteam-registry** — `ConfirmationRegistry` was in-memory/per-wave/discarded. New
  `EscalationLedger` (append-only `<base>/<slug>.escalations.jsonl`, 0600, O_NOFOLLOW+S_ISREG+st_nlink==1 path-safe,
  redact_tool_args-scrubbed), engagement-scoped in deploy_fireteam, rehydrates _pending/_resolved with append-only
  precedence + deny-by-default (a forged approved never restores a denied allow). Cross-process durable + a
  fail-closed resolve read-back (Tier-B foundation). Red-pen PASS + 2 advisories **for the Tier-B slice**:
  (1) `wiring.py` maps ALL confirmation.* → sink.refusal — inert now (only `register` reachable) but a future
  resolve() would mis-file an APPROVED escalation as a `refusal` in the SIGNED chain → Tier-B must exclude
  `approved`; (2) add a planted-node (symlink/hardlink/FIFO) path-safety test. Disclosed: `redact_tool_args` has a
  pre-existing compositional residual (a `word:`/`word=` prefix before a secret can swallow it) — shared primitive,
  not introduced here.

**OPERATOR DEEP-REVIEW BATCH (2026-08-15/17) — "look deep, confirm before writing any code": 3 reproducible
defects + 6 op-boundaries + a 5-item mandatory list. ALL verified-before-fix (parallel workflow, ALL
CONFIRMED_REAL; found 2 boundaries WORSE than stated — off-box timer was local-disk PrivateNetwork; a "local"
Strix session still egressed cloud). ALL MERGED #372-#377 (+#378 artifacts):**
- **#372 the 3 defects** — Defect 2 (prune-only transparency update wasn't immediately witnessed: the checkpoint
  idempotency key must include base_seq/base_count so a monotonic prune-boundary change re-emits; **class-bug
  across all THREE emitters** — first fix hit `transparency.CheckpointEmitter` which has ZERO prod callers; the
  PRODUCTION emitter is `witnessed_anchor.emit_witnessed`, and a THIRD is `apps/sigil/spine/witness.emit_checkpoint`;
  fixed all 3, red-pen confirmed exactly 3). Defect 3 (`vigil backup` ignored `--crucible-root` → honor args then
  fall back to resolver). (C-S1/C-S4 root-NOT-fork ADRs landed #367-#371: cumulative_merkle_root is a
  schedule-dependent fold, NOT a sound fork signal — head_hash is.)
- **#373 Mandatory #1 signed+atomic fireteam resolver (Tier-B)** — `escalation_approval_bytes` domain-v2, verified
  against a PINNED approver key (never record-supplied), O_EXCL `_TerminalCas` compare-and-set; offense VERIFIES
  ONLY. Picks up #366's 2 advisories (Tier-B excludes `approved` from the refusal-mapping; planted-node test).
- **#374 Mandatory #2 Strix model-sovereignty** — pin STRIX_LLM/LLM_API_BASE per session in `console/actions.py`.
  Red-pen: fixed only `launch_assessment`; the SIBLING `retry_run` re-spawned with the cloud model (real leak) —
  same "sibling site unguarded" pattern → fixed both.
- **#375 Mandatory #3 strict signed high-water floors** — `VIGIL_STRICT_HIGHWATER` strict profile REJECTS an
  unsigned / stripped-to-unsigned floor (per-variant domains: attestation-log vs evidence). Advisory-doc honesty:
  strict enforced at the evidence path; posture/attestation writer-signing is defense-in-depth.
- **#376 Mandatory #4 off-host backup + staged restore + drills** — 2 red-pens. **BLOCK-1**: `--force` restore
  whole-root-swapped a SUBSET-capture dest → DELETED live un-captured data (in a dev checkout crucible_root
  auto-resolves to the in-repo framework tree → wiped the CRUCIBLE code). Fix = UNIT-scoped restore (crucible:
  swap only `.blackboard/store.sqlite` + `.console/runs`, drop the replaced db's stale WAL/SHM sidecars; sigil:
  per top-level unit; base stays whole-swap = whole-capture). Force-gate keys off captured-units-present, not
  whole-dir non-empty. Rewrote 3 green-washed tests + added the missing negative controls. **Re-red-pen**: all 7
  behavioral props HELD; one honesty overclaim ("never deletes un-captured data" — but `.console/runs` is
  wholesale-replaced so a post-backup run drops) → scoped wording. CI guard `test_ci_framework_tests_run_in_offense_leg`
  caught the 4 new framework-importorskip backup tests not wired into the offense CI leg → wired.
- **#377 Mandatory #5 per-user command-UI auth** — proxy delegates to sovereign `/api/whoami` (fail-closed),
  hop-only owner token, coarse offense role-floor. **3 red-pen findings, each fixed+re-verified:** (1) CRITICAL
  viewer→owner — the offense console serves its OWN static index with `__CONSOLE_TOKEN__`, relayed verbatim to a
  viewer who scrapes+replays → fix = redact `self.server.token` out of every relayed NON-SSE body (equal-length
  marker, len-1 carry). (2) MEDIUM redaction was Content-Encoding-blind + doc overclaimed an unconditional
  invariant → force `Accept-Encoding: identity` on the hop + decode-before-scan-or-fail-closed-502. (3) MEDIUM the
  decompression-bomb decoded-cap was checked AFTER a one-shot `gzip.decompress` (OOM) → STREAMING bounded inflate
  (`zlib.decompressobj`, 1 MiB steps, stop mid-inflate). CI: the FATAL-2 `sys.modules` test was pollution-sensitive
  in the shared P5 pytest process (a sibling suite legitimately imports sigil) → run it in a CLEAN SUBPROCESS.
  (F1 formal failure was transient GH infra: setup-java 429/503.)
- **#378 Claim 6 artifacts** — `docs/audit/claim-6-rbac-dossier.md` + `docs/pilot/claim-6-pilot-runbook.md` +
  the MISSING `targets/_template/charter.md`. Honesty red-pen CLEAR (roles/perms byte-exact vs `governor/accounts.py`
  frozensets; 8-row verify checklist all reproduces). The protected-domain guard is described as the GLOBAL
  `VIGIL_ALLOW_PROTECTED_DOMAINS` toggle (fail-safe ON, owner-only OFF, defense-in-depth pre-filter; scope + egress
  floor in force regardless) — explicitly correcting a stale per-host-`.gov.cm`-exception framing.

**LESSONS (this batch):** verify-before-fix caught 2 boundaries worse than stated; a class-bug recurs at EVERY
unguarded site (defect-2 = 3 emitters; Strix launch+retry; overclaim wording at each doc site) — mirror everywhere;
a fix's DOC overclaim is itself a red-pen BLOCK (meet-up, never soften); NEVER trust an auth/destructive fix on
first green — the 2nd/3rd re-red-pen found the gzip-blind, the bomb, and the honesty overclaims; FATAL-2
`sys.modules` tests MUST run in a clean subprocess (shared-process import pollution); a stalled build agent's
uncommitted worktree can be finished by hand + verified; the CI framework-importorskip guard silently protects
against new suites never running.


## Claim 6 DEFERRED RESIDUALS — COMPLETE (S1-S7 + 5 operator defects), MERGED #379-#389

The dossier §9 "deferred" list was then BUILT OUT end-to-end. All merged:

- **S1 per-action offense-console RBAC (#380)** — new stdlib-only `vigil_core/rbac.py` (`ROLES`/`PERMISSIONS`/
  `role_can`/`OFFENSE_ACTION_PERM`) importable by the offense console WITHOUT touching `sigil.*` (FATAL-2 clean).
  The console token is SHARED with the proxy, so token presence cannot prove "came via the proxy" → a DISTINCT
  `VIGIL_CONSOLE_HOP_KEY` HMAC hop-assertion (`principal|role|method|path|ts`, ±30s). No hop header = a direct
  token-holder = owner-equivalent (no regression). **RED-PEN BLOCK: killswitch-trip was owner-tier — a privilege
  INVERSION (a protective emergency-stop must not need the highest tier) → re-tiered to `read`.**
- **S2 hard-prune accounts fold (#381)** — `accounts._fold()` was a genesis scan; a prune below the first grant
  silently DROPPED the account AND reset the per-username anti-replay high-water. Fixed with the promotion.py
  snapshot-seed idiom + a `stranded_active_accounts` referential floor (prune fails closed).
  **RED-PEN BLOCK: green-wash — tests exercised windowing, not physical prune → rewritten with `PrunedView`.**
- **S3 per-user cryptographic identities (#382)** — owner-bound `user_pubkey` + `enroll_pubkey`; POST-only
  `/api/login/challenge` mints a single-use CSPRNG nonce (O_EXCL ledger), `/api/login` PoP branch verifies Ed25519
  over a domain-tagged challenge. **RED-PEN BLOCK: an unconditional `_CORE` change LOCKS OUT every pre-existing
  account → CONDITIONAL-CORE encoding: `_core_fields(payload)` appends an optional field ONLY when truthy, so a
  grant without it stays BYTE-IDENTICAL to pre-slice; malleability inverse closed (strip→False, add→False).**
- **S4 MFA/TOTP + password (#387)** — stdlib RFC-6238 (no new dep), secret SEALED via `Vault.seal_secret`,
  per-(username,step) O_EXCL replay ledger. **RED-PEN BLOCK: password TIMING ORACLE — unknown/no-password users
  skipped scrypt (59x delta) → decoy scrypt hash (1.01x).**
- **S5 OIDC RP (#388)** — ships OFF (`SIGIL_OIDC_ENABLED` unset ⇒ routes NOT registered ⇒ 404, byte-identical).
  Signature-before-parse, RS256/PS256/ES256 + kid selection, NO HMAC path, iss/aud/azp/exp/nbf/iat/nonce, rejects
  non-finite time claims, single-use state↔nonce. **ROLE NEVER COMES FROM A CLAIM** — OIDC authenticates *who*, an
  owner-signed grant authorizes *what* (operator decision: operator-run IdP on the private tunnel).
- **S6 HA close-out** — the profile was already largely built (anti-rollback promotion INTERLOCK, not a lease);
  added `tools/ha/mirror-sync.sh` + honesty reconcile. Deliberately NO lease (would break the witnessed-floor doctrine).
- **S7 honesty pass (#389)** — moved delivered items out of §9 into the enforced sections. **HONESTY RED-PEN BLOCK
  ×3:** (1) dossier row 9 + runbook 3.6 credited a VIEWER's offense-POST 403 to the new console `role_can` gate —
  but a viewer lacks `run_engagement` so the PRE-EXISTING COARSE PROXY FLOOR (uiproxy.py) refuses it one hop
  earlier; the genuine S1 demo is an OPERATOR → owner-tier route. (2) docs said `GET /api/login/challenge` — it is
  POST-only (server.py:695). (3) footer claimed runbook reproduction for "rows 1-13" but row 13 has no runbook phase.

**Operator 5-defect batch (verify-first, all merged):** (1) K8s HA manifests wouldn't start → `vigil up --proxy-only`
+ configurable backends + pod-IP bind via downward API + NetworkPolicy; **red-pen CRITICAL: the remote cockpit LEAKED
the owner token because the redactor blanked only `self.server.token` → value-agnostic `_scrub_html_tokens`.**
(2) fireteam approval returned before durable → `_finish` durable-append-first-and-checked, fail-closed non-terminal
PENDING, self-cleaning `_TerminalCas.claim`. (3) pilot backup CLI syntax. (4) docs behind code (Strix pinning, "no HA").
(5) **new sovereign tests not in CI → whole-dir `pytest apps/sigil/tests/` + `test_ci_sigil_tests_all_run.py` asserting
`set(ci_ignores) == set(KNOWN_EXCLUDED)` — structurally closing the silent-skip class (it surfaced 56 non-running tests).**

**LESSONS (residuals batch):** an optional signed field MUST be conditionally encoded or it locks out existing
principals; a protective emergency-stop tiered too HIGH is a privilege inversion; auth timing is a real oracle —
add a DECOY hash on every early-exit path; a "verification artifact" can mis-attribute WHICH mechanism produced a
403 (the honesty gate exists to catch exactly that); GitHub `apt-get install` on runners can hang for 37+ min with
no job `timeout-minutes` — an infra hang, not a diff failure.

**KNOWN OPEN (found while investigating the Fixes screen, NOT yet fixed):** (a) the rendered console report DROPS
`check_id`, so the in-UI "Apply fix (gated)" button never renders (empirically: 2,634 saved findings, ZERO with a
check_id, ZERO grounding=='fact'); (b) `vigil patch` calls `autopatch_live` WITHOUT a verify_oracle (cli.py:262-263)
so it can open a PR but can NEVER mint a signed "remediated" — it caps at `opened-pr-unverified`. Also LAP-1's
"sandbox build" is only `git apply` (no compile/run), so there is no running patched service for an oracle to
re-fire against; the live re-proof lives in the separate `vigil remediate --prove`.

See [[claim-discipline-meet-up]], [[vigil-authorship-contributors]] (no Co-Authored-By), [[vigil-lww-governance-replay-gap]], [[vigil-chat-orchestrator-program]].
