---
name: vigil-up-systemd-autorestart
description: "VIGIL `vigil up` is managed by a systemd unit that auto-restarts it and rotates the token; edits to uiproxy.py need a service restart, not a hot reload"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-08-25T20:00:16.282Z
---

`vigil up` on the /home/kali/vigil box is managed by the **`vigil-command.service`** systemd
unit (`infra/systemd/vigil-command.service`), which runs `vigil up --port 8770 --no-browser`.

Consequences that cost time if you don't know them:

- **It auto-restarts.** `vigil down` stops the current stack, then the unit brings it right back
  up as a NEW process — so a manual `vigil up` right after `vigil down` refuses with "port already
  in use" (the service won the port). Don't fight it; let the service own the lifecycle.
- **The session token rotates on every (re)start.** After any restart the URL is
  `http://127.0.0.1:8770/?token=<NEW>`. The user must reload with the new URL (read it from
  `.vigil-live/ui/index.html`). A restart is therefore disruptive to an open browser tab.
- **The offense venv imports the engine from SOURCE (editable install):**
  `.venv-offense/bin/python -c "import vigil_integration.uiproxy as u; print(u.__file__)"` →
  `/home/kali/vigil/integration/vigil_integration/uiproxy.py`. So a source edit to `uiproxy.py`
  (e.g. a new proxy route) takes effect **on the next service restart**, NOT hot-reloaded into the
  running proxy. A running proxy that predates the edit answers the new route with 404.
- **`run_up` re-assembles the serve dir from `packages/vigil-ui` at startup**, so a service restart
  also picks up new `app.js` / `components.css` automatically — no manual serve-dir patching needed
  after a restart.
- To confirm a proxy has new code without stopping anything: GET a POST-only plane route with a valid
  token — new code returns 405 "method not allowed (POST only)", old code returns 404 "not found".

**Token retrieval (Claim 6: the token is NOT embedded in the served page):** read the current URL from the
journal, NOT index.html — `journalctl --user -u vigil-command.service --no-pager | grep -oE "http://127.0.0.1:8770/\?token=[A-Za-z0-9_-]+" | tail -1`.

**SESSION 2026-08-25 — made the whole system boot-persistent + added browser hot-reload.**
- **Crash-loop root cause + fix:** `vigil-command.service` was crash-looping ~31,837× because `vigil up`
  aborts when the sovereign cockpit (`sigil serve`) refuses to start — the **SIGIL spine was in the LEGACY
  single-file layout**. FIX = `vigil upgrade` (backs up to `~/.sigil/backups/` + verifies + refuses a
  non-verifying spine; `--check` reports, mutates nothing). After migration `vigil up` starts both planes
  (sigil serve :8733 + crucible console :8787 + crucible api :8799 + learn-drain).
- **`vigil` MUST be resolvable** or `vigil doctor` flags "blocks bring-up": Terminal Run, agentic/fireteam
  engage, AND the B3 propose button all resolve `_vigil_bin()` = `$VIGIL_BIN` or `which('vigil')`. FIX =
  `ln -sf .venv-offense/bin/vigil ~/.local/bin/vigil` + `VIGIL_BIN=…/.venv-offense/bin/vigil` in the service's
  `EnvironmentFile` (`.vigil-live/service.env`, referenced `EnvironmentFile=-…`).
- **Boot-persistence = 3 things:** (1) `loginctl enable-linger kali` — WITHOUT linger, user services only
  start at login, NOT at boot; (2) `systemctl --user enable vigil-command.service` (WantedBy=default.target);
  (3) docker daemon enabled + containers `docker update --restart unless-stopped vigil-neo4j vigil-otel`
  (sigil-qdrant already was; `vigil services status` reports them "absent" because it looks for its OWN
  container names, but sigil-qdrant already serves 6333 — don't `vigil services up`, it'd collide).
- **HOT-RELOAD (browser refresh shows new code, no manual relaunch)** = a new **`vigil-dev-watch.service`**
  (enabled, boot-persistent) running `.vigil-live/dev-watch.py` (a gitignored stdlib mtime-poller): a
  **UI** change (packages/vigil-ui) → call `uiproxy.assemble_serve_dir(src, .vigil-live/ui, token="")` (the
  `token` arg is NEVER written into any asset, so re-assembling is byte-correct) → the content-hash **build
  id** changes → the `?v=<build>` cache-bust changes → refresh re-fetches, **NO restart, token/session
  preserved**; a **backend .py** change (integration / framework console+api / vigil_core; **tests excluded**
  so bucket-work doesn't thrash it) → `systemctl --user restart vigil-command.service` (token rotates).
  GOTCHA: do NOT put `PartOf=vigil-command.service` on the watcher — when the watcher restarts vigil-command,
  PartOf stops the watcher mid-restart (racy). Keep only `After=` for boot ordering.
- **Cadence/durability timers DON'T cleanly run in this rootless user session** — DO NOT blind-enable them
  (they just crash-loop like the bug above): `vigil-checkpoint`/`integrity` fail on `PrivateNetwork`/NAMESPACE
  sandboxing not permitted rootless; `vigil-reprove` fail-closes without an active engagement spine;
  `vigil-posture` "exits 0" but writes to a **literal `%h/` dir** because **systemd does NOT expand `%h`
  inside EnvironmentFile CONTENTS** (the seeded `~/.config/vigil/*.env` carry `%h` placeholders) — it created
  `/home/kali/vigil/%h/…` cruft; disabled it + cleaned. backup/backup-push/alerts/ha-mirror need secrets.
  The durability layer needs a proper deployment (system-level units and/or real absolute paths in the .env
  files + a passphrase/vault), which is the `--production` posture, not just `enable`.

Related: [[vigil-live-external-fact]], [[vigil-ui-completeness-program]], [[vigil-strix-hexstrike-wiring-program]].
