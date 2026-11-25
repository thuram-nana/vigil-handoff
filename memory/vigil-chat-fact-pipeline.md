---
name: vigil-chat-fact-pipeline
description: "How the autonomous chat/engage→SQLi-FACT pipeline was completed (PR #769) + the 4 stacked bugs and key gotchas"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-04T00:34:01.551Z
---

The autonomous chat→FACTs path (`vigil engage` OODA loop, what the chat launches) was under-wired
end-to-end: it drove real tools but never minted a FACT. Fixed via **PR #769** (branch
`vigil-model-default-fallback`), verified LIVE — opus-4-8 autonomously mints a **signed, evidence-backed
error-based-SQLi FACT** vs the MERIDIAN range (`http://127.0.0.1:19010/records/search?q=`). Four stacked
layers, each a real bug:

1. **Model targeted the engagement SLUG as the host.** The think prompt's TRUSTED header gave only
   `engagement: <slug>` + objective, no target host → the model built `http://<slug>/…` → egress-guard
   out-of-scope deny → zero tools. Fix: carry the seed on `AgentState.target`, render it in the trusted
   header (`_build_messages`), set in fresh+resume paths. ADVISORY (gate still enforces scope).
2. **Approval.** A stale `<base>/.vigil-live/approval-authority.json` forces the PER-ACTION-TOKEN gate path
   (wiring `effective_authority`), which OVERRIDES `--approve-offense` (the standing-boolean path only runs
   when NO authority file). Move it aside → `--approve-offense` fires the first tool (single-use standing
   grant = at-most-one auto-fire/run, by design). `sqlmap` is destructive → needs m-of-n owner keys (by
   design; not auto). httpx is A1 recon (in `_RECON`) and auto-runs. The FACT path is the NON-destructive
   error_based_sqli re-drive, no destructive tool needed.
3. **Re-drive unreachable (charter).** The T2 SQLi re-drive rides a CRUCIBLE `HttpExecutor` whose scope gate
   reads `targets/<slug>/charter.md` (`ethics.require_charter_signed`+`require_in_scope`) — a SEPARATE
   authorization from the engine's signed `--scope`. Autonomous slugs have no charter → `charter_missing` →
   `reachable=False` → LEAD. Fix: `wiring.ensure_loopback_charter(slug, scope)` auto-writes a signed charter
   ONLY when every scope host is loopback (never overwrites, honestly-labelled Signed line, external hosts
   still need a real charter). Charter format: `## 2. In-scope systems` md table (first col=host) + a
   non-placeholder `Signed:` line.
4. **Signature truncated (body cap).** `gated_fetch` returned only the first `_BODY_EXCERPT_BYTES`=**8 KiB**
   of the body, and oracles fire over THAT excerpt (full body only archived to evidence/). MERIDIAN's SQL
   error renders at byte ~12.6 KiB → oracle never saw it → admission minted nothing. Fix: bump to **64 KiB**
   (`engine/crucible/framework/v2/agents/http_executor.py`).
   Also documented `output_analysis.extracted_info` re-drive fields (`bug_class`/`insertion_point`/
   `request_payload`) in the think `_SYSTEM_PROMPT` so the model can TRIGGER the re-drive.

Re-drive chain: LLM use_tool + `output_analysis.exploit_succeeded` + extracted_info → engine `_redrive_spec`
→ `wiring._live_redrive_fact` builds a gated `HttpExecutor`+`LiveHttpAdapter.run_exploit_trial` (re-drives
`?<param>=<payload>&rc=<nonce>`) → `_context_from_body` → `_admit_and_mint(provenance="live_redrive")` →
error_signature oracle re-fires over FRESH bytes → signed FACT (LLM's claimed context is DISCARDED).

**Gotchas (cost me time):**
- `CRUCIBLE_ROOT` is the DATA root (`targets/`, `.authority/`), NOT the python import path — `vigil engage`
  imports `framework` from `/home/kali/vigil/engine/crucible/…`. I read the wrong http_executor copy under
  `/home/kali/Music/PENTEST/crucible` for a while (that copy's `gated_fetch` doesn't even exist).
- The running `vigil up` process's ANTHROPIC key went **DEAD (401)**; a valid key lives in
  `~/.sigil/sigil.env` (suffix OQAA). The "executor hang" from the prior session was really the dead key +
  normal multi-iteration LLM latency — the executor runs httpx in ~0.86s in-process.
- `think()` uses the live Anthropic client whenever `ANTHROPIC_API_KEY` is set and IGNORES `--replay`; for a
  deterministic keyless replay run use `env -u ANTHROPIC_API_KEY`.
- To make the LIVE chat UI demo mint a FACT, the running `vigil up` must be restarted with the valid key
  (engage subprocesses inherit its env). The code fixes are already picked up by new engage subprocesses
  (fresh import), but the dead key blocks reasoning until restart. See [[vigil-meridian-range-program]] for
  the two-tree gotcha (edit+RESTART `/home/kali/vigil` to affect the running demo; PR from `vigil-dev`).

**Act 4 (chat act) re-run 2026-09-03 — the PER-ACTION approval path (distinct from the --approve-offense
recipe above) + 3 findings:** the CHAT launcher (`chat_send`→`launch_assessment`, agentic) deliberately
does NOT pass `--approve-offense`, so a chat-launched engage QUEUES its first httpx recon for a SIGNED
owner approval and PAUSES (`awaiting_approval`, run exits `resumable`). The OODA engine's queue leg
(`engine.py` ~360) is NON-blocking: `_approved()` checks for an EXISTING signed token then pauses; the
ApprovalBroker's 300s `VIGIL_APPROVAL_WAIT_SECONDS` block-wait is the **Strix sidecar** path, NOT this loop.
So the honest chat→FACT is pause → sign → RESUME. Verified opus-4-8 STILL mints `FACTS (oracle-confirmed,
signed): 1` via the proven `--approve-offense` path (authority file moved aside; single-use standing grant →
httpx ran=1, denied=5, re-drive → FACT).
- **BUG (fixed #da80d6c8):** `api._pending_offense` (OFF /api/status.pending_approvals) referenced bare
  `os.environ` with `os` never imported in that scope → `NameError` swallowed by `except→0` → counter read 0
  forever while `api.approvals()` (local `import os`) listed them. Fix: local `import os`. Guard:
  `integration/tests/test_offense_pending_count.py` (listed in ci.yml offense leg).
- **GAP (fixed #ee7917b9, S1b):** the chat PBOX approval poll read only SOV `/api/snapshot` (sovereign
  seq-based approvals), which does NOT carry OFFENSE broker approvals — so a chat-launched engage's queued
  tool never surfaced in chat. Fix: `pboxApprovalPoll` ALSO polls `OFF /api/approvals/loopback`; new
  `pboxOffenseCard`/`pboxOffensePop`/`pboxOffenseMaybePop`/`pboxOffenseReset` render+pop them wired to the
  EXISTING route-via-sovereign handlers `offenseApprove`/`offenseDeny` (SOV `/api/action`
  offense_approve/offense_deny/offense_bind_authority in `apps/sigil/sigil/ui/offense_approvals.py`; cockpit
  signs with the owner key, offense console stays KEYLESS) + the `vigil approve sign` CLI fallback. jsdom +
  `docs/tests/test_chat_offense_approvals.py`.
- **FINDING #3 (FIXED — run-keyed checkpoint partitions; red-pen SHIP):** `--resume` of a paused-at-
  `awaiting_approval` run NO-OP'd (12 instant empty iterations) because the state-checkpoint spine was keyed by
  SLUG ("loopback"), so `rebuild_head("loopback")` read the GLOBAL latest loopback checkpoint — a PRIOR
  completed run's `done=True` — not this run's pause → done-guard `start_it=max` (engine.py:276-277) → empty
  loop. **Fix = RUN-keying** (NOT session-keying: a design workflow caught that one chat drives MANY engages
  under one stable session, so a completed earlier engagement's higher-seq done=True snapshot outranks a later
  paused one in the same session partition — the hole stays open). 8 edits: `EngineConfig.run_key` + ONE shared
  `_ckpt_key = (run_key or "").strip() or slug` used by BOTH `checkpoint()`+`rebuild()` seams (wiring.py; keying
  them apart = fatal asymmetric FRESH-start re-fire); `--run-key` CLI arg + config wire (cli.py); launch injects
  `gcmd += ["--run-key", run_id]` (actions.py ~2388); `trusted_finding.py:157` → `spine.rebuild()`
  (engagement=None; run-keyed checkpoints are no longer slug-tagged, per-slug `{slug}.spine` file = global-latest
  is behaviour-preserving). NO engine.py change (`start_it=prior.iteration+1` already correct; rewinding it would
  DOUBLE-FIRE the crash case). NO retry_run change (it reuses the parent argv; the bare run_id survives the
  run-DIR path-rewrite, so a `--resume` inherits the parent's `--run-key` = reads the parent partition;
  retry-of-a-retry keeps the ORIGINAL run_id). Blast radius = exactly those 3 state-spine sites (grep-verified).
  Tests: BLOCK-3 in test_engine_resume.py (two runs, one shared spine file, distinct run_keys: the paused run
  resumes to ITS OWN state + the tool fires + the pre-fix bug reproduced) + a build_engine round-trip guard in
  test_engine_live.py (run_key threads into the seams; `run_key=""`→slug fallback). 156 pass (only the
  pre-existing DNS-mock test_live_remote fails, unrelated). Deployed + verified: a live chat engage carries
  `--run-key <its run_id>`; token-mode runs now BLOCK at execution for the signature (`token_source` 300s wait).
  **2 LOW follow-ups (red-pen SHIP, deferred):** (1) `_new_run_id` (actions.py:442) is ~1ms-resolution → same-ms
  run_id collision → shared partition (pre-existing worse: same-ms already clobbers the run DIR; fix REDUCES the
  window). Harden with a uuid4/counter suffix. (2) a MANUAL `vigil engage --resume` with no `--run-key` falls
  back to the (empty) slug partition → FRESH-start re-firing A0/A1 recon (console/chat never mis-keys; A2+ still
  re-gated, no bypass). Harden: fail-closed refuse when resume=True + run-key partition empty BUT the per-slug
  spine file has other partitions.
- KEYLESS-console signing: offense approvals CANNOT be signed in the offense console (FATAL-2); either
  route-via-sovereign (cockpit holds `VIGIL_APPROVAL_OWNER_KEY`) or CLI `vigil approve
  provision-authority`→`sign`. The classifier BLOCKS `vigil approve sign` (owner action) — needs operator
  approval. Provisioning writes `.vigil-live/approval-authority.json` (the "stale authority forces token-mode"
  file from bug #2 above) — moving it aside re-enables `--approve-offense`.

Related: [[vigil-chat-orchestrator-program]] · [[vigil-meridian-range-program]] · [[anti-hallucination]]
(re-execution not string-trust — the re-drive is exactly that) · [[vigil-engagement-ux-program]] (S1b).
