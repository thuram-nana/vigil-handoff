---
name: vigil-owner-flip-delegation
description: Owner 2026-11-16 — NO VPS/server work at all (focus = the customer app); three production flips delegated to the agent only once a deployment exists and each gate is green (prod-exec after Gate 5; metered after W2+W3+M1c-2+wallet E2E; multi-shard after S4)
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-16T13:18:34.398Z
---

**Scope (owner, 2026-11-16, verbatim intent):** "we will no longer be dealing with vps at all — we are focused on our
customer app for now." So: never SSH to, deploy to, or change any server (the old Contabo VPS of
[[vigil-vps-deployment]] is out of scope). The work is the customer app and its code (the HANDOFF-SESSION.md §5
program), built and tested locally + in CI, default-OFF.

**Flip delegation (same day, pick-one answers)** — recorded for when the customer app gets a deployment; none is
done now because no server is in scope. The agent may then flip each one itself, only when its gate is green, and
report it:
- `VIGIL_CUSTOMER_PROD_EXECUTION` — only **after Gate 5 passes** (hands-on wallet E2E with sandbox payments, with
  evidence; handoff §13.2).
- `VIGIL_METERED_BILLING` (+ `VIGIL_MODEL_GATEWAY_ENABLED`, which it requires) — only **after W2 + W3 + M1c-2 are
  merged and the simulated wallet E2E (§11.12) is green**, and only once the OWNER has set
  `VIGIL_BILLING_TARGET_MARGIN` (private value, never in the repo) and the Anthropic key on that deployment;
  `vigil doctor` must show no hard issue.
- The multi-shard production flag — only **after Front C S4 is green** (the flag arrives with S2–S4).

**Why:** owner instructions in this session. **How to apply:** no server actions at all until the owner says a
deployment is back in scope; never flip early. Everything else in handoff §6 (secrets, accounts, the margin value,
fold cache in prod, CodeQL dismissals, any unrecorded money number) stays the owner's. Related:
[[vigil-strix-shell-hosting-decision]], [[vigil-wave-cadence]].

**SUPERSEDED FOR THE AEGIS PROGRAM (owner decision E1, 2026-11-19):** SIGIL-operated infrastructure (hosted engines,
TLS edge, per-tenant gateways, staging test tenant on AWS) is IN scope for the AEGIS SaaS program only, with production
go-live at explicit owner gates. Everything else in this directive still stands; the legacy Contabo VPS stays out.
See [[vigil-aegis-saas-program]].

**SUPERSEDED FOR THE VIGIL CUSTOMER-APP DEPLOYMENT (owner, 2026-11-23, 'option B'):** Claude may run the non-secret deploy steps on AWS
(eu-west-1) from this machine — provision.sh, EC2 launch, bootstrap, units, nginx, vigil doctor — with the owner approving each
command; the owner types every secret. Production flips above still follow their gates. See [[vigil-credentials-setup-2026-11]].
