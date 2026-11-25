---
name: vigil-deep-fix-engine
description: "VIGIL Claude-Code-class DEEP FIX engine (#780 merged) + console Strix toggle (#781) + gateway/strix live bring-up fixes (#782/#783) + the live-run gotchas"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-09-08T00:34:51.309Z
---

The Claude-Code-class DEEP FIX engine for VIGIL (repo thuram-nana/vigil-sovereign; canonical /home/kali/vigil-dev, demo /home/kali/vigil). Continues [[vigil-codebase-scan-fix-program]] and [[vigil-live-autopatch-program]]. Plan = parsed-popping-lemur.md.

**The engine (#780 MERGED @ cc3a4923):** repo-aware, multi-file, iterate-until-green, ORACLE-verified code fixing on the gated disposable-clone ladder. Phases A (multi-file `_gather_repo_context`, symlink-escape-guarded) · B (real build/test via `run_sandboxed`, `buildsys.detect_build_plan`) · C (bounded iterate w/ failure-feedback) · D (pre-PR oracle re-verify → status `verified-no-pr`, remediated=False) · E.1 (`--proposed-diff`) · E.2 (agentic Strix front-end `strix_fix.py`: UNTRUSTED diff → same ladder; empty/error → `no-agent-diff`, NEVER falls back to the inline coder). **Invariants intact:** oracle is SOLE certifier (tests are a GATE), `remediated` type-lock (signed evidence_ref + opened PR) untouched, disposable clone only.

**This session's PRs (all CI green/running, billing fixed):**
- **#781** — console **`--agent strix` toggle**: app.js "Agentic (Strix)" checkbox → deepFix POSTs `{agent}` in the body; server `/deepfix` reads it; `actions.deep_fix(agent=…)` appends `--agent strix` to the SAME gated argv (never --open-pr); bogus value fails safe to "none". **CI 25/25 green.**
- **#782** — **make the gated egress gateway actually come up on ANY host**. THREE bugs, none ever caught (CI only string-matches the compose, never does a real `docker compose up`): (a) sandbox subnet hardcoded → `render_compose` now emits `${VIGIL_GATEWAY_SANDBOX_SUBNET:-…}`/`${VIGIL_GATEWAY_GATEWAY_IP:-…}` (byte-identical unset); (b) **firewall/proxy `command:` repeated the image ENTRYPOINT token** (`["vigil-gateway","apply-firewall"]` with `ENTRYPOINT ["vigil-gateway"]` → `vigil-gateway vigil-gateway apply-firewall` → argparse rejects → sidecar exit 2 → whole gateway fails; a #430 image-hardening regression) → fixed to `["apply-firewall"]`/`["serve-proxy",…]`; (c) `strix_sandbox.preflight` used a BARE `SandboxNetworking()` → added `SandboxNetworking.from_env()` so a relocated gateway's proxy coords reach the sandbox. Committed compose is REGENERATED from `render_compose()` (the `committed==rendered` test is the drift guard); the committed-artifact env parsers (test_config + egress/release batteries) resolve `${VAR:-default}`.
- **#783** — **`resolve_strix_bin()` rc-127 fix**: it did `Path(sys.executable).resolve().parent/"strix"`, but a venv `bin/python` is a SYMLINK to `/usr/bin/python3.13`; `.resolve()` jumps to /usr/bin (no strix) → falls to `which` → bare `"strix"` → spawned with cwd=tempdir → FileNotFoundError → **rc 127, no diff**. Fixed: check the UNRESOLVED venv-bin parent first.

**LIVE-RUN PROOF (2026-09-07):**
- **Gateway comes up** via `vigil services up --compose <vigil-dev compose> --no-build` with `VIGIL_GATEWAY_SANDBOX_SUBNET=10.88.7.0/24 VIGIL_GATEWAY_GATEWAY_IP=10.88.7.2 VIGIL_GATEWAY_SCOPE_HOSTS=127.0.0.1` → gateway HEALTHY on 10.88.7.2, qrthuram untouched.
- **Inline deep-fix VERIFIED live**: `vigil patch --deep` on the codescan-demo weak-hash finding (CWE-327) → `status: verified-no-pr`, applied_paths `['svc/config.py']`, diff `hashlib.md5`→`hashlib.sha256`, gate=`python3 -m compileall -q .` passed (test suite honestly skipped: offline sandbox no deps), verify=DAA-rule-cleared-on-clone; remediated=False, opened_pr=False (type-lock held).
- **Agentic Strix path**: launches now (127 fixed), reasons with `anthropic/claude-opus-5`, but the VENDORED strix sandbox dies at file-copy with **exit 137** (a `mkdir` exec SIGKILLed during target copy; host had 21G free so NOT host OOM — a vendored-strix sandbox setup issue on the `--internal` net). **Fail-closed contract HELD at EVERY failure** → `no-agent-diff` / remediated=False / no inline fallback. (Positive agentic run still needs the vendored-strix sandbox-setup 137 resolved — third-party debugging.)

**GOTCHAS**
- **`vigil_egress` has no pinned subnet** → Docker auto-assigns it a 172.x /16 (default pool 172.16.0.0/12), which can enclose a chosen 172.x sandbox /24 → "Pool overlaps". **Relocate the sandbox to a 10.x /24** (outside the auto-pool). Noted in the compose.
- **`vigil services up` reads the STATIC `infra/docker/docker-compose.yml`** (env-substituted via `compose_up`'s `{**os.environ}`), NOT `render_compose` at runtime — but `render_compose` is the source of truth the committed file is regenerated from.
- Demo tree uses **editable installs** (imports from /home/kali/vigil/{gateway,integration}); copy changed .py in for a live demo (properly landed via PR). Strix reasons HOST-side (direct model egress); only the sandbox's tool traffic is gateway-gated. Valid model key in `~/.sigil/sigil.env` (the running one is dead). Strix images already built: `vigil-gateway:latest`, `vigil/strix-sandbox:local` (7.5G).
- Deep concurrent test-run gotcha: the ambient env's `vigil_core` editable install points at a STALE `/home/kali/vg36/484/...` worktree (missing `strip_url_credentials`) → doctor tests fail only in a batch; run in isolation → pass. Not a code bug; CI uses the pinned vigil_core.
