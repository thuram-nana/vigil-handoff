---
name: subagent-model-policy
description: "Owner's 2026-11-19 /usage decision on subagent models — keep Opus everywhere (no CLAUDE_CODE_SUBAGENT_MODEL default), reviews must stay Opus, graphify left unpinned; with the usage data behind it"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T14:04:15.559Z
---

**Decision (owner, 2026-11-19, after a /usage review flagged general-purpose subagents at 29% of usage):** no default subagent model (`CLAUDE_CODE_SUBAGENT_MODEL` left unset in `~/.claude/settings.json`), adversarial reviews stay on **Opus**, and the graphify skill's extraction agents are left unpinned (so they also inherit Opus). No settings, agent or skill file was changed.

**Why:** quality over cost. Reviews are VIGIL's soundness gate; that session they caught defects the builders' self-review missed (a LEAD that could open tracker issues, a trial-paywall bypass, a stateless fair queue).

**How to apply:** pass `model: "opus"` explicitly on every adversarial red-pen / soundness review you spawn (Agent tool or workflow agent()), so a future cheaper default can never silently downgrade them. Do not set a subagent model default or edit agent/skill model fields without asking — the owner wants to see each such change first. If cost comes up again, the remaining levers are: a `CLAUDE_CODE_SUBAGENT_MODEL` default (verified present in Claude Code 2.1.227), pinning graphify's extraction dispatch (SKILL.md lines 241-247), or spawning fewer agents per item.

**Data (that session's subagent transcripts):** 99.7% of general-purpose usage was the orchestrator's own builds/reviews/audits (graphify extraction 0.3%); by role ≈ 49% reviews, 26% build/fix, 16% audit/explore/triage, 10% misc — all inherited Opus. `general-purpose`, `Explore`, `Plan` are built-ins with no override file; custom agents in PENTEST-main/.claude/agents pin their own models (red-pen/oracle-smith/... opus; chronicler/report-wright/sensor-wright sonnet). Related: [[vigil-wave-cadence]], [[vigil-consumer-cascade-lessons]].
