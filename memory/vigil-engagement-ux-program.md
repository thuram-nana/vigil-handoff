---
name: vigil-engagement-ux-program
description: "The Claude-Code-class engagement UX slices for the VIGIL command UI (PR #769) + how the UI is tested headlessly"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-03T20:39:23.661Z
---

Making the VIGIL command UI (`packages/vigil-ui/app.js`, a ~10.5k-line custom hyperscript app: `V=window.VUI`,
`h=V.h`, `V.card/pill/tile/icon/mount`, a hash `router`, a slide-in `#drawer`, a `renderLive` per-run view
with SSE `attachStream`) a "full Claude-Code engagement experience." Built as tested increments on **PR #769**
(branch `vigil-model-default-fallback`); each slice = app.js/css + a headless jsdom behavioural test (dev-time)
+ a CI-runnable **docs/tests** drift guard (the docs-only leg reads the files; precedent:
`test_w17_10_agentic_run_renders_completed.py`).

Slices shipped (all verified end-to-end):
1. **Resizable / dockable / maximizable activity panel** — the `#drawer` gained a drag handle (`#drawer-dz`),
   dock right↔bottom, maximize, all PERSISTED in `localStorage "vigil.drawer"`, re-applied on `openDrawer`.
   New CSS `#drawer.dock-bottom/.max/.dz` + `--drawer-h`; 4 glyphs (maximize/minimize/dock-right/dock-bottom)
   added to the shared icon map in `ui.js`. Guard: `docs/tests/test_drawer_dock_resize_controls.py`.
2. **Per-run "Stop run"** (kill one of two same-prompt runs) — the backend already had real per-run kill
   `cancel_run(run_id)` at `/api/run/<id>/cancel` (SIGTERM→SIGKILL of the run's own pid, boot-guarded,
   idempotent); the Live-view Stop button was wrongly tripping the per-SLUG kill-switch. Now Stop = per-run
   cancel; a separate "Halt engagement" = the kill-switch; a `run <id>` pill tells duplicates apart. Real
   two-process offense-leg test `integration/tests/test_per_run_cancel_isolation.py` (LISTED in ci.yml offense
   leg — required by `test_ci_framework_tests_run_in_offense_leg`) + guard `test_per_run_kill_controls.py`.
3. **Interrupting approve/deny "propose" popup** — reuses `openModal(title,body,actions,opts)` (returns
   `{close,card}`). `maybePopApproval` baselines existing approvals on the FIRST snapshot (no nag), then a NEW
   proposal mid-watch pops ONE modal (approve/deny → same `act()` the cards use); module-level
   `liveApprovalModal` closed by `teardownLive()` (runs on every screen change). Guard:
   `test_approval_propose_popup.py`.
4. **Pause/end with memory kept + Resume** — `retry_run` at `/api/run/<id>/retry` resumes a `resumable` run
   (`--resume`, continues the spine from the last checkpoint); integration engages are `resumable:True` at
   launch and `list_runs` surfaces `status`+`resumable`. Live-view header now shows Resume run (resumable) /
   Restart run once ended, a "memory kept" pill, and Stop copy states the spine is KEPT. Guard:
   `test_lifecycle_resume_controls.py`.

Already existed (leveraged, not rebuilt): live SSE streaming (`renderLive.attachStream`), mid-run steering
(type-to-steer `injectIntoRun`→`/api/instruct`), the ask_user sticky banner, run selection across run_ids.

**Headless UI test recipe** (no repo JS toolchain — I `npm i jsdom` in the scratchpad): load `ui.js` via
`window.eval` → `window.VUI`; expose DOM globals (`Node`/`Element`/`MouseEvent`/… via `global[k]=window[k]`,
skip getter-only `navigator`); slice the REAL function bodies out of app.js with a brace-matcher and
`new window.Function(...deps, sliced + "; return {..}")`; drive with dispatched `MouseEvent("pointerdown"/…)`.
jsdom does NO layout → `getBoundingClientRect()` is 0 (persist the APPLIED inline size, not a re-measured rect
— a real improvement I made to the drawer resize).

**CHAT parity (all 12 scouted slices) — COMPLETE + DEPLOYED (PR #769).** After a deep scout of chat vs
Claude Code, brought the CHAT surface to parity, all tested (real-code jsdom + docs-leg drift guards +
offense/integration tests) and deployed to the running instance:
- **R0** shared `makeApprovalUX` factory (Live+chat one impl); **S1** approve/deny/**Deny-&-redirect** interrupt
  in chat (PBOX polls SOV `/api/snapshot`, pops on any screen but Live); **S2** multiple-choice ask_user +
  **"Other → type your own"** (backend `question_options` plumbed think→react→engine event→`post_agent_question`);
  **S3** message copy/edit-resend/regenerate; **S4** stop-generation (AbortController + Stop + Esc, no re-send);
  **S5** inline tool-call cards (hoisted `toolCardBody`/`toolPairFrom`; PBOX tool rows open the card); **S6**
  composer slash-commands (`/scan /url /plan /research /model /fireteam /new`, unknown → normal message); **S7**
  thinking indicator + token/context meter (`res["usage"]` from the stream); **S8** @-mentions (files + linked
  chats); **S9** live plan/hypothesis CHECKLIST (state icons + N/M-confirmed); **S10** XSS-safe markdown (DOM
  text nodes only, http(s)/mailto links only); **S11** image attachment thumbnails; **S12** rename-via-modal.
- Deployed by mirroring `packages/vigil-ui/{app.js,components.css,ui.js}` + `console/chat.py` + the integration
  files to `/home/kali/vigil` and `systemctl --user restart vigil-command.service` (service.env carries the
  valid key). Live smoke: all 12 features served + an oracle-confirmed SQLi FACT still minted.
- Headless-UI test recipe unchanged; a promise `.then` (e.g. `after()`) needs an `await tick()` in jsdom before
  asserting. Every slice: real-code jsdom harness + a `docs/tests/test_chat_*.py` drift guard (53 chat/live
  guards green); backend slices add integration/offense tests.

**Deploy mechanics (both Live + chat work are now DEPLOYED):** the running demo is the systemd user unit
`vigil-command.service` (ExecStart `vigil up --port 8770`, `EnvironmentFile=/home/kali/vigil/.vigil-live/
service.env`). I put the valid key + `CRUCIBLE_ROOT` + `CRUCIBLE_ANTHROPIC_MODEL=claude-opus-4-8` into
`service.env` (0600), so a `systemctl --user restart vigil-command.service` brings it up with a live key and
`run_up` reassembles the served UI dir from `packages/vigil-ui/`. To deploy: copy the changed files
`vigil-dev`→`vigil` (vigil-dev is the canonical superset; back up + import-check first — the running tree was
~10 behind), then restart. See [[vigil-meridian-range-program]] (two-tree gotcha) and
[[vigil-chat-fact-pipeline]] (chat→FACT work, same PR #769). PR #769 (`vigil-model-default-fallback`) now
carries: opus-4-8 default+fallback, chat→FACT 4-layer fix, Live-view UX, and all 12 chat-parity slices.
