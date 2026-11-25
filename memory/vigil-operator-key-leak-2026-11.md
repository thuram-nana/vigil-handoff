---
name: vigil-operator-key-leak-2026-11
description: "2026-11-19 security incident — a stray operator.key (Ed25519 incl. private half) + usage-ledger.jsonl + planreq.spine were committed at the repo root on public branch claude/ci-two-plane-skip-gap; assessed untrusted, branch force-pushed clean"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-11-19T08:53:16.991Z
---

**Found** 2026-11-19 by an all-refs `gitleaks detect --log-opts=--all` while triaging the CI history findings: commit `419ffe56` (2026-11-11, an earlier Claude session, "merge main; cross-plane guard and plugin ready for review") added runtime artifacts at the repo ROOT — `operator.key` (usage-attestation Ed25519 keypair JSON with `private_key_b64`), `usage-ledger.jsonl`, `planreq.spine`. Branch `claude/ci-two-plane-skip-gap`, pushed; **the repo is PUBLIC**. Main was never affected (main's `.gitignore:56` ignores `/operator.key`).

**Assessed (public-key fingerprints only, never the private half):** leaked fp `2c1c313b…` matched none of the 6 operator keys on the machine (real one: `/home/kali/vigil/.vigil-live/operator.key` fp `2b073bbe…`), and the public key was pinned in no config → stray dev key, **no rotation needed**. Ledger PII: git_email/git_name already public in commit metadata; os_login generic; target loopback; only new datum a 2-char hostname.

**Remediated** (user said "go"): lease-guarded force-push of the branch to its last clean commit `ce8102c5` — the 3 artifacts are 404 on the branch, all 3 `wip(ci)` commits (~790 lines of unmerged CI-guard work) preserved, rescan of all origin refs shows 0 artifact findings.

**Residual:** GitHub still serves `419ffe56` by SHA as a dangling object until its GC; a GitHub Support purge request is optional (owner action; low value since the key is untrusted). Treat that key as compromised forever — never pin it.

**Why it happened / how to apply:** a run with CWD = repo root wrote runtime state there, then a broad `git add` committed it on a branch that predated the `.gitignore` rule. When triaging secret findings, scan **all refs**, not just main — CI's history count (7) never showed this one. Related: [[vigil-consumer-cascade-lessons]], [[vigil-consumer-al-program]].
