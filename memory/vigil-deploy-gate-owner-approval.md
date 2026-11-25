---
name: vigil-deploy-gate-owner-approval
description: Owner rule (2026-11-23) — VIGIL is only DEPLOYED (installed/run on the server) once all code work is done AND the owner explicitly approves; until then only infrastructure setup
metadata:
  type: feedback
---

Owner, 2026-11-23: "we will only deploy vigil when all is done both in code and I approve, so today we are just setting up."

**Why:** the owner wants the code finished (PR-H, terms, webhook edge, the paused programs) and a deliberate go-live decision before any VIGIL software runs on the production server.

**How to apply:**
- Allowed now: AWS/Cloudflare/Stripe/Google/SES *setup* (accounts, keys created by the owner, provisioning, empty EC2 host, DNS planning).
- NOT allowed until explicit owner approval: cloning/bootstrapping VIGIL on the EC2 host, writing env files there, starting units, pointing DNS at the server, turning on payments.
- Ask before changing infra state (e.g. stopping/starting the instance). Related: [[vigil-credentials-setup-2026-11]], [[vigil-owner-flip-delegation]].
