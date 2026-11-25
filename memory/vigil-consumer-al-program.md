---
name: vigil-consumer-al-program
description: "VIGIL consumer-completeness program (11 areas -> 12 gated PRs A-L) — plan location, PR map, merge state as of 2026-11-19, and the A-J completion workflow"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T08:28:06.776Z
---

**Plan (durable copy):** `/home/kali/.claude/plans/consumer-completeness-plan.md` (891 lines; area sections + "Overall sequencing" L775 + owner-gated L827 + 100%-checklist L855). The original lived in the session scratchpad and was WIPED; recovered from agent transcripts on 2026-11-19.

**PR map (area):** A presentation spine/keystone (1/6/8/11) · B fix-first+posture UI (1) · C connect+ownership (2) · D notifications (3) · E CI-native+deploy-retest (4, needs APEX PR-3) · F fair queue+shard (5) · G billing/report/compliance (6) · H SSO/SCIM (7) · I proof badge+PCF (8) · J NL assistant (9) · K fix-it (10, needs PR-3+I) · L attack-path UI (11). B/G/I/L depend on A.

**State 2026-11-19:** merged D #1187, F #1188, J #1189 (J landed UNVERIFIED via `gh pr merge --auto`, see [[vigil-consumer-cascade-lessons]]). Open: A #1192 (independent red-pen PASS), C #1186, H #1190 (logic red-pen PASS; HTTP edge was left as a seam though Area 7 scopes it in), APEX PR-3 #1183, main gitleaks-FP fix #1193. In-scope gaps found: H's SAML/SCIM routes, J's wallet draw (Area 9 says billed against the org wallet).

**Update 2026-11-19 ~06:30 EST:** APEX PR-3 #1183 MERGED (APEX wave complete). Main fix #1193 merged; #1194 (gateway-test race) open. A #1192 re-updated + `standards_mirror.json` regenerated for blind_ssti (`tools/gen_standards_mirror.py`, offense venv) — any engine-vocab change needs that regen. Post-merge red-pens: **D, F, J all BLOCK** (D: FACT-only issue gate is a presence check — a LEAD opens a tracker issue; J: trial-paywall bypass + assistant spend never billed; F: shard-on-signup never wired, fair queue stateless) — all dormant behind default-OFF flags; fix-forward branches claude/consumer-pr{D,F,J}-fixfwd. Workflow hit the 5-h limit after ~46 min; builder partial work backed up at /home/kali/vigil-builder-backups/20261119-0555 and RESUMED.

**A-J completion workflow `wf_01a9b360-fe3`** (launched 2026-11-19 at user request "fully implement A to J"): builds B, E, G, I, H-edge, J2 (wallet draw), C-complete, each in `/home/kali/vigil-wt-pr<X>-*`; independent red-pen on every item incl. post-merge D/F/J (they reached main with builder self-review only — the Wave-1 workflow wf_6947ffdf had no red-pen phase); one fix round on BLOCK. Agents commit locally only; orchestrator pushes/PRs/merges serially. K and L are NOT in that run (user said A-J).

**Why:** the user's goal is a production-complete consumer app; owner-gated items (live-external authority, XML-DSig backend, PDF renderer outside the venv, we-host fleet) stay seams that fail closed. **How to apply:** resume from this state; check `gh pr list` + the workflow journal before acting; merge order #1193 -> PR-3 -> A -> rest. Related: [[vigil-wave-cadence]], [[vigil-apex-program]].

**Status 2026-11-19:** A merged #1192 (A/D/F/J done). Completion workflows wf_7288f6e7 (B,C,E,G,H,I,J2,D-ff,F-ff — build→Opus red-pen) + wf_b0bfc8e3 (K fix-it on existing PCR infra, L attack-path UI) supersede wf_01a9b360-fe3. Build agents commit, never push; orchestrator pushes + serial-merges PASSes. Before pushing ANY branch: run the hidden guards ([[vigil-ui-wiring-matrix-guard]], [[vigil-customer-kit-mirror-guard]], briefing, limitation inventory, two-env).

**Merge train 2026-11-19 (main = dd2d9281 after folders #1195):** open + post-merge-verified + CI running: #1196 L, #1197 K, #1198 B, #1199 G, #1200 I, #1201 J2, #1202 F-ff (all updated onto dd2d9281; battery v2 incl. onboarding + Gate-5 census green). Planned merge order = least shared-file overlap first (F-ff, K, J2, I, G, B, L); every later update = union-resolve config.py flags / customer_app nav+guide / Gate-5 _SCREENS+checklist, then re-run battery v2 post-merge. Still in fix panels: C #1186 (Gate-5 #/connect census, wf_ecb1397a), H #1190 + D-ff + E (wf_ad898fe9). Scripts: scratchpad known_guards2.sh (battery) + watch_pr.sh <pr> <head8>.

**State 2026-11-19 ~17:00 (after plan mode):** green + verified, ready to merge serially: #1197 K, #1196 L, #1201 J2,
#1200 I (all on dd2d9281). Real CI failures diagnosed from logs: #1199 G (fold-cache: flag-OFF report still carries
signed fix guidance — cache key missing the flag); #1202 F-ff (sigil shard backfill exits 3 not 2 — a guard from main
fires first); #1198 B (test_slug_lease exactly-one-winner saw TWO winners — possible real race in main's lease code).
3-lens panels BLOCKED (fix rounds queued): H #1190 (SCIM sign-in can link to ANOTHER member via hash fallback; SAML
replay window when skew>300s; 32 load-bearing guards untested — 16 SAML tests uncommitted + 16 edge patches in
scratchpad/prH-audit); D-ff (issue allowlist documented-not-enforced; non-atomic ledger; dedup cursor unenforced);
E (stale deploy-retest overrides newer contradicting verdict -> false FIXED). C #1186: Gate-5 #/connect registration
done UNCOMMITTED in the worktree (label-only census weakness is pre-existing, separate fix).

**2026-11-20 state:** merged #1196 L, #1201 J2, #1202 F-ff, #1198 B, #1200 I (main 1ff22b07). Open: #1186 C (merged main
1ff22b07 as union, pushed), #1203 Gate-5 census hardening (pushed), #1204 mcp 1.30.0 lock (CVE-2026-115563 turned the
REQUIRED A14 gate red on every branch — merge it FIRST), #1199 G + #1190 H (CI-fixed by shepherds; their adversarial
panels did NOT complete — hold until reviewed). Subagent WEEKLY limit hit 2026-11-20 (resets Mon Nov 24 4pm EST): all
workflows stopped; main loop continues the merge train. Machine rebooted 16:11 → /tmp (and the session scratchpad) wiped;
helper scripts now live in ~/.claude/plans/vigil-tools/ (known_guards*.sh, watch_pr.sh).
