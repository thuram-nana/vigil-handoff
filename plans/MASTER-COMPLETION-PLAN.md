# Master Completion Plan

PRIVATE. For the owner only. Do not copy this into any repo.

- Built 2026-11-24 from a read-only audit of this machine. Nothing was changed, merged or deleted.
- Reference points: VIGIL `origin/main` = d805c779 (PR #1215). PENTEST `main` = 85986a1.
- Item list (JSON): /home/kali/.claude/plans/master-completion-items.json

**Status words used below**

| Word | Meaning |
|---|---|
| Running now | An agent is working on it right now. Leave it alone. |
| Open PR | The work sits in a pull request, waiting for review or merge. |
| Half done | Work started and then stopped. Nobody is working on it. |
| Not started | Planned. No code yet. |

**Timing (corrected 2026-11-24 ~19:30):** the weekly subagent limit that hit at 16:00 lifted around 18:30. The reviews of PR-H, #1217, #1218 and #1219 are **running now** (workflow wf_39a93661-58b). Nothing waits for Nov 29 any more.

---

## 0. Scope and launch set (owner, 2026-11-24 — overrides the rest of this file)

- **Only VIGIL.** Work here concerns VIGIL, the cybersecurity tool. VIGIL-APEX (X01–X04), SIGIL (S01, S02), RECOR, COMPANY and FLAIRE-AI (O01–O03) are out of scope: not worked on unless the owner asks. CRUCIBLE work goes into VIGIL's engine/crucible; the standalone PENTEST repo items (C01–C13, A03) are out of scope unless they feed VIGIL.
- **Launch code = all of:** open PRs (V01–V05), the six queued fixes (V06–V10), seats (V12), evidence retention (V13), legal S2–S5 and S10–S12 (V15–V21), AEGIS (A01, A02), the multi-language code scan (V14). The investigation graph (V11) and ARTEX R-items ride with it. Legal S6, S7 and S9 are not in the launch set.
- **Deploy (V34)** only when the whole launch set is done AND the owner approves.
- Status 2026-11-24 ~19:50: #1217 MERGED (dceea03a). #1218, #1219, D-ff, E, slug-lease, legal S1 in workflow wf_3ffba888-758 (script ~/.claude/plans/vigil-tools/wf/launch-train-r2.js).

---

## 1. Summary

There are **59 unfinished items** across six projects:

| Project | Unfinished items |
|---|---|
| VIGIL platform | 34 |
| AEGIS | 3 (one covers the 9 slices being built now; one covers the ~63 later slices) |
| CRUCIBLE / PENTEST engine | 13 |
| SIGIL | 2 |
| VIGIL-APEX | 4 |
| Other repos (RECOR, COMPANY, FLAIRE-AI) | 3 |

By status: 11 running now, 4 open PRs, 31 half done, 13 not started.

Most of the half-done work is VIGIL legal/governance slices and old side branches. Two things are at real risk of loss:
- **VIGIL-APEX has no backup remote at all.** Its 337 commits exist only on this disk.
- **Some finished-looking work was never committed:** the VIGIL evidence modules, the SIGIL X kit and the mrbeanpanel engagement files.

Section 5 lists 27 more items that are old experiments, scratch or unclear. The appendix lists about 40 finished items.

**Conflicts between sources, and how I settled them**
- SIGIL hard-prune **Slice D is merged** (PR #21). One note said "not started", but git shows it merged.
- Slug-lease fix: one scan said "ready". The review notes say a fix round is still owed. I followed the review notes.
- PR-H: the newest tip is e4f75129, in /home/kali/vigil-wt-ci-1190-groupfix. The /home/kali/vigil-wt-prH-sso checkout is behind it.
- Consumer D-ff and E: memory notes say "building", but no workflow is running them. I treated both as half done.
- AEGIS branch heads differ a little between git and the workflow's saved state. The tables use the git heads.

---

## 2. Tables per project

### 2.1 VIGIL platform (/home/kali/vigil, GitHub vigil-sovereign)

**Open PRs and live reviews**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V01 | PR-H #1190 SSO/SCIM | Lets business customers sign in with their company login, and adds or removes their staff automatically. Off by default. | Running now | Branch claude/consumer-prH-sso. Newest tip e4f75129 in /home/kali/vigil-wt-ci-1190-groupfix (50 ahead). Also /home/kali/vigil-wt-prH-sso (38ef19d0, behind), /home/kali/vigil-wt-ci-1190 (306ea39f, older base), review rigs ~/.cache/vigil-tmp/rvH7 and rvh7-mut. Review workflow wf_39a93661-58b. | Six review rounds blocked, each finding less. Round 7 fixed all 22 gaps found. The round-7 review is running. | A clean independent review, then CI, then merge. Do not commit the reviewer's probe test copies. | Must merge before the six queued fixes start. | None. You already chose "do not split the PR". |
| V02 | PR #1217 launch pricing | Credit packs and monthly plans in the price list. | Running now (open PR) | claude/launch-pricing-catalog @898a6096, /home/kali/vigil-wt-launch-pricing. Rig ~/.cache/vigil-tmp/rp-launch-pricing/mut. | Built to your Nov 24 prices. | Independent review after Nov 29, then merge. | Live checkout stays off until payments are proven. | Set the private margin setting on deploy day. |
| V03 | PR #1218 terms for individuals | Opens the terms to individuals. Adds the EU 14-day withdrawal right. Requires terms acceptance at Stripe checkout. | Running now (open PR) | claude/terms-individuals @e8603413, /home/kali/vigil-wt-terms-individuals. Rigs ~/.cache/vigil-tmp/rev1218/base and /mut. | Counsel approved the wording. | Independent review after Nov 29, then merge. | Merge before #1219. | Add the Terms URL to Stripe's public details. |
| V04 | PR #1219 support page | Support page for the hosted service: "we aim to reply within 2 business days". | Running now (open PR) | claude/support-page-hosted @c20ad587, /home/kali/vigil-wt-support-page. | Built. Up to date with main. | Review after Nov 29. Merge after #1218. | #1218 | None. |
| V05 | Dependabot PRs #1205–#1214 | Ten routine library updates. | Open PR | GitHub, opened Nov 23. | Made automatically. Not reviewed. | Check CI and merge one at a time. Watch the supply-chain check. | None | None. |

**The six queued fixes**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V06 | Consumer D fix-forward | Hardens notifications: only proven findings may open tickets, and the ledger is made safe. | Half done | claude/consumer-prD-fixfwd @e43cf756, /home/kali/vigil-wt-prD-fixfwd. 19 ahead, clean, no PR. | Fixes built. The independent review never ran. | Review, fix, PR, merge. Keep the withdrawn figure withdrawn unless it is re-measured. | After PR-H. | None. |
| V07 | Consumer E: re-test on every deploy | Re-checks findings each time a customer deploys. The only consumer area not yet merged. | Half done | claude/consumer-prE-ciretest @e093e163, /home/kali/vigil-wt-prE-ciretest. 2 ahead, no PR. | Built. Review blocked it: an old result can override a newer one and wrongly show "fixed". | Fix round, review, PR, merge. | After PR-H. | None. |
| V08 | Slug-lease race fix | Fixes a bug where two processes could both "win" the same lock. | Half done | claude/fix-slug-lease-race @3b6aef28, /home/kali/vigil-wt-fix-slug-lease. 3 ahead, no PR. | Fix built and tested. Review objections are recorded but not yet fixed. | Fix round, review, PR, merge. Rebase on current main. | After PR-H. | None. |
| V09 | Legal-gov S1: settings shell | The customer Settings area that all other legal slices sit on. | Half done | claude/legalgov-s1-settings-shell @689e0a62, /home/kali/vigil-wt-legalgov-s1. 3 ahead, no PR. | Tests pass. Review found 3 medium data-loss bugs: a large profile can read back as blank; a read error can blank fields on save; the backup misses the profile. | Fix those 3, review, PR, merge. | After PR-H. Unblocks S2–S5 and S10–S12. | Policy text comes from you or counsel. Pages stay empty until then. |
| V10 | Investigation graph S1 + S3 fixes | S1 looks up evidence by ID safely. S3 adds charter goals to the graph (off by default). | Running now | claude/invgraph-s1-evidence-resolver @0bc1b2ea (11 ahead) and claude/invgraph-s3-objectives @19263e80 (3 ahead). Worktrees /home/kali/vigil-wt-invgraph-s1-resolver and -s3-objectives. Workflow wf_613eabec-e63. | An earlier review blocked both. Fix rounds are running. | Pass review, PR, merge. | None. | None. Keep the gap matrix private. |

**Rest of the investigation graph**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V11 | Investigation graph S2, S4, S5 | S2 links a finding only to backed evidence. S4 adds fix and re-test steps. S5 adds a coverage view. | Running now | claude/invgraph-s2-evidence-join @b13e5e60 (11 ahead), claude/invgraph-s4-lifecycle @5a60de70 (15 ahead). S5 not built yet. Same workflow as V10. | S2 and S4 built. S5 queued. | Rebase S2 and S4 on the fixed S1, build S5, review, PRs, merge. | Fixed S1 (for S2, S4). S2+S3+S4 (for S5). | None. |

**Launch features still to build**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V12 | Seat billing at launch (includes legal-gov S8) | Plans include seats. Extra members cost $25/month. Rules for adding and removing. A Team screen. | Not started | No branch. Your decision of Nov 24. The old slice S8 is folded in here. | Decided only. | Stripe seat billing, a member limit on invites, the Team screen, a seat-only plan for pay-as-you-go, terms §6 seat text. | After PR-H and #1217. | Counsel wording for terms §6 (needed before merge). |
| V13 | Evidence 90-day auto-delete + privacy text | Deletes raw scan evidence 90 days after an assessment ends. Keeps evidence that backs findings, and keeps reports. Then updates /privacy. | Not started | No branch. Your decision of Nov 24. The backup bucket's 90-day rule is already on. | Schedule decided. | First check that proof bundles still re-verify after deletion. Build the sweep. Then update /privacy. | After Nov 29. | None. |

**Larger program**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V14 | Multi-language code scan | Code scanning for JS/TS, Go, Java, Ruby and PHP at full grade; Rust and C/C++ at a lower grade. Off by default. | Half done | claude/multilang-codescan @8b1b8e7a, /home/kali/vigil-wt-multilang. 5 ahead, no PR. Plan ~/.claude/plans/multilang-codebase-plan.md. | Phase 1 checkpoint (~52 files). Not verified. | Verify phase 1, run six language builders, integrate, review, one PR. | Waits for the AEGIS and investigation-graph load to ease. | None for phase 1. |

**Legal and governance slices** (plan: ~/.claude/plans/legal-governance-gap-matrix.json, private)

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V15 | S2 privacy center | Privacy center and cookie consent. | Half done | claude/legalgov-s2-privacy-center @d4712bb5, /home/kali/vigil-wt-legalgov-s2. 3 ahead. | Built. Objections recorded. | Fix round, review, rebase on S1, PR. Match the cookie text to the Cloudflare scripts on the public site. | S1 | Cookie and subprocessor text. |
| V16 | S3 data rights | "Export my data" and "close my account". | Half done | claude/legalgov-s3-data-rights @f2691734, /home/kali/vigil-wt-legalgov-s3. 10 ahead. | Large, but labelled work in progress. | Verify redaction and deletion scope, review, PR. Match the 30-day reply rule. | S1 | Data-rights policy text. |
| V17 | S4 legal archive | A permanent record of which terms each customer accepted. | Half done | claude/legalgov-s4-legal-archive @120f0c3e, /home/kali/vigil-wt-legalgov-s4. 13 ahead. | Most advanced after S1. Has real restore tests. | Fix round, review, PR. | S1 | Final policy texts. |
| V18 | S5 AI governance | Explains how AI is used and where a human stays in control. | Half done | claude/legalgov-s5-ai-governance @4b77144c, /home/kali/vigil-wt-legalgov-s5. 8 ahead. | Close to complete. Not reviewed. | Review, fix, PR. | S1 | AI-use policy text. |
| V19 | S10 billing issue form | "Report a billing issue". | Half done | claude/legalgov-s10-billing-issue @10c0ff80, /home/kali/vigil-wt-legalgov-s10. 3 ahead. | Mid-build checkpoint. | Finish, review, PR. | S1 | None. |
| V20 | S11 plans + cancel | Self-serve plan change and cancel at period end. | Half done | claude/legalgov-s11-plans-cancel @29447126, /home/kali/vigil-wt-legalgov-s11. 4 ahead. | Mid-build checkpoint. | Finish, review, PR. Align with the #1217 plans. | S1; PR-G (merged) | None. |
| V21 | S12 authorizations page | Lets a customer see the engagement authorizations they signed. | Half done | claude/legalgov-s12-authorizations @f8149f5f, /home/kali/vigil-wt-legalgov-s12. 4 ahead. | Mid-build checkpoint. | Finish, review, PR. | S1; consumer C (merged) | None. |
| V22 | S7 seller identity | Shows SIGIL SARL as the seller on checkout and invoices. | Not started | No branch. | Planned only. | Build it. The plan puts it after S6, but S6 is parked. | S6 (recommend removing this link) | Which company details to show. |
| V23 | S6 tax on checkout | Itemised tax on checkout and invoice. | Half done, parked by you | claude/legalgov-s6-tax @c7f1196e, /home/kali/vigil-wt-legalgov-s6. 1 WIP commit. | Early. | Do not finish for launch. | Seller-of-record / VAT question | Only if you ask for it. |
| V24 | S9 promo codes | Promotional credits and codes. | Half done, parked by you | claude/legalgov-s9-promo @5d358b6e, /home/kali/vigil-wt-legalgov-s9. 1 WIP commit. | Barely started. | Do not finish for launch. Never create Stripe coupons. | None | Only if you ask for it. |

**Uncommitted or stale side work**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V25 | Evidence-build modules (3) | New evidence modules that were never committed. | Half done | /home/kali/vigil-wt-ev-moduleA (sensor edits), /home/kali/vigil-ev-moduleb (evidence_config, staged), /home/kali/vigil-ev-transport-wt (evidence_transport). All on the old base 7cc3ca30. Copies also in the refs/wip snapshots. | Written around Oct 25. Never committed. | Commit to a safety branch. If still wanted, rebase on main and PR. | None | Still wanted? |
| V26 | Main checkout loose edits | The main /home/kali/vigil checkout sits on an old merged branch with uncommitted edits. | Half done | /home/kali/vigil on meridian-s5-expansion. Edited: console/actions.py, console/api.py, vigil_core/rbac.py. Plus untracked docs, targets and scripts. | Edits exist; purpose unclear. | Save the edits to a branch. Move the checkout back to main. Check first that no agent is using it. | None | Keep these edits? |
| V27 | Key-like files in an old worktree | An old, merged worktree holds key and ledger files that were never committed. | Half done (cleanup) | /home/kali/sx-s2 (untracked; contents not opened). | Not committed, which is good. | Move them to safe storage. Confirm none were ever pushed. | None | Your call. Rotate if unsure. |
| V28 | CI nightly split | Moves the heaviest tests to a nightly run. | Half done | ci/crucible-core-nightly-split @70ee495e, /home/kali/vigil-wt-ci-nightly. 1 commit from Sep 29. | One commit, never a PR. About 2 months stale. | Rebase and PR, or park. | None | Still wanted? |
| V29 | Nuclei template pack | Bundles a vetted set of scanner templates. Off by default. | Half done | prod/m1-nuclei-corpus @672f9707, /home/kali/vigil-wt-nuclei. 3 ahead, 1397 behind. | The last review blocked it. A fix was made but not re-reviewed. | Re-review, large rebase, PR. Or park. | None | Revive or park? |
| V30 | Release / build-stamping tooling | Version stamps, a changelog generator and version tests. | Half done | iss-release-tooling-v @56391f2a, /home/kali/vigil-wt-release. 9 untracked files, 1910 behind. | Files written. Only a "salvage" commit exists. | Commit the files to a fresh branch and PR. Or park. | None | Revive or park? |

**Later work and owner-gated items**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| V31 | PCR full live run + JS package cache | One full autonomous fix run that produces a real code change. Plus a safe cache for JS packages. | Not started | Memory note vigil-pcr-program.md. | All blockers fixed. The last run stopped only because API credits ran out. | Top up credits and run. Build the cache. | API budget | Approve an API budget. |
| V32 | Known-limitations burndown (W16) | 28 documented, honest gaps, for example hardware attestation and some checks not yet wired in. | Not started | /home/kali/vigil/docs/limitations/inventory.json (dated Oct 8, out of date). | Each gap has a guard test. | Re-check against current code, close the fixed ones, file the rest as issues. | Some need deployment or hardware. | Choose an external audit team. |
| V33 | Authorship history rewrite | Apply the agreed author trailers to old commits. | Not started | Memory note vigil-authorship-contributors.md. | Policy agreed. Rewrite not done. | Rewrite and force-push. | No open PRs at the time (a force-push breaks them). | Approve the timing. |
| V34 | Production deployment (AWS) | Turn on the live service. | Not started (owner-gated) | Runbook docs/runbooks/AWS-DEPLOY.md. Infrastructure provisioned Nov 23. The server is stopped until deploy day. | Infrastructure ready. | Start the server, install, switch DNS, set up the edge hooks. Turn off the admin access key afterwards. | All launch code done, plus your approval. | Approve each command and type every secret yourself. Confirm the email login, payout currency, scan-engine key and Google branding. |

### 2.2 AEGIS

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| A01 | AEGIS Wave 0 + Wave 1 (9 slices) | The first 9 slices of the AEGIS production program. See the list below. | Running now | Worktrees /home/kali/vigil-wt-aegis-&lt;id&gt;, all cut from main b6d65028. Workflow wf_e1cfb6de-25a. Scratch ~/.cache/vigil-tmp/aegis-r5. No PRs yet. | Builds and fix rounds under way. | Finish, then review panels (at most 2 fix rounds). Every merge needs your recorded approval. | Your approval per merge. | Approve merges. Confirm the assumed decisions D13, D16, D29, D30, D31, D33 and D35. |
| A02 | AEGIS Waves 2–23 (~63 slices) | Gateway, UI, fleet, TLS, edge and DDoS protection, health checks, the staging rig (AS-26), operations, self-security and the rollout steps. | Not started | Plan ~/.claude/plans/parsed-popping-lemur.md. Private threat register ~/.claude/plans/aegis-threat-register.private.json. | Design complete. Approved Nov 19. | Build wave by wave. Prove each on staging (TT-A) first. Fix the top five defects in the threat register first. | W0/W1 merged; staging infrastructure. | Open decisions D2r–D28. Go-live gates R1–R4. Approval per merge. |
| A03 | FORGE RAMPART slice 2+ | The next steps of the proof-carrying web firewall: a TLS front end and agentless checks. | Not started | PENTEST repo /home/kali/Pictures/PENTEST-main. Slice 1 merged (PR #94). Charter docs/FORGE-RAMPART-CHARTER.md. | Slice 1 merged. | Build the next slices under a signed charter. | Charter per slice. | Sign the charter for each slice. |

**A01 detail: the 9 slices** (live git heads)

| Slice | Plain English | Branch head | Commits ahead | Note |
|---|---|---|---|---|
| AS-0 | Governance: rules, records, charters | claude/aegis-as0-governance @68a22afd | 10 | 8 uncommitted files. Fix round over earlier objections. |
| AS-25 | "Done" records for each defence class | claude/aegis-as25-done-records @15eea6a4 | 13 | 2 uncommitted. Fix round. |
| AS-1a | Test rig hygiene | claude/aegis-as1a-tt-rig @7f4218a0 | 13 | Resuming the build. |
| AS-2a | Authority records v2 (revocation, epochs) | claude/aegis-as2a-authority-v2 @74bf4bd4 | 6 | Resuming the build. |
| AS-7a | Licence gate | claude/aegis-as7a-licence-gate @ade8beb0 | 20 | 3 uncommitted. Fix round. The largest slice. |
| AS-27a | Break-glass emergency brake | claude/aegis-as27a-break-glass @2aeba6e7 | 10 | At the review panel. Has a mutation rig. |
| AS-3a | Strict drain and quarantine | claude/aegis-as3a-drain-strict @fa48d7f6 | 7 | Resuming the build. |
| AS-4a | Remove claims that go beyond proof | claude/aegis-as4a-overclaims @f431fb86 | 7 | 2 uncommitted. Fix round. |
| AS-2l | Org proof cascade on revocation | claude/aegis-as2l-proof-cascade @10536bda | 6 | Resuming the build. |

### 2.3 CRUCIBLE / PENTEST engine (/home/kali/Pictures/PENTEST-main)

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| C01 | PR #96 release tooling | A VERSION file, changelog, release workflow and a --version flag on each tool. | Open PR (blocked) | iss-release-tooling @a99bd6d. 2 ahead, 1 behind. | Pushed Aug 23. Mergeable, but checks or review block it. | Rebase, pass checks, merge. Or close. | None | Merge or close? |
| C02 | PR #95 docs-truth audits | A code-checked feature list and limitations list. | Open PR (blocked) | iss-docs-audits-w15 @155c6ba. 2 ahead, 1 behind. | Same as #96. | Same as #96. | None | Merge or close? |
| C03 | Phase C2b: cloud posture launch | Starts cloud and Kubernetes posture checks from the console without a seed target. | Running now (agent worktree) | phaseC2b-cloud-launch @7209ffa, .claude/worktrees/agent-a64493ae40f7c2ccb. 1 commit + 3 uncommitted files. | One slice done. Edits in progress. | Finish, commit, PR, merge. | None | Confirm an agent is still attached. |
| C04 | Phase C3: public-exposure proof | Proves a cloud resource is publicly reachable. | Running now (agent worktree) | phaseC3-active-exposure @7dda1e1, .claude/worktrees/agent-ac2adc9eac7d58743. | Slice committed. Clean. | Review, merge. | C2b | Confirm an agent is still attached. |
| C05 | Phase D: drift watch + zero-retention sessions | Watches for changes over time. Sessions that keep no data. | Running now (agent worktree) | phaseD-drift-ephemeral @34d5980, .claude/worktrees/agent-ae13c249ca5e88c41. 1 commit + 2 staged files. | Partly done. | Finish, PR, merge. | None | Confirm an agent is still attached. |
| C06 | Wave-4 D2: evidence binding | Ties identity, scope and freshness into each evidence certificate. | Running now (workflow worktree) | wave4-D2-binding @f2077fa, .claude/worktrees/wf_61c7559a-768-2. | Slice committed. Clean. | Review, merge. | None | Confirm the workflow is still running. |
| C07 | Forged-token acceptance check | A check that proves a forged login token is really accepted by the target. | Half done | coverage/wave3-4-forgery-acceptance @ac5bc67. Pushed, 1 ahead, no PR. | One commit, months old. | Open a PR and review it against the known soundness rules for this kind of check. Or retire it. | None | Revive or retire? |
| C08 | mrbeanpanel engagement | The first real engagement: 6 findings and 4 reports. | Half done | targets/mrbeanpanel/ (all untracked; attack-tree.md modified). | Charter signed. Findings and reports written. The retest page is an empty template. Finding numbers jump from 005 to 020. | Decide where it is stored. Explain the 006–019 gap. Run the retest after the fixes land. | Fixes on the target side | Where to keep it; the retest window. |
| C09 | Loose root documents | AGENTS.md, SIGIL.md (draft v1.1), benchmark-report.md, crucible-architecture.html and the AEGIS detection-mirror document. | Half done | Repo root, untracked. | Written, never committed. | Commit the ones you want. | None | Which ones belong in the repo? |
| C10 | AEGIS-CRUCIBLE package | Strategy dossier, PCF spec, RAMPART design and demo, 11 builder agents. | Half done | AEGIS-CRUCIBLE-package/ (untracked, 22 files). | Complete as a bundle. | Commit it, or keep it as an export. Keep it in step with the merged code. | RAMPART (#94, merged) | Commit or keep as an export? |
| C11 | Knowledge graph refresh | The code map that agents use is out of date. | Half done | graphify-out/needs_update marker. | Marker set. | Run `graphify update .` and commit. | None | None. |
| C12 | Cloud checks: live runners | The cloud checks are proven offline. The live, credential-gated runners are not built. | Not started | Memory note vigil-e-series-cloud-exploit-program.md. Checks merged (#286–#293). | Offline proof complete. | Build the gated live runners. Run them only on authorized targets. | Real credentials and a signed charter. | Provide authorized test accounts and a charter. |
| C13 | Seven empty iss-* branches | Names for future work: backup/DR, CI hygiene, CI matrix, quality gates, docs claims, signed manifests, supply-chain locks. | Not started | Local branches at a456263. No commits. | Names only. | Prioritise them, or leave them parked. | None | Which ones matter? |

### 2.4 SIGIL (/home/kali/sigil)

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| S01 | Hard-prune Slice E | The step that actually deletes pruned old data, safely even if it crashes, behind 3 owner gates. | Not started | No branch. Slices A–D merged (#18–#21, main 19d33d8). | Groundwork done. | Build both phases, the kill-9 crash test and the "pruned" responses. Pass all 3 gates before the first real prune. | Slices A–D (done) | 3 gate approvals. |
| S02 | X (Twitter) build-in-public kit | Scripts, 12 approved posts, 9 images, guardrails and a runbook for a public X account. | Half done | /home/kali/sigil working tree, untracked: scripts/, content/, strategy/, GUARDRAILS.md, HANDOFF.md, .env.example. README.md rewritten. | Looks finished. Never committed. | Commit it, here or in its own repo. Post only from your own machine. | None | Where to keep it; X keys; go-live approval. |

### 2.5 VIGIL-APEX (/home/kali/vigil-apex)

Note: this session also lists vigil-apex folders as working directories. Check that no other session is editing it before changing anything.

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| X01 | Tip portal rate limit (R-06) | Limits submissions per Tor circuit on the whistleblower tip form. | Half done | feat/gap-onion-rate-limit-real @64c02aa9. 4 ahead of local main. | Looks complete, with tests and a runbook. | Merge into local main. | None | Approve the merge. |
| X02 | No backup remote | The whole repo (337 commits) exists only on this disk. | Half done | /home/kali/vigil-apex. `git remote -v` is empty. | None. | Push to a private remote. | None | Where to host it. |
| X03 | Weaknesses W-10 and W-14 | W-10: a native hardware-key helper. W-14: grow the hallucination test set from 40 rows to about 200. | Half done | docs/weaknesses/W-10.md and W-14.md. | 24 of 27 weaknesses closed. | Build the W-10 helper (planned M3–M4). Grow the W-14 test set. | None | None. |
| X04 | Phase-1 closure gates | Hardware keys, key ceremonies, PGP exchange, legal declarations and partner letters. | Not started (owner only) | docs/TODO_PHASE1_CLOSURE.md. | All code that can ship has shipped. | Owner and architect actions only. | None | Yes, all of it. |

### 2.6 Other repos

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| O01 | RECOR draft PRs #497 and #498 | Two very large draft PRs with the October audit fixes. | Open PR (draft) | /home/kali/RECOR. Heads remediation/2026-10 and national-hosting/tip. | Drafts, never merged. Last updated Oct 25. | Finish and merge, split them up, or close them. | October 2026 audit | Decide their fate. |
| O02 | COMPANY content branches (8) | Bilingual website content: pilots, editions, news items. | Half done | /home/kali/COMPANY. content/* branches on origin. No PRs. | 1–2 commits each. | Check each public claim is true, then merge it. | None | Which ones to publish. |
| O03 | FLAIRE-AI CI fix | Makes the test pipeline pass. | Half done | /home/kali/FLAIRE-AI, branch fix/ci-green-2 @388c296. 3 ahead, no PR. | Done in the branch since May. | PR, merge. | None | Approve. |

---

## 3. Ordered work plan

Items marked **[OWNER FIRST]** cannot start until you give the input named.

**Wave 0: protect work at risk (any time; no merges; nothing deleted)**
1. X02: push VIGIL-APEX to a private remote. **[OWNER FIRST: choose the host]**
2. Commit to safety branches only: V25, V26, V30, S02, C09, C10. C08 too, unless you want it kept local. (VIGIL items also sit in the refs/wip snapshots, so they are less urgent.)
3. V27: secure the key-like files in /home/kali/sx-s2. **[OWNER FIRST]**

**Wave 1: open PRs, in this order**
1. V01 PR-H #1190 (running).
2. V02 #1217 pricing (review running now).
3. V03 #1218 terms.
4. V04 #1219 support (after #1218).
5. V05 Dependabot #1205–#1214, one at a time.
- Decisions only, in other repos: C01 and C02 (PENTEST #96, #95) and O01 (RECOR #497, #498). **[OWNER FIRST]**

**Wave 2: the six queued fixes (start after PR-H merges)**
1. V10 investigation graph S1 + S3 (already running).
2. V06 consumer D fix-forward: run the review.
3. V07 consumer E: fix round.
4. V08 slug-lease: fix round.
5. V09 legal-gov S1: fix the 3 data-loss bugs.
- Merge these one at a time.

**Wave 3: launch features**
1. V12 seats (after PR-H and #1217). Counsel's §6 text is needed before merge, not before building.
2. V13 evidence retention sweep, then the /privacy text.

**Wave 4: AEGIS**
1. A01: finish W0/W1 panels. **[OWNER: recorded approval for each merge; confirm the assumed decisions]**
2. A02: Waves 2–23, one wave at a time, staging first. **[OWNER FIRST: open decisions D2r–D28]**

**Wave 5: rest of the investigation graph**
- V11: S2 and S4 rebased on the fixed S1, then S5.

**Wave 6: multi-language code scan**
- V14.

**Wave 7: legal and governance S2–S12 (after S1 merges)**
1. V17 S4, V18 S5, V15 S2, V16 S3 (most advanced first).
2. V19 S10, V20 S11, V21 S12.
3. V22 S7, once it is decoupled from S6. **[OWNER FIRST: company details]**
- V23 S6 and V24 S9 stay parked.

**Wave 8: everything else**
1. CRUCIBLE: C03–C06 finish (already running); C11; C07 **[OWNER FIRST]**; C08 retest; C13 **[OWNER FIRST]**; C12 **[OWNER FIRST: credentials + charter]**.
2. SIGIL: S01 (3 gates), S02 posting **[OWNER FIRST: X keys]**.
3. VIGIL-APEX: X01, X03, X04 **[OWNER ONLY]**.
4. VIGIL side work: V25, V26, V28, V29, V30 if you revive them **[OWNER FIRST]**; V31 **[OWNER FIRST: API budget]**; V32; V33 when no PRs are open **[OWNER FIRST]**; A03 **[OWNER FIRST: charter]**.
5. Other repos: O02 **[OWNER FIRST]**, O03.

**Deploy gate (V34)**
- Deploy only when the launch code is done and you approve. **[OWNER FIRST]**
- Your rule says "all code done". I recommend you define a smaller launch set (see decision 1 below). With that set, the deploy could happen right after Wave 3 and the launch-relevant parts of Wave 7.

---

## 4. Decisions and inputs still needed from you

| # | Decision or input | Needed for | Recommended default |
|---|---|---|---|
| 1 | What counts as "launch code" for deploy | V34 | Open PRs (V01–V05), the six fixes (V06–V10), seats (V12), retention (V13), legal S2–S5 and S10–S12. Leave AEGIS, multilang and the rest for after launch. |
| 2 | Recorded approval for each AEGIS merge | A01, A02 | Approve each slice only after its review panel passes clean. |
| 3 | Confirm AEGIS assumed decisions D13, D16, D29, D30, D31, D33, D35 | A01 | Confirm them as assumed; the work is already built on them. |
| 4 | AEGIS open decisions D2r–D28 and go-live gates R1–R4 | A02 | Decide D2r–D28 before Wave 2 starts. Keep R1–R4 until staging proof is in. |
| 5 | Terms §6 seat wording | V12 | Send a draft to counsel now so it is ready when the build finishes. |
| 6 | Terms URL in Stripe public details | V03 | Use the hosted /terms page once #1218 merges. |
| 7 | Private margin setting | V02 | Set it on deploy day, outside the repo. |
| 8 | Policy texts: privacy, cookies, retention, AI use, subprocessors | V09, V15–V18 | Counsel supplies them. Pages stay empty (fail closed) until then. |
| 9 | Cookie wording for the Cloudflare scripts | V15 | Name the Cloudflare email-protection and analytics scripts in the cookie notice. |
| 10 | Tax (S6) and promos (S9) | V23, V24 | Keep both parked, as you decided. |
| 11 | Seller details for S7, and its link to S6 | V22 | Show the registered company name, city and registration number. Remove the S6 dependency. |
| 12 | Keep the evidence-build modules? | V25 | Commit them to a safety branch now; decide after launch. |
| 13 | Keep the main-checkout edits? | V26 | Commit them to a safety branch; move the checkout back to main. |
| 14 | Key-like files in /home/kali/sx-s2 | V27 | Move them to your secrets store. Confirm none were pushed. Rotate if in doubt. |
| 15 | Plain-text API-key files in your home folder and on the Desktop | Hygiene | Move them into your secrets store. |
| 16 | CI nightly split, nuclei pack, VIGIL release tooling | V28, V29, V30 | Park all three until after launch. |
| 17 | API budget for the PCR live run | V31 | Wait until after launch. |
| 18 | External audit team | V32 | Pick one after launch. |
| 19 | Timing of the authorship history rewrite | V33 | Do it only when no PRs are open. |
| 20 | Deploy-day details: email login, payout currency, scan-engine key, Google branding | V34 | Settle payouts in USD, since prices are in USD (check with your accountant). Verify the rest on deploy day. |
| 21 | PENTEST PRs #95 and #96 | C01, C02 | Rebase both, re-run checks, merge if green. |
| 22 | Are agents still attached to the PENTEST phase worktrees? | C03–C06 | Let them finish. If none is attached, treat them as half done. |
| 23 | Forged-token check branch | C07 | Open a PR and review it normally. Retire it if it is superseded. |
| 24 | Where to store the mrbeanpanel engagement | C08 | Keep it local and backed up privately. Do not put client engagement data in a shared repo. |
| 25 | Root docs and the AEGIS-CRUCIBLE package | C09, C10 | Commit docs that describe merged code. Keep the package as an export outside the repo. |
| 26 | Which iss-* stubs matter | C13 | Leave them parked. |
| 27 | Credentials and charter for live cloud runners | C12 | Wait until after launch. |
| 28 | SIGIL Slice E gates | S01 | Build it, then sign off each gate. The first real prune only after the crash test passes. |
| 29 | SIGIL X kit: where it lives, keys, go-live | S02 | Commit it to its own private repo. Post only after you add keys and approve. |
| 30 | VIGIL-APEX remote host | X02 | A new private GitHub repo, now. |
| 31 | VIGIL-APEX rate-limit merge | X01 | Merge into local main after its tests pass. |
| 32 | VIGIL-APEX Phase-1 gates | X04 | Owner and architect only. Schedule them yourself. |
| 33 | RECOR draft PRs | O01 | Keep them as drafts. Decide after the VIGIL launch, and split them into smaller PRs if resumed. |
| 34 | COMPANY content branches | O02 | Publish only claims that are true today. |
| 35 | FLAIRE-AI CI fix | O03 | Open a PR and merge it if CI passes. |
| 36 | Missing HANDOFF-SESSION.md (memory notes point to it) | Context | Rebuild it from the memory notes when it is next needed. |

---

## 5. Abandoned, experimental and unclear items

Nothing here should be deleted. Each has one recommendation: resume, archive (leave as is and stop tracking) or ask the owner.

| ID | Item | Where | Why it is listed | Recommendation |
|---|---|---|---|---|
| Z01 | CRUCIBLE_ROOT path-escape fix | /home/kali/vigil-wt-home-paths, fix-crucible-root-escapes-to-home @ae3dcda7 | One "salvage" commit on a 2-month-old base. | Ask the owner: is this fixed elsewhere? If not, redo it on current main. |
| Z02 | prod/ws11b-wave23 | /home/kali/vigil-wt-wave23 @6539bc1e | PR #881 was closed without merge. Later PRs (#874, #879, #919) closed the same issue. | Archive, after confirming the two fixes landed. |
| Z03 | Strix handoff branch | /home/kali/vigil-wt-repoconnect, strix/handoff-20261029 @e8924705 | Month-old handoff notes plus a fix plan with 3 issues. | Ask the owner: check those 3 issues are closed in current main. |
| Z04 | PCR self-approval mode | Memory note vigil-pcr-program.md | Deliberately not built. It would let the agent approve its own actions. | Archive. Keep it off by design. |
| Z05 | Nov 8–13 WIP branches on GitHub | claude/audit-e3a-seam-base, claude/audit-e8-paused-expiry-settles, claude/front-c-prune-pr-a, claude/codeql-clone-symlinks, claude/codeql-console-slug, claude/ci-two-plane-skip-gap, claude/pauseresume-pr1 and -pr2 | Recent work with no PR. audit-e8 was cut off by a usage limit. | Ask the owner. Triage the two codeql fixes and audit-e8 first. |
| Z06 | 327 unmerged remote branches | vigil-sovereign remote | Mostly historical program snapshots from July and older. | Archive. Optionally tag them as historical. |
| Z07 | 38 refs/wip snapshots | /home/kali/vigil refs/wip/* | An automatic safety net of uncommitted work. | Archive (keep). They protect V25 and V30. |
| Z08 | vigil-builder-backups | /home/kali/vigil-builder-backups/20261119-0555 | Snapshots of consumer branches B, C, G, I, J2. All five have since merged. | Archive (keep). |
| Z09 | FORGE Domain 7 anomaly + synthesis domains | PENTEST memory pcf-forge-program.md | A probabilistic signal cannot be a near-zero-false-positive proof. | Archive. Resume only with a different, lead-only design. |
| Z10 | ~40 leftover agent worktree branches | PENTEST local branches worktree-agent-*, worktree-wf-*; idle worktree agent-ae25b462d667e6222 | Old scaffolding from past runs. | Archive. |
| Z11 | Auto-made loopback targets | PENTEST targets/wanr, targets/diag-charter-2037903, targets/diag-unset-2149073 | Machine-made test charters only. | Archive. |
| Z12 | mrbeanpanel UTI demo | PENTEST targets/mrbeanpanel-uti-demo | Unsigned draft demo; replaced by the real engagement. | Archive (or keep as a fixture). |
| Z13 | codescan-demo | PENTEST codescan-demo/ (its own nested repo) | Permanent demo fixture. It contains a key file. | Archive (keep as a fixture). Never publish its key. |
| Z14 | strix_runs output | PENTEST strix_runs/ | Runtime output from one run. | Archive. Add the folder to .gitignore. |
| Z15 | .agents and .codex config | PENTEST repo root | Tooling for other agent harnesses. Not tracked. | Ask the owner: share it in the repo or keep it local? |
| Z16 | Stray framework copy | /home/kali/framework | An unexplained copy of framework/v2. | Ask the owner. Compare it with the PENTEST copy first. |
| Z17 | Stray targets folder | /home/kali/targets | Small scratch folders from old remediation tests. | Archive. |
| Z18 | VIGIL-APEX audit output and Untitled-1 | /home/kali/vigil-apex audit/, Untitled-1 | An earlier audit's output and a loose prompt note. | Archive (keep). |
| Z19 | VIGIL-APEX Python prototype | /home/kali/VIGIL-APEX (not a git repo) | An early spike, replaced by the full monorepo. | Archive. |
| Z20 | FLAIRE (original) | /home/kali/FLAIRE | Replaced by FLAIRE-AI. Its GitHub remote no longer resolves. | Archive. |
| Z21 | RECOR reference-doc build | /home/kali/recor-ref-build | A one-off document assembly folder. | Archive. |
| Z22 | RECOR build caches | /home/kali/recor-check, recor-check-target, recor-test-target | Rebuildable compiler output, about 6.4 GB. | Archive. It can be rebuilt if disk space is ever needed. |
| Z23 | vigil-codescan-demo | /home/kali/vigil-codescan-demo | A small local demo repo. | Archive (keep). |
| Z24 | vigil-demo-output | /home/kali/vigil-demo-output | Finished demo output and a video. | Archive (keep). |
| Z25 | vigil-backups | /home/kali/vigil-backups | October configuration backups. | Archive (keep). |
| Z26 | MAMA documents with no code | /home/kali/Documents/MAMA | Pitch, architecture and concept notes. No repo found. | Ask the owner: is MAMA a planned product? |
| Z27 | Missing HANDOFF-SESSION.md | Expected in /home/kali/vigil | Several memory notes point to it, but it does not exist. | Ask the owner (see decision 36). |

---

## Appendix: already finished (one line each)

**VIGIL**
- meridian-s5 governed-engage FACT classes: PR #941.
- Strix S2 transport intake: PR #865.
- Consumer PR-A presentation and gateway race fix: #1192, #1194 (guardsweep worktree).
- Consumer PR-B posture: #1198 (ci-1198 and prB worktrees).
- Consumer PR-C connect: #1186.
- Consumer PR-D notifications: #1187.
- Consumer PR-F launch queue and its fix-forward: #1188, #1202.
- Consumer PR-G billing: #1199.
- Consumer PR-I proof badge: #1200.
- Consumer PR-J wallet and J2: #1189, #1201.
- Consumer PR-K: #1197.
- Consumer PR-L attack path: #1196.
- Gate-5 census, count-aware: #1203.
- MCP CVE lock: #1204.
- Public terms page and legal edge scripts: #1215.
- Stripe webhook edge route (hooks host): merged (#1216).
- Wave-4 M2.1 fixpoint: #901.
- Wave-5 M3 anti-relabel: #903 area.
- Billing W2b-3 (vigil-wt-main detached checkout): #1134.
- Forge local fix-apply (test-merged worktree): #927.
- WaveA1 seal-at-capture: merged.
- W7 graph-index performance work: merged.

**AEGIS**
- Multi-gateway blocking and UI token fix: PR #944.
- FORGE PCF foundation, Domain 7 slices 1–2, Domain 10 email-auth, standing dual gate: #92, #93.
- RAMPART slice 1 (signed certificate for every inline block): #94.

**CRUCIBLE / PENTEST**
- Coverage M1–M6, veracity layer, intel engine, ops console, SBOM oracle and related programs: #67–#91.
- E-series cloud checks, offline-proven: #286–#293.
- CRUCIBLE_ROOT runtime folder (/home/kali/Music/PENTEST): in place.

**SIGIL**
- Hard-prune Slices A–D: #18–#21.
- Zero open PRs. All remote branches merged.

---

## Added by the completeness check

Appended 2026-11-24 by an independent, read-only completeness pass over the same machine (worktrees, all 327 unmerged remote branches, open PRs on all three repos, every file in the plans dir, the memory program notes, and the git stashes). Nothing else was changed. These are items the main audit did not list individually, plus three corrections.

**New items**

| ID | Item | What it is | Status | Where | How far | What is left | Depends on | Owner input |
|---|---|---|---|---|---|---|---|---|
| M1 | AEGIS-SaaS AE-4b..AE-7 orphan branches | The earlier (Nov 5) AEGIS-SaaS push: AE-4b sovereign monitoring-report spool drainer + tenant monitoring read route; AE-5 per-protected-host monthly billing (monitor_authorize needs an active plan + free slot, block needs the respond tier); AE-6 managed tenant gateway deploy + per-(org,host) report secrets; AE-7 cross-plane acceptance test + offense-leg CI; plus an e2e-verification-25 checkpoint. | Half done, likely superseded | 5 remote branches, no PR, not merged, base 7c74200d: origin/claude/aegis-ae4b-ph9x2k (13 ahead, head 555009d1), -ae5- (13 ahead, 42478e15), -ae6- (13 ahead, 871cc596), -ae7- (13 ahead, 40ccd971), origin/claude/e2e-verification-25-ph9x2k (10 ahead, 2c26adfb). Shared WIP tip cccce012 ("AE-4b WIP ... UNTESTED"). | Committed on the branches; the top commit is explicitly WIP/UNTESTED — tests + red-pen were to land "in the parallel workflow" that never finished. The AE-1..AE-3 / SaaS base of the same session DID merge (#996 plus ops/billing/offboarding/livefire #991–#999). | Confirm whether the monitoring-drainer, per-host billing, managed-gateway and cross-plane-acceptance work is carried by the NEW AEGIS program (A01/A02, approved Nov 19, plan parsed-popping-lemur.md). If yes, archive these branches. If no, salvage the design onto the new waves — do not rebuild blind. | A01/A02 (new AEGIS program) | Is AE-4b..AE-7 superseded by the new AEGIS waves, or is any of it still wanted? |
| M2 | VIGIL salvage git stash | A git stash holding old dirty main state. | Half done (stale) | /home/kali/vigil stash@{0} "On main: salvage-main-dirty-327behind-2026-09-30". | A single stash from Sep 30, 327 commits behind current main. Separate from V26 (the current meridian-s5 checkout). | Inspect, then drop or save to a branch. Almost certainly stale. | None | Keep or drop? |
| M3 | VIGIL-APEX git stash | A stash of the graphify-out knowledge-graph state before an audit step. | Half done (stale) | /home/kali/vigil-apex stash@{0} "On audit/AUDIT-092-trufflehog-merge-fix: graphify-out before AUDIT-092". | One stash; graphify-out is regenerable. | Drop, or regenerate graphify-out. Low value. | None | Keep or drop? |

**Corrections to the main audit (not new work)**

- Z05 lists `claude/pauseresume-pr1-substrate` and `-pr2-verify-refuses` as "recent work with no PR." They are in fact MERGED (PRs #1033, #1034). The genuinely-open Nov WIP in Z05 is the other six: audit-e3a-seam-base, audit-e8-paused-expiry-settles, front-c-prune-pr-a, codeql-clone-symlinks, codeql-console-slug, ci-two-plane-skip-gap (all NO_PR, confirmed).
- Z06 ("327 unmerged remote branches ... mostly historical snapshots from July and older") understates recency. ~35 of the 327 are Nov 6–8 feature branches that are SQUASH-MERGED (PRs #996–#1034) and only dangle because squash rewrites SHAs — they are DONE, not pending. The 5 branches in M1 are the real recent-and-unmerged exception. The bulk "archive" recommendation still holds.
- VIGIL has ~18 worktrees pointing at already-merged branches (ci-1198/1199/1202, prB, prC, prF, prG, prI, prJ2, prL, gate5-census, legal-scripts, terms-page, m21, m3fix, mcp-cve, stripe-webhook-edge, waveA1, aegis-multigateway, guardsweep, the test-merged checkout). This is the VIGIL analog of Z10's PENTEST leftover worktrees: cleanup only, no unfinished work. Prune/remove after confirming no agent is attached.

**Everything else reconciled.** All 14 vigil-sovereign open PRs = V01–V05; both PENTEST open PRs (#95/#96) = C01/C02; SIGIL has zero open PRs; every active/WIP worktree maps to an item; the plans-dir subfolders hold no untracked work (aegis-spec → A02; vigil-tools = owner automation; wip-snapshots-2026-11-19 = A01 backups; scratch-recovered-2026-11-20 = ephemera); and the MEMORY.md "PCR Wave D ... left" and "E-series E2–E5 designed" index lines are stale — Wave D is complete (#792–#795) and E2–E5 merged offline (#286–#293), with the real remainders already captured as V31 (PCR live run + JS cache) and C12 (live credential-gated cloud runners).


---

## ARTEX gap closure (added 2026-11-24 from ~/.claude/plans/artex-gap-analysis.md)

Research used ONLY the public GitHub repo BaaRai/ARTEX. No ARTEX code, prompts or assets copied. The local zip was never opened.
Every item ships default-OFF, lead-only, oracle-authoritative, and extends an existing store (intel store, signed spine, F4 graph, ownership proof log). No new database.

| ID | Item | VIGIL-way design | Status | Effort | ARTEX wave | Depends on |
|---|---|---|---|---|---|---|
| R01 | Merge evidence-verified graph + host-in-scope demote | Merge invgraph S1 then S2; add a demote-only check that the cert's oracle-context host equals the finding's asset host and is in charter scope. | Not started | S | A | V10,V11 |
| R02 | Persistent per-engagement traffic capture index | Default-OFF exchange index inside the evidence tree (host/method/status/body-sha/action_id); Caido history exported at teardown as LEAD-only sandbox-capture; TrafficCorpus points at it. Gateway stays MITM-free. | Not started | M | C | R01 |
| R03 | Org-wide asset inventory | Org-scoped read-model over the intel store + signed spine, keyed by canonical EntityRef id (same key in F4), tenant_id stamped, seen_in per asset, structural SUBDOMAIN_OF edge from the pinned PSL. No new DB. | Not started | M | B | R01 |
| R04 | TESTED anchor for the S5 coverage view | ChainStep -[TESTED]-> asset edge written from the executor's PINNED target (what the gate authorized), keyed by EntityRef id; tested_no_finding needs a retained benign response; coverage states separate from verdict. | Not started | M | B | V11(S5),R03 |
| R05 | Attribution fold over the ownership proof log | Pure fold so revoke/expiry drops attribution automatically; attribute only to the sole current proof-holder (legal-gov S5 current_holders) or a verified apex with opt-in; RDAP/ASN stay displayed LEADs. | Not started | M | B | R03 |
| R06 | Charter scope hardening | Refuse public-suffix and bare-TLD wildcards at authority mint (pinned PSL, fail-closed); CIDR only as an owner-signed netblock with a minimum prefix and the .gov/.mil/.edu/.int floor. | Not started | S | B | None |
| R07 | Operator replay pane over the existing Repeater | Default-OFF console route + UI pane calling ONLY the gated Repeater facade; seeded from an evidence exchange or capture-index row; hygiene refusals; results are provenance-labelled observations with 'hand to oracle'. | Not started | M | C | R02 |
| R08 | By-asset findings tree + fixed-grammar filter | Read-only tenant-scoped tree counting CONFIRMED and LEAD separately; allowlisted-field filter (no SQL/eval); console + customer app. | Not started | M | B | R03,R04 |
| R09 | Edge-triggered multi-cycle loop | Coordinator wakes on blackboard append with debounce + heartbeat patrol; engage_autonomous extended to a budget/watchdog-bounded multi-cycle loop; serial for PROOF; default-OFF. | Not started | L | D | None |
| R10 | LLM goal decomposition into objectives | Kernel binding splits the charter objective into falsifiable objectives (refute_on + oracle-class mapping) feeding planner/seed.py; archetype-seed fallback; persisted via invgraph S3. | Not started | M | D | V10(S3) |
| R11 | Per-leaf claim lease (SEARCH phase only) | Extend slug_lease to leaf granularity: claim nonce, heartbeat, sweep returning crashed leaves to the frontier, explicit terminal states; PROOF stays serial. | Not started | M/L | D | V08 |
| R12 | Per-leaf control: pause / cancel / steer | Per-leaf steer queue drained before the next gated tool call (WARDEN pre-exec hook shape); per-leaf recoverable cancel; typed abort causes; surfaced in the customer app. | Not started | M | D | R11 |
| R13 | Narrow cross-session inheritance | retrieve_priors unions consented partitions: asset nodes as GROUNDING_INTEL (charter-filtered) + terminal outcomes/failure lessons as dead-end priors; wire query_prior_chains; read-only, non-transitive, consent-gated. | Not started | M | D | R03 |
| R14 | Background enrichment queue | Deterministic enrichment on the opt-in discover path; per-(kind,id) cooldown on monotonic seq; every probe via the gated HttpExecutor with retained exchange; INTEL only, never FACT. | Not started | M | D | None |
| R15 | Graph compaction (hot/cold digest) | Read-only digest view over F4 GraphView; SUMMARY digests cite member ids + Merkle root (chainast.merkle_root); members never deleted; wire chainast into the think loop. | Not started | M | D | None |
| R16 | bbot import parser | Typed fields only -> LEAD observations; scope_distance kept as attribute; data/data_json inert. | Not started | S | C | None |
| R17 | nmap -oX file import | Reuse the sensor XML normaliser + service_observations minter; lead-only; item cap + truncation reporting. | Not started | S | C | None |
| R18 | HAR import/export | Import as a passive Tier-1 sensor (GROUNDING_INTEL, out-of-scope hosts dropped); export evidence exchanges redacted, each entry carrying action_id/cert digest. | Not started | S | C | R02 |
| R19 | Declarative request-rewrite rules (request phase only) | No hook VM; rules applied inside VIGIL's gated send path, WARDEN-gated, default-OFF; rule id+digest recorded; any response-rewritten exchange ineligible for minting. | Not started | M | E | None |
| R20 | WebSocket/SSE frame recording | Ordered bounded frames artifact per stream in the action evidence dir (manifest-hashed); no separate store. | Not started | S | C | R02 |
| R21 | SOCKS5 CONNECT front end | Default-OFF, calls the same authorize(); UDP ASSOCIATE stays refused; metadata-only until the capture index exists. | Not started | M | E | R02 |
| R22 | Idle-turn nudge | Detect a no-action/no-text think turn, re-prompt once (cap ~2), suppress on provider refusal, then ASK_USER; bounded by budget + watchdog. | Not started | S | E | None |
| R23 | LLM backend health memory | In-process cooldown per backend that reorders permitted_preference(); re-asserts sovereignty; never leaves the tier; advisory only. | Not started | S | E | None |

**Where they slot into the ordered plan**
- ARTEX wave A (R01) = master Wave 5 (investigation graph S1/S2, already running) plus one new demote check.
- ARTEX wave B (R03–R06, R08) = master Wave 5, as new investigation-graph slices S6+ after S5 merges. R04 extends S5 (S5 as specified has the benign-record rule but no structural TESTED edge).
- ARTEX waves C–E (R02, R07, R09–R23) = master Wave 8, CRUCIBLE engine work, after launch.

**Deliberately NOT adopted:** an ungoverned Kali-Bash worker; LLM-mandatory mode; TLS MITM on the containment plane; response rewriting that can reach an oracle; UDP egress; authorization from declared scope rules; ICP-licence and keyword scope rules.

**Gap in the research:** the "vulnerability-testing tools and skill library" area agent was stopped by safety filters, so ARTEX's tool catalogue was not compared. Everything else was.


---

## APEX (VIGIL's autonomous pentester program) — added 2026-11-24

**Owner launch decisions (2026-11-24):** launch ALL three layers for customers; impact is RE-CHECK ONLY at launch (live impact AP09 stays owner-gated); OOB collector = oob.vigilguard.net; ADD a master VIGIL_APEX_ENABLED kill-switch (AP11). AP09 drops out of the launch set; AP02/AP03/AP07/AP11 are in.

Not the VIGIL-APEX procurement project. PRs #1181, #1182, #1183, #1184, #1185 are MERGED (Nov 18–19) but every APEX flag is default-OFF and no deploy env, unit or runbook turns it on. Full report: ~/.claude/plans/apex-status.md (private). AP05 is already being fixed in the consumer-E round (wf_3ffba888-758). Treated as launch code under the owner's "only VIGIL" scope.

| ID | Item | What | Status | Effort | Depends on | Owner input |
|---|---|---|---|---|---|---|
| AP01 | APEX enablement path in deploy env + runbook | Add every APEX flag (autonomous, oob, impact, presentation, consumer surfaces) to customer.env.example + cockpit.env.example (identical on both planes), to OWNER-SWITCHES.md, and an APEX enablement page in GO-LIVE-quickstart.md. Today there is NO supported way | Not started | S | None | Decide which APEX layers launch (engine autonomy / OOB-first / impact-closure) and which consumer surfaces (posture, attack-path, blind-vuln |
| AP02 | Production driver for PCR close-the-loop | Build a sovereign CLI subcommand + systemd/worker that injects governance signers, GitHub App material, the m-of-n destruction quorum and the patched-build redrive into remediation.apex_pcr.close_remediation_loop, so a confirmed FACT drives a gated fix PR + si | Not started | L | AP01 | Supply target repo/branch + governance signer quorum; approve opening real fix PRs (m-of-n), since the PR leg touches a target repo. |
| AP03 | Production ticker for continuous retest scheduler | Add a cadence job (systemd oneshot+timer mirroring vigil-reprove) that loads a persisted RetestRegistry, populates it from confirmed findings, injects redpen_fix.refire_reproduction as refire, calls jobs.apex_retest_scheduler.run_cadence_tick on a cadence, and | Not started | M | AP01 | Decide the cadence and where the registry file lives; approve enabling the timer and VIGIL_APEX_IMPACT. |
| AP04 | Provision a public OOB collector host | Stand up a public TLS reverse proxy fronting the loopback relay (advertise_url); an owner-owned domain delegated (NS) to the DNS collector bound on routable :53; set CRUCIBLE_OOB_RELAY_SECRET + LiveConfig oob_enable/oob_relay_host/oob_advertise_base_url; run t | Not started | M | None | Provide the public collector host, a dedicated DNS name + delegated base domain, open :53 and the TLS port, and run the governance-signed pr |
| AP05 | Unify the VIGIL_APEX_IMPACT flag reader | Make sigil.config.apex_impact_enabled delegate to vigil_integration.apex_flags.apex_impact_enabled (one shared reader, fail-closed on ImportError), OR align the truthy sets. Today the sovereign reader accepts 'enabled' (5-element set) while the engine + integr | Not started | S | None | Decide whether to land the consumer-E fix or patch the divergence directly on main before any sovereign-plane consumer ships. |
| AP06 | Driver resume durability | On --resume the autonomous driver is skipped (no open executor) and its oracle-confirmed findings are never re-persisted (report is persisted at scan-complete, before the driver folds its findings), so a resumed run loses them. Either re-snapshot the report af | Not started | M | None | Decide the intended resume semantics: recover-and-skip (re-persist), re-run on resume, or documented non-resumable. |
| AP07 | Owner-gated customer-path driver wiring | The customer launch worker reaches run_engagement, but the flag is wired into no deploy env / UI / launch gate, and the separate live/engine.py 'unified OODA engine' never runs the APEX driver. If hosted customers should ever run it, add explicit owner-gated d | Not started | M | AP01 | Confirm whether PR-1 is operator/CLI-only for now, or whether hosted customers should run the autonomous driver (owner-gated at go-live). |
| AP08 | Widen the autonomous discovery arsenal | The in-band discovery probe is 5 near-zero-FP checks (XSS/open-redirect/path-traversal/SSTI x2). SQLi is excluded (single-shot FP) and OOB (SSRF/XXE/RCE) self-skips without a receiver. So injection/ssrf-oob/auth/access-control/business-logic/exploit-chain only | Not started | L | AP04 | Confirm the intended division of labor: PR-1 discovers only the 5 in-band classes; OOB/SQLi discovery arrives via PR-2 provisioning + future |
| AP09 | Wire the in-engagement live-impact arm for authorized prod | engage calls pursue_impact with no gate and no live_prover, so in production chaining-to-impact is offline oracle re-fire only; the live data-access/privesc/ATO demonstration runs only in tests. If live impact pursuit is intended in prod, add an authorized pro | Not started | M | None | Owner approval + prod-execution authority (VIGIL_CUSTOMER_PROD_EXECUTION + signed charter) before any live impact pursuit touches a target. |
| AP10 | Reconcile Gate-5 sign-off with the shipped config | The owner's Gate-5 E2E walk turns the presentation surfaces ON (posture + presentation; attack-path/blind-vuln), but customer.env.example ships them OFF, so a green Gate-5 does not reflect what the hosted launch serves. Either ship the surfaces on (set the fla | Not started | S | AP01 | Confirm whether Fix-first / Attack-path / Retest are part of the launch surface. |
| AP11 | Master VIGIL_APEX_ENABLED kill-switch (or record the decision) | Appendix Z specified a master VIGIL_APEX_ENABLED (default OFF) ANDed into each sub-flag. It was never implemented; only independent sub-flags exist, so there is no single off-switch for the whole program. Either implement it or update the plan to record that t | Not started | S | None | Confirm whether a single master kill-switch is wanted for operations. |
| AP12 | Refresh V2-LIMITATIONS and add an APEX residuals section | V2-LIMITATIONS.md section 31 still calls the nosqli/ldap/xpath datastore_error branches 'LEAD-only (fact_capable: false)', but #1182 promoted them to fact_capable=true via the parser-state-toggle oracle. Rewrite section 31, and add an APEX residuals section (p | Not started | S | None | None. |
| AP13 | Render impact_report in reports/UI | EngagementResult.impact_report is computed but surfaced nowhere — no report renderer or UI consumes it, so the demonstrated-impact deliverable is invisible to the operator. Render demonstrated paths, terminal_impact, foothold_findings and authorization in the  | Not started | S | None | None. |
| AP14 | Range-backed MERIDIAN e2e acceptance | The committed PR-1 e2e acceptance runs against a loopback pytest_httpserver, not the MERIDIAN range the plan names. The promote->crawl->probe->oracle chain is faithful and the capability is demonstrated end-to-end on an equivalent target. Add a MERIDIAN-range- | Not started | S | None | Confirm whether the loopback-equivalent is an acceptable stand-in for the MERIDIAN acceptance, or a range-backed e2e is required. |
| AP15 | Reconcile customer-facing copy with shipped flags | Add when-gates to the in-app Guide steps (attackpath/retest/OOB-badge) so the Guide matches the served surface, and review public-site 'autonomous pentesting engine' / 'verified fix suggestions' copy against the shipped defaults. Either enable the capabilities | Not started | S | None | None if the flags are enabled; otherwise approve the one-line guide gating fix + copy softening. |


---

## Red-Pen go-live (GitHub App audit, 2026-11-24, wf_c0b97393-8a0)

App settings are correct for today (permissions, events, public = "Any account", URL https://hooks.vigilguard.net/webhooks/github-app). Red-Pen is NOT go-live ready on the VIGIL side. Launch code (consumer Area 4 / E).

| ID | Item | What | Effort | Branch |
|---|---|---|---|---|
| RP01 | Hooks edge forwards POST /webhooks/github-app | Add a third byte-exact POST-only route to vigil-hooks.nginx.conf, vigil-hooks.Caddyfile, infra/edge/Caddyfile.example; extend test_hooks_webhook_edge.py WEBHOOKS (+ resolve _GH_APP_WEBHOOK_PATH, proxy_pass count 3; /webhooks/github stays 404); AWS-DEPLOY + GO-LIVE say three routes + add the path to the Cloudflare WAF skip rule. | S | main (independent) |
| RP02 | App private key: SSM reader accepts a multi-line PEM | vigil_core.aws_ssm.read_secure_string rejects values that are not one line (aws_ssm.py:83) and parses --output text, so /vigil/gh-app-private-key (a PEM) fails closed. Give github_app._private_key a PEM-aware SSM read (--output json, keep fail-closed + never logged) with tests incl. a negative control. | S | main (independent) |
| RP03 | Webhook secret delivery to the proxy | uiproxy reads VIGIL_GH_WEBHOOK_SECRET only from its own env (service.env); no SSM reader. Add a boot oneshot/ExecStartPre that loads /vigil/gh-webhook-secret into the proxy env (no plaintext file), or document service.env + cockpit.env placement; doctor must check the PROXY env. | S | main |
| RP04 | Red-Pen env + units + IAM in code | Env examples + runbook for VIGIL_REDPEN_ENABLED (proxy, cockpit, customer, offense workers), VIGIL_GH_WEBHOOK_SPOOL, VIGIL_GITHUB_APP_ID=5804935, VIGIL_GITHUB_APP_SLUG=vigil-red-pen, VIGIL_GITHUB_APP_PRIVATE_KEY_SSM=/vigil/gh-app-private-key, VIGIL_AWS_REGION; fix E's placeholder path /vigil/github-app/private-key; systemd units for redpen-webhook-drain + redpen-review with a dedicated --queue-root; provision.sh grants the 5 /vigil/ ARNs. | S/M | after E merges |
| RP05 | Repo connect verifier | installation_verifier is never injected, so every install callback returns verification_required and no connected-repo record is ever made. Wire an OAuth verifier (client id/secret in SSM, GET /user/installations) or an owner-only `sigil redpen connect <org> <installation_id>`. Owner then sets Setup URL https://app.vigilguard.net/api/customer/code/connected + Redirect on update + OAuth during install. | M | after E |
| RP06 | Review execution live path | redpen-review reserve is hard-wired to refuse, reviewer proposes nothing, clone/comment use a PAT: sovereign reserve publisher, live reviewer, installation-token broker, repo-slug authority (HANDOFF O-6 live-flip items). | L | after RP05 |
| RP07 | Check writer posts a check for refused reviews | A refused webhook review has empty org_id and redpen_check.py:184 (E) rejects it, so no check at all appears; resolve the org sovereign-side by repo (or carry it) so a neutral check is written. | S | in E (before merge) |
| RP08 | Least-privilege installation tokens | mint_installation_token posts {} (all repos, all perms); mint with repositories=[repo] + permissions={checks: write} for the check write. | S | main |
| RP09 | Installation webhooks reconcile disconnects | installation deleted/suspend + installation_repositories removed must disconnect ConnectedRepoRegistry records (today acknowledged only). | M | after RP05 |
| RP10 | Doctor covers the Red-Pen chain | vigil doctor (proxy env: flag+spool+secret together, spool draining) and sigil doctor (JWT build without egress = SSM read + PEM parse, slug). | S | after RP02/RP04 |

Owner deploy-day: Cloudflare A `hooks` -> 54.73.236.42 **Proxied**; add /webhooks/github-app to the hooks WAF skip rule; when RP05 lands set Setup URL + Redirect on update + OAuth during install; delete the old 2026-11-17 key.
