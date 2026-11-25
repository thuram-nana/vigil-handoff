---
name: vigil-chat-end-to-end-program
description: "Making the VIGIL chat end-to-end: FACT coverage across classes, source review, graphify Q&A, and the run-lifecycle setbacks (PR #769, branch vigil-model-default-fallback)"
metadata:
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-05T07:56:33.931Z
---

Extended the autonomous chat/engage→FACT pipeline (see [[vigil-chat-fact-pipeline]]) into an end-to-end
program on branch `vigil-model-default-fallback` (PR #769). 9 waves; each FACT-minting slice was
independently red-penned (subagent), which caught REAL false-FACTs nearly every time.

**FACT coverage — the SOUND boundary (Waves 1-2, shipped + red-penned):** the chat re-drive now mints a
signed, offline-re-verifiable FACT for **8 classes** — error_based_sqli, open_redirect, cors,
host_header_injection, graphql_introspection (Wave 1, by REUSING the reviewed `live/web_redrive.py` from
the engage oracle seam `wiring._build_oracle` + `engine._redrive_spec`), and path_traversal, reflected
xss, exposure (Wave 2, a new `live/runtime_redrive.py` dispatching to side_effect/reflection_context/
predicate oracles). **PRINCIPLE that defines the boundary:** only auto-confirm a class where (a) VIGIL owns
the decision logic (not an LLM-authored predicate) AND (b) confirmation is a NON-destructive read. Fixed
the SQLi "landmine" (MERIDIAN plants it as `sqli`; the gate wanted `error_based_sqli`) by accepting the
whole family + normalizing via `verifier.normalize_bug_class`.

**Red-pen caught real false-FACTs — do not skip it:**
- Wave-1 BLOCK-1 (`be61f6de`): an LLM `oidc_redirect_uri` claim minted an A07 cert on a plain open redirect
  (evidence byte-identical to open_redirect; the OIDC impact is claim-only). Fix: `web_redrive.
  LLM_CLAIM_WEB_FACT_CLASSES` excludes oidc from the LLM-claim seam (proof.run still accepts it).
- Wave-2 BLOCK-1/2/3 (`3dfcfbc6`): the reused checks prove PRESENCE not CAUSATION → 4 signed false FACTs
  (a docs page containing `root:x:0:0:`; a 200 soft-404 SPA naming DB_PASSWORD; a quote-escaped `<script>`
  string). Fix: path_traversal needs a benign CONTROL request; exposure needs a random-path soft-404
  control + value-shaped signature; xss accepts ONLY the reflection oracle's `html_tag` (breakout) context.
- **Wave-3 (bola/idor) BUILT then REVERTED (`601340f6`):** a two-identity read (even with an anon-control
  leg) still false-FACTs on a SHARED authenticated resource and a CORRECTLY per-tenant-isolated endpoint
  (both tenants return an identical empty envelope), and the same-identity guard (raw cookie-string compare)
  is bypassed by a decorated cookie. A sound BOLA needs an attacker BASELINE (its own object) + server-side
  identity confirmation (a `/me` call). Per "a false FACT is the worst outcome", deferred, not shipped.

**Deferred FACT follow-ons (documented, not built):** sound-BOLA (bola/idor); a gated destructive/write
program (business_logic/bfla/privesc/stored_xss need a state-mutating write + an operator-defined
dangerous-state predicate — an LLM-authored predicate can be crafted to always fire); OOB (ssrf/xxe, per
operator choice earlier). `authentication` has no oracle (honestly not fact-capable). `llm_injection` got a
`prompt_injection` alias (Wave 4).

**Beyond FACTs (shipped):** Wave 5 — native source review from chat (`mode="sast"` → `framework.v2 analysis
review`, distinct from Docker-Strix; `scan_sast` proposal uses the SERVER-computed scan_root). Wave 6 —
codebase Q&A grounded by a `graphify-out/GRAPH_REPORT.md` inside the attached root (bounded, root-confined,
symlink-refused). **Wave 7 (the core "not end to end" fix)** — a paused run no longer reports "✓ Done": the
engine emits a terminal `run_summary` spine event, the supervisor marks status `paused` (not done), the UI
shows an amber "⏸ Paused — <why>" + a Resume button, and approving auto-resumes a run that paused awaiting
the signature. Wave 8 — an `awaiting_approval` notice now posts to the chat transcript (not only the
floating process box), and the offense approval popup gained Deny-&-redirect. Wave 9 — `make deploy-console`
(mirrors a fixed manifest to the running-demo tree + restarts; kills the two-tree skew) + slash verbs
(/resume /stop /approve /findings /report /status). Deferred polish: real thinking stream, usage on
non-streamed turns, engine plan records, @-mentions, bubble timestamps.

**Gotchas:** a framework-dependent integration test must be BOTH `--ignore`'d in the sovereign CI leg AND
listed in the offense leg (guard `test_ci_framework_tests_run_in_offense_leg`). The pure-JSON predicate
`not` op takes a SINGLE node (`{"not": {...}}`), not a list. The evidence-branch registry is
contract-enforced (`test_claim_discipline` + `tool_manifest.validate_branch_registry`): response_body ⇒
clean_capable:false + `body_semantically_available` precondition; fact_capable ⇒ a real `oracle_version`;
implementation_refs file:symbol must exist; a lowered target needs a `target_downgrade_rationale`. Env:
`test_engine_live.py` has ~9 PRE-EXISTING failures here (executor can't run + anthropic SDK unavailable) —
verify by stashing before blaming a change. Two-tree deploy remains ([[vigil-meridian-range-program]]).

**LIVE-FIRE VALIDATION (real key, opus-4-8, MERIDIAN :19010):** the AUTONOMOUS LLM path now mints real
signed FACTs for the new classes — verified live for open_redirect (`web:open_redirect`), path_traversal
(`rt:path_traversal`) and exposure (`rt:exposure`), each in 2 iterations. Deterministic `--replay` also
proved open_redirect/cors/path_traversal/exposure/reflected-xss mint against the range AND a false claim
(path_traversal on a non-traversal endpoint) mints ZERO. Two real end-to-end bugs were found+fixed by
live-fire: (1) `OutputAnalysis.findings` required list-of-dicts but the model emits strings → the whole
LLMDecision was rejected, dropping exploit_succeeded → a lenient `@field_validator` coerces it
(agent/state.py; findings are advisory, so soundness is unaffected); (2) the model is BLIND to tool output
by design (`_recent_actions_digest` surfaces args+outcome, never the response bytes — secret-safety), so the
re-drive prompt was reframed to have it claim exploit_succeeded from the REQUEST shape it probed (the oracle
re-drives + verifies, so a wrong guess stays a LEAD). Gotchas: `anthropic` SDK lives ONLY in the demo venv
(`/home/kali/vigil/.venv-offense`), not vigil-dev; `vigil` on PATH runs from `/home/kali/vigil`; a live
engage needs `dangerouslyDisableSandbox` (Bash sandbox drops TCP incl. loopback); `rm -rf` in a compound
Bash command gets permission-denied — split it out; `make deploy-console` (the whole `vigil_integration`
package) is how new code reaches the running demo.

**RESOLVED — approve→FACT is SOUND (the "0 facts on resume" was a TEST ARTIFACT, not a product bug).**
A dead-man's-switch bounds a signed approval token: `consume_token` rejects any token whose window
(`not_after - not_before`) exceeds `ApprovalPolicy.max_token_lifetime = 900s` with reason
`"token window exceeds max lifetime (dormant-token bound)"` (approval_token.py:260). The prior
diagnostic minted tokens with 3600s/7200s windows → every consume rejected → 0 FACTs, which looked
like a broken resume. The UI's `sign_pending` mints a **300s** window (`_TOKEN_TTL_SECONDS`,
offense_approvals.py:32) — well within the bound — so real approvals are valid. Proven live vs
MERIDIAN open_redirect with a controlled owner keypair, BOTH paths mint `[open_redirect] oracle-
confirmed exploit` in ≤3 iterations:
  1. **Primary path** (service default `VIGIL_APPROVAL_WAIT_SECONDS=300`): the child engage
     BLOCK-WAITS in `ApprovalBroker.token_source`'s poll loop; approving mid-wait (sign a 300s token)
     is picked up within ~0.25s → the SAME process consumes + continues + mints. NO resume needed.
     The chat launch does NOT set the wait env (actions.py `_integration_engage_cmd`), so it inherits
     the service default 300s — i.e. approve-then-continue is normally IN-PLACE, not a resume.
  2. **Resume path** (`wait=0`, or the 300s window elapsed → run PAUSES `awaiting_approval`): a later
     `vigil engage … --resume` (retry_run appends `--resume`, reuses parent argv incl. `--run-key`)
     rebinds the same action, `find_signed_token` matches by ACTION (tool,target,action_digest —
     nonce-independent) + `consume_token` burns single-use → mints. UI auto-fires this via
     `_afterOffenseApprove`→pboxRetry (1500ms) only when `PBOX.run.status==="paused" &&
     paused==="awaiting_approval"`.
The user's live 0-facts was environmental: testing BEFORE the Wave 7 + httpx-label fixes deployed,
5 tangled CONCURRENT same-slug runs (shared `{slug}.spine`; `retry_run` refuses a 2nd running run of
a slug via `_slug_has_running_run`), and diagnostic-burned nonces. Cleaned the stale live approval
queue (8 pending/8 signed moved to `.vigil-live/approvals/_stale-backup-20260906`, reversible; the
`approval-nonces` replay ledger left INTACT). Gotchas: `think()` precedence is client > local backend
> **ANTHROPIC_API_KEY env** > replay > safest — a stale key in the shell makes `--replay` silently
hit the (failing) live path, so `env -u ANTHROPIC_API_KEY` to exercise replay. Open ENHANCEMENTS (not
bugs): distinguish "token found-but-rejected (expired/replayed)" from "waiting for first approval" in
the pause reason so a recoverable state isn't an invisible loop; guard/serialize concurrent same-slug
chat runs.

**FOLLOW-ONS BUILT + SHIPPED (2026-09-06, commit 1cbdae48 on `vigil-model-default-fallback` / PR #769,
pushed).** The two enhancements flagged in the RESOLVED note above are now production, each judge-panel-
designed then adversarially red-penned (the red-pen caught + I fixed a real BLOCK and a real HIGH before
commit; the no-widening lens HELD):
- **ENH1 — distinct `approval_rejected` pause + the loop fix.** When the M2 gate FINDS an owner-signed token
  bound to the action but `consume_token` REJECTS it (expired past the 300s window / already single-use-spent),
  the run pauses as `approval_rejected` (operator told to APPROVE AGAIN) instead of the generic
  `awaiting_approval` invisible loop. SECURITY UNCHANGED: the reject branch still returns
  `outcome="queue"/allowed=False`; consume_token is the sole authority and burns nothing on a reject; a
  CRUCIBLE deny is never widened. The reject reason is a FIXED phrase `"owner approval rejected"` — **spaces
  make it un-spoofable by an echoed target host** (a bare single-token marker was NOT immune — red-pen); it is
  never the raw consume reason, so no planted `token.key_id` reaches the append-only spine. Engine matches it
  BEFORE the awaiting matcher (ordering, not disjointness, is the guarantee). **The actual root-cause of the
  user's real 0-facts loop was SITE 8**: `find_signed_token` returned the lexicographically-smallest
  request_id (window-uncorrelated), so an accumulated EXPIRED shadow file kept winning and every re-approval
  re-rejected it. Fixed by making `find_signed_token(now=, is_consumed=)` PREFER a live, unspent token (pure
  pre-filter; the shared `NonceLedger` feeds is_consumed; consume_token stays sole burn authority). Two paired
  constants in approval_token.py: `APPROVAL_REJECTED_REASON` (gate-reason substring + matcher) vs
  `APPROVAL_REJECTED_PAUSE` (report.paused + UI label). Data flow verified end-to-end incl. UI hops
  (CHAT_ENGINE_KINDS, a red chat bubble before the isLead push, pboxStepText, `_afterOffenseApprove`
  auto-resume widened to approval_rejected). LIVE-VERIFIED on the deployed tree: expired shadow →
  `approval_rejected`; shadow+live token → SITE 8 picks live → `[open_redirect]` FACT minted.
- **ENH2 — globally-unique engagement slug per agentic launch (approach B, chosen by the judge over
  atomic-claim-reuse D and serialize-refuse C).** `_unique_engagement_slug` = base + run_id + a `secrets`
  tail, so two concurrent chat sends own DISJOINT `{slug}.spine` hash-chains (invariant = a property of the
  slug string; storage-level test proves two writers on one path make `verify()` False). Red-pen BLOCK fixed:
  `_new_run_id` made collision-free (random tail) + the retry_run atomic guard no longer trusts
  `exclude=new_id` (a same-ms id collision would have blessed a fork). Red-pen HIGH fixed: the emergency-stop
  containment (`_launch_contained_reason` → `restricted_mode.is_restricted`, refuses a new agentic launch in
  restricted mode — compensates for a fresh slug's own untripped kill-switch) now records+reads the ENTER
  under the SAME `_live_base()` (`run_emergency_stop` pins `--base-dir`), so it is NOT inert in the
  VIGIL_LIVE_DIR production setup. retry_run/resume-on-reply/engage_instruct all key off the recorded unique
  slug (continuity preserved). 20 new tests + touched-surface regressions green.
- **Workflow/process:** used a design judge-panel workflow (B>C>D) then a 4-lens red-pen workflow
  (no-widening/enh2-fork/enh1-correctness/mustfix-compliance) → verify pass. Deploy via `make deploy-console`
  (running proxy at :8770 now serves the enh1 UI hops). Deferred (named): stale signed/pending file GC after a
  burn; per-run `{slug}.spine`/charter retention/GC (unique slugs multiply artifacts).

**WHOLE-APP ROAMING + GOVERNED MULTI-AGENT FROM CHAT (2026-09-07, PR #777 MERGED → main 7c5d09bd).**
The chat only drove the single-loop agentic `vigil engage` (no crawler), so "find all vulns on the app root"
did one probe and stopped. Root cause (3 grounded Explore agents): **TWO engage engines** — the agentic
`VigilEngine` (integration/vigil_integration/live/engine.py: LLM OODA, NO crawl, never fed discovered
endpoints/tool-output, only counts+seed+last-3) vs the **framework** `python3 -m framework.v2 engage
--autonomous` (engine/crucible/framework/v2/engage.py: real crawler scanner/crawler.py + WebScanCampaign +
recon/VOI + autonomous discover/crawl-expand/multi-probe), reachable only via New Assessment mode=suite. The
chat never routed to it. Fixes:
- **Phase 2 (whole-app):** chat_send routes a "whole app / find all / /scan-all" request on a LOOPBACK url to
  `launch_assessment(mode="suite", scan_mode="deep")` (the autonomous roamer) instead of the single loop; and
  launch_assessment auto-provisions the loopback charter (reuse `wiring.ensure_loopback_charter`) so the
  framework engine stops refusing `charter_missing`. PROVEN: the autonomous engine crawls MERIDIAN from `/`
  (discovers /apply,/login,/records/search,/track), multi-probes, confirms boolean+error SQLi + 122 passive.
- **Phase 3 (multiple agents, GOVERNED):** the fireteam (≤5 LLM specialists, ≤3 concurrent, capped A1/A2
  never A3) is now deployable from the chat — the think prompt documents the member schema; the approval()
  seam gained a **per-action `deploy_fireteam` token** (bind ApprovalAction("deploy_fireteam", slug,
  action_digest(...members)) → publish pending → consume_token is the SOLE authority: signed, single-use,
  member-bound) — NO blanket `--approve-offense`; the existing offense Approve/sign_pending signs it; app.js
  shows "Approve fan-out". The fireteam fans out over ROLES (not endpoints) — whole-*app* coverage is Phase
  2's crawl; the fireteam adds parallel specialist depth.
- **Red-pen (essential; found real issues):** fan-out gate SOUND. **HIGH fixed** — the Phase-2 whole-app
  suite route minted a fresh slug whose kill-switch is untripped, bypassing restricted-mode (emergency-stop)
  containment (the agentic branch's `_launch_contained_reason` was skipped); the framework-engage branch now
  consults it and refuses while restricted. Hardening: `_is_loopback_url` now validates the host via
  `ipaddress.is_loopback` (was `startswith("127.")` → matched `127.0.0.1.evil.com`); explicit member-digest
  re-check in the fan-out token path.
- **GOTCHAS learned:** the framework autonomous engine needs a **signed charter** at
  `CRUCIBLE_ROOT(/home/kali/Music/PENTEST/crucible)/targets/<slug>/charter.md` AND a **greenfield** slug (an
  authority doc without a trust-root refuses W16-2) — run standalone it prompts destructive POSTs on stdin
  (default-deny, no tty). A NEW **console** test importing `vigil_integration.*` FAILS in the CRUCIBLE-core CI
  leg (two-env: integration not on the path) — stub the console function instead of importing across the
  boundary. PR #769 was already merged before this work, so Phase 2/3 needed a NEW PR (#777) off main
  (cherry-pick, since #769 squash-merged) — a push to a closed-PR branch triggers NO ci.yml (its triggers are
  push:main + pull_request).
