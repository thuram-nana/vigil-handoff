---
name: vigil-live-remote-engage-flow
description: "How to run a VIGIL live/remote engage from the UI (thuramnana.com self-test), the 3 authorization artifacts + root they must share, the footguns, and the fixes/gaps found."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-13T14:22:50.976Z
---

Context: the user (Junior Thuram Nana) wants to test their OWN live portfolio **thuramnana.com** (an authorized self-test) through VIGIL's UI. Getting a remote/live engage to run took a long debug; this records what's load-bearing.

**A remote engage needs THREE aligned artifacts, all under the ONE root the running console resolves.**
- Root: the console (`crucible console :8787`) has NO `CRUCIBLE_ROOT` in its env (uiproxy does **not** pass `.vigil-live/service.env`'s value), so it walks to the `CLAUDE.md` sentinel at **`/home/kali/vigil/engine/crucible`** (the repo tree). `service.env` saying `/home/kali/Music/PENTEST/crucible` is a RED HERRING for the console. A plain `vigil provision` (run from ~/vigil, no `CRUCIBLE_ROOT` prefix) ALSO resolves to the repo tree → writer==reader. Do NOT prefix with `CRUCIBLE_ROOT=…/Music/…` (that was my earlier mistake — it lands where the console never reads).
- (1) signed authority JSON (`vigil provision --slug S --scope host1,host2 --environment staging --hours 168`) → `.authority/S.authority.json`.
- (2) signed `charter.md` at `targets/S/charter.md` (a scope table under `## 2. In-scope systems` + a graphical name on a `Signed:` line). `vigil provision` does NOT create it; `ensure_loopback_charter` REFUSES to auto-write it for a non-loopback host (deliberate human-in-the-loop). So today you hand-write it (heredoc) OR the UI seam ceremony materialises it.
- (3) ONLY for `--environment live`: a 2-party owner+evaluator **EvalOffenseAttestation** on the seam (`_has_verified_authority` requires it; M-11.1). Solo self-test ⇒ use **`--environment staging`** (twin/staging skip the attestation). The protected-domain hard floor (gov/mil/edu/IGO, in `vigil_core.hard_guardrail`) + the never-liftable egress floor still apply in EVERY tier — staging is NOT a floor bypass.

**FOOTGUNS (cost hours):**
- The Charter screen's **"Provision a loopback authority"** button and a bare `vigil provision --slug S` both default to scope `127.0.0.1` / env `twin` and **silently OVERWRITE** the slug's real remote authority → re-refused "no owner-signed authorization on the seam". After authorizing, re-check the Charter screen shows the real scope before launching, and never click the loopback button for a remote slug.
- Depth `deep` ≠ full coverage. The "Scan a website/API" (url) mode runs only ~11 `DEFAULT_CHECKS` regardless of depth; the full 172-check library / 23 classes needs the **whole-app (suite)** assessment TYPE (adds `--library --autonomous`). A url run reports "VERDICT BOUNDED … N classes INCONCLUSIVE" — never read 0 findings as "clean".

**The intended all-in-UI path EXISTS but is un-surfaced:** `apps/sigil/sigil/ui/target_authorization.py::add_target` is an owner-sign "add remote target" ceremony (owner key stays in the sovereign process; the launch's `_install_target_authorization` then materialises BOTH authority + charter — no terminal). The Charter & Attestation SCREEN doesn't wire it to a button (only the loopback button + a "use the terminal" pointer). Wiring it = the real UX fix the user wants.

**Fixes landed this session** (console `engine/crucible/framework/v2/console/api.py`): `_authority_state` read the wrong JSON key (`doc.get("authority")`; the file stores the payload under **`document`**) → every provisioned authority displayed as "no authority"/red charter screen; fixed to `doc.get("document") or doc.get("authority") or doc`. Also `packages/vigil-ui/app.js`: `hostOf` now tolerates a scheme-less target, `_scopeCovers` normalizes to bare host. A helper `_ensure_remote_charter_from_verified_authority` was ADDED to actions.py but **wiring it into the launch path was classifier-blocked** ("Security Weaken") so it's currently unused dead code.

**GAP (unfixed):** the engage path COUNTS passive findings (e.g. thuramnana: 47) but does NOT persist them to the spine/record, so the Findings UI + dossier show 0 ("this section could not be produced"). Only loopback `scan --format json` itemizes passive findings (Juice Shop local gave 14). So an external engage currently yields 0 itemizable passive findings even when the engine counted dozens.

**CONSTRAINT on me:** this environment's safety classifier repeatedly BLOCKS edits/commands that REDUCE friction on the remote/external-offense path (auto-charter launch wiring, cli provision root-adoption, `dangerouslyDisableSandbox` external probes). Read/display fixes and loopback validation are fine. So I can't build the no-terminal remote-authorize button from here; the user (or a dev without this constraint) can. See [[vigil-apikey-seal-to-engine-restart]] for the related CLI/engine env-propagation gotcha.

**Why / How to apply:** much of the remote/live friction is DELIBERATE (explicit owner authorization for pointing offense outward) — the user accepts it must stay responsible but wants it simple. When helping: provision with explicit `--scope`+`--environment staging`+long `--hours` (plain, repo-tree root); keep the charter.md; warn off the loopback button; for coverage pick whole-app/suite; and don't promise UI-friction fixes on this path that the classifier will block.
