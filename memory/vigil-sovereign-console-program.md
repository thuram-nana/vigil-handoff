---
name: vigil-sovereign-console-program
description: "VIGIL SOVEREIGN CONSOLE program — full UI control of every flow + one all-up startup + live preflight (missing-dep report + % bar) + README; APPROVED plan, WS0 first"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-08-25T21:20:57.569Z
---

**APPROVED program (2026-08-25)** — make VIGIL fully browser-controllable end to end + a single
startup that brings up *everything*. Plan = the **top section of `~/.claude/plans/parsed-popping-lemur.md`**
(the STRIX+HEXSTRIKE plan is parked below the divider, still the offence-engine-wiring reference).

**Operator decisions (binding):**
- **Full UI control of EVERY flow**, incl. the sensitive ones (m-of-n destruction ceremony, owner-key
  signing, PR-opening auto-patch). **Guardrail (non-negotiable, not a silent weakening):** the browser
  *drives* these; m-of-n thresholds, owner Ed25519 signatures, gate chain, kill-switch, and offline
  verifiability stay **server-enforced**, and private key material **never traverses the browser**
  (signing runs on the plane that already holds the key). WS3 Tier-3 slices get **mandatory per-slice red-pen**.
- **Startup builds every image if-absent + starts every container** ("once PC on, everything works") and
  the UI shows a **percentage progress bar** for build/pull/health/plane-start.

**5 workstreams, phased PRs (order: WS0 → WS1∥WS2 → WS3 → WS4):**
- **WS0** shared UI surfaces: fix `getJSON` (packages/vigil-ui/ui.js:90-93 — throws only status+url, MASKS
  4xx/5xx as "offline/empty"; mirror `postJSON` at ui.js:95-104); global failure banner + default `sse()`
  onError (ui.js:218-223) on `readyState===CLOSED` only (not transient reconnects); reusable %-bar +
  `{op,label,pct,phase,error?}` SSE progress protocol (mirror the `onProgress(sent,total,phase)` idiom in
  ui.js `uploadChunked`).
- **WS1** one all-up startup + structured preflight: default `vigil up` to services (add `--no-services`);
  ensure-images-if-absent for strix+aegis (mirror gateway's `ensure_content_addressed_image` in
  gateway/vigil_gateway/docker.py:459-500); `vigil doctor` gains `docker info` (daemon — today only binary
  checked, doctor.py:844), offense-tool probes, venv health, images; per-item `{present,how_to_install}`;
  run doctor as a gating preflight streamed to UI; endpoint-aware service detection (services.py:66-102 →
  fixes false "qdrant absent" while sigil-qdrant serves 6333).
- **WS2** error/progress everywhere: docker bring-up % bar + stop/restart (replace silent `.catch(){}` at
  app.js:5572); standardize long-op feedback; read panels show the real server error.
- **WS3** surface every CLI-only capability. Backbone: new console route SPAWNS the integration CLI (never
  import across FATAL-2), add SPA control, flip its `docs/features/wiring-status.json` row (CI guard
  docs/tests/test_feature_wiring_status.py). Tier1 backup/restore + doctor-run + posture-mint/reprove +
  verify-integrity/ledger; Tier2 gauntlet, fireteam approve/resolve, witness/floor+upgrade, emergency-stop,
  MCP start/stop, wire the orphaned gated-API `/offense/api/v1` (uiproxy.py:942 — zero browser callers today);
  Tier3 sensitive (destruction ceremony, owner-key signing, PR-open `vigil patch --open-pr`). Background
  daemons get a status strip, not buttons.
- **WS4** README rewrite: single-command quickstart near hero; Preflight section (doctor + bootstrap
  consented-install); complete prereqs (TPM2-tools, venv names/ports, built images; "no Node needed");
  feature list generated from `docs/features/wiring-status.json` (32 screens) + honest "CLI-only (today)" list.

Backbone facts: one origin :8770 (uiproxy.py) federates sigil :8733 `/sovereign/*`, crucible console :8787
`/offense/*`, gated API :8799 `/offense/api/v1/*`. Boot-persistence already = `vigil-command.service`
(reuse). Honesty backbone = `docs/features/wiring-status.json` (92 features, audit_version 2).
Related: [[vigil-strix-hexstrike-wiring-program]], [[vigil-up-systemd-autorestart]], [[vigil-command-ui-program]].
