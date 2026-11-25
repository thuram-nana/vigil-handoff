---
name: tmp-tmpfs-inode-exhaustion
description: /tmp on this Kali box is a 16 GB RAM tmpfs capped at 1,048,576 inodes — parallel agent test runs exhausted it (ENOSPC looks like a code bug); keep test temp on disk + run the janitor
metadata:
  type: project
---

2026-11-20 ~03:00 EST: /tmp (tmpfs, size 16G, nr_inodes=1048576) hit 100% inodes / 13 GB with ~45 agents running
VIGIL tests: ~159k leaked `tmpXXXXXXXX` mkdtemp dirs + sigil-home-*/op-root-*/actor-journal-*/crucible-cdp-* leftovers,
plus ~400k inodes of my own scratchpad probe/mutation copies. Symptom: tests ERROR/FAIL with "No space left on device"
(e.g. test_customer_proof_export) — an agent or panel can mistake it for a real defect, so re-check any BLOCK whose
repro touches temp files.

**Remediation (owner-approved "delete stale test temp"):** `<scratchpad>/tmp_janitor.sh` deletes top-level /tmp (and
/home/kali/.cache/vigil-tmp) entries owned by kali, >60 min old, matching test-temp patterns, not referenced by any
process cwd/fd/environ/cmdline, never claude*/browser/sockets — freed ~290k entries; looped every 15 min. Batteries
export `TMPDIR=/home/kali/.cache/vigil-tmp` (main disk). **How to apply:** every future agent test leg should prefix
`TMPDIR=/home/kali/.cache/vigil-tmp`; check `df -i /tmp` when tests fail oddly; mutation/probe dirs must be removed by
the agent that made them. Related: [[parallel-development-stacking]], [[crucible-testing-and-gotchas]].

**Disk (2026-11-25):** the big consumer is reviewer scratch under ~/.cache/vigil-tmp/{aegis-r5,launch-r2,invgraph-r5,legalgov-r5}/<lens dirs>
holding 200 MB repo copies (clone/copy/mcopy/repo/merged/wt) that reviewers don't delete. Safe cleanup = only for lenses whose result is
already in the workflow journal, untouched for 30 min, no process cwd inside: remove the .git-bearing copies, KEEP probes/logs
(fixers re-run repros from them). Did this once: 15 copies, +2.3 GB (10.0→12.3 GB free). tmp_janitor.sh alone frees little.
