---
name: vigil-a14-multigate-lockbump
description: "VIGIL's A14 supply-chain gate is a FOUR-gate system and a dependency bump must satisfy all four PLUS two downstream guards; a new transitive CVE turns A14 red repo-wide independent of any PR"
metadata:
  node_type: memory
  type: project
  originSessionId: 7758e121-f349-47d5-886b-6bb5a1d60e27
  modified: 2026-10-02T09:07:35.833Z
---

A dependency/lock change in vigil-sovereign must clear MORE than Trivy. Learned bumping urllib3
2.7.0→2.8.0 + fastembed 0.8.0→0.8.1 (the #889 keystone, 2026-10-02). The A14 job
(.github/workflows/supply-chain.yml) runs these sub-gates, ALL blocking:
1. **lock staleness** — regenerates each pip-compile lock with EXACT flags `pip-compile --quiet
   --generate-hashes --no-header --strip-extras` seeded from the committed lock, from the lock's
   canonical CWD (sovereign=repo root; offense=engine/crucible), pinned **pip-tools==7.6.1**
   (match it locally or the lock byte-differs). Hashes are sorted ALPHABETICALLY by sha256 — a
   hand-edit in the wrong order fails. To bump a transitive: `pip-compile --upgrade-package <pkg>`;
   verify byte-stability by re-running a plain seeded regen on the result (no diff).
2. **Trivy vuln gate** — `trivy fs --file-patterns 'pip:.*\.lock\.txt$' --severity HIGH,CRITICAL`.
   Scans every `*.lock.txt` + auto-detects `vendor/strix/uv.lock` (uv). Does NOT scan
   `infra/supply-chain/strix.lock` (ends `.lock`, not `.lock.txt`) nor the cdx.json image SBOM.
3. **committed OFFENSE SBOM** — `python3 -m framework.v2.tools.gen_sbom --check` (engine/crucible);
   a lock bump drifts `framework/v2/sbom.json` → regenerate with `gen_sbom` (stdlib, no net).
4. **committed Strix image SBOM** — `gen_image_sbom.py --check` ties to the DOCKERFILE hash + image
   refs, NOT pip versions, so a lock urllib3 bump does NOT require touching
   `sbom-strix-sandbox.cdx.json` (leave it honest to the un-rebuilt image; it regenerates at deploy).

PLUS two guards OUTSIDE A14 (they fire in the two-env/P5 leg):
5. **vendored-strix patch-series** — any change to `vendor/strix/**` (incl. its uv.lock) drifts
   `docs/strix-patches/series.json` → `test_strix_patch_series.py` fails. Fix: commit the vendor
   change, then `python3 tools/vendoring/regen_strix_series.py` (derives a patch from the commit +
   updates the manifest; needs git history, no net).
6. **importorskip offense-leg listing** — any integration test with `pytest.importorskip("framework…")`
   skips in the sovereign/two-env leg, so `test_every_framework_importorskip_test_runs_in_the_offense_leg`
   REQUIRES it be listed on a real run-list line (not `--ignore=`, not `#`) in ci.yml's offense leg
   (~line 517+) or it never runs anywhere. [[vigil-crucible-core-gateway-importorskip]]

**A fresh transitive CVE (e.g. urllib3) turns A14 red on EVERY open PR at once** — it is not any PR's
fault; fix = bump the dep across all locks on main (the keystone PR) and others clear on rebase.
Network: github.com resolves with `dangerouslyDisableSandbox`, pypi.org via curl too, but
`pip-compile`'s own resolver may DNS-fail intermittently — curl the PyPI JSON for hashes if so.
[[shell-commit-message-backticks]] [[vigil-branch-protection-and-ci-drift]]
