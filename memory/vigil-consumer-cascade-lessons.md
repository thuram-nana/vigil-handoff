---
name: vigil-consumer-cascade-lessons
description: "Hard-won vigil-sovereign merge/CI lessons from the 2026-11-19 consumer cascade — gh --auto merges immediately, gitleaks FP handling, guards that break new consumer PRs, serial-merge conflicts, faithful local repro"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T08:28:19.004Z
---

1. **`gh pr merge --merge --auto` does NOT wait for CI here.** Auto-merge is disabled repo-wide ("Auto merge is not allowed"); on a PR GitHub considers mergeable it merges IMMEDIATELY. That is how PR-J #1189 reached main with a red gitleaks check, turning main red. **How to apply:** never use `--auto` as "merge when green"; verify every check terminal + green on the exact head, then plain `--merge`.

2. **gitleaks false positive -> inline `# gitleaks:allow - <reason>` on that line** (precedent: sigil/vectors/embed.py). A `.gitleaks.toml` `[[allowlists]]` with `condition = "AND"` + paths + regexes muted the ENTIRE file on the local distro gitleaks build (planted secrets on other lines all went undetected); CI pins 8.28.0 but the scoping could not be proven. **Always negative-control any secret-scan suppression** (plant fakes on other lines; they must still be caught). Note the pre-commit job also runs a full-tree `gitleaks detect --no-git`, so one FP reds TWO checks. The FP that hit was `getattr(owner_key, "private_key_b64", "")` (generic-api-key extracts the attribute name).

3. **Guards every new consumer PR tripped this week:** (a) a sovereign module appending a `kind="event"` governor record must join `_GOVERNOR_EVENT_WRITERS` in `apps/sigil/tests/test_latch_writer_shape.py`; (b) honest-limit prose ("not wired", "seam", "-gated", TODO) must be registered in `docs/limitations/inventory.json` (bidirectional) — run the WHOLE `pytest docs/tests -q` (the required "briefing" job); (c) a new oracle/bug class must update standards.py `_STANDARDS` + `_WAVE2_GROUPS`, `test_m3_blind_library.py` and the W16-4 ADR counts (PR-2's `blind_ssti` broke main; a fixed first assertion can unmask a second); (d) a NEW entry in `docs/limitations/inventory.json` also trips `docs/tests/test_assurance_mapping.py` — route it to an objective and re-run `python3 docs/assurance/gen_assurance_mapping.py` (my PR-H limitation fix missed this); (e) an engine-vocab change also needs PR-A's `standards_mirror.json` regenerated (`tools/gen_standards_mirror.py`). Bake these into every builder prompt.

4. **Serial-merge cascade:** append-only `config.py` flags + `test_two_env_boundary.py` `_PROBE` make each merge DIRTY every other open PR, so merges are serial with re-update between. Non-append files (custui.py / provision.py / server.py) conflicted too — ADDITIVE that time, but **inspect hunks before keep-both**; marker removal also drops PEP 8 blank lines, which ruff's default rules don't flag. Mixing server-side `gh pr update-branch` with local merges diverged a branch — reset to origin, then merge.

5. **Faithful local repro:** sovereign = `.venv-sovereign` + `PYTHONPATH=apps/sigil:packages/core/vigil_core:integration` (without `integration` -> 31 collection errors); offense = `.venv-offense` + worktree `packages/core/vigil_core` and `engine/crucible` FIRST (vigil_core is pure Python; the venv copy is stale). The full apps/sigil suite locally is ~40 min and env-noisy (embed golden vectors, backup/home_unit) -> unfaithful; read CI's real failures with `gh run view --job <id> --log | grep "FAILED "`.

6. **Workflow `isolation: 'worktree'` targets the SESSION repo (PENTEST-main), not /home/kali/vigil** — give vigil agents explicit `/home/kali/vigil-wt-*` paths. **Plans saved in the session scratchpad get wiped** — save them to `~/.claude/plans/`.

7. **Write workflow prompts resume-safe from the start.** A rate limit kills agents mid-build; on resume the same prompt re-runs, so a setup step like `git reset --hard` / "tree must be clean" WIPES the partial work. The resume cache is POSITIONAL: editing an early agent() call re-runs every later call, so patch only calls after the cached ones (keep shared prompt text byte-identical). Back up worktrees (diff + untracked tar + bundle) before resuming.

8. **Prove a flaky-test race deterministically before fixing it.** The governor-gates job has timing-sensitive shutdown tests (idle-stop flaked on 6879ae34; test_a_close_waits_for_the_requests_in_flight on 48dc507d, fixed in #1194: `n` was read before the in-flight request reached the fake upstream). Get the REAL assertion from `gh api .../actions/jobs/<id>/logs` (not a paraphrase), then inject the exact interleaving so the original fails with CI's message and the fix passes. My first injection (wrong window) did NOT reproduce — don't ship a fix for an unproven mechanism. Scope injection anchors to the test function (fixture files repeat lines).

**Why:** each item cost real time on 2026-11-19, and #1 let unverified code onto main. Related: [[vigil-consumer-al-program]], [[vigil-wave-cadence]], [[vigil-branch-protection-and-ci-drift]], [[shell-commit-message-backticks]].

**Merge mechanics (2026-11-19, #1195 → dd2d9281):** merge with `gh pr merge N --merge --match-head-commit "$(gh pr view N --json headRefOid --jq .headRefOid)"` after confirming that SHA == the locally verified HEAD and `mergeStateStatus==CLEAN`. A hand-spliced SHA was (correctly) refused with "Head branch was modified" — the guard works; never type the SHA. After each merge every other PR is behind (strict): `git fetch origin main`, `git merge-tree --write-tree HEAD origin/main` to preview conflicts, then `git merge origin/main --no-ff -F <msg with trailer>` and re-run the pre-push battery POST-merge (a clean textual merge can still red the ui-wiring golden / kit mirror / screen censuses).

**Merge-train lessons 2026-11-20:**
- A textually CLEAN merge can be semantically broken: PR-L added `blindVulnBadge` to the shared customer_app
  `findingsBlock`, so any PR-side node harness that slices `findingsBlock` without the blind-vuln region throws
  `ReferenceError` (PR-I's test_customer_proof_badge_ui.py) — git never flags it because the test exists on one side
  only. After every main merge run the merged-area SCREEN tests, not just the conflicted files.
- `docs/limitations/inventory.json` append-conflicts: both sides add an object at the same spot and git puts the
  markers INSIDE the object with a SHARED TAIL after `>>>>>>>` (e.g. disposition/w16_issue). Union = ours + shared tail
  + `},{` + theirs (when ours ends with a comma), else ours + `},{` + theirs; then json.loads to validate. Never
  hand-merge `docs/assurance/assurance-mapping.json` — regenerate it with `docs/assurance/gen_assurance_mapping.py`
  and run test_limitation_inventory + test_assurance_mapping.
- Never `git add` a conflicted file in the same command line as an unchecked resolver (a failed resolver left
  markers that a following `git add` staged).
