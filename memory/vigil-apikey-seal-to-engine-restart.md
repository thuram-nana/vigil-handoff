---
name: vigil-apikey-seal-to-engine-restart
description: "Re-sealing the Anthropic key in the VIGIL UI now HOT-RELOADS into the running offense engine (plaintext sigil.env backend) — no restart; a keyring/TPM-vault secret still needs a Start-offense. The old restart-required behavior below now applies only to the sealed backends."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-20T11:30:51.870Z
---

**RESOLVED for the plaintext `~/.sigil/sigil.env` backend (2026-10-20) — API-KEY HOT-RELOAD.**
Branch `apikey-hotreload` (commits 6e64171e + 592b9196; PR pushed, open it at
github.com/thuram-nana/vigil-sovereign pull/new/apikey-hotreload — `gh pr create` was classifier-denied; NOT
yet merged to main) and **DEPLOYED LIVE to the Contabo VPS** (respawned `vigil up` carries
`VIGIL_LIVE_KEY_RELOAD=1` in its offense children). The running offense engine now RE-READS the current
persisted key at think time, so a UI re-seal is picked up on the NEXT LLM call with NO Stop→Start. Mechanism:
`framework.v2.common.live_keys.read_persisted_anthropic_key()` (the one sigil.env reader; mirrors
sigil.config._load_env_file parse + SIGIL_ANTHROPIC_API_KEY>ANTHROPIC_API_KEY) + `refresh_live_anthropic_key()`
(flag-gated in-place os.environ update, returns changed-ness so a cached SDK client rebuilds); wired at
`live.think_claude._resolve_key` (engage — delegates to the shared reader), `console.chat` (the key checks),
and `kernel.backends.anthropic._client_obj` (rebuilds its cached client on change). GATED by
`VIGIL_LIVE_KEY_RELOAD=1`, set ONLY by `uiproxy._child_env` for managed children — a test or bare-shell
`vigil engage` keeps plain-env behavior and never reads a stray ~/.sigil key; a shell-only operator (no
persisted key) is unaffected. HONEST RESIDUAL: a keyring/TPM-vault secret is NOT re-read (the offense can't
unseal the sovereign vault) → those backends STILL need a Start-offense, so the detailed restart guidance
below remains correct for THEM. Deploying the feature itself took one restart (to load the code); after that,
key changes never need one.

Sealing an offense-delivered secret (e.g. ANTHROPIC_API_KEY) via the UI **API Keys** screen
(`set_secret` → `SecretStore.set`, apps/sigil/sigil/platform/secrets.py:88/94/97) writes
keyring→TPM-sealed→0600 `~/.sigil/sigil.env` AND sets the **sovereign** process `os.environ`
live — so the **Test** button (live GET `api.anthropic.com/v1/models`, secret_probes.py:74) flips
green immediately, and a bad key returns a truthful HTTP 401. BUT the **offense engine** (crucible
console :8787 / gated-api :8799) reads the key per-call via `os.environ.get("ANTHROPIC_API_KEY")`
from a process env **frozen at Popen spawn** (uiproxy.py:2332). The key is unsealed into the
children ONCE, at `vigil up`, via `sigil settings export-runtime-env --include-secrets`
(uiproxy.py:3123/2478; ANTHROPIC_API_KEY is on `_OFFENSE_ENV_ALLOWLIST` uiproxy.py:2393 and is an
`_OFFENSE_DELIVERED_SECRETS` member, settings.py:322). So **re-sealing while the engine is running
does NOT reach it — sovereign green ≠ engine has the key.**

To propagate a freshly-sealed key to the engine, RESTART the offense plane:
- **In-UI (smoothest; no re-sign-in, token unchanged):** offense chip / System & Services →
  **Stop offense → Start offense** → `PlaneControl.start_offense` → `_reresolve_offense_env`
  re-unseals + re-spawns (uiproxy.py:1051/3166). TRAP: a bare **Start while already "up" is a
  no-op** (`already_running`, uiproxy.py:1041) — must Stop first.
- **Full:** `vigil down` + `vigil up` (foreground = Ctrl-C then `vigil up`; systemd =
  `systemctl --user restart vigil-command`, because Restart=always reverts a bare Ctrl-C in ~5s).

Traps that look green-but-fail: (1) `_resolve_offense_llm_env` returns `{}` on ANY sovereign error
→ children re-spawn **keyless**, engagements 401 with only a "runs keyless" note (uiproxy.py:2519).
(2) A key only in a shell `export` (never sealed to keyring/TPM/sigil.env, absent from
`.vigil-live/service.env`) works under a hand-run `vigil up` but is ABSENT under systemd — always
SEAL it. (3) Token posture: dev PERSISTS the bootstrap token (same `?token=` URL across restarts);
production mints an EPHEMERAL token each start (re-open the new banner URL). (4) `settings_status`
shows a freshly-set key as `unchecked` until a live probe runs — green "set" ≠ authenticates.
(5) Use a STANDARD key (`sk-ant-api03-…` from a billed workspace), not an Admin key
(`sk-ant-admin…`) — the Admin API key 401s against the Messages API.

**Why:** a Linux process env is immutable after fork; the offense plane is keyless-by-design and
receives secrets injected only at spawn, so a mid-session re-seal is invisible to the already-
running engine.
**How to apply:** after re-sealing any model/integration key, restart the offense plane (Stop→Start,
or `vigil up`) and VERIFY via a real engine call (a Chat message or a small engagement — Chat reads
the key at chat.py:2075), never just the green Test. See [[vigil-up-systemd-autorestart]].
