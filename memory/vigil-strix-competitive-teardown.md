---
name: vigil-strix-competitive-teardown
description: "Strix.ai competitive teardown + VIGIL gap analysis, a Claude Docs deliverable (claude.ai/artifact/Bd4woFtqyp2R3Vdub9dseY). What Strix has that VIGIL lacks (multitenancy, dual-MCP, connect-repo + PR-bot, posture dashboard, business model) and a prioritized build roadmap; what VIGIL keeps as differentiators. Load for VIGIL product-direction / what-to-build-next / commercialization / multitenancy / MCP-surface / signup-identity work."
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-24T22:16:35.578Z
---

Deep competitive teardown of **strix.ai** (vendor OmniSecure, Inc.; OSS `usestrix/strix` Apache-2.0 ~71K stars, v1.7.0; hosted app.strix.ai) vs VIGIL, done via a 6-lane parallel research workflow (product/screens · pricing · integrations-MCP-API · auth-multitenancy · OSS-agent · UX+reviews). Deliverable = a **private Claude Docs doc at https://claude.ai/artifact/Bd4woFtqyp2R3Vdub9dseY** (12 sections: exec summary, verdict scorecard, 13-screen map, business model, integrations/MCP/API, auth+multitenancy, OSS engine, UX, reviewer verdict, gap analysis, roadmap, where-VIGIL-leads). Private until shared from the page Share menu. Raw research digest cached at scratchpad `strix-digest.md` (210KB) for this session.

**One-line strategy: "Strix's shell on VIGIL's spine."** None of Strix's advantages touch the engine — they are the product shell, which is the buildable part.

**Strix soundness discipline = our doctrine by another name:** their OSS `counterevidence` skill forces every candidate to close as **confirmed / ruled_out / open_proof_gap** (name a control at file:line that runs before the sink on every reachable path, or record an honest gap) — this is VIGIL's FACT / refuted / LEAD trichotomy. Validation is at parity; VIGIL's EDGE is that our FACTs are Ed25519-signed + offline-re-verifiable (their PoCs are convincing but unsigned).

**What VIGIL LACKS (prioritized gaps, all shell not engine):**
- **P0 identity/tenancy:** no org=workspace tenant primitive; no self-serve signup + email verification + Google/GitHub social (Strix rents **WorkOS AuthKit**); no anti-mixup session pinning (their `X-Strix-Workspace` header + hard **HTTP 409** on concurrent cross-org switch is the cheap strong control to copy); `common/tenant.assert_tenant_fleet_isolated()` is built-but-unwired.
- **P0/P1 MCP+integrations:** CRUCIBLE doesn't expose itself as an MCP server ("every REST op → one role-gated MCP tool, operationId→snake_case, readOnly/destructive hints, core/full profiles"); no "consume external MCP mid-engagement" evidence connector (read logs to CONFIRM an exploit — pure prove-don't-guess, the most on-brand thing to copy); no 1-click repo connect (GitHub App / GitLab-Bitbucket PAT); no `@mention` PR-review bot; no signed webhooks (HMAC over `timestamp.rawBody`, 5-min replay window) or Jira/Linear/ServiceNow sync.
- **P1 product surface:** no posture dashboard (KEV/EPSS exploitability, MTTR, CWE clustering, security score, stock-vs-flow); no Asset Discovery; no Knowledge base (Documents/Policies/per-repo Profiles fed into every run); no Test Users + managed inbox (TOTP/email-OTP/magic-link) for authenticated/BOLA testing.
- **P1-P2 GTM (only if commercial):** Strix = open-core, **$29/seat/mo + $1/credit pre-disclosed-before-launch**, Rightsized $100-300 / Full Audit $2000+; gate on deployment+compliance NOT capability; their #1 user complaint is unpredictable cost ($17 to find nothing) — exactly what VIGIL's oracle-gating avoids.

**What VIGIL KEEPS as differentiators (do NOT regress):** cryptographic offline-re-verifiable FACTs; sovereignty-by-default (local-first, owner keys, never-liftable egress floor) vs Strix's cloud-SaaS-with-VPC-option; the AEGIS defensive dual (Strix is offense-only); m-of-n destruction quorum + signed charters + fail-closed A0-A3 + FATAL-2 two-env.

**3 quick wins to copy verbatim:** (a) webhook signing scheme; (b) the three-state closure as an explicit agent skill; (c) the `X-Strix-Workspace` + 409 anti-mixup guard.

Research gotcha: the 6-lane workflow's 2 richest lanes (product-features, oss-agent) hit the StructuredOutput schema-retry cap but their research was fully RECOVERABLE from the subagent transcripts' rejected tool-call inputs (biggest `summary`-only attempt + `worth_copying`/`open_questions`) — don't re-run a schema-failed research lane before checking its transcript. Links: [[vigil-aegis-multigateway-blocking-offense-toggle]], [[vigil-sovereign-fabric-vision]], [[prover-to-discoverer-program]].
