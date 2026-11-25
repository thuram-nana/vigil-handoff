---
name: vigil-meridian-s5-factgaps
description: "VIGIL fact-gap expansion on branch meridian-s5-expansion (local @ /home/kali/vigil): 9 sound governed-engage FACT classes, all adversarially red-penned and merged to the integration branch — NOT yet on main or the VPS."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-20T13:21:37.268Z
---

Snapshot Oct 2026 (session 7758e121). Local checkout `/home/kali/vigil`, branch **meridian-s5-expansion** (tip `f0a431ad`, FF'd from `592b9196`). The "don't demote — build every FACT gap sound" program. All classes are governed-engage FACT rails (engine `_redrive_spec` → wiring `_live_*_redrive_fact`), each with a sound oracle + a DECISIVE same-shape negative control, each adversarially red-penned (parallel worktree lenses, non-negotiable), `make gate` holds **(TP,FP,FN)=(11,0,0), requests_sent=1860, findings_reported=11**.

Classes landed this program:
- Earlier wave (committed before the s5-factslices merge): **cmdi (timing dose-scaled `;sleep N`, NOT computed-product — the calculator fakes it), el_injection (evaluation `${N*M}`), csp_posture, dom_xss, prototype_pollution**, + API-key hot-reload (`592b9196`, see [[vigil-apikey-seal-to-engine-restart]]).
- s5-factslices merge (`3c5df3fa`..`f0a431ad`): **access_control (idor/bola/bfla/mass_assignment)** triple-lock fail-closed (discriminator residual = compromised-deployment-only, named); **workflow/reset (request_race/business_logic + password_reset_reuse/collision/cross_user)** owner-signed-spec gated; **graphql_batching** — a red-pen found a BENIGN-origin false FACT (see [[reduction-oracle-sameshape-and-frozen-verify]]), REBUILT sound (distinct per-op aliases `{ v{i}: __typename }` + `eq[count,sent_batch]`) + re-red-penned HOLD_SOUND + capability-matrix prose refreshed; **sensitive_exposure** — wired LEAD-ONLY (`fact_capable:false`, provably cannot mint even when its oracle fires).

**Deployment gap (load-bearing):** these 9 classes are on meridian-s5-expansion ONLY. They are NOT on `main` and NOT on the Contabo VPS (see [[vigil-vps-deployment]]) — the VPS carries only the API-key hot-reload overlay. Pushing meridian-s5-expansion → origin / PR → main is an outward-facing step that was NOT yet done (awaiting owner go-ahead). The cmdi demotion re-baselined the gate pin 1899→1860. Two non-blocking follow-ons named by red-pen: workflow request_race predicate should bind a per-response-unique commit token; graphql_batching could take a per-probe high-entropy alias to retire its deceptive-target residual.
