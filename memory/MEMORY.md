<!-- One line per memory: title link + short hook. Detail lives in each topic file. gh=thuram-nana. -->

## SIGIL (local-first AI orchestrator @ /home/kali/sigil, repo thuram-nana/sigil)
- [SIGIL hard-prune](sigil-hardprune-program.md) — VAULT-PRUNE deletion; A–D merged, Slice E (crash-safe cutover, 3 owner gates) left.
- [Spine rotation](sigil-spine-rotation.md) #14 · [prod hardening](sigil-production-hardening.md) #12/#13 — merged.
- Phases (all MERGED): [P0 memory loop](sigil-phase0-memory-loop.md) · [P1 KERNEL+WARDEN](sigil-phase1-kernel-warden.md) · [P2 voice](sigil-phase2-voice.md) (ElevenLabs wDsJlOXPqcvIUKdLXjDs, key in sigil.env) · [P3 agent mesh](sigil-phase3-agent-mesh.md) · [P4 ARTIFICER+SCHOLAR](sigil-phase4-artificer-scholar.md) · [P5 perception+BASTION](sigil-phase5-perception-bastion.md) · [P6 governor](sigil-phase6-hardening.md) · [P7 embodiment](sigil-phase7-embodiment.md) · [P8 FRONTIER](sigil-phase8-frontier.md) · [P9 COMPANION](sigil-phase9-companion.md).

## CRUCIBLE (offensive engine @ /home/kali/Pictures/PENTEST-main/framework/v2)
- [anti-hallucination](anti-hallucination.md) — veracity firewall = RE-EXECUTION (#39-47) · [intel engine](intel-engine.md) · [enterprise platform](enterprise-platform.md) · [unified-autonomy](unified-autonomy-program.md) · [speed](speed-program.md) (scan stays SERIAL) · [nervous-system](nervous-system.md) (RL re-rank/defer only; oracle sole authority).
- [beyond-SOTA](crucible-beyond-sota-program.md) · [coverage](coverage-program.md) · [credibility](credibility-program.md) · [Ops Console](ops-console.md) · [testing & gotchas](crucible-testing-and-gotchas.md) (bs4/numpy/z3 absent) · [Prover→Discoverer](prover-to-discoverer-program.md) · [Gap-closure 13-workstream](gap-closure-13-workstream-program.md).

### Soundness lessons (load-bearing)
- [Same-shape + frozen verify](reduction-oracle-sameshape-and-frozen-verify.md) — negative control exercises the ADVERSARIAL shape; verify over the FROZEN signed reduction.
- [ACHIEVED_STATE sampling unsound](achieved-state-sampling-unsound.md) — needs STRUCTURAL grammar + provenance-bound ref.
- [runtime_redrive false-FACT](runtime-redrive-live-false-fact.md) — control must be SAME-SHAPE as the payload.
- [Corpus triple-lock](weaponized-arm-corpus-provenance-triplelock.md) — #894 no corpus from router · trusted channel · LEAD-fallback omits class.
- [Wave-1 gaps](vigil-wave1-remaining-gaps-lessons.md) — framework importable locally ⇒ two-env repros UNFAITHFUL; nuclei substring can't mint a FACT.
- [Timing phase-lock](timing-concurrency-phase-lock.md) — concurrency for SEARCH, serial for PROOF.
- [Allowlist self-reference trap](allowlist-self-reference-trap.md) — never derive an allowlist from the target's own lookup.
- [CLEAN soundness](clean-verdict-soundness.md) — ONE observation of ONE identified peer; never silence-as-negative.
- [Demotion merge hazards](demotion-merge-hazards.md) — a demotion must reach EVERY render path; sibling slices re-arm it.
- [oracle_version class hole](oracle-version-class-hole.md) — test by MUTATING the procedure, in subprocesses.
- [Fake-fixture pattern](fake-fixture-pattern.md) — a fixture must HAVE the property, not simulate its signature.
- [Measured-claim discipline](measured-claim-discipline.md) — commit the harness or withdraw the number.
- [TM-B: attest the CAPTURE](tmb-attest-the-capture-not-the-injection.md) — attest the captured operand via a NON-OFFENSE collector.
- [Egress-floor residual](vigil-engine-egress-floor-residual.md) — CLOSED #874/#879/#919; don't move the denylist to vigil_core.

## AEGIS (defensive dual of CRUCIBLE)
- [Runtime-defense frontier](aegis-runtime-defense-frontier.md) — provable firewall + oracles; merged. · [PCF + FORGE](pcf-forge-program.md) — PCF into the real evidence layer, then FORGE; RAMPART slice 1 #94.

## VIGIL (monorepo @ /home/kali/vigil, repo thuram-nana/vigil-sovereign)
Active / standing:
- [Machine work stays local](machine-work-stays-local.md) — owner: organise everything HERE, no pushing to back up; master plan audit + refs/wip snapshots.
- [Launch scope 11-24](vigil-launch-scope-2026-11-24.md) — ONLY VIGIL; launch = open PRs+six fixes+seats+retention+legal S2–S5/S10–S12+AEGIS+multilang.
- [APEX = VIGIL pentester program](vigil-apex-pentester-program.md) — PRs 1181–1185 merged, default-OFF; NOT the vigil-apex procurement repo.
- [Data retention 11-24](vigil-data-retention-decisions-2026-11-24.md) — evidence 90d auto-delete (to build); billing 10y; audit 3y; web logs 30d; S3 backups 90d; deletion requests 30d.
- [Launch pricing 11-24](vigil-launch-pricing-decisions-2026-11-24.md) — decided packs $5–$100 no bonus; plans retired at launch; trial 450; NO purchase limits/custom amount.
- [Deploy gate](vigil-deploy-gate-owner-approval.md) — owner: install/run VIGIL on the server ONLY when all code is done AND owner approves; today = setup only.
- [Credentials setup 11-23](vigil-credentials-setup-2026-11.md) — owner placing SaaS keys pre-deploy; Anthropic key in SSM done; Google/SES/Stripe owed; no server yet.
- **PAUSED 2026-11-25 05:40 — read the TOP of ~/.claude/plans/RESUME-2026-11-24.md first** (6 runs stopped by owner; run ids + heads).
- [Workflow resume state 11-22](workflow-resume-state-2026-11-22.md) — where every workflow stopped; owner: clear all open PRs in order first.
- [Parallel development](parallel-development-stacking.md) — owner: never idle on merges/approvals/limits; stack dependent slices, keep every gate.
- [Subagent model policy](subagent-model-policy.md) — subagents stay Opus; reviews ALWAYS Opus (pass model); ask before model/agent-file changes.
- [Consumer A-L](vigil-consumer-al-program.md) — A/D/F/J merged; B,C,E,G,H,I,J2,K,L,D-ff,F-ff building (wf_7288f6e7 + wf_b0bfc8e3); plan ~/.claude/plans/consumer-completeness-plan.md.
- [Multilang codescan](vigil-multilang-codescan-program.md) — registry + tree-sitter + 6 FACT languages; plan ~/.claude/plans/multilang-codebase-plan.md; wf_a43a90a6-50b building.
- [Investigation graph](vigil-investigation-graph-program.md) — audit done: 5 slices, no new DB (~/.claude/plans/investigation-graph-gap-matrix.json, PRIVATE); S1+S3 building; NEVER open the ARTEX zip.
- [Legal/Governance](vigil-legal-governance-program.md) — SIGIL SARL; WIRE to existing enforcement; 12 slices (~/.claude/plans/legal-governance-gap-matrix.json); S1 building; 9 owner inputs pending.
- [AEGIS SaaS program](vigil-aegis-saas-program.md) — APPROVED 2026-11-19: E1 full prod via test tenant (supersedes no-VPS for AEGIS only), E2 ONE unified platform on AWS, E3 rent pipes/own intelligence; plan ~/.claude/plans/parsed-popping-lemur.md; ~72 slices/24 waves; AEGIS merges need recorded human approval.
- [operator.key leak](vigil-operator-key-leak-2026-11.md) — stray untrusted key on a public branch; force-pushed clean; 419ffe56 SHA-fetchable until GC.
- [Consumer cascade lessons](vigil-consumer-cascade-lessons.md) — `gh --auto` merges IMMEDIATELY; gitleaks FP → inline allow; serial config.py merges.
- [B+C+D build](vigil-bcd-build-program.md) · [Strix-shell + MCP](vigil-strix-shell-mcp-wave.md) (vetted shell, NOT an open offense portal) · [hosting decision](vigil-strix-shell-hosting-decision.md) (k8s+Firecracker, AWS) · [parallel-3 recon](vigil-parallel3-recon.md) (org==tenant_id) · [Strix teardown](vigil-strix-competitive-teardown.md).
- [Sovereign-fabric vision](vigil-sovereign-fabric-vision.md) — Claude ADVISES; oracle+governance are the authority. · [Wave cadence](vigil-wave-cadence.md) — build→red-pen (NON-NEGOTIABLE)→serial merge. · [Ceremony verify](vigil-ceremony-verify-workflow.md).
- [Authorship](vigil-authorship-contributors.md) — HANDOFF §0.8 trailers; history-rewrite PENDING. · [Claim-discipline: meet-up](claim-discipline-meet-up.md) — BUILD up to the claim, never soften it.

Operational gotchas (load-bearing):
- [/tmp inode exhaustion](tmp-tmpfs-inode-exhaustion.md) — /tmp is a 1M-inode RAM tmpfs; agent tests filled it (ENOSPC ≠ bug); TMPDIR on disk + janitor.
- [VPS](vigil-vps-deployment.md) + [owner flip delegation](vigil-owner-flip-delegation.md) — NO VPS/server work since 2026-11-16; flips only once a deployment exists.
- [Stripe](vigil-payment-provider-stripe.md) (Flutterwave paused; disputes remove credits at OPEN) · [NYX UI name](vigil-nyx-ui-name.md) (display only) · [Onboarding guide](vigil-onboarding-guide.md) #1141.
- [OOB + authority gate](vigil-oob-subsystem-and-authority-gate.md) — pinned collector key = sole FACT determinant; CRUCIBLE_ROOT=/home/kali/Music/PENTEST/crucible.
- [Prod bring-up + UI E2E](vigil-production-bringup-and-ui-e2e.md) #924 · [Run freeze + fact gaps](vigil-run-freeze-and-fact-gaps.md) (ALWAYS `target status` first) · [Live/remote engage](vigil-live-remote-engage-flow.md) · [Live EXTERNAL FACT](vigil-live-external-fact.md).
- [`vigil up` auto-restart](vigil-up-systemd-autorestart.md) · [API-key seal→restart](vigil-apikey-seal-to-engine-restart.md) · [Gateway importorskip](vigil-crucible-core-gateway-importorskip.md) · [Tests honor CRUCIBLE_ROOT](crucible-engine-tests-honor-crucible-root.md).
- [Customer kit-mirror guard](vigil-customer-kit-mirror-guard.md) — kit css/ui.js edits must mirror into customer_app; reds governor-gates+fold-cache and LOOKS like a flake — read the job log first.
- [UI-wiring matrix guard](vigil-ui-wiring-matrix-guard.md) — each route on its own `path ==` line; regen the golden with VIGIL_UIWIRING_REGEN=1.
- [Branch protection](vigil-branch-protection-and-ci-drift.md) (14 checks+strict+signed) · [Main reds](vigil-main-integration-reds-and-restore.md) · [A14 + OracleKind guards](vigil-a14-multigate-lockbump.md) · [CI maint](vigil-ci-maintenance-a14-formal.md).
- [Backticks in commit msg execute](shell-commit-message-backticks.md) — always `-F`. · [Parallel worktree hazard](parallel-session-worktree-hazard.md) — split ownership, never stash. · [LWW replay gap](vigil-lww-governance-replay-gap.md) — HIGH #282.

Programs (historical; merged unless noted):
- [AEGIS multi-gateway](vigil-aegis-multigateway-blocking-offense-toggle.md) #944 · [meridian-s5](vigil-meridian-s5-factgaps.md) #941 · [FACT-coverage](vigil-fact-coverage-program.md) · [GOLD-STANDARD](vigil-gold-standard-program.md) · [TRUTHENOVATION](vigil-truthenovation-program.md) · [Verifiable-Fact](vigil-verifiable-fact-program.md).
- [PCR](vigil-pcr-program.md) (Wave D Go/JVM/JS left) · [DEEP FIX](vigil-deep-fix-engine.md) · [LAP](vigil-live-autopatch-program.md) · [Codescan+fix](vigil-codebase-scan-fix-program.md).
- [Strix FACT-breadth](vigil-strix-fact-breadth-program.md) · [Strix/HexStrike wiring](vigil-strix-hexstrike-wiring-program.md) · [Promotion pipeline](vigil-strix-hexstrike-promotion-pipeline.md) · [HexStrike completion](vigil-hexstrike-completion-program.md) · [HexStrike catalogue](vigil-hexstrike-catalogue-program.md).
- [chat→FACT](vigil-chat-fact-pipeline.md) · [chat e2e](vigil-chat-end-to-end-program.md) (bola/idor reverted) · [chat-orchestrator](vigil-chat-orchestrator-program.md) · [engagement-UX](vigil-engagement-ux-program.md) · [Proof Studio](vigil-proof-studio.md).
- [FULL-UI-CONTROL](vigil-full-ui-control-program.md) · [SOVEREIGN CONSOLE](vigil-sovereign-console-program.md) · [COMMAND UI](vigil-command-ui-program.md) · [UI-completeness](vigil-ui-completeness-program.md) · [Feature-catalog](vigil-ui-terminal-dossier-program.md).
- [MERIDIAN range](vigil-meridian-range-program.md) · [Whole-app roaming](vigil-whole-app-roaming-and-findings-classification.md) · [E-series cloud-exploit](vigil-e-series-cloud-exploit-program.md) (E2-E5 designed).
- [Production-readiness](vigil-production-readiness-program.md) · [Eval-lab](vigil-investor-eval-lab-program.md) · [Live-external](vigil-live-external-program.md) · [MASTER plan](vigil-end-to-end-completion-master-plan.md) · [Government-complete](vigil-government-complete-program.md) · [Claim 5/6](vigil-claim5-6-chat-gaps-program.md) · [Readiness-fixes](vigil-readiness-fixes-program.md) · [Posture](vigil-posture-and-brain-slot.md) · [36-issue burndown](vigil-36-issue-burndown-gotchas.md).
- [Sessions+embodiment](vigil-sessions-embodiment-program.md) · [Production-hardening](vigil-production-hardening-program.md) · [Audit hardening](vigil-audit-hardening-program.md) · [Pro-pentest](vigil-pro-pentest-program.md).
- Fusion lineage: [Unification](vigil-unification-program.md) · [Fusion](vigil-fusion-program.md) · [Redamon+pentagi](vigil-fusion-redamon-pentagi.md) · [VIGIL-LIVE](vigil-live-program.md) · [I2 log + I4 SCITT](vigil-i2-transparency-log.md).
