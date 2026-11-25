---
name: vigil-chat-orchestrator-program
description: Turn the VIGIL command-UI chat into a Claude-Code-class but gate/oracle/two-env-bounded agentic operator
metadata: 
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-08-16T14:49:56.518Z
---

Program: give the VIGIL command-UI **chat** "every Claude Code capability, well fine-tuned to the core"
while staying inside VIGIL doctrine. Governing charter: `engine/crucible/framework/v2/console/CHAT-VISION.md`
("Chat proposes. The gate decides."). Plan: `.claude/plans/parsed-popping-lemur.md`. Operator decisions:
codebase agent = BOTH modes (security + dev) fully UI-controllable · autonomy = bounded-auto + manual ·
egress = gated ON · model sovereignty = **ask per session**.

**Architecture (doctrine-clean):** chat lives in the OFFENSE-console plane (`framework.v2.console`), keyless.
It PROPOSES + ORCHESTRATES the existing gated integration engine (`vigil_integration`); it never runs the
agent loop in-process and never mints a fact. B0 insight: the integration `vigil engage` engine (OODA +
mid-run steer + --resume + approval broker + DEPLOY_FIRETEAM) runs **without NEO4J** (graph optional) — so
routing chat's agentic engagements to it was cheap. Engine→console bridge = `wiring.py` `_build_spine_poster`
ALWAYS appends to progress.jsonl, which the process box tails live.

**Shipped (all per-slice red-penned; a real defect caught on nearly every sharp slice):**
- #343-#346: A control surface (proposal chips + 4 distinct answer sources + engagement shape + ask-progress
  + live steps) · deep reasoning modes (ask/research/plan, adaptive thinking) · hypotheses-as-objects
  (auto-close on oracle fire) · engine bridge + B0 agentic routing · B1 mid-run inject (`/api/instruct`).
- #347 D1: gated git clone (SSRF-hardened: `_CLONE_HOSTS` allowlist + urlsplit host-match; confined
  `<live>/clones/<chat>/`; `--`-hardened argv). #348 D2: dev-mode edits (unified diff, `apply_dev_edit`
  git-apply clone-only, A2 gate, killswitch-checked). #349 D3: `run_codebase_tests` → gated `vigil sandbox`
  (A3, `--unshare-all` no-net, signed ExecRecord); `--workspace` added to the CLI verb.
- #350 D2b: diff viewer (XSS-safe: `h()` string child = text node) + chat edit/test affordances. Red-pen
  MEDIUM: panel was offered for ANY codebase run but only a git-CLONE lands under `clones/` → gate
  `codebase_path` on `actions._confined_clone_path` (same predicate the routes enforce) so the panel appears
  iff its actions succeed.
- #351 E3: per-session model sovereignty. `chat_models()`+`/api/chat/models` (trust class + permitted +
  consequence). `_reason(model=...)`: cloud pick = direct-SDK with CHOSEN model (was hardcoded opus-5);
  LOCAL pick = `_reason_local` via `kernel.llm.get_backend(force=...)` (asserts_permitted FIRST) + ONE
  backend, **NO failover** — unreachable/failed local REFUSES, never egresses. Red-pen BLOCK: "local" was
  name-classed → a remote self-hosted/Ollama endpoint would egress under a "nothing leaves this machine"
  label → fix `_endpoint_host_is_local` ENFORCES loopback (backend.base/host) before the call. Factored
  `_assemble_reason_parts`+`_reason_finish` (cloud byte-equivalent).
- #352 E1: fireteam VISIBILITY. `deploy_fireteam` now passes a `SingleWriterSpineQueue` whose writer routes
  each REDACTED member step to the chat feed via `spine_post("fireteam",…)` (KIND_META `fireteam`, attributed
  by role, lead-not-fact) + a `ConfirmationRegistry(spine=…)`; member_runner emits think/claim/escalation.
  Red-pen BLOCK: it read `decision.rationale` (nonexistent — field is `reasoning`) so the headline "see the
  agent work" was DEAD + the redaction test was vacuous (pydantic dropped the bad kwarg); fixed both + moved
  `registry.register` BEFORE the flush so the ledger is durable. Composer "Fireteam" affordance (honest nudge).
- #353 F1: streamed replies. `_reason_stream_cloud` (messages.stream) + `chat_stream` + `POST /api/chat/stream`
  (SSE-from-POST, lazy headers, JSON fallback); shared `_cloud_gate_and_key` (no stream can bypass the
  sovereignty ladder) + shared `_finish_question_turn` (streamed record ≡ non-streamed). Local pick answers
  non-streamed. Red-pen MEDIUM: client re-sent on a mid-stream disconnect AFTER the server committed →
  duplicate turn+billable call → fix `committed` flag + `{incomplete}` marker (re-send iff nothing persisted).
- #354 F2: cohesive layout — run-options grouped into one labeled `.chat-runopts` panel (behavior-identical;
  red-pen PASS).
- #355 F3: docs capstone (CHAT-VISION built-vs-charter table + honest follow-ups; FEATURES `### Chat` rewrite;
  KB routes). Red-pen BLOCK: the honesty doc ITSELF overclaimed — said F1 streamed reasoning is F3-scrubbed
  (it is NOT; only the E1 feed + the model INPUT are) → corrected to state the per-surface truth.

**Key facts:** `kernel/llm.py` provider layer is STRUCTURED-output only (`Prompt` needs a pydantic schema,
returns `LLMResult.parsed`) — local chat uses a `{reply:str}` schema. `kernel/sovereignty.py`: `classify()`,
`current().tier` (env `CRUCIBLE_SOVEREIGNTY_TIER`, read per-call unless sealed), `assert_permitted(name)`,
`direct_anthropic_backend_name()`. All codebase routes re-confine the client path server-side. `LLMDecision`
field is `reasoning` NOT `rationale`. POST routes gated by `_same_origin_as_console`; GET reads by
`_host_is_console`+`_token_ok`. P5 has a known egress-guard seccomp flake (re-run clears it).

**PROGRAM COMPLETE (A–F, #343–#355) + 4 FOLLOW-UPS (#356–#357).** Follow-ups:
- #356 polish: G2 redaction — investigation found the export paths ALREADY scrub (dossier `_scrub_jsonl_file`
  w/ `test_no_secret_leaks_into_the_session_dossier`; linked-chat `session_context` `_redact_ctx`); attachment
  input is CONSENTED egress → corrected the doc (was pessimistic). G3 — `finishSend` swallows render errors +
  chat_stream emits a `start` frame before the append so a post-commit abort never re-sends (no double turn).
  G4 — chat_stream appends BEFORE the reason-wanted check (shared `_finish_need_target`) so the first
  conversational follow-up STREAMS.
- #357 G1 Tier A: fireteam escalations recorded as a STRUCTURED row `RunReport.fireteam_escalations`
  (wave/member/seq/tool/target/tier/reason/status), target+reason SCRUBBED at source (redact_tool_args),
  row-build fully guarded (engage never raises). Red-pen caught: "durable" overclaim (RunReport is IN-MEMORY;
  the durable ledger is the ConfirmationRegistry) + verbatim target/reason → both fixed.
Remaining (honest, tiered — the fireteam resolve loop): **Tier B (sign)** — publish escalations via the
approval-token/broker stack + feed a signed ApproverFn into `ConfirmationRegistry.resolve` (fail-closes to
REJECTED w/o one); **Tier C (run)** — a new escalated-edge runner lifting the member A2 cap for exactly the
token-bound action (deliberately the hard boundary: members are ≤A2, never self-approving). Also: the F1 chat
ANSWER live view + on-disk transcript stay raw (operator-local; every EXPORT path scrubs).

See [[vigil-crucible-core-gateway-importorskip.md]] (P5 vs CRUCIBLE-core import guard), [[claim-discipline-meet-up]]
(build up, never soften), [[vigil-live-external-fact]].
