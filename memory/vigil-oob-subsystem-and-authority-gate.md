---
name: vigil-oob-subsystem-and-authority-gate
description: "VIGIL OOB (out-of-band) is a sound VF-2b signed-collector design; the pinned collector key is the SINGLE determinant of FACT-vs-inconclusive, and the pin only comes from an owner-root0-signed verified engagement authority. Proven end-to-end on the VIGIL side; live PenTrix OOB FACT blocked by owner-key + tripped kill-switch (both features)."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-11T13:50:55.065Z
---

Answered "is OOB working perfectly?" by end-to-end test (loopback PenTrix lab). Verdict: **the OOB engine is sound and fully functional; a live FACT is gated by intended owner-governance, not a defect.** See [[vigil-strix-hexstrike-promotion-pipeline]], [[vigil-engine-egress-floor-residual]], [[anti-hallucination]].

**Architecture (VF-2b signed-collector):** `collaborator serve --host --port --secret --collector-keypair-file` runs the HTTP OOB relay (also `serve-dns`); it mints/loads a collector keypair and PRINTS only the public key (python stdout is block-buffered to a non-TTY so the banner may not flush — pass `--collector-keypair-file` + `--secret` so you hold both). The collector SIGNS each receipt as it observes the hit; `verify/oob.py verify_oob_receipt(hit, collector_pubkey=PINNED)` is FAIL-CLOSED — True only if the receipt's Ed25519 sig verifies against the PINNED pubkey. Proven in isolation: callback recorded (token/method/path/query/client_ip/UA/host), signed (collector_sig), verifies True with the right pin; wrong pubkey→False, tampered token→False, wrong poll secret→HTTP 403.

**The pin is the SINGLE determinant of FACT-vs-INCONCLUSIVE** (proved by running `oob_callback_oracle` over a real signed hit):
- WITH pin → `fired=True, confidence=0.95, kind=OOB_CALLBACK, oob_verdict=VERIFIED, token_verified=True, receipt_verified=True` → the firewall renders grounding="fact", confirmed_by=OOB_CALLBACK (e.g. command_injection).
- WITHOUT pin → `fired=False, oob_verdict=UNVERIFIABLE_RECEIPT` ("VF-2b receipt present but no pinned HTTP collector key — fail-closed, NOT a silent token-only drop") → the INCONCLUSIVE-by-design outcome.

**Where the pin comes from — owner-root0-gated:** `oob_collector_pubkey` lives INSIDE the owner-signed canonical authority bytes (`vigil_core.authority.authority_signing_bytes`), so it can't be edited without RE-SIGNING. `engage._engage_oob_authority(slug)` threads the pin ONLY if `load_verified_authority(slug)` succeeds against the authority-root. **Bare `scan` NEVER wires a pin** (WebScanCampaign omits oob_collector_pubkey) → blind classes (ssrf/cmdi/blind_xxe/oob_sqli) correctly stay INCONCLUSIVE. There is a keyless helper `vigil_integration.live.wiring.provision_authority(...)` but it is BOOTSTRAP-ONCE: it only writes a fresh authority-root if NONE exists. On this deploy an authority-root already exists (bootstrapped 2026-09-20) pinning owner key **root0 = `MgZZY4QtFhsZKCMh3jx0P78AgnNdbuss0kYv5P7amI4=`**, so a fresh-key-signed authority fails `verify_authority` ("0 valid distinct signature(s) < threshold 1") and `_engage_oob_authority` returns `{}`.

**Two gates to a live OOB FACT on PenTrix (both FEATURES, not defects):** (1) a `loopback` engagement authority carrying `oob_collector_pubkey`=collaborator pubkey, SIGNED by the owner `root0` key (offense venv is keyless by FATAL-2 → cannot sign, must not forge); (2) the `loopback` **kill-switch is TRIPPED** ("UI panic", since 2026-09-28) — a hard stop on engage regardless. Path to a real OOB FACT: owner root0-signs the authority with the pin → clear the kill-switch → `engage loopback "http://127.0.0.1:3000/cmdi/ping?host=127.0.0.1" --oob-relay-url http://127.0.0.1:<port> --oob-relay-secret-file <f>`. Egress floor does NOT touch loopback OOB (the loopback charter scopes 127.0.0.0/8; the callback is target-side egress VIGIL doesn't gate).

**Env gotcha (load-bearing):** runtime state resolves under **`CRUCIBLE_ROOT=/home/kali/Music/PENTEST/crucible`** (set in the shell profile), a DIFFERENT tree from the repo's `/home/kali/vigil/engine/crucible` (not a symlink). All `.authority/`, `.authority-root/`, `.entitlement/`, charters, kill-switch live under CRUCIBLE_ROOT — reason about runtime state THERE, not the repo. [[vigil-production-bringup-and-ui-e2e]]

**State-mutation caveat:** a delegated subagent ran `provision_authority` during the probe and OVERWROTE the runtime `.authority/loopback.authority.json` (original bytes gitignored, unrecoverable); it restored a functional equivalent. Verified current state: file 0600, `oob_collector_pubkey=None`, `_engage_oob_authority('loopback')→{}` (no pin; pre-test-equivalent). LESSON: an agent told to "find out whether X is possible keylessly" may RUN a state-mutating helper to find out — scope such probes read-only or sandbox the base_dir explicitly.
